
![vul](../../assets/OAuth/vul.png)


---

## 🎯 **What is OAuth?** {#what-is-oauth}

---

### ⚡ **Simple Definition**

> **OAuth** (Open Authorization) = A framework that allows apps to get limited access to user accounts on other apps **WITHOUT** sharing passwords!

### 🏗️ **Real-Life Example**

```
Example: "Login with Google" on Spotify

👤 You visit Spotify
    ↓
🌐 Spotify asks: "Login with Google?"
    ↓
🔐 Goes to Google login page
    ↓
👤 You login to Google
    ↓
🔐 Google asks: "Allow Spotify to see your profile?"
    ↓
👤 You click "Allow"
    ↓
🔐 Google sends OK to Spotify
    ↓
🌐 Spotify logs you in
```


---

### 👥 **3 Main Players in OAuth**

---

**1️⃣ Resource Owner (You)**

```
👤 The user who owns the data
├── Has account on Google/Facebook/etc.
├── Controls what apps can access
└── Says YES or NO to access requests
```

**2️⃣ Client App (Spotify/App)**

```
🌐 The app wanting your data
├── Wants to access your info
├── Starts the OAuth process
└── Uses tokens to get your data

Examples:
• Spotify wanting your Google profile
• App requesting Facebook contacts
• Website using GitHub login
```

**3️⃣ OAuth Provider**

```
🔐 The big company hosting your data
├── 🛡️ Authorization Server
│   ├── Handles login
│   ├── Creates codes
│   └── Makes tokens
│
└── 📦 Resource Server
    ├── Stores your data
    ├── Checks tokens
    └── Gives data when asked

Examples:
• Google
• Facebook
• GitHub
• Microsoft
```

### 💡 **Common Uses**

```
🔐 Single Sign-On (SSO)
├── "Login with Google"
├── "Continue with Facebook"
└── "Sign in with GitHub"

📱 App Connections
├── "Give Spotify your Facebook friends"
├── "Allow app to read emails"
└── "Connect Instagram to posting tool"

🔗 Service Links
├── Link GitHub to Slack
├── Connect Google Drive to Dropbox
└── Sync calendar with task app
```

### ⚖️ **Good vs Bad**

```
✅ BENEFITS:
├── 🔒 You don't share passwords
├── 🎯 Control what apps can access
├── 🔄 Easy to remove access
├── 🌐 Same system everywhere
└── 📱 Works on phone and computer

❌ PROBLEMS:
├── 🔴 Rules can be confusing
├── 🔴 Many security options (easy to miss)
├── 🔴 Developers make mistakes
├── 🔴 Many moving parts
└── 🔴 Often misunderstood
```

### 📅 **History**

```
📅 2006 → Twitter talks about open auth
📅 2007 → OAuth 1.0 discussions start
📅 2010 → OAuth 1.0 published
📅 2012 → OAuth 2.0 released (current)
📅 2014 → OpenID Connect released
📅 2016 → More security rules added
📅 2019-2025 → Many security holes found
📅 2025 → OAuth 2.1 coming (more secure)
```

---

## 🆚 **OAuth 1.0 vs 2.0** {#oauth-versions}

---

### 📝 **OAuth 1.0a (Old)**

```
📜 Released: 2010
⚠️  Status: Old (don't use new)

Key Points:
├── Needs complex signatures
├── Hard to build right
├── Very secure by design
└── Every request must be signed

Why Not Used:
❌ Too hard for developers
❌ Bad for mobile/apps
❌ Complex to do right
```

### 🆕 **OAuth 2.0 (Current)**

```
📜 Released: 2012
✅ Status: What everyone uses now

Key Points:
├── Easier to build
├── Multiple ways to work
├── Uses simple bearer tokens
├── Great for mobile/web
└── Flexible framework

Security Warning:
⚠️  Easier = Some security trade-offs
⚠️  Needs HTTPS always
⚠️  Many optional security features
⚠️  Different apps do it differently
```

### 🔄 **Quick Comparison**

```
┌──────────────────┬──────────────┬──────────────┐
│                  │  OAuth 1.0a  │  OAuth 2.0   │
├──────────────────┼──────────────┼──────────────┤
│ Signatures       │ Required     │ Optional     │
│ HTTPS Required   │ No           │ Yes          │
│ Complexity       │ High         │ Low          │
│ Token Types      │ 1 type       │ Many types   │
│ Mobile Support   │ Bad          │ Excellent    │
│ Security         │ Built-in     │ Up to you    │
│ Easy to Build    │ No           │ Yes          │
└──────────────────┴──────────────┴──────────────┘
```

