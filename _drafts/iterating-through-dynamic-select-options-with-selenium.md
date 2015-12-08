---
layout: post
title: Iterating through Dynamic Select Options with Selenium
---

In this post I'll show how to iterate through all the possible values in a form
that uses [select elements](http://www.w3schools.com/tags/tag_select.asp) whose 
option values are dynamically genereated. I'll use [Selenium](https://selenium-python.readthedocs.org/) and [PhantomJS](http://phantomjs.org/) 
and show how Selenium can be used to wait for the option values to load. I'll 
wrap up the post by showing how we can refactor the code into a more generic solution, 
which will be useful since this use-case arises a lot in scraping.

The site I'll use in today's example is at the following URL:

<a target="_blank" href="http://icds-wcd.nic.in/icds/icdsawc.aspx">http://icds-wcd.nic.in/icds/icdsawc.aspx</a>

Click on the link and you'll be presented with the following form. 

![Form](/assets/icds/1.png)

Our object is to iterate through all of the State, District and Project dropdowns
and print out their option values. 

Here's the catch though: if you try selecting an option from the *Select District* 
dropdown, you'll see that it's empty.

![District Empty](/assets/icds/2.png)

You can inspect the *Select District* element to confirm this. Note that
there is only one option: "-Select-"

![District Empty](/assets/icds/3.png)

In this form, the options in the *Select District* dropdown aren't populated 
until a state has been chosen. This means the range of possible 
values in the district select element will always depend on the value of the 
current state selection.

Same thing goes for the *Select Project* dropdown.

![Project Empty](/assets/icds/4.png)

Its list of options is only filled *after* a district has been chosen.

With that in mind, here's the pseudocode sketch of the solution we will develop. 

```
for each state
  select state
  wait for district options to load

  for each district
    select district
    wait for project options to load

    for each project
      print state, district, project  
```

{% highlight python %}
#!/usr/bin/env python

import sys
import signal

from bs4 import BeautifulSoup
from selenium import webdriver
from selenium.webdriver.support.ui import Select
from selenium.webdriver.support.ui import WebDriverWait
from selenium.common.exceptions import StaleElementReferenceException

def sigint(signal, frame):
    sys.exit(0)

class Scraper(object):
    def __init__(self):
        self.url = 'http://icds-wcd.nic.in/icds/icdsawc.aspx'
        self.driver = webdriver.PhantomJS()
        self.driver.set_window_size(1120, 550)

if __name__ == '__main__':
    signal.signal(signal.SIGINT, sigint)
    scraper = Scraper()
    scraper.scrape()
{% endhighlight %}

{% highlight python %}
def load_page(self):
    self.driver.get(self.url)

    def page_loaded(driver):
        path = '//select[@id="ctl00_ContentPlaceHolder1_dropstate"]'
        return driver.find_element_by_xpath(path)

    wait = WebDriverWait(self.driver, 10)
    wait.until(page_loaded)            
{% endhighlight %}

{% highlight python %}
def scrape(self):
    self.load_page()

    for state in states():
        print state

        for district in districts():
            print 2*' ', district

            for project in projects():
                print 4*' ', project
{% endhighlight %}

{% highlight python %}
def states():
    state_select = self.get_state_select()
    state_select_option_values = [ 
        '%s' % o.get_attribute('value') 
        for o 
        in state_select.options[1:]
    ]

    for v in state_select_option_values:
        state_select = self.select_state_option(v)
        yield state_select.first_selected_option.text
{% endhighlight %}

{% highlight python %}
def get_state_select(self):
    path = '//select[@id="ctl00_ContentPlaceHolder1_dropstate"]'
    state_select_elem = self.driver.find_element_by_xpath(path)
    state_select = Select(state_select_elem)
    return state_select
{% endhighlight %}

{% highlight python %}
def select_state_option(self, value, dowait=True):
    '''
    Select state value from dropdown. Wait until district dropdown
    has loaded before returning.
    '''
    path = '//select[@id="ctl00_ContentPlaceHolder1_dropdistrict"]'
    district_select_elem = self.driver.find_element_by_xpath(path)

    def district_select_updated(driver):
        try:
            district_select_elem.text
        except StaleElementReferenceException:
            return True
        except:
            pass

        return False

    state_select = self.get_state_select()
    state_select.select_by_value(value)

    if dowait:
        wait = WebDriverWait(self.driver, 10)
        wait.until(district_select_updated)

    return self.get_state_select()
{% endhighlight %}

The method determines when the district options have loaded by:

1. Getting a reference to the district select element
2. Selecting an option in the state select dropdown
3. Waiting for the district select element from step 1 to return a StaleElementReferenceException
   when we reference its *text* attribute.

Essentially, we get a reference to the district select element *before* it has been 
dynamically updated, and then wait for that reference to become stale *after*
we select a state.

{% highlight python %}
#--- DISTRICT --------------------------------------------------
def get_district_select(self):
    path = '//select[@id="ctl00_ContentPlaceHolder1_dropdistrict"]'
    district_select_elem = self.driver.find_element_by_xpath(path)
    district_select = Select(district_select_elem)
    return district_select

def select_district_option(self, value, dowait=True):
    '''
    Select district value from dropdown. Wait until district dropdown
    has loaded before returning.
    '''
    path = '//select[@id="ctl00_ContentPlaceHolder1_dropdistrict"]'
    district_select_elem = self.driver.find_element_by_xpath(path)

    def district_select_updated(driver):
        try:
            district_select_elem.text
        except StaleElementReferenceException:
            return True
        except:
            pass

        return False

    district_select = self.get_district_select()
    district_select.select_by_value(value)

    if dowait:
        wait = WebDriverWait(self.driver, 10)
        wait.until(district_select_updated)

    return self.get_district_select()

#--- PROJECT ---------------------------------------------------
def get_project_select(self):
    path = '//select[@id="ctl00_ContentPlaceHolder1_dropproject"]'
    project_select_elem = self.driver.find_element_by_xpath(path)
    project_select = Select(project_select_elem)
    return project_select

def select_project_option(self, value, dowait=True):
    project_select = self.get_project_select()
    project_select.select_by_value(value)
    return self.get_project_select()
{% endhighlight %}

Note that for both states and districts we are repeating the following pattern:

- Select an option
- Wait for next select element's options to load

We can refactor this pattern out into something more generic. 

The following function looks up an element given an xpath and then returns a function that 
will wait for that element to become stale in the current document. 

{% highlight python %}
def make_waitfor_elem_updated_predicate(driver, waitfor_elem_xpath):
    elem = driver.find_element_by_xpath(waitfor_elem_xpath)

    def elem_updated(driver):
        try:
            elem.text
        except StaleElementReferenceException:
            return True
        except:
            pass

        return False

    return lambda driver: elem_updated(driver)
{% endhighlight %}

Now we don't have to have three different methods (*select\_state\_option*, *select\_district\_option*, and *select\_project\_option*)
that all repeat the same basic pattern. 

Instead, we can make a generic *select_option* method that selects an option from 
a dropdown and then waits for another dropdown select finished loading by waiting
for it to become stale.

{% highlight python %}
def select_option(self, xpath, value, waitfor_elem_xpath=None):
    if waitfor_elem_xpath:
        func = make_waitfor_elem_updated_predicate(
            self.driver, 
            waitfor_elem_xpath
        )

    select = self.get_select(xpath)
    select.select_by_value(value)

    if waitfor_elem_xpath:
        wait = WebDriverWait(self.driver, 10)
        wait.until(func)

    return self.get_select(xpath)
{% endhighlight %}

We can refactor the *states()*, *districts()* and *projects()* generators using this
new method:

{% highlight python %}
def make_select_option_iterator(self, xpath, waitfor_elem_xpath):
    def next_option(xpath, waitfor_elem_xpath):
        select = self.get_select(xpath)
        select_option_values = [ 
            '%s' % o.get_attribute('value') 
            for o 
            in select.options 
            if o.text != '-Select-'
        ]

        for v in select_option_values:
            select = self.select_option(xpath, v, waitfor_elem_xpath)
            yield select.first_selected_option.text

    return lambda: next_option(xpath, waitfor_elem_xpath)
{% endhighlight %}

{% highlight python %}
states = self.make_select_option_iterator(
    '//select[@id="ctl00_ContentPlaceHolder1_dropstate"]',
    '//select[@id="ctl00_ContentPlaceHolder1_dropdistrict"]'
)

districts = self.make_select_option_iterator(
    '//select[@id="ctl00_ContentPlaceHolder1_dropdistrict"]',
    '//select[@id="ctl00_ContentPlaceHolder1_dropdistrict"]'
)

projects = self.make_select_option_iterator(
    '//select[@id="ctl00_ContentPlaceHolder1_dropproject"]',
    None
)
{% endhighlight %}

