---
layout: post
title: Form Handling With Mechanize And Beautifulsoup
---

Python <a target="_blank" href="http://wwwsearch.sourceforge.net/mechanize/">Mechanize</a> 
is a module that provides an API for programmatically browsing web pages and manipulating 
HTML forms. <a target="_blank" href="http://www.crummy.com/software/BeautifulSoup/">BeautifulSoup</a> 
is a library for parsing and extracting data from HTML. Together they form a powerful 
combination of tools for web scraping.

In this post, I'll highlight some of the features that Mechanize provides for
manipulating HTML forms. Specifically, I'll go over the following:

* Opening a page with mechanize and selecting a form
* Filling in the form
* Submitting the form
* What to do when things go wrong

### Setup

Execute the following commands in order to install Mechanize and BeautifulSoup within a 
virtual environment: 

{% highlight bash %}
$ mkdir scraper && cd scraper
$ virtualenv venv
$ source venv/bin/activate
$ pip install mechanize
$ pip install beautifulsoup4
{% endhighlight %}

## Opening a page and selecting a form

To open a page with mechanize, instantiate a `Browser` and provide the url of the page
to the `open` method:

### Opening a page

{% highlight python %}
#!/usr/bin/env python

import mechanize

br = mechanize.Browser()
br.open('http://thayton.github.io')

response = br.response()

print response.geturl() # URL of the page we just opened
print response.info()   # headers
print response.read()   # body
{% endhighlight %}

### Form selection

Once you've opened a page, select a form using `select_form` with the name of the form as the argument:

{% highlight python %}
br.select_form('searchForm')
{% endhighlight %}

### Form selection using a predicate function

If the form does not have a name attribute, you can use a predicate function to select the form based off
of one of its other attributes. 

For instance, the following form has no `name` attribute:

{% highlight html %}
<form method="post" action="/en/search/" id="form1">
{% endhighlight %}

We can select this form by filtering on its `id`.

{% highlight python %}
def select_form(form):
  return form.attrs.get('id', None) == 'form1'

br.select_form(predicate=select_form)
br.submit()
{% endhighlight %}

Or we can select against the form's `action` instead:

{% highlight python %}
def select_form(form):
  return form.attrs.get('action', None) == '/en/search/'

br.select_form(predicate=select_form)
br.submit()
{% endhighlight %}

## Filling in the form

### Setting control values by name

After you have selected a form, fill it in by providing the name of the form control you 
want to set along with its associated value using the following format:

{% highlight python %}
form[name] = value
{% endhighlight %}

For example,

{% highlight python %}
br.form['boption'] = 'CLoad'
br.form['hitsPerPage'] = [ '50' ]
{% endhighlight %}

That's equivalient to:

{% highlight python %}
self.br.form.set_value('CLoad', 'boption')
self.br.form.set_value(['50'], 'hitsPerPage')
{% endhighlight %}

Which is also equivalent to:

{% highlight python %}
ctl =  self.br.form.find_control('boption')
ctl.value = 'CLoad'

ctl =  self.br.form.find_control('hitsPerPage')
ctl.value = ['50']
{% endhighlight %}

If you get the read-only error 

`ValueError: control '<em>control_name</em>' is readonly` 

when filling out the form, change the control's readonly attribute to `False`:

{% highlight python %}
br.form.find_control('control_name').readonly = False
{% endhighlight %}

or call `set_all_readonly(False)` on the form to make all of the controls writable.

{% highlight python %}
br.form.set_all_readonly(False)
{% endhighlight %}

### Control selection using a predicate function

You can also use predicate functions in conjunction with regular-expressions to
select controls within a form that match a specific pattern:

{% highlight python %}
import re

def select_control(control):
  r = re.compile(r'ctl\d+_HiddenField')
  return bool(re.match(r, control.name))

ctl = br.form.find_control(predicate=select_control)
ctl.value = 'someval'
{% endhighlight %}

## Submitting the form

### Submitting a form

Once you've filled out all of the form values, submit the form by
calling `submit`:

{% highlight python %}
br.submit()
{% endhighlight %}

### Form submission when multiple submit buttons present

Sometimes you can't just call `submit` with no arguments because there's more than 
one submit button and you want mechanize to choose a specific one. A common scenario 
is when one of the buttons is for 'Search' and the other is for Resetting or 
Clearing the form.

{% highlight html %}
<form name="searchForm" method="post" action="search.do">
  ...
  <input type="image" name="reset" src="Reset.gif" alt="Reset Form">
  <input type="image" name="input" src="Search.gif" id="searchButton" alt="Search">
  ...
</form>
{% endhighlight %}

In this case, pass the id of the desired control when calling `submit`:

{% highlight python %}
br.select_form('searchForm')
br.submit(id='searchButton')
{% endhighlight %}

or specify the name:

{% highlight python %}
br.select_form('searchForm')
br.submit(name='input')
{% endhighlight %}

## When things go wrong

Here are some common issues you may encounter when handling forms with mechanize.

### Handling 'ParseError: OPTION outside of SELECT'

If you attempt to select a form but it fails with the following error:

{% highlight python %}
mechanize._form.ParseError: OPTION outside of SELECT
{% endhighlight %}

Fix the situation by running the page through BeautifulSoup before selecting the 
form with mechanize:

{% highlight python %}
import mechanize
from bs4 import BeautifulSoup

br = mechanize.Browser()
br.open(url)

