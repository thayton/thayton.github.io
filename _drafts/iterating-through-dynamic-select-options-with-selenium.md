---
layout: post
title: Iterating through Dynamic Select Options with Selenium
---

<a target="_blank" href="http://icds-wcd.nic.in/icds/icdsawc.aspx">ICDS</a>

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

