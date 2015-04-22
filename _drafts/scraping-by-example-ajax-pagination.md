---
layout: post
title: Scraping ASP.NET Pages with AJAX Pagination
---

## Background

### Analyzing the Form Submission 

![Form](/assets/scraping-by-example-ajax-pagination/form.png)

Open the Chrome developer tools by selecting View > Developer > Developer Tools

![Developer Tools](/assets/scraping-ajax-pages-with-python/developer_tools.png)

Select 'Alaska' from the state dropdown list.

![Form State](/assets/scraping-by-example-ajax-pagination/form_state.png)

Click the Search button. After a moment our two, you'll see the area below the search
form populated with the results.

![Results](/assets/scraping-by-example-ajax-pagination/results.png)

In the developer tools, click on the Network tab. You'll see a list of requests that were
sent when the form was submitted.

![Network Tab Requests](/assets/scraping-by-example-ajax-pagination/network_tab_requests.png)

Click on the POST request to frmSearch.aspx. With the Headers tab selected, scroll down to see 
the Form Data that was sent with the request.

![POST_headers](/assets/scraping-by-example-ajax-pagination/POST_headers.png)

These are the variables our scraper needs to send when it submits the form. The listing below
shows the full list of variables with the __VIEWSTATE value truncated to make the whole thing
easier to read.

{% highlight text %}
ctl00$ScriptManager1:ctl00$ScriptManager1|ctl00$ContentPlaceHolder1$btnSearch
__EVENTTARGET:
__EVENTARGUMENT:
__VIEWSTATE:/wEPDwUKMTU0OTkzNjEx...
ctl00$ContentPlaceHolder1$search:rdbCityState
ctl00$ContentPlaceHolder1$txtCity:
ctl00$ContentPlaceHolder1$drpState:AK
ctl00$ContentPlaceHolder1$txtZip:
ctl00$ContentPlaceHolder1$drpRadius:1
ctl00$ContentPlaceHolder1$drpBuilingType:
ctl00$ContentPlaceHolder1$drpCountry:
ctl00$ContentPlaceHolder1$dpdCandaStates:
ctl00$ContentPlaceHolder1$txtFirmname:
ctl00$ContentPlaceHolder1$hdnTabShow:0
ctl00$ContentPlaceHolder1$hdnTotalRows:
__ASYNCPOST:true
ctl00$ContentPlaceHolder1$btnSearch:Search
{% endhighlight %}

Now let's take a look at the response. Click on the Response tab in the developer tools.

![POST1_response](/assets/scraping-by-example-ajax-pagination/POST1_response.png)

The response is a pipe-delimited string. We are interested in two of the values from this string.
The first is the HTML right after the ctl00_ContentPlaceHolder1_pnlgrdSearchResult variable. That
HTML string contains the search results for the form submission. 

The other value is the string after the __VIEWSTATE variable. We'll need to extract the __VIEWSTATE 
in order to do pagination. We'll cover that in the next section. For now, let's look at the HTML
results.

{% highlight text %}
24137  | updatePanel | ctl00_ContentPlaceHolder1_pnlgrdSearchResult | <input type="hidden" name="ctl00$ContentPlaceHolder1$hdnTabShow" id="ctl00_ContentPlaceHolder1_hdnTabShow" value="0" /><div><div style="font-weight: bold;"><span id="ctl00_ContentPlaceHolder1_lblRowCountMessage">1 - 20 of 73 Results</span></div><input type="hidden" name="ctl00$ContentPlaceHolder1$hdnTotalRows" id="ctl00_ContentPlaceHolder1_hdnTotalRows" value="73" /></div>...|
0      | hiddenField | __EVENTTARGET ||
0      | hiddenField | __EVENTARGUMENT ||
148128 | hiddenField | __VIEWSTATE | /wEPDwUKMTU0OTkzNjExNg... |
121    | asyncPostBackControlIDs || ctl00$ContentPlaceHolder1$btnSearch,ctl00$ContentPlaceHolder1$btnfrmSearch,ctl00$ContentPlaceHolder1$tmrLoadSearchResults|
0      | postBackControlIDs |||
45     | updatePanelIDs || tctl00$ContentPlaceHolder1$pnlgrdSearchResult |
0      | childUpdatePanelIDs |||
44     | panelsToRefreshIDs || ctl00$ContentPlaceHolder1$pnlgrdSearchResult |
3      | asyncPostBackTimeout || 600 |
14     | formAction || frmSearch.aspx |
{% endhighlight %}

