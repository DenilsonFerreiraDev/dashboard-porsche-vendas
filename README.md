# Dashboard de Vendas Porsche

Dashboard interativa em HTML construída a partir de uma planilha de 100 vendas da Porsche, como desafio do bootcamp da DIO. Os dados ficam embutidos no próprio `index.html`, e os filtros atualizam todos os gráficos.

**Dashboard publicada:** [PREENCHER: https://SEU-USUARIO.github.io/NOME-DO-REPOSITORIO/](https://SEU-USUARIO.github.io/NOME-DO-REPOSITORIO/)

## Resultado

![Dashboard completa](prints/dashboard.png)

Exemplo de filtro aplicado. Os indicadores e os três gráficos se atualizam juntos:

![Filtro aplicado](prints/filtro.png)

## Perguntas de negócio

Escolhi três perguntas. Cada uma sustenta uma decisão e é respondida com honestidade pelos dados disponíveis.

**1. Quais famílias de modelo geram mais receita, e quais têm o maior ticket médio?**
Orienta que linhas de produto priorizar no estoque e na comunicação. Na base, o 911 lidera em receita (cerca de US$ 4,27 mi em 23 vendas) e tem o maior ticket médio. O Macan vende quase o mesmo volume que o Cayenne, mas com ticket bem menor. Volume e receita contam histórias diferentes.

**2. Como os clientes pagam, e o método de pagamento muda com o valor da venda?**
Ajuda a decidir que condições comerciais oferecer e onde há dependência de um único canal. *Wire Transfer* concentra o maior número de vendas (26), enquanto *Crypto Payment* tem poucas vendas (6), mas o maior ticket médio.

**3. Quanto da receita já foi entregue, quanto está em andamento e quanto foi cancelado?**
Separa receita realizada de receita em risco. Dos US$ 12,83 mi vendidos, cerca de US$ 5,08 mi estão entregues (41 vendas), cerca de US$ 7,12 mi em andamento (52 vendas) e cerca de US$ 0,63 mi cancelados (7 vendas).

### Por que não escolhi outras

- **Perguntas sobre evolução no tempo:** 24 das 100 datas são `INVALID`, então o resultado ficaria enviesado.
- **"Vendas por estado" como pergunta central:** CA e TX concentram 30 das 100 vendas e a maioria dos outros estados tem só 1 ou 2, então percentuais por estado enganam. Por isso estado virou **filtro**, e não gráfico.

## Tratamento da base

A planilha original tinha colunas brutas (com formatos variados) e colunas já sanitizadas. O arquivo `porsche_vendas_tratada.csv` (e a versão `.xlsx`) foi gerado assim:

| Etapa | O que foi feito | Por quê |
|---|---|---|
| Colunas mantidas | `sale_id` e as 9 colunas `*Sanitized` | Dados já padronizados |
| Colunas removidas | `customer_name`, `salesperson` e todas as colunas brutas | Dados pessoais não podem ir para um HTML público |
| Conversão | Ano, preço e quilometragem de texto para número | Para a IA e o JavaScript calcularem sem erro |
| Valores | Tudo colado como valor, sem fórmulas | Sem risco de `#REF!` |
| Conferência | 100 registros, `sale_id` únicos, cada cidade com um único estado, preços entre US$ 58.900 e US$ 286.500 | Garantir consistência |

**Decisão sobre os registros `INVALID`:** 24 datas continuaram `INVALID`. Eu as mantive e **não usei a coluna de data** na dashboard. Perguntas sobre tempo ficaram de fora. Uma segunda regra para recuperar essas datas a partir da coluna bruta fica como evolução.

**Regras de agrupamento na dashboard** (feitas no código, sem alterar a base):

- **Família do modelo:** os 41 modelos foram agrupados em 6 famílias pelo início do nome (911, 718, Cayenne, Macan, Taycan, Panamera), porque cada modelo tem só 1 a 4 vendas.
- **Status de entrega:** os 10 status viraram 3 grupos: *Entregue* (`Delivered`), *Cancelado* (`Cancelled`) e *Em andamento* (todos os demais). Vendas canceladas entram nos números, mas aparecem separadas por cor.
- **Pagamento:** mantive os 9 métodos separados.

## Prompt

### Versão 1 (briefing inicial)

```
Anexei uma base com 100 vendas da Porsche (CSV). Colunas:
sale_id, SaleDateSanitized, PorscheModelSanitized, ModelYearSanitized,
SalesPriceSanitized (USD), VehicleMileageSanitized, PayMethodSanitized,
CitySanitized, StateSanitized, DeliveryStatusSanitized.

TAREFA
Crie uma dashboard em UM único arquivo HTML (HTML, CSS e JavaScript
juntos). Os dados devem ficar embutidos no próprio HTML, sem depender
de arquivos externos.

PERGUNTAS DE NEGÓCIO QUE A DASHBOARD DEVE RESPONDER
1. Quais famílias de modelo geram mais receita e quais têm o maior
   ticket médio? Agrupe os modelos em 6 famílias pelo início do nome:
   911, 718, Cayenne, Macan, Taycan e Panamera.
2. Como os clientes pagam e o método de pagamento muda com o valor da
   venda? Mostre quantidade de vendas e ticket médio por método.
3. Quanto da receita já foi entregue, está em andamento ou foi cancelada?
   Agrupe o status de entrega assim:
   - Entregue: Delivered
   - Cancelado: Cancelled
   - Em andamento: todos os demais status

FILTROS (todos devem atualizar TODOS os gráficos e os indicadores)
Estado, família do modelo, ano do modelo, método de pagamento e
status (grupo). Inclua um botão "Limpar filtros".

INDICADORES NO TOPO
Receita total, número de vendas e ticket médio (conforme os filtros).

REGRAS DOS DADOS
- Ignore a coluna SaleDateSanitized: tem valores "INVALID". Não crie
  gráficos por tempo.
- Preço e ano são números.
- Gráficos com muito poucas vendas devem mostrar a quantidade, para
  ninguém tirar conclusões erradas.

VISUAL
Moderno, tecnológico e futurista. Modo escuro, fundo #0B1020, cards
#131A2E com cantos arredondados e leve brilho. Cor principal ciano
#22D3EE, secundária violeta #8B5CF6. Verde #34D399 para entregue,
âmbar #FBBF24 para em andamento, rosa #FB7185 para cancelado.
Fonte geométrica (Space Grotesk ou Inter). Layout responsivo.
Cada gráfico deve ter um título que diga a pergunta que responde.
```

### O que mudou até a versão final

A primeira versão do prompt sugeria usar o Canvas do ChatGPT e Chart.js via CDN. Como usei o Claude, o arquivo foi gerado direto, e dois pontos mudaram em relação ao briefing:

| Mudança | Motivo |
|---|---|
| Gráficos feitos em HTML e CSS puros, sem Chart.js | A página não depende de nenhuma biblioteca externa, abre em qualquer lugar e o código fica mais simples de explicar |
| Barras com menos de 5 vendas ficam esmaecidas, com nota no rodapé | Em vez de só mostrar a quantidade, o aviso de amostra pequena ficou visual |

**[PREENCHER: se você pedir ajustes depois, liste aqui cada pedido, por exemplo "troquei a pergunta 2", "adicionei gráfico por ano do modelo", e o motivo]**

### Versão final

**[PREENCHER: cole aqui o prompt final, com as mudanças acima aplicadas, se você ajustar o texto]**

## Ferramenta usada

Usei o **Claude** (Anthropic), em uma única conversa, do tratamento da planilha à geração do HTML e do README. Não usei o ChatGPT com Canvas nem um agente com skill, então **não fiz a comparação entre os dois caminhos** proposta no desafio. Escolhi uma única ferramenta porque estava trabalhando e queria manter todo o processo em um só lugar, com a mesma conversa guardando o contexto da base, das perguntas e das decisões.

## Como usar a dashboard

1. Abra o link da dashboard.
2. Use os filtros no topo (estado, família, ano, pagamento, status). Indicadores e gráficos se atualizam juntos.
3. Clique em **Limpar filtros** para voltar à visão geral.

Para conferir: sem filtros, a receita total deve ser de cerca de **US$ 12,83 mi** em **100 vendas**.

## Estrutura do repositório

```
.
├── index.html                    # dashboard (dados embutidos)
├── porsche_vendas_tratada.csv    # base tratada usada na dashboard
├── porsche_vendas_tratada.xlsx   # mesma base em planilha
├── README.md
└── prints/
    ├── dashboard.png
    └── filtro.png
```

## Ideias para evoluir

- Trocar o filtro de estado por cidade e comparar o que muda (cidade tem 77 valores quase sem repetição).
- Escrever uma segunda regra para recuperar as 24 datas `INVALID` e criar uma pergunta sobre evolução no tempo.
- Agrupar *Wire Transfer*, *Bank Transfer* e *ACH Payment* na pergunta de pagamento.
- Atualizar a dashboard enviando o HTML atual junto com uma base nova.
- Usar outra marca como referência visual e comparar o resultado.
