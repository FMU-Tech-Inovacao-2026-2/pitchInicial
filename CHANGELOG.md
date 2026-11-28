# Changelog — pitchInicial

Mudanças na documentação compartilhada, no contrato e no protótipo. Formato baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.1.0/). Datas em DD/MM/AAAA.

## [Não publicado]

### Adicionado
- `docs/visao-geral.md`, `docs/desenvolvimento.md`, `docs/demo-e-metricas.md` (com medições reais do Gemini), `docs/limites-e-licencas.md` e `docs/JOURNAL.md` (diário de decisões), completando o conjunto do projeto de referência.
- Contrato: `itens_reprovados`, regras G2, campo `ia`, `decisoes` e `reprovacoes`.
- `docs/arquitetura.md`: módulos, fluxos (diagramas Mermaid), garantias de projeto e limites.
- `docs/dados-e-seguranca.md`: dados tratados, base legal declarada, aceite, uso de IA, controles e pendências LGPD.
- `docs/README.md`: índice da documentação.
- Contrato: fichário de pacientes (`/pacientes`), transcrição de PDF (`/alta/transcricoes`) e extração com `origem: "ficha"`.
- `CHANGELOG.md`.

### Alterado
- `docs/arquitetura.md` reescrito para a arquitetura hexagonal, com a tabela de salvaguardas S1–S3, G1–G5 e R1.
- `docs/dados-e-seguranca.md`: motivos de reprovação, consumo da IA e auditoria JSONL.
- `docs/deploy.md`: auditoria em produção via Cloud Logging.
- `docs/deploy.md`: variáveis de modelo do Gemini e reservas.

## [1.0.0] — 27/11/2026

### Adicionado
- Contrato com termo de aceite, novas rotas, códigos de erro e plano com aceite.
- Guia de deploy no Cloud Run com Vertex AI.
- CI de documentação e publicação do protótipo no GitHub Pages.