---

## 🔬 **How OAuth 2.0 Works** {#how-oauth-works}

---

### 🔄 **Basic Flow (8 Steps)**

```
Step 1: User Clicks Login
👤 You click "Login with Google" on App

Step 2: Go to Google
🌐 App → 👤 → Goes to Google

Step 3: You Login
👤 You enter Google username/password
🔐 Google checks it's really you

Step 4: You Say Yes
🔐 Google asks: "Allow App to see your profile?"
👤 You click "Allow"

Step 5: Google Sends Code
🔐 Google → 👤 → Returns to App with code

Step 6: App Gets Token
🌐 App → 🔐 Google → Trades code for token

Step 7: App Gets Your Data
🌐 App → 📦 Google → Uses token to get your info

Step 8: You're Logged In
🌐 App → 👤 → Creates session, you're in!
```

---

### 🗝️ **Key Parts of OAuth**

---

#### **🔑 Access Token**

```
What: Short-lived key for API access
Format: Random string or JWT
Lasts: 1-60 minutes usually
Use: To get protected data

Example:
Bearer eyJhbGciOiJIUzI1NiIs...
```

#### **🔄 Refresh Token**

```
What: Long-lived key to get new access tokens
Format: Random string
Lasts: Days to years
Use: Get new tokens without login

⚠️  Security: VERY important - keep safe!
```

#### **📜 Authorization Code**

```
What: Temp code traded for token
Format: Random string
Lasts: 30-60 seconds
Use: Middle step in some flows

Example:
4/P7q7W91a-oMsCeLvIaQm6bTrgtp7
```

#### **🎫 ID Token**

```
What: Proof of who you are (JWT)
Format: JWT
Use: Says "this is really this user"
Has: User ID, email, name, etc.

Example:
eyJhbGciOiJSUzI1NiIsImtpZCI6IjFlOWdkazcifQ...
```

---

### 📋 **Important Parameters**

**Login Request:**

```
client_id       → Which app is asking
redirect_uri    → Where to send user after
response_type   → What to return (code/token)
scope           → What permissions needed
state           → Security token (stop CSRF)
nonce           → Stop replay attacks
code_challenge  → Extra security (PKCE)
```

**Example Login URL:**

```http
https://accounts.google.com/o/oauth2/v2/auth?
  client_id=123456789.apps.googleusercontent.com&
  redirect_uri=https://myapp.com/callback&
  response_type=code&
  scope=openid profile email&
  state=af0ifjsldkj&
  nonce=n-0S6_WzA2Mj
```

**Token Request:**

```
grant_type      → What type (code, refresh)
code            → Code received
client_id       → App ID
client_secret   → App secret (if has one)
redirect_uri    → Must match
refresh_token   → For new tokens
```

**Example Token Request:**

```http
POST /token
Content-Type: application/x-www-form-urlencoded

grant_type=authorization_code&
code=4/P7q7W91a-oMsCeLvIaQm6bTrgtp7&
client_id=123456789.apps.googleusercontent.com&
client_secret=gX_rR3RvA7wke47dwxMvSg_E&
redirect_uri=https://myapp.com/callback
```

**Token Response:**

```json
{
  "access_token": "ya29.a0AfH6SMBx...",
  "expires_in": 3600,
  "token_type": "Bearer",
  "scope": "openid profile email",
  "refresh_token": "1//0gOj4ZfYRMG5C...",
  "id_token": "eyJhbGciOiJSUzI1NiIs..."
}
```

### 🎯 **Common Scopes**

```
What: Defines what app can do
Format: Words separated by spaces

Common Ones:
├── openid        → For login
├── profile       → Basic info (name, photo)
├── email         → Email address
├── read:user     → Read user data
├── write:repo    → Write to repos
├── admin:org     → Admin access
└── offline_access→ Get refresh token

Example:
scope=openid profile email
```

---

## 🎭 **OAuth 2.0 Grant Types** {#grant-types}

---

### 🟢 **TYPE 1: Authorization Code (Most Secure)**

