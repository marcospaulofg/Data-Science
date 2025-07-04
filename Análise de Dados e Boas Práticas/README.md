# 1️⃣ Definição do problema

## 1.1 - Dataset

O dataset deste MVP será um conjunto de dados públicos de e-commerce brasileiro com pedidos feitos na Olist Store, a maior loja de departamentos do marketplace brasileiro. O conjunto de dados contém informações de aproximadamente 100 mil pedidos de 2016 a 2018, feitos em diversos marketplaces no Brasil.

## 1.2 - Descrição do problema

Visto que um dos principais desafios de qualquer comércio é garantir a satisfação do cliente, este projeto visa analisar os fatores que influenciam essa satisfação em compras online.

O problema central será preparar os dados para futuramente construir um modelo capaz de prever se um cliente ficará satisfeito (dando uma nota de avaliação alta), insatisfeito (dando uma nota baixa) ou indiferente (dando uma nota intermediária) com base nas características do seu pedido.

## 1.3 - Tipo de problema

Este é um problema de aprendizado supervisionado de classificação.

Supervisionado porque temos labels nos dados, que são as notas dadas pelos clientes, e classificação porque a variável alvo será transformada em categorias discretas (por exemplo, "satisfeito", "insatisfeito", "indiferente", etc).

## 1.4 - Hipóteses do problema

1. O tempo de entrega é o fator mais crítico para a satisfação do cliente. Entregas que ultrapassam a data estimada geram avaliações negativas.

2. O valor do frete, especialmente quando comparado ao valor do produto, influencia a percepção de valor e, consequentemente, a satisfação.

3. Certas categorias de produtos (ex: produtos considerados frágeis, eletrônicos) podem ser mais propensas a problemas e, portanto, a avaliações piores.

4. A relação entre o preço pago pelo produto e sua qualidade é diretamente proporcional. Se um produto for muito caro, mas não apresentar a qualidade condizente, pode gerar insatisfação. Porém um produto de baixa qualidade, mas que seja barato, pode não gerar tanta insatisfação, pois a falta de qualidade já seria esperada.

## 1.5 - Condições impostas para selecionar os dados

A análise está restrita aos dados coletados e disponibilizados pela Olist, cobrindo o período de setembro de 2016 a agosto de 2018. Não temos acesso informações sobre a comunicação entre cliente e vendedor, o que poderia ajudar a avaliar ainda mais os motivos de insatisfação.

## 1.6 - Atributos do dataset

Este dataset está dividido em 9 arquivos `.csv` que se relacionam. São eles:

- `olist_orders_dataset.csv`: Contém a informação central de cada pedido.
  - `order_id`: Identificador único do pedido.  
  - `customer_id`: Identificador único do cliente.  
  - `order_status`: Status do pedido (delivered, shipped, etc.).  
  - `order_purchase_timestamp`: Data e hora da compra.  
  - `order_approved_at`: Data e hora da aprovação do pagamento.  
  - `order_delivered_carrier_date`: Data e hora do envio à transportadora.  
  - `order_delivered_customer_date`: Data da entrega ao cliente.  
  - `order_estimated_delivery_date`: Data estimada da entrega.

- `olist_order_items_dataset.csv`: Contém os itens de cada pedido.  
  - `order_id`: Chave para conectar com a tabela de pedidos.  
  - `order_item_id`: Número sequencial que identifica o número de itens incluídos no mesmo pedido.  
  - `product_id`: Identificador do produto.  
  - `seller_id`: Identificador do vendedor.  
  - `shipping_limit_date`: Data limite de envio do vendedor.
  - `price`: Preço do produto.  
  - `freight_value`: Valor do frete (se um pedido tiver mais de um item o valor do frete será dividido entre os itens).  

