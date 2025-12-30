---
title: "Your own MCP"
parent: "Model Context Protocol (MCP)"
nav_order: 2
---

Now that you have gained an initial idea of how MCPs work through a simple example, it is time to explore them further. Here we provide you with several ideas to gain more experience with building your own MCP. Feel free to choose the option that is most interesting to you, as you can always come back later to practice with the other options. Here are the possible directions:

1. Extend the MCP for a user database that we discussed in the [Quickstart](quickstart.md).
2. Build a MCP based upon a different public dataset. 
3. Build a MCP unrelated to a dataset

{: .tip}
> For each of the options, you will be writing your own functions for the MCPs. The following tips may be helpful for each of the exercises.
> * When composing functions in a MCP, it is vital that the functions are as descriptive as possible. If not, the LLM may misunderstand how the function works and not use it correctly. Pay attention to the following things:
>   - Add the type to each of the input variables of the function (e.g. `n: int = 5`, instead of `n = 5`).
>   - At the beginning of each function, add descriptive documentation. Have a look at the functions in the [Quickstart](quickstart.md) for examples.
>   - The function should return a descriptive sentence. To illustrate, a function should return _"There are 50 men in the dataset"_, instead of returning _50_, for the LLM to understand what is happening.
> * If you wish to test a python function, we recommend you to do so locally. This can either be in your IDE of preference (e.g. VSCode), or thro
> * When launching a tool server, you can use `tmux` to avoid it blocking your terminal, as discussed in the [Quickstart](quickstart.md). Alternatively, you can simply log onto your server again in a second terminal. 

## Extend the user database MCP
In this exercise, you will add more functionalities to the MCP for a user database that we constructed in the [Quickstart](quickstart.md). You can choose the new functionality yourself, based on the attributes available in the dataset. Alternatively, you can choose one of the suggestions below. We recommend you to try and compose the functions yourself, to gain experience with setting up such a MCP. Once you are done, or if you are short on time, you can verify your work with the solution that is available below. If you run into persistent problems, feel free to ask for help to one of the workshop organizers. Good luck!

* Find the most common first and last names in the dataset, both for men and women.
* For a given e-mail domain, count the number of men and women using this domain.
* Get a random sample of users from the dataset.

<details>
<summary>Show solution</summary>

<pre><code class="language-python">
@mcp.tool
def count_by_age_and_job(min_age: int, max_age: int, keyword: str):
    """
    Count the number of males and females within a specific age range who have a job title containing a given keyword.

    Parameters:
    - min_age (int): Minimum age (inclusive)
    - max_age (int): Maximum age (inclusive)
    - keyword (str): Keyword to search for in job titles (e.g., 'manager'). Case-insensitive.

    Returns:
    - dict: {"male": int, "female": int}
    """
    keyword = keyword.lower()
    filtered = df[
        (df['Age'] >= min_age)
        & (df['Age'] <= max_age)
        & (df['Job Title'].str.contains(keyword, na=False))
    ]
    males = filtered[filtered['Sex'] == 'male'].shape[0]
    females = filtered[filtered['Sex'] == 'female'].shape[0]
    return (
        f"There are {males} men and {females} women within the ages "
        f"{min_age} and {max_age} with a job title containing the keyword "
        f"{keyword}."
    )


@mcp.tool
def most_common_names(gender: str, top_n: int = 10):
    """
    Return the most common first and last names for a given gender.

    Parameters:
    - gender (str): "male" or "female" (case-insensitive). Accepts variations like "man", "woman", "f", "m".
    - top_n (int): Number of top names to return (default: 10)

    Returns:
    - dict: {"first_names": {name: count}, "last_names": {name: count}}
    """
    gender = normalize_gender(gender)
    filtered = df[df['Sex'] == gender]
    first_names = filtered['First Name'].value_counts().head(top_n).to_dict()
    last_names = filtered['Last Name'].value_counts().head(top_n).to_dict()
    return (
        f"The top {top_n} most common first names for the gender {gender} "
        f"are {first_names}, the most common last names are {last_names}."
    )


