---
layout: post
title: Scraping by example
---

Scraping by example

Open the Chrome developer tools by selecting View > Developer > Developer Tools

![Developer Tools](/assets/scraping-ajax-pages-with-python/developer_tools.png)

![Form](/assets/scraping-by-example-ajax-pagination/form.png)
![Results](/assets/scraping-by-example-ajax-pagination/results.png)

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

{% highlight html %}
<form name="aspnetForm" method="post" action="frmSearch.aspx" onsubmit="javascript:return WebForm_OnSubmit();" id="aspnetForm">
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
24137|updatePanel|ctl00_ContentPlaceHolder1_pnlgrdSearchResult|
  <input type="hidden" name="ctl00$ContentPlaceHolder1$hdnTabShow" id="ctl00_ContentPlaceHolder1_hdnTabShow" value="0" />
    <div>
      <div style="font-weight: bold;">
        <span id="ctl00_ContentPlaceHolder1_lblRowCountMessage">1 - 20 of 73 Results</span></div>
          <input type="hidden" name="ctl00$ContentPlaceHolder1$hdnTotalRows" id="ctl00_ContentPlaceHolder1_hdnTotalRows" value="73" />
        </div>
        ...
|0|hiddenField|__EVENTTARGET||0|hiddenField|__EVENTARGUMENT||148128|hiddenField|__VIEWSTATE|/wEPDwUKMTU0OTkzNjExNg...|121|asyncPostBackControlIDs||ctl00$ContentPlaceHolder1$btnSearch,ctl00$ContentPlaceHolder1$btnfrmSearch,ctl00$ContentPlaceHolder1$tmrLoadSearchResults|0|postBackControlIDs|||45|updatePanelIDs||tctl00$ContentPlaceHolder1$pnlgrdSearchResult|0|childUpdatePanelIDs|||44|panelsToRefreshIDs||ctl00$ContentPlaceHolder1$pnlgrdSearchResult|3|asyncPostBackTimeout||600|14|formAction||frmSearch.aspx|
{% endhighlight %}

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


