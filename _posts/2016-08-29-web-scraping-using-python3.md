---
title: Web scraping using python 3.5
date: 2016-08-02 07:14:14 +0545
description: python, webscraping 
categories: python
---

So, you want to harvest other people website(sometimes yours) using python3 when you don't care about API's(Sometimes I also don't). 

Web scraping has been around for a long time, and it is a techniques about extracting the useful data from website (Read terms and condition of a website before scraping). WebScraping is illegal in some countries, check out the wiki for some [facts](https://en.wikipedia.org/wiki/Web_scraping#Legal_issues).

So let us delve into the world of scraping using BS4 (*Beautiful Soup*) and **requests**.

To scrape a website we will use  [Requests](http://docs.python-requests.org/en/latest/) a simple yet powerful python HTTP library, and a [Beautiful Soup](http://http://www.crummy.com/software/BeautifulSoup/) 
to extract the HTML content.

The website we are going to scrape is [Nagariknews.com](http://nagariknews.com). Our goal is to extract latest news title and its corresponding URL. After scraping the **News headline** and **URL** we will tweet this latest headline to [twitter](https://twitter.com) using **Twitter** API(*probably in Part-2*).

So, Before scraping a website, we need to think and analyze the page structure of our target website, we will trying to find a URL on a website, which contains a list of latest news, which we will use to send a request to that link on a website and grab that page served by that **URL** to scrape the content.

OK, on analyzing the nagarkinews.com, I found that the URL `http://nagariknews.com/main-story/`, provides the latest headlines.

so let's grab this page using our HTTP python library `requests`

If you haven't install requests, requests is the third party python library, it will not be available on your system by default.you need to install it.

To install requests, as I am using **Linux Ubuntu 14.04 **, we can simply shoot out the series of commands in our terminal as shown below.
{% highlight bash %}
sudo apt-get install python-setuptools
sudo pip3 install requests
{% endhighlight %}

The first line of command will install the necessary python packages required for your system and second will install the requests library.

As, we are using python3, If you aren't sure, which version of python you are running, you can check it out by typing `python` in your **bash** shell(*some people use ZSH, csh, csh*), or in your *CMD(if you are a windows user)*, this will return the version of python you are using.

OK, before grabbing a page, let us install the *BS4*(Beautiful soup 4) a HTML parser(*also able to parse XML*), to install this parser, just issue the 
commands shown below.
{% highlight bash %}
sudo pip3 install BeautifulSoup4
{% endhighlight %}

OK, You have now all the python libraries on your system to scrape a website.

First we will grab a target page using requests,  a HTTP library and then, we will parse this raw HTML data using bs4. Save this code as `open_url.py` in some Dir name `utilities`. If you haven't create `utilities` Dir just go and create a `utilities` directory by issuing `mkdir utilities && cd utilities`.

OK, let us make a module  `url_open` by writing some python code in `url_open.py` file.

{% highlight python %}
#!/usr/bin/env python3
import requests
from bs4 import BeautifulSoup

def url_open(url):
    '''
    Extract raw_html data using requests and 
    return the nicely formatted BeautifulSoup object
    for the given url
    '''
    try:
        resp = requests.get(url)
        data = BeautifulSoup(resp)
    except Exception as e:
        print(e)
    else:
        return data

if __name__ == '__main__':
    url = 'http://nagariknews.com/main-story/'
    url_open(url)
{% endhighlight %}

In the above code, we grab a raw data in the form of `response object` using requests and format this data using `BeautifulSoup`, for further processing, we return this data in a nicely formatted BS4 object.

So, now we got a raw HTML data(Bs4 object), our next step is to find out the content in which we are interested. To parse the content, let us visit [Nagarik news portal](http://nagariknews.com/main-news)  and inspect the content.
So our target is to grab news title and URL from 'http://nagariknews.com/main-story' link.

So on inspecting the `http://nagariknews.com/main-story` main-news page, we found that news-title are inside the `h1` tag.

<img src="/images/nagariknews.png" alt="Nagariknews.com web inspection" width="130%">

we will extract all the h1 tag of this page, and for the next part, we will grab the URL and text content from the children tag `<a>` of these `h1` tag.
using our parse 'BeautifulSoup'.

OK, let us write some python code to extract all the  news and title from this raw-HTML data. First we will import our module `url-open` which we have just 
written and then we will pass the URL. Our `url-open` function will return the formatted BS4 object. We will inspect the `news headline` and `news-link` in that 
BS4 object and save this in the form of dict. We will accomplish this task by creating a function named `nagarik-crawler`. 

OK, let us create a file called nagarik_crawler.py file and write some python code
{% highlight python %}
#!/usr/bin/env python3

import url_open

def nagarik_crawler(url):
    '''pull nagariknews.com latest news
       search for the h1 tag which contain news title and url
       fetch title and url form <a>, a children tag of each <h1> tag
    '''
    title_and_url = dict()
    data_object = url_open(url)
    data_item = data_object.findAll('h1')

    for news in data_item:
        news = news.findAll('a')
        for newsobj in news:
            news_title = newsobj.get_text()
            news_url = url.split('/main-story/')[0]+ newsobj.get('href')
            title_and_url[news_title] = news_url
    return title_and_url

if __name__ == '__main__':
    url = 'http://nagariknews.com/main-story/'
    news = nagarik_crawler('http://nagariknews.com/main-story/')
    #Print your List of DICT objeect
    print(news)

{% endhighlight %}


The above code will return the news headline and its corresponding URL in the form of dict. You can use this returned data and do what ever you want. Here, is the final output of your code, that you have written.

<img src="/images/nagarik_output.png" alt="Nagarik news crawler output" width="130%">

In the second part  we will use the twitter API to tweet these links to twitter.
