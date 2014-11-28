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

### Form selection using a predicate function

{% highlight python %}
# raytheon.py
def select_form(form):
  return form.attrs.get('id', None) == 'form1'

br.select_form(predicate=select_form)
br.form.set_all_readonly(False)

# ergo.py
def select_form(form):
  return form.attrs.get('action', None) == '/ergo/de/result'
{% endhighlight %}

### Selecting links using a predicate function

{% highlight python %}
  def select_iframe(iframe):
    return dict(iframe.attrs).get('id', None) == 'main'

  br.follow_link(br.find_link(tag='iframe', predicate=select_iframe))
{% endhighlight %}

### control selection using a predicate function

{% highlight python %}
  # raytheon.py
  def select_control(control):
    r = re.compile(r'ctl\d+_HiddenField')
    return bool(re.match(r, control.name))

  ctl = br.form.find_control(predicate=select_control)
  ctl.value = 'someval'
{% endhighlight %}

### setting form action

### form submission when multiple submit buttons present

{% highlight python %}
  find . -name "*.py" | xargs grep -E "submit\([^)]"

  ./peopleclick/siemens.py:        br.submit(id='searchButton')
  ./peopleclick/siemens.py:        br.submit(name=i['name'])
  ./peopleclick/sourcefire.py:     br.submit(name='input')
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

### sending ajax requests
