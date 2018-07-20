---
layout: post
title: Scraping with Puppeteer
---

In this post I'll guide you through web scraping with [Puppeteer](https://github.com/GoogleChrome/puppeteer){:target="_blank"}, a Node library used to
control Chrome (or Chromium) using the [DevTools Protocol](https://chromedevtools.github.io/devtools-protocol/){:target="_blank"}. I'll also cover how
to use Node's built-in [debugger](https://nodejs.org/api/debugger.html){:target="_blank"} so that you can step through the code to see how everything
works.

## Dependencies

In order to run Puppeteer, you'll first need to install [Node](https://nodejs.org/en/) if it's not already on your system.
Since the code in this article uses async/await, you'll need Node v7.6.0 or higher.

## Setup

To get started, first create a new directory for the scraper. Inside the new directory, run `npm init` followed by
`npm install puppeteer --save`.

```bash
$ mkdir scraper/
$ cd scraper/
$ npm init -y
$ npm install puppeteer --save
```

When you install Puppeteer in the last step, it will download a version of Chromium for you that is guaranteed to work with the API.

## Background

The site I'll use for this article is the ASPX search form at:

[https://myaccount.rid.org/Public/Search/Member.aspx](https://myaccount.rid.org/Public/Search/Member.aspx){:target="_blank"}

Here's a screenshot of the form:

![Search Form](/assets/puppeteer/search_form.png)

If you try to submit the form by clicking on Find Member, without filling in any of the fields, you'll see the following warning:

![Warning](/assets/puppeteer/warning.png)

For this article we'll fill out the State and Freelance Status fields. Specifically, we'll iterate through all of the states in
the State field and specify `Yes` for the Freelance Status field. This will give us a chance to see how to handle select
dropdowns, pagination, and dynamically loaded data.

## Implementation

Let's get started. Open up your editor and enter the following code. Save it into a file named `rid_scraper.js`.

```javascript
const puppeteer = require('puppeteer');
const url = 'https://myaccount.rid.org/Public/Search/Member.aspx';

async function main() {
    const browser = await puppeteer.launch({ headless: false, slowMo: 250 });
    const page = await browser.newPage();

    await page.goto(url);
    console.log(await page.title());
    browser.close();    
}

main();
```

Next, let's step through it in the command line [debugger](https://nodejs.org/api/debugger.html){:target="_blank"} as that more
accurately reflects the development process you'll go through when writing a scraper:

```bash
$ node inspect rid_scraper.js 
< Debugger listening on ws://127.0.0.1:9229/71ad933d-21d8-46d6-a8b9-2bf2c145f77c
< For help see https://nodejs.org/en/docs/inspector
< Debugger attached.
Break on start in rid_scraper.js:1
> 1 (function (exports, require, module, __filename, __dirname) { const puppeteer = require('puppeteer');
  2 const url = 'https://myaccount.rid.org/Public/Search/Member.aspx';
  3
```

Once the debugger is running, use `list()` to see the code. The `>` symbol indicates the next line you
are about to execute.

```
debug> list()
> 1 (function (exports, require, module, __filename, __dirname) { const puppeteer = require('puppeteer');
  2 const url = 'https://myaccount.rid.org/Public/Search/Member.aspx';
  3 
  4 async function main() {
  5     const browser = await puppeteer.launch({ headless: false, slowMo: 250 });
  6     const page = await browser.newPage();
```

Type in `help` to see the available commands.

```bash
debug> help
run, restart, r       Run the application or reconnect
kill                  Kill a running application or disconnect

cont, c               Resume execution
next, n               Continue to next line in current file
step, s               Step into, potentially entering a function
out, o                Step out, leaving the current function
backtrace, bt         Print the current backtrace
list                  Print the source around the current line where execution
                      is currently paused

setBreakpoint, sb     Set a breakpoint
clearBreakpoint, cb   Clear a breakpoint
breakpoints           List all known breakpoints
breakOnException      Pause execution whenever an exception is thrown
...
```

Next, set a breakpoint on line 5 using the set breakpoint command `sb(5)`:

```bash
debug> sb(5)
  1 (function (exports, require, module, __filename, __dirname) { const puppeteer = require('puppeteer');
  2 const url = 'https://myaccount.rid.org/Public/Search/Member.aspx';
  3 
  4 async function main() {
> 5     const browser = await puppeteer.launch({ headless: false, slowMo: 250 });
  6     const page = await browser.newPage();
  7 
  8     await page.goto(url);
  9     console.log(await page.title());
 10     browser.close();
```

Type `list` again you'll notice that there's now a `*` symbol on line 5. The debugger is giving us a
visual indicator that there's a breakpoint on that line:

```bash
debug> list()
> 1 (function (exports, require, module, __filename, __dirname) { const puppeteer = require('puppeteer');
  2 const url = 'https://myaccount.rid.org/Public/Search/Member.aspx';
  3 
  4 async function main() {
* 5     const browser = await puppeteer.launch({ headless: false, slowMo: 250 });
```

Now type `c` to continue execution until the breakpoint.  Once you hit the breakpoint on line 5, type `n`
to execute one line at a time until you get to the `console.log()` statement on line 9:

```bash
debug> c
break in rid_scraper.js:5
  3 
  4 async function main() {
> 5     const browser = await puppeteer.launch({ headless: false, slowMo: 250 });
  6     const page = await browser.newPage();
  7 
debug> n
break in rid_scraper.js:6
  4 async function main() {
* 5     const browser = await puppeteer.launch({ headless: false, slowMo: 250 });
> 6     const page = await browser.newPage();
  7 
  8     await page.goto(url);
debug> n
break in rid_scraper.js:8
  6     const page = await browser.newPage();
  7 
> 8     await page.goto(url);
  9     console.log(await page.title());
 10     browser.close();    
debug> n
break in rid_scraper.js:9
  7 
  8     await page.goto(url);
> 9     console.log(await page.title());
 10     browser.close();    
 11 }
```

Type `n` again to execute the `console.log()` statement on line 9. You'll see the page title "Find an RID Member" appear  prefixed
with the `<` symbol:

```bash
debug> n
break in rid_scraper.js:9
  7 
  8     await page.goto(url);
> 9     console.log(await page.title());
 10     browser.close();    
 11 }
debug> n
< Find an RID Member
break in rid_scraper.js:10
  8     await page.goto(url);
  9     console.log(await page.title());
>10     browser.close();    
 11 }
 12
```

Finally, type in `c` to continue execution until the end of the program. Type in `<CTRL-D>` to exit the debugger:

```bash
debug> c
< Waiting for the debugger to disconnect...
debug> ^D
```

Now that you've seen the basics of how to use the debugger, let's start adding the core functionality to our scraper. First, let's
write the code to get the list of states from the State dropdown menu.

To do that we need to know what selector to use for locating the state dropdown. Open up Chrome developer tools and inspect the
state dropdown element to see its `id` value:

```html
<select id="FormContentPlaceHolder_Panel_stateDropDownList">
	<option selected="selected" value=""></option>
	<option value="d8fcccc5-cf2f-e511-80c5-00155d631510">Alabama</option>
	<option value="dafcccc5-cf2f-e511-80c5-00155d631510">Alaska</option>
	<option value="befcccc5-cf2f-e511-80c5-00155d631510">Alberta</option>
        ...
</select>
```

Now that we know the id, let's create a generic function that can return a list of all the options under a select element. We'll use Puppeteer's
[page.evaluate](https://github.com/GoogleChrome/puppeteer/blob/master/docs/api.md#pageevaluatepagefunction-args) function to locate the `<select>`
element that matches `selector` and return all of the available options for that element as an array of text,value pairs.

```javascript
async function getSelectOptions(page, selector) {
    const options = await page.evaluate(optionSelector => {
        return Array.from(document.querySelectorAll(optionSelector))
            .filter(o => o.value)
            .map(o => {
                return {
                    name: o.text,
                    value: o.value
                };
            });        
    }, selector);

    return options;
}
```

Now we can easily retrieve the state option values using:

```javascript
async function getStates(page) {
    return await getSelectOptions(page,
        'select#FormContentPlaceHolder_Panel_stateDropDownList > option'
    );
}
```

Now let's go back to `main()` and update it to iterate through the list of states. We'll also add
code to set the Freelance Status field to `Yes`. 

```javascript
async function main() {
    const browser = await puppeteer.launch({ headless: false, slowMo: 250 });
    const page = await browser.newPage();

    await page.goto(url);

    let states = await getStates(page);
    
    for (const [ i, state ] of states.entries()) {
        console.log(`[${i+1}/${states.length}] Scraping data for ${state.name}`);

        await page.select('#FormContentPlaceHolder_Panel_stateDropDownList', state.value);
        await page.select('#FormContentPlaceHolder_Panel_freelanceDropDownList', '1');

        await page.click('#FormContentPlaceHolder_Panel_searchButtonStrip_searchButton');
        await page.waitForSelector('#FormContentPlaceHolder_Panel_resultsGrid');

        break;
    }
    
    browser.close();
}

main();
```

Here we use `page.select` to select the State and Freelance status values, `page.click` to click the Find Members
button. After we submit the search by clicking the Find Member search button we wait until the results table appears
using `waitForSelector`.

Now run the script in the debugger, so you can step through the code a line at a time and watch as the State and
Freelance status fields get updated as the code runs. If you'd like to examine the values of the variables as you're
stepping through the code use the `repl` command:

```bash
debug> repl
Press Ctrl + C to leave debug repl
> i
0
> states.length
73
> states[0]
{ name: 'Alabama',
  value: 'd8fcccc5-cf2f-e511-80c5-00155d631510' }
> ^C
```


```javascript
async function scrapeMemberTable(page) {
    const data = await page.evaluate(() => {
        const ths = Array.from(document.querySelectorAll('table th'));
        const trs = Array.from(document.querySelectorAll('table tr.RowStyle'));
        const headers = ths.map(th => th.innerText);

        let results = [];

        console.log(`${trs.length} rows in member table!`);
        
        trs.forEach(tr => {
            let r = {};            
            let tds = Array.from(tr.querySelectorAll('td')).map(td => td.innerText);

            headers.forEach((k,i) => r[k] = tds[i]);
            results.push(r);
        });

        return results;
    });

    console.log(`Got ${data.length} records`);
    return data;
}
```

```javascript
async function scrapeAllPages(page) {
    let results = [];
    let pageno = 2;
    
    while (true) {
        console.log(`Scraping page ${pageno - 1}`);
        
        results = results.concat(
            await scrapeMemberTable(page)
        );

        const noMorePages = await gotoNextPage(page, pageno++)
        if (noMorePages) {
            break;
        }
    }

    /*
     * The pager won't reset back to page 1 on its own so we have to explicitly 
     * click on the page 1 link
     */
    await gotoFirstPage(page);
    return results;
}
```


![Before Page 2](/assets/puppeteer/before_page_2.png)
![After Page 2](/assets/puppeteer/after_page_2.png)

![Page 2 link](/assets/puppeteer/page2_link.png)
![Page 2 current](/assets/puppeteer/page2_current.png)

```javascript
/*------------------------------------------------------------------------------
 * Look for link for pageno in pager. So if pageno was 6 we'd look for 'Page$6' 
 * in href:
 *
 * <a href="javascript:__doPostBack('ctl00$FormContentPlaceHolder$Panel$resultsGrid','Page$6')">...</a>
 *
 * After the next page link gets clicked and the new page is loaded the pager
 * will show the current page within a span (not as a link). So we wait until 
 * pageno appears within a span to indicate that the next page has finished 
 * loading.
 */ 
async function gotoNextPage(page, pageno) {
    let noMorePages = true;
    let nextPageXp = `//tr[@class='PagerStyle']/td/table/tbody/tr/td/a[contains(@href,'Page$${pageno}')]`;
    let currPageXp = `//tr[@class='PagerStyle']/td/table/tbody/tr/td/span[text()='${pageno}']`;
    let nextPage;

    nextPage = await page.$x(nextPageXp)
    
    if (nextPage.length > 0) {
        console.log(`Going to page ${pageno}`);
        
        await nextPage[0].click();
        await page.waitForXPath(currPageXp);
        
        noMorePages = false;
    }

    return noMorePages;    
}
```

```javascript
/*
 * Wait until elem becomes detached from DOM
 */
async function waitUntilStale(page, elem) {
    await page.waitForFunction(
        e => !e.ownerDocument.contains(e),
        { polling: 'raf' }, elem
    );
}
```

```javascript
async function setMaxPageSize(page) {
    let html = await page.content();
    let pageSizeNameRe = new RegExp(
        'ctl00\\$FormContentPlaceHolder\\$Panel\\$resultsGrid\\$ctl\\d+\\$ctl\\d+'
    );

    let match = pageSizeNameRe.exec(html);
    if (match.length <= 0) {
        return;
    } 

    let pageSizeName = match[0];
    let resultsTable = await page.$('#FormContentPlaceHolder_Panel_resultsGrid');    

    await page.select(`select[name="${pageSizeName}"]`, '50');

    /*
     * Selecting the page size triggers an ajax request for the new table results. 
     * We need to wait until that new table data gets loaded before trying to scrape.
     * So we wait until the old member table gets detached from the DOM as the signal
     * that the new table has been loaded
     */
    await waitUntilStale(page, resultsTable);
}
```
