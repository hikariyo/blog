---
title: 使用 FastAPI+aiosqlite+databases 搭建服务端的基础用法
date: 2023-01-13 10:04:43.473
updated: 2023-01-13 13:24:25.387

categories: 
- Dev
tags: 
- Python
---

## 说明

本文是主要为从未使用过这些框架的人做一个介绍，并没有太多的技术含量。

## 连接池

首先先把我最焦虑的问题放在这里，到底要不要一个连接池？

我在 `aiosqlite` 的仓库中搜到了这样一条 [issue](https://github.com/omnilib/aiosqlite/issues/163)，作者为我们介绍了为什么使用 `sqlite` 数据库时连接池不是**那么重要**。我并没有说它不重要，只是在轻量级使用中可以不在意这点。我这里把原文复制过来，并且把重要句子标粗。

---

Generally speaking, the other DB libraries are pooling connections because **a) making a network connection takes a significant amount of time between TCP handshake, etc**, so having connections already made and waiting is faster; and **b) because network bandwidth is limited**, so preventing too many simultaneous connections can help ensure better network utilization.
 
However, since sqlite by definition is a local database, either on-disk or in memory, the benefits from connection pooling are minimal: there is no network delay to opening a file on disk or allocating memory, and disk bandwidth is much higher (and better managed by the kernel). "Connections" are lightweight compared to mysql or postgres, and can generally be opened or closed at will. The biggest cost to a new connection will likely be spinning up another thread.

I added a simple perf test to measure speed of creating connections, and on my M1 Mac Mini with Python 3.10.2, I was able to open ~4700 connections per second from a file, or ~5300/s for in-memory DBs:
```
(.venv) jreese@mordin ~/workspace/aiosqlite main  » python -m unittest aiosqlite.tests.perf.PerfTest -k test_connection
Running perf tests for at least 2.0s each...
..
Perf Test                 Iterations  Duration         Rate
connection_file                 9454      2.0s     4726.7/s
connection_memory              10504      2.0s     5251.8/s

Ran 2 tests in 4.005s
```

When it comes to concurrency, you can simply create more connections with aiosqlite, **without needing a connection pool**. Pooling connections could potentially still help in that regard if you expect to be making a very large number of concurrent requests and want to limit the number of background threads, though you will likely end up with contention on the limited number of connections in the pool instead.

My general suggestion for most use cases with aiosqlite is to just keep the code simple, and **open a new connection using a context manager anywhere in the code you need to talk to the db**. This also helps keep transactions isolated, and prevents a wide class of bugs around managing queries on shared connection threads.

---

简而言之，就是使用 `sqlite` 这种本地文件的数据库，连接池相较于 `MySQL` 那种通过网络连接的数据库来说就没有那么必要了，所以你可以放心大胆的在需要的时候创建数据库连接。


## 依赖

均可通过 `pip install` 下载。

+ [databases](https://github.com/encode/databases)
+ [aiosqlite](https://github.com/omnilib/aiosqlite)
+ [fastapi](https://github.com/tiangolo/fastapi)
+ [uvicorn](https://github.com/encode/uvicorn)
+ [pydantic](https://github.com/pydantic/pydantic)


## 实践

首先，如果去看 `databases/core.py` 源码的话，可以注意到以下几行：

```python
async def __aenter__(self) -> "Database":
    await self.connect()
    return self


async def __aexit__(
    self,
    exc_type: typing.Optional[typing.Type[BaseException]] = None,
    exc_value: typing.Optional[BaseException] = None,
    traceback: typing.Optional[TracebackType] = None,
) -> None:
    await self.disconnect()
```

所以说我们可以通过异步的 `context manager` 也就是 `async with` 来管理一个 `databases.Database` 对象。

假设我们有一个储存文本，并根据关键字随机获取的需求，我们需要创建一个表，包含 `id` 和 `text` 字段。那么我为了方便，创建了一个工具类来帮助我们管理。

```python
import databases
from typing import Iterable, Optional, AsyncGenerator


class Database:
    _instance: Optional['Database'] = None

    
    def __init__(self, db: databases.Database) -> None:
        self._db = db

    
    @classmethod
    async def get_instance(cls) -> 'Database':
        if cls._instance is not None:
            return cls._instance

        db = databases.Database('sqlite+aiosqlite:///./db.sqlite')

        async with db:
            # Initialize tables.
            await db.execute('''
            CREATE TABLE IF NOT EXISTS texts (
                `id` INTEGER PRIMARY KEY AUTOINCREMENT NOT NULL,
                `text` TEXT NOT NULL UNIQUE
            );
            ''')

        cls._instance = cls(db)
        return cls._instance


    async def add_texts(self, texts: Iterable[str]) -> None:
        async with self._db as db:
            query = 'INSERT OR IGNORE INTO texts (text) VALUES (:text)'
            values = [{'text': text} for text in texts]
            await db.execute_many(query, values)

    
    async def get_random_text(self, keyword: str) -> Optional[str]:
        async with self._db as db:
            query = 'SELECT text FROM texts WHERE INSTR (text, :keyword) > 0 ORDER BY RANDOM() LIMIT 1'
            values = {'keyword': keyword}
            if result := await db.fetch_one(query, values):
                return result[0]
```

为了方便，本项目直接写 `SQL` 语句了，就没用 `sqlalchemy`，如果数据复杂就不要直接写 `SQL` 语句了。

## FastAPI

介绍完了我们的工具类，接下来就是集成到 `FastAPI` 中了。这里我们使用了**依赖注入**来获取工具类 `Database` 的实例。

```python
from pydantic import BaseModel
from fastapi import FastAPI, Depends
from fastapi.responses import JSONResponse


app = FastAPI()


class PostBody(BaseModel):
    texts: list[str]


async def get_db() -> Database:
    return await Database.get_instance()


@app.post('/text')
async def handle_add_texts(body: PostBody, db: Database = Depends(get_db)):
    await db.add_texts(body.texts)


@app.get('/text')
async def handle_text(keyword: str = '', db: Database = Depends(get_db)):
    if text := await db.get_random_text(keyword):
        return text

    return JSONResponse(
        content=f'no text contains given keyword {keyword}',
        status_code=404,
    )
```

之后用 `uvicorn` 运行，就搭建了一个简单的服务端了，基本上 `CRUD` 都可以实现。

```bash
uvicorn server:app --port 8080
```
