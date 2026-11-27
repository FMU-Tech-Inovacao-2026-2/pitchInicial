# Sumários fictícios

Esta pasta contém arquivos de texto inventados para demonstração e desenvolvimento do Alta Assistida. Não use, copie ou substitua seu conteúdo por dados reais ou identificáveis de pacientes.

## Arquivos disponíveis

| Arquivo | Conteúdo | ID correspondente no catálogo da API |
| --- | --- | --- |
| [`01-cardiaco.txt`](01-cardiaco.txt) | Caso de alta cardiológica com medicações, cuidado, retorno e sinal de alerta. | `sumario-cardiaco-01` |
| [`02-pos-operatorio.txt`](02-pos-operatorio.txt) | Caso após procedimento com analgesia, cuidado local e retorno. | `sumario-pos-operatorio-02` |

Os mesmos casos são cadastrados no repositório `Backend/back-end`, em `src/modules/alta/alta.repository.ts`, para a opção “Escolher da base”. Essa cópia no pitch também pode ser importada como arquivo no modo integrado. Como os repositórios são independentes, mantenha os textos sincronizados ao editar um caso.

## Uso e limites

- Importe os `.txt` pelo fluxo de arquivo do front-end integrado ou abra-os para copiar o texto.
- O catálogo local da API pode ser selecionado sem importar um arquivo.
- Não há resultados esperados em JSON: as respostas da IA podem variar e precisam ser revisadas.
- O conteúdo é apenas material de teste acadêmico, não orientação clínica. A IA não diagnostica nem recomenda tratamento; somente um profissional pode aprovar sugestões para o plano.