```
✅ Most secure way
✅ Back-channel used
✅ Best for web apps
✅ Needs app secret
✅ Token never in browser
```

**How It Works:**

```
1. User → App
   Click "Login with Google"

2. App → User (Browser)
   Go to: Google?client_id=APP&response_type=code

3. User → Google
   Login & say "Allow"

4. Google → User (Browser)
   Return to: app.com/callback?code=XYZ

5. App → Google (Server)
   POST: trade code for token

6. Google → App
   Give: access_token, refresh_token

7. App → Google API
   Use token to get user data

8. App → User
   Create session, user logged in
```

**Security:**

```
✅ Code used only once
✅ Code lasts 30-60 seconds
✅ Needs app secret
✅ Token via secure channel
✅ Token never in browser
✅ state stops CSRF
```

**Best For:**

```
✅ Websites with backend
✅ Apps that can keep secrets
✅ When security is #1
✅ Traditional web apps
```

---

### 🟡 **TYPE 2: Implicit (Deprecated)**

```
⚠️  DON'T USE (OAuth 2.1 removes it)
⚠️  Less secure
⚠️  Token in browser
⚠️  No app secret
```

**Flow:**

```
1. User → App
   Click "Login"

2. App → User
   Go to: Google?response_type=token

3. User → Google
   Login & allow

4. Google → User
   Return to: app.com#access_token=XYZ

5. App (JavaScript) gets token from URL

6. App → Google API
   Use token to get data
```

**Problems:**

```
❌ Token in URL (browser sees it)
❌ Browser history has token
❌ No app check
❌ Token in JavaScript
❌ XSS attacks can steal it
```

**Why Not Use:**

```
🔴 OAuth 2.1 removes it
🔴 Use Authorization Code + PKCE instead
🔴 Too many security risks
```

---

### 🔵 **TYPE 3: Authorization Code + PKCE**

```
✅ Secure for public apps
✅ No app secret needed
✅ Best for mobile/SPA
✅ Standard in OAuth 2.1
✅ Stops code stealing
```

**PKCE Parts:**

```
🔑 code_verifier  → Random string (43-128 chars)
🔒 code_challenge → SHA256(code_verifier)
📊 method         → "S256" or "plain"
```

**Flow:**

```
1. App makes random code_verifier
   verifier = "dBjftJeZ4CVP-mB92K27uhbUJU1p1r_wW1gFWFOEjXk"

2. App makes code_challenge
   challenge = SHA256(verifier) then base64

3. App → User
   Go to: Google?code_challenge=E9Melhoa...

4. User → Google
   Login & allow

5. Google → User
   Return to: app.com?code=XYZ

6. App → Google
   POST: code=XYZ & code_verifier=original

7. Google checks:
   SHA256(verifier) == challenge?

8. If match, gives token
```

**Security Benefits:**

```
✅ Stops code stealing
✅ No app secret needed
✅ Even if code stolen, can't use without verifier
✅ Good for public apps
```

**Best For:**

```
✅ Mobile apps
✅ Single-page apps (SPA)
✅ Desktop apps
✅ Any app without secret
✅ RECOMMENDED FOR ALL in OAuth 2.1
```

---

### 🟣 **TYPE 4: Password Credentials (Avoid)**

```
⚠️  User gives password to app
⚠️  Only for very trusted apps
⚠️  NOT recommended
⚠️  Legacy use only
```

**Flow:**

```
1. User → App
   Give username/password

2. App → Google
   POST: username=user&password=pass&grant_type=password

3. Google → App
   Give access_token
```

**Why Avoid:**

```
❌ App sees your password
❌ Password exposed to app
❌ No "allow" screen
❌ Hard to remove access
❌ Security anti-pattern
```

**Only Use If:**

```
🔸 Moving from old system to OAuth
🔸 Same company's mobile app
🔸 Very trusted only
🔸 No other option

Better: ✅ Authorization Code + PKCE
```

---

### 🟠 **TYPE 5: Client Credentials**

```
✅ App-to-app only
✅ No user involved
✅ Service accounts
✅ Backend talking to backend
```

**Flow:**

```
1. Service A → Google
   POST: client_id=SERVICE&client_secret=SECRET&grant_type=client_credentials

2. Google → Service A
   Give access_token

3. Service A → Service B
   Use token to call API
```

**Use Cases:**

