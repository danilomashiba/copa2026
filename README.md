# Análise de Performance: Copa do Mundo 2026
> Desafio hands-on com dataset Copa do Mundo 2026.

## Parte 0: Download e preparação

### Qual caminho usou e por quê? Como garante que o arquivo veio íntegro?

Optei pelo caminho C, **kagglehub**, dentro do Google Colab para evitar etapas manuais e tornar o processo de download reproduzível. A versão 1 do dataset foi fixada para evitar alterações caso novas versões sejam publicadas.

- **Dataset:** FIFA World Cup 2026 Player Performance Dataset
- **Autor:** rauffauzanrambe
- **Link:** https://www.kaggle.com/datasets/rauffauzanrambe/fifa-world-cup-2026-player-performance-dataset
- **Handle:** `rauffauzanrambe/fifa-world-cup-2026-player-performance-dataset/versions/1`
- **Licença:** MIT
- **Credenciais:** não foram necessárias, pois o dataset é público.

Foram realizadas as seguintes verificações de integridade e consistência:

- **Nome do arquivo:** `fifa_world_cup_2026_player_performance.csv`, o único arquivo baixado.
- **Tamanho em disco:** 17,2 MB, como esperado.
- **Dimensões:** 54.600 linhas × 75 colunas. As 75 colunas conferem com o enunciado.
- **SHA-256:** `322c5efd933578a61ac6de612b4400011849efa7c6a1b88037be67f100fb4e99`, utilizado como identificação da versão analisada do arquivo.
- **Inspeção visual:** `df.head()` apresentou os primeiros registros sem anomalias aparentes.

### Stack utilizada: 

Foi utilizado **Python com pandas** porque o volume do dataset é compatível com processamento em memória e a biblioteca permite realizar exploração, validação e transformação de forma direta. **Matplotlib** foi utilizado para as visualizações e **KaggleHub** para tornar o download reproduzível.

### Onde as credenciais (`kaggle.json`) não devem ser guardadas — e por quê?

Não devem ser guardadas no repositório Git, em células do notebook, em prints de tela ou em pastas compartilhadas, pois as chaves podem expor a conta do Kaggle a acessos não autorizados. Em repositórios Git, mesmo que o arquivo seja removido em um commit posterior, a chave pode permanecer registrada no histórico. Em caso de vazamento, é necessário revogar o token no Kaggle e gerar um novo.

### O que a licença MIT permite e exige ao publicar um resultado?

A licença MIT permite usar, copiar, modificar e distribuir o dataset, desde que, ao redistribuí-lo, sejam mantidos o aviso de copyright e o texto da licença, com atribuição ao autor.

### O que deve e o que não deve ser versionado no Git?

| Versionar | Não versionar |
|---|---|
| Notebook, README, `.gitignore`, `requirements.txt` | CSV original |
| Código e scripts | `kaggle.json`, senhas e tokens |
| Gráficos finais pequenos | Arquivos temporários ou desnecessários para reproduzir o projeto |

## Parte 1 — Primeiro contato e sanidade dos dados

### T1.1 — Dimensão, memória e otimização

O dataset possui **54.600 linhas e 75 colunas**, ocupando aproximadamente **67,28 MB** em memória quando medido com `deep=True`.

A maior parte das colunas numéricas estava originalmente em `int64` ou `float64`, enquanto variáveis textuais estavam como `object`.

Para reduzir o consumo de memória sem alterar a estrutura do dataset:

- colunas categóricas foram convertidas para `category`;
- `match_date` foi convertida para `datetime`;
- colunas numéricas foram reduzidas para tipos menores compatíveis com seus valores.

Após a otimização, o uso de memória caiu para aproximadamente **13,15 MB**, uma redução de cerca de **80,5%**, mantendo as mesmas **54.600 linhas e 75 colunas**.

### T1.2 — Mapa de qualidade

O mapa de qualidade mostrou **75 colunas sem valores nulos e sem variáveis constantes**.

Considerando apenas a consistência interna dos dados, as três colunas mais suspeitas são:

- **`total_minutes_tournament`** — apresenta múltiplos valores para o mesmo jogador e reduções entre registros, não se comportando claramente como total final ou acumulado.
- **`total_goals_tournament`** — apresenta o mesmo comportamento, com valores que podem diminuir ao longo dos registros de um jogador.
- **`total_assists_tournament`** — também varia e apresenta reduções, dificultando sua interpretação como métrica acumulada do torneio.

