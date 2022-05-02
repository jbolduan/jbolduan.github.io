---
title:  "Getting Redirected URI's In Powershell"
date:   2018-05-19 19:08:54 +0000
categories: uri powershell redirection
---
I recently ran into an issue where I needed the direct resource URI in a Powershell script. This is incredibly useful if you need to parse the actual URI instead of just pulling the resource the redirecting URI is pointing at. In my case I wanted the Firefox URI which points at the executable so I could pull the version out the of URI without having to download and analyze the executable.

You need to first grab the response head from an Invoke-Webrequest:

```powershell
$request = Invoke-WebRequest -Method Head -Uri $Uri
```

Next, we need to determine if we're using Powershell 5 or Powershell Core and pull the Absolute URI out of the request object:

```powershell
if ($request.BaseResponse.ResponseUri -ne $null) {
    # This is for Powershell 5
    $redirectUri = $request.BaseResponse.ResponseUri.AbsoluteUri
}
elseif ($request.BaseResponse.RequestMessage.RequestUri -ne $null) {
    # This is for Powershell core
    $redirectUri = $request.BaseResponse.RequestMessage.RequestUri.AbsoluteUri
}
```

Now, sometimes you may get another redirected URI as a response.  In these cases you'll need to determine that and handle it.  This is done through error handling by catching and looking for HttpResponseException matching 302 and then running the whole thing again:

```powershell
if (($_.Exception.GetType() -match &quot;HttpResponseException&quot;) -and ($_.Exception -match &quot;302&quot;)) {
    $Uri = $_.Exception.Response.Headers.Location.AbsoluteUri
    $retry = $true
}
else {
    throw $_
}
```

This is a quick and easy way to pull the redirected URI's from a given URI.  Putting it all together we get the function below:

```powershell
function Get-RedirectedUri {
    <#
    .SYNOPSIS
        Gets the real download URL from the redirection.
    .DESCRIPTION
        Used to get the real URL for downloading a file, this will not work if downloading the file directly.
    .EXAMPLE
        Get-RedirectedURL -URL &quot;https://download.mozilla.org/?product=firefox-latest&amp;os=win&amp;lang=en-US&quot;
    .PARAMETER URL
        URL for the redirected URL to be un-obfuscated
    .NOTES
        Code from: Redone per issue #2896 in core https://github.com/PowerShell/PowerShell/issues/2896
    #>

    [CmdletBinding()]
    param (
        [Parameter(Mandatory = $true)]
        [string]$Uri
    )
    process {
        do {
            try {
                $request = Invoke-WebRequest -Method Head -Uri $Uri
                if ($request.BaseResponse.ResponseUri -ne $null) {
                    # This is for Powershell 5
                    $redirectUri = $request.BaseResponse.ResponseUri.AbsoluteUri
                }
                elseif ($request.BaseResponse.RequestMessage.RequestUri -ne $null) {
                    # This is for Powershell core
                    $redirectUri = $request.BaseResponse.RequestMessage.RequestUri.AbsoluteUri
                }

                $retry = $false
            }
            catch {
                if (($_.Exception.GetType() -match &quot;HttpResponseException&quot;) -and ($_.Exception -match &quot;302&quot;)) {
                    $Uri = $_.Exception.Response.Headers.Location.AbsoluteUri
                    $retry = $true
                }
                else {
                    throw $_
                }
            }
        } while ($retry)

        $redirectUri
    }
}
```
