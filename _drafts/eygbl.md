---
layout: post
title: Scraping by Example - SelectMinds ATS
---

In this post I'll show how to develop a scraper for the SelectMinds Applicant Tracking System (ATS). For my example,
I'll use the jobs site at [https://eygbl.referrals.selectminds.com/](https://eygbl.referrals.selectminds.com/){:target="_blank"}

![search_page](/assets/eygbl/search_page.png)

## Background

Before developing the code for the scraper, let's inspect the site and see how the jobs are loaded into the page. Click on the
link above to open the jobs site. Then open the Chrome developer tools and switch to the Network tab. Click the Search button to
load all jobs.

In the Network tab you'll see a series of XHR requests being made. I've highlighted the two requests (first and last) that we'll
need to recreate in order to scrape the jobs from the site.

![0](/assets/eygbl/0.png)

The first request is a POST to

`https://eygbl.referrals.selectminds.com/ajax/jobs/search/create`

with the following parameters and form data:

![1](/assets/eygbl/1.png)

We also need to pay attention to the request headers:

![4](/assets/eygbl/4.png)

The `tss-token` header is required. Without it we'll get back a 403 Invalid Access response from the
server. When a valid request is made the response sent back will be a JSON string with the following fields:

```json
{
   "Status": "OK",
   "UserMessage": "",
   "Result": {
     "JobSearch.id": 84067040
   }
}
```

We'll need to save the `JobSearch.id` value from the response because it's used as a parameter in the second XHR request, a POST to

`https://eygbl.referrals.selectminds.com/ajax/content/job_results`

with the following query string parameters

![3](/assets/eygbl/3.png)

We'll see where the rest of the parameters come from later in the article when we dig into how this request
gets generated. The response for this request contains the HTML for the jobs that we'll scrape in the `Result`
field of the JSON string returned:

```json
{
  "Status":"OK", "UserMessage": "",
  "Result": "<div class=\"results_content..."
}
```

## Implementation

Now that we've gotten an overview of the requests, let's dig into the site's code to see how the two XHR requests
are being generated so that we can duplicate them in our scraper. First, let's create a base class with some boilerplate
code for our SelectMinds scraper. We'll add methods to it as we go:

```python
import re
import sys
import json
import time
import logging
import requests

from bs4 import BeautifulSoup
from datetime import datetime
from urllib.parse import urljoin

class SelectMindsJobScraper(object):
    def __init__(self):
        FORMAT = "%(asctime)s [ %(filename)s:%(lineno)s - %(funcName)s() ] %(message)s"
        logging.basicConfig(format=FORMAT, datefmt='%Y-%m-%d %H:%M:%S')

        self.logger = logging.getLogger(__name__)
        self.logger.setLevel(logging.DEBUG)

        self.session = requests.Session()
```

Now we'll go back and take a look at that first XHR request again. In the Chrome developer tools, search for the URL used in the
first POST request:

![1st_request_search](/assets/eygbl/1st_request_search.png)

