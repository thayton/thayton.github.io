---
layout: post
title: Iterating through Select Items With Mechanize
---

Have a small scraping task you'd like to get done for free? Email me the details and if 
I think it will make good blog post I'll post a solution for free in my next 'Scraping by Example'
post.

### Getting a list of items

Sometimes we want to get all of the results for every item in a select-dropdown.
To do so, get a list of all the items in the select control before hand and then
submit them one a time to get the results for each item. 

For instance, the following scraping job came up on oDesk a few weeks back:

> Scrape the page at https://wish.wis.ntu.edu.sg/webexe/owa/aus_subj_cont.main
>
> Specifically, retrieve all of the results returned for every selectiion in the second dropdown menu 
> for the most recent academic year.

Here's a screenshot of the form they want results for:

![Form Image](/assets/ntu-edu/1.png)

The first select dropdown is for selecting an academic year. The second select dropdown is for
selecting an academic department (accounting, art, business, etc.). Note that the first dropdown
has the most recent academic year selected by default. 

If we take a look at the HTML for the form, we see the following:

{% highlight html %}
<form action="AUS_SUBJ_CONT.main_display1" method="POST" target="subjects">
  <select name="acadsem" onchange="this.form.action='AUS_SUBJ_CONT.main_display';this.form.target='_self';this.form.r_subj_code.value='';submit()">
    <option value="2006_S">Acad Yr 2006 Special Term I</option>
    <option value="2006_T">Acad Yr 2006 Special Term II</option>
    <option value="2006_1">Acad Yr 2006 1</option>
    ...
    <option selected="" value="2014_2">Acad Yr 2014 2</option>
  </select>
  <select name="r_course_yr" style="font-family: Arial">
    <option value="">---Select an Option---</option>
    <option value="ACC;GA;1;F">Accountancy (GA) Year 1</option>
    <option value="ACC;GA;2;F">Accountancy (GA) Year 2</option>
    <option value="ACC;GA;3;F">Accountancy (GA) Year 3</option>
    ...
  </select>  
</form>
{% endhighlight %}

The task then is to iterate through every option in the `r_course_yr` select dropdown, submit
the form for each option, and save the results we get back into a file.

Execute the following commands in order to install Mechanize and BeautifulSoup within a 
virtual environment: 

{% highlight bash %}
$ mkdir scraper && cd scraper
$ virtualenv venv
$ source venv/bin/activate
$ pip install mechanize
$ pip install beautifulsoup4
{% endhighlight %}

{% highlight python %}
#!/usr/bin/env python

import sys, signal, logging
import mechanize, time
from bs4 import BeautifulSoup, Comment

URL = 'https://wish.wis.ntu.edu.sg/webexe/owa/aus_subj_cont.main'
DELAY = 5

def sigint(signal, frame):
    sys.stderr.write('Exiting...\n')
    sys.exit(0)    

class NtuEduScraper:
    def __init__(self):
        self.br = mechanize.Browser()
        self.url = URL
        self.delay = 5
        self.items = []

if __name__ == '__main__':
    signal.signal(signal.SIGINT, sigint)
    scraper = NtuEduScraper()
    scraper.scrape()

{% endhighlight %}

* Open the page
* Select the form
* Get a list of all the items in the second select dropdown menu

Note that the form doesn't have a name attribute. 

{% highlight html %}
<form action="AUS_SUBJ_CONT.main_display1" method="POST" target="subjects">
{% endhighlight %}

As I noted in a [previous post]({% post_url 2014-12-08-form-handling-with-mechanize-and-beautifulsoup %})
we can use a predicate function to select the form.

Use the `get_items()` method to retrieve a list of all of the items in the select control
with the name `r_course_yr`.

{% highlight python %}
def select_form(form):
    '''
    Select the course display form
    '''
    return form.attrs.get('target', None) == 'subjects'

    def get_items(self):
        '''
        Get the list of items in the second dropdown of the form
        '''
        sys.stderr.write('Generating list of items for form selection\n')

        self.br.open(self.url)
        self.br.select_form(predicate=select_form)

        items = self.br.form.find_control('r_course_yr').get_items()

        sys.stderr.write('Got %d items for form selection\n' % len(items))

        return items
{% endhighlight %}

{% highlight python %}
    def scrape(self):
        self.items = self.get_items()

        for item in self.items:
            # Skip invalid/blank item selections
            if len(item.name) < 1:
                continue

            self.submit_form(item)
            time.sleep(3)
{% endhighlight %}

{% highlight python %}
    def submit_form(self, item):
        '''
        Submit form using selection item.name and write the results
        to file named according to item.label
        '''
        maxtries = 3
        numtries = 0

        sys.stderr.write('Submitting form for item %s\n' % item.name)

        while numtries < maxtries:
            try:
                self.br.open(self.url)
                self.br.select_form(predicate=select_form)
                self.br.form['r_course_yr'] = [ item.name ]
                self.br.form.find_control('boption').readonly = False
                self.br.form['boption'] = 'CLoad'
                self.br.submit()
                break
            except (mechanize.HTTPError, mechanize.URLError) as e:
                if isinstance(e,mechanize.HTTPError):
                    print e.code
                else:
                    print e.reason.args

            numtries += 1
            time.sleep(numtries * self.delay)

        if numtries == maxtries:
            raise

        self.item_results_to_file(item, self.br.response().read())
{% endhighlight %}

{% highlight python %}
    def item_results_to_file(self, item, results):
        label = ' '.join([label.text for label in item.get_labels()])
        label = '-'.join(label.split())

        sys.stderr.write('Writing results for item %s to file %s.html\n' % (item.name, label))

        with open("%s.html" % label, 'w') as f:
            f.write(results)
            f.close()
{% endhighlight %}