# Limites e licenças

## O que o sistema não faz

- **Não é dispositivo médico.** É um projeto acadêmico. Não use com pacientes reais.
- **Não diagnostica nem recomenda tratamento.** A IA só organiza o que está escrito no sumário.
- **Não completa lacunas.** As lacunas apontadas pela IA (`nao_informado`) não são exaustivas.
- **Não guarda dados de forma persistente.** Tudo fica em memória e some ao reiniciar a API.
- **Não tem OCR próprio.** PDF digitalizado depende da transcrição pela IA, com conferência humana.
- **Não tem vínculo profissional–paciente.** Neste piloto, todo profissional vê todos os pacientes.
- **Não tem recuperação de senha, MFA nem provedor de identidade.**
- **Não tem canal de atendimento ao titular de dados** (LGPD, art. 18).
- **Prescrição Assistida e Paciente Assistido** estão só na visão de produto.

## Dependências e licenças

Versões instaladas e licenças declaradas em `node_modules/<pacote>/package.json` em 27/11/2026.

### Back-end — produção

| Pacote | Versão | Licença |
| --- | --- | --- |
| `@google/genai` | 3.6.1 | Apache-2.0 |
| `cors` | 2.8.6 | MIT |
| `dotenv` | 18.0.10 | BSD-2-Clause |
| `express` | 5.3.0 | MIT |
| `helmet` | 8.3.0 | MIT |
| `pino` | 10.4.0 | MIT |
| `zod` | 4.6.7 | MIT |

### Back-end — desenvolvimento

`@types/cors`, `@types/express`, `@types/node`, `@types/supertest`, `pino-pretty`, `supertest`, `tsx` (MIT) e `typescript` 7.0.2 (Apache-2.0).

### Front-end — produção

| Pacote | Versão | Licença |
| --- | --- | --- |
| `@fontsource-variable/inter` | 5.3.0 | OFL-1.1 |
| `@supabase/supabase-js` | 2.119.1 | MIT |
| `@tailwindcss/vite` | 4.3.5 | MIT |
| `pdfjs-dist` | 6.5.438 | Apache-2.0 |
| `react` / `react-dom` | 19.3.1 | MIT |
| `react-router` | 8.5.2 | MIT |
| `tailwindcss` | 4.3.5 | MIT |

### Front-end — desenvolvimento

`@testing-library/*`, `@types/*`, `@vitejs/plugin-react`, `jsdom`, `oxlint`, `vite`, `vitest` (MIT) e `typescript` 7.0.2 (Apache-2.0).

As licenças acima são permissivas. A OFL-1.1 da fonte Inter permite uso e distribuição com o software, sem venda isolada da fonte. Para uso comercial ou redistribuição, confirme os termos com o jurídico.

## Serviços externos

| Serviço | Uso | Observação |
| --- | --- | --- |
| Gemini API / Vertex AI (Google) | Extração e transcrição | Sujeito aos termos do Google Cloud ou do Google AI Studio. Envia o texto do sumário ou o PDF |
| Google Cloud Run, Artifact Registry e Secret Manager | Deploy | Opcional |
| GitHub Actions e GitHub Pages | CI e protótipo | Opcional |

---

[← Índice da documentação](README.md)