```
✅ Microservices talking
✅ Background jobs
✅ Scheduled tasks
✅ Server-to-server calls
✅ Automated processes
```

---

### 🔄 **TYPE 6: Refresh Token**

```
✅ Get new token without login
✅ Long-lived token
✅ More sensitive than access token
✅ Can be removed
```

**Flow:**

```
1. Token expires
   App tries API, gets 401 error

2. App → Google
   POST: grant_type=refresh_token&refresh_token=OLD_TOKEN

3. Google → App
   Give new access_token, maybe new refresh_token

4. App uses new token
```

**Security Notes:**

```
✅ Refresh tokens last long
✅ Store VERY securely
✅ User can remove them
✅ New refresh token each time is good
✅ Tie to specific app
```

---

## 📊 **Grant Type Comparison** {#grant-comparison}

---

```
┌──────────────────┬──────────┬──────────┬──────────┬──────────┐
│                  │Auth Code │Implicit  │  PKCE    │ Client   │
│                  │          │(Old)     │          │ Creds    │
├──────────────────┼──────────┼──────────┼──────────┼──────────┤
│User Involved     │   Yes    │   Yes    │   Yes    │    No    │
│App Secret Needed │ Required │    No    │    No    │ Required │
│Security Level    │   High   │   Low    │   High   │  Medium  │
│Token in Browser  │    No    │   Yes    │    No    │    No    │
│Best For          │  Web     │   N/A    │ Mobile/  │  App-to- │
│                  │  Apps    │          │   SPA    │   App    │
│OAuth 2.1 Status  │   ✅    │    ❌    │    ✅    │    ✅    │
└──────────────────┴──────────┴──────────┴──────────┴──────────┘
```

---

## 🔓 **Using OAuth for Login** {#oauth-authentication}

---

### 🎯 **How Apps Use OAuth for Login**

```
Original OAuth Purpose:
├── Let apps access data
└── NOT for proving who you are

Common Use:
├── "Sign in with Google"
├── "Login with Facebook"
└── "Continue with GitHub"
```

**How It's Done:**

```
Step 1: Ask for basic info
🌐 App asks to see your profile

Step 2: OAuth happens
🔐 App gets access token

Step 3: Get your info
📦 App asks for: /userinfo

Step 4: App logs you in
🌐 App uses email to know who you are
🌐 Creates session for you

Result: You're logged in via OAuth
```

### ⚠️ **Problems**

```
🔴 OAuth wasn't made for login
🔴 No standard way to get user info
🔴 Token ≠ proof of identity
🔴 No standard user ID
🔴 No login event info
🔴 Token might be reused

Solution: OpenID Connect (OIDC)
```

---

## 🆔 **OpenID Connect (OIDC)** {#openid-connect}

---

### 📝 **What is OIDC?**

```
OpenID Connect = Login layer on top of OAuth 2.0

Purpose:
✅ Standard login via OAuth
✅ Get user identity info
✅ Add login-specific features
✅ Fix OAuth-for-login problems
```

### 🆚 **OIDC vs OAuth 2.0**

```
OAuth 2.0:
├── Access control
├── "What can you access?"
└── Gives access tokens

OpenID Connect:
├── Login system
├── "Who are you?"
├── Gives access + ID tokens
└── Standard user info
```

### 🔑 **Key OIDC Features**

#### **1️⃣ ID Token (JWT)**

```js
What: JWT with who-you-are info
Format: JWT (header.payload.signature)
Use: Prove user login
Has: User identity + login details

Example:
eyJhbGciOiJSUzI1NiIsImtpZCI6IjFlOWdkazcifQ.
eyJpc3MiOiJodHRwczovL2FjY291bnRzLmdvb2dsZS5jb20i...

What's Inside:
{
  "iss": "https://accounts.google.com",
  "sub": "110169484474386276334",
  "email": "user@example.com",
  "email_verified": true,
  "name": "John Doe",
  "picture": "https://lh3.googleusercontent.com/..."
}
```

**ID Token Info:**

```js
iss (issuer)     → Who made token
sub (subject)    → User's unique ID
aud (audience)   → Who token is for
exp (expires)    → When token ends
iat (made at)    → When token made
auth_time        → When user logged in
nonce           → Stop replay attacks
```

#### **2️⃣ /userinfo Endpoint**

