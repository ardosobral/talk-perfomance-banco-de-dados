# Performance de banco de dados: a magia dos índices

Se chegou até aqui é porque viu minha palestra e ela te despertou interesse de alguma forma. Aqui separei algumas queries úteis para avaliar a performance do seu banco caso não disponha de nenhuma ferramenta de monitoramento. Aqui você tem um arquivo SQL com um banco com duas tabelas já indexadas e populadas com 1M de linhas cada.

## Modelagem do banco de dados:

<p align="center"><img src="https://github.com/user-attachments/assets/57c0a497-b923-4207-a6d3-d4b6319c9e56" width="800"></p>

## As queries mais executadas:
```sql
SELECT 
    DIGEST_TEXT AS query_text,
    COUNT_STAR AS execution_count,
    SUM_TIMER_WAIT / 1000000000000 AS total_exec_time_sec,
    AVG_TIMER_WAIT / 1000000000000 AS avg_exec_time_sec
FROM 
    performance_schema.events_statements_summary_by_digest
ORDER BY 
    execution_count DESC
LIMIT 10;
```

## Os índices MENOS utilizados:
```sql
SELECT 
    s.TABLE_SCHEMA,
    s.TABLE_NAME,
    s.INDEX_NAME,
    s.SEQ_IN_INDEX,
    s.COLUMN_NAME
FROM 
    information_schema.STATISTICS s
LEFT JOIN 
    performance_schema.table_io_waits_summary_by_index_usage ps
    ON s.TABLE_SCHEMA = ps.OBJECT_SCHEMA 
    AND s.TABLE_NAME = ps.OBJECT_NAME 
    AND s.INDEX_NAME = ps.INDEX_NAME
WHERE 
    ps.INDEX_NAME IS NULL 
    OR ps.COUNT_READ = 0
    AND s.NON_UNIQUE = 1
ORDER BY 
    s.TABLE_SCHEMA, s.TABLE_NAME, s.INDEX_NAME
LIMIT 10;

```

## Os índices MAIS utilizados:
```sql
SELECT 
    OBJECT_SCHEMA AS schema_name,
    OBJECT_NAME AS table_name,
    INDEX_NAME AS index_name,
    COUNT_READ AS total_read,
    COUNT_WRITE AS total_write,
    SUM_TIMER_READ / 1000000000000 AS total_read_time_sec,
    SUM_TIMER_WRITE / 1000000000000 AS total_write_time_sec
FROM 
    performance_schema.table_io_waits_summary_by_index_usage
WHERE 
    INDEX_NAME IS NOT NULL
ORDER BY 
    total_read DESC
LIMIT 10;
```

⚠️ Como mencionei na apresentação, índices são a ponta do iceberg, é muito importante que performance seja um estudo constante, podemos aplicar a solução que mais se adequa a nossa necessidade.

## Referências:
- **[Apresentação no Canva](https://www.canva.com/design/DAGMGp8suy4/-lSBg_cqIH-A9uaOuZvOZA/edit?utm_content=DAGMGp8suy4&utm_campaign=designshare&utm_medium=link2&utm_source=sharebutton)**
- **[High Performance Mysql](https://www.amazon.com.br/High-Performance-MySQL-Strategies-Operating/dp/1492080519)**
- **[Visual Explain](https://dev.mysql.com/doc/workbench/en/wb-performance-explain.html)**
- **[How to use Indexes](https://dev.mysql.com/doc/refman/8.4/en/mysql-indexes.html)**

<p align="center"><img src="https://github.com/user-attachments/assets/c863184a-fd45-4d56-b60d-a5debe40d8d1" width="400"></p>

Desde já muito obrigado 🤗
