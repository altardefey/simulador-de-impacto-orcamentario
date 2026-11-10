# Interface do Simulador de Impacto Orçamentário (Salvador - BA)

Este documento descreve a **interface gráfica** do projeto de Big Data "Simulador de Impacto Orçamentário". Ela permite que um gestor ajuste um cenário hoteleiro (ocupação, diária, evento e mês) e veja a estimativa de **receita turística** e de **arrecadação de ISS** produzida pelo modelo.

A interface roda **dentro do Google Colab**, no mesmo notebook do backend, sem servidor externo.

## Sumário

1. [Tecnologias](#tecnologias)
2. [Visão geral das telas](#visão-geral-das-telas)
3. [Como executar](#como-executar)
4. [Estrutura das células](#estrutura-das-células)
5. [Integração com o backend](#integração-com-o-backend)
6. [Dados esperados](#dados-esperados)
7. [Decisões de projeto](#decisões-de-projeto)
8. [Personalização](#personalização)
9. [Solução de problemas](#solução-de-problemas)
10. [Limitações conhecidas](#limitações-conhecidas)

## Tecnologias

| Biblioteca           | Uso                                                                            |
| -------------------- | ------------------------------------------------------------------------------ |
| `ipywidgets`         | Sliders, menus, botões e abas;                                                  |
| `plotly`             | Gráficos (versão 5.5 ou superior, por causa do rótulo dentro do mapa de calor) |
| `pandas` / `numpy`   | Manipulação da tabela histórica                                                |
| HTML e CSS embutidos | Cabeçalho, cartões de resultado e tabelas                                      |
| `scikit-learn`       | Usado apenas no modelo provisório (veja [Modo provisório](#modo-provisório))   |

Todas já vêm instaladas no Colab.

## Visão geral das telas

A interface tem um cabeçalho fixo (título, período dos dados e situação do modelo) e três abas.

### Simulador

- **Painel de controles (esquerda):** taxa de ocupação (%), diária média (R$), fator de evento (mês regular, festivais ou alta temporada) e mês de referência. Há também o botão **Restaurar valores**.
- **Resultados (direita):**
  - **ISS estimado** em destaque, com a diferença em relação à média histórica do mês escolhido.
  - **Receita turística estimada**, também comparada com a média do mês.
  - **RevPAR do cenário**, calculado como ocupação × diária.
  - **Régua histórica** sob o ISS e a receita turística: uma barra mostra a faixa de valores daquele mês na base, com um traço na média e um ponto no cenário simulado.
  - **Gráfico** com o mesmo mês em cada ano da base e, ao lado, o cenário simulado.

### Histórico

Alternância entre receita turística e ISS, com quatro gráficos: evolução mensal, média por mês do ano (sazonalidade), efeito do fator de evento e correlação entre as variáveis.

### Sobre o modelo

- Tabela de desempenho no período de teste (erro médio e R²), comparando o modelo com a referência "mesmo mês do ano anterior". O menor erro de cada alvo aparece em negrito.
- Gráfico das variáveis que mais pesam na receita turística.
- Lista das bases de dados utilizadas.

## Como executar

Pré-requisito: acesso à pasta `Dados_turismo` no Google Drive ou ao arquivo `big data.zip` com os cinco CSVs.

1. Abra o notebook (`Simulador_Impacto_Orcamentario.ipynb`) no Colab.
2. Execute todas as células

Ordem final do notebook:

```
1. Leitura e consolidação (backend)
2. Treino e validação (backend)
3. Adaptador              (interface)
4. Config e dados         (interface)
5. Serviço                (interface)
6. Gráficos               (interface)
7. Interface              (interface)
```

## Estrutura das células

| Célula             | Função                                                                                                                                                                        |
| ------------------ | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| **Adaptador**      | Cria `simular()` e `metricas_modelo()` no formato que a interface espera, a partir das variáveis do backend. Só define as funções se `modelos` existir.                       |
| **Config e dados** | Imports, textos, paleta, estilo (CSS), funções de formatação (`fmt_reais`, `fmt_brl`, `fmt_pct`) e a tabela `hist`. Calcula também os limites dos sliders a partir dos dados. |
| **Serviço**        | Camada entre a interface e o backend. Usa `simular()` e `metricas_modelo()` se existirem; caso contrário, usa o modelo provisório.                                            |
| **Gráficos**       | Funções que devolvem figuras Plotly com estilo único: `grafico_cenario`, `serie_temporal`, `sazonalidade`, `efeito_evento`, `correlacao` e `grafico_importancias`.            |
| **Interface**      | Cria os widgets, as abas e a lógica de atualização. Deve ser a última célula executada.                                                                                       |

## Integração com o backend

A interface não importa módulos: ela detecta, no próprio notebook, as funções `simular` e `metricas_modelo`. Se ambas existirem, passa a usá-las.

### O que o Adaptador precisa encontrar

Variáveis geradas pelas células de treino do backend:

| Nome                                | Descrição                                                                                       |
| ----------------------------------- | ----------------------------------------------------------------------------------------------- |
| `modelos`                           | Dicionário com os modelos treinados: `modelos['receita_turistica']` e `modelos['receita_iss']`  |
| `entradas`                          | Lista das colunas usadas como entrada, na ordem do treino                                       |
| `resultados`                        | Lista de dicionários com as métricas, cada um com as chaves `Alvo`, `Método`, `MAE (R$)` e `R²` |
| `calcular_revpar(ocupacao, diaria)` | Função que devolve o RevPAR (ocupação em percentual)                                            |

Se algum desses nomes mudar, ajuste a célula do Adaptador.

### Contrato das funções

```python
def simular(ocupacao, diaria, fator_evento, mes) -> dict:
    # ocupacao em PERCENTUAL (por exemplo, 46.32), como nos CSVs
    return {'revpar': float, 'receita_turistica': float, 'receita_iss': float}

def metricas_modelo() -> dict:
    return {
        'r2_turismo': float, 'r2_iss': float,
        'mae_turismo': float, 'mae_iss': float,
        'importancias': {'taxa_ocupacao': 0.23, ...},
        'tabela': [  # opcional: alimenta a tabela comparativa
            {'Alvo': 'ISS', 'Método': 'Random Forest', 'MAE': float, 'R2': float},
        ],
    }
```

Se `tabela` não for informada, a aba "Sobre o modelo" monta uma tabela simples com os campos `mae_*` e `r2_*`.

### Modo provisório

Se `simular` ou `metricas_modelo` não existirem, a interface treina uma **regressão linear** provisória com os dados históricos e exibe um selo amarelo no cabeçalho ("Modelo provisório"). Serve apenas para testar a tela. As métricas desse modo são calculadas nos próprios dados de treino e **não** representam o desempenho real.

## Dados esperados

A interface usa o `df_final` do backend, com estas colunas:

| Coluna              | Descrição                                                                         |
| ------------------- | --------------------------------------------------------------------------------- |
| `ano_mes`           | Período mensal (`YYYY-MM`). Pode ser coluna ou índice (`PeriodIndex`)             |
| `taxa_ocupacao`     | Ocupação hoteleira, em percentual                                                 |
| `diaria_media`      | Diária média (R$)                                                                 |
| `revpar`            | RevPAR informado nos CSVs. Se existir `revpar_calculado`, a interface usa essa    |
| `fator_evento`      | 0 regular, 1 festivais, 2 alta temporada e Carnaval                               |
| `receita_turistica` | Receita turística (R$)                                                            |
| `receita_iss`       | Receita de ISS (R$)                                                               |
| `mes`               | Número do mês (1 a 12). Se não existir, a interface calcula a partir de `ano_mes` |

## Decisões de projeto

- **Sliders limitados à faixa histórica.** Ocupação e diária só variam entre o mínimo e o máximo observados na base. O Random Forest não extrapola: fora dessa faixa, a previsão apenas repetiria o valor da borda.
- **Ocupação em percentual.** Os CSVs usam valores como 46,32. A conversão para fração acontece dentro de `calcular_revpar`.
- **RevPAR calculado.** O RevPAR informado nos CSVs difere de ocupação × diária em 15 dos 68 meses (os 12 meses de 2021 e mais três de 2023). O backend treina e simula com o valor calculado, e a interface mostra o mesmo valor nos cartões e no gráfico de correlação.
- **Comparação com a média do mês.** Os cartões comparam o cenário com a média histórica do mês escolhido. Como o ISS cresce ao longo dos anos, a média reúne valores de anos diferentes.
- **Estimativas condicionadas.** Os valores dependem das entradas do cenário e não indicam relação de causa e efeito. O aviso aparece no rodapé do Simulador.
- **Tema claro e escuro.** O cabeçalho e o bloco de ISS têm fundo próprio. O restante herda as cores do Colab, e os gráficos usam fundo transparente.

## Personalização

| O que mudar         | Onde                                                                                                                              |
| ------------------- | --------------------------------------------------------------------------------------------------------------------------------- |
| Cores               | Constantes `BAIA`, `MARE`, `DENDE` e `CINZA` na célula **Config e dados** (as mesmas cores também aparecem dentro do bloco `CSS`) |
| Fonte               | Linha `@import` e `font-family` no bloco `CSS`. A interface usa a Public Sans, com fonte do sistema como alternativa              |
| Textos e rótulos    | Dicionários `ROTULOS_EVENTO`, `ROTULOS_CURTOS` e `DESCRICAO_DATASETS`                                                             |
| Limites dos sliders | Calculados a partir dos dados na mesma célula (`OCUPACAO_MIN`, `DIARIA_MAX` etc.)                                                 |
| Mês inicial         | `MES_PADRAO` (por padrão, o mês mais recente da base)                                                                             |
| Altura dos gráficos | Segundo argumento de `_estilo(fig, altura)` em cada função de gráfico                                                             |

## Solução de problemas

| Sintoma                                   | Causa provável                                                                     | O que fazer                                                                                                  |
| ----------------------------------------- | ---------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------ |
| Selo amarelo "Modelo provisório"          | O Adaptador não encontrou `modelos`                                                | Execute a célula de treino do backend e depois as da interface                                               |
| `NameError: modelos` ou `entradas`        | Célula de treino não executada                                                     | Rode as células do backend antes das da interface                                                            |
| `TypeError` ou erro estranho em `simular` | Uma versão antiga da `simular` (que só exibe HTML) ainda está na memória da sessão | Use a versão atual do notebook e **Reiniciar sessão e executar tudo**                                        |
| `TraitError` ao montar o painel           | Um objeto `HTML` do IPython foi colocado dentro de um `VBox`                       | Use `_hw()` (widget) em contêineres e `_h()` apenas com `display(...)`                                       |
| `KeyError: 'ano_mes'`                     | Versão antiga da célula Config e dados                                             | Use a versão atual, que aceita `ano_mes` como coluna ou índice                                               |
| Gráficos em branco                        | A célula de gráficos não foi executada, ou o Plotly está desatualizado             | Rode a célula de gráficos e depois a de interface. Se persistir, atualize o Plotly (`pip install -U plotly`) |
| Fonte diferente da esperada               | O Colab bloqueou o Google Fonts                                                    | Sem efeito funcional: a fonte do sistema é usada                                                             |

## Limitações conhecidas

- **Modelo.** Na validação feita nos dados reais, o Random Forest atual perde para a referência "mesmo mês do ano anterior". Os principais motivos observados foram uma quebra na receita turística em 2024 e uma tendência de crescimento do ISS que árvores de decisão não acompanham. Sugestões em estudo para o backend: uma variável indicadora de período (por exemplo, pós-2024) para a receita turística e um modelo linear com tendência para o ISS. A interface não precisa mudar com isso, desde que os nomes do Adaptador sejam mantidos.
- **ISS e período.** Se o ISS passar a depender de tendência temporal, será preciso definir qual período usar na simulação (por exemplo, o mais recente ou um seletor de ano na interface).
- **Poucos dados.** A base tem 68 meses. As métricas calculadas em 12 meses de teste variam bastante, e o R² deve ser lido junto com o erro médio.
- **Ambiente.** A interface foi pensada para o Colab. Em Jupyter local, é necessário ter `ipywidgets` habilitado.
- **Testes.** A lógica foi verificada com os dados reais usando componentes de teste no lugar dos widgets e dos gráficos. A aparência final dos sliders, das abas e dos gráficos deve ser conferida no Colab.
