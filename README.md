
![Pypi Publish](https://github.com/ejtraderLabs/ejtraderNS/actions/workflows/python-publish.yml/badge.svg)
[![PyPI version](https://badge.fury.io/py/ejtraderns.svg)](https://badge.fury.io/py/ejtraderNS)
[![PyPi downloads](https://img.shields.io/pypi/dm/ejtraderns?style=flat-square&logo=pypi&logoColor=white)](https://pypi.org/project/ejtraderns/)
![GitHub release (latest by date)](https://img.shields.io/github/v/release/ejtraderLabs/ejtraderNS)
[![License](https://img.shields.io/github/license/ejtraderLabs/ejtraderNS)](https://github.com/ejtraderLabs/ejtraderNS/blob/main/LICENSE)
[![Contributions welcome](https://img.shields.io/badge/contributions-welcome-brightgreen.svg)](CONTRIBUTING.md)
# ejtraderNS
**Programmatically collect normalized news from (almost) any website.**

Filter by **topic**, **country**, or **language**.

## Installation
`pip install ejtraderNS --upgrade` 


## Quick Start
```python
from ejtraderNS import Client

```

Get the latest news from [nytimes.com](https://www.nytimes.com/) 
(_we support thousands of news websites, try yourself!_) main news feed
```python
api = ejtraderNS(website = 'nytimes.com')
results = api.get_news()

# results.keys()
# 'url', 'topic', 'language', 'country', 'articles'

# Get the articles
articles = results['articles']

first_article_summary = articles[0]['summary']
first_article_title = articles[0]['title']
```

Get the latest news from [nytimes.com](https://www.nytimes.com/) **politics** feed

```python
api = ejtraderNS(website = 'nytimes.com', topic = 'politics')

results = api.get_news()
articles = results['articles']
```

Some websites support multiple countries, such as  [investing.com](https://www.investing.com) or [tradingeconomics.com](https://www.tradingeconomics.com)


In this example, I will demonstrate a website that supports multiple countries,
 retrieve multiple topics, and convert the data into a pandas dataframe.


```python
import pandas as pd
from ejtraderNS import Client
from datetime import datetime

url = 'investing.com' # or tradingeconomics.com
country = 'GB'
country_topic = ["finance","news","economics"]
dfs = []

for topic in country_topic:
    api = Client(website=url, topic=topic, country=country)
    getdata = api.get_news()
    print(f"topic: {topic}")

    if getdata is None:
        continue

    data = []

    for article in getdata['articles']:
        article_data = {
            'topic': getdata['topic'],
            'author': article['author'],
            'date': article['published_parsed'] if article['published_parsed'] else article['published'],
            'country': getdata['country'],
            'language': getdata['language'],
            'title': article['title'],
            'summary': article.get('summary', article['title'])
        }
        data.append(article_data)

    df = pd.DataFrame(data)

    df['date'] = pd.to_datetime(df['date'].apply(lambda x: datetime(*x[:6]) if isinstance(x, tuple) else x), utc=True, errors='coerce')
    df.set_index('date', inplace=True)
    dfs.append(df)

df = pd.concat(dfs)
df.sort_index(inplace=True)
print(df)

```
output example:

| topic     | author        | country   | language   | title                                                                            | summary                                                                          |
|:----------|:--------------|:----------|:-----------|:---------------------------------------------------------------------------------|:---------------------------------------------------------------------------------|
| finance   | Reuters       | GB        | en         | Italy pushes to limit executive pay in listed state-run firms                    | Italy pushes to limit executive pay in listed state-run firms                    |
| economics | Reuters       | GB        | en         | UK's Cleverly raises Xinjiang and Taiwan with Chinese vice president             | UK's Cleverly raises Xinjiang and Taiwan with Chinese vice president             |
| news      | Reuters       | GB        | en         | Ukraine hails return of 45 Azov fighters, Russia says 3 pilots released          | Ukraine hails return of 45 Azov fighters, Russia says 3 pilots released          |





There is a limited set of topic that you might find:
``` 'tech', 'news', 'business', 'science', 'finance', 'food', 'politics', 'economics', 'travel', 'entertainment', 'music', 'sport', 'world' ```

extras topics only for [investing.com](https://www.investing.com)

``` 'crypto', 'forex', 'stock', 'commodities', 'central_bank', 'forex_analysis', 'forex_technical', 'forex_fundamental', 'forex_opinion', 'forex_signal', 'bonds_analysis', 'bonds_technical', 'bonds_fundamental', 'bonds_opinion', 'bonds_strategy', 'bonds_government', 'bonds_corporate', 'stock_analysis', 'stock_technical', 'stock_fundamental', 'stock_opinion', 'stock_picks', 'indices_analysis', 'futures_analysis', 'options_analysis', 'commodities_analysis', 'commodities_technical', 'commodities_Fundamental', 'commodities_opinion', 'commodities_strategy', 'commodities_metals', 'commodities_energy', 'commodities_agriculture', 'overview_analysis', 'overview_technical', 'overview_fundamental', 'overview_opinion', 'overview_investing', 'crypto_opinion'```
  




However, not all topics are supported by every newspaper.

How to check which topics are supported by which newspaper:
```python
from ejtraderNS import describe_url

describe = describe_url('nytimes.com')

print(describe['topics'])
```


### Get the list of all news feeds by topic/language/country
If you want to find the full list of supported news websites 
you can always do so using `urls()` function
```python
from ejtraderNS import urls

# URLs by TOPIC
politic_urls = urls(topic = 'politics')

# URLs by COUNTRY
american_urls = urls(country = 'US')

# URLs by LANGUAGE
english_urls = urls(language = 'en')

# Combine any from topic, country, language
american_english_politics_urls = urls(country = 'US', topic = 'politics', language = 'en') 

# note some websites do not explicitly declare their language 
# as a result they will be excluded from queries based on language
```




## Documentation

### `ejtraderNS` Class
```python
from ejtraderNS import Client

Client(website, topic = None)
```
**Please take the base form url of a website** (without `www.`,neither `https://`, nor `/` at the end of url).

For example: “nytimes”.com, “news.ycombinator.com” or “theverge.com”.
___
`Client.get_news()` - Get the latest news from the website of interest.

Allowed topics:
`tech`, `news`, `business`, `science`, `finance`, `food`, 
`politics`, `economics`, `travel`, `entertainment`, 
`music`, `sport`, `world`

If no topic is provided, the main feed is returned.

Returns a dictionary of 5 elements:
1. `url` - URL of the website
2. `topic` - topic of the returned feed
3. `language` - language of returned feed
4. `country` - country of returned feed
5. `articles` - articles of the feed. [Feedparser object]((https://pythonhosted.org/feedparser/reference.html))

___

`Client.get_headlines()` - Returns only the headlines

___
`Client.print_headlines(n)` - Print top `n` headlines


<br> 
<br> 
<br> 

### `describe_url()` & `urls()`
Those functions exist to help you navigate through this package

___
```python
from ejtraderNS import describe_url
```

`describe_url(website)` - Get the main info on the website. 

Returns a dictionary of 5 elements:
1. `url` - URL of the website
2. `topics` - list of all supported topics
3. `language` - language of website
4. `country` - country of returned feed
5. `main_topic` - main topic of a website

___
```python
from ejtraderNS import urls
```

`urls(topic = None, language = None, country = None)` - Get a list of all supported 
news websites given any combination of `topic`, `language`, `country`

Returns a list of websites that match your combination of `topic`, `language`, `country`

Supported topics:
`tech`, `news`, `business`, `science`, `finance`, `food`, 
`politics`, `economics`, `travel`, `entertainment`, 
`music`, `sport`, `world`


Supported countries:
`US`, `GB`, `DE`, `FR`, `IN`, `RU`, `ES`, `BR`, `IT`, `CA`, `AU`, `NL`, `PL`, `NZ`, `PT`, `RO`, `UA`, `JP`, `AR`, `IR`, `IE`, `PH`, `IS`, `ZA`, `AT`, `CL`, `HR`, `BG`, `HU`, `KR`, `SZ`, `AE`, `EG`, `VE`, `CO`, `SE`, `CZ`, `ZH`, `MT`, `AZ`, `GR`, `BE`, `LU`, `IL`, `LT`, `NI`, `MY`, `TR`, `BM`, `NO`, `ME`, `SA`, `RS`, `BA`

Supported languages:
`EL`, `IT`, `ZH`, `EN`, `RU`, `CS`, `RO`, `FR`, `JA`, `DE`, `PT`, `ES`, `AR`, `HE`, `UK`, `PL`, `NL`, `TR`, `VI`, `KO`, `TH`, `ID`, `HR`, `DA`, `BG`, `NO`, `SK`, `FA`, `ET`, `SV`, `BN`, `GU`, `MK`, `PA`, `HU`, `SL`, `FI`, `LT`, `MR`, `HI`



## Tech/framework used
The package itself is nothing more than a SQLite database with 
RSS feed endpoints for each website and some basic wrapper of
[feedparser](https://pythonhosted.org/feedparser/index.html).


## Acknowledgements

I would like to express my gratitude to [@kotartemiy](https://github.com/kotartemiy) for creating the initial project. Their work has been an invaluable starting point for my modifications and improvements.

