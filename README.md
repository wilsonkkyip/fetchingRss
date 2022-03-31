Acquiring RSS Feeds
================

## Introduction

This document aims to describe how to read RSS file from the web.
[RSS](https://en.wikipedia.org/wiki/RSS) is a web feed that allows users
and applications to access updates to websites in a standardized,
computer-readable format. It is mainly used to read updated news or blog
posts.

### Example RSS File

An example of an RSS file is shown below

``` html
<?xml version="1.0" encoding="UTF-8" ?>
<rss version="2.0">
<channel>
  <title>RSS Title</title>
  <description>This is an example of an RSS feed</description>
  <link>http://www.example.com/main.html</link>
  <copyright>2020 Example.com All rights reserved</copyright>
  <lastBuildDate>Mon, 06 Sep 2010 00:01:00 +0000</lastBuildDate>
  <pubDate>Sun, 06 Sep 2009 16:20:00 +0000</pubDate>
  <ttl>1800</ttl>

  <item>
    <title>Example entry</title>
    <description>Here is some text containing an interesting description.</description>
    <link>http://www.example.com/blog/post/1</link>
    <guid isPermaLink="false">7bd204c6-1655-4c27-aeee-53f933c5395f</guid>
    <pubDate>Sun, 06 Sep 2009 16:20:00 +0000</pubDate>
  </item>

</channel>
</rss>
```

### Common RSS sources

Below shows some common RSS sources

| Source         | URL                                                      |
|:---------------|:---------------------------------------------------------|
| BBC            | <https://feeds.bbci.co.uk/news/rss.xml>                  |
| CNN            | <http://rss.cnn.com/rss/cnn_topstories.rss>              |
| Foreign Policy | <https://foreignpolicy.com/feed/>                        |
| New York Times | <https://rss.nytimes.com/services/xml/rss/nyt/World.xml> |

## Fetching RSS Feeds

### In Python

The script below fetches the RSS feeds into a `Pandas` data frame.

``` python
def removeHTMLtag(x):
    import re
    return re.sub("<.*?>", "", x)

def removeWhiteSpace(x):
    import re
    return re.sub("\s+", "", x)

def getNodes(x, y):
    import re
    result = re.findall("<%s.*?>(.*?)</%s>" % (y, y), x)
    if len(result) == 1: 
        return result[0]
    else:
        return result

def getCDATA(x):
    import re
    result = re.sub("^<!\[CDATA\[", "", x)
    result = re.sub("\]\]>$", "", result)
    return result

def getResponse(url):
    import requests
    import pandas as pd
    from datetime import datetime
    from urllib.parse import urlparse
    from xml.sax.saxutils import unescape
    
    urlParse = urlparse(url)
    # The problem of beautifulSoup is that its removes all the content in <title> and <description>
    response = requests.get(url).content
    response = response.decode("utf-8").replace("\n", "")
    items = getNodes(response, "item")
    
    df = [["datetime", "title", "description", "guid", "links"]]
    for item in items:
        
        date_ = getNodes(item, "pubDate")
        if len(date_) == 0:
            date_ = datetime.now().strftime("%s")
        else:
            if urlParse.hostname in ["rss.nytimes.com", "foreignpolicy.com"]:
                date_ = datetime.strptime(date_, "%a, %d %b %Y %H:%M:%S %z").strftime("%s")
            else:
                date_ = datetime.strptime(date_, "%a, %d %b %Y %H:%M:%S %Z").strftime("%s")
        
        title_ = getCDATA(getNodes(item, "title"))
        description_ = unescape(getCDATA(getNodes(item, "description")))
        description_ = removeHTMLtag(description_).strip()
        guid = getNodes(item, "guid")
        link_ = getNodes(item, "link")
        
        df.append([date_, title_, description_, guid, link_])
    
    df = pd.DataFrame(df[1:], columns = df[0])
    
    return df
```

#### Example

``` python
def getUrls():
    result = {
        "bbc": "https://feeds.bbci.co.uk/news/rss.xml",
        "cnn": "http://rss.cnn.com/rss/cnn_topstories.rss",
        "fp": "https://foreignpolicy.com/feed/",
        "nytimes": "https://rss.nytimes.com/services/xml/rss/nyt/World.xml"
    }
    return result


urls = getUrls()
feeds = {k:getResponse(x) for (k, x) in urls.items()}
```

Below shows the RSS feeds from BBC.

| datetime            | title                                                                           | description                                                                                              | guid                                                          | links                                                                                           |
|:--------------------|:--------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------|:------------------------------------------------------------------------------------------------|
| 2022-03-31 14:27:38 | Putin sets midnight deadline over rouble gas payments                           | Vladimir Putin signs a decree ordering foreign buyers of Russian gas to open Russian bank accounts.      | <https://www.bbc.co.uk/news/business-60945248>                | <https://www.bbc.co.uk/news/business-60945248?at_medium=RSS&at_campaign=KARANGA>                |
| 2022-03-31 14:19:11 | Ukraine war: Ukraine sends buses to Mariupol for rescue effort                  | A convoy of 45 buses is heading for the besieged city in the latest attempt to rescue trapped civilians. | <https://www.bbc.co.uk/news/world-europe-60938429>            | <https://www.bbc.co.uk/news/world-europe-60938429?at_medium=RSS&at_campaign=KARANGA>            |
| 2022-03-31 08:46:48 | Chris Rock says he is still processing Smith slap                               | The comedian says he is “still processing what happened” and denies reports he has spoken to Smith.      | <https://www.bbc.co.uk/news/entertainment-arts-60939316>      | <https://www.bbc.co.uk/news/entertainment-arts-60939316?at_medium=RSS&at_campaign=KARANGA>      |
| 2022-03-31 11:34:04 | UK weather: Spring snow as parts of country hit by cold snap                    | “Spring is on hold,” says BBC Weather, in a sharp change to last week which saw highs of 20C.            | <https://www.bbc.co.uk/news/uk-60941529>                      | <https://www.bbc.co.uk/news/uk-60941529?at_medium=RSS&at_campaign=KARANGA>                      |
| 2022-03-31 14:12:14 | Energy websites crash in meter readings rush                                    | Suppliers including Shell and EDF are working to resolve issues, ahead of a price rise on Friday.        | <https://www.bbc.co.uk/news/business-60938730>                | <https://www.bbc.co.uk/news/business-60938730?at_medium=RSS&at_campaign=KARANGA>                |
| 2022-03-31 12:02:25 | Lincoln City to silence Dambusters March and air raid sirens due to Ukraine war | Lincoln City has asked supporters to support the move, sparked by Russia’s invasion of Ukraine.          | <https://www.bbc.co.uk/news/uk-england-lincolnshire-60938930> | <https://www.bbc.co.uk/news/uk-england-lincolnshire-60938930?at_medium=RSS&at_campaign=KARANGA> |

### In R

The script below fetches the RSS feeds into R data frame.

``` r
removeHTMLtag <- function(x) gsub("<.*?>", "", x)
removeNextLine <- function(x) gsub("\\s+", " ", x)

getResponse <- function(url) {
  response <- httr::content(httr::GET(url), encoding = "utf-8")
  if (httr::parse_url(url)$hostname == "foreignpolicy.com") {
    response <- xml2::as_xml_document(rawToChar(response))
  }
  
  items <- rvest::html_nodes(response, "item")
  
  df <- plyr::rbind.fill(lapply(items, function(x) {
    data.frame(
      datetime = strptime(rvest::html_text(rvest::html_node(x, "pubDate")), format = "%a, %d %b %Y %T"),
      title = rvest::html_text(rvest::html_nodes(x, "title")),
      description = rvest::html_text(rvest::html_nodes(x, "description")),
      guid = rvest::html_text(rvest::html_nodes(x, "guid")),
      links = rvest::html_text(rvest::html_nodes(x, "link"))
    )
  }))
  
  df$datetime[is.na(df$datetime)] <- Sys.time()
  
  df <- as.data.frame(lapply(df, function(x) {
    result <- removeNextLine(removeHTMLtag(x))
    result <- stringr::str_trim(result)
  }))
  
  return(df)
}
```

#### Example

``` r
getUrls <- function() {
  list(
    "bbc" = "https://feeds.bbci.co.uk/news/rss.xml",
    "cnn" = "http://rss.cnn.com/rss/cnn_topstories.rss",
    "fp" = "https://foreignpolicy.com/feed/",
    "nyTimes" = "https://rss.nytimes.com/services/xml/rss/nyt/World.xml"
  )
}

urls <- getUrls()
feeds <- lapply(urls, getResponse)
names(feeds) <- names(urls)
```

Below shows the RSS feeds from BBC.

| datetime            | title                                                                           | description                                                                                              | guid                                                          | links                                                                                           |
|:--------------------|:--------------------------------------------------------------------------------|:---------------------------------------------------------------------------------------------------------|:--------------------------------------------------------------|:------------------------------------------------------------------------------------------------|
| 2022-03-31 14:27:38 | Putin sets midnight deadline over rouble gas payments                           | Vladimir Putin signs a decree ordering foreign buyers of Russian gas to open Russian bank accounts.      | <https://www.bbc.co.uk/news/business-60945248>                | <https://www.bbc.co.uk/news/business-60945248?at_medium=RSS&at_campaign=KARANGA>                |
| 2022-03-31 14:19:11 | Ukraine war: Ukraine sends buses to Mariupol for rescue effort                  | A convoy of 45 buses is heading for the besieged city in the latest attempt to rescue trapped civilians. | <https://www.bbc.co.uk/news/world-europe-60938429>            | <https://www.bbc.co.uk/news/world-europe-60938429?at_medium=RSS&at_campaign=KARANGA>            |
| 2022-03-31 08:46:48 | Chris Rock says he is still processing Smith slap                               | The comedian says he is “still processing what happened” and denies reports he has spoken to Smith.      | <https://www.bbc.co.uk/news/entertainment-arts-60939316>      | <https://www.bbc.co.uk/news/entertainment-arts-60939316?at_medium=RSS&at_campaign=KARANGA>      |
| 2022-03-31 11:34:04 | UK weather: Spring snow as parts of country hit by cold snap                    | “Spring is on hold,” says BBC Weather, in a sharp change to last week which saw highs of 20C.            | <https://www.bbc.co.uk/news/uk-60941529>                      | <https://www.bbc.co.uk/news/uk-60941529?at_medium=RSS&at_campaign=KARANGA>                      |
| 2022-03-31 14:12:14 | Energy websites crash in meter readings rush                                    | Suppliers including Shell and EDF are working to resolve issues, ahead of a price rise on Friday.        | <https://www.bbc.co.uk/news/business-60938730>                | <https://www.bbc.co.uk/news/business-60938730?at_medium=RSS&at_campaign=KARANGA>                |
| 2022-03-31 12:02:25 | Lincoln City to silence Dambusters March and air raid sirens due to Ukraine war | Lincoln City has asked supporters to support the move, sparked by Russia’s invasion of Ukraine.          | <https://www.bbc.co.uk/news/uk-england-lincolnshire-60938930> | <https://www.bbc.co.uk/news/uk-england-lincolnshire-60938930?at_medium=RSS&at_campaign=KARANGA> |
