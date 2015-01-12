---
layout: post
title: Scraping by Example - Handling JSON data
---

A common scraping task is to get all of the results returned for every option
in a select menu on a given form.

For instance, the following scraping job came up on oDesk a few weeks back:

![Form Image 1](/assets/brassring/1.png)
![Form Image 1](/assets/brassring/2.png)
![Form Image 1](/assets/brassring/3.png)

<table>
  <thead>
    <tr>
      <th>Key</th>
      <th>Value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td>FORMTEXT12</td>
      <td>Milpitas</td>
    </tr>
    <tr>
      <td>FORMTEXT13</td> 
      <td>{% highlight html %}
<input type='hidden' value="leadteacher">
</input>
<a href="jobdetails.aspx?SID=%5eY4xaA%2fHuxO7HUAxWgQZaaF5irzzrwqM_slp_rhc_Fd7LOkibpxyWh7faRjNg%2f29PF5WbMpEH&jobId=427661&type=search&JobReqLang=1&recordstart=1&JobSiteId=5216&JobSiteInfo=427661_5216&GQId=480">
  Child Care Lead Teacher
</a>{% endhighlight %}</td>
    </tr>
    <tr>
      <td>FORMTEXT10</td>
      <td> Bright Horizons in Milpitas</td>
    </tr>
    <tr>
      <td>FORMTEXT11</td> 
      <td>95035</td>
    </tr>
    <tr>
      <td>FORMTEXT8</td>
      <td>California</td>
    </tr>
    <tr>
      <td>currentRowHiddenData</td>
      <td>{% highlight html %}
<input type='hidden' 
  name='hidJobSiteId' 
  id='hidJobSiteId_427661'  
  value='427661_5216' />
<input type='hidden' 
  name='hidJobGQId'  
  value='480' />
      {% endhighlight %}</td>
    </tr>
    <tr>
      <td>AutoReq</td>
      <td>17611BR</td>
    </tr>
    <tr>
      <td>ORMTEXT5</td> 
      <td>Full-Time</td>
    </tr>
    <tr>
      <td>FORMTEXT3</td>
      <td>Regular</td>
    </tr>
    <tr>
      <td>chkColumn</td>
      <td>{% highlight html %}
<label for='427661'></label>
<div style='text-align:center'>
  <input type='checkbox' title='leadteacher' 
      id='427661' name='chkJobClientIds' 
      value='427661|1|480' 
      onClick=javascript:onCheckJob('427661',427661,'1','480','1'); 
  />
</div>
     {% endhighlight %}</td>
    </tr>
  </tbody>
</table>

{% highlight python %}
import re, json
import mechanize
import urlparse
import urlutil
from bs4 import BeautifulSoup

class BrassringJobScraper(object):
    def __init__(self, url):
        self.br = mechanize.Browser()
        self.url = url
        self.soup = None
        self.jobs = []
        self.numJobsSeen = 0
{% endhighlight %}

{% highlight python %}
def scrape(self):
    self.open_search_openings_page()
    self.submit_search_form()
    self.scrape_jobs()
{% endhighlight %}

{% highlight python %}
def open_search_openings_page(self):
    r = re.compile(r'Search openings', re.I)
    self.br.open(self.url)
    self.br.follow_link(self.br.find_link(text_regex=r))

{% endhighlight %}

{% highlight python %}
def submit_search_form(self):
    self.soupify_form(soup=None, form_name='aspnetForm')
    self.br.select_form('aspnetForm')
    self.br.submit()
    self.soup = BeautifulSoup(self.br.response().read())
{% endhighlight %}

{% highlight python %}
def soupify_form(self, soup, form_name):
    #
    # Some of script contents throw off mechanize
    # and it gives error 'ParseError: OPTION outside of SELECT'
    # So we soupify it to remove script contents
    #
    if not soup:
        soup = BeautifulSoup(self.br.response().read())

    form = soup.find('form', attrs={'name': form_name})
    html = str(form)
    resp = mechanize.make_response(
        html, 
        [("Content-Type", "text/html")],
        self.br.geturl(),
        200, "OK"
    )
    self.br.set_response(resp)

{% endhighlight %}

