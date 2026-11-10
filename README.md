# Simulador de Impacto Orçamentário — Salvador (BA)

E se a ocupação dos hotéis subisse durante um festival em Salvador? Neste projeto de faculdade, juntamos dados mensais de turismo e arrecadação para explorar como cenários de ocupação, diária e eventos se refletem nas estimativas de receita turística e de ISS (Imposto Sobre Serviços).

## Como rodar

1. Abra o [notebook no Google Colab](https://colab.research.google.com/github/altardefey/simulador-de-impacto-orcamentario/blob/main/Simulador_Impacto_Orcamentario.ipynb).
2. Clique em **Executar tudo**. Quando o Colab pedir acesso ao Google Drive, autorize a conexão. O notebook procura os CSVs na pasta `Meu Drive/Dados_turismo`. Se a pasta não existir, envie um ZIP com os cinco CSVs quando solicitado. As bases usadas no trabalho ficam separadas do repositório.
3. Na seção **Painel de cenários**, ajuste a ocupação hoteleira, a diária média, o tipo de evento e o mês. O painel atualiza o RevPAR, a receita turística estimada e o ISS estimado. A diferença de ISS usa a média histórica do mês como referência.

## Dados e modelo

Cruzamos cinco bases pela coluna `ano_mes`, reunindo 68 meses de janeiro de 2021 a agosto de 2026.

| Arquivo | O que contém |
| --- | --- |
| `Taxa_ocupação.csv` | Ocupação hoteleira mensal, em percentual |
| `Diaria_media_RevPar.csv` | Diária média e receita por quarto disponível (RevPAR) |
| `Fator_evento.csv` | Mês regular (0), festival (1) ou alta temporada (2) |
| `Receita_turistica.csv` | Receita turística mensal |
| `Receita_iss.csv` | Arrecadação mensal de ISS dos setores ligados ao turismo |

Usamos **pandas** para reunir as bases, **scikit-learn** para treinar dois modelos Random Forest (um por receita) e **ipywidgets** para montar o painel. Calculamos o RevPAR no treino e na simulação da mesma forma, multiplicando a diária média pela taxa de ocupação dividida por 100.

## O que apareceu no teste

Reservamos os 12 meses mais recentes (setembro de 2025 a agosto de 2026) e comparamos o Random Forest com uma referência simples, que repete o valor do mesmo mês do ano anterior.

| Erro absoluto médio | Random Forest | Mesmo mês do ano anterior |
| --- | ---: | ---: |
| Receita turística | R$ 146,9 milhões | R$ 69,4 milhões |
| ISS | R$ 35,7 milhões | R$ 21,9 milhões |

A referência teve erro menor nos dois casos. O painel serve para explorar cenários. A diferença de ISS em relação à média histórica indica quanto a estimativa ficou acima ou abaixo dessa referência.

## Equipe

- [altardefey](https://github.com/altardefey)
- [Caroline-SC](https://github.com/Caroline-SC)
- [LariDev0](https://github.com/LariDev0)
- [Skytale97](https://github.com/Skytale97)
- [Recif3](https://github.com/Recif3)