- `olist_order_reviews_dataset.csv`: Contém as avaliações dos pedidos.  
  - `review_id`: Identificador único da avaliação.  
  - `order_id`: Chave para conectar com a tabela de pedidos.  
  - `review_score`: Nossa variável-alvo com notas de 1 a 5.  
  - `review_comment_title`: Título do comentário da avaliação deixada pelo cliente.  
  - `review_comment_message`: Mensagem de comentário da avaliação deixada pelo cliente.  
  - `review_creation_date`: Data em que a pesquisa de satisfação foi enviada ao cliente.  
  - `review_answer_timestamp`: Data e hora da resposta da pesquisa de satisfação.  

- `olist_products_dataset.csv`: Contém informações sobre os produtos.  
  - `product_id`: Chave para conectar com a tabela de itens.  
  - `product_category_name`: Categoria do produto.
  - `product_name_lenght`: Número de caracteres do nome do produto.  
  - `product_description_lenght`: Número de caracteres da descrição do produto.  
  - `product_photos_qty`: Número de fotos de produtos publicadas.  
  - `product_weight_g`: Peso do produto em gramas.  
  - `product_length_cm`: Comprimento do produto em centímetros.  
  - `product_height_cm`: Altura do produto em centímetros.  
  - `product_width_cm`: Largura do produto em centímetros.

- `olist_customers_dataset.csv`: Contém informações sobre clientes.
  - `customer_id`: Chave para conectar com a tabela de pedidos. Cada pedido possui um customer_id exclusivo.  
  - `customer_unique_id`: Identificador único do cliente.  
  - `customer_zip_code_prefix`: Primeiros cinco dígitos do CEP do cliente.  
  - `customer_city`: Cidade do cliente.  
  - `customer_state`: Estado do cliente.

- `olist_order_payments_dataset.csv`: Contém informações sobre pagamentos.
  - `order_id`: Identificador único do pedido.  
  - `payment_sequential`: Um cliente pode pagar um pedido com mais de um método de pagamento. Se o fizer, será criada uma sequência para acomodar todos os pagamentos.  
  - `payment_type`: Forma de pagamento.  
  - `payment_installments`: Número de parcelas.  
  - `payment_value`: Valor pago.  

Outras tabelas incluem dados de vendedores (`olist_sellers_dataset.csv`), tradução de nomes das categorias (`product_category_name_translation.csv`) e localizações geográficas (`olist_geolocation_dataset.csv`), porém essas não serão usadas em nossa análise. Também não utilizaremos todas as colunas, portanto mais adiante iremos removê-las após analisar quais são descartáveis para nosso projeto.

Segue um esquema fornecido pela própria Olist para identificação das relações entre as tabelas.

