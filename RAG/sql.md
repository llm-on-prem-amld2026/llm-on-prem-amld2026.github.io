---
title: "Linking a SQL Database"
parent: "Powering LLMs with RAG"
nav_order: 2
---

ADD SHORT INTRO TO SQL DATABASE

## Adding a SQL Database
We will now add an existing SQL server to our Open WebUI instance. Specifically, we will connect a [soccer database](https://www.kaggle.com/datasets/hugomathien/soccer) from Kaggle, containing over 25,000 matches and over 10,000 players. We set up this server using [PostgreSQL](https://www.w3schools.com/postgresql/), a widely used open-source object-relational database management system. We will first add a **tool** to our Open WebUI, which will specify how the LLM should execute SQL queries. Then, we will specify the specifics of our SQL server in the valves of the tool.

{: .action}
> 1. Go to your **workspace**, on the left side of the screen. Then, move to **Tools** at the top of the screen.
> 2. Click on **+ New Tool**, and replace the template code with the code below. Give your tool a name and description, and don't forget to save it.

<details markdown="1">
<summary>Show filter code</summary>

```python
"""
title: SQL Server Access
author: MENG
author_urls:
  - https://github.com/mengvision
description: A tool for reading database information and executing SQL queries, supporting multiple databases such as MySQL, PostgreSQL, SQLite, and Oracle. It provides functionalities for listing all tables, describing table schemas, and returning query results in CSV format. A versatile DB Agent for seamless database interactions.
required_open_webui_version: 0.5.4
requirements: pymysql, sqlalchemy, cx_Oracle
version: 0.1.6
licence: MIT

# Changelog
## [0.1.6] - 2025-03-11
### Added
- Added `get_table_indexes` method to retrieve index information for a specific table, supporting MySQL, PostgreSQL, SQLite, and Oracle.
- Enhanced metadata capabilities by providing detailed index descriptions (e.g., index name, columns, and type).
- Improved documentation to include the new `get_table_indexes` method and its usage examples.
- Updated error handling in `get_table_indexes` to provide more detailed feedback for unsupported database types.

## [0.1.5] - 2025-01-20
### Changed
- Updated `list_all_tables` and `table_data_schema` methods to accept `db_name` as a function parameter instead of using `self.valves.db_name`.
- Improved flexibility by decoupling database name from class variables, allowing dynamic database selection at runtime.

## [0.1.4] - 2025-01-17
### Added
- Added support for Oracle database using `cx_Oracle` driver.
- Added dynamic engine creation in each method to ensure fresh database connections for every operation.
- Added support for Oracle-specific queries in `list_all_tables` and `table_data_schema` methods.

### Changed
- Moved `self._get_engine()` from `__init__` to individual methods for better flexibility and tool compatibility.
- Updated `_get_engine` method to support Oracle database connection URL.
- Improved `table_data_schema` method to handle Oracle-specific column metadata.

### Fixed
- Fixed potential connection issues by ensuring each method creates its own database engine.
- Improved error handling for Oracle-specific queries and edge cases.

## [0.1.3] - 2025-01-17
### Added
- Added support for multiple database types (e.g., MySQL, PostgreSQL, SQLite) using SQLAlchemy.
- Added configuration flexibility through environment variables or external configuration files.
- Enhanced query security with stricter validation and SQL injection prevention.
- Improved error handling with detailed exception messages for better debugging.

### Changed
- Replaced `pymysql` with SQLAlchemy for broader database compatibility.
- Abstracted database connection logic into a reusable `_get_engine` method.
- Updated `table_data_schema` method to support multiple database types.

### Fixed
- Fixed potential SQL injection vulnerabilities in query execution.
- Improved handling of edge cases in query validation and execution.

## [0.1.2] - 2025-01-16
### Added
- Added support for specifying the database port with a default value of `3306`.
- Abstracted database connection logic into a reusable `_get_connection` method.

## [0.1.1] - 2025-01-16
### Added
- Support for additional read-only query types: `SHOW`, `DESCRIBE`, `EXPLAIN`, and `USE`.
- Enhanced query validation to block sensitive keywords (e.g., `INSERT`, `UPDATE`, `DELETE`, `CREATE`, `DROP`, `ALTER`).

### Fixed
- Improved handling of queries starting with `WITH` (CTE queries).
- Fixed case sensitivity issues in query validation.

## [0.1.0] - 2025-01-09
### Initial Release
- Basic functionality for listing tables, describing table schemas, and executing `SELECT` queries.
- Query results returned in CSV format.
"""

import os
from typing import List, Dict, Any
from pydantic import BaseModel, Field
import re
from sqlalchemy import create_engine, text
from sqlalchemy.engine.base import Engine
from sqlalchemy.exc import SQLAlchemyError


class Tools:
    class Valves(BaseModel):
        db_host: str = Field(
            default="localhost",
            description="The host of the database. Replace with your own host.",
        )
        db_user: str = Field(
            default="admin",
            description="The username for the database. Replace with your own username.",
        )
        db_password: str = Field(
            default="admin",
            description="The password for the database. Replace with your own password.",
        )
        db_name: str = Field(
            default="db",
            description="The name of the database. Replace with your own database name.",
        )
        db_port: int = Field(
            default=3306,  # Oracle 默认端口
            description="The port of the database. Replace with your own port.",
        )
        db_type: str = Field(
            default="mysql",
            description="The type of the database (e.g., mysql, postgresql, sqlite, oracle).",
        )

    def __init__(self):
        """
        Initialize the Tools class with the credentials for the database.
        """
        print("Initializing database tool class")
        self.citation = True
        self.valves = Tools.Valves()

    def _get_engine(self) -> Engine:
        """
        Create and return a database engine using the current configuration.
        """
        if self.valves.db_type == "mysql":
            db_url = f"mysql+pymysql://{self.valves.db_user}:{self.valves.db_password}@{self.valves.db_host}:{self.valves.db_port}/{self.valves.db_name}"
        elif self.valves.db_type == "postgresql":
            db_url = f"postgresql://{self.valves.db_user}:{self.valves.db_password}@{self.valves.db_host}:{self.valves.db_port}/{self.valves.db_name}"
        elif self.valves.db_type == "sqlite":
            db_url = f"sqlite:///{self.valves.db_name}"
        elif self.valves.db_type == "oracle":
            db_url = f"oracle+cx_oracle://{self.valves.db_user}:{self.valves.db_password}@{self.valves.db_host}:{self.valves.db_port}/?service_name={self.valves.db_name}"
        else:
            raise ValueError(f"Unsupported database type: {self.valves.db_type}")

        return create_engine(db_url)

    def list_all_tables(self, db_name: str) -> str:
        """
        List all tables in the database.
        :param db_name: The name of the database.
        :return: A string containing the names of all tables.
        """
        print("Listing all tables in the database")
        engine = self._get_engine()  # 动态创建引擎
        try:
            with engine.connect() as conn:
                if self.valves.db_type == "mysql":
                    result = conn.execute(text("SHOW TABLES;"))
                elif self.valves.db_type == "postgresql":
                    result = conn.execute(
                        text(
                            "SELECT table_name FROM information_schema.tables WHERE table_schema = 'public';"
                        )
                    )
                elif self.valves.db_type == "sqlite":
                    result = conn.execute(
                        text("SELECT name FROM sqlite_master WHERE type='table';")
                    )
                elif self.valves.db_type == "oracle":
                    result = conn.execute(text("SELECT table_name FROM user_tables;"))
                else:
                    return "Unsupported database type."
                tables = [row[0] for row in result.fetchall()]
                if tables:
                    return (
                        "Here is a list of all the tables in the database:\n\n"
                        + "\n".join(tables)
                    )
                else:
                    return "No tables found."
        except SQLAlchemyError as e:
            return f"Error listing tables: {str(e)}"

    def get_table_indexes(self, db_name: str, table_name: str) -> str:
        """
        Get the indexes of a specific table in the database.
        :param db_name: The name of the database.
        :param table_name: The name of the table.
        :return: A string describing the indexes of the table.
        """
        print(f"Getting indexes for table: {table_name}")
        engine = self._get_engine()
        try:
            with engine.connect() as conn:
                if self.valves.db_type == "mysql":
                    query = text(
                        """
                        SHOW INDEX FROM :table_name;
                        """
                    )
                elif self.valves.db_type == "postgresql":
                    query = text(
                        """
                        SELECT indexname, indexdef
                        FROM pg_indexes
                        WHERE tablename = :table_name;
                        """
                    )
                elif self.valves.db_type == "sqlite":
                    query = text(
                        """
                        PRAGMA index_list(:table_name);
                        """
                    )
                elif self.valves.db_type == "oracle":
                    query = text(
                        """
                        SELECT index_name, column_name
                        FROM user_ind_columns
                        WHERE table_name = :table_name;
                        """
                    )
                else:
                    return "Unsupported database type."
                result = conn.execute(query, {"table_name": table_name})
                indexes = result.fetchall()
                if not indexes:
                    return f"No indexes found for table: {table_name}"
                description = f"Indexes for table '{table_name}':\n"
                for index in indexes:
                    description += f"- {index[0]}: {index[1]}\n"
                return description
        except SQLAlchemyError as e:
            return f"Error getting indexes: {str(e)}"

    def table_data_schema(self, db_name: str, table_name: str) -> str:
        """
        Describe the schema of a specific table in the database, including column comments.
        :param db_name: The name of the database.
        :param table_name: The name of the table to describe.
        :return: A string describing the data schema of the table.
        """
        print(f"Describing table: {table_name}")
        engine = self._get_engine()  # 动态创建引擎
        try:
            with engine.connect() as conn:
                if self.valves.db_type == "mysql":
                    query = text(
                        """
                        SELECT COLUMN_NAME, COLUMN_TYPE, IS_NULLABLE, COLUMN_KEY, COLUMN_COMMENT
                        FROM INFORMATION_SCHEMA.COLUMNS
                        WHERE TABLE_SCHEMA = :db_name AND TABLE_NAME = :table_name;
                    """
                    )
                elif self.valves.db_type == "postgresql":
                    query = text(
                        """
                        SELECT column_name, data_type, is_nullable, column_default, ''
                        FROM information_schema.columns
                        WHERE table_name = :table_name;
                    """
                    )
                elif self.valves.db_type == "sqlite":
                    query = text("PRAGMA table_info(:table_name);")
                elif self.valves.db_type == "oracle":
                    query = text(
                        """
                        SELECT column_name, data_type, nullable, data_default, comments
                        FROM user_tab_columns
                        LEFT JOIN user_col_comments
                        ON user_tab_columns.table_name = user_col_comments.table_name
                        AND user_tab_columns.column_name = user_col_comments.column_name
                        WHERE user_tab_columns.table_name = :table_name;
                    """
                    )
                else:
                    return "Unsupported database type."
                result = conn.execute(
                    query, {"db_name": db_name, "table_name": table_name}
                )
                columns = result.fetchall()
                if not columns:
                    return f"No such table: {table_name}"
                description = (
                    f"Table '{table_name}' in the database has the following columns:\n"
                )
                for column in columns:
                    if self.valves.db_type == "sqlite":
                        column_name, data_type, is_nullable, _, _, _ = column
                        column_comment = ""
                    elif self.valves.db_type == "oracle":
                        (
                            column_name,
                            data_type,
                            is_nullable,
                            data_default,
                            column_comment,
                        ) = column
                    else:
                        (
                            column_name,
                            data_type,
                            is_nullable,
                            column_key,
                            column_comment,
                        ) = column
                    description += f"- {column_name} ({data_type})"
                    if is_nullable == "YES" or is_nullable == "Y":
                        description += " [Nullable]"
                    if column_key == "PRI":
                        description += " [Primary Key]"
                    if column_comment:
                        description += f" [Comment: {column_comment}]"
                    description += "\n"
                return description
        except SQLAlchemyError as e:
            return f"Error describing table: {str(e)}"

    def execute_read_query(self, query: str) -> str:
        """
        Execute a read query and return the result in CSV format.
        :param query: The SQL query to execute.
        :return: A string containing the result of the query in CSV format.
        """
        print(f"Executing query: {query}")
        normalized_query = query.strip().lower()
        if not re.match(
            r"^\s*(select|with|show|describe|desc|explain|use)\s", normalized_query
        ):
            return "Error: Only read-only queries (SELECT, WITH, SHOW, DESCRIBE, EXPLAIN, USE) are allowed. CREATE, DELETE, INSERT, UPDATE, DROP, and ALTER operations are not permitted."

        sensitive_keywords = [
            "insert",
            "update",
            "delete",
            "create",
            "drop",
            "alter",
            "truncate",
            "grant",
            "revoke",
            "replace",
        ]
        for keyword in sensitive_keywords:
            if re.search(rf"\b{keyword}\b", normalized_query):
                return f"Error: Query contains a sensitive keyword '{keyword}'. Only read operations are allowed."

        engine = self._get_engine()  # 动态创建引擎
        try:
            with engine.connect() as conn:
                result = conn.execute(text(query))
                rows = result.fetchall()
                if not rows:
                    return "No data returned from query."

                column_names = result.keys()
                csv_data = f"Query executed successfully. Below is the actual result of the query {query} running against the database in CSV format:\n\n"
                csv_data += ",".join(column_names) + "\n"
                for row in rows:
                    csv_data += ",".join(map(str, row)) + "\n"
                return csv_data
        except SQLAlchemyError as e:
            return f"Error executing query: {str(e)}"
```
</details>

{: .note}
> ADD A FEW NOTES ON THE CODE ABOVE

Now that we have added the tool, all that remains is to specify the details of our SQL server in the valves. Then, we can use our LLM to execute SQL queries of our liking.

{: .action}
> In your **Workspace**, under **Tools**, click on the wheel to adjust the valves. Specify the details below:
> * **Db Host:** 91.92.143.253
> * **Db User:** participant
> * **Db Password:** _<See screen slides>_
> * **Db Name:** soccer_db
> * **Db Port:** 5432
> * **Db Type:** postgresql

Now you're tool is all ready! It is time to start experimenting and evaluate how well your LLM can query the database. Have a look at the [database](https://www.kaggle.com/datasets/hugomathien/soccer), and think about what questions you may ask.

{: .action}
> When starting a new chat, in your chat window, click on _Integration -> Tools_, and toggle on your tool. Now the access to the SQL database is enabled. You can now ask the LLM questions that can be answered based on the database. Try it out, how well does it perform? In what cases does it not behave as you expected? 

## Additional exercise: Launching Your Own SQL Server
