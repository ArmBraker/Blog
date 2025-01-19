+++ 
draft = false
date = 2025-01-19T16:19:30+01:00
title = "HHC24 - Hardware Hacking 101"
description = "A simple hardware terminal application enhanced by database and hashing exploitation"
slug = ""
authors = []
tags = [
    "HHC24",
    "Sans",
    "CTF",
]
categories = [
    "Writeup",
    "Hardware",
    "Databases",
    "Hashing",
    "ReverseEngineering",
]
externalLink = ""
series = []
+++


## Objective

We need to access the terminal and modify the access database. We're looking to grant access to card number 42.

``` bash
--------------------------------------------------------
                   ___     _      _  _   
                  / __|   | |    | || |  
                  \__ \   | |__  | __ |  
                  |___/   |____| |_||_|  
                _|"""""|_|"""""|_|"""""| 
                "`-0-0-'"`-0-0-'"`-0-0-' 
--------------------------------------------------------
  Santa's Little Helper - Access Card Maintenance Tool

          Tool Name: slh

options:
  -h, --help            show this help message and exit
  --view-config         View current configuration.
  --view-cards          View current values of all access cards.
  --view-card ID        View a single access card by ID.
  --set-access ACCESS_LEVEL
                        Set access level of access card. Must be 0 (No Access) or 1 (Full Access).
  --id ID               ID of card to modify.
  --passcode PASSCODE   Passcode to make changes.
  --new-card            Generate a new card ID.
--------------------------------------------------------
```

- History command revealed following information:

```bash
   1  cd /var/www/html
    2  ls -l
    3  sudo nano index.html
    4  cd ..
    5  rm -rf repo
    6  sudo apt update
    7  sudo apt upgrade -y
    8  ping 1.1.1.1
    9  slh --help
   10  slg --config
   11  slh --passcode CandyCaneCrunch77 --set-access 1 --id 143
   12  df -h
   13  top
   14  ps aux | grep apache
   15  sudo systemctl restart apache2
   16  history | grep ssh
   17  clear
   18  whoami
   19  crontab -e
   20  crontab -l
   21  alias ll='ls -lah'
   22  unalias ll
   23  echo "Hello, World!"
   24  cat /etc/passwd
   25  sudo tail -f /var/log/syslog
   26  mv archive.tar.gz /backup/
   27  rm archive.tar.gz
   28  find / -name "*.log"
   29  grep "error" /var/log/apache2/error.log
   30  history
```

- Card ID 143 content:

```bash
slh --view-card 143

(143, 'fa9717e8-e1af-42e9-ad68-eeb96ab39d13', 0, '728f25dda2ef6168ada53065883756140cfe663bce4e827084b745a32255157d')
```

- According to documentation, the full access is **1**, but the card with ID 143 has value **0**.
- To finish the objective for broz, following command needs to be executed:

```bash
 slh --passcode CandyCaneCrunch77 --set-access 1 --id 42 
```

- **SUCCESS**

```powershell

       *   *   *   *   *   *   *   *   *   *   *
   *                                             *
*      ❄  ❄  ❄  ❄  ❄  ❄  ❄  ❄  ❄  ❄  ❄  ❄  ❄     *
 *  $$$$$$\   $$$$$$\   $$$$$$\  $$$$$$$$\  $$$$$$\   $$$$$$\  * 
  * $$  __$$\ $$  __$$\ $$  __$$\ $$  _____|$$  __$$\ $$  __$$\ *
   *$$ /  $$ |$$ /  \__|$$ /  \__|$$ |      $$ /  \__|$$ /  \__| *
    $$$$$$$$ |$$ |      $$ |      $$$$$\    \$$$$$$\  \$$$$$$\   
   *$$  __$$ |$$ |      $$ |      $$  __|    \____$$\  \____$$\  *
  * $$ |  $$ |$$ |  $$\ $$ |  $$\ $$ |      $$\   $$ |$$\   $$ | *
*   $$ |  $$ |\$$$$$$  |\$$$$$$  |$$$$$$$$\ \$$$$$$  |\$$$$$$  |   *
 *  \__|  \__| \______/  \______/ \________| \______/  \______/  *
*         *    ❄             ❄           *        ❄    ❄    ❄   *
   *        *     *     *      *     *      *    *      *      *
   *  $$$$$$\  $$$$$$$\   $$$$$$\  $$\   $$\ $$$$$$$$\ $$$$$$$$\ $$$$$$$\  $$\  *
   * $$  __$$\ $$  __$$\ $$  __$$\ $$$\  $$ |\__$$  __|$$  _____|$$  __$$\ $$ | *
  *  $$ /  \__|$$ |  $$ |$$ /  $$ |$$$$\ $$ |   $$ |   $$ |      $$ |  $$ |$$ |*
  *  $$ |$$$$\ $$$$$$$  |$$$$$$$$ |$$ $$\$$ |   $$ |   $$$$$\    $$ |  $$ |$$ | *
 *   $$ |\_$$ |$$  __$$< $$  __$$ |$$ \$$$$ |   $$ |   $$  __|   $$ |  $$ |\__|*
  *  $$ |  $$ |$$ |  $$ |$$ |  $$ |$$ |\$$$ |   $$ |   $$ |      $$ |  $$ |   *
