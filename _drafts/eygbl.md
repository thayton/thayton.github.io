---
layout: post
title: Scraping by Example - SelectMinds ATS
---

In this post I'll show how to develop a scraper for the SelectMinds Applicant Tracking System (ATS). For my example,
I'll use the jobs site at [https://eygbl.referrals.selectminds.com/](https://eygbl.referrals.selectminds.com/){:target="_blank"}

Before developing the code for the scraper, let's inspect the site and see how the jobs are loaded into the page. Open the Chrome
developer tools and switch to the Network tab. Then click the Search button to load all jobs. In the Network tab you'll see a series
of XHR requests being made which we'll need to recreate in order to scrape jobs from the site. The first request is a POST request to
`https://eygbl.referrals.selectminds.com/ajax/jobs/search/create` with the following query parameters and form data:

![first request](/assets/eygbl/1.png)

This request is sent by the code in https://eygbl.referrals.selectminds.com/job_search_banner.js which gets triggered
when the Search button is clicked:

```javascript
// Main submit binding
j$('#jSearchSubmit', search_banner).click(function() {
  ....
  j$.ajax({
    type: 'POST',
    url: TVAPP.guid('/ajax/jobs/search/create'),
    data: data,
    success: function(result){
      job_search_id = result.Result['JobSearch.id'];
      j$.log('job_search_id: ' + job_search_id);
      // Load results
      j$(document).trigger('loadSearchResults', {'job_search_id':job_search_id});
    },
    dataType: 'json',
    error: function(xhr, textStatus, error) {
      TVAPP.masterErrorHandler(xhr, textStatus, error, null);
    }
  });
});
```

The response sent back is a JSON string like the following:

```json
{"Status":"OK","UserMessage":"","Result":{"JobSearch.id":84067040}}
```

As shown in the code above, the `JobSearch.id` value from the resonse is passed as the argument to `loadSearchResults`. `loadSearchResults`
is defined in https://eygbl.referrals.selectminds.com/job_list.js:

```javascript
// function to find existing or get new search results
function loadSearchResults(data) {
  ....
  
  // load new results and display them
  if (cached_results.length && !data.force_refresh) {
    ....  
  } else {
    // Load new results
    var context = 'JobSearch.id=' + data.job_search_id;
    if (TVAPP.property.fb) context += '&fb=true';
    if (campaign_id) context += '&Campaign.id=' + campaign_id + '&campaign_page=2';
    if (data.page) context += '&page_index=' + data.page;

    // Sync load is bad news bears. instead, let's async and callback:
    j$.ajax({
      type: 'POST',
      url: TVAPP.guid('/ajax/content/job_results?' + context + '&site-name=' + TVAPP.property.site.short_name + '&include_site=true'),
      dataType: 'json',
      success: function(response) {
        var new_results = j$(response.Result);
        new_results.prependTo('#jResultsArea').hide();
        // keep cache to a max of 5 recent searches
        if (j$('.jResultsContent', '#jResultsArea').length > 5) {
          j$('.jResultsContent:last', '#jResultsArea').remove();
        }
        // initialize the new results
        initFilters(new_results);
        initPagination(new_results);

        // display the new results
        displayLoadedResults(new_results, data);
      },
      error: function(xhr, textStatus,error) {
        TVAPP.masterErrorHandler(xhr, textStatus, error, null);
      }
    });
  }
};
```


