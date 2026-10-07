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

Para demonstrar o entendimento das variáveis, foi selecionada uma coluna representativa de cada uma das 10 categorias do dataset.

| Categoria | Coluna | O que mede | Unidade |
|---|---|---|---|
| Identificação | `player_id` | Identificador único do jogador | Sem unidade |
| Perfil | `age` | Idade do jogador | Anos |
| Partida | `minutes_played` | Tempo jogado na partida | Minutos |
| Ataque | `goals` | Gols marcados pelo jogador | Quantidade de gols |
| Passe/Criação | `pass_accuracy` | Proporção de passes certos | Proporção de 0 a 1 |
| Defesa | `tackles` | Desarmes realizados | Quantidade |
| Disciplina | `yellow_cards` | Cartões amarelos recebidos | Quantidade |
| Goleiro | `saves` | Defesas realizadas | Quantidade |
| Físico | `distance_covered_km` | Distância percorrida | Quilômetros |
| Performance | `player_rating` | Nota de desempenho do jogador | Pontuação de 0 a 10 |

Os intervalos observados nos dados foram utilizados para validar as unidades e a interpretação das variáveis, como em `pass_accuracy`, que varia de **0,42 a 0,97** e, portanto, está armazenada como proporção.

### T1.4 — Regras de coerência

Foram testadas as seguintes regras:

- `shots_on_target <= shots`
- `0 <= minutes_played <= 90`
- `total_minutes_tournament <= partidas * 90`
- estatísticas de goleiro apenas para jogadores da posição `Goalkeeper`
- idade, altura e peso dentro de faixas plausíveis

Nenhuma violação foi encontrada nas regras avaliadas.

| Regra | Violações |
|---|---:|
| `shots_on_target <= shots` | 0 |
| `0 <= minutes_played <= 90` | 0 |
| `total_minutes_tournament <= partidas * 90` | 0 |
| Estatísticas de goleiro apenas para goleiros | 0 |
| Idade, altura e peso plausíveis | 0 |

Apesar dos problemas de granularidade identificados na T1.2, as regras básicas de coerência entre variáveis apresentaram comportamento consistente.

## Parte 2: Transformação e métricas

### T2.1 — Métricas por 90 minutos

Foram criadas as métricas:

- `goals_per90`
- `assists_per90`
- `key_passes_per90`

As métricas foram calculadas apenas para registros com `minutes_played > 0`, evitando divisão por zero.

A validação confirmou:

- **0 valores nulos**
- **0 valores infinitos**

Os resultados foram conferidos manualmente em registros de exemplo e apresentaram cálculo consistente.

### T2.2 — Top 10 artilheiros por 90 minutos

O ranking bruto por registro apresentou valores extremos causados por baixa amostra, como **1 gol em 5 minutos = 18 gols/90**.

Para tornar o ranking mais representativo, os dados foram agregados por jogador antes do cálculo de `goals_per90`, utilizando o total de gols e minutos de cada atleta.

Também foi aplicado um corte mínimo de **180 minutos**. O corte não alterou o Top 10 final, pois os jogadores líderes já possuíam volume de minutos suficiente.

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

A principal conclusão é que métricas por 90 minutos precisam ser acompanhadas de um volume mínimo de participação para evitar rankings distorcidos por poucos minutos em campo.

### T2.3 — Agregado por seleção

Foram agregados por seleção:

- soma de gols;
- soma de assistências;
- média de `player_rating`;
- soma de `distance_covered_km`.

| Seleção | Gols | Assistências | Rating médio | Distância total (km) |
|---|---:|---:|---:|---:|
| Qatar | 95 | 93 | 3,66 | 6.872,3 |
| Netherlands | 94 | 85 | 3,71 | 5.614,0 |
| Panama | 90 | 81 | 3,71 | 4.969,6 |
| Cameroon | 88 | 66 | 3,71 | 4.602,4 |
| Saudi Arabia | 82 | 68 | 3,63 | 5.431,9 |
| Jamaica | 79 | 80 | 3,66 | 6.117,3 |
| Tunisia | 78 | 48 | 3,62 | 4.921,1 |
| Costa Rica | 76 | 61 | 3,60 | 4.708,9 |
| Ghana | 75 | 65 | 3,70 | 4.722,4 |
| Iran | 72 | 70 | 3,70 | 3.954,2 |

Os volumes de gols, assistências e distância são elevados, reflexo da estrutura sintética e da quantidade de partidas presentes no dataset.

Também chama atenção a baixa variação do `player_rating` médio entre seleções, que ficou entre aproximadamente **3,52 e 3,76**, mesmo com diferenças relevantes nas demais métricas.

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

A validação confirmou que a estrutura foi preservada:

- **DataFrame tratado:** 54.600 linhas × 78 colunas
- **Arquivo Parquet:** 54.600 linhas × 78 colunas
- **Estrutura preservada:** `True`
- **Tamanho do Parquet:** 1,94 MB

Comparado ao CSV original de aproximadamente **17,2 MB**, o arquivo Parquet ficou cerca de **89% menor**.

Duas vantagens do Parquet neste projeto:

- **Menor armazenamento:** utiliza compressão eficiente, reduzindo significativamente o tamanho do arquivo.
- **Leitura analítica mais eficiente:** o formato colunar permite carregar apenas as colunas necessárias em análises e consultas.

## Parte 3: Visualização

### T3.1 — Scatter xG × Goals

Foi criado um gráfico de dispersão entre `expected_goals_xg` e `goals`, com a linha `y = x` como referência.

Os principais outliers destacados foram:

- **Memphis Zerrouki:** +20,78 gols acima do xG
- **Kasey Hector:** +13,10
- **Mohannad Majeed:** +12,87

Jogadores acima da linha marcaram mais gols do que o esperado pelo xG; jogadores abaixo da linha marcaram menos.

O gráfico reforça a forte assimetria positiva observada anteriormente nas métricas de `over_performance`.

<img width="767" height="547" alt="image" src="https://github.com/user-attachments/assets/f39bce9d-1489-4bd2-9979-987a4e8f6410" />

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

O arquivo gerado foi:

`ranking_interativo.html`

## Parte 4: Pergunta de negócio e comunicação

_(a preencher)_

## Limitações e o que não consegui validar

- **Hipótese a validar:** 54.600 linhas parece incompatível com uma Copa real, considerando a quantidade esperada de jogadores e partidas. O dataset pode conter dados sintéticos ou registros em granularidade diferente da esperada. Essa hipótese será investigada na Parte 1.

## Créditos e licença

- **Dataset:** FIFA World Cup 2026 Player Performance Dataset
- **Autor:** **rauffauzanrambe**
- **Fonte:** https://www.kaggle.com/datasets/rauffauzanrambe/fifa-world-cup-2026-player-performance-dataset
- **Licença do dataset:** MIT
- **Atribuição:** mantida ao autor original do dataset.
