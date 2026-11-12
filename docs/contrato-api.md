# Contrato da API (rascunho)

Rascunho para a fase de fundação e protótipo. Todas as rotas, exceto o login, exigem o token do Supabase Auth no cabeçalho `Authorization: Bearer <token>`. O back-end confere o perfil (`profissional` ou `paciente`) em cada rota.

As rotas do Alta Assistida ganharam o prefixo `/alta` porque agora ele é um módulo da HealthIA.

## Endpoints do MVP

| Módulo | Método | Rota | Perfil | Função |
| --- | --- | --- | --- | --- |
| Login | POST | `/auth/login` | Todos | Entrada com e-mail e senha (via Supabase Auth); devolve o token e o perfil |
| Home | GET | `/modulos` | Todos | Lista os módulos liberados para o perfil |
| Alta Assistida | POST | `/alta/extracoes` | Profissional | Recebe o texto do sumário e devolve a extração com status `pendente` |
| Alta Assistida | POST | `/alta/extracoes/{id}/aprovar` | Profissional | Aprova a extração |
| Alta Assistida | GET | `/alta/planos/{id}` | Paciente | Plano aprovado para o dashboard do paciente |
| Prescrição Assistida | POST | `/prescricoes` | Profissional | Recebe o texto da prescrição e o `paciente_id`; devolve a extração `pendente` |
| Prescrição Assistida | POST | `/prescricoes/{id}/aprovar` | Profissional | Aprova e grava a prescrição na ficha do paciente |
| Paciente Assistido | GET | `/pacientes/{id}/ficha` | Profissional | Prescrições, internações e alergias do paciente |
| Paciente Assistido | POST | `/pacientes/{id}/chat` | Profissional | Pergunta em linguagem natural; devolve a resposta com as fontes |

## Alta Assistida — formato da extração

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

## Prescrição Assistida — formato da extração

```json
{
  "status": "pendente",
  "paciente_id": "[id]",
  "itens": [
    {
      "medicamento": "[nome]",
      "dose": "[ex.: 100 mg]",
      "via": "[ex.: oral]",
      "frequencia": "[ex.: 1 vez ao dia]",
      "duracao": "[ex.: uso contínuo]",
      "orientacao": "[ex.: após o almoço]",
      "trecho_origem": "[trecho copiado da prescrição]",
      "origem_verificada": true,
      "alerta_alergia": { "alergia_registrada": "[alergia]", "registro_origem": "[id do registro]" }
    }
  ],
  "nao_informado": ["[campo que a prescrição não traz]"]
}
```

`alerta_alergia` só aparece quando o medicamento coincide com uma alergia da ficha. A comparação é feita pelo back-end, nunca pela IA, e o aviso não bloqueia a aprovação: a decisão é do médico.

## Paciente Assistido — chat

Requisição:

```json
{ "pergunta": "[texto do médico]", "periodo": "geral" }
```

`periodo` aceita `geral` ou `recente`.

Resposta:

```json
{
  "resposta": "[texto]",
  "fontes": [
    { "tipo": "prescricao", "registro_id": "[id]", "trecho": "[trecho do registro]", "origem_verificada": true }
  ],
  "consta_no_historico": true
}
```

`tipo` aceita `prescricao`, `internacao` ou `alergia`. Quando a informação não está na ficha, `consta_no_historico` é `false`, `fontes` vem vazio e a resposta diz que não consta no histórico.

## Regra de verificação de origem

`origem_verificada` é calculado pelo back-end, nunca pela IA.

- **Alta e Prescrição:** o campo é `true` somente se `trecho_origem` aparece no texto enviado (comparação após normalizar espaços e maiúsculas/minúsculas). Itens com `false` aparecem em destaque para o profissional e não vão para o plano nem para a ficha sem correção.
- **Chat:** o campo é `true` somente se o `registro_id` citado existe na ficha do paciente da conversa e o `trecho` aparece nesse registro. Fontes com `false` são descartadas da resposta e sinalizadas.
