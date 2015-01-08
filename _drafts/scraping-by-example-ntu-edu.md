---
layout: post
title: Scraping by Example - Iterating through Select Items With Mechanize
---

A common scraping task is to get all of the results returned for every option
in a select menu on a given form.

For instance, the following scraping job came up on oDesk a few weeks back:

> Scrape the page at https://wish.wis.ntu.edu.sg/webexe/owa/aus\_subj\_cont.main
>
> Specifically, retrieve all of the results returned for every selection in the second dropdown menu 
> for the most recent academic year.

In this post, I'll show a working solution to the above task in Python using
<a target="_blank" href="http://wwwsearch.sourceforge.net/mechanize/">Mechanize</a>.

Here's a screenshot of the form they want results for:

![Form Image](/assets/ntu-edu/1.png)

The first select dropdown is for selecting an academic year. The second select dropdown is for
selecting an academic department (accounting, art, business, etc.). 

Note that the first dropdown already has the most recent academic year selected by default. 

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
    onclick="<see-below>">
</form>
{% endhighlight %}

The javascript in the `onclick` attribute of the Load Content button
is shown below. It sets the form action and a control named `boption`
before submitting the form when the button is clicked.

{% highlight javascript %}
if (select_option(this.form)) {
  this.form.action='AUS_SUBJ_CONT.main_display1';
  this.form.boption.value='CLoad';
  submit()
}
{% endhighlight %}

The task then is to iterate through every item in the `r_course_yr` select dropdown, submit
the form for that item, and then save the results we get back into a file. 

First lets lay the initial groundwork by sketching out a scraper class. 

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

Now lets add a top-level method named `scrape` that performs the following sub-tasks:

* Get a list of items from the second select menu
* Submit the form for each item
* Save the results returned for each item

{% highlight python %}
    def scrape(self):
        '''
        Get the list of items in the second dropdown menu and submit 
        the form for each item. Save the results to file.
        '''
        items = self.get_items()

        for item in items:
            # Skip invalid/blank item selections
            if len(item.name) < 1:
                continue

            results = self.submit_form(item)
            self.item_results_to_file(item, results)
{% endhighlight %}

Now let's implement each of the sub-tasks in turn.

### Part1: Getting a list of items

To get a list of items from the second select menu we need to do the following:

* Open the page
* Select the form
* Return a list of all the items in the second select dropdown menu

We can open the page and select the form with the browser object's `open` and `select_form` methods. 
However, note that the form doesn't have a name attribute. 

{% highlight html %}
<form action="AUS_SUBJ_CONT.main_display1" method="POST" target="subjects">
{% endhighlight %}

As I noted in a [previous post]({% post_url 2014-12-08-form-handling-with-mechanize-and-beautifulsoup %})
we can use a predicate function to select a form when it doesn't have a name attribute. In this case 
we'll select it based on its `target` attribute.

{% highlight python %}
def select_form(form):
    '''
    Select the course display form
    '''
    return form.attrs.get('target', None) == 'subjects'
{% endhighlight %}

Now we need to get a list of items for the `r_course_r` select control. 

First we'll retrieve a reference to the `r_course_r` control. Then, we'll use that control's `get_items()` 
method to return a list of all the items attached to that control. 

Here's the code:

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

### Part2: Submitting an item and reading the results

Now that we've gotten the items, let's write the code to submit the form
with that item selected and then read back the results. Our code will need
to do the following:

* Select the form
* Assign `item.name` to the `r_course_r` select control
* Assign value `CLoad` to the `boption` control  (like the Load button's onclick Javascript does)
* Submit the form
* Return the results

Here's the code. I've added error checking and a simple back-off/delay/retry loop in 
the event that the first couple of requests fail because we're hitting their server too quickly.

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

We've got the results. Now let's write them into a file whose named is based
off of the item whose data we're saving:

{% highlight python %}
    def item_results_to_file(self, item, results):
        label = ' '.join([label.text for label in item.get_labels()])
        label = '-'.join(label.split())

        with open("%s.html" % label, 'w') as f:
            f.write(results)
            f.close()
{% endhighlight %}

### Wrapping Up

[github](https://github.com/thayton/ntu-edu/blob/master/ntu-edu.py)

### Conclusion

Have a small scraping task you'd like to get done for free? 

Email me the details and if I think it will make an instructive example I'll develop a solution for free in a future post.