soup = BeautifulSoup(br.response().read())
html = str(soup)
resp = mechanize.make_response(html, [("Content-Type", "text/html")],
                               br.geturl(), 200, "OK")
br.set_response(resp)
br.select_form('aspnetForm')
br.submit()
{% endhighlight %}

### Adding controls

Sometimes mechanize won't pick up certain hidden form controls. I've encountered this with ASP.NET
pages where mechanize won't pick up the `__EVENTTARGET`, `__EVENTARGUMENT`, and `__LASTFOCUS` controls 
in a form like the following:
 
{% highlight html %}
<form name="ctl00" method="post" action="search" id="ctl00"> 
  <div class="aspNetHidden">
    <input type="hidden" name="__EVENTTARGET" id="__EVENTTARGET" value="">
    <input type="hidden" name="__EVENTARGUMENT" id="__EVENTARGUMENT" value="">
    <input type="hidden" name="__LASTFOCUS" id="__LASTFOCUS" value="">
    <input type="hidden" name="__VIEWSTATE" id="__VIEWSTATE" value="/wEPD....">
  </div>
</form>
{% endhighlight %}

Since mechanize doesn't pick up these controls, you will need to create them manually
in order to get the form submission to work. Use the `new_control` method to add these 
controls to the form and set their values at the same time:

{% highlight python %}
br.form.new_control('hidden', '__EVENTTARGET',   {'value': '$next_page'})
br.form.new_control('hidden', '__EVENTARGUMENT', {'value': ''})
br.form.new_control('hidden', '__LASTFOCUS',     {'value': ''})
br.form.fixup()
{% endhighlight %}

### Adding items dynamically

Sometimes a control's values are set dynamically via javascript. So when you go
to set the values for these controls with mechanize, there's no list of items to choose
from because mechanize can't run the javascript that would have created them in the 
first place.

I've encountered this with forms that dynamically generate a select drop down for region,
country, state, city, etc. Here's an example of that scenario:

{% highlight html %}
<select id="searchAuxRegionID" name="searchAuxRegionID" 
  onchange="PopulateCombo(document.frmSearch.searchAuxCountryID);">

  <option value="">-- Select --</option>
  <option value="1">Africa</option>
  <option value="3">Asia</option>
  <option value="4">Australia</option>
  <option value="5">Europe</option>
  <option value="6">North America</option>
</select>
<select id="searchAuxCountryID" name="searchAuxCountryID" 
  onchange="PopulateCombo(document.frmSearch.searchAuxStateID);">

  <option value="">-- Select --</option>
</select>
{% endhighlight %}

Note in the above form that the second select list for country is empty. That's because it doesn't get populated until 
you've chosen a value for region from the first select list.

Once you select the region, it will trigger javascript code in the onchange attribute to dynamically generate a 
list of country items. 

For instance, say I select North America from the dropdown. This causes the searchAuxCountryID select dropdown to 
be populated with the following list of values. 

{% highlight html %}
<select id="searchAuxCountryID" name="searchAuxCountryID" onchange="PopulateCombo(document.frmSearch.searchAuxStateID);">
  <option value="">-- Select --</option>
  <option value="4">Canada</option>
  <option value="3">USA</option>
</select>
{% endhighlight %}

Now I can select which country I want to submit in the form.

But how can we handle this in mechanize where the javascript code won't be running? 

Can you just try setting the value of the `searchAuxCountryID` to `3` or `4`? 

Try it and you will see that it will fail with the error `*** ItemNotFoundError: insufficient items with name '<your-value>'`:

{% highlight python %}
(Pdb) print self.br.form

<frmSearch POST https://www.example.com/search application/x-www-form-urlencoded
  <SelectControl(searchAuxRegionID=[, 1, 3, 4, 5, *6])>
  <SelectControl(searchAuxCountryID=[*])>
  <SelectControl(searchAuxStateID=[*])>
  <SelectControl(searchAuxCityID=[*])>>

(Pdb) self.br.form['searchAuxCountryID'] = ['3']

*** ItemNotFoundError: insufficient items with name '3'
{% endhighlight %}

What you must do is create an `Item` for the value manually and attach it to the 
control you want it to be part of. In this case we want it to be an item with a value of `3`
within the `searchAuxCountryID` control:

{% highlight python %}
item = Item(br.form.find_control(name='searchAuxCountryID'),
           {'contents': '3', 'value': '3', 'label': 3})

# Now it works!
br.form['searchAuxCountryID'] = ['3']
{% endhighlight %}

### Removing controls to make form submissions work

All of the previous examples have assumed that we need to add controls to make a form submission work. But sometimes
you will encounter cases where you need to remove controls in order to get the form submission to work. 

Remove controls by calling `br.form.controls.remove` with the control you wish to delete as the argument. 

Here's an example where we remove all of the submit, image, or checkbox controls from the form:

{% highlight python %}
for control in br.form.controls[:]:
  if control.type in ['submit', 'image', 'checkbox']:
    br.form.controls.remove(control)
{% endhighlight %}

Note that in the above example we are iterating through all of the controls in the form and filtering
based on the type of the control. We can also look for specific controls to remove by first finding
the control by name (or predicate function) and then removing it:

{% highlight python %}
ctl = br.form.find_control('ctl00$cphMain$btnSearch')
br.form.controls.remove(ctl)
{% endhighlight %}

## Shameless Plug

Have a scraping project you'd like done? I'm available for hire. [Contact me](/contact) 
with some details about your project and I'll give you a quote.
