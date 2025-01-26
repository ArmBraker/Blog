+++ 
draft = false
date = 2025-01-26T14:58:44+01:00
title = "HHC24 - Dronmagedon"
description = "OSINT/Web-based hacking challenge"
slug = ""
authors = []
tags = [
    "HHC24",
    "Sans",
    "CTF",
]
categories = [
    "Writeup",
    "OSINT",
    "GoogleEarth",
    "Cartographic",
    "SQLinjection"
]
externalLink = ""
series = [
    "HHC24",
]
+++


## Introduction

Elf give us a hint that we will work with GEO data in ```.KML``` file. This files can be imported into GIS programs or to well known Google Earth.

## Silver 🥈

### 1 Phase - Login

After uploading the file into Google Earth we got a "password" into web application: ```GUMDROP1```.
The username for the application is also stored in the ```.KML``` file :

```xml
<name>fritjolf-Path</name>
```

The username is: ```fritjolf```

Now we are Authenticated.

### 2 Phase

We found a CSV file in pilot profile.
The file contains some coordinates, so lets create our own KML file:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<kml xmlns="http://www.opengis.net/kml/2.2" xmlns:gx="http://www.google.com/kml/ext/2.2" xmlns:kml="http://www.opengis.net/kml/2.2" xmlns:atom="http://www.w3.org/2005/Atom">
<Document>
    <name>fritjolf-Path</name>
    <gx:CascadingStyle kml:id="__managed_style_2398E937F13251C4AAA6">
        <Style>
            <IconStyle>
                <scale>1.2</scale>
                <Icon>
                    <href>https://earth.google.com/earth/rpc/cc/icon?color=1976d2&amp;id=2000&amp;scale=4</href>
                </Icon>
                <hotSpot x="64" y="128" xunits="pixels" yunits="insetPixels"/>
            </IconStyle>
            <LabelStyle>
            </LabelStyle>
            <LineStyle>
                <color>ff2dc0fb</color>
                <width>6</width>
            </LineStyle>
            <PolyStyle>
                <color>40ffffff</color>
            </PolyStyle>
            <BalloonStyle>
                <displayMode>hide</displayMode>
            </BalloonStyle>
        </Style>
    </gx:CascadingStyle>
    <gx:CascadingStyle kml:id="__managed_style_177256F1433251C4AAA6">
        <Style>
            <IconStyle>
                <Icon>
                    <href>https://earth.google.com/earth/rpc/cc/icon?color=1976d2&amp;id=2000&amp;scale=4</href>
                </Icon>
                <hotSpot x="64" y="128" xunits="pixels" yunits="insetPixels"/>
            </IconStyle>
            <LabelStyle>
            </LabelStyle>
            <LineStyle>
                <color>ff2dc0fb</color>
                <width>4</width>
            </LineStyle>
            <PolyStyle>
                <color>40ffffff</color>
            </PolyStyle>
            <BalloonStyle>
                <displayMode>hide</displayMode>
            </BalloonStyle>
        </Style>
    </gx:CascadingStyle>
    <StyleMap id="__managed_style_08C7CAFB983251C4AAA6">
        <Pair>
            <key>normal</key>
            <styleUrl>#__managed_style_177256F1433251C4AAA6</styleUrl>
        </Pair>
        <Pair>
            <key>highlight</key>
            <styleUrl>#__managed_style_2398E937F13251C4AAA6</styleUrl>
        </Pair>
    </StyleMap>
    <Placemark id="06AEB73AFD323DB38AEB">
        <name>fritjolf-Path</name>
        <LookAt>
            <longitude>98.45958448116764</longitude>
            <latitude>-86.34782448283728</latitude>
            <altitude>3933.282925076723</altitude>
            <heading>266.5424437916514</heading>
            <tilt>0</tilt>
            <gx:fovy>35</gx:fovy>
            <range>5615503.459923863</range>
            <altitudeMode>absolute</altitudeMode>
        </LookAt>
        <styleUrl>#__managed_style_08C7CAFB983251C4AAA6</styleUrl>
        <LineString>
            <coordinates>
                144.8567879816271,-37.42277804382341,329.1721074691694 
                142.4357677725014,-38.0569169391843,136.9853073940529 
                143.9329094555584,-37.80217469906655,377.501624914929 
                142.2754454646221,-38.0682499155867,143.2939472276593 
                141.756352258091,-34.52324587244343,61.14087621810435 
                145.513859306511,-36.74357572393437,168.6693280467137 
                144.745994150535,-37.89721189352699,4.240033669814256 
                145.8966329539992,-37.00702150480869,303.1828443856635
            </coordinates>
        </LineString>
    </Placemark>