@mcp.tool
def email_domain_stats(domain: str):
    """
    Count the number of males and females using a specific email domain.

    Parameters:
    - domain (str): Email domain to search for (e.g., 'gmail.com'). Case-insensitive.

    Returns:
    - dict: {"male": int, "female": int, "total": int}
    """
    domain = domain.lower()
    filtered = df[df['Email'].str.endswith(domain)]
    males = filtered[filtered['Sex'] == 'male'].shape[0]
    females = filtered[filtered['Sex'] == 'female'].shape[0]
    return (
        f"There are in total {filtered.shape[0]} users with emails ending "
        f"with {domain}, of which {males} men and {females} women."
    )


@mcp.tool
def random_sample(n: int = 5, gender: str = None):
    """
    Return a random sample of users, optionally filtered by gender.

    Returns:
    - dict:
        {
          "description": str,
          "count": int,
          "users": list[dict]
        }
    """
    sample_df = df
    gender_label = "all users"

    if gender:
        gender = normalize_gender(gender)
        sample_df = sample_df[sample_df['Sex'] == gender]
        gender_label = f"{gender} users"

    total_available = sample_df.shape[0]
    sample_size = min(n, total_available)
    sampled = sample_df.sample(sample_size)

    users = sampled[
        ['First Name', 'Last Name', 'Age', 'Job Title']
    ].to_dict(orient='records')

    description = (
        f"Here is a random sample of {sample_size} {gender_label} "
        f"out of {total_available} matching users."
    )

    return {
        "description": description,
        "count": sample_size,
        "users": users,
    }
</code></pre>

</details>


{: .note}
> When you have added new functions to the `tools.py` file, and you have started the tool server, do not forget to add the function names in the Open WebUI instance. Specifically, in the _Function Name Filter List_. When in doubt, you can go through the [Quickstart](quickstart.md) again to look at the sequence of actions for launching a MCP. 

## Build a MCP on a different data source
Before, you made a MCP with tools for analysing a toy CSV data file. Now, you can take the time to build a MCP server with tools for a more realistic dataset. Below, we provide several suggestions, but you can also choose a dataset of your own. Given that this is an open exercise, we do not provide a sample solution here. If you do wish to do an exercise with an available solution, have a look at the first or third option on this page. If you wish to do build another MCP using a small toy dataset, there are different CSV files available [here](https://www.datablist.com/learn/csv/download-sample-csv-files). Several suggestions for real datasets are:

* **Data from the stock market:** through [Stooq](https://stooq.com/db/h/) you can obtain free historical market data. To illustrate, `https://stooq.com/q/d/l/?s=msft.us&i=d` would give you daily data on the Microsoft stock (abbreviation `msft`). Using different abbreviations, you can obtain data for different stocks.
* **Covid data**: through [this link](https://data.rivm.nl/data/covid-19/COVID-19_aantallen_gemeente_cumulatief.csv), you can obtain data on Covid cases in the Netherlands.

{: .tip}
> You can download data in a python file by using the `requests` library, of

## LLM- or API-powered tools
The third option is to build a MCP server with LLM-powered tools. You can use the Ollama server that is spinning for your Open WebUI instance to power the tools in your MCP server. Below, you will find a template for the python file that you can use to launch your tool server. Now you can be creative, and construct your own LLM-powered tools. If you would like some inspiration, below are several options. We recommend you to try to independently go through these exercises. When you want, you can view the solution for each of the suggested tools below the template. 

* Compose a tool chaining multiple LLM calls, such as first summarizing a piece of text and then rewriting it in a specific style (e.g. humorous).
* Compose a tool that verifies whether the prompt violates pre-specified guidelines. 
* Compose a tool for getting the weather forecast in a specific city, using [this API](https://github.com/chubin/wttr.in), where we suggest to use the JSON output format of the API.


```python
import requests
from fastmcp import FastMCP
 
OLLAMA_URL = "http://127.0.0.1:11434/api/generate"
MODEL = "llama3.1:8b"

# Initialize FastMCP
mcp = FastMCP("llm-and-api-tools", port=8001)
mcp.settings.host = "0.0.0.0"

def ollama_generate(
    prompt: str,
    system: str | None = None,
    temperature: float = 0.7,
    max_tokens: int = 512,
) -> str:
    payload = {
        "model": MODEL,
        "prompt": prompt,
        "options": {
            "temperature": temperature,
            "num_predict": max_tokens,
        },
        "stream": False,
    }

    if system:
        payload["system"] = system

    response = requests.post(OLLAMA_URL, json=payload, timeout=60)
    response.raise_for_status()

    return response.json()["response"]

######### Tools #############
# To complete
```
    
