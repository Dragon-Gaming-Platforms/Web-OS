<h1 align="center">Fireburst OS</h1>

<div align="center">
  <img src="os/assets/images/dragon-login.png" height="425" />
</div>

## Table of Contents

- [Introduction](#Introduction)
- [Features](#Features)
- [Deploy](#Deployment)
- [Configure Backend](#Backend)

## Introduction

This repository hosts a branch of [WebVM](https://github.com/leaningtech/Webvm) with [Fireburst OS](https://github.com/Dragon-Gaming-Platforms/Web-Operating-System/), located in the [os/](os/) folder.

## Features
[WebVM](https://github.com/leaningtech/webvm/) is used to run the Terminal app in the OS and [cheerpj](https://github.com/leaningtech/cheerpj/) is used to run the Java Runner app.

## Deployment
This is a static repository and therefore can be hosted on GitHub Pages by GitHub actions. Simply navigate to [.github/workflows/deploy.yml](.github/workflows/deploy.yml) and execute [the workflow](.github/workflows/deploy.yml)! Configuration for the workflow is ```debian_mini``` ```950MB``` and
 - [x] deploy to pages 
 - [ ] update pages deployment
### Backend
Unfortunately, due to security restricions I cannot share the backend URL that I use. It is hosted on [Google Apps Scripts](https://script.google.com) and is therefore easy to deploy.

1. Create a new Apps Script at [script.google.com](https://script.google.com)
2. Delete everything in ```Code.gs``` and replace it with this:

```JAVASCRIPT
// ==========================================
// UNIFIED WEB OS BACKEND
// Handles: Authentication, Chat Rooms, and AI
// ==========================================

const ADMINS = ["EMAIL_HERE", "EMAIL_2_HERE"];
const GROQ_API_KEY = "placeholder"; 

// ALLOWED ORIGINS - Add your GitHub Pages domain
const ALLOWED_ORIGINS = [
  "https://dragon-gaming-platforms.github.io",
  "http://localhost:8000", // for local testing
  "http://localhost:3000",
  "http://127.0.0.1:8000"
  "Add your domain to this list here"
];

// ------------------------------------------
// 1. NETWORK & SECURITY ROUTERS
// ------------------------------------------

// Handles GET requests (JSONP bypass + CORS)
function doGet(e) {
  var origin = e.parameter.origin || getOriginFromReferer(e);
  var res = handleRequest(e.parameter);
  
  // JSONP callback support
  if (e.parameter.callback) {
    return ContentService.createTextOutput(e.parameter.callback + '(' + JSON.stringify(res) + ')')
      .setMimeType(ContentService.MimeType.JAVASCRIPT);
  }
  
  // Regular JSON with CORS headers
  return createCorsResponse(JSON.stringify(res), origin);
}

// Handles POST requests with CORS
function doPost(e) {
  var origin = getOriginFromReferer(e);
  var req;
  
  try { 
    req = JSON.parse(e.postData.contents); 
  } catch(err) { 
    req = e.parameter; 
  }
  
  var res = handleRequest(req);
  return createCorsResponse(JSON.stringify(res), origin);
}

// Handle OPTIONS requests (CORS preflight)
function doOptions(e) {
  var origin = getOriginFromReferer(e);
  return ContentService
    .createTextOutput('')
    .setMimeType(ContentService.MimeType.TEXT)
    .setHeader('Access-Control-Allow-Origin', origin)
    .setHeader('Access-Control-Allow-Methods', 'GET, POST, OPTIONS')
    .setHeader('Access-Control-Allow-Headers', 'Content-Type, Authorization')
    .setHeader('Access-Control-Max-Age', '86400');
}

// Helper: Create response with CORS headers
function createCorsResponse(content, origin) {
  return ContentService
    .createTextOutput(content)
    .setMimeType(ContentService.MimeType.JSON)
    .setHeader('Access-Control-Allow-Origin', origin)
    .setHeader('Access-Control-Allow-Methods', 'GET, POST, OPTIONS')
    .setHeader('Access-Control-Allow-Headers', 'Content-Type');
}

// Helper: Get origin from referer
function getOriginFromReferer(e) {
  try {
    var referer = e.parameter.origin || '';
    
    // Check if origin is in allowed list
    for (var i = 0; i < ALLOWED_ORIGINS.length; i++) {
      if (referer.indexOf(ALLOWED_ORIGINS[i]) === 0) {
        return ALLOWED_ORIGINS[i];
      }
    }
    
    // Default to first allowed origin
    return ALLOWED_ORIGINS[0];
  } catch(err) {
    return ALLOWED_ORIGINS[0];
  }
}

// ------------------------------------------
// 2. MASTER LOGIC CONTROLLER
// ------------------------------------------
function handleRequest(req) {
  try {
    var action = req.action;
    var email = (req.email || "").toLowerCase().trim();
    var password = req.password || "";
    var room = req.room || "general";
    const prop = PropertiesService.getScriptProperties();

    // --- AUTHENTICATION ACTIONS ---
    if (action === "register") {
      if (!email.includes("@")) return { error: "Invalid email." };
      if (password.length < 4) return { error: "Password must be at least 4 characters." };
      if (prop.getProperty("user_" + email)) return { error: "Email is already registered." };
      
      prop.setProperty("user_" + email, password);
      return { success: true };
    }
    
    if (action === "login") {
      var storedPass = prop.getProperty("user_" + email);
      if (!storedPass || storedPass !== password) return { error: "AuthFailed" };
      return { success: true, isAdmin: ADMINS.includes(email) };
    }

    // --- GLOBAL SECURITY GATE ---
    var authPass = prop.getProperty("user_" + email);
    if (!authPass || authPass !== password) return { error: "AuthFailed" };

    // --- CHAT ACTIONS ---
    if (action === "getData") {
      return { messages: getMessages(room), rooms: getRooms(), admin: ADMINS.includes(email) };
    }
    if (action === "send") {
      sendMessage(room, email, req.text); 
      return { success: true };
    }
    if (action === "createRoom") {
      createRoom(req.roomName); 
      return { success: true };
    }
    if (action === "deleteMessage") {
      if (ADMINS.includes(email)) deleteMessage(room, req.msgId); 
      return { success: true };
    }

    // --- AI ASSISTANT ACTIONS ---
    if (action === "ai_chat") {
      var aiMessages = typeof req.messages === 'string' ? JSON.parse(req.messages) : req.messages;
      
      var apiUrl = "https://api.groq.com/openai/v1/chat/completions";
      var payload = { 
        "model": "llama-3.1-8b-instant", 
        "messages": aiMessages 
      };

      var options = {
        "method": "post",
        "headers": { 
          "Authorization": "Bearer " + GROQ_API_KEY, 
          "Content-Type": "application/json" 
        },
        "payload": JSON.stringify(payload),
        "muteHttpExceptions": true
      };

      var response = UrlFetchApp.fetch(apiUrl, options);
      var result = JSON.parse(response.getContentText());

      if (result.error) return { error: "AI API Error: " + result.error.message };
      
      return { success: true, reply: result.choices[0].message.content };
    }

    return { error: "Unknown action requested." };
    
  } catch(err) { 
    return { error: err.toString() }; 
  }
}

// ------------------------------------------
// 3. DATABASE HELPER FUNCTIONS
// ------------------------------------------

function getRooms() { 
  return JSON.parse(PropertiesService.getScriptProperties().getProperty("rooms") || '["general"]'); 
}

function createRoom(name) { 
  var r = getRooms(); 
  if(!r.includes(name)) r.push(name); 
  PropertiesService.getScriptProperties().setProperty("rooms", JSON.stringify(r)); 
}

function getMessages(room) {
  var prop = PropertiesService.getScriptProperties();
  var index = Number(prop.getProperty("room_" + room + "_index") || 1);
  var msgs = [];
  
  for(let i = 1; i <= index; i++) { 
    var raw = prop.getProperty("room_" + room + "_" + i); 
    if(raw) msgs = msgs.concat(JSON.parse(raw)); 
  }
  
  return msgs.slice(-100); 
}

function sendMessage(room, email, text) {
  var prop = PropertiesService.getScriptProperties();
  var index = Number(prop.getProperty("room_" + room + "_index") || 1);
  var key = "room_" + room + "_" + index;
  var msgs = JSON.parse(prop.getProperty(key) || "[]");
  
  if(msgs.length >= 100) { 
    index++; 
    prop.setProperty("room_" + room + "_index", index); 
    key = "room_" + room + "_" + index; 
    msgs = []; 
  }
  
  msgs.push({ 
    id: Utilities.getUuid(), 
    user: email, 
    text: text, 
    time: Date.now() 
  });
  
  prop.setProperty(key, JSON.stringify(msgs)); 
}

function deleteMessage(room, id) {
  var prop = PropertiesService.getScriptProperties();
  var index = Number(prop.getProperty("room_" + room + "_index") || 1);
  
  for(let i = 1; i <= index; i++) { 
    var key = "room_" + room + "_" + i; 
    var msgs = JSON.parse(prop.getProperty(key) || "[]"); 
    var newMsgs = msgs.filter(m => m.id !== id); 
    
    if(newMsgs.length !== msgs.length) { 
      prop.setProperty(key, JSON.stringify(newMsgs)); 
      return; 
    } 
  } 
}
```
3. Replace ```const ADMINS=``` with your email and replace ```GROQ_API_KEY``` with your own as well. Finally add your domain to the list of ```ALLOWED_ORIGINS```
4. Next, press Deploy then input ```deploy as me (your email here)``` and ```Anyone```
5. Then, copy the URL and you are good to go!


# My Guide :)

## Heading 2

~~1234~~ 

_Italics_

**Bold Text** 
 
```Kotlin
// code snippet

 var language = "Kotlin "
 val len = language.trim().length()
```
### Links

https://kotlinlang.org/

you can find a similar repo [here](https://github.com/Ericgacoki/Markdown_Files)

or

[Click here](https://kotlinlang.org/  "Normal link")

or

[Referal link](README.md "Refers to a file within the repo")


### Username @mentions
@Ericgacoki

### Emojis

basecampy    :basecampy:

goberserk    :goberserk:

shipit       :shipit:

### Flags

pirate :pirate_flag:

crossed :crossed_flags:

checkered :checkered_flag:

rainbow :rainbow_flag:

**countries** 

Kenya :kenya:

Malaysia :malaysia:

## see more emojis [from github emoji API](https://github.com/ikatyang/emoji-cheat-sheet/blob/master/README.md#github-custom-emoji "Github Emojis")

### image

![Image](https://image.shutterstock.com/image-photo/image-260nw-1418646482.jpg "sample image")

### keywords

`for` (item `in` items){ `if` ...}

### tables

|                H1      |   H2  |    H3    |
|                  ---   |  ---  |   ---    |
| :technologist: Android |  Is   |    Fun   |
|                Go      |  Try  |It :wink: |

### Blockquotes

> this is a block quote

### colors

- ![#f03c15](https://via.placeholder.com/15/f03c15/000000?text=+) `#f03c15`
- ![#c5f015](https://via.placeholder.com/15/c5f015/000000?text=+) `#c5f015`
- ![#1589F0](https://via.placeholder.com/15/1589F0/000000?text=+) `#1589F0`


```diff
 colored text
 
- text in red
+ text in green
! text in orange
# text in gray
```

### Lists

1. list one 
2. list two
3. list three

- bullet1
- bullet2

4. list four

hr line1 .
___


hr line2 .

***