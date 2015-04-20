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
        <span id="ctl00_ContentPlaceHolder1_lblRowCountMessage">1 - 20 of 73 Results</span>
      </div>
      <input type="hidden" name="ctl00$ContentPlaceHolder1$hdnTotalRows" id="ctl00_ContentPlaceHolder1_hdnTotalRows" value="73" />
    </div>
    ...
|0|hiddenField|__EVENTTARGET||0|hiddenField|__EVENTARGUMENT||148128|hiddenField|__VIEWSTATE|/wEPDwUKMTU0OTkzNjExNg...|121|asyncPostBackControlIDs||ctl00$ContentPlaceHolder1$btnSearch,ctl00$ContentPlaceHolder1$btnfrmSearch,ctl00$ContentPlaceHolder1$tmrLoadSearchResults|0|postBackControlIDs|||45|updatePanelIDs||tctl00$ContentPlaceHolder1$pnlgrdSearchResult|0|childUpdatePanelIDs|||44|panelsToRefreshIDs||ctl00$ContentPlaceHolder1$pnlgrdSearchResult|3|asyncPostBackTimeout||600|14|formAction||frmSearch.aspx|
{% endhighlight %}

{% highlight python %}

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
    <tr class="Row1">
      <td class="gridcolumn" align="left" valign="top" style="width:205px;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl04_hpFirmName" href="frmFirmDetails.aspx?FirmID=4D49059B-F5FC-4BFC-AA46-3442D53928D1">61 North Architects</a>
      </td>
      <td align="left" style="width:150px;">
        <span id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl04_lblCity">Anchorage, AK  99503-3738</span>
      </td>
      <td align="left">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl04_hpShowProjects" href="frmFirmDetails.aspx?FirmID=4D49059B-F5FC-4BFC-AA46-3442D53928D1"></a>
      </td>
    </tr>
    <tr class="Row2">
      <td class="gridcolumn" align="left" valign="top" style="width:205px;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl05_hpFirmName" href="frmFirmDetails.aspx?FirmID=D847C6A5-588C-43E6-8517-F7ADEE7166E6">AK/J Architecture, Inc.</a>
      </td>
      <td align="left" style="width:150px;">
        <span id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl05_lblCity">Fairbanks, AK  99712-3236</span>
      </td>
      <td align="left">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl05_hpShowProjects" href="frmFirmDetails.aspx?FirmID=D847C6A5-588C-43E6-8517-F7ADEE7166E6"></a>
      </td>
    </tr>
    <tr class="Row1">
      <td class="gridcolumn" align="left" valign="top" style="width:205px;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl06_hpFirmName" href="frmFirmDetails.aspx?FirmID=14E0D4F3-B1E3-4EAA-B93C-757039B640AE">Architects Alaska</a>
      </td>
      <td align="left" style="width:150px;">
        <span id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl06_lblCity">Anchorage, AK  99501-2048</span>
      </td>
      <td align="left">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl06_hpShowProjects" href="frmFirmDetails.aspx?FirmID=14E0D4F3-B1E3-4EAA-B93C-757039B640AE"></a>
      </td>
    </tr>
    <tr class="Row2">
      <td class="gridcolumn" align="left" valign="top" style="width:205px;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl07_hpFirmName" href="frmFirmDetails.aspx?FirmID=0B0906FD-5780-4581-B7C9-5D7E596C13A6">Architects Alaska</a>
      </td>
      <td align="left" style="width:150px;">
        <span id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl07_lblCity">Anchorage, AK  99501-2045</span>
      </td>
      <td align="left">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl07_hpShowProjects" href="frmFirmDetails.aspx?FirmID=0B0906FD-5780-4581-B7C9-5D7E596C13A6"></a>
      </td>
    </tr>
    <tr class="Row1">
      <td class="gridcolumn" align="left" valign="top" style="width:205px;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl08_hpFirmName" href="frmFirmDetails.aspx?FirmID=04E714C2-EA46-483A-8D96-18439559C3CD">Barnes Architecture Inc.</a>
      </td>
      <td align="left" style="width:150px;">
        <span id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl08_lblCity">Anchorage, AK  99501-2508</span>
      </td>
      <td align="left">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl08_hpShowProjects" href="frmFirmDetails.aspx?FirmID=04E714C2-EA46-483A-8D96-18439559C3CD"></a>
      </td>
    </tr>
    <tr class="Row2">
      <td class="gridcolumn" align="left" valign="top" style="width:205px;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl09_hpFirmName" href="frmFirmDetails.aspx?FirmID=CF2D0F2F-D2BB-4930-8A40-58536543CE2C">Bettisworth North Architects and Planners Inc.</a>
      </td>
      <td align="left" style="width:150px;">
        <span id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl09_lblCity">Fairbanks, AK  99707-3209</span>
      </td>
      <td align="left">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl09_hpShowProjects" href="frmFirmDetails.aspx?FirmID=CF2D0F2F-D2BB-4930-8A40-58536543CE2C"></a>
      </td>
    </tr>
    <tr class="Row1">
      <td class="gridcolumn" align="left" valign="top" style="width:205px;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl10_hpFirmName" href="frmFirmDetails.aspx?FirmID=8D1326A5-953E-4FC6-9B60-D6B8D7687F89">Bettisworth North Architects and Planners Inc.</a>
      </td>
      <td align="left" style="width:150px;">
        <span id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl10_lblCity">Anchorage, AK  99503-2754</span>
      </td>
      <td align="left">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl10_hpShowProjects" href="frmFirmDetails.aspx?FirmID=8D1326A5-953E-4FC6-9B60-D6B8D7687F89"></a>
      </td>
    </tr>
    <tr class="Row2">
      <td class="gridcolumn" align="left" valign="top" style="width:205px;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl11_hpFirmName" href="frmFirmDetails.aspx?FirmID=5E65419A-D61C-498A-8E37-4100CD657DC6">Black & White Studio Architects</a>
      </td>
      <td align="left" style="width:150px;">
        <span id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl11_lblCity">Anchorage, AK  99503-3738</span>
      </td>
      <td align="left">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl11_hpShowProjects" href="frmFirmDetails.aspx?FirmID=5E65419A-D61C-498A-8E37-4100CD657DC6"></a>
      </td>
    </tr>
    <tr class="Row1">
      <td class="gridcolumn" align="left" valign="top" style="width:205px;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl12_hpFirmName" href="frmFirmDetails.aspx?FirmID=0136A5D0-8A2D-41A2-A396-7F3796DAC12F">Blue Sky Studio</a>
      </td>
      <td align="left" style="width:150px;">
        <span id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl12_lblCity">Anchorage, AK  99502-3973</span>
      </td>
      <td align="left">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl12_hpShowProjects" href="frmFirmDetails.aspx?FirmID=0136A5D0-8A2D-41A2-A396-7F3796DAC12F"></a>
      </td>
    </tr>
    <tr class="Row2">
      <td class="gridcolumn" align="left" valign="top" style="width:205px;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl13_hpFirmName" href="frmFirmDetails.aspx?FirmID=04EC3B2B-9D51-4EEC-B633-0114DDF6ACB3">Bruce Schulte</a>
      </td>
      <td align="left" style="width:150px;">
        <span id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl13_lblCity">Anchorage, AK  99509-3719</span>
      </td>
      <td align="left">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl13_hpShowProjects" href="frmFirmDetails.aspx?FirmID=04EC3B2B-9D51-4EEC-B633-0114DDF6ACB3"></a>
      </td>
    </tr>
    <tr class="Row1">
      <td class="gridcolumn" align="left" valign="top" style="width:205px;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl14_hpFirmName" href="frmFirmDetails.aspx?FirmID=374B9A3F-2B2D-491A-8A8C-C7E5BA175651">Burkhart Croft Architects</a>
      </td>
      <td align="left" style="width:150px;">
        <span id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl14_lblCity">Anchorage, AK  99501-3276</span>
      </td>
      <td align="left">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl14_hpShowProjects" href="frmFirmDetails.aspx?FirmID=374B9A3F-2B2D-491A-8A8C-C7E5BA175651"></a>
      </td>
    </tr>
    <tr class="Row2">
      <td class="gridcolumn" align="left" valign="top" style="width:205px;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl15_hpFirmName" href="frmFirmDetails.aspx?FirmID=D66769F7-5656-4989-9831-93690525F2AC">Burkhart Croft Architects, LLC</a>
      </td>
      <td align="left" style="width:150px;">
        <span id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl15_lblCity">Anchorage, AK  99501-3276</span>
      </td>
      <td align="left">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl15_hpShowProjects" href="frmFirmDetails.aspx?FirmID=D66769F7-5656-4989-9831-93690525F2AC"></a>
      </td>
    </tr>
    <tr class="Row1">
      <td class="gridcolumn" align="left" valign="top" style="width:205px;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl16_hpFirmName" href="frmFirmDetails.aspx?FirmID=A44BA93C-7781-4E0E-90A2-ABB8CD30098C">Burkhart Croft Architets, LLC</a>
      </td>
      <td align="left" style="width:150px;">
        <span id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl16_lblCity">Anchorage, AK  99501-2326</span>
      </td>
      <td align="left">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl16_hpShowProjects" href="frmFirmDetails.aspx?FirmID=A44BA93C-7781-4E0E-90A2-ABB8CD30098C"></a>
      </td>
    </tr>
    <tr class="Row2">
      <td class="gridcolumn" align="left" valign="top" style="width:205px;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl17_hpFirmName" href="frmFirmDetails.aspx?FirmID=8B4DE74A-67B8-4E3E-A0F7-8A07ACBB79D1">Carl Edwin Grundberg, Architect</a>
      </td>
      <td align="left" style="width:150px;">
        <span id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl17_lblCity">Anchorage, AK  99504-3947</span>
      </td>
      <td align="left">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl17_hpShowProjects" href="frmFirmDetails.aspx?FirmID=8B4DE74A-67B8-4E3E-A0F7-8A07ACBB79D1"></a>
      </td>
    </tr>
    <tr class="Row1">
      <td class="gridcolumn" align="left" valign="top" style="width:205px;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl18_hpFirmName" href="frmFirmDetails.aspx?FirmID=79105880-28EF-4DDE-89B7-705267038B8B">Chris Chiei Architect LLC</a>
      </td>
      <td align="left" style="width:150px;">
        <span id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl18_lblCity">Anchorage, AK  99501-2148</span>
      </td>
      <td align="left">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl18_hpShowProjects" href="frmFirmDetails.aspx?FirmID=79105880-28EF-4DDE-89B7-705267038B8B"></a>
      </td>
    </tr>
    <tr class="Row2">
      <td class="gridcolumn" align="left" valign="top" style="width:205px;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl19_hpFirmName" href="frmFirmDetails.aspx?FirmID=62892542-6457-462D-8FCD-D8AB103B8F11">Combs & Combs, AIA, Architecture, Interiors & Art</a>
      </td>
      <td align="left" style="width:150px;">
        <span id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl19_lblCity">Anchorage, AK  99507-6207</span>
      </td>
      <td align="left">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl19_hpShowProjects" href="frmFirmDetails.aspx?FirmID=62892542-6457-462D-8FCD-D8AB103B8F11"></a>
      </td>
    </tr>
    <tr class="Row1">
      <td class="gridcolumn" align="left" valign="top" style="width:205px;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl20_hpFirmName" href="frmFirmDetails.aspx?FirmID=5ED2966B-04FD-4D49-933C-742524FAB91D">Dalton R. Jansen, AIA</a>
      </td>
      <td align="left" style="width:150px;">
        <span id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl20_lblCity">Girdwood, AK  99587-0527</span>
      </td>
      <td align="left">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl20_hpShowProjects" href="frmFirmDetails.aspx?FirmID=5ED2966B-04FD-4D49-933C-742524FAB91D"></a>
      </td>
    </tr>
    <tr class="Row2">
      <td class="gridcolumn" align="left" valign="top" style="width:205px;">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl21_hpFirmName" href="frmFirmDetails.aspx?FirmID=6B454F69-9C31-440D-8068-16BA0A6ACAA3">David A. Whitmore Architect</a>
      </td>
      <td align="left" style="width:150px;">
        <span id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl21_lblCity">Fairbanks, AK  99701-4732</span>
      </td>
      <td align="left">
        <a id="ctl00_ContentPlaceHolder1_grdSearchResult_ctl21_hpShowProjects" href="frmFirmDetails.aspx?FirmID=6B454F69-9C31-440D-8068-16BA0A6ACAA3"></a>
      </td>
    </tr>
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

![Pagination](/assets/scraping-by-example-ajax-pagination/pagination.png)
![Inspect Page Number Link](/assets/scraping-by-example-ajax-pagination/inspect_page_number_link.png)

{% highlight html %}
<a class="LinkPaging" href="javascript:__doPostBack('ctl00$ContentPlaceHolder1$grdSearchResult$ctl23$ctl03','')">2</a>
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


