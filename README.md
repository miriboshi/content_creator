Analytics Pipeline: Creators & YouTube Content 🚀
=================================================

Este projeto estabelece uma infraestrutura de dados **end-to-end** para a descoberta, ingestão e análise de performance de criadores de conteúdo. A arquitetura foi desenhada sob os princípios de **idempotência, modularidade e eficiência de custos**, utilizando o ecossistema Databricks Lakehouse.

1\. Objetivo do Projeto
-----------------------

Projetar um pipeline capaz de operar **partindo do zero** (sem bases históricas locais), automatizando a descoberta de novos creators via Wikipedia e a atualização contínua de métricas de engajamento via YouTube Data API.

2\. Arquitetura e Fluxo de Dados (Medallion)
--------------------------------------------

O pipeline segue a **Medallion Architecture**, garantindo a governança e a qualidade dos dados em cada estágio de transformação:

*   **Bronze (Raw):** Ingestão via **Auto Loader (cloudFiles)**. Os dados são persistidos em formato Delta mantendo a fidelidade total da fonte (JSON) e metadados de carga.
    
*   **Silver (Cleaned):** Normalização de esquemas, tratamento de tipos (ex: conversão de Unix Timestamps BIGINT para TIMESTAMP) e deduplicação.
    
*   **Gold (Analytics):** Agregações de negócio, rankings de performance (Window Functions) e tabelas pivotadas para consumo executivo.

  
!(imagens/arquitetura.png)
    
3\. Decisões de Engenharia (Senior Design Reasoning)
----------------------------------------------------

### Extração de Dados e "Cold Start"

Como o projeto inicia sem dados, implementei uma estratégia de **Discovery & Enrichment**:

*   **Seed:** Uma UDF Python consome a API da Wikipedia para mapear páginas de criadores.
    
*   **Match:** Uso do algoritmo de **Levenshtein** (Distância de Edição) via Spark SQL para validar a similaridade entre o nome na Wiki e o ID retornado pela API, garantindo alta precisão no vínculo das tabelas sem depender de chaves naturais perfeitas.
    

### Orquestração e Escalabilidade

*   **Orquestrador Alvo:** Em ambiente corporativo, utilizaria o **Databricks Workflows**. Ele oferece integração nativa com o Unity Catalog e suporte a **File Arrival Triggers**.
    
*   **Eficiência de Custo (FinOps):** O pipeline foi desenvolvido com trigger(availableNow=True). Isso permite o processamento **Batch-Incremental**, onde o cluster processa apenas o delta de novos dados e encerra a computação imediatamente após a carga, otimizando o consumo de DBUs.
    

### Qualidade e Monitoramento

*   **Idempotência:** O uso de MERGE INTO e checkpoints do Auto Loader garante que o pipeline possa ser reexecutado a qualquer momento sem duplicar dados.
    
*   **Observabilidade:** Implementação de queries de _mismatch_ para auditoria (identificar posts sem cadastro de criador) e monitoramento de volumetria entre camadas.
    

4\. Modelagem de Dados (ERD)
----------------------------

A modelagem foi estruturada para suportar consultas de séries temporais e rankings:

*   **creators\_scrape\_wiki (Bronze):** Dados brutos da extração inicial.
    
*   **users\_yt (Silver/Dim):** Tabela de dimensões com user\_id (PK) e metadados do canal.
    
*   **posts\_creator (Silver/Fact):** Tabela de fatos com métricas de engajamento (likes, views) e published\_at.
    
*   **analytics\_creators\_ranking (Gold):** View materializada com o Top 3 posts por performance.
    

5\. Deployment e Limitações Técnicas
------------------------------------

> **Nota sobre o Ambiente Community Edition:**Devido às restrições da versão gratuita do Databricks, recursos de automação como **Workflows (Jobs)** e **File Arrival Triggers** não estão habilitados.

*   **Status Atual:** O pipeline é acionado manualmente via notebooks.
    
*   **Production Ready:** O código foi desenvolvido seguindo padrões de produção. Em uma migração para o Tier _Premium/Enterprise_, a automação seria imediata via Job Trigger, sem necessidade de refatoração do código de ETL.
    

6\. Boas Práticas e Engenharia de Software
------------------------------------------

*   **Gitflow:** Versionamento estruturado em branches main e develop.
    
*   **Segurança:** Gestão de tokens de API via **Databricks Secrets** (evitando chaves expostas no código).
    
*   **Modularidade:** Funções de configuração e exportação centralizadas em arquivos utils para máximo reuso entre notebooks.
