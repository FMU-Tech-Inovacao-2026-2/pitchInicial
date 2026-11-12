# Dados fictícios

Massa de teste do projeto. **Somente dados inventados pela equipe** — nenhum nome, documento ou caso real de paciente.

## Sumários de alta (Alta Assistida)

Cada sumário fica num arquivo `.txt`, acompanhado de um `.json` com a extração esperada (os medicamentos, alertas e retornos que o sistema precisa encontrar).

| Arquivo | Caso coberto |
| --- | --- |
| `01-padrao.txt` | Sumário completo, 3 a 5 medicamentos |
| `02-muitos-medicamentos.txt` | Polifarmácia, mudanças de dose e remédio suspenso |
| `03-curto.txt` | Sumário muito curto, com informações faltando |
| `04-confuso.txt` | Abreviações e texto desorganizado |
| `05-sem-medicamentos.txt` | Nenhum medicamento prescrito na alta |

## Prescrições e fichas (Prescrição Assistida e Paciente Assistido) — a criar

Pacientes inventados, cada um com alergias, internações e prescrições anteriores, para testar a conferência de alergia e o chat.

| Caso | O que testa |
| --- | --- |
| Prescrição simples | Extração de 2 a 3 itens com origem verificada |
| Prescrição com abreviações | Doses e frequências escritas de forma abreviada |
| Medicamento que coincide com alergia | Aviso de alergia na revisão |
| Prescrição incompleta | Campos ausentes aparecem como "não informado" |
| Paciente com várias internações | Perguntas sobre histórico geral e recente |
| Pergunta sem resposta na ficha | Resposta "não consta no histórico" |

Responsável: Pessoa 3.
