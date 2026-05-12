# langchain-chdb

[LangChain](https://github.com/langchain-ai/langchain) provider for [chDB](https://github.com/chdb-io/chdb) — the in-process OLAP SQL engine powered by ClickHouse.

`langchain-chdb` lets you use chDB as a vector store, document loader, and SQL backend for LangChain agents — with no server to operate, and with first-class federation to remote ClickHouse Cloud clusters.

> Status: pre-launch placeholder. The initial release is coming soon. Package layout follows [`langchain-postgres`](https://github.com/langchain-ai/langchain-postgres).

## What this is

LangChain's analytical-SQL story today centers on Postgres and DuckDB. chDB occupies a niche neither can fill:

- **Single engine for retrieval and analytics.** Native vector type plus `cosineDistance()` mean RAG and analytical SQL run in the same process — no separate vector store.
- **Federation.** A LangChain agent running against a local Parquet file can `JOIN` it with a ClickHouse Cloud cluster via `remoteSecure()` in a single query.
- **1000+ ClickHouse functions.** `windowFunnel`, `uniqHLL12`, `geoToH3`, typed JSON, and the rest of the ClickHouse SQL surface are available to agents.

## Install

```bash
pip install langchain-chdb
```

## Planned components

### `ChDB` vector store

```python
from langchain_chdb import ChDB
from langchain_openai import OpenAIEmbeddings

store = ChDB.from_texts(
    texts=["chDB is an embedded ClickHouse.", "It runs SQL on local files."],
    embedding=OpenAIEmbeddings(),
    persist_path="./chdb-store",
)

results = store.similarity_search("which engine embeds ClickHouse?", k=1)
```

Built on the native ClickHouse vector type and `cosineDistance()` — no separate vector engine, no additional dependencies.

### `ChDBLoader` document loader

```python
from langchain_chdb import ChDBLoader

loader = ChDBLoader(
    query="SELECT title, body FROM file('articles.parquet')",
    page_content_column="body",
    metadata_columns=["title"],
)
docs = loader.load()
```

API mirrors [`DuckDBLoader`](https://python.langchain.com/docs/integrations/document_loaders/duckdb) — porting a script is mostly a `from langchain_community.document_loaders` → `from langchain_chdb` rename.

### SQL toolkit integration

chDB plugs into LangChain's `SQLDatabaseToolkit` through the [chdb-sqlalchemy](https://github.com/chdb-io/chdb-sqlalchemy) dialect. Once that lands, an `SQLDatabase.from_uri("chdb:///path/to/db")` call is enough to expose chDB to any SQL-aware agent.

## Reference architecture

```
LangChain agent
      │
      ▼
langchain-chdb  ◀── this package
      │
      ▼
chDB (in-process)
      │       ╲
      ▼        ╲
Parquet/CSV/   remoteSecure() ──► ClickHouse Cloud
S3/HTTP files
```

No external services beyond what the agent already uses (LLM API, optional ClickHouse Cloud). All retrieval and analytical SQL happens inside the agent's process.

## License

Apache 2.0 — see [LICENSE](LICENSE).

## Related

- Main chDB repository: https://github.com/chdb-io/chdb
- chDB documentation: https://clickhouse.com/docs/chdb
- LLM-friendly index: https://clickhouse.com/docs/chdb/llms.txt
- LangChain: https://github.com/langchain-ai/langchain
- SQLAlchemy dialect: https://github.com/chdb-io/chdb-sqlalchemy
- Community: https://discord.gg/D2Daa2fM5K
