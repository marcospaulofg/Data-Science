# MVP - Engenharia de Dados
## Análise de índices de mortalidade no Brasil

# 1️⃣ Objetivo
_____

Este MVP tem como objetivo construir um pipeline de dados na nuvem para analisar dados de mortalidade no Brasil, utilizando tecnologias em nuvem com Databricks e seu Delta Lake. O pipeline envolverá as etapas de busca, coleta, modelagem, carga e análise dos dados, com o propósito de fornecer insights sobre padrões de mortalidade no país.

O problema central que este MVP busca resolver é a falta de uma visão consolidada e acessível dos dados de mortalidade, que permita identificar tendências, anomalias e fatores relevantes para a saúde pública. Para isso, serão respondidas as seguintes perguntas:

1)  Qual o número total de óbitos registrados no Brasil ao longo do tempo?

2)  Quais são as principais causas de morte no Brasil? 

3)  Qual a distribuição da mortalidade por faixa etária? 

4)  Quais são as causas de morte mais comuns entre crianças, adultos e idosos?

5)  O sexo das vítimas influencia as causas da morte?

6)  Houve mudanças nas principal causa de morte ao longo do tempo? 

7)  Qual é a proporção de óbitos que ocorreram em hospitais versus outros locais? 

8)  Qual é a distribuição de óbitos por raça/cor? 

9)  O estado civil tem relação com a mortalidade? 

10) Quais locais apresentam as maiores taxas de mortalidade? 

11) Em quais horários ocorrem mais mortes? 

12) A taxa de homicídios aumentou ou diminuiu ao longo dos anos? 

13) Quais regiões apresentam maior incidência de suicídios?

14) Qual é a distribuição de óbitos por causas relacionadas ao trabalho?



Ao final do projeto, espera-se entregar uma base de dados confiável com análises que contribuam para a compreensão dos fatores que impactam a mortalidade no Brasil.

# 2️⃣ Fonte dos Dados e Coleta
___


Os dados utilizados neste projeto foram obtidos de fontes oficiais e públicas, portanto há problemas com a confidencialidade destes dados. A base principal é o Sistema de Informações sobre Mortalidade (SIM), que contém registros detalhados sobre óbitos ocorridos no Brasil. Foram coletados os dados entre os anos de 2006 e 2024.

2.1 Tabela Fato – Mortalidade Geral

A tabela fato do projeto, denominada mortalidade_geral, foi construída a partir dos arquivos de mortalidade geral disponibilizados anualmente no portal de dados abertos do SUS:

