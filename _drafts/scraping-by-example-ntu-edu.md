### Getting a list of items

Sometimes we want to get all of the results for every item in a select-dropdown.
To do so, get a list of all the items in the select control before hand and then
submit them one a time to get the results for each item. 

For instance, imagine we have the following form and want to get the results for 
each major (acct, comp, math) available for selection:

{% highlight html %}
<form name="course_schedule" action="." method="post">
  <select name="major">
    <option value="acct">Accounting</option>
    <option value="comp">Computer Science</option>
    <option value="math">Mathematics</option>
  </select>       
  ...
</form>
{% endhighlight %}

Then we can use something like the following:

{% highlight python %}
br.select_form('course_schedule')
items = br.form.find_control('major').get_items()

for item in self.items:
  label = ' '.join([label.text for label in item.get_labels()])
  print 'Getting results for ', label

  br.open(url)
  br.select_form('course_schedule')
  br.form['major'] = [ item.name ]
  br.submit()

  # the results
  print br.response().read()
{% endhighlight %}
