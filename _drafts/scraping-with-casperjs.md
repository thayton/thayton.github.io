---
layout: post
title: Scraping with CasperJS
---

In a [previous post]({% post_url 2015-02-03-scraping-with-python-selenium-and-phantomjs %}), I showed how to scrape
a Javascript-heavy site by using the [Selenium](http://selenium-python.readthedocs.org/) bindings for Python to drive a headless 
browser ([PhantomJS](http://phantomjs.org/)). In this post, I'll show how to scrape the same site using [CasperJS](http://casperjs.org/).

The site being scraped is the job search page for the company [L-3 Klein Associates](http://www.l-3com.com/careers/us-job-search.html). 
They use the Taleo Applicant Tracking System and the pages are almost entirely generated via Javascript:

[https://l3com.taleo.net/careersection/l3\_ext\_us/jobsearch.ftl](https://l3com.taleo.net/careersection/l3_ext_us/jobsearch.ftl)

Click the link and you'll see a job listing like the following:

![Job Link](/assets/scraping-with-casperjs/joblink.png)

The jobs are listed inside a table whose `id` attribute is set to `jobs`:

![Jobs table](/assets/scraping-with-casperjs/jobstable.png)

{% highlight javascript %}
/**
 * Scrape job title, url, and location from Taleo jobs page at 
 * https://l3com.taleo.net/careersection/l3_ext_us/jobsearch.ftl
 *
 * Usage: $ casperjs scraper.js 
 */
var casper = require("casper").create({
    pageSettings: {
        userAgent: "Mozilla/5.0 (Macintosh; Intel Mac OS X 10.8; rv:23.0) Gecko/20130404 Firefox/23.0"
    }
});

var url = 'https://l3com.taleo.net/careersection/l3_ext_us/jobsearch.ftl';
var currentPage = 1;
var jobs = [];

var terminate = function() {
    this.echo("Exiting..").exit();
};

casper.start(url);
casper.waitForSelector('table#jobs', processPage, terminate);
casper.run();
{% endhighlight %}

Our `processPage` function will have three parts:

1. Scrape and print the jobs in the jobs table
2. Exit if we're finished scraping
3. Else, click the Next link and wait for the next page of jobs to load

{% highlight javascript %}
var processPage = function() {
    jobs = this.evaluate(getJobs);
    require('utils').dump(jobs);

    if (currentPage >= 3 || !this.exists("table#jobs")) {
        return terminate.call(casper);
    }

    currentPage++;

    this.thenClick("div#jobPager a#next").then(function() {
        this.waitFor(function() {
            return currentPage === this.evaluate(getSelectedPage);
        }, processPage, terminate);
    });
};
{% endhighlight %}

Each of the rows in this table has an `id` attribute that begins with the word job. 

[![Job Link CSS](/assets/scraping-with-casperjs/joblink_css.png)](/assets/scraping-with-casperjs/joblink_css.png)

Within each row is the url for the job. The url contains the string `jobdetail.ftl?job=`. The location for a job is 
contained within a `span` element in the column after the one containing the job link.

Based on the above attributes, we have all the information we need extract the title, url, and location for each
job in the table.

{% highlight javascript %}
function getJobs() {
    var rows = document.querySelectorAll('table#jobs tr[id^="job"]');
    var jobs = [];

    for (var i = 0, row; row = rows[i]; i++) {
        var a = row.cells[1].querySelector('a[href*="jobdetail.ftl?job="]');
        var l = row.cells[2].querySelector('span');
        var job = {};

        job['title'] = a.innerText;
        job['url'] = a.getAttribute('href');
        job['location'] = l.innerText;
        jobs.push(job);
    } 

    return jobs;       
}
{% endhighlight %}




![Pager Image](/assets/scraping-with-casperjs/pager.png)
![Pager CSS Image](/assets/scraping-with-casperjs/pager_css.png)

{% highlight javascript %}
// Return the current page by looking for the disabled page number link in the pager
function getSelectedPage() {
    var el = document.querySelector('li[class="navigation-link-disabled"]');
    return parseInt(el.textContent);
}
{% endhighlight %}