*    \$$$$$$  |$$ |  $$ |$$ |  $$ |$$ | \$$ |   $$ |   $$$$$$$$\ $$$$$$$  |$$\ *
 *    \______/ \__|  \__|\__|  \__|\__|  \__|   \__|   \________|\_______/ \__|  *
  *                                                            ❄    ❄    ❄   *
   *      *    *    *    *    *    *    *    *    *    *    *    *    *    *    
```

## Gold

- Elf says now: *"There’s a tougher route if you're up for the challenge to earn the Gold medal. It involves directly modifying the database and generating your own HMAC signature."*

- Playing around the command, the --new-card has generated following output:

```bash
slh@slhconsole\> slh --new-card
Generate new card with uuid: f8aae97c-8345-4684-84fe-d50b9f7b5a3d
```

- As the Elf said, need to modify the database, so let's check file type of access_cards:

```bash
file access_cards 

access_cards: SQLite 3.x database, last written using SQLite version 3040001, file counter 4, database pages 32, cookie 0x2, schema 4, UTF-8, version-valid-for 4
```

- We will use **sqllite3** tool available in the challenge console.

```bash
sqllite3 access_access_cards
```

### Database

- 2 tables:
  - access_cards
  - config

```bash
sqlite> .tables
access_cards  config 
```

- Reveal content of databases:

```SQL
SELECT * FROM config
1|hmac_secret|9ed1515819dec61fd361d5fdabb57f41ecce1a5fe1fe263b98c0d6943b9b232e
2|hmac_message_format|{access}{uuid}
3|admin_password|3a40ae3f3fd57b2a4513cca783609589dbe51ce5e69739a33141c5717c20c9c1
4|app_version|1.0
```

- Let's check content of Access Card 42:

```SQL
SELECT * FROM access_cards LIMIT 1 OFFSET 41;
42|c06018b6-5e80-4395-ab71-ae5124560189|0|ecb9de15a057305e5887502d46d434c9394f5ed7ef1a51d2930ad786b02f6ffd
```

- Now we know know all needed information:
  - HMAC secret
  - HMAC format
  - The card NO. which need to be modified (42).
  - Needed access level: **FULL (1)**

- **Hint:** We got hint from previous challenge that we need to use HMAC to sign the card signature:

  - In cyberchef I used following recipe: <https://gchq.github.io/CyberChef/#recipe=HMAC(%7B'option':'UTF8','string':'9ed1515819dec61fd361d5fdabb57f41ecce1a5fe1fe263b98c0d6943b9b232e'%7D,'SHA256')&input=MWMwNjAxOGI2LTVlODAtNDM5NS1hYjcxLWFlNTEyNDU2MDE4OQ>

  - Modifying the access level (1) for card 42:

```SQL
sqlite> UPDATE access_cards SET access = 1, sig = '135a32d5026c5628b1753e6c67015c0f04e26051ef7391c2552de2816b1b7096' WHERE id = 42;
```

```powershell
       *   *   *   *   *   *   *   *   *   *   *
   *                                             *
*      ❄  ❄  ❄  ❄  ❄  ❄  ❄  ❄  ❄  ❄  ❄  ❄  ❄     *
 *  $$$$$$\   $$$$$$\   $$$$$$\  $$$$$$$$\  $$$$$$\   $$$$$$\  * 
  * $$  __$$\ $$  __$$\ $$  __$$\ $$  _____|$$  __$$\ $$  __$$\ *
   *$$ /  $$ |$$ /  \__|$$ /  \__|$$ |      $$ /  \__|$$ /  \__| *
    $$$$$$$$ |$$ |      $$ |      $$$$$\    \$$$$$$\  \$$$$$$\   
   *$$  __$$ |$$ |      $$ |      $$  __|    \____$$\  \____$$\  *
  * $$ |  $$ |$$ |  $$\ $$ |  $$\ $$ |      $$\   $$ |$$\   $$ | *
*   $$ |  $$ |\$$$$$$  |\$$$$$$  |$$$$$$$$\ \$$$$$$  |\$$$$$$  |   *
 *  \__|  \__| \______/  \______/ \________| \______/  \______/  *
*         *    ❄             ❄           *        ❄    ❄    ❄   *
   *        *     *     *      *     *      *    *      *      *
   *  $$$$$$\  $$$$$$$\   $$$$$$\  $$\   $$\ $$$$$$$$\ $$$$$$$$\ $$$$$$$\  $$\  *
   * $$  __$$\ $$  __$$\ $$  __$$\ $$$\  $$ |\__$$  __|$$  _____|$$  __$$\ $$ | *
  *  $$ /  \__|$$ |  $$ |$$ /  $$ |$$$$\ $$ |   $$ |   $$ |      $$ |  $$ |$$ |*
  *  $$ |$$$$\ $$$$$$$  |$$$$$$$$ |$$ $$\$$ |   $$ |   $$$$$\    $$ |  $$ |$$ | *
 *   $$ |\_$$ |$$  __$$< $$  __$$ |$$ \$$$$ |   $$ |   $$  __|   $$ |  $$ |\__|*
  *  $$ |  $$ |$$ |  $$ |$$ |  $$ |$$ |\$$$ |   $$ |   $$ |      $$ |  $$ |   *
*    \$$$$$$  |$$ |  $$ |$$ |  $$ |$$ | \$$ |   $$ |   $$$$$$$$\ $$$$$$$  |$$\ *
 *    \______/ \__|  \__|\__|  \__|\__|  \__|   \__|   \________|\_______/ \__|  *
  *                                                            ❄    ❄    ❄   *
   *      *    *    *    *    *    *    *    *    *    *    *    *    *    *         
```
