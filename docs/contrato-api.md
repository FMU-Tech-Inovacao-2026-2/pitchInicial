# Contrato da API

Este documento descreve as rotas implementadas no back-end Node.js + Express + Zod. A autenticação usa JWT emitido e validado pela própria API. Usuários, extrações e planos ficam em memória. Todas as contas e sumários são fictícios.

Rotas protegidas recebem `Authorization: Bearer <accessToken>`. O servidor valida assinatura, expiração e perfil em cada rota. Erros seguem `{ "erro": { "codigo": "...", "mensagem": "..." } }`. Respostas autenticadas usam `Cache-Control: no-store`.

## Rotas

| Método | Rota | Perfil | Resultado |
| --- | --- | --- | --- |
| GET | `/health` | Público | Estado da API, versão e modo de extração |
| POST | `/auth/login` | Público | Emite JWT e usuário público |
| GET | `/auth/me` | Autenticado | Usuário da sessão |
| GET | `/pacientes` | Profissional | Pacientes fictícios disponíveis |
| GET | `/alta/sumarios` | Profissional | Sumários fictícios selecionáveis |
| GET | `/alta/termo-aceite` | Profissional | Termo de aceite vigente (versão e declarações) |
| GET | `/alta/extracoes` | Profissional | Extrações do próprio profissional, sem texto clínico |
| POST | `/alta/extracoes` | Profissional | Extrai sugestões a partir da base, texto ou arquivo já lido |
| GET | `/alta/extracoes/{id}` | Profissional | Extração do próprio profissional |
| POST | `/alta/extracoes/{id}/aprovar` | Profissional | Aprova sugestões selecionadas e verificadas, com aceite |
| GET | `/alta/extracoes/{id}/plano` | Profissional | Plano e comprovante de aceite de uma extração aprovada |
| GET | `/alta/planos/me` | Paciente | Plano aprovado do paciente autenticado |

Login: `POST /auth/login` recebe `{ "email": "...", "senha": "..." }` e devolve `{ "accessToken", "tokenType", "expiresIn", "user": { "id", "nome", "email", "role" } }`. As contas fictícias de desenvolvimento estão no README do back-end.

## Criar extração

O corpo é discriminado por `origem`. Todos os casos exigem `paciente_id`.

```json
{ "origem": "base", "paciente_id": "u-002", "sumario_id": "sumario-cardiaco-01" }
```

```json
{ "origem": "texto", "paciente_id": "u-002", "texto": "Texto fictício do sumário..." }
```

```json
{ "origem": "arquivo", "paciente_id": "u-002", "nome_arquivo": "alta.txt", "texto": "Texto extraído localmente do arquivo..." }
```

O front lê TXT e PDFs com camada de texto e envia só o conteúdo. O back-end não recebe nem armazena o binário. A resposta é a extração com `status: "pendente"`, `sugestoes`, `nao_informado`, `texto_original` e identificadores. O servidor calcula `origem_verificada` comparando cada citação com o texto original; o que o modelo declara não basta.

Com `GEMINI_PROVEDOR=demonstracao`, só os sumários do catálogo geram sugestões. Um deles traz, de propósito, uma citação inexistente, que chega com `origem_verificada: false`. Textos livres recebem `503 SERVICO_INDISPONIVEL`.

## Termo de aceite

`GET /alta/termo-aceite` devolve o texto vigente. O servidor é a fonte única; o front exibe exatamente essas declarações.

```json
{
  "versao": "2026-11-v1",
  "titulo": "Termo de aceite profissional",
  "resumo": "As sugestões foram organizadas por inteligência artificial...",
  "declaracoes": [
    { "id": "revisao_integral", "texto": "Revisei cada item aprovado e conferi o trecho de origem..." },
    { "id": "ciente_limitacoes_ia", "texto": "Estou ciente de que a IA pode omitir, interpretar mal ou errar informações..." },
    { "id": "decisao_profissional", "texto": "Assumo a decisão profissional sobre o conteúdo aprovado..." },
    { "id": "finalidade_lgpd", "texto": "Uso os dados de saúde deste paciente somente para a tutela da sua saúde... (Lei nº 13.709/2018, art. 11, II, \"f\")." }
  ]
}
```

A redação é acadêmica e precisa de validação jurídica antes de uso com pacientes reais.

## Aprovação e plano

`POST /alta/extracoes/{id}/aprovar` recebe os IDs aceitos, as edições opcionais e o aceite:

```json
{
  "itens_aprovados": ["<id-sugestao>"],
  "sugestoes_editadas": [
    { "id": "<id-sugestao>", "titulo": "Título revisado", "descricao": "Descrição revisada" }
  ],
  "aceite": {
    "versao_termo": "2026-11-v1",
    "declaracoes": ["revisao_integral", "ciente_limitacoes_ia", "decisao_profissional", "finalidade_lgpd"]
  }
}
```

Regras validadas no servidor:

| Situação | Resposta |
| --- | --- |
| Nenhum item aprovado ou corpo inválido | `400 DADOS_INVALIDOS` |
| Versão do termo diferente da vigente ou declaração faltando | `422 ACEITE_INCOMPLETO` |
| Edição de item não selecionado ou item sem origem verificada | `403 SEM_PERMISSAO` |
| Item que não pertence à extração, ou extração de outro profissional | `404 NAO_ENCONTRADO` |
| Extração já aprovada | `409 CONFLITO` |

A citação original continua ligada ao item editado. A resposta é o plano aprovado, com o registro do aceite:

```json
{
  "id": "<id-plano>",
  "paciente_id": "u-002",
  "profissional_id": "u-001",
  "extracao_id": "<id-extracao>",
  "itens": [ { "id": "...", "categoria": "medicamento", "titulo": "...", "descricao": "...", "trecho_origem": "...", "origem_verificada": true } ],
  "criado_em": "2026-11-27T12:00:00.000Z",
  "aceite": {
    "versao_termo": "2026-11-v1",
    "declaracoes": [ { "id": "revisao_integral", "texto": "..." } ],
    "profissional_id": "u-001",
    "profissional_nome": "Dra. Ana Demo",
    "registrado_em": "2026-11-27T12:00:00.000Z",
    "hash_conteudo": "<sha-256 de extração, versão do termo e itens aprovados>"
  }
}
```

O paciente consulta somente o próprio plano em `GET /alta/planos/me` (`404` enquanto não houver plano). O profissional reabre o comprovante em `GET /alta/extracoes/{id}/plano`.

## Auditoria

A API registra eventos de login, extração e aprovação com IDs, contagens, versão do termo e hash, sem texto clínico, senha ou token.

## Limites atuais

- Catálogo, usuários, extrações e planos ficam em memória e são apagados quando o servidor reinicia. Por isso o deploy usa uma única instância.
- O modo demonstração do front reproduz estas regras no navegador e não chama a API.
- A extração real usa Gemini pela Gemini API (`GEMINI_PROVEDOR=api-key`) ou pelo Vertex AI (`GEMINI_PROVEDOR=vertex`).
- Prescrição Assistida e Paciente Assistido ainda não têm rotas.
- Não use dados reais de pacientes. A IA organiza o conteúdo recebido; não diagnostica nem recomenda tratamento.
