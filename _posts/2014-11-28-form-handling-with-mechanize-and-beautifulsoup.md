* Opening a page and selecting a form
* Filling in the form
* Submitting the form
* When things go wrong

### Setup
{% highlight bash %}
$ mkdir scraper && cd scraper
$ virtualenv venv
$ source venv/bin/activate
$ pip install mechanize
$ pip install beautifulsoup4
{% endhighlight %}

## Opening a page and selecting a form

{% highlight python %}
import mechanize
br = mechanize.Browser()
br.addheaders = [('User-agent',
                  'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_6_8) AppleWebKit/535.7 (KHTML, like Gecko) Chrome/16.0.912.63 Safari/535.7')]
br.set_handle_robots(False)
br.open(url)
print br.response().read()
{% endhighlight %}

### Form selection

Select a form using `select_form` with the name of the form as the argument:

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

We can select this form by filtering based on its `id`.

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

{% highlight python %}
br.select_form('searchForm')
br.form.set_all_readonly(False)
br.form['boption'] = 'CLoad'
br.form['com.peopleclick.cp.formdata.hitsPerPage'] = [ '50' ]
{% endhighlight %}

### Control selection using a predicate function

Use predicate functions in conjunction with regular-expressions to
select controls within a form that match a specific pattern:

{% highlight python %}
import re

# raytheon.py
def select_control(control):
  r = re.compile(r'ctl\d+_HiddenField')
  return bool(re.match(r, control.name))

ctl = br.form.find_control(predicate=select_control)
ctl.value = 'someval'
{% endhighlight %}

## Submitting the form

### Submitting a form

Once of you have all of the values for a form set, submit it by
calling `submit`:

{% highlight python %}
br.submit()
{% endhighlight %}

### Form submission when multiple submit buttons present

Sometimes you can't just call `submit` with no arguments because there's more than 
one submit button and you want mechanize to choose a specific one. A common scenario 
is when one of these buttons is for 'Search' and the other is for 'Reset'ing or 
'Clear'ing the form.

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

### Getting a list of items

Sometimes we want to get all of the results for every item in a select-dropdown.
To do so, get a list of all the items in the select control before hand and then
submit them one a time to get the results for each item:

{% highlight html %}
<select name="course_schedule">
<option value="acct">Accounting</option>
<option value="comp">Computer Science</option>
<option value="math">Mathematcis</option>
</select>       
{% endhighlight %}

{% highlight python %}
br.select_form(predicate=select_form)
items = br.form.find_control('course_schedule').get_items()

for item in self.items:
  label = ' '.join([label.text for label in item.get_labels()])
  print 'Getting results for ', label

  br.open(self.url)
  br.select_form(predicate=select_form)
  br.form['course'] = [ item.name ]
  br.submit()
{% endhighlight %}

## When things go wrong

### Handling 'ParseError: OPTION outside of SELECT' by prettyfing form

If you attempt to select a form but it fails with the following error:

{% highlight python %}
mechanize._form.ParseError: OPTION outside of SELECT
{% endhighlight %}

Fix the situation by running the HTML for the page through BeautifulSoup 
before selecting the form with mechanize:

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

### Setting htmlresponse with mechanize

### Adding controls

Sometimes mechanize will not pick up certain hidden form controls. You will often encounter this with ASP.NET
pages where you will see the following controls within the form:
 
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

There will also be a built-in javascript method `__doPostBack(eventTarget, eventArgument)` that will fill
in these values when the form is submitted.

{% highlight javascript %}
var theForm = document.forms['form1'];
if (!theForm) {
    theForm = document.form1;
}

function __doPostBack(eventTarget, eventArgument) {
    if (!theForm.onsubmit || (theForm.onsubmit() != false)) {
        theForm.__EVENTTARGET.value = eventTarget;
        theForm.__EVENTARGUMENT.value = eventArgument;
        theForm.submit();
    }
}
{% endhighlight %}

Later you will see this method called when a link is clicked. 

{% highlight javascript %}
<a href="javascript:__doPostBack('$next_page','')">
  Next
</a>
{% endhighlight %}

Since mechanize does not pick up these controls, yYou will need to create them yourself
to get the form submission to work:

{% highlight python %}

br.form.new_control('hidden', '__EVENTTARGET',   
  {'value': 'maincontent_0$jobsearchresults_0$next_page'})
br.form.new_control('hidden', '__EVENTARGUMENT', {'value': ''})
br.form.new_control('hidden', '__LASTFOCUS',     {'value': ''})
br.form.fixup()
{% endhighlight %}

### Adding items dynamically

{% highlight python %}
# chevron.py
br.form.set_value_by_label(['North America'], name='searchAuxRegionID')
item = Item(br.form.find_control(name='searchAuxCountryID'),
           {'contents': '3', 'value': '3', 'label': 3})
br.form['searchAuxCountryID'] = ['3']
{% endhighlight %}


### Removing controls to make form submissions work

{% highlight python %}
# raytheon.py
for control in br.form.controls[:]:
  if control.type in ['submit', 'image', 'checkbox']:
    br.form.controls.remove(control)
{% endhighlight %}

