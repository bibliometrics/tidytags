---
title: "tidytags: Simple Collection and Powerful Analysis of Twitter Data""
authors:
  - affiliation: 1
    name: K. Bret Staudt Willet
    orcid: 0000-0002-6984-416X
  - affiliation: 2
    name: Joshua M. Rosenberg
    orcid: 0000-0003-2170-0447
date: 03 June 2020
bibliography: paper.bib
tags:
  - R
  - Twitter
  - social media
  - data science
  - data mining
  - wrapper
affiliations:
  - index: 1
    name: Michigan State University
  - index: 2
    name: University of Tennessee, Knoxville
---

# Statement of Need

An essential component of understanding behavior across the social sciences is to study the artifacts and behaviors of group members over time. Social media platforms such as Twitter are a context for analysis and research of a variety of topics that have a temporal component. For instance, online communities often struggle with attrition and lack of commitment, so it would be beneficial to understand why some users continue to sustain participation while others gradually drop out `[@xing_gao2018:2018]`. Also, because `@veletsianos_et_al2019:2019` found that scholars' social media use is interwoven with changes in their personal lives and societal transitions, their social media practices must be studied over time.

Twitter data are best collected in the moment that it is being produced. Collecting historical data from Twitter can be difficult and expensive, while time travel is hard, if not impossible. Access to Twitter data is limited by the platform’s API. For instance, a researcher using the Twitter API in the summer of 2020 to search for tweets about the 2019 conference of the Association for Educational Communication & Technology, [AECT](https://aect.org/) using hashtags #AECT19 or #AECTinspired would not be able to readily access tweets from the time of the convention (which occurred in October 2019). Accessing historical content from Twitter is not impossible but has its own obstacles. There are companies that collect historical Twitter data and make these available to academic researchers for the right price, an approach that can become expensive. There are also technical solutions to collect past tweets, but these require advanced technical skills and risk the likely violation of [Twitter's Terms of Service](https://twitter.com/en/tos) agreements. Meanwhile, the obstacles to time travel are familiar enough that they need not be repeated here.

Yet, many approaches to collecting social media data in the moment require important technical skill that may dissuade social scientists from getting started. For Twitter data, a more accessible solution is to use a [Twitter Archiving Google Sheet](https://tags.hawksey.info/), (TAGS). Getting started with TAGS is as simple as setting up a new Google Sheet, which will then automatically query (i.e., search) the Twitter API every hour going forward. However, although TAGS provides several advantages for **data collection**, it has important limitations related to **data analysis**. For example, TAGS returns limited metadata compared to what is available from the Twitter API: approximately 20% of all categories of information. That is, a TAGS returns the time, sender, and text of tweets, but not many additional details such as a list of the hashtags or hyperlinks contained in a tweet. 

We introduce the {tidytags} package as an approach that allows for both simple data collection and thorough data analysis. In short, {tidytags} first uses TAGS to easily collect tweet ID numbers and then uses the R package {rtweet} to re-query the Twitter API to collect additional metadata.

# Summary

{tidytags} coordinates the simplicity of collecting tweets over time with a [Twitter Archiving Google Sheet](https://tags.hawksey.info/) (TAGS) and the utility of the [{rtweet} package](https://docs.ropensci.org/rtweet/index.html) for processing and preparing additional Twitter metadata. {tidytags} also introduces functions developed to facilitate systematic yet flexible analyses of data from Twitter.

# Preconditions for a {tidytags} Analysis

## First, Publish Data Collected with TAGS to the Web

{tidytags} facilitates working with a [Twitter Archiving Google Sheet](https://tags.hawksey.info/) (TAGS) in R through the [{googlesheets4} package](https://CRAN.R-project.org/package=googlesheets4). One requirement for using {googlesheets4} is that the TAGS tracker has been "published to the web." To do this, with the TAGS page open in a web browser, go to `File >> Publish to the web`. The `Link` field should show 'Entire document' and the `Embed` field should be 'Web page.' Once everything looks right, click the `Publish` button. Next, click the `Share` button in the top right corner of the Google Sheets window, select `Get shareable link`, and set the permissions to 'Anyone with the link can view.' The input needed for the `tidytags::read_tags()` function is either the entire URL from the top of the web browser when opened to a TAGS tracker, or a Google Sheet identifier (i.e., the alphanumeric string following "https://docs.google.com/spreadsheets/d/" in the TAGS tracker's URL). Be sure to put quotations marks around the URL or sheet identifier when entering it into `read_tags()` function.

## Second, Obtain a Twitter API Key

{tidytags} also allows the processing of tweets and preparation of additional Twitter metadata by building upon the [{rtweet} package](https://docs.ropensci.org/rtweet/index.html) (via `rtweet::lookup_statuses()`) to query the Twitter API. However, using {rtweet} requires Twitter API keys associated with an approved developer account. Fortunately, the {rtweet} vignette, [Obtaining and using access tokens](https://rtweet.info/articles/auth.html), provides a thorough guide to obtaining Twitter API keys. We recommend the second suggested method listed in the {rtweet} vignette, "2. Access token/secret method." Following these directions, you will run the `rtweet::create_token()` function, which saves your Twitter API keys to the `.Renviron` file. You can also edit this file directly using the `usethis::edit_r_environ(scope='user')` function.

# The {tidytags} Workflow

A workflow for Twitter research has been formalized in {tidytags}. This workflow is simple enough for beginning programmers to get started but powerful enough to serve as the analytic foundation of research that has been featured in academic journals such as *Computers & Education* `[@greenhalgh_et_al2018:2018]`, *Journal of Research on Technology in Education* `[@staudtwillet2019:2019]`, and *TechTrends* `[@greenhalgh_et_al2018:2018]`.

The {tidytags} workflow for exploring Twitter data over time using R includes:

1. Set up a Twitter Archiving Google Sheet [TAGS](https://tags.hawksey.info/) tweet collector.

2. View tweets collected by TAGS using the function `get_tags()` and either the TAGS tracker URL or the Google Sheet identifier (i.e., the alphanumeric string following "https://docs.google.com/spreadsheets/d/" in the TAGS tracker's URL).

```{r}
aect_tweets_tags <- read_tags("18clYlQeJOc6W5QRuSlJ6_v3snqKJImFhU42bRkM_OX8")
```

3. Pull additional tweet metadata using the function `pull_tweet_data()`.

```{r}
aect_tweets_full <- pull_tweet_data(aect_tweets_tags)
```

4. Calculate additional tweet attributes using the function `process_tweets()`.

```{r}
aect_tweets_processed <- process_tweets(aect_tweets_full)
```

5. Analyze hyperlinks and web domains in tweets using the function `get_url_domain()`.

```{r}
tweet_urls <- purrr::flatten_chr(aect_tweets_processed$urls_url)
tweet_urls <- tweet_urls[!is.na(tweet_urls)]  # Remove NA values
tweet_domains <- get_url_domain(tweet_urls)
```

6. Geocode tweeter locations and creating map visualizations using the function `geocode_tags()`.

```{r}
aect_places <- dplyr::distinct(aect_tweets_processed, location, .keep_all = TRUE)
aect_geo_coords <- geocode_tags(aect_places)
mapview::mapview(aect_geo_coords)
```

7. Analyze the social network of tweeters using the function `create_edgelist()`.

```{r}
aect_edgelist <- create_edgelist(aect_tweets_processed)
```

8. Append additional tweeter information to the edgelist using the function `add_users_data()`.

```{r}
aect_senders_receivers_data <- add_users_data(aect_edgelist)
```

From here, the data are shaped for straightforward use of the [{igraph} package](https://igraph.org/r/) or the [{tidygraph} package](https://www.data-imaginist.com/2017/introducing-tidygraph/) for network analysis and the [{ggraph} package](https://ggraph.data-imaginist.com/) for network visualization.

# Conclusion

{tidytags} is intended to lower barriers to powerful analyses of Twitter data. By combining an easy-to-use tool which can collect a large volume of longitudinal data from Twitter (TAGS), analysis from the {rtweet} R package, and functions that facilitate and extend their combined use, {tidytags} has the potential to assist in the collection of Tweets for a range of social-science-related analyses and research. 

# References
