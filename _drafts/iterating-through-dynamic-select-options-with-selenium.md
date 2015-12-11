---
layout: post
title: Iterating through Dynamic Select Options with Selenium
---

In this post I'll show how to iterate through all of the dropdown menus in a form
that uses [SELECT] (http://www.w3schools.com/tags/tag_select.asp) elements whose 
option values are dynamically generated. I'll use [Selenium](https://selenium-python.readthedocs.org/) and [PhantomJS](http://phantomjs.org/) 
and show how Selenium can be used to wait for the option values to load. I'll 
wrap up the post by showing how we can refactor the code into a more generic solution, 
which will be useful since this use-case arises frequently in scraping.

## Background 

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
until a state has been chosen. This means that the range of possible 
values in the district SELECT element will always depend on the current value 
of the state selection.

Same thing goes for the *Select Project* dropdown:

![Project Empty](/assets/icds/4.png)

Its list of options is only filled *after* a district has been chosen.

To summarize how this form operates: first we select a *state*. That triggers 
an update which causes the *district* options to load. Then we select a *district*, 
which triggers another update to dynamically load the *project* values.

With that in mind, here's the pseudocode sketch of the solution we will develop. 

<a name="pseudocode"></a>

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

## Implementation

Let's begin. Here's the boilerplate for our scraper. It assumes that you 
have [Selenium](https://selenium-python.readthedocs.org/) and [PhantomJS](http://phantomjs.org/) 
installed:

{% highlight python %}
#!/usr/bin/env python

import sys
import signal

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

First we create a *load\_page* method which retrieves the form page 
and waits until the state SELECT element is rendered before returning:

{% highlight python %}
def load_page(self):
    self.driver.get(self.url)

    def page_loaded(driver):
        path = '//select[@id="ctl00_ContentPlaceHolder1_dropstate"]'
        return driver.find_element_by_xpath(path)

    wait = WebDriverWait(self.driver, 10)
    wait.until(page_loaded)            
{% endhighlight %}

Next, let's write a *scrape* method to implement the [pseudocode](#pseudocode)
presented above:

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

The *scrape* method uses generators to iterate through all of the
option values for the *state*, *district* and *project* SELECT elements.

Let's take a look at the *states* generator:

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

The *states* function generates a list of all the option values contained
in the state SELECT element. Then it uses *yield* to allow the caller to iterate
through that list, selecting the next option each time *states* gets called.

The *districts* and *projects* generators are implemented the same way:

{% highlight python %}
def districts():
    district_select = self.get_district_select()
    district_select_option_values = [ 
        '%s' % o.get_attribute('value') 
        for o 
        in district_select.options 
        if o.text != '-Select-' 
    ]

    for v in district_select_option_values:
        district_select = self.select_district_option(v)
        yield district_select.first_selected_option.text
            
def projects():
    project_select = self.get_project_select()
    project_select_option_values = [ 
        '%s' % o.get_attribute('value') 
        for o 
        in project_select.options[1:]
    ]

    for v in project_select_option_values:
        project_select = self.select_project_option(v)
        yield project_select.first_selected_option.text
{% endhighlight %}

There are two types of helper methods used by the generators.
Both types follow the same patterns:

- Get a reference to a SELECT element
- Select one of the options

Let's look at the first type: methods used to get a reference to a SELECT
element. Here is the code for *get\_state\_select* which returns a reference
to the state SELECT element:

{% highlight python %}
def get_state_select(self):
    path = '//select[@id="ctl00_ContentPlaceHolder1_dropstate"]'
    state_select_elem = self.driver.find_element_by_xpath(path)
    state_select = Select(state_select_elem)
    return state_select
{% endhighlight %}

It looks up a reference to an element given its xpath. Then it uses
the [Select](https://selenium.googlecode.com/git/docs/api/py/webdriver_support/selenium.webdriver.support.select.html)
constructor to create an instance of the WebDriver Select support 
class which is used to interact with SELECT elements.

We get references to the district and project SELECT elements the
same way:

{% highlight python %}
def get_district_select(self):
    path = '//select[@id="ctl00_ContentPlaceHolder1_dropdistrict"]'
    district_select_elem = self.driver.find_element_by_xpath(path)
    district_select = Select(district_select_elem)
    return district_select

def get_project_select(self):
    path = '//select[@id="ctl00_ContentPlaceHolder1_dropproject"]'
    project_select_elem = self.driver.find_element_by_xpath(path)
    project_select = Select(project_select_elem)
    return project_select
{% endhighlight %}

Now let's take a look at the methods used to select option values.
First we'll examine *select\_state\_option*:

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

This method selects a state value and then waits for the district options
to load. It determines when the district options have loaded by:

1. Getting a reference to the district SELECT element
2. Selecting an option in the state select dropdown
3. Waiting for the district SELECT element from step 1 to return a *StaleElementReferenceException*
   when we reference its *text* attribute.

When we select a state, the [DOM](http://www.w3.org/DOM/) gets updated with new district 
options corresponding to the chosen state. That causes any old references to the district 
SELECT to become "stale." 

That means there's an easy litmus test to determine when a select dropdown has fininshed loading: 
wait for any old references to it to become stale.

We'll repeat this same pattern for the district and project SELECT elements:

{% highlight python %}
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
{% endhighlight %}

The project dropdown doesn't trigger any updates when we select 
a value, so its implementation is very simple:

{% highlight python %}
def select_project_option(self, value, dowait=True):
    project_select = self.get_project_select()
    project_select.select_by_value(value)
    return self.get_project_select()
{% endhighlight %}

Now we have a working implementation. This is version 1 of our scraper.

You can view the source for this version at:

[https://github.com/thayton/icds/blob/master/v1.scraper.py](https://github.com/thayton/icds/blob/master/v1.scraper.py)

Let's try running it:

```
$ ./v1.scraper.py
Andaman & Nicobar Islands
   Nicobars
   North  & Middle Andaman
   South Andaman
Andhra Pradesh
   Adilabad
     Adilabad U
     Asifabad
     Bellampally
     Boath
     Chennur
     JAINOOR
     Kaghaznagar
     Khanapur
     ...
```

## Refactoring

In this section, I'm going to show how we can refactor the code we just 
developed into a more concise implementation.

First, the *get\_state\_select*, *get\_district\_select* and *get\_state\_select* methods
can all be replaced with a generic *get_select* method that takes the xpath of the 
SELECT element as an argument.

{% highlight python %}
def get_select(self, xpath):
    select_elem = self.driver.find_element_by_xpath(xpath)
    select = Select(select_elem)
    return select
{% endhighlight %}

Next, if you take a look at the code for selecting the state and district option values
(*select\_state\_option* and *select\_district\_option*) you might notice that both 
methods are repeating the following pattern:

- Select an option
- Wait for some other SELECT element's options to load

This pattern appears frequently in forms with SELECT elements whose option values are dynamically
generated.

We can refactor this pattern into something more generic. 

We'll replace *select\_state\_option*, *select\_district\_option*, and 
*select\_project\_option* methods with a new generic *select\_option* method. 

Our new method takes the xpath of a SELECT element, an option value to choose, and the 
xpath of another SELECT element that we'll wait to see updated before returning.

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

The *select_option* method uses *make\_waitfor\_elem\_updated\_predicate* to generate 
a function that will wait for an element to become stale in the current document. 

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

Now we can do something like the following to select a state and then wait
for the district options to load:

{% highlight python %}
# '35' is the option value of the state "Andaman & Nicobar Islands"
self.select_option(
    '//select[@id="ctl00_ContentPlaceHolder1_dropstate"]',
    '35',
    '//select[@id="ctl00_ContentPlaceHolder1_dropdistrict"]'
)
{% endhighlight %}

Now let's revisit the *states()*, *districts()* and *projects()* generators. They 
can all be refactored since they also follow a common pattern:

- Get a reference to a SELECT element
- Generate its list of values
- Iterate through those values

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

Now we can make generators for the states, districts and projects
with just a few lines of code:

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

Here is our final implementation. It is much more concise than our
earlier version:

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

class Scraper(object):
    def __init__(self):
        self.url = 'http://icds-wcd.nic.in/icds/icdsawc.aspx'
        self.driver = webdriver.PhantomJS()
        self.driver.set_window_size(1120, 550)

    def get_select(self, xpath):
        select_elem = self.driver.find_element_by_xpath(xpath)
        select = Select(select_elem)
        return select

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

    def load_page(self):
        self.driver.get(self.url)

        def page_loaded(driver):
            path = '//select[@id="ctl00_ContentPlaceHolder1_dropstate"]'
            return driver.find_element_by_xpath(path)

        wait = WebDriverWait(self.driver, 10)
        wait.until(page_loaded)            

    def scrape(self):
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

        self.load_page()

        for state in states():
            print state
            for district in districts():
                print 2*' ', district
                for project in projects():
                    print 4*' ', project

if __name__ == '__main__':
    signal.signal(signal.SIGINT, sigint)
    scraper = Scraper()
    scraper.scrape()
{% endhighlight %}

## Shameless Plug

Have a scraping project you'd like done? I'm available for hire. [Contact me](/contact) 
for a free quote.