{% highlight html %}
<input type="hidden" name="ctl00$ContentPlaceHolder1$hdnTabShow" id="ctl00_ContentPlaceHolder1_hdnTabShow" value="0" />
<div>
  <div style="font-weight: bold;">
    <span id="ctl00_ContentPlaceHolder1_lblRowCountMessage">1 - 20 of 73 Results</span>
  </div>
  <input type="hidden" name="ctl00$ContentPlaceHolder1$hdnTotalRows" id="ctl00_ContentPlaceHolder1_hdnTotalRows" value="73" />
</div>
<div>
  <table class="table_grid_b" cellspacing="0" rules="all" border="1" id="ctl00_ContentPlaceHolder1_grdSearchResult" style="border-collapse:collapse;">
    <tr style="color:#555555;">
      <th align="left" scope="col" style="width:30%;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl01_lnkFirmName" title="Sort" href="javascript:__doPostBack('ctl00$ContentPlaceHolder1$grdSearchResult$ctl01$lnkFirmName','')" style="text-decoration:none;">Firm Name</a>
        <img src="Images/up_down.gif" style="border-width:0px;" />
      </th>
      <th align="left" scope="col" style="width:35%;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl01_lnkCity" title="Sort" href="javascript:__doPostBack('ctl00$ContentPlaceHolder1$grdSearchResult$ctl01$lnkCity','')" style="text-decoration:none;">Location</a>
        <img src="Images/up_down.gif" style="border-width:0px;" />
      </th>
      <th align="left" scope="col" style="width:15%;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl01_lnkProjects" title="Sort" href="javascript:__doPostBack('ctl00$ContentPlaceHolder1$grdSearchResult$ctl01$lnkProjects','')" style="text-decoration:none;">Projects</a>
        <img src="Images/down_arrow.png" style="border-width:0px;" />
      </th>
    </tr>
    <tr class="Row1">
      <td class="gridcolumn" align="left" valign="top" style="width:205px;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl02_hpFirmName" href="frmFirmDetails.aspx?FirmID=F2B34EE8-96BF-4C5E-816B-732F68F06CA5">Bezek Durst Seiser Inc</a>
      </td>
      <td align="left" style="width:150px;">
        <span id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl02_lblCity">Anchorage, AK  99503-3957</span>
      </td>
      <td align="left">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl02_hpShowProjects" href="frmFirmDetails.aspx?FirmID=F2B34EE8-96BF-4C5E-816B-732F68F06CA5">View projects</a>
      </td>
    </tr>
    <tr class="Row2">
      <td class="gridcolumn" align="left" valign="top" style="width:205px;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl03_hpFirmName" href="frmFirmDetails.aspx?FirmID=F12ED5B3-88A1-49EC-96BC-ACFAA90C68F1">Kumin Associates, Inc.</a>
      </td>
      <td align="left" style="width:150px;">
        <span id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl03_lblCity">Anchorage, AK  99501-3578</span>
      </td>
      <td align="left">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl03_hpShowProjects" href="frmFirmDetails.aspx?FirmID=F12ED5B3-88A1-49EC-96BC-ACFAA90C68F1">View projects</a>
      </td>
    </tr>
    ...
    <tr class="footer_grid" align="right">
      <td colspan="3">
        <table>
          <tr>
            <td>
              <a disabled="disabled" class="dis_class" style="display:inline-block;width:50px;">&lt;&lt; first</a>
              <a disabled="disabled" class="dis_class" style="display:inline-block;width:50px;">&lt; prev</a>
              <a class="LinkPaging" href="javascript:__doPostBack('ctl00$ContentPlaceHolder1$grdSearchResult$ctl23$ctl02','')" style="display:inline-block;background-color:#E2E2E2;width:20px;">1</a>
              <a class="LinkPaging" href="javascript:__doPostBack('ctl00$ContentPlaceHolder1$grdSearchResult$ctl23$ctl03','')" style="display:inline-block;width:20px;">2</a>
              <a class="LinkPaging" href="javascript:__doPostBack('ctl00$ContentPlaceHolder1$grdSearchResult$ctl23$ctl04','')" style="display:inline-block;width:20px;">3</a>
              <a class="LinkPaging" href="javascript:__doPostBack('ctl00$ContentPlaceHolder1$grdSearchResult$ctl23$ctl05','')" style="display:inline-block;width:20px;">4</a>
              <a href="javascript:__doPostBack('ctl00$ContentPlaceHolder1$grdSearchResult$ctl23$ctl06','')" style="display:inline-block;width:50px;">next &gt;</a>
              <a href="javascript:__doPostBack('ctl00$ContentPlaceHolder1$grdSearchResult$ctl23$ctl07','')" style="display:inline-block;width:50px;">last &gt;&gt;</a>
            </td>
          </tr>
        </table>
      </td>
    </tr>
  </table>
