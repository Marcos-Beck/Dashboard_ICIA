# Painel BI do Índice de Confiança da Indústria do Ferro 
#### Por Marcos Beck
#### [Link público do Dashboard](https://app.powerbi.com/view?r=eyJrIjoiMDgxYjc2ZmQtMTI5Mi00NTEwLTlmMjAtMDMxOGNjYTdlNDBlIiwidCI6IjY1OWNlMmI4LTA3MTQtNDE5OC04YzM4LWRjOWI2MGFhYmI1NyJ9&pageName=c201bcc3c4352cb77162)
---
# Índice
1. [Sobre o Painel](https://github.com/Marcos-Beck/Dashboard_ICIA/new/main?filename=README.md#sobre-o-painel-)
2. [Dados](https://github.com/Marcos-Beck/Dashboard_ICIA/new/main?filename=README.md#dados-)
3. [Visualizações](https://github.com/Marcos-Beck/Dashboard_ICIA/new/main?filename=README.md#visualiza%C3%A7%C3%B5es-)
4. [Medidas Principais](https://github.com/Marcos-Beck/Dashboard_ICIA/new/main?filename=README.md#medidas-principais-)
5. [Tecnologias usadas](https://github.com/Marcos-Beck/Dashboard_ICIA/new/main?filename=README.md#tecnologias-usadas-%EF%B8%8F)
6. [Possiveis Melhorias](https://github.com/Marcos-Beck/Dashboard_ICIA/new/main?filename=README.md#tecnologias-usadas-%EF%B8%8F)
7. [Vantagens do Painel](https://github.com/Marcos-Beck/Dashboard_ICIA/new/main?filename=README.md#vantagens-do-painel)

## Sobre o Painel 🔍
Feito para **monitorar e analisar o ICIA** de maneira clara, com objetivo de oferecer facilidade ao acesso das informações e sua visualização filtrando apenas as informações principais a serem repassadas com o auxilio de medidas de variações relacionadas a pontuação atual e histórica.

A ideia do projeto surgiu ao reparar as múltiplas etapas *~~desnecessárias~~* realizadas pela empresa anterior que trabalhava para a apresentação desses dados aos respectivos superiores. Você pode conferir as etapas e as vantagens que o painel apresenta aqui 👉 [vantagens](https://github.com/Marcos-Beck/Dashboard_ICIA/new/main?filename=README.md#vantagens-do-painel).

## Dados 🎲
Os dados são reais e disponibilizados pelo [Instituto Aço Brasil](https://www.acobrasil.org.br/) mensalmente desde Abril de 2019 através do arquivo de relatório da série histórica no formato .xlsx.
> Curiosidade: o link do arquivo de histórico sofre alterações conforme o mês/ano publicado, provável que seja um arquivo totalmente novo a cada mês já que nos testes os links de datas passadas ainda funcionavam perfeitamente. 
>
> Por conta disso precisei utilizar variáveis na fonte para sempre retornar o arquivo da data atual.

O relatório é atualizado via Power Automate já que o serviço do Power BI não permite agendar atualizações mensais.

Fluxo Power Automate:
~~~~ 
Gatilho:
{
  "type": "Recurrence",
  "recurrence": {
    "interval": 1,
    "frequency": "Month",
    "timeZone": "E. South America Standard Time",
    "startTime": "2025-12-01T08:00:00"
  },
  "conditions": []
} 

Ação:
{
  "type": "OpenApiConnection",
  "inputs": {
    "parameters": {
      "groupid": "myworkspace",
      "datasetid": "f3e8d15a-8cdc-4d00-90ea-511057a476de"
    },
    "host": {
      "apiId": "/providers/Microsoft.PowerApps/apis/shared_powerbi",
      "connection": "shared_powerbi",
      "operationId": "RefreshDataset"
    }
  },
  "runAfter": {}
}

~~~~

## Visualizações 📊
1. Home 
    1. Card Ano: Apresenta a média (pts) do ano atual juntamente com as variações (%) e (pts) em relação ao indicador mínimo de satisfação (50) e ao ano anterior. 
    2. Card Mês: Apresenta o valor (pts) do mês atual juntamente com as variações (%) e (pts) em relação ao indicador mínimo de satisfação (50) e ao mês anterior.
    3. Comentário: Resume de forma dinâmica e direta o desempenho do mês atual.
    4. ICIA - Últimos 6 m: Apresenta os valores dos últimos 6 meses anterior ao mês atual em gráfico de linha com shade area para o indicador minimo de satisfação trazendo melhor perspectiva do cenário. 
    5. Última atualização: Deixa claro ao usuário quando foi depositado o último conjunto de dados do histórico.
2. Charts: 
    1. Card Único:Apresenta as informações principais relacionada aos gráficos, como a média do ano, média movél de 6 meses, o mês, o maior e menor registro conforme o periodo selecionado. 
    2. Índice de Confiança da Indústria do Aço: Apresenta os valores históricos do ICIA conforme o periodo selecionado em gráfico de linha com shade area para o indicador minimo de satisfação trazendo melhor perspectiva do cenário. 
    3. Média Ano: Apresenta uma visão geral conforme os anos em gráfico de barras.
    4. Filtros: Ano e mês para auxiliar na seleção do periodo desejado, modelo dropdown. 
3. Tooltips: 
    1. Índice de Confiança da Indústria do Aço: Apresenta os detalhes necessários para uma analise mais detalhada do mês em questão; ano, nome do mês, pontuação 00.00, variação (%) e (pts) MoM.
    2. Média Ano: Apresenta os detalhes necessários para uma analise mais detalhada do ano em questão; ano, pontuação média 00.00, variação (%) e (pts) YoY.

## Medidas Principais/Interessantes 🧮
**1. Calendário:** Cria a tabela calendário com base na menor e maior data presente na tabela F_ICIA - Histórico 
~~~~

D_CALENDARIO = 
    CALENDAR(   
        MIN('F_ICIA - Histórico'[Data]), 
        MAX('F_ICIA - Histórico'[Data])
        )
~~~~
**2. Comentário:** Gera um texto automático conforme o desempenho do mês atual; identifica alta, estabilidade ou baixa comparando os meses; verifica se o índice está acima ou abaixo do mínimo aceitável e calcula há quantos meses isso ocorre de forma consecutiva. 
~~~~
M_Comentario_ICIA = 
VAR MesAtual =
    CALCULATE(
        [M_ICIA],
        FILTER(
            ALL(D_CALENDARIO),
            D_CALENDARIO[Date] = MAX(D_CALENDARIO[Date])
        )
    )
VAR MesAnterior =
    CALCULATE(
        [M_ICIA],
        DATEADD(D_CALENDARIO[Date], -1, MONTH)
    )
VAR MinAceitavel = [M_Min_Indicador]
VAR VariacaoMoM = FORMAT([M_VAR%_MoM], "0.0%")
VAR Tendencia =
    SWITCH(
        TRUE(),
        MesAtual > MesAnterior, "leve alta",
        MesAtual < MesAnterior, "queda",
        "estabilidade"
    )
VAR EstaAcima = MesAtual >= MinAceitavel
VAR _LastDate =
    CALCULATE(
        MAX(D_CALENDARIO[Date]),
        REMOVEFILTERS(D_CALENDARIO)
    )
VAR TabResumo =
    ADDCOLUMNS(
        SUMMARIZE(
            FILTER(
                ALL(D_CALENDARIO),
                NOT(ISBLANK([M_ICIA])) &&
                D_CALENDARIO[Date] <= _LastDate
            ),
            D_CALENDARIO[Date]
        ),
        "ICIA", [M_ICIA]
    )
VAR LastBelowDate =
    MAXX(
        FILTER(TabResumo, [ICIA] < MinAceitavel),
        D_CALENDARIO[Date]
    )
VAR LastAboveDate =
    MAXX(
        FILTER(TabResumo, [ICIA] >= MinAceitavel),
        D_CALENDARIO[Date]
    )
VAR MesesSeguidos =
    IF(
        EstaAcima,
        COUNTROWS(
            FILTER(
                TabResumo,
                D_CALENDARIO[Date] > COALESCE(LastBelowDate, MINX(TabResumo, D_CALENDARIO[Date])) &&
                [ICIA] >= MinAceitavel
            )
        ),
        COUNTROWS(
            FILTER(
                TabResumo,
                D_CALENDARIO[Date] > COALESCE(LastAboveDate, MINX(TabResumo, D_CALENDARIO[Date])) &&
                [ICIA] < MinAceitavel
            )
        )
    )
VAR MesNome = FORMAT(EDATE(_LastDate, 0), "mmmm")
VAR AnoNome = FORMAT(EDATE(_LastDate, -1), "yyyy")
VAR TextoMeses =
    MesesSeguidos & " " &
    IF(MesesSeguidos = 1, "mês consecutivo", "meses consecutivos")
RETURN
"Em " & MesNome & " de " & AnoNome & ", o ICIA apresentou " &
Tendencia & " de " & VariacaoMoM & ", permanecendo " &
IF(EstaAcima, "acima", "abaixo") &
" do mínimo aceitável por " & TextoMeses & "."
~~~~
**3. Última atualicação:** valida a data mais recente entre a tabela D_Calendario e a tabela F_ICIA - Historico e retorna texto.
~~~~
M_Ultima_Att = 
VAR DataAtt =
    MINX(
        {
            MAX(D_CALENDARIO[Date]),
            MAX('F_ICIA - Histórico'[Data])
        },
        [Value]
    )
RETURN "Base ICIA - Última atualização: " & Format(DataAtt, "dd/mm/yyyy")
~~~~
**4. Maior valor do periodo (mês):** resume os valores mensais, encontra o mês com maior soma do indicador e retorna o nome formatado (mesma coisa para o menor mês apenas alterando DESC para ASC)
~~~~
M_Maior_Valor_MES = 
VAR TabResumo =
    CALCULATETABLE(
        SUMMARIZE(
            'F_ICIA - Histórico',
            'F_ICIA - Histórico'[Data],
            "PontuacaoTotal", SUM('F_ICIA - Histórico'[ICIA])
        ),
        REMOVEFILTERS(D_CALENDARIO[Date].[Mês])
    )
VAR LinhaMax =
    TOPN(1, TabResumo, [PontuacaoTotal], DESC)
VAR DataMax =
    SELECTCOLUMNS(LinhaMax, "Data", 'F_ICIA - Histórico'[Data])
RETURN
IFERROR(CONCATENATEX(DataMax, UPPER(FORMAT(DataMax, "mmmm yyyy"))),BLANK())
~~~~
**5. Maior valor do periodo (pts):** calcula a soma do indicador por mês e identifica o valor máximo encontrado no histórico (mesma coisa para o menor mês apenas alterando MAXX para MINX)
~~~~
M_Maior_Valor_PTS = 
CALCULATE(
    MAXX(
        SUMMARIZE('F_ICIA - Histórico', 'F_ICIA - Histórico'[Data], "PontuaçãoTotal", SUM('F_ICIA - Histórico'[ICIA])),
        [PontuaçãoTotal]
    ),
    REMOVEFILTERS(D_CALENDARIO[Date].[Mês])
)
~~~~
## Tecnologias Usadas ⚙️
Power BI, Power Query, Power Automate, DAX
## Possiveis Melhorias 🎯
Adicionar detalhes ao comentário HOME

Polir as médidas DAX futuramente

Implementar Forecast

## Vantagens do Painel 🚀
### Passos para acessar informações 
**Sem painel:**

1. Entrar no navegador
2. Abrir o site
3. Baixar relatório do mês (PDF)
4. Abrir Power Point
5. Abrir relátorio
6. Copiar gráfico
7. Colar gráfico
8. Copiar informações adicionais
9. Colar informações adicionais

**Com painel:** 

1. Abrir Power Point
2. Colar Relátorio (com suplemento do PBI)
---
### Pontos negativos dos relátorios originais
**Gráfico:**
* Muito carregado de informações
* Má formatação
* Apresenta dificuldade de leitura
* Noção de tempo distorcida
* Não possibilita filtragem de período

**PDF:**
* Muito conteúdo para análises rápidas

**Planilha:**
* Serve somente como fonte
* Posicionamento de coluna e linha inverso
