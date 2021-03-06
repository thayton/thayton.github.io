---
layout: post
title: Scraping with Python Selenium and PhantomJS
---

UPDATE 09/27/2018 - The site changed after this article was originally written. I've updated the code that waits for
the jobs to load, along with the description in this article.

In previous posts, I covered scraping using mechanize as the browser. Sometimes though 
a site uses so much Javascript to dynamically render its pages that using a tool like 
mechanize (which can't handle Javascript) isn't really feasable. For these cases, we have
to use a browser that can run the Javascript required to generate the pages. 

## Overview

[PhantomJS](http://phantomjs.org/) is a headless (non-gui) browser. [Selenium](http://www.seleniumhq.org/) 
 is a tool for automating browsers. In this post, we'll use the two together to scrape 
a Javascript heavy site. First we'll navigate to the site and then, after the HTML has 
been dynamically generated, we'll feed it into BeautifulSoup for parsing.

First let's set up our environment by installing PhantomJS along with the Selenium bindings 
for Python:

```bash
$ mkdir scraper && cd scraper
$ brew install phantomjs
$ virtualenv venv
$ source venv/bin/activate
$ pip install selenium
```

Now, let's look at the site we'll use for our example, the job search page for the company
[L-3 Klein Associates](http://www.l-3com.com/careers/us-job-search.html). They use the Taleo Applicant
Tracking System and the pages are almost entirely generated via Javascript:

[https://l3com.taleo.net/careersection/l3\_ext\_us/jobsearch.ftl](https://l3com.taleo.net/careersection/l3_ext_us/jobsearch.ftl)

In this post, we'll develop a script that can scrape, and then print out, all of the jobs listed on 
their Applicant Tracking System. 

Let's get started. 

## Implementation

First, let's sketch out our class, `TaleoJobScraper`. In the constructor we'll instantiate a webdriver
for PhantomJS. Our main method will be `scrape()`. It will call `scrape_job_links()` to iterate through
the job listings, and then call `driver.quit()` once it's complete.

```python
#!/usr/bin/env python

import re, urlparse

from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from bs4 import BeautifulSoup
from time import sleep

link = 'https://l3com.taleo.net/careersection/l3_ext_us/jobsearch.ftl'

class TaleoJobScraper(object):
    def __init__(self):
        self.driver = webdriver.PhantomJS()
        self.driver.set_window_size(1120, 550)
        self.wait = WebDriverWait(self.driver, 10)
        
    def scrape(self):
        jobs = self.scrape_job_links()
        for job in jobs:
            print job

        self.driver.quit()

if __name__ == '__main__':
    scraper = TaleoJobScraper()
    scraper.scrape()
```

Now let's take a look at the `scrape_job_links()` method, which is listed next:

```python
def scrape_job_links(self):
    self.driver.get(link)

    r = re.compile(r'\d+ - \d+ of \d+')
    f = lambda d: re.search(r, d.find_element_by_id('currentPageInfo').text)

    self.wait.until(f)

    m = f(self.driver)
    t = m.group(0)

    jobs = []
    pageno = 2

    while True:
        s = BeautifulSoup(self.driver.page_source)
        r = re.compile(r'jobdetail\.ftl\?job=\d+$')

        for a in s.findAll('a', href=r):
            tr = a.findParent('tr')
            td = tr.findAll('td')

            job = {}
            job['title'] = a.text
            job['url'] = urlparse.urljoin(link, a['href'])
            job['location'] = td[2].text
            jobs.append(job)

        next_page_elem = self.driver.find_element_by_id('next')
        next_page_link = s.find('a', text='%d' % pageno)

        if next_page_link:
            next_page_elem.click()

            # Watch for the transition from "1 - 25 of 1106" to "26 - 50 of 1106"
            self.wait.until(lambda d: f(d) and f(d).group(0) != t)

            m = f(self.driver)
            t = m.group(0)

            pageno += 1
            sleep(.75)
        else:
            break

    return jobs
```

First, we open the page with `driver.get()`. We wait for the jobs to load by monitoring the contents of the `span`
with id `currentPageInfo`. Once the jobs are loaded, the text within this span will get updated to show the total
number of jobs in the results. The text that appears will be something like `Job Openings 1 - 25 of 1106`.

Once the jobs are loaded, we feed the rendered HTML in `driver.page_source` into BeautifulSoup. Then we match against
the `href` attribute of the job links. For each job link we extract the title, url, and location.

To get all of the jobs, we also need to handle pagination. There's a pager at the bottom of the 
jobs listings. Below is a screenshot of the pager. A user can click a page number or the `Next` 
link to navigate through the listings.

![Form Image](/assets/taleo/pagination.png)

We use the `Next` link to iterate through every page of the results by first finding the `Next` element 
using the driver's [find\_element\_by\_id](http://selenium-python.readthedocs.org/en/latest/locating-elements.html) 
method and then calling `click()` if we're not on the last page.

After clicking the next page link, we wait until the next page of jobs loads by waiting for the text
contents of the `span#currentPageInfo` element to change. The text will get updated from something like
`Job Openings 1 - 25 of 11021` to `Job Openings 26 - 50 of 1102`:

```python
next_page_elem = self.driver.find_element_by_id('next')
next_page_link = s.find('a', text='%d' % pageno)

if next_page_link:
    next_page_elem.click()

    # Watch for the transition from "1 - 25 of 1106" to "26 - 50 of 1106"
    self.wait.until(lambda d: f(d) and f(d).group(0) != t)

    m = f(self.driver)
    t = m.group(0)

    pageno += 1
else:
    break
```

To determine if we're on the last page we search for a link whose text equals the current page 
number plus one. If no such link exists then we've reached the last page of results and break.

If you'd like to see a working version of the code developed in this post, it's available on 
github [here](https://github.com/thayton/taleo_job_scraper).

## Shameless Plug

Have a scraping project you'd like done? I'm available for hire. [Contact me](/contact) 
with some details about your project and I'll give you a quote.

