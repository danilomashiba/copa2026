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

> **Observação sobre a natureza dos dados:** o próprio Kaggle informa que o dataset foi **gerado sinteticamente e pode não refletir dados reais**. A análise confirmou esse afastamento em pontos relevantes: o dataset possui **1.050 partidas**, contra **104 partidas da Copa do Mundo de 2026**; todos os **1.248 jogadores** aparecem em múltiplas partidas na mesma data em algum momento, chegando a **5 partidas no mesmo dia**; e as métricas agregadas de torneio não permitem reconstruir de forma consistente uma trajetória única por jogador. Portanto, essas diferenças devem ser interpretadas considerando a natureza simulada da base, e não como erros de coleta de dados reais.

## Parte 2: Transformação e métricas

_(a preencher)_

## Parte 3: Visualização

_(a preencher)_

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