🔗 Fonte: Sistema de Informações sobre Mortalidade (SIM) – OpenDataSUS (https://opendatasus.saude.gov.br/dataset/sim)

Os arquivos foram baixados manualmente, ano a ano, no formato CSV, e posteriormente feito o upload no DBFS do Databricks.

2.2 Tabelas Dimensão

As tabelas dimensão foram obtidas de diferentes fontes, conforme descrito abaixo:

- CID-10 (Classificação Internacional de Doenças – 10ª Revisão)
Para obter a descrição das causas de morte, foi utilizada a tabela de subcategorias da CID-10, versão 2008, extraída de um arquivo ZIP disponível no portal do Datasus:

  🔗 Fonte: CID-10 – Subcategorias (http://www2.datasus.gov.br/cid10/V2008/descrcsv.htm)

  O arquivo extraído foi o CID-10-SUBCATEGORIAS.CSV, que contém as descrições das subcategorias e categorias da CID-10. Esse conjunto de dados foi escolhido porque apresenta os códigos padronizados utilizados para registrar as causas de óbitos no Brasil.

- Divisão Territorial Brasileira
Para relacionar os óbitos às respectivas localidades, foi utilizada a tabela da Divisão Territorial Brasileira 2023, disponibilizada pelo IBGE:

  🔗 Fonte: Divisão Territorial Brasileira – IBGE (https://www.ibge.gov.br/geociencias/organizacao-do-territorio/divisao-regional/23701-divisao-territorial-brasileira.html)

  O arquivo extraído do ZIP foi RELATORIO_DTB_BRASIL_MUNICIPIO.csv, que contém informações sobre regiões geográficas intermediárias e imediatas, municípios, distritos e seus respectivos códigos.

- Demais tabelas dimensão

  As demais tabelas dimensão foram criadas manualmente, pois possuem poucas colunas (apenas 2) e linhas (a maior delas possui apenas 7), tornando viável sua construção sem a necessidade de fontes externas. Foram criadas no formato CSV, separadas por ";", utilizando um editor de texto. Essas tabelas foram baseadas diretamente nas informações contidas na tabela fato mortalidade_geral e incluem:

  - Circunstância (circunstancia)

  - Raça/Cor (cor)

  - Estado Civil (estado_civil)

  - Local do Óbito (local_obito)

  - Sexo (sexo)

  Essas tabelas foram projetadas para servir de referência às respectivas colunas na base de mortalidade geral, garantindo a integridade dos dados no processo analítico.

# 3️⃣ Modelagem e Catálogo de Dados
___

Para estruturar e organizar os dados de forma eficiente, foi adotado o Esquema Estrela, um dos modelos mais utilizados em Data Warehousing e Business Intelligence.

## 3.1 Estrutura do Esquema Estrela


O esquema estrela do projeto foi construído com uma tabela fato principal contendo os registros de mortalidade e 7 tabelas dimensão para compor as análises. A estrutura ficou organizada da seguinte forma:

 📊 Tabela Fato: mortalidade_geral
  
  - Esta tabela contém os registros de óbitos e é o núcleo central do esquema. Cada linha representa um óbito registrado, com detalhes como data, local, causa da morte e características da pessoa falecida.

  📊 Tabelas Dimensão:

  - Foram criadas tabelas auxiliares para armazenar descrições de variáveis categóricas e facilitar a análise por meio de junções (joins) entre as tabelas.

## 3.2 Catálogo de Dados


Tabela `mortalidade_geral`:

A tabela fato foi criada a partir da base original do SIM, que continha 132 colunas. Durante o processo de ETL, foram removidos campos irrelevantes, resultando em uma estrutura mais enxuta e otimizada para análise. São eles:

`ACIDTRAB`:
- Descrição: Indica se o evento que desencadeou o óbito está relacionado ao processo de trabalho.
- Datatype: string.
- Tamanho : 8.
- Valores possíveis: Sim; Não; ignorado.

`ASSISTMED`:
- Descrição: Se refere ao atendimento médico continuado que o paciente recebeu, ou não, durante a enfermidade que ocasionou o óbito.
- Datatype: string.
- Tamanho: 8.
- Valores possíveis: Sim; Não; Ignorado.

`CAUSABAS`:
  - Descrição: Código da causa da morte na declaração de óbito.
  - Datatype: string.
  - Tamanho: de 3 a 4.
  - Valores possíveis: códigos CID-10, onde o primeiro caracter é uma letra e o restante um número, por ex: 'A012', 'T71' e 'U049'.
  - FK de `SUBCAT` da tabela `cid_10`.

`CIRCOBITO`:
- Descrição: Código do tipo de morte violenta ou circunstâncias em que se deu a morte não natural.
- Datatype: string.
- Tamanho: 1.
- Valores possíveis: 1; 2; 3; 4; 9.
- FK de `COD_CIRC` da tabela `circunstancia`.

`CODMUNOCOR`:
- Descrição: Código relativo ao município onde ocorreu o óbito.
- Datatype: string.
- Tamanho: 6.
- Valores possíveis: códigos formados por números, ex: '110010', '110037' e '110020'
- FK de `CODMUN` da tabela `municipios`.

`DTOBITO`:
- Descrição: Data em que occoreu o óbito.
- Datatype: date.
- Tamanho: 10.
- Valores possíveis: qualquer data no padrão 'yyyy-mm-dd', por ex: '2023-07-15'.

`ESTCIV`:
- Descrição: Código da situação  conjugal  do  falecido  informada  pelos  familiares.
- Datatype: string.
- Tamanho: 1.
- Valores possíveis: 1; 2; 3; 4; 5; 9.
- FK de `COD_ESTCIVIL` da tabela `estado_civil`.

`HORAOBITO`:
- Descrição: Horário do óbito.
- Datatype: string.
- Tamanho: 5.
- Valores possíveis: qualquer hora no padrão de 00(:00) a 23(:59), por ex: '08:45'.

Idade do falecido em minutos, horas, dias, meses ou anos. Composto de duas colunas: 

`IDADE_GRUPO`:
- Descrição: Indica a unidade da idade do falecido em minutos, horas, dias, meses ou anos:<br>
  se 0 = minuto;<br>
  se 1 = hora,<br>
  se 2 = dia;<br>
  se 3= mês;<br>
  se 4 = ano;<br>
  se 5 = idade maior que 100 anos;<br>
  se 9 = ignorado.
- Datatype: string.
- Tamanho: 1
- Valores possíveis: 0; 1; 2; 3; 4; 5; 9.

`IDADE_QTD`:
- Descrição: Indica a quantidade de unidades da idade do falecido:<br>
  Idade menor de 1 hora: campo varia de  01 a 59  (minutos);<br>
  De 1 a 23 horas: campo varia de 01 a 23 (horas);<br>
  De 24 horas a 29 dias: campo varia de 01 a 29 (dias);<br>
  De 1 mês a menos de 12 meses completos: campo varia de 01 a 11 (meses);<br>
  Anos: campo varia de 00 a 99;<br>
  Maior que 100: campo varia de 00 a 50;<br>
  Ignorado: 99.
- Datatype: string.
- Tamanho: 2
- Valores possíveis: de 00 a 99.

`LOCOCOR`:
- Descrição: Código do local  de  ocorrência  do  óbito.
- Datatype: string.
- Tamanho: 1
- Valores possíveis: 1; 2; 3; 4; 5; 6; 9.
- FK de `COD_LOCAL` da tabela `local_obito`.

`RACACOR`:
- Descrição: Código da cor informada pelo responsável pelas informações do falecido.
- Datatype: string.
- Tamanho: 1
- Valores possíveis: 1; 2; 3; 4; 5; 9.
- FK de `COD_COR` da tabela `cor`.

`SEXO`:
- Descrição: Código do sexo do falecido.
- Datatype: string.
- Tamanho: 1
- Valores possíveis: 1; 2; 9.
- FK de `COD_SEXO` da tabela `sexo`.


Tabela `cid_10`:

Relaciona os códigos da CID-10 com a descrição das doenças e causas de mortalidade. Sua colunas são:

`SUBCAT` (PK):
- Descrição: Código da causa da morte na declaração de óbito.
- Datatype: string.
- Tamanho: de 3 a 4.
- Valores possíveis: códigos CID-10, onde o primeiro caracter é uma letra e o restante um número, por ex: 'A012', 'T71' e 'U049'.
- Relacionamento: `mortalidade_geral.CAUSABAS = cid_10.SUBCAT`

`DESCRICAO`: 
- Descrição: Descrição da causa da morte na declaração de óbito.
- Datatype: string.
- Tamanho: de 5 a 114.
- Valores possíveis: 'Cólera devida a Vibrio cholerae 01, biótipo cholerae', 'Febre paratifóide A' e 'Botulismo'.


Tabela `circunstancia`:

Relaciona o código do tipo de morte com sua descrição. Suas colunas são:

`COD_CIRC` (PK):
- Descrição: Código do tipo de morte violenta ou circunstâncias em que se deu a morte não natural.
- Datatype: string
- Tamanho: 1
- Valores possíveis: 1; 2; 3; 4; 9.
- Relacionamento: `mortalidade_geral.CIRCOBITO = circunstancia.COD_CIRC`

`DESCR_CIRC`:
- Descrição: Descrição do tipo de morte.
- Datatype: string.
- Tamanho: de 6 a 9.
- Valores possíveis: acidente, suicídio, homicídio, outras circunstâncias ou ignorado.

%md
Tabela `municipios`:

Contém os códigos e nomes dos municípios e estados do Brasil. Suas colunas são:

`CODMUN` (PK):
- Descrição: Código relativo ao município onde ocorreu o óbito.
- Datatype: string
- Tamanho: 6
- Valores possíveis: códigos formados por números, ex: '110010', '110037' e '110020'.
- Relacionamento: `mortalidade_geral.CODMUNOCOR = municipios.CODMUN`

`MUNICIPIO`:
- Descrição: Nome do município onde ocorreu o óbito.
- Datatype: string
- Tamanho: de 3 a 32
- Valores possíveis: ex: Guajará-Mirim, Alto Alegre dos Parecis, Rio de Janeiro.

`UF`:
- Descrição: Código do estado onde ocorreu o óbito.
- Datatype: string
- Tamanho: 2
- Valores possíveis: ex: RJ, SP, MG.


Tabela `estado_civil`:

Armazena o código e sua descrição de estado civil. Suas colunas são:

`COD_ESTCIVIL` (PK):
- Descrição: Código da situação conjugal do falecido informada pelos familiares.
- Datatype: string.
- Tamanho: 1
- Valores possíveis: 1; 2; 3; 4; 5; 9.
- Relacionamento: `mortalidade_geral.ESTCIV = estado_civil.COD_ESTCIVIL`


`DESCR_ESTADO`:
- Descrição: Descrição da situação conjugal do falecido informada pelos familiares.
- Datatype: string.
- Tamanho: de 5 a 33.
- Valores possíveis: Solteiro; Casado; Viúvo; Separado judicialmente/divorciado; União estável; Ignorado.


Tabela `sexo`:

Relaciona o sexo do falecido com seu código. Suas colunas são:

`COD_SEXO` (PK):
- Descrição: Código do sexo do falecido.
- Datatype: string.
- Tamanho: 1
- Valores possíveis: 1; 2; 9.
- Relacionamento: `mortalidade_geral.SEXO = sexo.COD_SEXO`

`DESCR_SEXO`:

- Descrição: Sexo do falecido.
- Datatype: string.
- Tamanho: de 8 a 9.
- Valores possíveis: Masculino; Feminino Ignorado.


Tabela `cor`:

Relaciona a raça/cor do falecido com seu código. Suas colunas são:

`COD_COR` (PK):
- Descrição: Código da cor informada pelo responsável pelas informações do falecido.
- Datatype: string.
- Tamanho: 1
- Valores possíveis: 1; 2; 3; 4; 5; 9.
- Relacionamento: `mortalidade_geral.RACACOR = cor.COD_COR`

`DESCR_COR`:
- Descrição: Cor/raça do falecido.
- Datatype: string.
- Tamanho: de 5 a 9.
- Valores possíveis: Branca; Preta; Amarela; Parda; Indígena.


Tabela `local_obito`:

Relaciona o código do local de óbito com sua descrição. Suas colunas são:

`COD_LOCAL` (PK):
- Descrição: Código do local de ocorrência do óbito.
- Datatype: string.
- Tamanho: 1.
- Valores possíveis: 1; 2; 3; 4; 5; 6; 9.
- Relacionamento: `mortalidade_geral.LOCOCOR = local_obito.COD_LOCAL`

`DESCR_LOCAL`:
- Descrição: Descrição do local de óbito.
- Datatype: string.
- Tamanho: de 6 a 32.
- Valores possíveis: Hospital; Outros estabelecimentos de saúde; Domicílio; Via pública; Outros; Aldeia indígena; Ignorado.