A análise mostrou ainda que o mesmo jogador pode aparecer em múltiplas partidas na mesma data, sem uma variável que permita identificar diferentes cenários ou simulações. Isso impede validar com segurança a trajetória das métricas `total_*_tournament`.

> **Observação sobre a natureza dos dados:** o próprio Kaggle informa que o dataset foi **gerado sinteticamente** e pode não refletir dados reais. A análise confirmou esse afastamento em pontos relevantes: o dataset possui **1.050 partidas**, contra **104 partidas da Copa do Mundo de 2026**; todos os **1.248 jogadores** aparecem em múltiplas partidas na mesma data em algum momento, chegando a **5 partidas no mesmo dia**; e as métricas agregadas de torneio não permitem reconstruir de forma consistente uma trajetória única por jogador.
>
> Em uma situação real de produção, eu **não apresentaria conclusões de negócio com uma base que se afasta de forma material do que ocorreu em produção**. Antes de seguir com a análise, eu buscaria esclarecer a origem, a granularidade e a regra de geração desses registros. Neste case, prossigo considerando explicitamente a natureza sintética do dataset e tratando essas limitações como parte da análise.

### T1.3 — As 10 categorias

Para demonstrar o entendimento das variáveis, foi selecionada uma coluna representativa de cada um dos 10 grupos lógicos do dataset.

| Categoria | Coluna | O que mede | Unidade |
|---|---|---|---|
| Perfil do jogador | `age` | Idade do jogador | Anos |
| Partida | `minutes_played` | Tempo jogado na partida | Minutos |
| Ataque | `goals` | Gols marcados pelo jogador | Quantidade |
| Passe/Criação | `pass_accuracy` | Proporção de passes certos | Proporção de 0 a 1 |
| Defesa | `tackles` | Desarmes realizados | Quantidade |
| Disciplina | `yellow_cards` | Cartões amarelos recebidos | Quantidade |
| Goleiro | `saves` | Defesas realizadas | Quantidade |
| Físico | `distance_covered_km` | Distância percorrida | Quilômetros |
| Performance | `player_rating` | Nota de desempenho do jogador | Pontuação de 0 a 10 |
| Resumo do torneio | `total_goals_tournament` | Total de gols atribuído ao jogador no torneio | Quantidade |

Os intervalos observados nos dados foram utilizados para validar as unidades e a interpretação das variáveis, como em `pass_accuracy`, que varia de **0,42 a 0,97** e, portanto, está armazenada como proporção.

### T1.4 — Regras de coerência

Foram testadas as seguintes regras:

- `shots_on_target <= shots`
- `0 <= minutes_played <= 90`
- `total_minutes_tournament <= partidas * 90`
- estatísticas de goleiro apenas para jogadores da posição `Goalkeeper`
- idade, altura e peso dentro de faixas plausíveis
- jogadores com `minutes_played = 0` não deveriam registrar assistências

| Regra | Violações |
|---|---:|
| `shots_on_target <= shots` | 0 |
| `0 <= minutes_played <= 90` | 0 |
| `total_minutes_tournament <= partidas * 90` | 0 |
| Estatísticas de goleiro apenas para goleiros | 0 |
| Idade, altura e peso plausíveis | 0 |
| Assistências com `minutes_played = 0` | 30 |

Foi identificada uma inconsistência interna: **30 registros possuem assistências atribuídas a jogadores com zero minutos em campo**.

As demais regras básicas de coerência avaliadas não apresentaram violações.

## Parte 2: Transformação e métricas

### T2.1 — Métricas por 90 minutos

Foram criadas as métricas:

- `goals_per90`
- `assists_per90`
- `key_passes_per90`

As métricas foram calculadas apenas para registros com `minutes_played > 0`.

Nos registros com `minutes_played = 0`, as métricas por 90 minutos foram mantidas como `NaN`, pois não existe uma taxa por 90 definida para jogadores sem participação em campo.

A validação confirmou que:

- não existem valores infinitos;
- todos os valores `NaN` ocorrem exclusivamente em registros com `minutes_played = 0`;
- os cálculos foram conferidos manualmente em registros de exemplo e apresentaram resultados consistentes.

### T2.2 — Top 10 artilheiros por 90 minutos

O ranking bruto por registro apresentou valores extremos causados por baixa amostra, como **1 gol em 5 minutos = 18 gols/90**.

Para tornar a comparação mais representativa, os dados foram primeiro agregados por jogador, somando gols e minutos jogados. Em seguida, foi calculada a taxa de gols por 90 minutos sobre o total acumulado de cada atleta.

