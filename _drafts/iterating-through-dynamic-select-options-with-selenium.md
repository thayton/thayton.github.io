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

With this in mind, here's a pseudocode sketch of the solution we will develop. 

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

{% highlight python %}
def make_select_option_iterator(self, xpath, waitfor_elem_xpath):
    def next_option(xpath, waitfor_elem_xpath):
        select = self.get_select(xpath)
        select_option_values = [ '%s' % o.get_attribute('value') for o in select.options[1:] if o.text != '-Select-']

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

