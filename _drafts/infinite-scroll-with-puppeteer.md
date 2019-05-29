---
layout: post
title: Scraping by Example - Infinite Scroll with Puppeteer
---

In this post I'll demonstrate how to scrape a site that uses infinite scroll. The site I'll use for this
article is the jobs page at:

[https://finning.wd3.myworkdayjobs.com/External](https://finning.wd3.myworkdayjobs.com/External){:target="_blank"}

If you open the link, you'll see that the site loads additional jobs every time you scroll to the bottom
of the page. In this article, I'll demonstrate how to handle sites like this with Puppeteer. We'll go over
how to trigger the scrolling and how to determine when we've reached the end of the results. 

## Dependencies

In order to run Puppeteer, you'll first need to install [Node](https://nodejs.org/en/){:target="_blank"} if it's not already on your system.
Since the code in this article uses async/await, you'll need Node v7.6.0 or higher. Then run the following commands to install Puppeteer:

```bash
$ npm init -y
$ npm install puppeteer --save
```
Now onto the code.

## Implementation

We'll put the code to run the scraper within a function named `main`:

```javascript
'use strict';

const puppeteer = require('puppeteer');

...

const main = async () => {
    const browser = await puppeteer.launch();    
    const [ page ] = await browser.pages();

    /* Log when the requests gets sent out */
    page.on('request', request => {
        if (request.resourceType() === "xhr" &&
            request.url().includes('External')) { 
            console.log('Request sent ' + request.url());
        }
    });

    /* Log when the responses comes back */
    page.on('response', response => {
        if (response.request().resourceType() === "xhr" &&
            response.request().url().includes('External')) {
            console.log('Response received');
        }
    });
    
    const jobs = await getJobLinks(page);

    await browser.close()
    return jobs;
};

main().then((jobs) => {
    console.log(JSON.stringify(jobs, null, 2));
});
```

Breaking this down, within `main`, we first launch Chrome and create a new page.

```javascript
    const browser = await puppeteer.launch();    
    const [ page ] = await browser.pages();
```

Then we use `page.on` to set up handlers for [request](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-request){:target="_blank"}/[response](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-response){:target="_blank"} events so that we can log the AJAX requests and
responses that get sent when the jobs are dynamically loaded. If you examine the requests you'll see that they're always sent to a
URL with 'External' in the path, so that is what we filter on:

```javascript
    /* Log when the requests gets sent out */
    page.on('request', request => {
        if (request.resourceType() === "xhr" &&
            request.url().includes('External')) { 
            console.log('Request sent ' + request.url());
        }
    });

    /* Log when the responses comes back */
    page.on('response', response => {
        if (response.request().resourceType() === "xhr" &&
            response.request().url().includes('External')) {
            console.log('Response received');
        }
    });
```

Finally, we call `getJobLinks` in order to load the full list of jobs (via scrolling) and then scrape
and return the jobs data.

```javascript
    const jobs = await getJobLinks(page);

    await browser.close()
    return jobs;
```

Now let's examine `getJobLinks`. First, we open the jobs site using `page.goto` and then wait for the initial page of jobs to load.

```javascript
const getJobLinks = async (page) => {
    await page.goto('https://finning.wd3.myworkdayjobs.com/External')
    await page.waitFor(
        id => document.querySelector(`span#${id}`),
        {}, /* opts */
        'wd-FacetedSearchResultList-PaginationText-facetSearchResultList\\.newFacetSearch\\.Report_Entry'
    );

    await scrollToEnd(page);
    
    let jobs = await extractJobs(page);
    return jobs;
};
```

The jobs get loaded into a span with ID 

```javascript
wd-FacetedSearchResultList-facetSearchResultList.newFacetSearch.Report_Entry
```

so we pass that ID as an argument to [waitFor](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pagewaitforselectororfunctionortimeout-options-args){:target_="_blank"}. Note that we escape the periods `.` in the ID in the call to `waitFor` as we don't want them interpreted as class selectors by `document.querySelector`.

Once the first page of jobs have loaded we call `scrollToEnd` to repeatedly scroll to the bottom of the
page until the rest of the jobs have been loaded. 

```javascript
const scrollToEnd = async (page) => {
    let totalNumJobs = await getTotalNumJobs(page);
    let numJobs = await page.evaluate(getNumJobsListed);
    let scrollHeight = await page.evaluate('document.documentElement.scrollHeight');

    while (numJobs < totalNumJobs) {
        console.log(`# jobs = ${numJobs}`);
        
        await page.evaluate('window.scrollTo(0, document.documentElement.scrollHeight)');
        await page.waitForFunction(`document.documentElement.scrollHeight > ${scrollHeight}`);
        
        scrollHeight = await page.evaluate('document.body.scrollHeight');
        numJobs = await page.evaluate(getNumJobsListed);
    }

    console.log(`# jobs = ${numJobs}`); 
};
```

The `scrollToEnd` function works by comparing the number of jobs currently listed with the number of
jobs the page says it contains in all. In the screenshot below we see the **112 Results** heading. 

![Total Number Jobs](/assets/infinite-scroll-with-puppeteer/total-num-jobs.png)

That corresponds to `totalNumJobs` which we extract using a regex:

```javascript
/* Return the total number of jobs in the system */
const getTotalNumJobs = async (page) => {
    let totalJobsStr = await page.evaluate(
        id => document.querySelector(`span#${id}`).innerText,
        'wd-FacetedSearchResultList-PaginationText-facetSearchResultList\\.newFacetSearch\\.Report_Entry'
    );

    return parseInt(
            /(\d+)\s+Results/.exec(totalJobsStr)[1]
    );
};
```

We retrieve the number of jobs loaded so far using `getNumJobsListed`:

```javascript
/* Return the number of jobs currently listed on the page */
const getNumJobsListed = () => {
    return Array.from(document.querySelectorAll('div[id^="promptOption-gwt-uid-"]')).length;
};
```

If the number jobs listed is still less than the total number of jobs the system says it
contains then we scroll to the bottom of the page to trigger another load:

```javascript
    ...
    let scrollHeight = await page.evaluate('document.documentElement.scrollHeight');

    while (numJobs < totalNumJobs) {
        console.log(`# jobs = ${numJobs}`);
        
        await page.evaluate('window.scrollTo(0, document.documentElement.scrollHeight)');
        await page.waitForFunction(`document.documentElement.scrollHeight > ${scrollHeight}`);
        
        scrollHeight = await page.evaluate('document.body.scrollHeight');
        numJobs = await page.evaluate(getNumJobsListed);
    }
```

If we go back to `getJobLinks` we see that once we've scrolled to the end and loaded all of the jobs
into the page, we then call `extractJobs` to scrape the job titles and their links.
