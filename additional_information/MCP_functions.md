---
parent: "Additional Information"
title: "MCP tool code"
nav_order: 4
---

## MCP tool code
In the [MCP quickstart](../MCP/quickstart.md) you saw the implementation of a tool for the analysis of a small CSV file. Here, we explain the code in more detail. 

### Code explanation
We will now explain the code that is used for the tool server, as seen in the [section on MCPs](../MCP/quickstart.md) and explain it block by block. 

First, we import all the necessary libraries
```python
from fastmcp import FastMCP
import pandas as pd
from datetime import datetime, date
import requests
import io
```

Next, we initialize the tool server. Note that we use the [FastMCP](https://github.com/jlowin/fastmcp) library, which is one of the most popular frameworks for MCP applications. We choose to use port 8000 for access to the tool server.
```python
# Initialize FastMCP
mcp = FastMCP("user-tools-demo", port=8000)
mcp.settings.host = "0.0.0.0"
```

Now, we download the CSV file with employee data. For this we use the `requests` package, and make sure to have a verbose error if it fails.
```python
# Download CSV from Google Drive
CSV_URL = "https://drive.google.com/uc?id=1VEi-dnEh4RbBKa97fyl_Eenkvu2NC6ki&export=download"
try:
    response = requests.get(CSV_URL)
    response.raise_for_status()
    df = pd.read_csv(io.StringIO(response.text))
except Exception as e:
    raise RuntimeError(f"Failed to download CSV: {e}")
```

We make sure to preprocess the columns, specifically we lowercase all the data and strip it of whitespaces. This avoids potential mismatches due to capitalization or trailing whitespaces.
```python
# Preprocess columns
df['First Name'] = df['First Name'].str.strip().str.lower()
df['Last Name'] = df['Last Name'].str.strip().str.lower()
df['Job Title'] = df['Job Title'].str.strip().str.lower()
df['Sex'] = df['Sex'].str.strip().str.lower()
df['Email'] = df['Email'].str.strip().str.lower()
df['Phone'] = df['Phone'].str.strip()

# Convert Date of birth to datetime
df['Date of birth'] = pd.to_datetime(df['Date of birth'], errors='coerce')
```

Now, we can make the two functions. For this, we leverage the ability to filter pandas dataframes based on attribute values. In case you are unfamiliar with Pandas, we recommend [this tutorial](https://www.datacamp.com/tutorial/pandas-tutorial-dataframe-python).
```python
@mcp.tool
def count_by_first_name(first_name: str):
    """
    Count the number of males and females with a given first name.

    Parameters:
    - first_name (str): The first name to search for (e.g., 'Alice'). Case-insensitive.

    Returns:
    - dict: {"male": int, "female": int}
    """
    first_name = first_name.lower()
    males = df[(df['First Name'] == first_name) & (df['Sex'] == 'male')].shape[0]
    females = df[(df['First Name'] == first_name) & (df['Sex'] == 'female')].shape[0]
    return f"There are {males} men with the first name {first_name} and {females} women with the first name {first_name}"

@mcp.tool
def count_by_job_keyword(keyword: str):
    """
    Count the number of males and females whose job title contains a given keyword.

    Parameters:
    - keyword (str): Keyword to search for in job titles (e.g., 'engineer'). Case-insensitive.

    Returns:
    - dict: {"male": int, "female": int}
    """
    keyword = keyword.lower()
    males = df[(df['Job Title'].str.contains(keyword, na=False)) & (df['Sex'] == 'male')].shape[0]
    females = df[(df['Job Title'].str.contains(keyword, na=False)) & (df['Sex'] == 'female')].shape[0]
    return f"There are {males} with a job containing the keyword {keyword}, and {females} women with a job with the keyword {keyword}"
```

Now we launch the server! We set the mode to _streamable-http_, meaning that the LLMs can both send and receive data to the tool server over HTTP.
```python
if __name__ == "__main__":
    mcp.run(transport="streamable-http")
```

_Author: [Alexander Sternfeld](https://ch.linkedin.com/in/alexander-sternfeld-93a01799)_
