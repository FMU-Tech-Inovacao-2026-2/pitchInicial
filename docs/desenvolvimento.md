# Desenvolvimento

## Pré-requisitos

- Node.js 20.19 ou superior (o CI usa Node.js 22).
- Docker, opcional, para testar as imagens de produção.
- Chave da Gemini API, opcional. Sem chave, use `GEMINI_PROVEDOR=demonstracao`.

## Rodar sem Docker

**API** (`Backend/back-end`):

```powershell
npm install
Copy-Item .env.example .env
npm run dev
```

Edite o `.env`: defina `JWT_SECRET` (32 caracteres ou mais) e o modo de IA. Confira em `http://localhost:3000/health`.

**Front-end** (`FrontEnd/front-end`):

```powershell
npm install
Copy-Item .env.example .env.local
npm run dev
```

Para usar a API, defina no `.env.local`: `VITE_MODO_BACKEND=true` e `VITE_API_URL=http://localhost:3000`. Abra `http://localhost:5173`.

## Rodar com Docker

```powershell
docker compose up api-dev     # API com recarga, porta 3000
docker compose up api-prod    # imagem de produção da API, porta 3001
```

```powershell
docker build -t healthia-web .
docker run -p 8080:8080 -e HEALTHIA_API_URL=http://localhost:3000 healthia-web
```

## Testar a API no Postman

Importe `Backend/back-end/docs/postman/HealthIA.postman_collection.json`. Rode, na ordem: **Login profissional → Termo de aceite vigente → Criar extração (texto) → Aprovar extração → Login paciente → Meu plano**. Os scripts guardam token, IDs e versão do termo nas variáveis da coleção.

## Comandos

| Repositório | Comando | O que faz |
| --- | --- | --- |
| back-end | `npm run typecheck` | Checagem de tipos |
| back-end | `npm test` | Testes (node:test) |
| back-end | `npm run build` | Compila para `dist/` |
| front-end | `npm run lint` | oxlint |
| front-end | `npm test` | Testes (Vitest) |
| front-end | `npm run build` | Checagem de tipos e bundle do Vite |

## CI

Cada repositório tem `.github/workflows/ci.yml`, executado em push e pull request:

- **back-end:** tipos, testes, build, `npm audit` das dependências de produção e build da imagem com checagem de `/health`.
- **front-end:** lint, testes, build e build da imagem, conferindo `config.js` e a CSP.
- **pitchInicial:** nenhum `.env` versionado, JSON válido e links locais dos Markdown.

## Onde mexer

| Quero… | Camada (back-end) |
| --- | --- |
| Mudar uma regra clínica ou de segurança | `src/domain/` |
| Criar um caso de uso | `src/application/` (e uma porta, se precisar de algo externo) |
| Trocar a IA, o banco ou a criptografia | `src/adapters/` + ligação em `src/server.ts` |
| Criar ou mudar uma rota | `src/entrypoints/http/` (rota + esquema Zod) |

Ao mudar comportamento, atualize no mesmo trabalho o `CHANGELOG.md` e os documentos afetados (regra do `AGENTS.md`).

## Diagnóstico rápido

| Sintoma | Causa provável |
| --- | --- |
| `/health` mostra `demonstracao` com chave configurada | `GEMINI_PROVEDOR=demonstracao` no `.env` |
| Log com 404 do Gemini | Modelo descontinuado para a conta; ajuste `GEMINI_MODEL` |
| Log com 503 ou 429 do Gemini | Sobrecarga ou cota; a API tenta os modelos de reserva |
| Acentos estranhos no console do Windows (ex.: `extra├º├úo`) | Página de código do terminal, não do app; rode `chcp 65001` antes de `npm run dev` |
| Front-end mostra erro de conexão | API fora do ar ou `CORS_ORIGIN` diferente da URL do front-end |

---

[← Índice da documentação](README.md)
