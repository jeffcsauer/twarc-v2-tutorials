# Introduction to full archive searching using twarc v2

  

In late 2020 Twitter began rolling out version 2 (v2) of the Twitter API. Included in this rollout is a new way to access Twitter data for academic research projects. This new method, formally referred to as the 'Academic Research product track' or 'Academic API' for short, provides free access to the full-archive Twitter repository and other v2 endpoints. The Academic API is exciting as previous access to the full-archive data came at a financial cost. With this expanded access, Twitter hopes to enable more social media research from around the world.

  

This announcement was met with excitement from members of the open source software community who began immediately laying plans for community-based tools to handle the Academic API. Among the many open source tools to handle Twitter data is twarc, a well-established command line tool and Python library used for accessing and archiving Twitter data. Twarc has been used extensively by archivists, activists, researchers and more to gather data from the v1 API. However, the v2 API provides a [fundamentally different data payload](https://dev.to/twitterdev/understanding-the-new-tweet-payload-in-the-twitter-api-v2-1fg5). With dedicated work and contributions from scholars from around the globe, twarc was able to quickly respond to the new API and start allowing users access to an updated tool called [twarc v2](https://github.com/DocNow/twarc/tree/v2).

  

The following tutorial demonstrates the basic installation and use of twarc v2 for accessing Twitter's full archive search. It is meant as an introductory guide for researchers who may be new to gathering Twitter data. As always, please refer to the official [twarc](https://twarc-project.readthedocs.io/en/latest/) and [Twitter](https://developer.twitter.com/en/docs/twitter-api/early-access) documentation for more advanced details.

  

# Installing twarc v2

  

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

--access-token TEXT Twitter app access token for user

authentication.

  

--access-token-secret TEXT Twitter app access token secret for user

authentication.

...

```

  
  
  

# Configuring twarc v2 with your API keys

  

Before we can gather Twitter data we need to provide some information that proves we have been approved to use the Academic API. Users who have been approved for the Academic API will be provided with a set of [*keys*](https://cloud.ibm.com/docs/account?topic=account-manapikey). These keys authorize you to access the various Twitter API endpoints and gather data. As with any other key - digital or material - your keys should be kept in a secure location and should not be shared with others.

  

The final setup step of twarc v2 is registering your API keys. Twarc v2 conveniently offers a set-up tool such that API keys only need to be registered once (per local install). Run the following line to launch the set-up tool:

  

```console

twarc configure

```

  

You will then be guided through a series of prompts asking for your consumer key, secret consumer key, access token key, and secret access token key. Input your respective keys into each prompt and hit `[Enter]` to proceed.

  

You should now be able to execute searches using twarc and twarc v2!

  

# Understanding the recent Twitter archive versus the full Twitter archive

  

Twitter provides two different archives of past tweets. The first, generally referred to as the *recent* archive, is for tweets generated in the past 6 to 9 days. The second, generally referred to as the *full* archive, is for all tweets part of the public conversation. To construct a twarc v2 query to access these archives you can use the `search` command.

  

To access the recent archive, simply type:

  

```console

> twarc2 search '#blacklivesmatter'

```

  

You will see your terminal flooded with text. To stop a data pull, hit `Ctrl+C`.

  

To access the full archive, add a the `--archive` option like:

  

```console

> twarc2 search '#blacklivesmatter' --archive

```

  

Note that in almost all situations adding the `--archive` option will tap a much larger pool of tweets. These queries may take longer, result in larger files, and use up a greater share of the monthly cap. In the following sections we build on these basic searches to refine our API queries.

  

# Building a more speciifc twarc v2 full archive query

  

In general, you may want to add more rules on your full archive search to gather more specific tweets. For example, let's say you want to gather tweets related to Black Lives Matter from 2015 to 2016. Well, you can add flags for `--start-time` and `--end-time`. Note that these times should be in [%Y-%m-%d|%Y-%m-%dT%H:%M:%S] format. Let's see a full archive search looks like with start and end rules:

  

```console

> twarc2 search '#blacklivesmatter' \

--start-time 2015-01-01 \

--end-time 2016-12-31 \

--archive

```

  

If you run the above command you will be pulling form a very large pool of tweets - #blacklivesmatter is a popular topic and we are considering tweets across an entire year. Some additional ways to reduce the number of tweets could be to exclude retweets and grab a specific number of tweets, say, 5000. Well, twarc has options for these rules as well! Let's see what these additional rules look like:

  

```console

> twarc2 search '#blacklivesmatter -is:retweet' \

--start-time 2015-01-01 \

--end-time 2016-12-31 \

--limit 5000

--archive

```

  

Note that the rule to exclude retweets (`-is:retweet`) actually appears *within* the quoted search query, whereas the limit on the number of tweets appears as an option flag. At this point we have set several additional rules to create a more specific query of the Twitter full archive.

  

An additional choice that many users may find helpful is to *flatten* the data provided by twarc. Because not all Twitter users provide the same profile information - some individuals do not provide geographic information, profile information, etc - the data available between records may be inconsistent. Invoking the `--flatten` option will help to make the output data balanced such that each record contains the same fields, even if some of those fields ar empty.

  

```console

> twarc2 search '#blacklivesmatter -is:retweet' \

--start-time 2015-01-01 \

--end-time 2016-12-31 \

--limit 5000

--flatten

--archive

```

  

Lastly, we can save the results of a twarc search by providing an output file path at the end of our command. For example:

  

```console

> twarc2 search '#blacklivesmatter -is:retweet' \

--start-time 2015-01-01 \

--end-time 2016-12-31 \

--limit 5000

--flatten

--archive

C:/Where/You/Want/ToSaveResults.json

```

  

Note that twarc will automatically overwrite the existing file if a new output file path is not specified.

  

# Processing data into common analysis formats

  

The resulting file is a data type called JSON, which has many advantages for moving large amounts of text information. However, we need to take some steps to transform the JSON into a form more common for data analysis. We can use popular python libraries like `json` and `pandas` to carry out these transformations.

  

Let's extract each record from the JSON file produced by twarc v2 into a python list.

  

```python

# Load the json library

import json

# Crete a variable that points to the twarc JSON file

TwarcFile = "C:/Where/You/Want/ToSaveResults.json"

# Create an empty list

TwarcJsonData = []

# Loop through each line of the twarc JSON file, loading the

# appending the line to the empty list

for line in open(TwarcFile, 'r'):

     TwarcJsonData.append(json.loads(line))

```

  

Now can use the `pandas` library to convert the JSON data into a DataFrame format. This is accomplished using the `json_normalize()` function.

  

```python

import pandas

# Convert JSON data to a pandas DataFrame

TwarcDF = pandas.json_normalize(TwarcJsonData)

```

  

The resulting `pandas` dataframe contains a number of columns, some of which may be more or less relevant to your interests.

  

```python

# twarc returns all possible fields from the api by default. Let's reduce the dataset to only the columns of interest.

ColumnsOfInterest = ['source', 'text', 'possibly_sensitive', 'author_id', 'id', 'lang', 'conversation_id', 'created_at']

# Subset the TwarcDF dataframe

TwarcDF = TwarcDF[ColumnsOfInterest]

```

  

The final object, `TwarcDF`, can be saved as a text-delimited file or another data format suitable to your needs.

  

# Concluding remarks

  

By completing this tutorial you should now be able to:

  

- Install twarc and twarc v2, a well-established open source software used to gather data from the Twitter API

- Perform searches of the recent and full Twitter archive

- Process data from the API into data

  

As with any other dataset, users should be aware of the many caveats to analyzing data gathered from Twitter (see for example work by [Crawford, 2013](https://ellaholland2022.com/s/05-The-Hidden-Biases-in-Big-Data-Crawford.pdf), and [Jiang, 2020](https://www.tandfonline.com/doi/full/10.1080/15230406.2018.1434834?casa_token=sdnCPS7byGgAAAAA%3AIAbd5tpmHEOTyLj8NETo2PXFio7arZObxReN7AbZjxNjAqI8nXdoVfI76H5dN7kdZ2WJfDqplMYUvQ)). Happy twarcing!

  

# Additional twarc resources

  

-  [Twarc github](https://github.com/DocNow/twarc)

-  [Twarc tutorials from University of Virginia](http://digitalcollecting.lib.virginia.edu/toolkit/docs/social-media/collect-tweets/)

-  [Example of using twarc in the cloud](https://medium.com/@justinlittman_38536/twarc-cloud-twitter-data-collection-in-the-cloud-d1cac85f57a5)

-  [Introduction to Cultural Analytics & Python](https://melaniewalsh.github.io/Intro-Cultural-Analytics/Data-Collection/Twitter-Data-Collection.html)

-  [Documenting the Now](https://melaniewalsh.github.io/Intro-Cultural-Analytics/Data-Collection/Twitter-Data-Collection.html)
