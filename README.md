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

## Parte 1: Primeiro contato

_(a preencher)_

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
