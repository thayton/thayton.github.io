---
layout: post
title: Reader Submission - ramajudicial
---

[![1](/assets/ramajudicial/1.png)](/assets/ramajudicial/1.png)
[![2](/assets/ramajudicial/2.png)](/assets/ramajudicial/2.png)
[![3](/assets/ramajudicial/3.png)](/assets/ramajudicial/3.png)

{% highlight html %}
<script type="text/javascript">
//<![CDATA[
Sys.WebForms.PageRequestManager._initialize(
'managerScript', 'form1', 
['tupPanelCiudad',
 'upPanelCiudad',
 'fupCaptcha',
 'upCaptcha',
 'fupdLista',
 'updLista',
 'fupPanelActuaciones',
 'upPanelActuaciones',
 'fupControl',
 'upControl'
], 
['btnChangeCaptcha',
 'btnChangeCaptcha', 
 'btnConsultaNom',
 'btnConsultaNom',
 'btnConsultarConsNum',
 'btnConsultarConsNum',
 'btnCosultaActFecha',
 'btnCosultaActFecha',
 'btnConsultarNum',
 'btnConsultarNum'
], [], 9000, '');
//]]>
</script>
{% endhighlight %}

Request that returns the options for the second select dropdown.

{% highlight text %}
managerScript:upPanelCiudad|ddlCiudad
managerScript_HiddenField:
__EVENTTARGET:ddlCiudad
__EVENTARGUMENT:
__LASTFOCUS:
__VIEWSTATE:/wEPDwUKMjA2MDQyMDE0OA9kFgICAw9kFggCBw9kFgJmD2QWAgIFDxAPFgYeDkRhdGFWYWx1ZUZpZWxkBQxBMDY1Q09ESUNJVUQeDURhdGFUZXh0RmllbGQFDEEwNjVERVNDQ0lVRB4LXyFEYXRhQm91bmRnZA8WHwIBAgICAwIEAgUCBgIHAggCCQIKAgsCDAINAg4CDwIQAhECEgITAhQCFQIWAhcCGAIZAhoCGwIcAh0CHgIfFh8QBQhBUk1FTklBIAUFNjMwMDFnEAUNQkFSUkFOUVVJTExBIAUFMDgwMDFnEAUMQk9HT1RBLCBELkMuBQUxMTAwMWcQBQxCVUNBUkFNQU5HQSAFBTY4MDAxZxAFBUJVR0EgBQU3NjExMWcQBQVDQUxJIAUFNzYwMDFnEAUKQ0FSVEFHRU5BIAUFMTMwMDFnEAUHQ1VDVVRBIAUFNTQwMDFnEAUIRFVJVEFNQSAFBTE1MjM4ZxAFCUVOVklHQURPIAUFMDUyNjZnEAUKRkxPUkVOQ0lBIAUFMTgwMDFnEAUHSUJBR1VFIAUFNzMwMDFnEAUHSVRBR1VJIAUFMDUzNjBnEAUKTUFOSVpBTEVTIAUFMTcwMDFnEAUJTUVERUxMSU4gBQUwNTAwMWcQBQlNT05URVJJQSAFBTIzMDAxZxAFBk5FSVZBIAUFNDEwMDFnEAUIUEFMTUlSQSAFBTc2NTIwZxAFBlBBU1RPIAUFNTIwMDFnEAUIUEVSRUlSQSAFBTY2MDAxZxAFCFBPUEFZQU4gBQUxOTAwMWcQBQdRVUlCRE8gBQUyNzAwMWcQBQlSSU9IQUNIQSAFBTQ0MDAxZxAFCVJJT05FR1JPIAUFMDU2MTVnEAULU0FOIEFORFJFUyAFBTg4MDAxZxAFDFNBTlRBIE1BUlRBIAUFNDcwMDFnEAUWU0FOVEEgUk9TQSBERSBWSVRFUkJPIAUFMTU2OTNnEAUKU0lOQ0VMRUpPIAUFNzAwMDFnEAUGVFVOSkEgBQUxNTAwMWcQBQtWQUxMRURVUEFSIAUFMjAwMDFnEAUOVklMTEFWSUNFTkNJTyAFBTUwMDAxZxYBZmQCKw8QZA8WIGYCAQICAgMCBAIFAgYCBwIIAgkCCgILAgwCDQIOAg8CEAIRAhICEwIUAhUCFgIXAhgCGQIaAhsCHAIdAh4CHxYgEAUDLi4uBQMuLi5nEAUEMTk5MAUEMTk5MGcQBQQxOTkxBQQxOTkxZxAFBDE5OTIFBDE5OTJnEAUEMTk5MwUEMTk5M2cQBQQxOTk0BQQxOTk0ZxAFBDE5OTUFBDE5OTVnEAUEMTk5NgUEMTk5NmcQBQQxOTk3BQQxOTk3ZxAFBDE5OTgFBDE5OThnEAUEMTk5OQUEMTk5OWcQBQQyMDAwBQQyMDAwZxAFBDIwMDEFBDIwMDFnEAUEMjAwMgUEMjAwMmcQBQQyMDAzBQQyMDAzZxAFBDIwMDQFBDIwMDRnEAUEMjAwNQUEMjAwNWcQBQQyMDA2BQQyMDA2ZxAFBDIwMDcFBDIwMDdnEAUEMjAwOAUEMjAwOGcQBQQyMDA5BQQyMDA5ZxAFBDIwMTAFBDIwMTBnEAUEMjAxMQUEMjAxMWcQBQQyMDEyBQQyMDEyZxAFBDIwMTMFBDIwMTNnEAUEMjAxNAUEMjAxNGcQBQQyMDE1BQQyMDE1ZxAFBDIwMTYFBDIwMTZnEAUEMjAxNwUEMjAxN2cQBQQyMDE4BQQyMDE4ZxAFBDIwMTkFBDIwMTlnEAUEMjAyMAUEMjAyMGdkZAI9Dw8WAh4EVGV4dAU1UHJvY2Vzb3MgY29uIHJlZ2lzdHJvIGRlIGFjdHVhY2lvbmVzIGRlc2RlIDIwMTUtMDMtMjRkZAJVD2QWAmYPZBYCAgMPPCsAEQIBEBYAFgAWAAwUKwAAZBgBBQ9ndlJlc3VsdGFkb3NOdW0PZ2TAoAmN5Ty9mF1KU7PYGKW9zmFvjIT8g0lj3Ic80FUlPg==
ddlCiudad:11001
rblConsulta:1
e1zrkoghovrd1dkuzwtawlkf:
ddlTipoSujeto:0
ddlTipoPersona:0
txtNatural:
ddlDespacho:
ddlYear:...
tbxRadicacion:
txtConsecutivo:
tbxNumeroConstruido:
ddlTipoSujeto2:0
ddlTipoPersona2:0
txtNombre:
txtResultCaptcha:
HumanVerification:SLIDER
txtNumeroProcesoID:e1zrkoghovrd1dkuzwtawlkf
ddlJuzgados:0
BotDetector:BotValue
hdfNumRadicaion:
hdControl:
__ASYNCPOST:true
:
{% endhighlight %}

