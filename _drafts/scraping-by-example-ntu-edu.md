---
layout: post
title: Scraping by Example - Iterating through Select Items With Mechanize
---

A common scraping task is to get all of the results returned for every option
in a select menu on a given form.

For instance, the following scraping job came up on oDesk a few weeks back:

> Scrape the page at https://wish.wis.ntu.edu.sg/webexe/owa/aus_subj_cont.main
>
> Specifically, retrieve all of the results returned for every selection in the second dropdown menu 
> for the most recent academic year.

In this post, I'll show a working solution to the above task in Python using
<a target="_blank" href="http://wwwsearch.sourceforge.net/mechanize/">Mechanize</a> 


Here's a screenshot of the form they want results for:

![Form Image](/assets/ntu-edu/1.png)

The first select dropdown is for selecting an academic year. The second select dropdown is for
selecting an academic department (accounting, art, business, etc.). 

Note that the first dropdown has the most recent academic year selected by default. 

If we take a look at the HTML for the form, we see the following:

{% highlight html %}
<form action="AUS_SUBJ_CONT.main_display1" method="POST" target="subjects">
  <select name="acadsem">
    <option value="2006_S">Acad Yr 2006 Special Term I</option>
    <option value="2006_T">Acad Yr 2006 Special Term II</option>
    <option value="2006_1">Acad Yr 2006 1</option>
    ...
    <option selected="" value="2014_2">Acad Yr 2014 2</option>
  </select>
  <select name="r_course_yr">
    <option value="">---Select an Option---</option>
    <option value="ACC;GA;1;F">Accountancy (GA) Year 1</option>
    <option value="ACC;GA;2;F">Accountancy (GA) Year 2</option>
    <option value="ACC;GA;3;F">Accountancy (GA) Year 3</option>
    ...
  </select>  
  ...
  <input type="button" value="Load Content of Course(s)" 
    onclick="<see below>">
</form>
{% endhighlight %}

{% highlight javascript %}
if (select_option(this.form)) {
  this.form.action='AUS_SUBJ_CONT.main_display1';
  this.form.boption.value='CLoad';
  submit()
}
{% endhighlight %}

The task then is to iterate through every item in the `r_course_yr` select dropdown, submit
the form for that item, and save the results we get back into a file.

Execute the following commands in order to install Mechanize and BeautifulSoup within a 
virtual environment: 

{% highlight bash %}
$ mkdir scraper && cd scraper
$ virtualenv venv
$ source venv/bin/activate
$ pip install mechanize
{% endhighlight %}

{% highlight python %}
#!/usr/bin/env python

import sys, signal
import mechanize, time

URL = 'https://wish.wis.ntu.edu.sg/webexe/owa/aus_subj_cont.main'
DELAY = 5

def sigint(signal, frame):
    sys.stderr.write('Exiting...\n')
    sys.exit(0)    

class NtuEduScraper:
    def __init__(self, url=URL, delay=DELAY):
        self.br = mechanize.Browser()
        self.url = url
        self.delay = delay
        self.items = []

if __name__ == '__main__':
    signal.signal(signal.SIGINT, sigint)
    scraper = NtuEduScraper()
    scraper.scrape()

{% endhighlight %}

{% highlight python %}
    def scrape(self):
        '''
        Get the list of items in the second dropdown menu and submit 
        the form for each item. Save the results to file.
        '''
        self.items = self.get_items()

        for item in self.items:
            # Skip invalid/blank item selections
            if len(item.name) < 1:
                continue

            results = self.submit_form(item)
            self.item_results_to_file(item, results)
            time.sleep(self.delay)
{% endhighlight %}

### Getting a list of items

first getting a list 
of all the items in the select control before hand and then submitting them one 
a time to get the results for each item. 

* Open the page
* Select the form
* Get a list of all the items in the second select dropdown menu

Note that the form doesn't have a name attribute. 

{% highlight html %}
<form action="AUS_SUBJ_CONT.main_display1" method="POST" target="subjects">
{% endhighlight %}

As I noted in a [previous post]({% post_url 2014-12-08-form-handling-with-mechanize-and-beautifulsoup %})
we can use a predicate function to select the form.

{% highlight python %}
def select_form(form):
    '''
    Select the course display form
    '''
    return form.attrs.get('target', None) == 'subjects'
{% endhighlight %}

Use the `get_items()` method to retrieve a list of all of the items in the select control
with the name `r_course_yr`.

{% highlight python %}
    def get_items(self):
        '''
        Get the list of items in the second dropdown of the form
        '''
        self.br.open(self.url)
        self.br.select_form(predicate=select_form)

        items = self.br.form.find_control('r_course_yr').get_items()
        return items
{% endhighlight %}


### Submitting an item and reading the results

{% highlight python %}
    def submit_form(self, item):
        '''
        Submit form using selection item.name and write the results
        to file named according to item.label
        '''
        maxtries = 3
        numtries = 0

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

        return self.br.response().read()
{% endhighlight %}

### Writing the results to file

{% highlight python %}
    def item_results_to_file(self, item, results):
        label = ' '.join([label.text for label in item.get_labels()])
        label = '-'.join(label.split())

        with open("%s.html" % label, 'w') as f:
            f.write(results)
            f.close()
{% endhighlight %}

### Conclusion

Have a small scraping task you'd like to get done for free? 

Email me the details and if I think it will make an instructive example I'll post a solution for free in a future post.
