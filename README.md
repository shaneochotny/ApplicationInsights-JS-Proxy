# Application Insights Javascript Proxy

## Description
Show how to proxy requests from the [Application Insights Javascript SDK](https://github.com/Microsoft/ApplicationInsights-JS) instead of sending telemetry directly to dc.services.visualstudio.com. This allows you to do things such as use your own custom domain and/or hide the Instrumentation Key.

### High-Level Steps
1. You'll need to use something like Azure API Management or Azure Functions to create a new API endpoint to call and proxy the telemetry over to the Application Insights URL https://dc.services.visualstudio.com/v2/track/. It needs to accept OPTIONS and POST.
2. Configure the [endpointUrl](https://github.com/Microsoft/ApplicationInsights-JS/blob/master/API-reference.md#config) option in the client-side Javascript to point to your API.


### Example
As an example, I used Azure API Management to act as my new telemetry endpoint which forwards to Application Insights. The purpose of this scenario was to ensure that all telemetry was sent back to the same domain my app is hosted on, in this case shaneochotny.com. I previously ran into scenarios where customers were filtering dc.services.visualstudio.com, but whitelisted my apps domain. This allows me to collect telemetry data where it would have otherwise been blocked.

1. Create an Azure API Management resource and add your custom domain. In my scenario, I used analytics.shaneochotny.com.

2. Add a new blank API and set <b>https://dc.services.visualstudio.com/v2/track/</b> as the <b>Web service URL</b>

3. Add an operation and specify the <b>OPTIONS</b> method.

4. Add a second operation and specify the <b>POST</b> method.

4. Select <b>All operations</b>, and then <b>Add policy</b> to <b>Inbound processing</b>.

5. Within <b>inbound</b>, use <b>set-header</b> to create a new header attribute with the name <b>iKey</b> and the value should be configured to your [Instrumentation Key](https://docs.microsoft.com/en-us/azure/application-insights/app-insights-create-new-resource).

```
<policies>
    <inbound>
        <base />
        <set-header name="iKey" exists-action="override">
            <value>xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx</value>
        </set-header>
    </inbound>
    <backend>
        <base />
    </backend>
    <outbound>
        <base />
    </outbound>
    <on-error>
        <base />
    </on-error>
</policies>
```

6. Add the script reference for the [Application Insights Javascript SDK](https://github.com/Microsoft/ApplicationInsights-JS) like you would normally. Remove <b>instrumentationKey</b> and add <b>endpointUrl</b> to the configuration. Note that I also hosted the SDK locally to avoid pulling from msecnd.net.

```
<script type="text/javascript">
    var appInsights=window.appInsights||function(a){
    function b(a){c[a]=function(){var b=arguments;c.queue.push(function(){c[a].apply(c,b)})}}var c={config:a},d=document,e=window;setTimeout(function(){var b=d.createElement("script");b.src=a.url||"https://www.shaneochotny.com/scripts/ai.0.js",d.getElementsByTagName("script")[0].parentNode.appendChild(b)});try{c.cookie=d.cookie}catch(a){}c.queue=[];for(var f=["Event","Exception","Metric","PageView","Trace","Dependency"];f.length;)b("track"+f.pop());if(b("setAuthenticatedUserContext"),b("clearAuthenticatedUserContext"),b("startTrackEvent"),b("stopTrackEvent"),b("startTrackPage"),b("stopTrackPage"),b("flush"),!a.disableExceptionTracking){f="onerror",b("_"+f);var g=e[f];e[f]=function(a,b,d,e,h){var i=g&&g(a,b,d,e,h);return!0!==i&&c["_"+f](a,b,d,e,h),i}}return c
    }({
        endpointUrl: "https://analytics.shaneochotny.com"
    });

    window.appInsights=appInsights,appInsights.queue&&0===appInsights.queue.length&&appInsights.trackPageView();
</script>
```
