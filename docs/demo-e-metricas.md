# Dados da demonstração e métricas

## Dados fictícios

Todo o conteúdo é inventado. Nenhum dado de paciente real foi usado.

| Item | Onde |
| --- | --- |
| Sumários do catálogo (`sumario-cardiaco-01`, `sumario-pos-operatorio-02`) | `Backend/back-end/src/adapters/ia/catalogo.ts` e `data/sumarios-ficticios/` |
| Respostas preparadas da demonstração, com um erro proposital da IA | `catalogo.ts` (back-end) e `src/api/demonstracao.ts` (front-end) |
| Contas de desenvolvimento | README do back-end |
| Sumário usado na coleção Postman | `Backend/back-end/docs/postman/HealthIA.postman_collection.json` |

O sumário cardiológico da demonstração inclui a sugestão “Enalapril 10 mg à noite”, cuja citação não existe no texto. Ela serve para mostrar a salvaguarda S2: o servidor marca `origem_verificada: false` e a sugestão não pode ser aprovada.

## Métricas medidas

**Medição de 27/11/2026**, com a Gemini API, modelo `gemini-3.5-flash`, `temperature: 0`, 3 rodadas por sumário do catálogo, a partir da máquina de desenvolvimento.

| Sumário | Sugestões por rodada | Origem verificada | `nao_informado` | Tempo (ms) | Tokens de entrada | Tokens de saída | Reparos (S1) |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Cardiológico | 5, 5, 5 | 15 de 15 | 1 por rodada | 7.126 · 6.218 · 7.744 | 348 | 1.528 · 1.442 · 1.789 | 0 |
| Pós-operatório | 3, 3, 3 | 9 de 9 | 0 | 11.185 · 7.283 · 13.022 | 291 | 1.258 · 1.369 · 1.500 | 0 |

No mesmo dia, o sumário da coleção Postman gerou 5 sugestões, todas com origem verificada, em 8.236 ms (334 tokens de entrada e 2.022 de saída).

**Leitura dos resultados**

- Nenhuma citação inventada nessas 6 rodadas. Ainda assim, a verificação S2 continua obrigatória: a demonstração mostra que o bloqueio funciona quando a IA erra.
- As contagens por categoria foram estáveis entre rodadas. O texto das sugestões varia um pouco.
- Os tokens de saída incluem o raciocínio interno do modelo (`thoughtsTokenCount`), por isso superam o tamanho do JSON.
- **Limitação observada:** o sumário pós-operatório não tem sinais de alerta, e a IA não registrou essa ausência em `nao_informado`. As lacunas apontadas pela IA não são exaustivas; o profissional continua responsável por avaliar o que falta.

**Como reproduzir:** cada extração grava o consumo em `ia` (modelo, tokens, tempo e reparos) e na auditoria (`ia.chamada`). Configure `AUDITORIA_ARQUIVO` e agregue os eventos do JSONL.

Estes números valem para o modelo e a data informados. A disponibilidade e o desempenho dos modelos do Gemini mudam com o tempo.

## Roteiro de apresentação (5 minutos)

1. Entre como profissional e abra **Pacientes → Paciente Demo → Adicionar sumário**. Mostre a importação de PDF.
2. Use **Nova alta** com o sumário cardiológico da base.
3. Na revisão, mostre a sugestão bloqueada (citação inexistente) e o botão **Mostrar no sumário**.
4. Reprove um item e mostre que o motivo é obrigatório.
5. Na revisão final, marque as declarações do termo e aprove. Mostre o comprovante com hash e reprovações.
6. Entre como paciente e mostre o plano, com os sinais de alerta em destaque.

---

[← Índice da documentação](README.md)
