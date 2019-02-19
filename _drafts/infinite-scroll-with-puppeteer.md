---
layout: post
title: Scraping by Example - Infinite Scroll with Puppeteer
---

In this post I'll demonstrate how to scrape a site that uses infinite scroll. The site I'll use
for demonstration is the job site at:

[https://finning.wd3.myworkdayjobs.com/External](https://finning.wd3.myworkdayjobs.com/External){:target="_blank"}

If you open the link, you'll see that the site loads additional jobs every time you scroll to the bottom
of the page.

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
    //const browser = await puppeteer.launch({ slowMo: 250, headless: false, devtools: true });
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

Within `main`, we first we launch Chrome and create a new page. Then we use `page.on`
to set up handlers for [request](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-request){:target="_blank"}/[response](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#class-response){:target="_blank"} events so that we can log the AJAX requests and
responses that get sent when the jobs are dynamically loaded. Finally, we call
`getJobLinks` in order to load the full list of jobs (via scrolling) and then scrape
the jobs data.

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

```javascript

```