```js
Where: /userinfo
Use: Get detailed user info
How: GET with token

Request:
GET /userinfo
Authorization: Bearer TOKEN

Response:
{
  "sub": "248289761001",
  "name": "Jane Doe",
  "email": "jane.doe@example.com",
  "email_verified": true
}
```

#### **3️⃣ Standard Scopes**

```js
openid          → Required for OIDC
profile         → Basic profile (name, photo)
email           → Email address
address         → Home address
phone           → Phone number
offline_access  → Get refresh token

Example:
scope=openid profile email
```

#### **4️⃣ Discovery Endpoint**

```js
Where: /.well-known/openid-configuration
Use: Find OIDC info automatically

Example URL:
https://accounts.google.com/.well-known/openid-configuration

Returns:
{
  "issuer": "https://accounts.google.com",
  "authorization_endpoint": ".../auth",
  "token_endpoint": ".../token",
  "userinfo_endpoint": ".../userinfo"
}
```

---

## 🔍 **Finding OAuth/OIDC** {#identifying-oauth}

---

### 🎯 **How to Spot OAuth**

**See It:**

```
✅ "Sign in with Google" button
✅ "Continue with Facebook" option
✅ "Login with GitHub" link
✅ Social login buttons
✅ SSO choices
```

**Technical Signs:**

```js
1. Use Burp Suite to watch traffic

2. Look for /authorize endpoint

3. Check URL for:
   ├── client_id
   ├── redirect_uri
   ├── response_type (code/token/id_token)
   ├── scope
   └── state

Example:
GET /oauth/authorize?
    client_id=123456&
    redirect_uri=https://app.com/callback&
    response_type=code&
    scope=read:user&
    state=random123
```

**OIDC Signs:**

```
✅ scope has "openid"
✅ response_type has "id_token"
✅ /.well-known/openid-configuration exists
✅ ID token in response
✅ /userinfo endpoint there

Example OIDC:
GET /authorize?
    client_id=123456&
    redirect_uri=https://app.com/callback&
    response_type=code&
    scope=openid profile email&
    state=random123&
    nonce=abc789
```

---

### 🔎 **Finding Steps**

**Step 1: Who's the Provider?**

```
Check login page host:
├── accounts.google.com → Google
├── www.facebook.com → Facebook
├── github.com → GitHub
├── login.microsoftonline.com → Microsoft
```

**Step 2: Map the Flow**

```
1. Watch login request
   ├── Note all parameters
   ├── Check response_type
   └── Find grant type

2. Watch callback
   ├── Check for code
   ├── Check for token in URL
   └── Note state

3. Watch token exchange
   ├── POST to /token
   ├── Check for client_secret
   └── Note grant_type

4. Watch API calls
   ├── Look for Authorization header
   ├── Note Bearer tokens
   └── See what data is fetched
```

**Step 3: Check for OIDC**

```
Visit: https://provider.com/.well-known/openid-configuration

If exists → Using OIDC

Check for:
✅ Endpoints list
✅ Features supported
✅ Token types
✅ Grant types
```

---

### 🔭 **Keep Learning**

```
📚 OAuth 2.1 Specification (when available)
📚 OpenID Connect Core 1.0
📚 OWASP Authentication Cheat Sheet
📚 RFC 6749 (OAuth 2.0)
📚 RFC 6750 (Bearer Tokens)
📚 RFC 7636 (PKCE)
📚 RFC 8705 (OAuth 2.0 Mutual TLS)
📚 RFC 9449 (OAuth 2.0 Demonstrating Proof of Possession)
```

### 🛠️ **Tools for Testing**

```
🔧 Burp Suite Professional (OAuth/OpenID modules)
🔧 OAuth 2.0 Playground (Google)
🔧 Postman OAuth 2.0 flows
🔧 OWASP ZAP (with OAuth add-on)
🔧 mitmproxy
🔧 jwt.io (JWT debugging)
🔧 CanaryTokens for token leakage detection
🔧 Custom scripts for automation
```

### 📞 **Get Help**

```
🆘 OAuth 2.0 Security Best Practices (IETF)
🆘 OWASP Web Security Testing Guide
🆘 Security StackExchange
🆘 Your organization's security team
🆘 Professional penetration testers
🆘 OAuth.net community
```

---

**✨ Remember: OAuth is powerful but complex. Always prioritize security over convenience. Test thoroughly, implement carefully, and monitor continuously. ✨**

---
