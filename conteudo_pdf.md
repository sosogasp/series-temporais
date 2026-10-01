TRABALHO - SÉRIES TEMPORAIS 
Página 1 
Modelagem Comparativa de Séries Temporais 
com Variáveis Externas 
1. Finalidade 
Cada grupo deverá investigar cinco bases comuns, aplicar modelos estatísticos e de Machine Learning e 
produzir uma comparação crítica dos resultados obtidos. 
2. Objetivos 
• Documentar e explorar séries temporais com variáveis externas. 
• Aplicar decomposição STL e interpretar tendência, sazonalidade e componente residual. 
• Realizar Feature Engineering sem utilizar informações indisponíveis na data da previsão. 
• Ajustar e comparar SARIMAX, Holt-Winters, Random Forest e um modelo adicional. 
• Pesquisar e aplicar estratégias de otimização de hiperparâmetros. 
• Avaliar os modelos por MAE utilizando validação walk-forward. 
• Analisar os resíduos por métodos estatísticos e gráficos. 
• Investigar a importância das features por um método compatível com cada modelo. 
• Comunicar os resultados em relatório técnico, apresentação oral e registro de gestão do grupo. 
3. Organização dos grupos e modelos 
A turma será organizada em cinco grupos. Todos utilizarão as mesmas cinco bases e executarão os três 
modelos comuns, além de um modelo de especialização exclusivo. Cada grupo realizará 20 combinações 
principais: cinco bases multiplicadas por quatro modelos. 
Grupo Modelo de especialização Família 
1 XGBoost Regressor Boosting baseado em árvores 
2 Support Vector Regression (SVR) Regressão baseada em margem 
3 Elastic Net Regressão linear regularizada 
4 MLP Regressor Rede neural artificial 
5 PLS Regression Regressão por componentes supervisionados 
 
