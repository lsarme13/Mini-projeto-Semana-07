# Sanitização das Bases de Dados da Olist
Nome: Luan Sarmento Orsi da Silva

Turma: T3 - Machine Learning e Visão COmputacional


## Descrição do Projeto



A Olist é uma plataforma brasileira de marketplace que conecta pequenos e médios lojistas a grandes canais de venda. A equipe de Engenharia de Dados extraiu dois lotes de dados brutos — `olist_products_dataset.csv` (catálogo de produtos) e `olist_orders_dataset.csv` (histórico de pedidos) — e identificou uma série de inconsistências (valores ausentes, strings não padronizadas, datas em formatos variados e regras de negócio não validadas) que estavam travando a geração automática de relatórios.



O objetivo deste notebook (`sanitizacao_dados_olist.ipynb`) é implementar, **usando apenas bibliotecas nativas do Python** (`csv`, `os`, `re`, `datetime`), um pipeline completo de sanitização que percorre cinco etapas:



1. **Validação e tratamento de dados ausentes** — preenchimento de categorias de produto vazias com `"sem categoria"` e imputação das dimensões físicas ausentes (peso, comprimento, altura, largura) pela média da coluna.

2. **Padronização de strings e regex** — normalização das categorias de produto (`strip`, `lower` e remoção de caracteres especiais via regex).

3. **Regras de negócio** — classificação dos pedidos sem data de entrega (cancelado, entregue sem data, em andamento, indisponível, status desconhecido) e teste da hipótese "data de entrega nula implica pedido cancelado".

4. **Formatação temporal** — conversão da data de aprovação do pedido do formato ISO (`aaaa-mm-dd hh:mm:ss`) para o formato brasileiro (`dd/mm/aaaa`).

5. **Relatório de status manual** — consolidação de todas as métricas do pipeline e validação final de integridade das bases (checagem de que não restaram valores ausentes nos campos tratados).



Ao final, o pipeline grava dois novos arquivos CSV sanitizados: `olist_products_sanitizado.csv` e `olist_orders_sanitizado.csv`.



Uma decisão de design importante refere-se à etapa de formatação temporal: em vez de sobrescrever a coluna original `order_approved_at`, o pipeline **adiciona uma nova coluna** `order_approved_at_br` com a data no formato brasileiro (`dd/mm/aaaa`). Essa escolha é deliberada e superior por três motivos principais. Primeiro, **preserva o dado bruto como fonte de verdade**: a coluna original continua disponível para auditoria, comparação e reprocessamento, sem risco de perda irreversível de informação. Segundo, **mantém a compatibilidade com as demais etapas do pipeline**: como `order_approved_at` permanece intacta, nenhuma outra função que dependa do formato ISO original precisa ser alterada, reduzindo o acoplamento entre as etapas de sanitização. Terceiro, **facilita a evolução do pipeline**: se no futuro for necessário reformatar a data de outra forma (por exemplo, incluindo horário), basta criar outra coluna derivada, sem reexecução destrutiva sobre o dado fonte — um princípio conhecido como transformações aditivas, que torna o processo mais seguro e rastreável.



## Guia de Execução do Código



**Pré-requisitos:**

- Python 3.x instalado (o notebook foi desenvolvido com Python 3.14, mas qualquer versão 3.8+ deve funcionar, pois só usa bibliotecas padrão).

- Jupyter Notebook ou Jupyter Lab (ou a extensão de notebooks do VS Code).



**Passo a passo:**



1. **Organize a estrutura de pastas.** Crie uma pasta `data/raw/` no mesmo diretório do notebook e coloque dentro dela os arquivos originais:

   ```
   data/raw/olist_products_dataset.csv
   data/raw/olist_orders_dataset.csv
   ```

2. **Abra o notebook** `sanitizacao_dados_olist.ipynb` no Jupyter Notebook, Jupyter Lab ou VS Code.

3. **Execute as células em ordem, de cima para baixo** (não é necessário instalar nenhuma dependência externa, pois o notebook usa somente os módulos `csv`, `os`, `re` e `datetime`, nativos do Python):

   - Primeiro rodam as células de importação e declaração de funções (leitura, tratamento de nulos, regex, regras de negócio, formatação de datas e escrita de arquivos).

   - Em seguida, a célula **"Execução do Pipeline Completo"** dispara todo o processamento: leitura dos CSVs brutos, sanitização de produtos e pedidos, e gravação dos arquivos de saída.

   - Por fim, as células de **relatório de status** e **verificação visual** imprimem um resumo das correções aplicadas e uma amostra dos dados tratados.

4. **Confira a saída.** Ao final da execução, os arquivos sanitizados estarão disponíveis em:

   ```
   data/olist_products_sanitizado.csv
   data/olist_orders_sanitizado.csv
   ```

   O relatório impresso no notebook mostra o total de linhas processadas, quantos valores foram corrigidos em cada etapa e o resultado da validação final (0 valores ausentes restantes nos campos tratados).



## Reflexão Teórica sobre Machine Learning



A lógica de programação aplicada neste pipeline — validar tipos e nulos antes de qualquer cálculo, padronizar strings de forma determinística e formalizar regras de negócio em estruturas condicionais explícitas — é o que garante que os dados entregues a um futuro modelo de Machine Learning representem fielmente o fenômeno que se deseja modelar, e não ruídos do processo de coleta. O princípio de *Garbage In, Garbage Out* explica bem por que isso importa: se um modelo é treinado com categorias de produto grafadas de formas diferentes para o mesmo conceito (ex.: `" Perfumaria"`, `"perfumaria"`, `"PERFUMARIA!"`), ele pode aprender a tratá-las como classes distintas, criando padrões errôneos que só existem por causa da sujeira dos dados — um caminho direto para o **overfitting**, já que o modelo passa a memorizar particularidades do ruído no lugar do sinal real.



O tratamento cuidadoso de valores ausentes também auxilia na prevenção do **viés**. Descartar registros com dados faltantes, por exemplo, pode remover desproporcionalmente certos subgrupos da amostra (produtos de categorias menos populares, pedidos com determinados status), distorcendo a distribuição que o modelo aprende e produzindo previsões enviesadas para esses subgrupos. Por isso, neste pipeline optou-se por técnicas de imputação (como a média das dimensões físicas) apenas quando a proporção de dados faltantes é mínima e comprovadamente não afeta a distribuição da coluna — preservando registros com informação útil sem introduzir distorções artificiais. Da mesma forma, a validação explícita de hipóteses de negócio (como a relação entre pedidos cancelados e ausência de data de entrega) evita que suposições incorretas sejam silenciosamente incorporadas como regras de rotulagem em um futuro modelo supervisionado, o que também é uma fonte comum de viés sistemático.
