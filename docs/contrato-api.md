# Contrato da API

Este documento descreve as rotas implementadas no back-end Node.js + Express + Zod. A autenticação usa JWT emitido e validado pela própria API. Usuários, extrações e planos ficam em memória. Todas as contas e sumários são fictícios.

Rotas protegidas recebem `Authorization: Bearer <accessToken>`. O servidor valida assinatura, expiração e perfil em cada rota. Erros seguem `{ "erro": { "codigo": "...", "mensagem": "..." } }`. Respostas autenticadas usam `Cache-Control: no-store`. O CORS aceita `GET`, `POST` e `PATCH`.

## Rotas

| Método | Rota | Perfil | Resultado |
| --- | --- | --- | --- |
| GET | `/health` | Público | Estado da API, versão e modo de extração |
| POST | `/auth/login` | Público | Emite JWT e usuário público |
| GET | `/auth/me` | Autenticado | Usuário da sessão |
| GET | `/pacientes?busca=` | Profissional | Fichário: pacientes com contagem de sumários |
| POST | `/pacientes` | Profissional | Cadastra paciente e cria a conta de acesso |
| GET | `/pacientes/{id}` | Profissional | Ficha: dados, sumários, altas do profissional e plano vigente |
| PATCH | `/pacientes/{id}` | Profissional | Atualiza dados cadastrais |
| POST | `/pacientes/{id}/sumarios` | Profissional | Adiciona sumário de alta à ficha |
| GET | `/pacientes/{id}/sumarios/{sumarioId}` | Profissional | Texto completo de um sumário da ficha |
| GET | `/alta/sumarios` | Profissional | Sumários fictícios selecionáveis |
| GET | `/alta/termo-aceite` | Profissional | Termo de aceite vigente (versão e declarações) |
| GET | `/alta/extracoes` | Profissional | Extrações do próprio profissional, sem texto clínico |
| POST | `/alta/transcricoes` | Profissional | A IA transcreve um PDF para conferência humana (não salva) |
| POST | `/alta/extracoes` | Profissional | Extrai sugestões a partir da base, da ficha, de texto ou de arquivo já lido |
| GET | `/alta/extracoes/{id}` | Profissional | Extração do próprio profissional |
| POST | `/alta/extracoes/{id}/aprovar` | Profissional | Aprova sugestões selecionadas e verificadas, com aceite |
| GET | `/alta/extracoes/{id}/plano` | Profissional | Plano e comprovante de aceite de uma extração aprovada |
| GET | `/alta/planos/me` | Paciente | Plano aprovado do paciente autenticado |

Login: `POST /auth/login` recebe `{ "email": "...", "senha": "..." }` e devolve `{ "accessToken", "tokenType", "expiresIn", "user": { "id", "nome", "email", "role" } }`. As contas fictícias de desenvolvimento estão no README do back-end.

## Fichário de pacientes

### Cadastrar paciente

`POST /pacientes` cria a ficha e a conta de acesso do paciente (perfil `paciente`).

```json
{
  "nome": "Paciente Fictício da Silva",
  "email": "paciente.ficticio@exemplo.dev",
  "data_nascimento": "1970-06-20",
  "numero_prontuario": "PAC-0100",
  "telefone": "(11) 0000-0000",
  "responsavel": "Familiar fictício",
  "observacoes": "Anotação administrativa"
}
```

`telefone`, `responsavel` e `observacoes` são opcionais. O prontuário é gravado em maiúsculas. A resposta (`201`) é `{ "paciente": { ... }, "senha_provisoria": "Hia-..." }`. A senha provisória aparece **só nesta resposta**; o servidor guarda apenas o hash. E-mail ou prontuário repetidos recebem `409 CONFLITO`. Erros de validação recebem `400 DADOS_INVALIDOS` com `detalhes: [{ campo, mensagem }]`.

O fichário não coleta CPF, endereço nem dados financeiros (minimização, LGPD art. 6º, III). `observacoes` é administrativo: a IA não lê este campo.

### Consultar a ficha

`GET /pacientes/{id}` devolve:

```json
{
  "paciente": { "id": "...", "nome": "...", "numero_prontuario": "PAC-0100", "data_nascimento": "1970-06-20", "...": "..." },
  "sumarios": [ { "id": "...", "titulo": "...", "origem": "pdf_ia", "nome_arquivo": "alta.pdf", "transcricao_ia": true, "caracteres": 317, "criado_em": "...", "criado_por": "u-001" } ],
  "extracoes": [ { "id": "...", "titulo": "...", "status": "pendente", "...": "..." } ],
  "plano_vigente": null
}
```

A lista de sumários não traz o texto clínico; use `GET /pacientes/{id}/sumarios/{sumarioId}`. `extracoes` traz só as do profissional autenticado.

### Adicionar sumário à ficha

`POST /pacientes/{id}/sumarios`, discriminado por `origem`:

| `origem` | Campos | Uso |
| --- | --- | --- |
| `texto` | `titulo`, `texto` | Texto digitado ou colado |
| `arquivo` | `titulo`, `nome_arquivo`, `texto` | TXT ou PDF com texto selecionável, lido no navegador |
| `pdf_ia` | `titulo`, `nome_arquivo`, `texto`, `transcricao_conferida: true` | Texto transcrito pela IA e conferido pelo profissional |

Sem `transcricao_conferida: true`, um sumário `pdf_ia` é recusado (`400`).

## Transcrição de PDF pela IA