Click on the search result to open up the code in the Sources tab. Inside the Sources tab you'll see that the first XHR
request is implemented by code in [job_search_banner.js](https://eygbl.referrals.selectminds.com/job_search_banner.js)
which gets triggered when the Search button is clicked:

<span id="first_request"></span>

```javascript
// Main submit binding
j$('#jSearchSubmit', search_banner).click(function() {
  ....
  j$.ajax({
    type: 'POST',
    url: TVAPP.guid('/ajax/jobs/search/create'),
    data: data,
    success: function(result){
      job_search_id = result.Result['JobSearch.id'];
      j$.log('job_search_id: ' + job_search_id);
      // Load results
      j$(document).trigger('loadSearchResults', {'job_search_id':job_search_id});
    },
    dataType: 'json',
    error: function(xhr, textStatus, error) {
      TVAPP.masterErrorHandler(xhr, textStatus, error, null);
    }
  });
});
```

The URL is set to the return value of the call to

`TVAPP.guid('/ajax/jobs/search/create')`

This method is where the `uid: 219`parameter is being generated. To find the source code for this method go
into the console of the Chrome developer tools and type in `TVAPP.guid`. The console will display the beginning
of the code for this method.

![guid_method](/assets/eygbl/5.png)

Click on that code and it will take you into the Sources tab. The Sources tab shows that the `guid` method is defined in
[desktop.js](https://eygbl.referrals.selectminds.com/desktop.js):

```javascript
TVAPP.guid = function(url) {
  var date = new Date
  var uid = date.getMilliseconds();

  var additionType = "?uid=";

  for (var i = 0; i < url.length; i++) {
    if(url.charAt(i) == '?') {
      additionType = "&uid="
    }
  }

  var newURL = url + additionType + uid;

  return newURL;
};
```

So the `uid` comes from the milliseconds portion of the current local time `date.getMilliseconds()`. Let's add the corresponding
method inside our scraper:

```python
def guid(self):
    dt = datetime.now()
    guid = dt.microsecond / 1000
    return guid
```

Now that we've seen how the `uid` parameter is being generated, let's see how to get the `tss-token` value required in the
request headers. Search the HTML for `tss` inside the job site's main page and you'll find an `<input>` element containing
the token value:

```html
<input type="hidden" name="tsstoken" id="tsstoken" value="BNT9g...">
```

We can look that up using the tag name and id in BeautifulSoup:

```python
def get_tss_token(self, soup):
    i = soup.find('input', id='tsstoken')
    tss_token = i['value']

    return tss_token
```

At this point, we've seen enough to implement the first request in our scraper:

```python
def guid(self):
    dt = datetime.now()
    guid = dt.microsecond / 1000
    return guid

def get_tss_token(self, soup):
    i = soup.find('input', id='tsstoken')
    tss_token = i['value']

    return tss_token

def get_job_search_id(self, tss_token):
    headers = {
        'X-Requested-With': 'XMLHttpRequest',
        'tss-token': tss_token,
        'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.120 Safari/537.36'
    }
        
    uid = self.guid()
    params = {
        'uid': uid
    }
        
    data = {
        'keywords': ''
    }
        
    url = urljoin(self.url, '/ajax/jobs/search/create')

    resp = self.session.post(url, headers=headers, params=params, data=data)
    data = resp.json()

    return data['Result']['JobSearch.id']

def scrape(self):
    jobs = []
        
    resp = self.session.get(self.url)
    soup = BeautifulSoup(resp.text, 'lxml')
        
    tss_token = self.get_tss_token(soup)
    job_search_id = self.get_job_search_id(tss_token)
    
    ...
```

Now that we have the `job_search_id` value from the first request, we're ready to move on and see how the second XHR request is generated.
[As shown earlier](#first_request) in the code bound to the Search button, the job_search_id value from the first response is passed as the
argument to `loadSearchResults`:

```javascript
// Main submit binding
j$('#jSearchSubmit', search_banner).click(function() {
    ...
      j$(document).trigger('loadSearchResults', {'job_search_id':job_search_id});
```

Let's examine the code for `loadSearchResults`, which is defined in [job_list.js](https://eygbl.referrals.selectminds.com/job_list.js):

```javascript
// function to find existing or get new search results
function loadSearchResults(data) {
  ...
  // load new results and display them
  if (cached_results.length && !data.force_refresh) {
    ...  
  } else {
    // Load new results
    var context = 'JobSearch.id=' + data.job_search_id;
    if (TVAPP.property.fb) context += '&fb=true';
    if (campaign_id) context += '&Campaign.id=' + campaign_id + '&campaign_page=2';
    if (data.page) context += '&page_index=' + data.page;

    // Sync load is bad news bears. instead, let's async and callback:
    j$.ajax({
      type: 'POST',
      url: TVAPP.guid('/ajax/content/job_results?' + context + '&site-name=' + TVAPP.property.site.short_name + '&include_site=true'),
      dataType: 'json',
      success: function(response) {
        var new_results = j$(response.Result);
        ...
        // display the new results
        displayLoadedResults(new_results, data);
      },
      error: function(xhr, textStatus,error) {
        TVAPP.masterErrorHandler(xhr, textStatus, error, null);
      }
    });
  }
};
```

Once again we have a call to `TVAPP.guid` generating the URL for the AJAX request:

```javascript
TVAPP.guid('/ajax/content/job_results?' + context + '&site-name=' + TVAPP.property.site.short_name + '&include_site=true'),
```

As you can see, we need the values for the `context` and `site-name` parameters. The `context` variable is defined just
above the call to `guid()`. It's initialized with `JobSearch.id` and updated with `page_index` if a page number is set
via `page` in `data`:

```javascript
    var context = 'JobSearch.id=' + data.job_search_id;
    if (TVAPP.property.fb) context += '&fb=true';
    if (campaign_id) context += '&Campaign.id=' + campaign_id + '&campaign_page=2';
    if (data.page) context += '&page_index=' + data.page;
```

We can extract `TVAPP.property.site.short_name` from the HTML on the main page. View the HTML on the job site's main search
page and search for `TVAPP` and you'll find where `TVAPP.property.site.short_name` is set:

```HTML
 <script type="text/javascript">
   var TVAPP = TVAPP || {}; // There can only be ONE.
   TVAPP.property = {
     ...
     site: {
         id: "2",
         short_name: "default909"
     },     

```

We'll use a regex to extract the `short_name` value: 

```python
def get_site_short_name(self, soup):
    x = { 'type': 'text/javascript' }
    r = re.compile(r'short_name:\s+"([^"]+)')
    m = None
        
    for script in soup.find_all('script', attrs=x):
        m = re.search(r, script.text)
        if m:
            break

    short_name = None
    if m:
        short_name = m.group(1)
            
    return short_name
```

Now we have enough information to replicate the second request with our own version of `loadSearchResults`:

```python
def load_search_results(self, tss_token, job_search_id, short_name, pageno=1):
    headers = {
        'tss-token': tss_token,
    }

    params = {
        'JobSearch.id': job_search_id,
        'page_index': pageno,
        'site-name': short_name, # From TVAPP.property.site.short_name : short_name: "default909"
        'include_site': 'true',
        'uid': self.guid()
    }

    url = urljoin(self.url, '/ajax/content/job_results')

    resp = self.session.post(url, headers=headers, params=params)
    data = json.loads(resp.text)
        
    return data['Result']
```

We have everything we need at this point to complete our `scrape()` method:

```python
def scrape(self):
    jobs = []
        
    resp = self.session.get(self.url)
    soup = BeautifulSoup(resp.text, 'lxml')
        
    tss_token = self.get_tss_token(soup)
    short_name = self.get_site_short_name(soup)

    job_search_id = self.get_job_search_id(tss_token)

    pageno = 1

    while True:
        self.logger.info(f'Getting page {pageno}')
            
        html = self.load_search_results(tss_token, job_search_id, short_name, pageno)
        soup = BeautifulSoup(html, 'html.parser')

        d = soup.find('div', id='job_results_list_hldr')
            
        x = {'class': 'job_link'}
        y = {'class': 'location'}
            
        for a in d.find_all('a', attrs=x):
            l = a.find_next('span', attrs=y)
            
            job = {}
            job['title'] = a.text
            job['url'] = urljoin(self.url, a['href'])
            job['location'] = l.text.strip()

            jobs.append(job)

        self.logger.info(f'{len(jobs)} jobs scraped')

        d = soup.find('div', id='jPaginateNumPages')
        num_pages = int(float(d.text))
            
        if pageno >= num_pages:
            break

        time.sleep(1) # Don't hit the server too quickly
        pageno += 1
```

Let's go over the parts of the code that are new. First we'll start with the code that parses the HTML
returned by `load_search_results`. Examine the HTML on jobs page after submitting a search:

![jobs_html](/assets/eygbl/jobs_html.png)

We see that jobs are contained in `div#job_results_list_hldr`. We can find the job links and location
under this div using classes `job_link` and `location` respectively after feeding the HTML into
BeautifulSoup:

```python
html = self.load_search_results(tss_token, job_search_id, short_name, pageno)
soup = BeautifulSoup(html, 'html.parser')

d = soup.find('div', id='job_results_list_hldr')
           
x = {'class': 'job_link'}
y = {'class': 'location'}
            
for a in d.find_all('a', attrs=x):
    l = a.find_next('span', attrs=y)
```

Since pagination is handled via a parameter we can goto the next page by simply incrementing the `pageno`
variable that gets passed to `load_search_results`. We do need a way to check when we've reached the last
page though.

Examine the HTML for the pagination links at the bottom of the search results and you'll see that one of
the `div` elements contains the total number of pages for the search results:

![pagination_html](/assets/eygbl/pagination_html.png)

We can compare this element's value with the number of pages we've scraped so far, and if the two values
are equal, it means we've reached the last page:

```python
d = soup.find('div', id='jPaginateNumPages')
num_pages = int(float(d.text))
            
if pageno >= num_pages:
    break

time.sleep(1) # Don't hit the server too quickly
pageno += 1
```

```python
import re
import sys
import json
import time
import logging
import requests

from bs4 import BeautifulSoup
from datetime import datetime
from urllib.parse import urljoin

class SelectMindsJobScraper(object):
    def __init__(self):
        FORMAT = "%(asctime)s [ %(filename)s:%(lineno)s - %(funcName)s() ] %(message)s"
        logging.basicConfig(format=FORMAT, datefmt='%Y-%m-%d %H:%M:%S')

        self.logger = logging.getLogger(__name__)
        self.logger.setLevel(logging.DEBUG)

        self.session = requests.Session()

    def guid(self):
        dt = datetime.now()
        guid = dt.microsecond / 1000
        return guid

    def get_site_short_name(self, soup):
        x = { 'type': 'text/javascript' }
        r = re.compile(r'short_name:\s+"([^"]+)')
        m = None
        
        for script in soup.find_all('script', attrs=x):
            m = re.search(r, script.text)
            if m:
                break

        short_name = None
        
        if m:
            short_name = m.group(1)
            
        return short_name
        
    def get_tss_token(self, soup):
        i = soup.find('input', id='tsstoken')
        tss_token = i['value']

        return tss_token
    
    def get_job_search_id(self, tss_token):
        headers = {
            'X-Requested-With': 'XMLHttpRequest',
            'tss-token': tss_token,
            'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_12_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/77.0.3865.120 Safari/537.36'
        }
        
        uid = self.guid()
        params = {
            'uid': uid
        }
        
        data = {
            'keywords': ''
        }
        
        url = urljoin(self.url, '/ajax/jobs/search/create')

        resp = self.session.post(url, headers=headers, params=params, data=data)
        data = resp.json()

        return data['Result']['JobSearch.id']

    def load_search_results(self, tss_token, job_search_id, short_name, pageno=1):
        headers = {
            'tss-token': tss_token,
        }

        params = {
            'JobSearch.id': job_search_id,
            'page_index': pageno,
            'site-name': short_name, # From TVAPP.property.site.short_name : short_name: "default909"
            'include_site': 'true',
            'uid': self.guid()
        }

        url = urljoin(self.url, '/ajax/content/job_results')

        resp = self.session.post(url, headers=headers, params=params)
        data = json.loads(resp.text)
        
        return data['Result']
    
    def scrape(self):
        jobs = []
        
        resp = self.session.get(self.url)
        soup = BeautifulSoup(resp.text, 'lxml')
        
        tss_token = self.get_tss_token(soup)
        short_name = self.get_site_short_name(soup)

        job_search_id = self.get_job_search_id(tss_token)

        pageno = 1

        while True:
            self.logger.info(f'Getting page {pageno}')
            
            html = self.load_search_results(tss_token, job_search_id, short_name, pageno)
            soup = BeautifulSoup(html, 'html.parser')

            d = soup.find('div', id='job_results_list_hldr')
            
            x = {'class': 'job_link'}
            y = {'class': 'location'}
            
            for a in d.findAll('a', attrs=x):
                l = a.findNext('span', attrs=y)
            
                job = {}

                job['title'] = a.text
                job['url'] = urljoin(self.url, a['href'])
                job['location'] = l.text.strip()

                jobs.append(job)

            self.logger.info(f'{len(jobs)} jobs scraped')

            d = soup.find('div', id='jPaginateNumPages')
            num_pages = int(float(d.text))
            
            if pageno >= num_pages:
                break

            time.sleep(1) # Don't hit the server too quickly
            pageno += 1

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("-d", "--debug", help="Display debugging messages", action="store_true")
    
    scraper = EygblJobScraper()
    args = parser.parse_args()

    if args.debug:
        scraper.logger.setLevel(logging.DEBUG)

    scraper.scrape()

if __name__ == '__main__':
    main()
```