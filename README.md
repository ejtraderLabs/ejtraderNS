
![Pypi Publish](https://github.com/ejtraderLabs/ejtraderNS/actions/workflows/python-publish.yml/badge.svg)
![GitHub release (latest by date)](https://img.shields.io/github/v/release/ejtraderLabs/ejtraderNS)
[![License](https://img.shields.io/github/license/ejtraderLabs/ejtraderNS)](https://github.com/ejtraderLabs/ejtraderNS/blob/main/LICENSE)


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

There is a limited set of topic that you might find:

``` 'tech', 'news', 'business', 'science', 'finance', 'food', 'politics', 'economics', 'travel', 'entertainment', 'music', 'sport', 'world' ```


### some url supports multiple languages
```python
import pandas as pd
from ejtraderNS import Client, describe_url
from datetime import datetime

country_topic = describe_url('investing.com')

# Criando uma lista vazia para armazenar os DataFrames
dfs = []

# Iterando pelos países, idiomas e tópicos

    
for topic in country_topic['topics']:
    try:
        api = Client(website="investing.com", topic=topic, country="BR")
    except:
        pass
    print(f"topic: {topic}")
    
    try:
        getdata = api.get_news()
    except:
        pass
    # Coletando os dados
    

    # Se getdata for None, pule para o próximo tópico
    if getdata is None:
        continue

    # Criando uma lista vazia para armazenar as informações
    data = []

    # Iterando pelos artigos e extraindo as informações relevantes
    for article in getdata['articles']:
        article_data = {}
        article_data['topic'] = getdata['topic']
        article_data['author'] = article['author']
        article_data['date'] = article['published_parsed'] if article['published_parsed'] else article['published']

        article_data['country'] = getdata['country']
        article_data['language'] = getdata['language']

        article_data['title'] = article['title']
        
        try:
            article_data['summary'] = article['summary']
            article_data['url'] = article['link']
        except:
            article_data['url'] = None
            article_data['summary'] = article['title']
            pass
        
        data.append(article_data)

    # Converter objetos time.struct_time para objetos datetime
    for item in data:
        try:
            item['date'] = datetime(*item['date'][:6])
        except:
            pass
    # Criando o dataframe com as informações extraídas
    df = pd.DataFrame(data)
    df['date'] = pd.to_datetime(df['date'], utc=True)
    df.set_index('date', inplace=True)

    # Adicionando o DataFrame atual à lista de DataFrames
    dfs.append(df)

# Concatenando todos os DataFrames na lista dfs
df = pd.concat(dfs)

# Reordenando o índice
df.sort_index(inplace=True)
print(df)

```

for investing.com  is a limited set of topic base on country 

``` 'news', 'crypto_news', 'forex_news', 'popular_news', 'commodities_news', 'stock_news', 'economic_indicators_news', 'economy_news', 'central_bank', 'crypto_opinion', 'forex_analysis', 'forex_technical', 'forex_fundamental', 'forex_opinion', 'forex_signal', 'overview_analysis', 'overview_technical', 'overview_fundamental', 'overview_opinion', 'overview_investing', 'commodities_analysis', 'commodities_technical', 'commodities_Fundamental', 'commodities_opinion', 'commodities_strategy', 'commodities_metals', 'commodities_energy', 'commodities_agriculture', 'bonds_analysis', 'bonds_technical', 'bonds_fundamental', 'bonds_opinion', 'bonds_trategy', 'bonds_government', 'bonds_corporate', 'stock_analysis', 'stock_technical', 'stock_fundamental', 'stock_opinion', 'stock_picks', 'stock', 'indices', 'futures', 'options', 'politics_news', 'world_news' ```

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


## About Us
We are ejtraderNS API team. We are glad that you liked our package.

If you want to search for any news data, consider using [our API](https://ejtraderNSapi.com/)

![](ejtraderNS_oneliner.png)


[Artem Bugara]() - co-founder of ejtraderNS, made v.0.1.0

[Maksym Sugonyaka](https://www.linkedin.com/mwlite/in/msugonyaka) - co-founder of ejtraderNS, made v.0.1.0

[Becket Trotter](https://www.linkedin.com/in/beckettrotter/) - Python Developer, made v.0.2.0

## Licence
MIT
