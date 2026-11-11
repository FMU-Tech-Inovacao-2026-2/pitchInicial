# Contrato da API (rascunho)

Rascunho para a fase de fundação. Pessoas 2 e 3 revisam os campos com base nos sumários fictícios.

## Endpoints do MVP

| Método | Rota | Função |
| --- | --- | --- |
| POST | `/extracoes` | Recebe o texto do sumário e devolve a extração com status `pendente` |
| POST | `/extracoes/{id}/aprovar` | Profissional aprova a extração |
| GET | `/planos/{id}` | Plano aprovado para o dashboard do paciente |

## Formato da extração

```json
{
  "status": "pendente",
  "medicamentos": [
    {
      "nome": "[nome]",
      "dose": "[ex.: 500 mg]",
      "frequencia": "[ex.: de 8 em 8 horas]",
      "duracao": "[ex.: 7 dias]",
      "orientacao": "[ex.: após as refeições]",
      "trecho_origem": "[trecho copiado do sumário]",
      "origem_verificada": true
    }
  ],
  "alertas": [
    { "texto": "[alerta]", "trecho_origem": "[trecho]", "origem_verificada": true }
  ],
  "retornos": [
    { "texto": "[retorno ou exame agendado]", "trecho_origem": "[trecho]", "origem_verificada": true }
  ],
  "nao_informado": ["[campo que o sumário não traz]"]
}
```

## Regra de verificação de origem

`origem_verificada` é calculado pelo back-end, nunca pela IA: o campo é `true` somente se `trecho_origem` aparece no texto enviado (comparação após normalizar espaços e maiúsculas/minúsculas). Itens com `false` aparecem em destaque para o profissional e não vão para o plano do paciente sem correção.