</div>
{% endhighlight %}


### Analyzing Pagination

![Pagination](/assets/scraping-by-example-ajax-pagination/pagination.png)
![Inspect Page Number Link](/assets/scraping-by-example-ajax-pagination/inspect_page_number_link.png)

{% highlight html %}
<a class="LinkPaging" href="javascript:__doPostBack('ctl00$ContentPlaceHolder1$grdSearchResult$ctl23$ctl03','')">2</a>
{% endhighlight %}

{% highlight javascript %}
var theForm = document.forms['aspnetForm'];
if (!theForm) {
    theForm = document.aspnetForm;
}
function __doPostBack(eventTarget, eventArgument) {
    if (!theForm.onsubmit || (theForm.onsubmit() != false)) {
        theForm.__EVENTTARGET.value = eventTarget;
        theForm.__EVENTARGUMENT.value = eventArgument;
        theForm.submit();
    }
}
{% endhighlight %}

{% highlight text %}
ctl00$ScriptManager1:ctl00$ContentPlaceHolder1$pnlgrdSearchResult|ctl00$ContentPlaceHolder1$grdSearchResult$ctl23$ctl03
ctl00$ContentPlaceHolder1$search:rdbCityState
ctl00$ContentPlaceHolder1$txtCity:
ctl00$ContentPlaceHolder1$drpState:AK
ctl00$ContentPlaceHolder1$txtZip:
ctl00$ContentPlaceHolder1$drpRadius:1
ctl00$ContentPlaceHolder1$drpBuilingType:
ctl00$ContentPlaceHolder1$drpCountry:
ctl00$ContentPlaceHolder1$dpdCandaStates:
ctl00$ContentPlaceHolder1$txtFirmname:
ctl00$ContentPlaceHolder1$hdnTabShow:0
ctl00$ContentPlaceHolder1$hdnTotalRows:73
__EVENTTARGET:ctl00$ContentPlaceHolder1$grdSearchResult$ctl23$ctl03
__EVENTARGUMENT:
__VIEWSTATE:/wEPDwUKMTU0OTkzNj...
__ASYNCPOST:true
:
{% endhighlight %}

## Implementation

### Submitting the Form in a Script

{% highlight html %}
<form name="aspnetForm" method="post" action="frmSearch.aspx" onsubmit="javascript:return WebForm_OnSubmit();" id="aspnetForm">
{% endhighlight %}

{% highlight html %}
<input type="submit" name="ctl00$ContentPlaceHolder1$btnSearch" value="Search" onclick="javascript:WebForm_DoPostBackWithOptions(new WebForm_PostBackOptions('ctl00$ContentPlaceHolder1$btnSearch', '', true, 'vgLocationSearch', '', false, false))" id="ctl00_ContentPlaceHolder1_btnSearch">
{% endhighlight %}

{% highlight python %}
(Pdb) print '\n'.join(['%s:%s' % (c.name,c.value) for c in self.br.form.controls])
{% endhighlight %}

