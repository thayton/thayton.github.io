---
layout: post
title: Scraping AJAX Pages with Python
---

Scraping AJAX Pages with Python.

For this scraper, we'll use [Requests](http://docs.python-requests.org/en/latest/) and 
[BeautifulSoup](http://www.crummy.com/software/BeautifulSoup/). The instructions assume you are using the Chrome
browser on OSX. For those using other browsers/OS combinations, the concepts remains the same.

For this example, we'll use the jobs page for Apple.com 

Open the page <a href="https://jobs.apple.com/us/search" target="_blank">https://jobs.apple.com/us/search</a>.
Scroll down a bit and you'll see a jobs listing like the following.

![Jobs Page](/assets/scraping-ajax-pages-with-python/jobs_page.png)

Open the Chrome developer tools by selecting View > Developer > Developer Tools

![Developer Tools](/assets/scraping-ajax-pages-with-python/developer_tools.png)

Your browser screen should split with the developer tools window appearing in the bottom half. Select the 
Network tab.

![Split Screen](/assets/scraping-ajax-pages-with-python/split_screen.png)

Refresh the page. You should see Network tab fill up with the network requests being made for the Apple jobs page. 
Scroll down until you see the POST request to search-result.

![Search Result Line](/assets/scraping-ajax-pages-with-python/search_result_line.png)

Click on that line to see the details of that request. 

![Headers](/assets/scraping-ajax-pages-with-python/headers.png)

Under the headers tab, scroll down until you see the Form Data. 

[![Request](/assets/scraping-ajax-pages-with-python/request.png)](/assets/scraping-ajax-pages-with-python/request.png)

There are two parameters, `searchRequestJson` and `clientOffset`. The `searchRequestionJson` is a JSON string whose 
`pageNumber` field controls which page of results is returned for a request. Here's a formatted listing of the 
`searchRequestJson` fields.

{% highlight json %}
{  
   "searchString":"",
   "jobType":0,
   "sortBy":"req_open_dt",
   "sortOrder":"1",
   "language":null,
   "autocomplete":null,
   "delta":0,
   "numberOfResults":0,
   "pageNumber":0,
   "internalExternalIndicator":0,
   "lastRunDate":0,
   "countryLang":null,
   "filters":{  
      "locations":{  
         "location":[  
            {  
               "type":0,
               "code":"USA",
               "countryCode":null,
               "stateCode":null,
               "cityCode":null,
               "cityName":null
            }
         ]
      },
      "languageSkills":null,
      "jobFunctions":null,
      "retailJobSpecs":null,
      "businessLine":null,
      "hiringManagerId":null
   },
   "requisitionIds":null
}
{% endhighlight %}

Here's the equivalent Python dictionary.

{% highlight python %}
{
    "searchString":"",
    "jobType":0,
    "sortBy":"req_open_dt",
    "sortOrder":"1",
    "language":None,
    "autocomplete":None,
    "delta":0,
    "numberOfResults":0,
    "pageNumber":0,
    "internalExternalIndicator":0,
    "lastRunDate":0,
    "countryLang":None,
    "filters":{
        "locations":{
            "location":[{
                    "type":0,
                    "code":"USA",
                    "countryCode":None,
                    "stateCode":None,
                    "cityCode":None,
                    "cityName":None
            }]
        },
        "languageSkills":None,
        "jobFunctions":None,
        "retailJobSpecs":None,
        "businessLine":None,
        "hiringManagerId":None
    },
    "requisitionIds":None
}
{% endhighlight %}

Now click on the Response tab to see how jobs are returned for a query.

![Response Tab](/assets/scraping-ajax-pages-with-python/response_tab.png)

Jobs are returned in XML. If you select the entire response and format the results you'll
get a listing like the following.

{% highlight xml %}
<?xml version="1.0" encoding="UTF-8" standalone="yes"?>
<result>
  <count>2557</count>
  <requisition>
    <jobfunction>Retail</jobfunction>
    <jobId>USABL</jobId>
    <jobTypeCategory>Retail</jobTypeCategory>
    <location>Various</location>
    <retailPostingDate>Mar. 7, 2015</retailPostingDate>
    <retailPostingTitle>US-Business Leader</retailPostingTitle>
  </requisition>
  <requisition>
    <jobfunction>Retail</jobfunction>
    <jobId>USABM</jobId>
    <jobTypeCategory>Retail</jobTypeCategory>
    <location>Various</location>
    <retailPostingDate>Mar. 7, 2015</retailPostingDate>
    <retailPostingTitle>US-Business Manager</retailPostingTitle>
  </requisition>
  ...
</result>
{% endhighlight %}

For our scraper, we'll extract the job title, ID, and location for each job in the listing.

Before we start with the code, let's summarize what we need our scraper to do:

1. Construct the `searchRequestJson` dictionary. 
2. Initialize `searchRequestJson['pageNumber']` to 0. 
2. Convert the `searchRequestJson` dict to a JSON string
3. Send a POST request to `https://jobs.apple.com/us/search/search-result` with `searchRequestJson` and `clientOffset` parameters.
4. Parse the XML response with BeautifulSoup and extract the job title, id, and location for each job.
5. Increment `pageNumber` field of `searchRequestJson` dict.
6. Go to step 3 to get the next page of results.

Now let's write the code. 

First, we create a class named AppleJobScraper with a dict named `search_request` for building the `searchRequestJson` string.

{% highlight python %}
#!/usr/bin/env python

import json
import requests
from bs4 import BeautifulSoup

class AppleJobsScraper(object):
    def __init__(self):
        self.search_request = {
            "searchString":"",
            "jobType":0,
            "sortBy":"req_open_dt",
            "sortOrder":"1",
            "language":None,
            "autocomplete":None,
            "delta":0,
            "numberOfResults":0,
            "pageNumber":None,
            "internalExternalIndicator":0,
            "lastRunDate":0,
            "countryLang":None,
            "filters":{
                "locations":{
                    "location":[{
                            "type":0,
                            "code":"USA",
                            "countryCode":None,
                            "stateCode":None,
                            "cityCode":None,
                            "cityName":None
                    }]
                },
                "languageSkills":None,
                "jobFunctions":None,
                "retailJobSpecs":None,
                "businessLine":None,
                "hiringManagerId":None},
            "requisitionIds":None
        }
{% endhighlight %}

Next, we'll create a method named `scrape` which will call `scrape_jobs` to scrape the jobs
and print out the results.

{% highlight python %}
def scrape(self):
    jobs = self.scrape_jobs()
    for job in jobs:
        print job
{% endhighlight %}

The `scrape_jobs` method is where we implement the steps discussed earlier.

{% highlight python linenos %}
def scrape_jobs(self, max_pages=3):
    jobs = []
    pageno = 0
    self.search_request['pageNumber'] = pageno

    while pageno < max_pages:
        payload = { 
            'searchRequestJson': json.dumps(self.search_request),
            'clientOffset': '-300'
        }

        r = requests.post(
            url='https://jobs.apple.com/us/search/search-result',
            data=payload,
            headers={
                'X-Requested-With': 'XMLHttpRequest'
            }
        )

        s = BeautifulSoup(r.text)
        if not s.requisition:
            break

        for r in s.findAll('requisition'):
            job = {}
            job['jobid'] = r.jobid.text
            job['title'] = r.postingtitle and \
                r.postingtitle.text or r.retailpostingtitle.text
            job['location'] = r.location.text
            jobs.append(job)

        # Next page
        pageno += 1
        self.search_request['pageNumber'] = pageno

    return jobs
{% endhighlight %}

On line 4 we initialize `pageNumber` to 0 to get the first page of jobs.

Then on lines 7-10 we create a dict for the parameters we'll be sending in the POST request and 
convert `search_request` into a JSON string using `json.dumps` in the process. 

Next on lines 12-18 we send the POST request. I've included the headers argument to show how you can control what 
headers are sent in your request but it's not necessary in this case to make the request work. 

Once we've sent the POST request, we parse our response using BeautifulSoup and extract the desired fields. 

On lines 33-34 we increment the page and then repeat our previous steps until we've gotten `max_pages` (by default 3) worth of results.

Let's try it out by instantiating the scraper and calling `scrape()`.

{% highlight python %}
if __name__ == '__main__':
    scraper = AppleJobsScraper()
    scraper.scrape()
{% endhighlight %}

{% highlight bash %}
$ ./scraper.py | head
{'title': u'US-Business Leader', 'location': u'Various', 'jobid': u'USABL'}
{'title': u'US-Business Manager', 'location': u'Various', 'jobid': u'USABM'}
{'title': u'US-Business Specialist', 'location': u'Various', 'jobid': u'USABS'}
{'title': u'US-Apple Store Leader Program', 'location': u'Various', 'jobid': u'USALP'}
{'title': u'US-Creative', 'location': u'Various', 'jobid': u'USACR'}
{'title': u'US-Expert', 'location': u'Various', 'jobid': u'USAEX'}
{'title': u'US-Inventory Specialist', 'location': u'Various', 'jobid': u'USAIS'}
{'title': u'US-Manager', 'location': u'Various', 'jobid': u'USAMN'}
{'title': u'US-Market Leader', 'location': u'Various', 'jobid': u'USAML'}
{'title': u'US-Genius', 'location': u'Various', 'jobid': u'USAGN'}
{% endhighlight %}
