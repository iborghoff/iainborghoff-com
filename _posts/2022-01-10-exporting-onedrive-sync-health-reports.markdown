---
layout: post
title:  "Exporting OneDrive Sync Health Reports"
date:   2022-01-10 10:00:49 +0100
categories:
  - OneDrive
  - PowerShell
---
Currently in preview via the [Microsoft 365 Apps admin center](https://config.office.com/officeSettings/onedrive), OneDrive sync health reports allow you to see the sync status of all devices with in your tenant. 
It's not possible (yet) to export this data via the admin center or an API, so we need to get a little creative if we want to export it.

The first thing we need to do is work out the requests which are happening in your browser when you are accessing the admin center. You can see this via your browsers admin tools. I'm using Edge, so for me this is accessed via More Tools > Developer Tools (or Ctrl+Shift+I).

We'll be working from the Network tool.

![Edge network tool](/images/edge-network-tool.png)

With the Network tool open, navigate to Health > OneDrive Sync. This would have generated some requests we aren't interested in, so lets first clear those requests via the Clear button ![clear button](/images/edge-network-tool-clear.png).

Select 'View devices which have sync errors', this will generate two requests in the Network tool.

![requests](/images/onedrive-sync-errors-requests.png)

Select one of the requests in the Network Tool, and then select Headers. This will show some more information about the request. Find the request which has a Request Method of GET, right click this and select Copy > Copy as PowerShell.

*Please note that I'll replace the Bearer token with 123456 and removed the SID's in these examples, so your code will look a little different.*

This will give us the code below. 

```powershell
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession
$session.UserAgent = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/97.0.4692.71 Safari/537.36 Edg/97.0.1072.55"
Invoke-WebRequest -UseBasicParsing -Uri "https://clients.config.office.net/odbhealth/v1.0/synchealth/reports?top=30&filter=cast(TotalErrorCount,%27Int32%27)+ne+0&orderby=UserName+asc" `
-WebSession $session `
-Headers @{
"method"="GET"
  "authority"="clients.config.office.net"
  "scheme"="https"
  "path"="/odbhealth/v1.0/synchealth/reports?top=30&filter=cast(TotalErrorCount,%27Int32%27)+ne+0&orderby=UserName+asc"
  "x-api-name"="https://clients.config.office.net/odbhealth/v1.0/synchealth/- api name not register"
  "sec-ch-ua-mobile"="?0"
  "authorization"="Bearer 123456"
  "accept"="application/json"
  "x-requested-with"="XMLHttpRequest"
  "sec-ch-ua"="`" Not;A Brand`";v=`"99`", `"Microsoft Edge`";v=`"97`", `"Chromium`";v=`"97`""
  "sec-ch-ua-platform"="`"Windows`""
  "origin"="https://config.office.com"
  "sec-fetch-site"="cross-site"
  "sec-fetch-mode"="cors"
  "sec-fetch-dest"="empty"
  "referer"="https://config.office.com/"
  "accept-encoding"="gzip, deflate, br"
  "accept-language"="en-US,en;q=0.9"
}
```
I find Invoke-RestMethod much easier to work with, so let's update this code to the below.

```powershell
$results = Invoke-RestMethod -Method Get -Uri "https://clients.config.office.net/odbhealth/v1.0/synchealth/reports?top=30&filter=cast(TotalErrorCount,%27Int32%27)+ne+0&orderby=UserName+asc" `
-Headers @{
  "authority"                 = "clients.config.office.net"
  "scheme"                    = "https"
  "path"                      = "/odbhealth/v1.0/synchealth/reports?top=30&filter=cast(TotalErrorCount,%27Int32%27)+ne+0&orderby=UserName+asc"
  "x-api-name"                = "https://clients.config.office.net/odbhealth/v1.0/synchealth/- api name not register"
  "sec-ch-ua-mobile"          = "?0"
  "authorization"             = "Bearer 123456"
  "accept"                    = "application/json"
  "x-requested-with"          = "XMLHttpRequest"
  "sec-ch-ua"                 = "`" Not;A Brand`";v=`"99`", `"Microsoft Edge`";v=`"97`", `"Chromium`";v=`"97`""
  "sec-ch-ua-platform"        = "`"Windows`""
  "origin"                    = "https://config.office.com"
  "sec-fetch-site"            = "cross-site"
  "sec-fetch-mode"            = "cors"
  "sec-fetch-dest"            = "empty"
  "referer"                   = "https://config.office.com/"
  "accept-encoding"           = "gzip, deflate, br"
  "accept-language"           = "en-US,en;q=0.9"
}
```

The $results variable contains an object we can now easily work with, for example $retults.reports will return each result. Here's an example.

```powershell
type                         : odb
schemaVersion                : 1.0
oneDriveDeviceId             : 123456
userName                     : Blogs, Joe
userEmail                    : joe.blogs@company.com
deviceName                   : ADEVICENAME
```

There are a couple of caveats to all of this:

<ol>
<li>It's not possible to generate a Bearer token any other way than via the Apps admin center</li>
<li>Results from the request are limited to 30 devices/users</li>
</ol>

Unfortunately point 1 above is something I wasn't able to build a work around for, if you want to access the data like this then you'll need to generate the Bearer token manually via the admin center each time. But point 2 is possible to solve.

If you want to see how I tackled it, then head over to my [GitHub](https://github.com/iborghoff/EndpointManagement/tree/main/OneDrive).