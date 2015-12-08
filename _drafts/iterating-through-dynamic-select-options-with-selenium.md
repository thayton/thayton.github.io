---
layout: post
title: Iterating through Dynamic Select Options with Selenium
---

In this post I'll show how to iterate through all the possible values in a form
that uses select dropdowns whose option values are dynamically genereated. I'll 
use Selenium and PhantomJS and show how Selenium can be used to wait for the 
option values to load. I'll wrap up the post by showing how we can refactor the
code into a more generic solution, which will be useful since this use-case arises 
a lot in scraping.

<a target="_blank" href="http://icds-wcd.nic.in/icds/icdsawc.aspx">ICDS</a>

![Form](/assets/icds/1.png)

If you try selecting an option from the *Select District* dropdown, you'll
see that the dropdown list is empty.

![District Empty](/assets/icds/2.png)

You can inspect the *Select District* element to confirm this. Note that
there is only one option: "-Select-"

![District Empty](/assets/icds/3.png)

In this form, the options in the *Select District* dropdown are populated 
dynamically after a state has been chosen. So the range of possible values 
in the district select element will always depend on the value of the current
state selection.

The *Select Project* down is the same. Its list of options is empty until
a distrct has been chosen.

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