`POST /alta/transcricoes` recebe `{ "nome_arquivo": "alta.pdf", "pdf_base64": "<base64>" }` (até 10 MB de arquivo) e devolve `{ "texto": "...", "modelo": "gemini-3.5-flash", "transcricao_ia": true }`.

- O servidor confere a assinatura `%PDF-`; arquivo que não é PDF recebe `400`. Acima do limite, `413 ARQUIVO_GRANDE`.
- O texto **não é salvo**. Ele volta para o profissional conferir com o PDF original e corrigir, se preciso.
- As citações das sugestões são verificadas contra o texto conferido, nunca contra o PDF.
- Com `GEMINI_PROVEDOR=demonstracao`, a rota responde `503 SERVICO_INDISPONIVEL`.
- A rota divide o limite de 20 chamadas a cada 10 minutos com `POST /alta/extracoes`.

## Criar extração

O corpo é discriminado por `origem`. Todos os casos exigem `paciente_id`.

```json
{ "origem": "base", "paciente_id": "u-002", "sumario_id": "sumario-cardiaco-01" }
```

```json
{ "origem": "texto", "paciente_id": "u-002", "texto": "Texto fictício do sumário..." }
```

```json
{ "origem": "arquivo", "paciente_id": "u-002", "nome_arquivo": "alta.pdf", "texto": "Texto do arquivo...", "transcricao_ia": true }
```

```json
{ "origem": "ficha", "paciente_id": "u-002", "sumario_id": "<id do sumário na ficha>" }
```

`transcricao_ia` é opcional e indica texto transcrito pela IA e conferido pelo profissional. Na origem `ficha`, o sumário precisa pertencer ao paciente informado (senão, `404`); a extração herda `nome_arquivo` e `transcricao_ia` do sumário.

A extração devolve também `ia`: `{ "modelo": "gemini-3.5-flash", "tokens_entrada": 334, "tokens_saida": 2022, "duracao_ms": 8236, "reparos": 0 }`. Em `reparos`, o número de pedidos de correção feitos à IA quando a resposta não seguiu o esquema (salvaguarda S1, até 2). No modo demonstração, `modelo` é `demonstracao`.

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

`POST /alta/extracoes/{id}/aprovar` recebe os IDs aceitos, as reprovações com motivo, as edições opcionais e o aceite:

```json
{
  "itens_aprovados": ["<id-sugestao>"],
  "itens_reprovados": [
    { "id": "<id-sugestao>", "motivo": "Orientação será dada por outro meio" }
  ],
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
| Sugestão com origem verificada sem decisão, ou reprovada sem motivo (mínimo de 3 caracteres) | `400 DADOS_INVALIDOS` |
| Mesmo item aprovado e reprovado | `400 DADOS_INVALIDOS` |
| Versão do termo diferente da vigente ou declaração faltando | `422 ACEITE_INCOMPLETO` |
| Edição de item não selecionado ou item sem origem verificada | `403 SEM_PERMISSAO` |
| Item que não pertence à extração, ou extração de outro profissional | `404 NAO_ENCONTRADO` |
| Extração já aprovada | `409 CONFLITO` |

**Salvaguarda G2:** toda sugestão recebe decisão explícita. Sugestões sem origem verificada não precisam de motivo: o servidor as reprova com o motivo automático “Origem não confirmada no texto original”. A citação original continua ligada ao item editado.

A resposta é o plano aprovado, com o registro do aceite e as reprovações (estas só para o profissional):

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
    "hash_conteudo": "<sha-256 de extração, versão do termo, itens aprovados e reprovações>"
  },
  "reprovacoes": [
    { "id": "...", "categoria": "medicamento", "titulo": "...", "trecho_origem": "...", "motivo": "...", "automatica": false }
  ]
}
```

O paciente consulta somente o próprio plano em `GET /alta/planos/me` (`404` enquanto não houver plano), **sem** o campo `reprovacoes`. O profissional reabre o comprovante, com as reprovações, em `GET /alta/extracoes/{id}/plano`. Depois da aprovação, `GET /alta/extracoes/{id}` traz `decisoes`: `{ id, decisao, motivo?, automatica, editada }` para cada sugestão.

## Auditoria

A API registra eventos de login, fichário, extração, transcrição, chamadas de IA (`ia.chamada`: modelo, tokens, tempo e reparos) e aprovação, com IDs, contagens, versão do termo e hash, sem texto clínico, motivo, nome, senha ou token. Os eventos vão ao log estruturado e, se `AUDITORIA_ARQUIVO` estiver definido, a um arquivo JSONL.

Uma coleção Postman com este fluxo está em `Backend/back-end/docs/postman/HealthIA.postman_collection.json`.

## Limites atuais

- Catálogo, usuários, extrações e planos ficam em memória e são apagados quando o servidor reinicia. Por isso o deploy usa uma única instância.
- O modo demonstração do front reproduz estas regras no navegador e não chama a API.
- A extração real usa Gemini pela Gemini API (`GEMINI_PROVEDOR=api-key`) ou pelo Vertex AI (`GEMINI_PROVEDOR=vertex`). O back-end tenta `GEMINI_MODEL` e, se o modelo estiver indisponível, sobrecarregado ou descontinuado, os modelos de `GEMINI_MODELOS_RESERVA`, com tempo limite por chamada.
- Neste piloto, todo profissional autenticado vê todos os pacientes da unidade. Vínculo profissional–paciente fica para uma versão com banco persistente.
- Prescrição Assistida e Paciente Assistido ainda não têm rotas.
- Não use dados reais de pacientes. A IA organiza o conteúdo recebido; não diagnostica nem recomenda tratamento.