{% highlight text %}
__EVENTTARGET:
__EVENTARGUMENT:
__VIEWSTATE:/wEPDwUKMTU0OTkzNjExN...
ctl00$ContentPlaceHolder1$btnAccept:Ok
None:None
ctl00$ContentPlaceHolder1$search:['rdbCityState']
ctl00$ContentPlaceHolder1$txtCity:
ctl00$ContentPlaceHolder1$drpState:['AK']
ctl00$ContentPlaceHolder1$txtZip:
ctl00$ContentPlaceHolder1$drpRadius:['1']
ctl00$ContentPlaceHolder1$drpBuilingType:['']
ctl00$ContentPlaceHolder1$drpCountry:['']
ctl00$ContentPlaceHolder1$dpdCandaStates:['']
ctl00$ContentPlaceHolder1$btnSearch:Search
ctl00$ContentPlaceHolder1$txtFirmname:
ctl00$ContentPlaceHolder1$btnfrmSearch:Search
ctl00$ContentPlaceHolder1$hdnTabShow:0
ctl00$ContentPlaceHolder1$hdnTotalRows:
None:None
{% endhighlight %}

### Pagination in the Script

{% highlight python %}
def scrape_state_firms(self, state_item):
        self.br.open(self.url)
        
        s = soupify(self.br.response().read())
        saved_form = s.find('form', id='aspnetForm').prettify()

        self.br.select_form('aspnetForm')

        self.br.form.new_control('hidden', '__EVENTTARGET',   {'value': ''})
        self.br.form.new_control('hidden', '__EVENTARGUMENT', {'value': ''})
        self.br.form.new_control('hidden', '__ASYNCPOST',     {'value': 'true'})
        self.br.form.new_control('hidden', 'ctl00$ScriptManager1', {'value': 'ctl00$ScriptManager1|ctl00$ContentPlaceHolder1$btnSearch'})
        self.br.form.fixup()
        self.br.form['ctl00$ContentPlaceHolder1$drpState'] = [ state_item.name ]

        ctl = self.br.form.find_control('ctl00$ContentPlaceHolder1$btnfrmSearch')
        self.br.form.controls.remove(ctl)

        ctl = self.br.form.find_control('ctl00$ContentPlaceHolder1$btnAccept')
        self.br.form.controls.remove(ctl)

        ctl = self.br.form.find_control('ctl00$ContentPlaceHolder1$btnSearch')
        ctl.disabled = False

        self.br.submit()
{% endhighlight %}

{% highlight python %}
pageno = 2

        while True:
            r = self.br.response()
            s = BeautifulSoup(r.read())
            r = re.compile(r'^frmFirmDetails\.aspx\?FirmID=([A-Z0-9-]+)$')

            for a in s.findAll('a', href=r):
                m = re.search(r, a['href'])
                g = m.group(1)

                if ArchitectureFirm.objects.filter(frmid=g).exists():
                    continue

                firm = ArchitectureFirm()
                firm.name = a.text
                firm.frmid = m.group(1)
                firm.save()

                print a


            a = s.find('a', text='%d' % pageno)
            if not a:
                break

            pageno += 1
{% endhighlight %}

{% highlight python %}
r = re.compile(r'VIEWSTATE\|([^|]+)')
            m = re.search(r, str(s))
            view_state = m.group(1)

            r = re.compile(r"__doPostBack\('([^']+)")
            m = re.search(r, a['href'])

            html = saved_form.encode('utf8')
            resp = mechanize.make_response(html, [("Content-Type", "text/html")],
                                           self.br.geturl(), 200, "OK")
            self.br.set_response(resp)
            self.br.select_form('aspnetForm')

            self.br.form.set_all_readonly(False)
            self.br.form['__EVENTTARGET'] = m.group(1)
            self.br.form['__VIEWSTATE'] = view_state
            self.br.form['ctl00$ContentPlaceHolder1$drpState'] = [ state_item.name ]
            self.br.form.new_control('hidden', '__ASYNCPOST',     {'value': 'true'})
            self.br.form.new_control('hidden', 'ctl00$ScriptManager1', {'value': 'ctl00$ContentPlaceHolder1$pnlgrdSearchResult|'+m.group(1)})
            self.br.form.fixup()

            ctl = self.br.form.find_control('ctl00$ContentPlaceHolder1$btnfrmSearch')
            self.br.form.controls.remove(ctl)

            ctl = self.br.form.find_control('ctl00$ContentPlaceHolder1$btnAccept')
            self.br.form.controls.remove(ctl)

            ctl = self.br.form.find_control('ctl00$ContentPlaceHolder1$btnSearch')
            self.br.form.controls.remove(ctl)

            self.br.submit()
{% endhighlight %}


