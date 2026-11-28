# Dados, segurança e LGPD

Este documento registra como o HealthIA trata dados na versão atual. É um projeto acadêmico com **dados fictícios**. Antes de qualquer uso com pacientes reais, o tratamento precisa de validação do jurídico responsável e de um encarregado (DPO) definido.

Referência legal: Lei nº 13.709/2018 (LGPD). As citações de artigos abaixo indicam a base usada no desenho; a aplicação a um caso real deve ser confirmada pelo jurídico.

## Dados tratados

| Dado | Onde | Finalidade | Observação |
| --- | --- | --- | --- |
| Nome, e-mail e perfil da conta | `users` | Autenticação e autorização | Senha guardada só como hash scrypt |
| Data de nascimento e número de prontuário | Ficha (`pacientes`) | Identificar o paciente na unidade | Sem CPF, endereço ou dados financeiros (minimização, art. 6º, III) |
| Telefone, responsável, observações | Ficha (opcionais) | Contato e notas administrativas | A IA não lê estes campos |
| Texto do sumário de alta | Ficha e extrações | Organizar as orientações de alta | Dado de saúde: dado pessoal sensível (art. 5º, II) |
| Sugestões, decisões e plano | Extrações e planos | Entregar ao paciente o plano aprovado | Só após aceite do profissional |
| Motivos de reprovação | Extração (decisões) e comprovante | Responsabilização da decisão clínica | Visíveis só ao profissional; nunca ao paciente nem à auditoria |
| Consumo da IA | Extração (`ia`) e auditoria | Medir custo e desempenho | Modelo, tokens, tempo e reparos; sem conteúdo |
| Registro do aceite | Plano | Responsabilização (art. 6º, X) | Profissional, data e hora, versão do termo, declarações e hash SHA-256 |
| Trilha de auditoria | Log estruturado e JSONL opcional | Rastrear quem fez o quê e quando | Sem texto clínico, motivo, nome, nascimento, telefone, senha ou token |

## Base legal declarada no termo

O termo de aceite do profissional cita a tutela da saúde (art. 11, II, “f”). A redação está em `Backend/back-end/src/modules/alta/alta.termo.ts` e é versionada: ao mudar o texto, a versão muda e aprovações com versão antiga são recusadas.

## Aceite profissional

Para aprovar um plano, o profissional confirma, uma a uma, as declarações do termo vigente:

1. Revisou cada item aprovado e conferiu o trecho de origem.
2. Está ciente de que a IA pode omitir, interpretar mal ou errar, e de que as sugestões não substituem o julgamento clínico.
3. Assume a decisão profissional sobre o conteúdo aprovado.
4. Usa os dados somente para a tutela da saúde do paciente.

O servidor recusa aceite incompleto (`422`). Além disso, cada sugestão precisa de uma decisão explícita: reprovar exige motivo (salvaguarda G2), e o motivo entra no hash de integridade. O comprovante fica disponível ao profissional, e o paciente vê quem aprovou e quando, sem as reprovações.

## Uso de IA

- No modo integrado, o texto do sumário e, quando o profissional pede, o PDF são enviados ao Gemini (Google), pela Gemini API ou pelo Vertex AI, conforme a configuração do servidor. Isso é um compartilhamento com operador; um uso real exige contrato e avaliação de transferência internacional (art. 33).
- No modo demonstração, nada sai do navegador.
- A transcrição de PDF não é salva até o profissional conferi-la e confirmar.
- As instruções da IA proíbem diagnosticar, recomendar conduta e completar lacunas, e tratam o documento como dado, nunca como instrução.

## Controles técnicos

| Controle | Implementação |
| --- | --- |
| Autorização por perfil | `requireRole` em cada rota; paciente acessa só o próprio plano |
| Senhas | scrypt com sal; tempo constante; limite de tentativas de login |
| Senha provisória | Gerada no servidor, exibida uma vez, nunca registrada em log |
| Tokens | JWT HS256 com expiração; algoritmo fixo (bloqueia `alg=none`) |
| Logs | Pino com redação de autorização, senhas, tokens, textos clínicos e PDFs; o `.env` é lido antes de criar o logger |
| Auditoria | Filtro de campos proibidos (texto, motivo, nome, contato, senha, token) antes de registrar; JSONL gravado em ordem, sem bloquear a requisição |
| Saída da IA | Esquema Zod com até 2 pedidos de correção (S1); verificação de origem no servidor (S2) |
| Cache | `Cache-Control: no-store` em todas as respostas da API |
| Limites | JSON de 1 MB; PDF de até 10 MB; 20 chamadas de IA a cada 10 minutos por profissional |
| Front-end | CSP restrita, HSTS, `X-Frame-Options: DENY`, `nosniff`, `Referrer-Policy: no-referrer` |
| Segredos | Fora das imagens e do Git; JWT no Secret Manager em produção |

## Retenção

Na versão atual, todos os dados ficam em memória e são apagados quando o servidor reinicia. Uma política de retenção e descarte é pré-requisito para persistência real.

## Direitos do titular

A LGPD garante confirmação, acesso, correção, anonimização, portabilidade, eliminação e informação sobre compartilhamento (art. 18). O protótipo não tem canal de atendimento ao titular; ele deve ser definido junto com o controlador e o encarregado.

## Pendências para uso real

- Validação jurídica do termo, do aviso de privacidade e das bases legais.
- Relatório de impacto à proteção de dados (RIPD), dado o volume de dados sensíveis.
- Contrato com o operador de IA e avaliação de transferência internacional.
- Banco persistente com criptografia, backup, retenção e descarte.
- Vínculo profissional–paciente e trilha de auditoria imutável.