Response

{% highlight text %}
1|#||4|14250|updatePanel|upPanelCiudad|
                                            <table class="contenedor">
                                                <tr>
                                                    <td colspan="2" align="left">
                                                        <h3>
                                                            <span id="lblTitulo">Seleccione donde esta localizado el proceso</span>
                                                        </h3>
                                                    </td>
                                                </tr>
                                                <tr>
                                                    <td align="right" width="20%">
                                                        <span id="lblCiudad">Ciudad:</span>
                                                    </td>
                                                    <td align="left" width="80%">
                                                        <select name="ddlCiudad" onchange="javascript:setTimeout(&#39;__doPostBack(\&#39;ddlCiudad\&#39;,\&#39;\&#39;)&#39;, 0)" id="ddlCiudad" class="campos" style="width:70%;">
							<option value="0">Seleccione la Ciudad...</option>
							<option value="63001">ARMENIA </option>
							<option value="08001">BARRANQUILLA </option>
{% endhighlight %}


In replace `<table class="contenedor">` with the updated table we received in the AJAX response.

{% highlight html %}
<div id="upPanelCiudad">
 <table class="contenedor">
{% endhighlight %}

Request for filing number

{% highlight text %}
managerScript:managerScript|btnConsultarNum
managerScript_HiddenField:
ddlCiudad:11001
ddlEntidadEspecialidad:397-True-3103-11001-Juzgado de Circuito-Civil
rblConsulta:1
tqsurrp4n5dcbu2dtnnzjkur:11001310300220140043000
ddlTipoSujeto:0
ddlTipoPersona:0
txtNatural:
ddlDespacho:
ddlYear:...
tbxRadicacion:
txtConsecutivo:
tbxNumeroConstruido:110013103
ddlTipoSujeto2:0
ddlTipoPersona2:0
txtNombre:
txtResultCaptcha:
HumanVerification:SLIDER
txtNumeroProcesoID:tqsurrp4n5dcbu2dtnnzjkur
ddlJuzgados:0
BotDetector:BotValue
hdfNumRadicaion:
hdControl:
__EVENTTARGET:
__EVENTARGUMENT:
__LASTFOCUS:
__VIEWSTATE:/wEPDwUKMjA2MDQyMDE0OA9kFgICAw9kFgg...
__ASYNCPOST:true
btnConsultarNum:Consultar
{% endhighlight %}

Response for filing number

{% highlight text %}
1|#||4|27558|updatePanel|upPanelActuaciones|
                                            <!-- Numero Consultado -->
                                            
                                            <!-- Fin Numero Consultado-->
                                            <div align="center" id="divActuaciones" class="actuaciones">
                                                <div class="titulos">
                                                    Detalle del Registro
                                                </div>
                                                <div>
                                                    <br />
                                                    <span id="lblFechaSistema">jueves, 16 de abril de 2015 - 10:29:49 a.m.</span>
                                                </div>
{% endhighlight %}