Os três modelos comuns a todos os grupos serão SARIMAX, Holt-Winters e Random Forest. O Holt-
Winters deverá ser tratado como referência univariada, pois não utiliza variáveis externas. SARIMAX, 
Random Forest e o modelo de especialização deverão incorporar as variáveis externas quando forem 
compatíveis com o algoritmo. 
TRABALHO - SÉRIES TEMPORAIS 
Página 2 
4. Requisitos das bases de dados 
As cinco bases serão comuns a toda a turma. Cada base deverá ser congelada antes do início da 
modelagem para que todos trabalhem com a mesma versão dos dados. 
• Possuir uma variável-alvo numérica organizada no tempo. 
• Apresentar frequência temporal regular ou ser adequadamente regularizada pelo grupo. 
• Conter quantidade suficiente de observações e ciclos sazonais para análise. 
• Possuir pelo menos duas variáveis externas potencialmente relacionadas ao alvo. 
• Apresentar fonte, período de cobertura, unidade, frequência e significado das variáveis. 
• Permitir a definição clara do horizonte de previsão e da data de origem das previsões. 
4.1 Disponibilidade das variáveis externas 
Para cada variável externa, o grupo deverá explicar se ela estaria disponível no momento real da 
previsão. Valores futuros observados não poderão ser utilizados como entrada, salvo quando forem 
efetivamente conhecidos com antecedência. Quando a variável não for conhecida no futuro, o grupo 
deverá utilizar valores defasados, uma previsão disponível na data de origem ou justificar sua exclusão. 
Exemplo de variável Disponibilidade para uma previsão futura 
Dia da semana e feriados Conhecidos antecipadamente 
Promoção ou preço previamente planejado Conhecidos antecipadamente 
Temperatura realmente observada no 
futuro Indisponível; causaria vazamento 
Previsão meteorológica disponível na 
origem Pode ser utilizada e deve ser identificada como previsão 
Vendas futuras Indisponíveis; não podem ser utilizadas como feature 
 
  
TRABALHO - SÉRIES TEMPORAIS 
Página 3 
5. Etapas obrigatórias 
5.1 Documentação e exploração das cinco bases 
• Apresentar fonte, descrição, período, frequência e unidade da variável-alvo. 
• Construir um dicionário das variáveis externas. 
• Verificar valores ausentes, duplicidades, irregularidades temporais e valores atípicos. 
• Apresentar gráficos da série-alvo e das principais variáveis externas. 
• Descrever as decisões de limpeza e transformação. 
5.2 Decomposição e características temporais 
• Realizar a decomposição STL da variável-alvo de cada base. 
• Apresentar e interpretar visualmente o componente de tendência, indicando períodos de 
crescimento, queda, estabilidade e mudanças de direção. 
• Apresentar e interpretar o componente sazonal e o componente residual. 
• Calcular e interpretar a força da sazonalidade conforme o método apresentado em sala. 
5.3 Feature Engineering 
Os modelos baseados em tabela deverão utilizar um conjunto de features construído de maneira 
coerente com o horizonte de previsão. As mesmas features deverão ser utilizadas no Random Forest e 
no modelo de especialização sempre que houver compatibilidade, evitando que a comparação seja 
determinada por conjuntos de dados diferentes. 
• Lags da variável-alvo e, quando pertinente, das variáveis externas. 
• Estatísticas de janelas móveis, como média e desvio padrão. 
• Features de calendário e encoding cíclico. 
• Novas features que o grupo julgar necessárias e importantes. 
• Variáveis externas disponíveis na data da previsão. 
• Tratamento documentado dos valores ausentes produzidos pelos lags e janelas. 
TRABALHO - SÉRIES TEMPORAIS 
Página 4 
5.4 Validação walk-forward 
A validação walk-forward simula o funcionamento real de um modelo de previsão ao longo do tempo. O 
processo começa com uma janela inicial de treinamento, utilizando somente as observações disponíveis 
até aquele momento. O modelo produz uma previsão para o horizonte definido; em seguida, o tempo 
avança, novas observações reais são incorporadas ao histórico e uma nova previsão é realizada. Esse 
procedimento é repetido até o fim do período de teste, gerando uma sequência de previsões fora da 
amostra. Todos os modelos deverão utilizar as mesmas datas de origem, o mesmo horizonte e o mesmo 
conjunto de teste. Os grupos serão responsáveis por pesquisar e implementar esse método, mantendo 
fixos, durante o teste final, os hiperparâmetros previamente selecionados. 
5.5 Otimização dos hiperparâmetros 
Cada grupo deverá pesquisar como realizar a otimização dos quatro modelos utilizados em cada base. A 
seleção dos hiperparâmetros deverá ocorrer antes da avaliação final, sem utilizar o conjunto de teste 
para tomar decisões. O relatório deverá informar o espaço de busca, o procedimento adotado, os 
valores selecionados e a justificativa das escolhas. 
Modelo Elementos mínimos a investigar 
SARIMAX Ordens p, d, q; ordens sazonais P, D, Q; período m; AIC/BIC; variáveis externas 
Holt-Winters Tendência; sazonalidade; tendência amortecida; período sazonal; parâmetros de suavização 
Random Forest n_estimators; max_depth; min_samples_split; min_samples_leaf; max_features 
Modelo escolhido Principais hiperparâmetros, efeitos, intervalos testados, método de busca e configuração final 
 
TRABALHO - SÉRIES TEMPORAIS 
Página 5 
5.6 Treinamento e previsão 
• Executar os quatro modelos em cada uma das cinco bases. 
• Utilizar as mesmas origens de previsão e o mesmo horizonte em todos os modelos. 
• Impedir que observações futuras sejam utilizadas durante treinamento, criação de features ou 
otimização. 
• Registrar parâmetros, tempo de execução, previsões e resíduos de cada combinação. 
6. Avaliação comparativa por MAE 
O Mean Absolute Error (MAE) será a métrica principal de comparação. O MAE deverá ser calculado 
sobre as previsões fora da amostra produzidas pelo walk-forward. Como as bases podem apresentar 
escalas diferentes, os valores de MAE não deverão ser somados nem calculados em média diretamente 
entre bases. 
• Apresentar o MAE dos quatro modelos em cada base. 
• Classificar os modelos do menor para o maior MAE dentro de cada base. 
• Identificar o melhor modelo em cada base. 
• Calcular a quantidade de vitórias e a posição média de cada modelo. 
• Discutir em quais características das bases cada modelo apresentou melhor ou pior desempenho. 
TRABALHO - SÉRIES TEMPORAIS 
Página 6 
7. Análise dos resíduos 
A análise deverá utilizar os erros fora da amostra, definidos pela diferença entre os valores reais e as 
previsões. Para cada combinação entre base e modelo, deverão ser apresentados métodos estatísticos e 
gráficos. 
• Gráfico dos resíduos ao longo do tempo. 
• Gráfico da função de autocorrelação (ACF) dos resíduos. 
• Teste de Ljung-Box e interpretação do p-valor. 
• Discussão sobre viés, padrões remanescentes, variabilidade e autocorrelação. 
A tabela consolidada com os resultados do Ljung-Box deverá aparecer no corpo principal do relatório. 
Gráficos complementares poderão ser organizados no apêndice, desde que os resultados mais 
relevantes sejam discutidos no texto principal. 
TRABALHO - SÉRIES TEMPORAIS 
Página 7 
8. Importância das features e interpretação 
A importância das features deverá ser analisada no Random Forest e no modelo de especialização. 
Quando o modelo não possuir uma medida nativa, deverá ser empregado um método compatível, como 
Permutation Importance. As variáveis externas deverão ser destacadas e sua disponibilidade temporal 
deverá ser retomada na interpretação. 
Modelo Método de interpretação indicado 
SARIMAX Coeficientes das variáveis externas, sinal, magnitude e significância 
Holt-Winters Não possui feature importance; interpretar nível, tendência e sazonalidade 
Random Forest Importância nativa e/ou Permutation Importance 
XGBoost Gain, Permutation Importance ou SHAP 
SVR Permutation Importance ou método equivalente justificado 
Elastic Net Coeficientes após padronização 
MLP Regressor Permutation Importance ou método equivalente justificado 
PLS Regression Coeficientes, Permutation Importance ou VIP Scores 
 
