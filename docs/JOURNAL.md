# Diário de decisões

Registro das decisões técnicas e de produto, da mais recente para a mais antiga. Cada entrada diz o que foi decidido, por quê e o que foi descartado.

## 27/11/2026 — Arquitetura hexagonal no back-end

**Decisão:** reorganizar o back-end em `domain`, `application`, `adapters` e `entrypoints`, seguindo o projeto de referência.
**Por quê:** isolar as regras clínicas e de segurança (verificação de origem, aceite e decisões) de Express, Gemini e persistência; permitir trocar a IA ou o banco sem tocar nas regras.
**Descartado:** manter a organização por módulo (`modules/alta`, …), que misturava regra de negócio, HTTP e acesso a dados no mesmo serviço.

## 27/11/2026 — Motivo obrigatório na reprovação (G2)

**Decisão:** toda sugestão com origem verificada e não aprovada exige motivo. As reprovações ficam no comprovante e entram no hash de integridade, mas não vão ao paciente.
**Por quê:** responsabilização (LGPD, art. 6º, X) e rastreabilidade da decisão clínica, no modelo da salvaguarda G2 do projeto de referência.
**Descartado:** registrar só os itens aprovados, o que apagava o que a IA sugeriu e o profissional recusou.

## 27/11/2026 — Reparo da resposta da IA (S1)

**Decisão:** se a resposta do Gemini não seguir o esquema, devolver o erro ao mesmo modelo e pedir correção, até 2 vezes.
**Por quê:** falhas de formato são recuperáveis e não devem virar erro para o profissional.
**Descartado:** falhar direto, como antes.

## 27/11/2026 — Auditoria em JSONL e consumo da IA

**Decisão:** além do log estruturado, gravar a auditoria em JSONL quando `AUDITORIA_ARQUIVO` estiver definido; registrar modelo, tokens, tempo e reparos de cada chamada (`ia.chamada` e campo `ia` da extração).
**Por quê:** permitir medir custo e desempenho e auditar sem expor conteúdo clínico.

## 27/11/2026 — Logger carrega o `.env` antes de iniciar

**Decisão:** o logger importa `dotenv/config` antes de ler `NODE_ENV` e `LOG_LEVEL`.
**Por quê:** o log local de um usuário mostrou JSON bruto em desenvolvimento. O logger era criado antes de o `.env` ser lido, então `NODE_ENV=development` e `LOG_LEVEL` eram ignorados.

## 27/11/2026 — Modelo padrão `gemini-3.5-flash` com reservas

**Decisão:** trocar o padrão de `gemini-2.5-flash` para `gemini-3.5-flash`, com `GEMINI_MODELOS_RESERVA` e tempo limite por chamada.
**Por quê:** o 2.5 respondeu 404 para contas novas; no teste, `gemini-3.8-flash` deu 503 e `gemini-3.7-flash` passou de 3 minutos sem resposta.

## 27/11/2026 — Fichário e PDF lido pela IA com conferência humana

**Decisão:** criar o fichário de pacientes e permitir que a IA transcreva PDFs digitalizados. A transcrição volta para o profissional e só é usada depois da confirmação.
**Por quê:** PDFs digitalizados não têm texto selecionável. A verificação de origem (S2) só tem valor se o texto de referência foi conferido por uma pessoa.
**Descartado:** salvar a transcrição direto, o que tornaria a verificação de origem circular (IA conferindo a própria transcrição).

## 27/11/2026 — Aceite profissional versionado (LGPD)

**Decisão:** o plano só existe com o aceite de todas as declarações da versão vigente do termo, que o servidor fornece.
**Por quê:** deixar explícito que o profissional conferiu as origens, sabe que a IA pode errar e assume a decisão.

## 27/11/2026 — Deploy no Cloud Run com Workload Identity Federation

**Decisão:** imagens sem privilégios, segredo do JWT no Secret Manager e autenticação do GitHub por OIDC, sem chave JSON.
**Por quê:** reduzir segredos de longa duração. A API roda com uma instância porque os dados estão em memória.

---

[← Índice da documentação](README.md)
