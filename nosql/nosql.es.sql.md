
# Elasticsearch SQL

参考文档:

- [SQL access | Elasticsearch Guide [7.6] | Elastic - ElasticSearch 官方文档](https://www.elastic.co/guide/en/elasticsearch/reference/7.6/xpack-sql.html)

## Elasticsearch SQL API

### SQL 介绍

Elasticsearch SQL 是 Elasticsearch 官方 XPack 提供的一系列 DSL 的 API, 可以使用 SQL 的方式访问 Elasticsearch 中的数据, 而不是去拼接原生的 Elasticsearch JSON 查询接口 (即使有 SDK 也很麻烦).

对比一下 数据库 (SQL) 和 Elasticsearch 对应关系:

| SQL | ElasticSearch |
| --- | --- |
| column 列 | field 字段 |
| row 行 | document 文档 |
| table 表 | index 索引 |
| schema 模式 | mapping 映射 |
| database server 数据库服务器 | ElasticSearch 集群实例 |


ElasticSearch SQL 支持以下语法, 不支持JOIN, 不支持较复杂的子查询:

```sql
SELECT select_expr [, ...]
[ FROM table_name ]
[ WHERE condition ]
[ GROUP BY grouping_element [, ...] ]
[ HAVING condition]
[ ORDER BY expression [ ASC | DESC ] [, ...] ]
[ LIMIT [ count ] ]
[ PIVOT ( aggregation_expr FOR column IN ( value [ [ AS ] alias ] [, ...] ) ) ]
```

还可以使用以下语句查看对应的索引和可用的函数:

```sql
SHOW TABLES;
SHOW FUNCTIONS;
```


### API 接口

有两个核心接口:

- `_xpack/sql/translate`
- `_xpack/sql`

调试翻译接口, 你可以发送一段包含 Elasticsearch SQL 给 Elasticsearch, 它会翻译成具体的 JSON 查询接口. 但是其中的表名和字段必须服务端上实际存在的, 才能得到正确的响应. 拿到接口后可以使用 Elasticsearch 的 SDK 来生成这个查询请求.

```bash
curl -XPOST https://elasticsearch:9200/_xpack/sql/translate \
     -H "Authorization: Basic xxxxxxx" \
     -H "Content-Type: application/json" \
     -d '{"query": "select * from \"my-index*\" limit 10", "fetch_size": 10}'
```

或者直接使用这个接口进行查询:

```bash
curl -XPOST https://elasticsearch:9200/_xpack/sql?format=json \
     -H "Authorization: Basic xxxxxxx" \
     -H "Content-Type: application/json" \
     -d '{"query": "select * from \"my-index*\" limit 10"}'
```

于此同时 Elasticsearch 甚至提供 PreparedStatement 风格式的查询, 它接受一个参数的数组并将 SQL 语句中的占位符 `?` 替换为参数. 性能方面, 与数据库类似, 这条查询会被 Elasticsearch 缓存下来, 不会每次查询都触发硬解析.

```sql
curl -XPOST https://elasticsearch:9200/_xpack/sql?format=json \
     -H "Authorization: Basic xxxxxxx" \
     -H "Content-Type: application/json" \
     -d '{"query": "select * from \"my-index*\" where @timestamp > ? limit 10", "params": ["2022-07-26T00:00:00Z"]}'
```


### 局限性

- 仅只是 SELECT 语句和一部分 SHOW 语句, 完全不支持 INSERT, DELETE, UPDATE 语句

## Elasticsearch JDBC

Elasticsearch 也提供 JDBC 的 Jar 包, 但需要白金版 License 或者~~破解~~.

```xml
<repositories>
  <repository>
    <id>elastic.co</id>
    <url>https://artifacts.elastic.co/maven</url>
  </repository>
</repositories>

<dependencies>
  <dependency>
    <groupId>org.elasticsearch.plugin</groupId>
    <artifactId>x-pack-sql-jdbc</artifactId>
    <version>7.0.1</version>
  </dependency>
</dependencies>
```
