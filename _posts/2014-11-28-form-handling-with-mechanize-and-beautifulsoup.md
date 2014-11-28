### Setup
{% highlight bash %}
$ mkdir scraper && cd scraper
$ virtualenv venv
$ source venv/bin/activate
$ pip install mechanize
$ pip install beautifulsoup4
{% endhighlight %}

{% highlight python %}
import mechanize
br = mechanize.Browser()
{% endhighlight %}

### Setting the user-agent

{% highlight python %}
br.addheaders = [('User-agent',
                  'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_6_8) AppleWebKit/535.7 (KHTML, like Gecko) Chrome/16.0.912.63 Safari/535.7')]
{% endhighlight %}

### Handling robots

{% highlight python %}
br.set_handle_robots(False)
{% endhighlight %}

### Form selection

Select a form using select_form() with the name of the form as the argument:

{% highlight python %}

{% endhighlight %}

### Form selection using a predicate function

If the form does not have a name attribute, you can use a predicate function to select the form based off
of one of its other attributes. The following form has no name attribute:

{% highlight html %}
<form method="post" action="/en/search/" id="form1">
{% endhighlight %}

So we'll select this form by filtering based on its id.

{% highlight python %}
def select_form(form):
  return form.attrs.get('id', None) == 'form1'

br.select_form(predicate=select_form)
br.submit()
{% endhighlight %}

We could have selected against the form's action instead:

{% highlight python %}
def select_form(form):
  return form.attrs.get('action', None) == '/en/search/'

br.select_form(predicate=select_form)
br.submit()
{% endhighlight %}

### Selecting links using a predicate function

You can select links in a similar fashion. The following code
selects and follows an iframe using a predicate function to identify
the iframe based on its id:

{% highlight python %}
def select_iframe(iframe):
  return dict(iframe.attrs).get('id', None) == 'main'

br.follow_link(br.find_link(tag='iframe', predicate=select_iframe))
{% endhighlight %}

### control selection using a predicate function

Use predicate functions in conjunction with regular-expressions to
select controls with a form that match a specific pattern:

{% highlight python %}
import re

# raytheon.py
def select_control(control):
  r = re.compile(r'ctl\d+_HiddenField')
  return bool(re.match(r, control.name))

ctl = br.form.find_control(predicate=select_control)
ctl.value = 'someval'
{% endhighlight %}

### Submitting a form

Once of you have all of the values for a form set, submit it by
calling submit():

{% highlight python %}
br.submit()
{% endhighlight %}

### Form submission when multiple submit buttons present

Sometimes you can't just call submit() because there's more than one submit button 
and you want mechanize to choose a specific one. A common scenario is when one of 
these buttons is for 'Search' and the other will be for 'Reset'ing the form:

{% highlight %}
<form name="searchForm" method="post" action="search.do">
  ...
  <input type="image" name="reset" src="Reset.gif" alt="Reset Form">
  <input type="image" name="input" src="Search.gif" id="searchButton" alt="Search">
  ...
</form>
{% endhighlight %}

In this case, pass the id of the control desired when calling submit():

{% highlight python %}
br.select_form('searchForm')
br.submit(id='searchButton')
{% endhighlight %}

or specify the name:

{% highlight python %}
br.select_form('searchForm')
br.submit(name='input')
{% highlight python %}
{% endhighlight %}

### putting response into BeautifulSoup

### Simulating AJAX requests

{% highlight python %}
# arbornetworks.py
url = 'http://arbornetworks.jobs/ajax/joblisting/?num_items=50&offset=0'

d = url_query_get(url, ['num_items', 'offset'])
num_items = int(d['num_items'])
offset = int(d['offset'])

br.addheaders = [('X-Requested-With', 'XMLHttpRequest')]
br.open(url)

...

# Next page of items  
offset += num_items
u = url_query_add(br.geturl(), {'offset': '%d' % offset}.items())
br.open(u)

# apple.py
data_orig = { 'searchRequestJson': '{"searchString":"","jobType":1,"filters":{"locations":null,"retailJobSpecs":null,"businessLine":null,"jobFunctions":null,"languageSkills":null},"sortBy":"req_open_dt","sortOrder":"1","pageNumber":"%d"}',
               'csrfToken': 'null',
               'clientOffset': '-300'
            }

data = data_orig.copy()
data['searchRequestJson'] = data['searchRequestJson'] % pageno
pageno += 1

data = urllib.urlencode(data)
resp = mechanize.Request('https://jobs.apple.com/us/search/search-result', data)

r = mechanize.urlopen(resp)
{% endhighlight %}

### handling 'ParseError: OPTION outside of SELECT' by prettyfing form

{% highlight python %}
# mechanize._form.ParseError: OPTION outside of SELECT
# Yale.py / brassring.py
  
s = soupify(br.response().read())
# html = s.prettify()                                                                                                                                                   
html = str(s)
resp = mechanize.make_response(html, [("Content-Type", "text/html")],
                               br.geturl(), 200, "OK")
br.set_response(resp)
{% endhighlight %}

### setting htmlresponse with mechanize

### adding controls

{% highlight python %}
# raytheon.py
br.form.new_control('hidden', '__EVENTTARGET',   {'value': m.group(1)})
br.form.new_control('hidden', '__EVENTARGUMENT', {'value': ''})
br.form.new_control('hidden', '__LASTFOCUS',     {'value': ''})
br.form.fixup()
{% endhighlight %}

### removing controls to make form submissions work

{% highlight python %}
# raytheon.py
for control in br.form.controls[:]:
  if control.type in ['submit', 'image', 'checkbox']:
    br.form.controls.remove(control)
{% endhighlight %}

### getting a list of items

{% highlight python %}
br.select_form(predicate=select_form)
items = br.form.find_control('r_course_yr').get_items()

for item in self.items:
  label = ' '.join([label.text for label in item.get_labels()])
  print 'Getting results for ', label

  br.open(self.url)
  br.select_form(predicate=select_form)
  br.form['r_course_yr'] = [ item.name ]
  br.submit()
{% endhighlight %}
