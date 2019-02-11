---
layout: post
title: Revisiting Taleo with Puppeteer
---

I've demonstrated how to scrape Taleo sites in a couple of my previous posts [[1]][casperjs] [[2]][phantomjs].
In those articles I used the CasperJS and Python/Selenium to scrape the Taleo job site at [https://l3com.taleo.net](https://l3com.taleo.net/careersection/l3\_ext\_us/jobsearch.ftl){:target="blank"}. In this post, I'll show how to scrape that same site again, this time using Puppeteer.

[casperjs]: {% post_url 2015-03-20-scraping-with-casperjs %}
[phantomjs]: {% post_url 2015-02-03-scraping-with-python-selenium-and-phantomjs %}

We begin by importing puppeteer and defining the URL we're going to scrape:

```javascript
const puppeteer = require('puppeteer');
const url = 'https://l3com.taleo.net/careersection/l3_ext_us/jobsearch.ftl';
```

The main logic of the scraper will go into function `main`. 

```javascript
async function main() {
    const browser = await puppeteer.launch();    
    const page = await browser.newPage();
    
    await page.goto(url);
    await waitForJobsToLoad(page);

    let pageno = 2;
    
    while (true) {
        const jobs = await getJobs(page);

        const noMorePages = await gotoNextPage(page, pageno++);
        if (noMorePages) {
            break;
        }

        /* Don't hit the server too quickly... */
        await page.waitFor(1000);
    }
    
    await browser.close();
}

main().then(() => console.log('Complete!'));
```

Inside `main` we first launch Chrome via Puppeteer and then load the Taleo site in a new page:

```javascript
    const browser = await puppeteer.launch();    
    const page = await browser.newPage();
    
    await page.goto(url);
```

Now, before we start scraping the jobs we have to wait until they'e finished loading. If you
inspect the network and HTML you'll see that the jobs are being loaded into element `table#jobs`
via an AJAX call. We need a way to determine once that loading has completed.

If you open up the site in your browser, you'll see a spinning progress indicator appears while
the jobs are being loaded and then disappears once the jobs get rendered. One approach we could
take is to wait for that 'loading' progress indicator to appear and then wait for it to go away. 

The progress indicator shows up in the `div#progressIndicator` element. If we inspect the site's
code in SearchHandler.js, we can see that the progress indicator has its width set to 0 to make it
'invisible' once the jobs have finished loading:

```javascript
   this.hideProgress = function() {
     $('#progressIndicator').width(0);
   };
```

So we can determine when the progress indicator is visible and when it has gone away by checking
that element's `offsetWidth`:

```javascript
async function waitForJobsToLoad(page) {    
    await page.waitFor(() => document.querySelector('div#progressIndicator').offsetWidth !== 0);    
    await page.waitFor(() => document.querySelector('div#progressIndicator').offsetWidth === 0);
}
```

Using this approach worked when I tested it, but I don't think it's robust as it relies on the timing
working out in our favor. If the jobs have already loaded by the time the first `waitFor` is called,
we'll end up timing out waiting for the progress indicator to appear.

In this case it's better to wait for the contents of the `span#reloadMessage` to change. This element
is where the message describing how many jobs were loaded appears. Initially the contents of the element
are an empty string but after the jobs have loaded it will be something like `Job Openings 1 - 25 of 1051`:

```javascript
var waitForJobsToLoad = (function () {
    let reloadMessage = '';
    
    return async function(page) {
        await page.waitForFunction(
            oldText => document.querySelector('span#reloadMessage').innerText !== oldText,
            {}, reloadMessage
        );
        reloadMessage = await page.$eval('span#reloadMessage', e => e.innerText);
    };
})();
```

I've used an IIFE for `waitForJobsToLoad` so that we can determine whenever the contents in the `span#reloadMessage`
element change. When we first open the jobs page, the contents of the `span#reloadMessage` element will be empty so,
which is my `reloadMessage` starts out as the empty string. As you'll see though, each time we click on the next
page the reloadMessage gets updated and that's how'll we'll monitor when the next page of jobs has finished loading:

Back in `main`, once the jobs have finished loading we scrape the jobs on the current page and then
click onto the next page, continuing until we've reached the last page.

```javascript
    let pageno = 2;
    
    while (true) {
        const jobs = await getJobs(page);

        const noMorePages = await gotoNextPage(page, pageno++);
        if (noMorePages) {
            break;
        }

        /* Don't hit the server too quickly... */
        await page.waitFor(1000);
    }
```

The `getJobs` function is where we scrape the job attributes:

```javascript
async function getJobs(page) {
    const jobs = await page.evaluate(jobSelector => {
        //debugger;
        var results = [];
        
        Array.from(document.querySelectorAll(jobSelector)).forEach((tr) => {
            th = tr.querySelector('th');
            td = tr.querySelectorAll('td');
            
            results.push({
                'title': th.innerText.trim(),
                'href': th.querySelector('a').href,
                'location': td[1].innerText.trim(),
                'postingDate': td[2].innerText.trim()
            });
        });

        return results;
    }, 'table#jobs tr[id^="job"]');

    return jobs;    
}
```

The `debugger` statement is commented out. But if you uncomment it and launch Chrome with
`headless` set to false and `devtools` set to `true`, then you can use Chrome's debugger
to step through the code in `getJobs` once execution reaches the `debugger` statement:

```javascript
    const browser = await puppeteer.launch({ headless: false, devtools: true });
```


```javascript
/*------------------------------------------------------------------------------
 * Look for link for pageno in pager. So if pageno was 6 we'd look for 'Page$6' 
 * in href:
 *
 * <a href="#" title="Go to page 6" aria-disabled="false">6</a>
 *
 * After the next page link gets clicked and the new page is loaded the pager
 * will show the current page within a span (not as a link). So we wait until 
 * pageno appears within a span to indicate that the next page has finished 
 * loading.
 */ 
async function gotoNextPage(page, pageno) {
    let noMorePages = true;
    let nextPageXp = `//ul[@class='pager']/li[@class='pagerlink']/a[text()='${pageno}']`;    
    let currPageXp = `//ul[@class='pager']/li[@class='navigation-link-disabled']/a[text()='${pageno}']`;
    let nextPage;

    nextPage = await page.$x(nextPageXp)
    
    if (nextPage.length > 0) {
        const reloadMessage = await page.$eval('span#reloadMessage', e => e.innerText);
        
        console.log(`Going to page ${pageno}`);

        await nextPage[0].click();
        await page.waitForXPath(currPageXp);

        /* 
         * The text inside the reload message span will change once the next
         * page of jobs have been loaded:
         *
         *   "The list of jobs has reloaded. 1 – 25 of 1107. Page 1"
         *
         * So we can wait for the span's text to change in order to verify the 
         * next page of jobs have been loaded into the jobs table
         */
        await page.waitForFunction(
            oldText => oldText !== document.querySelector('span#reloadMessage').innerText,
            {}, reloadMessage
        );
        
        noMorePages = false;
    }

    return noMorePages;    
}
```