TRABALHO - SÉRIES TEMPORAIS 
Página 8 
9. Estudo do modelo de especialização 
Além da implementação, cada grupo deverá produzir uma explicação técnica e didática do modelo de 
especialização. A seção deverá demonstrar domínio conceitual e não se limitar à descrição do código 
utilizado. 
• Funcionamento geral e intuição do algoritmo. 
• Principais hipóteses, vantagens, limitações e requisitos de preparação dos dados. 
• Papel e efeito dos principais hiperparâmetros. 
• Método adotado para otimização. 
• Método de importância das features e interpretação dos resultados. 
• Desempenho do modelo nas cinco bases e possíveis explicações para as diferenças. 
TRABALHO - SÉRIES TEMPORAIS 
Página 9 
10. Relatório técnico 
O grupo deverá produzir um relatório HTML paginado e uma versão final em PDF, ambos com o mesmo 
conteúdo. O documento deverá apresentar uma narrativa comparativa, explicar as decisões 
metodológicas e interpretar os resultados, evitando a simples reprodução de códigos e saídas. 
10.1 Estrutura mínima do relatório 
1. Resumo executivo. 
2. Identificação dos integrantes e divisão de responsabilidades. 
3. Documentação das cinco bases e das variáveis externas. 
4. Limpeza, preparação e Feature Engineering. 
5. STL, tendência e força da sazonalidade. 
6. Protocolo walk-forward. 
7. Modelos, hiperparâmetros e otimização. 
8. Resultados comparativos por MAE. 
9. Análise dos resíduos e teste de Ljung-Box. 
10. Importância das features. 
11. Explicação aprofundada do modelo escolhido. 
12. Conclusões, limitações e recomendações. 
13. Referências. 
14. Apêndices técnicos e registro de demandas. (Apenas na Entrega para o professor) 

14. Critérios de avaliação 
Critério Peso 
Documentação, qualidade e compreensão das cinco bases 5% 
STL, interpretação da tendência e força da sazonalidade 5% 
Feature Engineering, disponibilidade das variáveis e prevenção de vazamento 10% 
Implementação dos modelos e validação walk-forward 15% 
Pesquisa e otimização dos hiperparâmetros 10% 
Comparação dos resultados por MAE 10% 
Análise dos resíduos, ACF e Ljung-Box 5% 
Modelo escolhido e importância das features 20% 
Qualidade técnica do HTML e do PDF 10% 
Apresentação oral utilizando o relatório paginado 5% 
Gestão diária das demandas e participação dos integrantes 5% 
 
A nota individual poderá ser ajustada quando o registro de demandas, os arquivos entregues e a 
participação na apresentação indicarem diferenças relevantes de contribuição entre os integrantes. 
15. Regras gerais 
• Todos os resultados deverão ser reproduzíveis a partir dos arquivos entregues. 
• As fontes das bases, bibliotecas, referências técnicas e materiais consultados deverão ser citadas. 
• Os grupos deverão utilizar exatamente as versões congeladas das cinco bases. 
• Qualquer alteração de modelo, base, horizonte ou protocolo deverá ser previamente aprovada. 
• O uso de ferramentas de Inteligência Artificial deverá ser acompanhado de revisão crítica, 
compreensão do código e responsabilidade sobre os resultados apresentados. 