🛒 Varejo Vendas Brasil — Data Warehouse & BI

Projeto de Engenharia de Dados e Business Intelligence aplicado a uma empresa de varejo de médio porte com operações físicas e e-commerce. Pipeline completo de ETL com Pentaho, modelagem Star Schema em PostgreSQL e dashboards estratégicos.


📌 Contexto do Problema
A Varejo Vendas Brasil opera desde 2015 no segmento de eletrônicos, eletrodomésticos e utilidades domésticas. Com o crescimento acelerado, os dados operacionais explodiram — centenas de notas fiscais por dia, múltiplos canais de venda, equipe comercial distribuída — mas a capacidade analítica não acompanhou o ritmo.
Problemas reais que este projeto resolve:
DorSolução aplicadaRelatórios manuais e inconsistentesCamada Stage com padronização e validação via PentahoImpossível cruzar vendas × produtos × vendedoresStar Schema com 6 dimensões integradasSem visão histórica consolidadaDim_Tempo com granularidade diária e suporte a YTD/MoMDecisões por intuiçãoConsultas OLAP com KPIs de margem, ranking e sazonalidade

🏗️ Arquitetura da Solução
┌─────────────────┐              ┌──────────────┐             ┌─────────────────┐
│  Banco OLTP     │  Pentaho ETL │  Stage (STG) │ Pentaho ETL │  Data Warehouse │
│  varejo_vendas  │ ──────────► │  varejo_stage│ ──────────► │  varejo_dw      │
│  (transacional) │   .ktr/.kjb  │ (padronização│   .ktr/.kjb │  (Star Schema)  │
└─────────────────┘              └──────────────┘             └─────────────────┘
                                                                       │
                                                                       ▼
                                                              ┌─────────────────┐
                                                              │  Análises OLAP  │
                                                              │  (SQL / BI)     │
                                                              └─────────────────┘
Camadas do pipeline
OLTP (varejo_vendas_brasil) → Stage (varejo_stage) → DW (varejo_data_warehouse)

OLTP: banco transacional com dados operacionais brutos
Stage: limpeza, padronização de tipos, tratamento de nulos — orquestrado com Jobs do Pentaho (.kjb)
DW: modelo dimensional Star Schema pronto para análise


📐 Modelagem — Star Schema
                    ┌──────────────┐
                    │  dim_tempo   │
                    └──────┬───────┘
                           │
┌──────────────┐    ┌──────┴───────┐    ┌──────────────────┐
│ dim_cliente  │────│  fato_venda  │────│   dim_produto    │
└──────────────┘    └──────┬───────┘    └──────────────────┘
                           │
         ┌─────────────────┼──────────────────┐
         │                 │                  │
┌────────┴──────┐  ┌───────┴──────┐  ┌───────┴──────────┐
│ dim_vendedor  │  │dim_pagamento │  │  dim_fornecedor  │
└───────────────┘  └──────────────┘  └──────────────────┘
Tabela Fato — fato_venda
ColunaTipoDescriçãosk_vendaINT (PK)Surrogate keyfk_clienteINT→ dim_clientefk_produtoINT→ dim_produtofk_vendedorINT→ dim_vendedorfk_pagamentoINT→ dim_pagamentofk_fornecedorINT→ dim_fornecedorfk_tempoINT→ dim_tempoqt_itensINTQuantidade vendidavl_brutoDECIMALValor bruto da vendavl_custoDECIMALCusto do produtovl_descontoDECIMALDesconto aplicadovl_liquidoDECIMALReceita líquidamargem_lucroDECIMALMargem calculada (%)

🛠️ Stack Tecnológica
CategoriaTecnologiaBanco de dadosPostgreSQL 15ETL (transformações)Pentaho Data Integration (PDI / Kettle)Orquestração dos jobsPentaho Jobs (.kjb)Transformações unitáriasPentaho Transformations (.ktr)Modelagem e DDLSQL puroVersionamentoGit / GitHub

📂 Estrutura do Repositório
varejo-vendas-brasil-dw/
│
├── Banco Relacional/
│   └── varejo_vendas_brasil.sql      # DDL do banco transacional (OLTP)
│
├── Stage Banco/
│   └── varejo_stage.sql              # DDL da camada Stage
│
├── DataWarehouse Banco/
│   └── varejo_data_warehouse.sql     # DDL do DW (Star Schema)
│
├── KTR/
│   ├── Jobs/
│   │   ├── projeto01_stage_job.kjb   # Job orquestrador da carga Stage
│   │   └── projeto01_dw_job.kjb      # Job orquestrador da carga DW
│   │
│   ├── projeto01_stage_clientes.ktr
│   ├── projeto01_stage_forma_pagamento.ktr
│   ├── projeto01_stage_fornecedor.ktr
│   ├── projeto01_stage_item_nota_fiscal.ktr
│   ├── projeto01_stage_nota_fiscal.ktr
│   ├── projeto01_stage_produtos.ktr
│   ├── projeto01_stage_vendedor.ktr
│   │
│   ├── projeto01_dw_cliente.ktr
│   ├── projeto01_dw_forma_pagamento.ktr
│   ├── projeto01_dw_fornecedor.ktr
│   ├── projeto01_dw_produto.ktr
│   ├── projeto01_dw_tempo.ktr
│   ├── projeto01_dw_vendedor.ktr
│   └── projeto01_dw_fato_vendas.ktr  # Carga da tabela fato (executada por último)
│
└── README.md

🔄 Fluxo de Execução do ETL
O pipeline é dividido em dois Jobs principais, cada um orquestrando suas transformações na sequência correta:
Job 1 — Carga Stage (projeto01_stage_job.kjb)
clientes → forma_pagamento → fornecedor → produtos → vendedor → nota_fiscal → item_nota_fiscal
Job 2 — Carga DW (projeto01_dw_job.kjb)
dim_tempo → dim_cliente → dim_vendedor → dim_fornecedor → dim_forma_pagamento → dim_produto → fato_vendas

⚠️ A fato_vendas é sempre carregada por último, pois depende de todas as dimensões já populadas.


📊 Perguntas de Negócio Respondidas

📅 Sazonalidade: Como evoluíram as vendas por mês e categoria nos últimos anos?
🧍 Performance comercial: Quais vendedores lideram por trimestre?
💰 Rentabilidade: Quais produtos e categorias geram maior margem de lucro?
🌍 Fornecedores: Quais fornecedores concentram maior volume de produtos vendidos?
🧾 Meios de pagamento: Quais formas são mais usadas e em que perfil de compra?


🚀 Como Executar
Pré-requisitos

PostgreSQL instalado e rodando
Pentaho Data Integration (PDI) — versão Community Edition

1. Clone o repositório
bashgit clone https://github.com/alanoregis/varejo-vendas-brasil-dw.git
cd varejo-vendas-brasil-dw
2. Crie os bancos no PostgreSQL
Execute os scripts SQL na seguinte ordem:
sql-- 1. Banco transacional
\i "Banco Relacional/varejo_vendas_brasil.sql"

-- 2. Camada Stage
\i "Stage Banco/varejo_stage.sql"

-- 3. Data Warehouse
\i "DataWarehouse Banco/varejo_data_warehouse.sql"
3. Configure a conexão no Pentaho
Abra o PDI e configure a conexão com o PostgreSQL local (host, porta, usuário e senha).
4. Execute os Jobs na ordem
KTR/Jobs/projeto01_stage_job.kjb   ← primeiro
KTR/Jobs/projeto01_dw_job.kjb      ← depois

📈 Resultados

✅ Pipeline ETL completo em duas camadas: Stage → DW
✅ 14 transformações .ktr versionadas (7 Stage + 7 DW)
✅ Modelo Star Schema com 6 dimensões e 1 tabela fato pronta para análise OLAP
✅ Separação clara de responsabilidades entre camadas (OLTP, Stage, DW)
⏱️ Tempo de geração de relatórios: de dias → minutos
