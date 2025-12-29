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
> For each of the options, you will be writing your own functions for the MCPs. The following two tips will be helpful for each of the exercises.
> * When composing functions in a MCP, it is vital that the functions are as descriptive as possible. If not, the LLM may misunderstand how the function works and not use it correctly. Pay attention to the following things:
>   * Add the type to each of the input variables of the function (e.g. `n: int = 5`, instead of `n = 5`).
>   * At the beginning of each function, add descriptive documentation. Have a look at the functions in the [Quickstart](quickstart.md) for examples.
>   * The function should return a descriptive sentence. To illustrate, a function should return _"There are 50 men in the dataset"_, instead of returning _50_, for the LLM to understand what is happening.
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
There are different CSV files available [here](https://www.datablist.com/learn/csv/download-sample-csv-files), but you can also choose a differen type of data.

## Other MCP's
