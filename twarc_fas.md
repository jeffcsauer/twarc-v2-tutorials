# Introduction to full archive searching using twarc v2

In late 2020 Twitter began rolling out version 2 (v2) of the Twitter API. Included in this rollout is a new way to access Twitter data for academic research projects. This new method, formally referred to as the 'Academic Research product track' or 'Academic API' for short, provides free access to the full-archive Twitter repository and other v2 endpoints. The Academic API is exciting as previous access to the full-archive data came at a financial cost. With this expanded access, Twitter hopes to enable more social media research from around the world.

This announcement was met with excitement from members of the open source software community who began immediately laying plans for community-based tools to handle the Academic API. Among the many open source tools to handle Twitter data is twarc, a well-established command line tool and Python library used for accessing and archiving Twitter data. Twarc has been used extensively by archivists, activists, researchers and more to gather data from the v1 API. However, the v2 API provides a [fundamentally different data payload](https://dev.to/twitterdev/understanding-the-new-tweet-payload-in-the-twitter-api-v2-1fg5). With dedicated work and contributions from scholars from around the globe, twarc was able to quickly respond to the new API and start allowing users access to an updated tool called [twarc v2](https://github.com/DocNow/twarc/tree/v2).

The following tutorial demonstrates the basic installation and use of twarc v2 for accessing Twitter's full archive search. It is meant as an introductory guide for researchers who may be new to gathering Twitter data. As always, please refer to the official [twarc](https://twarc-project.readthedocs.io/en/latest/) and [Twitter](https://developer.twitter.com/en/docs/twitter-api/early-access) documentation for more advanced details.

## Installing twarc v2

Before installing twarc, make sure that you have a recent and updated installation of Python 2 or 3. Python can be downloaded from the official [python.org](https://www.python.org/downloads/) website. After installing python you can proceed with the installation of twarc or twarc v2 using the popular package-management system called pip.

Open up a new terminal and install twarc v2 by typing:

```console

pip install twarc

```

To ensure that twarc v2 has been installed, try typing `twarc2` into the command line. You should get the following output:

```console

> twarc2

Usage: twarc2 [OPTIONS] COMMAND [ARGS]...
Collect data from the Twitter V2 API.

Options:

--consumer-key TEXT Twitter app consumer key (aka "App Key")
--consumer-secret TEXT Twitter app consumer secret (aka "App Secret")
--access-token TEXT Twitter app access token for user authentication.
--access-token-secret TEXT Twitter app access token secret for user authentication.
...
```

## Configuring twarc v2 with your API keys

Before we can gather Twitter data we need to provide some information that proves we have been approved to use the Academic API. Users who have been approved for the Academic API will be provided with a set of [*keys*](https://cloud.ibm.com/docs/account?topic=account-manapikey). These keys authorize you to access the various Twitter API endpoints and gather data. As with any other key - digital or material - your keys should be kept in a secure location and should not be shared with others.

The final setup step of twarc v2 is registering your API keys. Twarc v2 conveniently offers a set-up tool such that API keys only need to be registered once (per local install). Run the following line to launch the set-up tool:

```console

twarc2 configure

```

You will then be guided through a series of prompts asking for your consumer key, secret consumer key, access token key, and secret access token key. Input your respective keys into each prompt and hit `[Enter]` to proceed.

You should now be able to execute searches using twarc and twarc v2!

## Understanding the recent Twitter archive versus the full Twitter archive

Twitter provides two different archives of past tweets. The first, generally referred to as the *recent* archive, is for tweets generated in the past 6 to 9 days. The second, generally referred to as the *full* archive, is for all tweets part of the public conversation. To construct a twarc v2 query to access these archives you can use the `search` command.

To access the recent archive, simply type:

```console

> twarc2 search '#blacklivesmatter'

```

You will see your terminal flooded with text. To stop a data pull, hit `Ctrl+C`.

In order to write that data to a file instead of the terminal window supply the filename to write the data to:

```console
> twarc2 search '#blacklivesmatter' tweets.jsonl
```

The `.jsonl` file extension is often used to indicate that the file contains [line oriented json](https://jsonlines.org/). 

To access the full archive (back to 2006), add a the `--archive` option like:

```console

> twarc2 search '#blacklivesmatter' --archive

```

Note that in almost all situations adding the `--archive` option will tap a much larger pool of tweets. These queries may take longer, result in larger files, and use up a greater share of the monthly cap. In the following sections we build on these basic searches to refine our API queries.

## Building a more specific twarc v2 full archive query

In general, you may want to add more rules on your full archive search to gather more specific tweets. For example, let's say you want to gather tweets related to Black Lives Matter from 2015 to 2016. Well, you can add flags for `--start-time` and `--end-time`. Note that these times should be in [%Y-%m-%d|%Y-%m-%dT%H:%M:%S] format. Let's see a full archive search looks like with start and end rules:

```console
> twarc2 search '#blacklivesmatter' --start-time 2015-01-01 --end-time 2016-12-31 --archive  
```

If you run the above command you will be pulling form a very large pool of tweets - #blacklivesmatter is a popular topic and we are considering tweets across an entire year. Some additional ways to reduce the number of tweets could be to exclude retweets and grab a specific number of tweets, say, 5000. Well, twarc has options for these rules as well! Let's see what these additional rules look like:

```console
> twarc2 search '#blacklivesmatter -is:retweet' --start-time 2015-01-01 --end-time 2016-12-31 --limit 5000 --archive
```

Note that the rule to exclude retweets (`-is:retweet`) actually appears *within* the quoted search query, whereas the limit on the number of tweets appears as an option flag. At this point we have set several additional rules to create a more specific query of the Twitter full archive.

An additional choice that many users may find helpful is to *flatten* the data provided by twarc. This is useful in data pipelines that expect there to be one tweet per line in the output file. If you've collected some tweets and would like to flatten them you can:

```console
> twarc2 flatten tweets.jsonl flattened-tweets.jsonl
```

## Processing data into common analysis formats

The resulting file is a data type called JSON, which has many advantages for moving large amounts of structured data. However, we need to take some steps to transform the JSON into a form more common for data analysis. We can use the [twarc-csv](https://github.com/docnow/twarc-csv) module to convert the line oriented JSON to CSV which then should be more easy to use as DataFrames in tools like [Pandas](https://pandas.pydata.org/) and [R](https://www.r-project.org/). Twarc plugins are distributed separately from twarc, and they extend the base twarc2 command with additional subcommands, in the case of twarc-csv a `csv` subcommand will be added.

```console

> pip install twarc-csv
> twarc2 csv tweets.jsonl tweets.csv
```

Then you can load the CSV into a Pandas DataFrame:

```python
import pandas
pandas.read_csv('tweets.csv')
```

Of course you can process the tweets directly as JSON. To make your life easier you may want to flatten it first to ensure that each line contains a single tweet.

```console
twarc2 flatten tweets.jsonl flattened-tweets.jsonl
```

Then you can create a small Python program you can read in each line, parses it as JSON, and then does something with the data (in this case it prints out the tweet ID and text):

```python
import json

for line in open('tweets.jsonl'):
    tweet = json.loads(line)
    print(tweet['data']['id'], tweets['data']['text'])
```

Python is used here just as an example, and modern programming language you might find will have a JSON parsing library.

## Concluding remarks

By completing this tutorial you should now be able to:

- Install twarc and twarc v2, a well-established open source software used to gather data from the Twitter API
- Perform searches of the recent and full Twitter archive
- Process data from the API into data

As with any other dataset, users should be aware of the many caveats to analyzing data gathered from Twitter (see for example work by [Crawford, 2013](https://ellaholland2022.com/s/05-The-Hidden-Biases-in-Big-Data-Crawford.pdf), and [Jiang, 2020](https://www.tandfonline.com/doi/full/10.1080/15230406.2018.1434834)). Happy twarcing!

# Additional twarc resources

-  [Twarc github](https://github.com/DocNow/twarc)
-  [Twarc tutorials from University of Virginia](http://digitalcollecting.lib.virginia.edu/toolkit/docs/social-media/collect-tweets/)
-  [Example of using twarc in the cloud](https://medium.com/@justinlittman_38536/twarc-cloud-twitter-data-collection-in-the-cloud-d1cac85f57a5)
-  [Introduction to Cultural Analytics & Python](https://melaniewalsh.github.io/Intro-Cultural-Analytics/Data-Collection/Twitter-Data-Collection.html)
-  [Documenting the Now](https://melaniewalsh.github.io/Intro-Cultural-Analytics/Data-Collection/Twitter-Data-Collection.html)