</Document>
</kml>
```

- Importing to Google Earth, it will create a new flying's coordinates.
  After looking into the new map, we noticed that each point is hiding some letters:

```K,w,A,H,I,E,L,F```

- Seems that the string doesn't make any sense, lets follow the coordinates in order:
```E-FLHAWK```
- The string still doesn't make any sense, may I make a typo, but in shor iteration I found a correct name:
```ELF-HAWK```
- Drone details:

```
Name: ELF-HAWK, Quantity: 40, Weapons: Snowball-launcher
Comments for ELF-HAWK
These drones will work great to find Alabasters snowball warehouses. I have hid the activation code in the dataset ELF-HAWK-dump.csv. We need to keep it safe, for now it's under /files/secret.
We need to make sure we have enough of these drones ready for the upcoming operation. Well done on hiding the activation code in the dataset. If anyone finds it, it will take them a LONG time or forever to carve the data out, preferably the LATTER.
```

```https://hhc24-dronepath.holidayhackchallenge.com/files/secret/ELF-HAWK-dump.csv```

- Examine the CSV and found the same patter of Admin Code hidden in coordinates. I used Query tool of CSV Rainbow and extract these coordinated and create a new KML file from previous template. After importing this KML file into Google Earth and noticed that coordinates are re-written on 3D model of planet, so I tried to look for another online GIS tool which created me a flat 2D model of flight patter on 2D map.

- Admin password for Silver medal 🥈 was: ```DroneDataAnalystExpertMedal```

### GOLD

- SQLinjection in workshop' endpoint:

```SQL
ELF-HAWK'OR '1' = '1
```

- Reveals all drons' names:

**Name**: ELF-HAWK, Quantity: 40, Weapons: Snowball-launcher
**Name**: Pigeon-Lookalike-v4, Quantity: 20, Weapons: Surveillance Camera
**Name**: FlyingZoomer, Quantity: 4, Weapons: Snowball-Dropper
**Name**: Zapper, Quantity: 5, Weapons: CarrotSpike

- Comments for each drone:

***Comments for Pigeon-Lookalike-v4***

```txt
This is a great drone for surveillance, but we need to keep it out of the rain.
I cant believe we are using pigeons for surveillance. If anyone finds out, there will most likely be a conspiracy theory about it.
I heard a rumor that there is something fishing with some of the files. There was some talk about only TRUE carvers would find secrets and that FALSE ones would never find it.
```

***Comments for FlyingZoomer***

```txt
This drone is perfect for dropping snowballs on unsuspecting targets
```

***Comments for Zapper***

```txt
This is sort of primitive, but it works!
```

- From comments of Pigeon-Lookalike-v4 `I heard a rumor that there is something fishing with some of the files. There was some talk about only TRUE carvers would find secrets and that FALSE ones would never find it.` I know that I have to look into already provided files and looks for `True` or `False` values.
- The latest obtained file ```ELF-HAWK-dump.csv``` contains lots of columns with `True` or `False` values.
- After couple of hours, I figure out the meaning of the HINT. I need to replace the True and False with Binary representations of the values (0,1).
- Excel or Google Docs are great tools for this.
- After a clean up of the .CSV and transfiguration of Booleans, I copied the values into CyberChef with following recipe: <https://gchq.github.io/CyberChef/#recipe=From_Binary('None',8)Unicode_Text_Format(false,false)Remove_null_bytes()&oenc=65001&ieol=CRLF>

- The result was ASCII art:

