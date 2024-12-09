# **Relatório DAU**

---

## **1. Coleta e Tratamento dos Dados**

### **Leitura e Entendimento dos Dados**
Foram fornecidas quatro tabelas principais, as quais passaram por uma análise individual e posterior integração. A seguir, detalhamos os processos aplicados:

- **Tabela *daumau***:
  - Contém valores diários de DAU (*Daily Active Users*) e MAU (*Monthly Active Users*) para diferentes aplicativos.
  - **Deduplicação**: 153 linhas duplicadas foram removidas.
  - **Tratamento de valores nulos**:
    - 194 registros com valores nulos na coluna `dauReal` foram preenchidos com zero, assumindo que nesses dias nenhum usuário acessou o aplicativo.
    - Registros restantes com valores nulos foram descartados, pois representavam uma fração insignificante do dataset.

- **Tabela de Instalações**:
  - **Deduplicação**: 171 linhas duplicadas foram eliminadas.

Após o pré-processamento, os dados foram integrados em uma única tabela utilizando as colunas `appId` e `Date` como chaves primárias.

---

## **2. Análise Exploratória**

Durante esta etapa, foram realizadas ações específicas para limpeza e compreensão do dataset:

1. **Eliminação de Colunas Redundantes**:
   - As colunas `lang` e `country`, com valores únicos (respectivamente `pt` e `br`), foram descartadas por não agregarem valor à análise.

2. **Tratamento de Valores Nulos na Coluna *Category***:
   - Após a integração, alguns aplicativos apresentaram valores nulos em `category`. Esses valores foram preenchidos com base nos registros existentes de outras datas para os mesmos aplicativos.

3. **Identificação e Exclusão de Outliers de Datas**:
   - Datas anômalas (2220, 2044, 1912 e 1980) foram identificadas como outliers e descartadas.
   - Confirmou-se que todos os registros válidos datam de 2024.

4. **Remoção de Valores Inválidos**:
   - As colunas `daily_ratings` e `daily_reviews` apresentavam valores negativos, que foram removidos para garantir a integridade do dataset.

---

## **3. Modelagem e Treinamento**

Para modelar o problema, foi necessário adaptar o dataset, pois:

1. **Ordem Temporal**:
   - Os dados possuem uma ordem temporal intrínseca, sendo que colunas como `ratings` e `reviews` representam somas acumuladas de valores diários e históricos.

2. **Disponibilidade de Dados**:
   - Exceto pela coluna `category`, as demais informações refletem o dia atual e, portanto, não estão disponíveis no momento da previsão.

### **Transformações Realizadas**
- **Criação de Features Temporais**:
  - A variável-alvo `dauReal` foi transformada utilizando janelas temporais de sete dias. Essa abordagem considera os acessos dos últimos sete dias (excluindo o dia atual) para prever o próximo dia.
  - Foram criadas variáveis adicionais relacionadas a datas, como finais de semana e feriados, devido ao potencial impacto nos acessos.

### **Modelos Avaliados**
Os seguintes modelos foram testados utilizando validação cruzada para séries temporais:

1. **Regressão Linear Simples**:
   - Serviu como *baseline*, proporcionando uma métrica inicial de desempenho.

2. **Random Forest**:
   - Modelo robusto baseado em árvores de decisão. Obteve o melhor desempenho na validação cruzada com um **MAE de 70.371,58**.

3. **XGBoost**:
   - Modelo eficiente que utiliza técnicas de *boosting*, treinando modelos fracos de forma sequencial para formar um modelo mais forte.
   - Apesar de não atingir o menor MAE (**82.560,69**), foi escolhido devido à sua eficiência e melhor escalabilidade para grandes volumes de dados.

### **Escolha do Modelo**
Embora o **Random Forest** tenha obtido o menor MAE, o **XGBoost** foi escolhido pelo compromisso entre desempenho e eficiência computacional, além de apresentar uma métrica competitiva.

### **Otimização de Hiperparâmetros**
A tunagem foi realizada utilizando **Otimização Bayesiana**, escolhida por sua eficiência em encontrar os melhores parâmetros com menos avaliações de modelos, superando abordagens tradicionais como *Grid Search* e *Random Search*. O método utiliza distribuições probabilísticas para explorar o espaço de busca de forma mais inteligente.

Os hiperparâmetros obtidos foram:
```python
OrderedDict([
    ('colsample_bytree', 0.978),
    ('gamma', 7),
    ('learning_rate', 0.194),
    ('max_depth', 5),
    ('n_estimators', 145),
    ('reg_alpha', 0.211),
    ('reg_lambda', 6.11e-05),
    ('subsample', 0.685)
])
```

## **4. Resultados**

O modelo final obteve um MAE de 41.676,29, indicando boa generalização, embora ainda haja espaço para melhorias.

## **5. Melhorias**

Com mais tempo, algumas iniciativas poderiam melhorar o desempenho do modelo:

1. Construção de Novas Features:
   - Identificar variáveis adicionais que expliquem melhor o comportamento de dauReal.

2. Exploração de Diferentes Janelas Temporais:
   - Testar janelas temporais de tamanhos variados para encontrar a mais adequada.

3. Análise de Correlações:
   - Explorar melhor relações entre variáveis que possam revelar insights adicionais.