{% highlight python %}
def scrape_jobs(self):
    while not self.seen_all_jobs():
        t = 'ctl00_MainContent_GridFormatter_json_tabledata'
        i = self.soup.find('input', id=t)
        j = json.loads(i['value'])

        for x in j:
            # debug
            print '\n'.join('%s\t%s' % (y,z) for y,z in x.items())
            print '\n'

            job = {}
            job['title'] = self.get_title_from_job_dict(x)
            job['location'] = self.get_location_from_job_dict(x)
            job['url'] = self.get_url_from_job_dict(x)

            self.jobs.append(job)

        self.numJobsSeen += len(j)

        # Next page
        self.goto_next_page()
{% endhighlight %}

{% highlight python %}
def seen_all_jobs(self):
    self.soupify_form(soup=self.soup, form_name='frmMassSelect')
    self.br.select_form('frmMassSelect')
    return self.numJobsSeen >= int(self.br.form['totalrecords'])
{% endhighlight %}

### Pagination

Records are returned 50 results at a time. 

{% highlight html %}
<form name="frmMassSelect" method="post" action="searchresults.aspx?SID=^Ma_slp_rhc_ooksXuPqc_slp_rhc__slp_rhc_4aAdjbWLwoLkE4hXrS/w7UiUsLxM6pDX4cUsEv3OAkRPQnDuWC" style="visibility:hidden" aria-hidden="true">
  <input type="hidden" name="JobInfo" value="">
  <input type="hidden" name="recordstart" value="1">
  <input type="hidden" name="totalrecords" value="917">
  <input type="hidden" name="sortOrder" value="ASC">
  <input type="hidden" name="sortField" value="FORMTEXT13">
  <input type="hidden" name="sorting" value="">
  <input type="hidden" name="JobSiteInfo" value="">
</form>
{% endhighlight %}

Form data sent is as follows:

{% highlight javascript %}
JobInfo:%%
recordstart:51
totalrecords:917
sortOrder:ASC
sortField:FORMTEXT13
sorting:
JobSiteInfo:
{% endhighlight %}

{% highlight python %}
def goto_next_page(self):
    self.br.select_form('frmMassSelect')
    self.br.form.set_all_readonly(False)
    self.br.form['recordstart'] = '%d' % (self.numJobsSeen + 1)
    self.br.submit()
    self.soup = BeautifulSoup(self.br.response().read())
{% endhighlight %}

{% highlight python %}
def get_title_from_job_dict(self, job_dict):
    pass

def get_location_from_job_dict(self, job_dict):
    pass

def get_soup_anchor_from_job_dict(self, job_dict):
    pass

def get_url_from_job_dict(self, job_dict):
    a = self.get_soup_anchor_from_job_dict(job_dict)
    u = urlparse.urljoin(self.br.geturl(), a['href'])
    u = self.refine_url(u)
    return u
{% endhighlight %}

{% highlight python %}
def refine_url(self, job_url):
    """
    """
    items = urlutil.url_query_get(
        self.url.lower(), 
        ['partnerid', 'siteid']
    )

    url = urlutil.url_query_filter(job_url, 'jobId')
    url = urlutil.url_query_add(url, items.iteritems())

    return url
{% endhighlight %}

{% highlight python %}
#!/usr/bin/env python

import sys, signal
from brassring import BrassringJobScraper
from bs4 import BeautifulSoup

# BrightHorizons
URL = 'https://sjobs.brassring.com/TGWebHost/home.aspx?partnerid=25595&siteid=5216'

def sigint(signal, frame):
    sys.stderr.write('Exiting...\n')
    sys.exit(0)    

class BrightHorizonsJobScraper(BrassringJobScraper):
    def __init__(self):
        super(BrightHorizonsJobScraper, self).__init__(url=URL)

    def get_title_from_job_dict(self, job_dict):
        t = job_dict['FORMTEXT13']
        t = BeautifulSoup(t)
        return t.text

    def get_location_from_job_dict(self, job_dict):
        l = job_dict['FORMTEXT12'] + ', ' + job_dict['FORMTEXT8']
        l = l.strip()
        return l

    def get_soup_anchor_from_job_dict(self, job_dict):
        t = job_dict['FORMTEXT13']
        t = BeautifulSoup(t)
        return t.a

if __name__ == '__main__':
    signal.signal(signal.SIGINT, sigint)

    scraper = BrightHorizonsJobScraper()
    scraper.scrape()

    for j in scraper.jobs:
        print j
{% endhighlight %}