```text
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::*::::::::::::::::::
:::::::::::::::::::::::::::::::-------------=--------::::::::::::::::::::::::::::::::::::
::::::::::::::::::::::------------------------===-=======--=-::::::::::-:::::::::::::::::
::::::::::::::::::::------------:------------=-====================---:::::::::=+::::::::
:::::::::::::::::------------------------------=====================-------::::::::::::::
::::::::::::::-------------------------------------================:------:::::::::::::::
::::::::::::--------------------------------------==============-::--------:::::::::-::::
::::::::::::-------:--------@+:::::::::--=@--------:===========-::-::----==---:::::::::::
::::-------:::::----------@---::::::---+-==+@--------=========-:--:------=====---::::::::
::::--------::::::-------#--------------=-+@------------===------::-----====--==---::::::
::::-------:-:::::::------@=@=++#+++++@@@@@=-----------------:::--------------==---::::::
::::----------::::=-#-:----**%@+++++++%@@=::::::---%@------:--------:--@-+::-------::::::
::::-----:----:::::::::::--::@@**%@--::::::::::::::--=+@------------@--:::::------@::::::
::::---+@::::::---+@:::::::::#@-@--:::::-:=*=-::-----=+*=*=--------@:--:::::::-----=:::::
::::@-:::-::::::-----=@:-:::@+@%---------------==-==+@@@@@=@------@---------:::::--==+%::
:::#:::::::::::-----=+*@:::%#@#-=---------===++*%@@+@=+*#-+*=@-----#====-----------**-%::
::@--::-:::--:---==++*@-:@=+@=+-@=*+++++++**@#%*@-##**-@##%=#%@@@@#*@###@=+**@*****@@@:::
:::@*=--++++++++**@@@@@@*#@-+%@*=*+****@@@+@***@%@@%%%@-%@*@@@@@@@@@@@@@@%%#%%%@@@@@%::::
:::@@@@@@@++#*####@@@@@@@==---====+##@*%=+@*@*%%@@@@@@@@@@@@@@@=--@+@@@@+@@@@@@@@@@-:::::
::::=*%%%%%%%%%%%@@%@@#@-#*+++++====@-++###@%@*@@@@+@@@@-**+--::::--@@%@%%@%%%%%@@@-:::::
::::---@@@@##@@@@@@@@@--+@%-#+#**+=+++**%@@@@@@@##%**%--:::::::--*----=*@@@@@@@*@@---::::
::::---@@***%%%%@@@@*@-=-+=@#=#%##***##@@@@@#@@*@%%==---:::::::::::----=+---------=--::::
::::----@+=%#@@@=@@-----##@+:-=%@@%##%@@@@@@@@@@@@*+=-----::::::::::::=+*-@:----===--::::
::::---------------------*@##=+@@%@==-+@@@@@@@@@@@-+=---------------===+**--=======-:::::
:::---------------:------%+#%@@@@@#%%%%@@@@#@@@@@@@-+======---------==***#@========-:::::
:::-%-%---------:---------*-*##%@@@@@@@@@@@@@@@@@--=@@-*===++++++++++***@*===++++++=-::::
:::--+---------=-------:-----#==#@%%%@@@@@*@%@@@----@+@@@=***@@@@***@@@@%===++++-++=-::::
:::--------------:::::--------------##-----@@--------@%@#@@%%%%@@@@@@#@=====+++++++=-::::
:::---------------::::::---------------------=====---@@##@@@@@@@@@@@#%#-=====+++++--:::::
:::---======-------------------------=----==========--*=@@%@++*@@%%%@@-======:----==-::::
:::---===============------------------===============-----#@@@@@-----===-::---=====-::::
:::--=============+===--------------===-==================--------======::----=======-:::
:::--================---::::-=======-======================+=====+====::------===+===-:::
:::--===================--:::::====================+====-:---==+++=::-----=======---=-:::
:::--========:===========------:=====================:::-----====:-----==========+===-:::
 / ___/ _ \|  _ \| ____\ \      / / _ \|  _ \|  _ \   _____  ====:-----==========+===-:::
| |  | | | | | | |  _|  \ \ /\ / / | | | |_) | | | | |_____| ====:-----==========+===-:::
| |__| |_| | |_| | |___  \ V  V /| |_| |  _ <| |_| | |_____| ====:-----==========+===-:::
 \____\___/|____/|_____|__\_/\_/__\___/|_| \_\____/  _  _________   ______    _    ____  
| ____\ \/ /  _ \| ____|  _ \_   _|_   _| | | |  _ \| |/ / ____\ \ / / ___|  / \  |  _ \ 
|  _|  \  /| |_) |  _| | |_) || |   | | | | | | |_) | ' /|  _|  \ V / |     / _ \ | |_) |
| |___ /  \|  __/| |___|  _ < | |   | | | |_| |  _ <| . \| |___  | || |___ / ___ \|  _ < 
|_____/_/\_\_| __|_____|_|_\_\|_| __|_|  \___/|_| \_\_|\_\_____| |_| \____/_/   \_\_| \_\
\ \   / / ____|  _ \|  \/  | ____|  _ \  / \  | |    ==========---======++++=+=--+++=-:::
 \ \ / /|  _| | |_) | |\/| |  _| | | | |/ _ \ | |    ==========---======++++=+=--+++=-:::
  \ V / | |___|  _ <| |  | | |___| |_| / ___ \| |___ ==========---======++++=+=--+++=-:::
   \_/  |_____|_| \_\_|  |_|_____|____/_/   \_\_____|==========---======++++=+=--+++=-:::
::::--====+++=---++++++=+========------::::=-:---==============---======++++=+=--+++=-:::
::::--==+++++++==---+++++++++++========-----================++++==-========-++=++====-:::
:::::--====+++++-++--++++++++++=--------=-==============+++---------=====++=+++++::::::::
::::::::======+++=+++=+++++++++++++++=++++===========++++:-------=---=-=----:::::::::::::
::::::::::::::::--=-=======++=++++++++++++++============--------------:::::::::::::::::::
:::::::::::::::::::::::::::------===-==-===-==-----::-:::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
:::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::::
```

- Admin password for GOLD 🥇 was `EXPERTTURKEYCARVERMEDAL`
