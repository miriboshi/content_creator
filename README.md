Analytics Pipeline: Creators & YouTube Content 🚀
Este projeto estabelece uma infraestrutura de dados end-to-end para a descoberta, ingestão e análise de performance de criadores de conteúdo. A arquitetura foi desenhada sob os princípios de idempotência, modularidade e eficiência de custos, utilizando o ecossistema Databricks Lakehouse.

1. Objetivo do Projeto
Projetar um pipeline capaz de operar partindo do zero (sem bases históricas locais), automatizando a descoberta de novos creators via Wikipedia e a atualização contínua de métricas de engajamento via YouTube Data API.

2. Arquitetura e Fluxo de Dados (Medallion)
O pipeline segue a Medallion Architecture, garantindo a governança e a qualidade dos dados em cada estágio de transformação:

Bronze (Raw): Ingestão via Auto Loader (cloudFiles). Os dados são persistidos em formato Delta mantendo a fidelidade total da fonte (JSON) e metadados de carga.

Silver (Cleaned): Normalização de esquemas, tratamento de tipos (ex: conversão de Unix Timestamps BIGINT para TIMESTAMP) e deduplicação.

Gold (Analytics): Agregações de negócio, rankings de performance (Window Functions) e tabelas pivotadas para consumo executivo.
