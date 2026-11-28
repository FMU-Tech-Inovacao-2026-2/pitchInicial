# Arquitetura

O back-end segue **arquitetura hexagonal** (portas e adaptadores), no mesmo modelo do projeto de referência `Luis01Felipe/hackathon`, adaptado a Node.js, Express, TypeScript e Zod. O domínio não importa Express, Zod, Gemini, pino nem bibliotecas de persistência.

```
src/
  domain/        entidades e regras puras: verificação de origem (S2), aceite, decisões (G2),
                 hash de integridade, termo de aceite, erros de negócio
  application/   casos de uso (auth, pacientes, alta) + portas (interfaces TypeScript)
  adapters/      Gemini (S1, consumo) · demonstração sem IA · catálogo fictício ·
                 persistência em memória · scrypt · JWT · auditoria JSONL · logger
  entrypoints/   HTTP (Express): app, rotas, esquemas Zod, autenticação, limites, erros
  config/        variáveis de ambiente validadas com Zod
  server.ts      raiz de composição: liga adaptadores às portas
```

A regra de dependência aponta sempre para dentro: `entrypoints → application → domain` e `adapters → application/domain`. O domínio não depende de nenhuma outra camada.

## Salvaguardas contra erro da IA

Todas são aplicadas por código, no servidor.

| # | Salvaguarda | Onde |
| --- | --- | --- |
| S1 | A resposta da IA é validada no esquema Zod. Se falhar, o erro volta ao mesmo modelo e há até 2 pedidos de correção | `adapters/ia/gemini.ts` |
| S2 | O trecho citado precisa existir no texto original (normaliza espaços, acentos, aspas, caixa e hifenização de PDF) | `domain/alta.ts` (`verificarOrigem`) |
| S3 | Sugestão sem origem verificada não pode ser aprovada; é reprovada automaticamente | `domain/alta.ts` (`decidirSugestoes`) |
| G1 | O plano só existe depois do aceite humano com a versão vigente do termo e todas as declarações | `domain/alta.ts` (`validarAceite`) |
| G2 | Toda sugestão recebe decisão explícita: aprovar exige origem; reprovar exige motivo. A reprovada continua registrada no comprovante (não vai ao paciente) | `domain/alta.ts` (`decidirSugestoes`) |
| G3 | O documento vai à IA entre delimitadores (`<sumario>`), como dado; as instruções mandam ignorar ordens contidas nele | `adapters/ia/gemini.ts` |
| G4 | Transcrição de PDF pela IA não é salva até o profissional conferir e confirmar | `application/alta.ts` + esquema `pdf_ia` |
| G5 | Integridade: hash SHA-256 da extração, versão do termo, itens aprovados e reprovações com motivo | `domain/alta.ts` (`hashDoConteudo`) |
| R1 | Resiliência: troca para modelos de reserva em 404, 429 e 5xx, com tempo limite por chamada | `adapters/ia/gemini.ts` |

## Portas

| Porta | Adaptador atual | Futuro |
| --- | --- | --- |
| `UserRepository`, `AltaRepository`, `FichaRepository` | Memória (`adapters/persistencia/memoria.ts`) | Banco gerenciado |
| `ExtratorIA` | Gemini API ou Vertex AI; demonstração sem IA | Outros provedores |
| `ServicoSenha` | scrypt | — |
| `ServicoToken` | JWT HS256 próprio | Provedor de identidade |
| `Auditoria` | Log estruturado + arquivo JSONL opcional | Armazenamento imutável |

Trocar um adaptador não exige mudar domínio nem casos de uso: basta implementar a porta e ligá-lo em `server.ts`.

## Fluxo do Alta Assistida

```mermaid
sequenceDiagram
    actor P as Profissional
    participant F as Front-end
    participant H as entrypoints/http
    participant C as application (casos de uso)
    participant D as domain
    participant G as adapters/ia (Gemini)

    P->>F: Cadastra paciente e adiciona sumário (texto, TXT ou PDF)
    alt PDF digitalizado
      F->>H: POST /alta/transcricoes
      H->>C: transcreverPdf
      C->>G: transcrição literal
      G-->>F: texto (não salvo) para conferência (G4)
    end
    F->>H: POST /alta/extracoes
    H->>C: criarExtracao
    C->>G: extrair (G3)
    G-->>C: JSON validado (S1) + consumo
    C->>D: verificarOrigem (S2)
    C-->>F: extração pendente
    P->>F: decide cada item; motivo nas reprovações; confirma o termo
    F->>H: POST /alta/extracoes/{id}/aprovar
    H->>C: aprovarExtracao
    C->>D: validarAceite (G1) · decidirSugestoes (S3, G2) · hashDoConteudo (G5)
    C-->>F: plano + comprovante com reprovações
```

O paciente consulta só o próprio plano em `GET /alta/planos/me`, sem as reprovações.

## Front-end

- `src/config.ts` decide o modo (`backend` ou `demonstracao`) a partir de `/config.js` (gerado no contêiner) ou das variáveis `VITE_`.
- `src/api` tem um cliente por domínio (`alta`, `pacientes`) e as versões de demonstração, que reproduzem as mesmas regras (S2, S3, G1, G2).
- `useTratarErro` centraliza a sessão expirada (401 → login, voltando à mesma página).
- As decisões e os motivos da revisão ficam salvos por extração na aba (`sessionStorage`); a revisão final nunca presume itens aprovados.

## Repositórios

| Repositório | Papel |
| --- | --- |
| `back-end` | API e regras de negócio (hexagonal) |
| `front-end` | Interface web |
| `pitchInicial` | Ideação, contrato, protótipo e documentação compartilhada |

## Infraestrutura

- Imagens Docker sem privilégios: `node:22-alpine` (API) e `nginx-unprivileged` (front-end, com CSP).
- CI em cada repositório; deploy no Cloud Run com Workload Identity Federation. Veja [`deploy.md`](deploy.md).

## Limites atuais e trabalho futuro

| Tema | Hoje | Futuro |
| --- | --- | --- |
| Persistência | Memória; some ao reiniciar; uma instância | Banco gerenciado com criptografia e backup |
| Identidade | JWT próprio, contas fictícias | Provedor de identidade, MFA, recuperação de senha |
| Vínculo profissional–paciente | Todo profissional vê todos os pacientes | Equipes e vínculos com autorização por paciente |
| Lacunas (`nao_informado`) | Dependem da IA e não são exaustivas | Checklist determinístico por categoria |
| Módulos | Alta Assistida e fichário | Prescrição Assistida e Paciente Assistido |

---

[← Índice da documentação](README.md)
