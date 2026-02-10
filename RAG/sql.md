---
title: "Linking a SQL Database"
parent: "Powering LLMs with RAG"
nav_order: 2
---

Relational databases are one of the most common ways real-world data is stored and accessed. SQL (Structured Query Language) databases organize data into tables with well-defined schemas, making them efficient for querying, filtering, and aggregating large datasets. By linking an SQL database to an LLM, we can move beyond static documents and enable the model to answer questions grounded in structured, up-to-date data. This is a powerful extension of Retrieval-Augmented Generation (RAG), allowing the LLM to reason over rows, columns, and relationships rather than just text. An introduction to SQL can be found [here](https://www.w3schools.com/sql/sql_intro.asp).

## Adding a SQL Database
We will now add an existing SQL server to our Open WebUI instance. Specifically, we will connect a [soccer database](https://www.kaggle.com/datasets/hugomathien/soccer) from Kaggle, containing over 25,000 matches and over 10,000 players. We set up this server using [PostgreSQL](https://www.w3schools.com/postgresql/), a widely used open-source object-relational database management system. We will first add a **tool** to our Open WebUI, which will specify how the LLM should execute SQL queries. Then, we will specify the specifics of our SQL server in the valves of the tool.

### 1. Adding the SQL Tool
We will first add a Tool to Open WebUI that teaches the LLM how to talk to a database.

{: .action}
> 1. Go to your **workspace**, on the left side of the screen. Then, move to **Tools** at the top of the screen.
> 2. Click on **+ New Tool**, name it `SQL_Soccer` and replace the template code with the code below. Don't forget to **save the tool**.

<details markdown="1">
<summary>Show filter code</summary>

```python
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
> * The tool is built using **SQLAlchemy**, which provides a unified interface for connecting to different SQL databases (PostgreSQL, MySQL, SQLite, Oracle) with minimal changes to the code.
> * Database connection details (host, user, password, port, database name, and type) are defined in the **Valves** class. These values can be adjusted from the Open WebUI interface without modifying the code itself.
> * The `_get_engine()` method dynamically constructs the correct connection string based on the selected database type and returns a reusable SQLAlchemy engine.
> * The `execute_read_query` function strictly enforces **read-only access**. Only SELECT-style queries are allowed, and potentially destructive SQL keywords (e.g., INSERT, UPDATE, DELETE, DROP) are explicitly blocked.

### 2. Configuring the Connection (The Valves)
Now that we have added the tool, all that remains is to specify the details of our SQL server in the valves. Then, we can use our LLM to execute SQL queries of our liking.

{: .action}
> In your **Workspace**, under **Tools**, click on the gear icon to adjust the valves. Specify the details below (case sensitive!):
> * **Db Host:** 91.92.143.253
> * **Db User:** participant
> * **Db Password:** _See password on the screen in the room_
> * **Db Name:** soccer_db
> * **Db Port:** 5432
> * **Db Type:** postgresql

### 3. System Prompting
Tools are "passive" - the LLM often forgets to use them or refuses to run code due to safety training. We must give it a specific task description to force it to behave like a Data Analyst. Select and copy one of the prompts from the _Prompt Library_. Click [the link](https://docs.google.com/document/d/1jJExGSSCe7ub5EiPJOFTh6FkvOYtRDyHruoi97CqC5A/edit?usp=sharing) to see the different system prompt options.

{: .action}
> 1. When you are in a chat, go to your Chat Controls (top right settings icon).
> 2. Paste the system prompt you chose into the System Prompt field.
> 3. Click Save (or close the panel).

### 4. Testing the Connection
Now you're tool is all ready! It is time to start experimenting and evaluate how well your LLM can query the database. Have a look at the [database](https://www.kaggle.com/datasets/hugomathien/soccer), and think about what questions you may ask.

{: .action}
> When starting a new chat, in your chat window, click on _Integration -> Tools_, and toggle on your tool. Now the access to the SQL database is enabled. You can now ask the LLM questions that can be answered based on the database. Try it out, how well does it perform? In what cases does it not behave as you expected? Below are a few questions you can test
> * **Discovery:** _"List all the tables in the database."_ (Tests connection)
> * **Schema:** _"What columns are in the `player` table?"_ (Tests schema reading)
> * **Data Retrieval:** __"Get me the height and weight of player 'Aaron Appindangoye'. Do not guess."_ (Tests full execution)

{: .note}
> Important: SQL is case-sensitive. The table is likely named player, not Player.

## Additional exercise: Launching Your Own SQL Server
In the previous part, you were using a SQL database that we are running using PostgreSQL. Now, you will launch your own SQL database, using SQLite. Unlike PostgreSQL or MySQL, SQLite does not run as a separate server. Instead, the entire database lives in a single .sqlite file. This makes SQLite ideal for lightweight experiments and local development. We will first set up our environment on our compute instance, after which we pull a database from Kaggle.

{: .action}
> You will first need to make an account on [Kaggle](https://kaggle.com), and set up an API key. Once you made your account, click on your profile picture at the top right to go to the settings. In the settings, generate an **API token** and save it carefully, you will need it to pull a dataset to your compute instance.

Now, we will guide you through the steps to launch your own SQL server and add it to your Open WebUI. Follow the steps below carefully.

{: .action}
> Follow the steps below to prepare your environment with `uv`. For each step, copy the code and paste and execute it on the command line of your compute instance.

* **Step 1:** Install the `uv` environment manager:
```
curl -LsSf https://astral.sh/uv/install.sh | sh
source $HOME/.local/bin/env
```

* **Step 2:** Initialize your environment:
```
cd ~
uv init sql
cd sql
source .venv/bin/activate
uv pip install kaggle
```

* **Step 3:** Configure your Kaggle API credentials. 
    * First make the directory: `mkdir -p ~/.kaggle`
    * Now create the file: `nano ~/.kaggle/kaggle.json`
    * Add the credentials in this format: `{"username":"<your_username>","key":"<your_api_key>"}`
    * Set the permissions correctly: `chmod 600 ~/.kaggle/kaggle.json`

* **Step 4:** Download a specific dataset, for example [tweets on US airline sentiment](https://www.kaggle.com/datasets/crowdflower/twitter-airline-sentiment?select=database.sqlite)
```
kaggle datasets download -d crowdflower/twitter-airline-sentiment
sudo apt install unzip
unzip twitter-airline-sentiment.zip
```
* **Step 5:** Install sqlite on your compute instance, in case you want to explore queries locally. Then copy the database to your Open WebUI backend.
```
sudo apt-get install sqlite3 -y
sudo cp database.sqlite ../data_openwebui/
```

* **Step 6:** Adjust the valves of your tool to point to your local database. Specifically, you should specify:
    * **Db Host:** localhost
    * **Db User:** admin (default value)
    * **Db Password:** admin (default value)
    * **Db Name:** /app/backend/data/database.sqlite
    * **Db Port:** 3306 (default value)
    * **Db Type:** sqlite
 
* **Step 7:** Explore the newly linked SQL database! You are now exploring an SQL database hosted on your own compute instance. What can you find out about the data? You can still not modify your data, due to the constraints in the tool code. If you wish, you can try to alter the tool code to allow yourself to modify the database. 

{: .note}
> You may have to help the LLM to find the right column names, in order to compose the correct queries. Below, you can see an overview of the columns of the database. For more information on the dataset, visit the [Kaggle page](https://www.kaggle.com/datasets/crowdflower/twitter-airline-sentiment?select=database.sqlite).

| Column Name | Description |
|-------------|-------------|
| tweet_id | Unique identifier for each tweet (Primary Key) |
| airline_sentiment | Sentiment classification of the tweet (positive, negative, or neutral) |
| airline_sentiment_confidence | Confidence score for the sentiment classification |
| negativereason | Specific reason for negative sentiment (e.g., "late flight", "bad service") |
| negativereason_confidence | Confidence score for the negative reason classification |
| airline | Name of the airline mentioned in the tweet |
| airline_sentiment_gold | Gold standard/manually verified sentiment label (for validation) |
| name | Twitter username of the person who posted the tweet |
| negativereason_gold | Gold standard/manually verified negative reason (for validation) |
| retweet_count | Number of times the tweet was retweeted |
| text | The actual tweet content/message |
| tweet_coord | Geographic coordinates where the tweet was posted |
| tweet_created | Timestamp when the tweet was created |
| tweet_location | Location information from the user's profile |
| user_timezone | Timezone setting of the user who posted the tweet |


## Next Step
Next, we will explore tools and MCP. Go to the next section [here](../MCP/quickstart.md).

_Author: [Alexander Sternfeld](https://ch.linkedin.com/in/alexander-sternfeld-93a01799)_