| Posição | Jogador | Seleção | Gols | Minutos | Gols/90 |
|---:|---|---|---:|---:|---:|
| 1 | Mohannad Majeed | Iraq | 15 | 1.396 | 0,967 |
| 2 | Memphis Zerrouki | Netherlands | 24 | 2.235 | 0,966 |
| 3 | Saman Azmoun | Iran | 13 | 1.219 | 0,960 |
| 4 | Timothy Weah | United States | 12 | 1.214 | 0,890 |
| 5 | Randal Duarte | Costa Rica | 14 | 1.477 | 0,853 |
| 6 | Vinicius Nunes | Brazil | 15 | 1.767 | 0,764 |
| 7 | Themba Xoki | South Africa | 13 | 1.586 | 0,738 |
| 8 | Moises Pellerano | Ecuador | 11 | 1.360 | 0,728 |
| 9 | Andre Bassogog | Cameroon | 13 | 1.634 | 0,716 |
| 10 | Kasey Hector | Jamaica | 16 | 2.091 | 0,689 |

A principal conclusão é que métricas por 90 minutos devem ser acompanhadas de um volume mínimo de participação para evitar rankings distorcidos por poucos minutos em campo.

### T2.3 — Agregado por seleção

Foram considerados apenas registros com `minutes_played > 0` e agregadas por seleção as seguintes métricas:

- soma de gols;
- soma de assistências;
- média de `player_rating`;
- soma de `distance_covered_km`.

| Seleção | Gols | Assistências | Rating médio | Distância total (km) |
|---|---:|---:|---:|---:|
| Qatar | 95 | 93 | 6,29 | 6.872,3 |
| Netherlands | 94 | 82 | 6,46 | 5.614,0 |
| Panama | 90 | 81 | 6,36 | 4.969,6 |
| Cameroon | 88 | 65 | 6,40 | 4.602,4 |
| Saudi Arabia | 82 | 67 | 6,29 | 5.431,9 |
| Jamaica | 79 | 79 | 6,31 | 6.117,3 |
| Tunisia | 78 | 48 | 6,18 | 4.921,1 |
| Costa Rica | 76 | 60 | 6,25 | 4.708,9 |
| Ghana | 75 | 64 | 6,39 | 4.722,4 |
| Iran | 72 | 70 | 6,41 | 3.954,2 |

Os volumes de gols, assistências e distância continuam elevados, coerentes com a natureza sintética e a quantidade de partidas presentes no dataset.

O `player_rating` médio entre seleções apresentou baixa variação, ficando entre aproximadamente **6,15 e 6,46**.

### T2.4 — Goals vs xG

Foi calculada a métrica:

`over_performance = goals - expected_goals_xg`

Valores positivos indicam jogadores que marcaram acima do esperado pelo xG; valores negativos indicam jogadores que marcaram abaixo do esperado.

#### Maiores over-performers

| Jogador | Seleção | Gols | xG | Over-performance |
|---|---|---:|---:|---:|
| Memphis Zerrouki | Netherlands | 24 | 3,22 | +20,78 |
| Kasey Hector | Jamaica | 16 | 2,90 | +13,10 |
| Mohannad Majeed | Iraq | 15 | 2,13 | +12,87 |
| Eric Rodriguez | Panama | 15 | 2,15 | +12,85 |
| Andre Bassogog | Cameroon | 13 | 1,69 | +11,31 |
| Themba Xoki | South Africa | 13 | 1,72 | +11,28 |
| Randal Duarte | Costa Rica | 14 | 2,77 | +11,23 |
| Eric Mba | Cameroon | 14 | 3,15 | +10,85 |
| Saman Azmoun | Iran | 13 | 2,23 | +10,77 |
| Timothy Weah | United States | 12 | 1,27 | +10,73 |

#### Maiores under-performers

| Jogador | Seleção | Gols | xG | Over-performance |
|---|---|---:|---:|---:|
| Granit Kobal | Switzerland | 6 | 7,95 | -1,95 |
| Gavi Le Normand | Spain | 4 | 5,88 | -1,88 |
| Theo Hernandez | France | 0 | 1,79 | -1,79 |
| Gary Delgado | Chile | 1 | 2,42 | -1,42 |
| Francisco Aguilar | Costa Rica | 0 | 1,40 | -1,40 |
| Ruslan Bondarenko | Ukraine | 0 | 1,39 | -1,39 |
| Mateo Stanisic | Croatia | 0 | 1,31 | -1,31 |
| Stuart Shankland | Scotland | 0 | 1,24 | -1,24 |
| Sphephelo Mbatha | South Africa | 0 | 1,21 | -1,21 |
| Wilfred Iheanacho | Nigeria | 0 | 1,20 | -1,20 |

