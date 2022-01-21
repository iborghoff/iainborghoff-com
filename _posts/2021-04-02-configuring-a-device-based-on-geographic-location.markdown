---
layout: post
title:  "Configuring A Device Based On Geographic Location"
date:   2021-04-02 13:55:49 +0100
categories: jekyll update
---
I wanted to take Michael Niehaus’s [AutopilotBranding](https://github.com/mtniehaus/AutopilotBranding) script one step further and determine what settings and languages to apply to a device based on it’s location. But how would we be able to determine a devices location during provisioning?

Using PowerShell we could easily get a devices location:

```
PS:\Invoke-RestMethod -Uri "http://ip-api.com/json"

country : United Kingdom
countryCode : GB
region : ENG
regionName : England
```

This works well, but then we are reliant on the API being available and not falling foul of any usage limitations (unless of course you pay for access). Another alternative, and the one I chose to go with, is to rely on the home location of the current Windows account. This is set when a user selects their region during OOBE and is easily accessed via the Get-WinHomeLocation command:

```
PS:\Get-WinHomeLocation

GeoId HomeLocation
----- ------------
242 United Kingdom
```

Nice. Now we can use the Geo ID or the HomeLocation as a reference in our Config.xml:

```
<_242>
    <TimeZone>GMT Standard Time</TimeZone>
    <Language>Language-en-GB.xml</Language>
    <LanguagePack>
        <Cab>Microsoft-Windows-Client-Language-Pack_x64_en-gb.cab</Cab>
    </LanguagePack>
    <AddFeatures>
        <Feature>Language.Basic~~~en-GB~0.0.1.0</Feature>
        <Feature>Language.Handwriting~~~en-GB~0.0.1.0</Feature>
        <Feature>Language.OCR~~~en-GB~0.0.1.0</Feature>
        <Feature>Language.Speech~~~en-GB~0.0.1.0</Feature>
        <Feature>Language.TextToSpeech~~~en-GB~0.0.1.0</Feature>
    </AddFeatures>
</_242>
```

Why the underscore before the GeoID? Well XML does not support numbers as the name of an element and requires a character it does support to prefix the number.

You can find the full solution on [GitHub](https://github.com/iborghoff/AutopilotDeviceConfiguration).