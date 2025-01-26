+++ 
draft = false
date = 2025-01-26T14:58:10+01:00
title = "HHC24 - Snowball-Showdown"
description = "Hacking of browser game by tweaking of JS files"
slug = ""
authors = []
tags = [
    "HHC24",
    "Sans",
    "CTF",
]
categories = [
    "Writeup",
    "JS",
    "GameHacking",

]
externalLink = ""
series = [
    "HHC24",
]
+++

## Bronze 🥉

```Raring to go? Terrific! Here's a real brain tickler. Navigator of chaos or maestro of subtlety, which will you be? Either way, remember our objective: bring victory to Alabaster.```

- There is a hint from Dusty Giftwrap: ```Adjust the right elements and victory for Alabaster can be secured with more subtlety. Intriguing, right?```
Means: That we would need to alter the client-side values in JS or in cookies as usual for this kind of challenges.

- Let's dive in. 🤿

- **Disclaimer**: I am not sure how I got the bronze, but you can setup a multiplayer session in single player lobby and then join the game with another active window. It will stop the Womble elf for play and your team will easily win by score.

## Silver 🥈

- As was spoiled in bronze section, we need to adjust the JS variables to tweak the game's results.
- To achieve this, I used the Google Chrome override function for JS files and modified ```phaser-snowball-game.js```

    **Note:** This is persistent overwrite so your changes will have an effect after reload/refresh.
    This JS files contains various variables for game's engine configuration of the scene.

I tweaked the ```this.snowBallBlastRadius``` variable to allow our snowball cause a bigger *"damage"* as it is client-side rendered object:

```javascript
this.snowBallBlastRadius = 240;
```

After the change I reloaded the game and destroy the wall with my new **snowball of disaster** and goes to the Wombley's side of game. Then I freeze all elf on his side and easily won the game by score! 🥈

## Gold 🥇

Dusty Giftwrap gives us another hints:

```Christmas is on the line! For a mischievous edge-up, dive into the game’s code - a few client-side tweaks to speed, movement, or power might shift the balance… or just help us find that secret weapon we misplaced!```

- The above notes seems that we need to alter another variables in JS files of the game.
Question is, what can be the ***secret weapon***?

- I didn't find the way for a gold yet, but I discover the fire-rate check which I modified as follows:

```js
// Check if the percentage shot power is zero or if the throw rate limit is exceeded
        if ((false) && !archonly) {
            return;
        }
```

- for the whole time, the answer for ***secret weapon*** was:

```js
this.moasb = () => { this.ws.sendMessage({ 'type': 'moasb' }) }
```