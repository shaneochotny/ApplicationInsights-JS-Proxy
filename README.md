# Application Insights Javascript Proxy

## Description
Show how to proxy requests from the [Application Insights Javascript SDK](https://github.com/Microsoft/ApplicationInsights-JS) instead of sending telemetry directly to dc.services.visualstudio.com. This allows you to do things such as use your own custom domain and/or hide the Instrumentation Key.

### High-Level Steps
1. You'll need to use something like Azure API Management or Azure Functions to create a new API endpoint to call and proxy the telemetry over to Application Insights.
2. Configure the endpointUrl option in the client-side Javascript to point to your API.


### Example
As an example, I used Azure API Management to act as my new telemetry endpoint which forwards to Application Insights. The purpose of this scenario was to ensure that all telemetry was sent back to the same domain my app is hosted on, in this case shaneochotny.com. I previously ran into scenarios where customers were filtering dc.services.visualstudio.com, but whitelisted my apps domain. This allows me to collect telemetry data where it would have otherwise been blocked.

1. 