![Alt text](https://i.imgur.com/HRhd2Y0.png "a title")

# 2️⃣ Análise de dados

Vamos inicar o projeto entendendo as informações disponíveis no dataset. Para isso, faremos algumas análises básicas e estatísticas como:

- Quantos atributos e instâncias existem;
- Quais são os tipos de dados dos atributos;
- O que chama atenção ao verificar as primeiras linhas do dataset;
- Verificar valores nulos, duplicados, outliers ou inconsistentes;
- Resumo estatístico dos atributos numéricos (mínimo, máximo, mediana, média, desvio-padrão, primeiro quartil, terceiro quartil e moda).

- Visualizações:
  - Verificar a distribuição dos atributo com histogramas e gráficos de contagem;
  - Analisar a relação entre as variáveis preditoras e variável alvo com boxplots;
  - Verificar a correlação entre todas as variáveis com a matriz de correlação.

#### `Todo o código e visualizações desta seção estão dentro do notebook.`

 # 3️⃣ Pré-processamento de dados

Agora iremos realizar operações de limpeza, tratamento e preparação dos dados. Para isso faremos:

- Limpeza inicial dos dados;
- Tratamento de valores faltantes;
- Remover valores duplicados;
- Feature selection;
- Divisão do conjunto entre treino e teste;
- Técnicas de transformação como padronização e transformações logarítmicas

#### `Todo o código e visualizações desta seção estão dentro do notebook.`

# 4️⃣ Conclusão

## 4.1 - Revisão

O objetivo central deste projeto foi realizar uma análise exploratória e um pré-processamento completo em um dataset de e-commerce da Olist, com o intuito de preparar os dados para a futura tarefa de prever a satisfação do cliente, separados em três categorias ('Insatisfeito', 'Indiferente' e 'Satisfeito'), através do treinamento de algum modelo de machine learning.

Durante a análise exploratória dos dados pudemos evidenciar algumas características do dataset, como:

1. Desbalanceamento das classes, com a extrema maioria das avaliações sendo positivas (notas 4 e 5);
2. Forte assimetria à direita das variáveis numéricas, como preço dos produtos e valor do frete, indicando a presença de outliers;
3. A confirmação visual e estatística de que o tempo de entrega é um fator com forte correlação negativa com a satisfação do cliente, confirmando nossa hipótese 1 do começo do projeto.

Durante o pré-processamento realizamos algumas etapas, como:

1. Engenharia de atributos, para criar uma variável mais informativa relacionada ao tempo de entrega dos pedidos. Essa etapa, apesar de fazer parte de uma preparação dos dados, foi realizada antes mesmo de iniciarmos nossas análises, pois já previmos que seria uma variável interessante de estar presente nas análises estatísticas;
2. Aplicamos transformação logarítmica nas features assimétricas, normalizando a forma de suas distribuições e mitigando os efeitos dos outliers, tornando-as mais adequadas para algoritmos de machine learning;
3. Transformamos os dados usando One-Hot Encoding para as features categóricas e padronizamos com StandardScaler as variáveis numéricas para que todas ficassem na mesma escala. Todas essas transformações foram aplicadas da forma correta após a divisão dos dados em conjuntos de treino e teste e sempre se atentando para não ocorrer data leakage;
4. Aplicamos a técnica SMOTE no conjunto de treino, aumentando artificialmente a quantidade de registros das classes minoritárias, gerando um conjunto de dados de treinamento perfeitamente balanceado, evitando que o modelo treinado venha a sofrer overfitting para a classe 'Satisfeito'.
5. Além de todo o tratamento de valores nulos, duplicados e inconsistentes.

## 4.2 - Possíveis melhorias

Poderíamos adicionar algumas hipóteses iniciais, como:

1. Considerar que os atrasos no frete podem estar ligados a processos de logísticas complicados. Com essa ideia poderíamos verificar alguma relação entre o peso e dimensões do produto com o tempo de entrega, partindo do pressuposto que produtos grandes ou pesados podem ser mais difíceis de manusear durante o transporte.

2. Outra hipótese poderia ser sobre a avaliação do cliente estar vinculada a utilização de um cupom de desconto ou não. A ideia é semelhante à hipótese 4: no caso de receber um desconto, o cliente pode ter uma percepção de "bom negócio", tornando-o mais tolerante a pequenas falhas no processo, resultando em avaliações mais altas.

## 4.3 - Considerações finais

Algumas hipóteses iniciais que tivemos no início do projeto foram descartadas ao analisarmos as estatísticas e plotarmos alguns gráficos dos dados, porém é importante perceber que obtivemos essas respostas devido ao desbalanceamento dos dados. Talvez se o dataset não estivesse não enviesado para notas 5 estrelas, teríamos uma conclusão diferente na análise. Esse desbalanceamento foi o principal "vilão" em nossa análise.

Outro ponto que vale ressaltar é que talvez esse dataset não seja o mais apropriado para servir de base para o treinamento de um modelo que visa prever a satisfação do cliente. Porém só pudemos tirar essa conclusão após fazermos as análises de cada feature do dataset, onde vimos a correlação entre features e target muito baixa.

Vale ressaltar que é válido realizar o treinamento do modelo tanto com os dados balanceados (`X_train_balanced` e `y_train_balanced`) quanto com os dados desbalanceados (`X_train_processed` e `y_train`), para validarmos a eficácia da utlização do `SMOTE`.

Dessa forma, avalio que este projeto executou com sucesso todo o ciclo de vida da preparação de dados, entregando as análises necessárias e preparando os dados para a etapa seguinte, que seria treinar e avaliar diferentes algoritmos de classificação para, enfim, extrair valor preditivo dos dados.