O `over_performance` variou de **-1,95 a +20,78**, com média de **+1,72**. A forte assimetria positiva chama atenção e pode estar relacionada à natureza sintética do dataset.

> **Limitação:** a diferença entre gols e xG não explica sozinha a qualidade de finalização. A métrica não captura fatores como contexto da chance, dificuldade da finalização, posição do jogador, volume de chutes ou qualidade do modelo utilizado para gerar o xG.

### T2.5 — Persistência em Parquet

O DataFrame tratado foi salvo em formato Parquet:

`fifa_world_cup_2026_tratado.parquet`

A validação confirmou que a estrutura e os tipos das colunas foram preservados:

- **DataFrame tratado:** 54.600 linhas × 78 colunas
- **Arquivo Parquet:** 54.600 linhas × 78 colunas
- **Estrutura preservada:** `True`
- **Tipos preservados:** `True`

Na comparação de tamanho, utilizando a mesma unidade de medida:

- **CSV original:** 17,20 MB
- **Parquet:** 2,04 MB
- **Redução de tamanho:** 88,2%

Duas vantagens do Parquet neste projeto:

- **Menor armazenamento:** o formato utiliza compressão eficiente e reduziu significativamente o tamanho do arquivo.
- **Leitura analítica mais eficiente:** por ser colunar, permite carregar apenas as colunas necessárias em análises e consultas.

## Parte 3: Visualização

### T3.1 — Scatter xG × Goals

Foi criado um gráfico de dispersão entre `expected_goals_xg` e `goals`, utilizando a linha `y = x` como referência.

Foram destacados os principais desvios nos dois sentidos:

- **Memphis Zerrouki:** +20,78 gols acima do xG
- **Kasey Hector:** +13,10
- **Granit Kobal:** -1,95
- **Gavi Le Normand:** -1,88

Jogadores acima da linha marcaram mais gols do que o esperado pelo xG; jogadores abaixo da linha marcaram menos.

O gráfico evidencia uma forte assimetria positiva, com os maiores desvios concentrados acima da linha de referência.

<img width="889" height="590" alt="image" src="https://github.com/user-attachments/assets/f215f48c-324f-4ca4-a884-c72fd295f895" />


### T3.2 — Player Rating por posição

Foram comparadas as distribuições de `player_rating` entre as quatro posições, considerando apenas registros com `minutes_played > 0`.

| Posição | Rating médio | Desvio-padrão | Mínimo | Máximo |
|---|---:|---:|---:|---:|
| Forward | 6,36 | 0,752 | 3,7 | 9,2 |
| Midfielder | 6,31 | 0,738 | 3,7 | 9,2 |
| Defender | 6,24 | 0,722 | 3,6 | 9,4 |
| Goalkeeper | 6,21 | 0,717 | 3,9 | 8,6 |

A posição com maior dispersão foi **Forward**, mas a diferença em relação às demais posições é pequena.

Esse resultado pode fazer sentido pela maior variabilidade do desempenho ofensivo, mas não indica uma diferença forte entre as posições.

<img width="678" height="547" alt="image" src="https://github.com/user-attachments/assets/3c3ec2a5-86fa-43f2-be06-59dbdb945b0b" />


### T3.3 — “Pizza 3D” do gerente

Foi criado um gráfico de pizza com **24 seleções** para evidenciar as limitações desse formato quando há muitas categorias.

Mesmo sem efeitos 3D ou sombra, a visualização apresenta baixa legibilidade:

- muitas fatias possuem proporções muito próximas;
- os rótulos ficam espalhados ao redor do gráfico;
- a comparação entre países é difícil;
- o ranking não fica evidente.

Por isso, eu não recomendaria uma pizza 3D para esse caso. A alternativa mais adequada seria um **gráfico de barras ordenado**, que facilita a comparação entre seleções.

O objetivo do pedido pode ser atendido melhor com uma visualização mais simples e mais precisa.

<img width="980" height="980" alt="image" src="https://github.com/user-attachments/assets/01e08a51-e0e7-4d5c-8da1-96ad68014e4f" />

<img width="912" height="701" alt="image" src="https://github.com/user-attachments/assets/778536a8-cd5b-44a2-9a6e-c61f166bbda4" />

### T3.4 — Bônus front-end

Foi criada uma **mini-página HTML self-contained**, sem uso de CDN ou bibliotecas externas.

