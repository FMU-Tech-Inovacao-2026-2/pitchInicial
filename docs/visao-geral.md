# Visão geral

## Problema

Na alta hospitalar, o paciente recebe um sumário técnico, longo e pouco claro. Remédios, cuidados, retornos e sinais de alerta ficam misturados. Organizar isso à mão toma tempo da equipe, e um erro na transcrição pode chegar ao paciente.

## O que o HealthIA faz

O módulo **Alta Assistida** transforma o sumário de alta num plano de cuidados que o paciente entende, sem tirar a decisão do profissional:

1. O profissional cadastra o paciente no **fichário** e adiciona o sumário (texto, TXT ou PDF; PDF digitalizado pode ser transcrito pela IA, com conferência obrigatória).
2. A IA **organiza** o texto em sugestões por categoria (medicamento, cuidado, retorno e sinal de alerta), cada uma com o trecho de origem.
3. O servidor **confere** se cada trecho existe no texto. Citação inventada é bloqueada.
4. O profissional **decide** cada item: aprova, edita ou reprova com motivo.
5. O profissional confirma o **termo de aceite**, ciente de que a IA pode errar. O sistema grava um comprovante com hash de integridade.
6. O paciente vê o **plano aprovado**, com sinais de alerta em destaque, quem aprovou e quando.

## O que o HealthIA não faz

- Não diagnostica, não recomenda tratamento e não altera prescrições.
- Não completa lacunas: o que não está escrito no sumário não vira sugestão.
- Não libera nada ao paciente sem aprovação profissional.

## Perfis

| Perfil | Pode |
| --- | --- |
| Profissional de saúde | Gerenciar o fichário, gerar e revisar sugestões, aprovar planos, ver comprovantes |
| Paciente | Ver o próprio plano aprovado |

## Modos de execução

| Modo | IA | Servidor | Uso |
| --- | --- | --- | --- |
| Integrado (`api-key` ou `vertex`) | Gemini real | API Node.js | Desenvolvimento e piloto com dados fictícios |
| Integrado (`demonstracao`) | Respostas preparadas | API Node.js | Apresentação sem credencial de IA |
| Demonstração do front-end | Respostas preparadas | Nenhum (navegador) | Apresentação offline |

## Módulos

| Módulo | Estado |
| --- | --- |
| Alta Assistida | Implementado |
| Fichário de pacientes | Implementado |
| Prescrição Assistida | Visão de produto; não implementado |
| Paciente Assistido | Visão de produto; não implementado |

## Referências

- Projeto de referência de arquitetura e salvaguardas: `Luis01Felipe/hackathon` (EditalCheck), adaptado ao domínio da saúde.
- Lei nº 13.709/2018 (LGPD). Veja [Dados, segurança e LGPD](dados-e-seguranca.md).

---

[← Índice da documentação](README.md)