A página permite alternar entre:

- **Jogadores**
  - Gols
  - Gols por 90 minutos
  - Over-performance (`goals - xG`)

- **Seleções**
  - Gols
  - Assistências
  - Rating médio
  - Distância total percorrida

Também é possível selecionar rankings de **Top 5, Top 10 ou Top 20**.

Os dados foram convertidos para **JSON** e incorporados diretamente no HTML. O gráfico de barras foi criado manualmente com **SVG e JavaScript**, atendendo ao requisito de uma página independente e sem dependências externas.

O arquivo gerado foi: https://danilomashiba.github.io/copa2026/ranking_interativo.html

## Parte 4: Pergunta de negócio e comunicação

### T4.1 — 5 jogadores mais subvalorizados

Para identificar jogadores com alto impacto ofensivo e baixo `player_rating`, foram considerados apenas atletas com pelo menos **90 minutos**.

O impacto ofensivo foi calculado a partir de:

- gols por 90 minutos;
- assistências por 90 minutos;
- key passes por 90 minutos.

As três métricas foram transformadas em **percentis**, colocando-as na mesma escala. O score de impacto representa a média desses percentis, variando de **0 a 100**.

O `player_rating` foi calculado como média ponderada pelos minutos jogados. Foram considerados subvalorizados os jogadores com rating abaixo da mediana do grupo, aproximadamente **6,20**.

| Jogador | Seleção | Gols | Assistências | Key Passes | Minutos | Rating | Impacto |
|---|---|---:|---:|---:|---:|---:|---:|
| Rodri Mikel | Spain | 6 | 6 | 29 | 1.528 | 6,18 | 87,15 |
| Michael Brown | Panama | 9 | 7 | 30 | 1.761 | 6,18 | 86,83 |
| Edouard Ndiaye | Senegal | 5 | 6 | 59 | 2.051 | 6,14 | 86,19 |
| Nicolas Anguissa | Cameroon | 8 | 4 | 27 | 1.527 | 6,01 | 83,76 |
| Aleksandar Lukic | Serbia | 2 | 7 | 40 | 1.558 | 6,20 | 82,61 |

**Rodri Mikel** apresentou o maior score de impacto entre os jogadores com rating abaixo da mediana, com **87,15 pontos de impacto** e rating médio de **6,18**.

> **Validação do corte:** também foi testado um corte mínimo de **180 minutos**, e o Top 5 permaneceu exatamente igual, indicando estabilidade do ranking em relação a esse parâmetro.

> **Limitação:** o score é uma construção analítica para este case e considera apenas gols, assistências e key passes. Ele não representa uma medida completa da contribuição de um jogador e pode deixar de capturar aspectos defensivos, função tática e contexto das partidas.

### T4.2 — Comunicação executiva

**Rodri Mikel se destaca como o jogador de maior impacto ofensivo entre aqueles com rating abaixo da mediana, com score de impacto de 87,15 e rating médio de 6,18.**

Número de suporte: **6 gols, 6 assistências e 29 key passes em 1.528 minutos**.

Ressalva: o score de impacto é uma métrica construída para esta análise e considera apenas produção ofensiva, não capturando integralmente contribuição defensiva, função tática ou contexto das partidas.

### T4.3 — Pensamento crítico

Uma correlação de **0,8 entre Sprints e Rating** não permite concluir que “correr mais melhora a nota”.

Correlação indica associação, não causalidade. Jogadores com maior intensidade podem também atuar em posições específicas, jogar mais minutos ou participar mais de ações ofensivas, fatores que podem elevar simultaneamente o número de sprints e o rating.

Para sustentar uma relação causal, seria necessário controlar essas variáveis e testar se o efeito permanece.


## Limitações da análise

O dataset é sintético e apresenta uma estrutura que não permite identificar diferentes cenários ou simulações. Por isso, não foi possível validar de forma confiável a trajetória das métricas `total_*_tournament`.

Também foram identificados registros inconsistentes, como assistências atribuídas a jogadores com zero minutos em campo.

Em um ambiente de produção, esses pontos seriam esclarecidos com a origem dos dados e as regras de geração antes da apresentação de conclusões de negócio.


## Créditos e licença

- **Dataset:** FIFA World Cup 2026 Player Performance Dataset
- **Autor:** **rauffauzanrambe**
- **Fonte:** https://www.kaggle.com/datasets/rauffauzanrambe/fifa-world-cup-2026-player-performance-dataset
- **Licença do dataset:** MIT
- **Atribuição:** mantida ao autor original do dataset.
