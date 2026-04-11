
![vul](../../assets/Access-Control/vul.png)


---
---

## **1. 🔍 What is Access Control ?**

---

👉 Access control is the application of **constraints on who or what is authorized to perform actions or access resources** in a web application. 🔹 It answers one question: _"Are you ALLOWED to do this?"_ 🔹 Without it — any authenticated user can access any resource.

> ⚠️ **OWASP Top 10:** Broken Access Control has been **#1** since 2021 — the most common and impactful web application vulnerability. 💬 94% of applications were tested for broken access control, with an average incidence rate of 3.81%.

---

## **2. 🧾 The Three Pillars — Auth, Session, Access Control**

---

|Pillar|Question It Answers|Example|
|---|---|---|
|🔑 **Authentication**|Who are you?|"Is this really Carlos?"|
|🎟️ **Session Management**|Are all your requests from the same person?|"Is this cookie tied to Carlos's login?"|
|🔒 **Access Control**|Are you ALLOWED to do this?|"Can Carlos access /admin/deleteUser?"|

> 💬 All three must work together. **Access control fails** when authentication succeeds but permissions are never checked!

---

## **3. 🗂️ Types of Access Control**

---

|Type|What It Restricts|Example|
|---|---|---|
|⬆️ **Vertical**|Functionality by user role|Admin can delete users, regular user cannot|
|↔️ **Horizontal**|Resources by user ownership|User A cannot see User B's bank account|
|🔄 **Context-Dependent**|Actions by application state|Cannot modify cart after payment is complete|

### ⬆️ Vertical Access Controls

- Restrict **sensitive functionality** to specific user types
- Example: Admin panel accessible only by admins, not regular users
- Implements: **separation of duties**, **least privilege** principles

### ↔️ Horizontal Access Controls

- Restrict access to **specific resources** even within the same role
- Example: Bank user can only see their OWN accounts
- Same privilege level — different data ownership

### 🔄 Context-Dependent Access Controls

- Restrict actions based on **application state** or **workflow order**
- Example: Shopping cart locked after payment is made
- Prevents users from performing actions in the **wrong order**

---

## **4. 🌊 How Vulnerabilities Arise — Root Causes**

---

**1️⃣ Developers Assume Authentication = Authorization**

- App checks "are you logged in?" but never checks "should YOU access THIS?"
- Any authenticated user can access any resource they can reference
- The critical missing question: **"Does this user OWN or have permission for this resource?"**

**2️⃣ Access Checks Only on the Frontend**

- UI hides admin buttons from regular users → but the endpoint still works!
- Direct URL access bypasses all client-side controls
- Security by obscurity is NOT access control

**3️⃣ Parameter-Based Trust**

- App trusts user-controlled values like `?admin=true` or `?role=1`
- Attacker modifies these → gains elevated access instantly
- Cookies, hidden fields, and query strings are ALL user-controllable

**4️⃣ Scattered Access Control Logic**

- Different developers implement checks inconsistently across the codebase
- Multi-step processes where some steps are protected but others aren't
- Access controls added per-feature rather than enforced centrally

**5️⃣ Platform Misconfiguration**

- Access rules defined at the platform layer (e.g., load balancer, WAF)
- App framework reads override headers like `X-Original-URL` → bypasses platform controls
- Method-based restrictions bypass via HTTP verb switching

🧩 Listed in **OWASP Top 10 — A01:2021 — Broken Access Control** (#1 since 2021)

---

## **5. 💥 Impact of Broken Access Control**

- 👤 **Account Takeover** → access any user's account and data
- 🔑 **Privilege Escalation** → regular user → admin → full app control
- 📂 **Sensitive Data Exposure** → PII, financial records, medical data, private messages
- ✏️ **Unauthorized Modification** → update, delete, or corrupt other users' data
- 💸 **Financial Fraud** → access billing, modify prices, transfer funds
- ⚖️ **Compliance Violations** → GDPR, HIPAA, PCI-DSS — breach of millions of records
- 🏢 **Business Logic Abuse** → purchase items at wrong price, access restricted features

---

## **6. 🗺️ Attack Methodology — Step-by-Step ( 5 Phases )**

---

```
PHASE 1: 🔍 Map — Identify all endpoints, parameters, and roles
              ↓
PHASE 2: 🔑 Authenticate — Get credentials for 2+ accounts at different privilege levels
              ↓
PHASE 3: 🧪 Test Vertical — Can low-priv user access high-priv functionality?
              ↓
PHASE 4: 🔁 Test Horizontal — Can User A access User B's resources?
              ↓
PHASE 5: 🔗 Chain — Combine horizontal → vertical for maximum impact
```

**Essential Setup Before Testing:**

- Create at minimum **2 user accounts**: one normal user + one admin (if possible)
- Capture all admin requests in Burp while logged in as admin
- Replay those requests while logged in as normal user → does it work?
- Use Burp's **Autorize** extension to automate this comparison

---

## **7. 🔍 Finding the Access Control Attack Surface**

---

👉 Before attacking — map everything that should have access restrictions.

|Area|What to Test|
|---|---|
|🔒 Admin pages|`/admin`, `/administrator`, `/admin-panel`, `/manage`|
|👤 User profiles|`/account?id=X`, `/user/X`, `/profile/X`|
|📁 Files & exports|`/download?file=X`, `/export?id=X`, `/invoice/X.pdf`|
|⚙️ Functions by role|Delete, update, promote, ban, view-all|
|🔄 Multi-step flows|Order confirmation, admin actions with multiple steps|
|🌐 API endpoints|`/api/v1/users/X`, `/api/admin/X`|
|📋 Hidden fields|`<input type="hidden" name="admin" value="false">`|
|🍪 Cookies & tokens|`role=user`, `isAdmin=false` in cookie values|
|🤖 robots.txt|Often reveals hidden admin and sensitive paths|
|📄 JavaScript source|Admin URLs sometimes leaked in JS files for role-based UI|

---

## **8. ⬆️ Vertical Privilege Escalation — All Techniques**

---

👉 Vertical privilege escalation = **low-privilege user gains access to high-privilege functionality**.

---

### 🚪 Technique 1 — Unprotected Admin Functionality (Direct URL Access)

👉 Admin pages exist but have **no access control** — anyone who knows the URL can reach them!

**Step 1 — Find the admin URL:**

- Try: `/admin`, `/administrator`, `/admin-panel`, `/management`, `/panel`
- Check `robots.txt` — often lists disallowed (sensitive!) paths:

```
https://target.com/robots.txt
→ Disallow: /admin
→ Disallow: /administrator-panel-yb556   ← "secret" URL still listed here!
```

- Check JavaScript source code — admin URLs often leaked for role-based UI:

```javascript
var isAdmin = false;
if (isAdmin) {
    var adminPanelTag = document.createElement('a');
    adminPanelTag.setAttribute('href', '/administrator-panel-yb556');
    // ← URL visible in JS source to ALL users regardless of role!
}
```

**Step 2 — Access directly without admin role:**

```
GET /administrator-panel-yb556 HTTP/1.1
Cookie: session=USER_SESSION_TOKEN   ← normal user session!
→ If admin panel loads → NO access control → vertical privilege escalation!
```

> 💡 Security by obscurity is NOT access control. A hard-to-guess URL still works once found!

---

### 🏷️ Technique 2 — Parameter-Based Access Control Bypass

👉 Application determines role from a **user-controllable parameter** — attacker changes it!

**Common Vulnerable Patterns:**

|Vulnerable Parameter|Attack|
|---|---|
|`?admin=false`|Change to `?admin=true`|
|`?role=user`|Change to `?role=admin` or `?role=1`|
|`?isAdmin=0`|Change to `?isAdmin=1`|
|Cookie: `role=user`|Change to `role=admin`|
|Hidden field: `<input name="admin" value="false">`|Change value to `true`|
|JWT claim: `"role": "user"`|Change to `"role": "admin"`|

**How to Test with Burp:**

1. Login as normal user → capture requests to admin endpoints
2. In Repeater → modify the role/admin parameter
3. Send request → does it work? → **Vertical privilege escalation!**

> ⚠️ NEVER trust client-side values for access control decisions. These are all attacker-controlled!

---

### 🔧 Technique 3 — Platform Misconfiguration (Header Override)

👉 Platform-level access rules restrict URLs, but the app honors override headers!

**The Problem:**

```
Platform rule:  DENY POST /admin/deleteUser for managers
                ↑ This rule works on the URL and method

But the application reads X-Original-URL for routing!
```

**Attack — Override URL via custom header:**

```http
POST / HTTP/1.1
Host: target.com
X-Original-URL: /admin/deleteUser
Content-Type: application/x-www-form-urlencoded

username=carlos
```

- Platform sees: `POST /` → no restriction applies!
- Application reads: `X-Original-URL: /admin/deleteUser` → routes to admin function!
- Access control bypassed!

**Headers to Try:**

```
X-Original-URL: /admin/deleteUser
X-Rewrite-URL: /admin/deleteUser
X-Override-URL: /admin/deleteUser
```

**Attack — HTTP Method Switching:**

```http
# Platform denies: POST /admin/deleteUser
# Try: GET /admin/deleteUser  (or POSTX, HEAD, etc.)
GET /admin/deleteUser?username=carlos HTTP/1.1
```

- Some apps perform the SAME action regardless of HTTP method
- If platform only blocks POST → GET performs the same delete!

---

### 🔀 Technique 4 — URL-Matching Discrepancies

👉 Access control checks one path format but app accepts another — they're different to the check but same to the router!

**Path variations to try:**

```
Protected:   /admin/deleteUser
Try:         /ADMIN/deleteUser         ← case variation
Try:         /admin/deleteUser/        ← trailing slash
Try:         /admin/deleteUser.anything ← Spring useSuffixPatternMatch!
Try:         /admin;/deleteUser        ← semicolon injection (Java/Spring)
Try:         /admin/./deleteUser       ← dot segment
Try:         /%61dmin/deleteUser       ← URL-encoded first letter
```

> 💡 Spring Framework: `useSuffixPatternMatch=true` (default pre-5.3) maps `/admin/deleteUser.json` → same as `/admin/deleteUser`!

---

## **9. 🔗 Horizontal Privilege Escalation**

---

👉 Horizontal privilege escalation = **accessing resources of OTHER users at the SAME privilege level**.

### 🔑 The Core Pattern — ID / Parameter Tampering

```
Normal:   https://target.com/myaccount?id=123   ← your account
Attack:   https://target.com/myaccount?id=124   ← someone else's account!
```

**What to Tamper With:**

|Location|Example|
|---|---|
|URL parameter|`?id=123` → `?id=124`|
|POST body parameter|`user_id=123` → `user_id=124`|
|Cookie value|`account=alice` → `account=carlos`|
|JSON body|`{"account_id": 123}` → `{"account_id": 124}`|
|File path|`/download/file1.txt` → `/download/file2.txt`|
|API path segment|`/api/users/123/orders` → `/api/users/124/orders`|

**ID Types and Strategies:**

|ID Format|Strategy|
|---|---|
|Sequential numbers `1, 2, 3`|Increment/decrement to enumerate|
|GUIDs/UUIDs|Look for leaks in messages, reviews, error responses|
|Hashed values|Find source — often leaked in other pages|
|Encoded values|Base64/URL decode → modify → re-encode|
|Custom formats|`user_123` → `user_124`, `invoice-2024-001` → `invoice-2024-002`|

> 💡 Even GUID-based IDs can be exploited! GUIDs often appear in: user messages, reviews, API responses, other users' profiles, email headers, shared documents.

---

### 📡 When Redirect Still Leaks Data

👉 App detects unauthorized access → sends 302 redirect → BUT response BODY still contains the data!

**Test:**

1. Access `/myaccount?id=456` (not your ID)
2. If response is 302 redirect → **don't follow it in Burp!**
3. Check the BODY of the 302 response → sensitive data may still be there!

```http
HTTP/1.1 302 Found
Location: /login

{"username":"carlos","email":"carlos@example.com","creditCard":"4111..."}
← DATA STILL IN BODY DESPITE REDIRECT!
```

**Burp Tip:** In Repeater → uncheck "Follow redirects" → examine raw response body!

---

## **10. 🔁 Horizontal → Vertical Privilege Escalation Chain**

---

👉 Start with horizontal IDOR → target an **admin account** → gain vertical escalation!

**The Attack Chain:**

```
Step 1: Find IDOR vulnerability on user account pages
        GET /myaccount?id=123 → 200 OK (your account)

Step 2: Enumerate other user IDs
        GET /myaccount?id=456 → Returns another user's account

Step 3: Target the ADMIN account
        GET /myaccount?id=1   ← Admin is often ID 1!
        → Returns admin's account page

Step 4: Extract admin credentials from account page
        → Password visible in page? (some apps show it)
        → Password reset option? → reset admin password → full control!
        → API key visible? → use it for admin API calls!
        → Direct admin functionality accessible from account page?
```

**Real Example Flow:**

```
1. Normal user's account page shows:
   {"id": 123, "username": "alice", "email": "alice@example.com"}

2. Admin's account page (via IDOR) shows:
   {"id": 1, "username": "administrator", "password": "admin123", "role": "admin"}
   ← Horizontal IDOR → exposes admin password → vertical escalation!
```

> 🔥 This chain is one of the highest-impact findings in bug bounty — full admin takeover from a simple ID change!

---

## **11. 🎯 Insecure Direct Object References ( IDOR )**

---

👉 IDOR = application uses **user-supplied input to access database objects or files directly** — without checking if the user owns or is authorized to access that object.

> 💡 IDOR is a **subcategory** of access control vulnerabilities, popularized by OWASP 2007 Top Ten. It's the most commonly found vulnerability in SaaS applications.

---

### 🗄️ IDOR on Database Objects

```
Vulnerable: https://target.com/customer_account?customer_number=132355
Attack:     https://target.com/customer_account?customer_number=132356
→ Returns another customer's account data — PII, billing, history!
```

**Vulnerable Code (PHP):**

```php
$user_id = $_GET['id'];
$query = "SELECT * FROM users WHERE id = $user_id";
// Missing: if ($current_user->id !== $user_id) { deny(); }
```

**Vulnerable Code (Node.js):**

```javascript
app.get('/api/profile/:id', (req, res) => {
    const profile = db.getProfile(req.params.id);
    // Missing: authorization check!
    res.json(profile);
});
```

---

### 📁 IDOR on Static Files

👉 Sensitive files stored server-side with predictable names → directly accessible!

```
Vulnerable: https://target.com/static/12144.txt  (chat transcript)
Attack:     https://target.com/static/12145.txt
            https://target.com/static/12143.txt
→ Read other users' private chat transcripts!
```

**Common Vulnerable File Patterns:**

|Pattern|Attack|
|---|---|
|`/download?file=invoice_123.pdf`|Change filename → enumerate invoices|
|`/receipts/2024/001.pdf`|Increment number|
|`/exports/user_456_data.csv`|Change user ID in filename|
|`/attachments/msg_789.jpg`|Enumerate message attachments|

---

### 🔍 Where to Find IDOR Opportunities

- → URL path parameters: `/users/123`, `/orders/456`
- → Query parameters: `?id=`, `?user=`, `?account=`, `?doc=`, `?file=`
- → POST body fields: `{"user_id": 123}`, `{"invoice_id": 456}`
- → Cookie values: `user_id=123`, `account=alice`
- → HTTP headers: custom headers with user/session identifiers
- → API endpoints: REST `/api/v1/resource/{id}`, GraphQL arguments
- → File downloads: `/download?token=abc` → token maps to specific user's file
- → Export functions: PDF generation, CSV exports — often missed in security review!

> 💡 Secondary features (export, PDF generation, notification settings) are where IDOR is most often found in production — they're rushed during development!

---

## **12. 🧩 Multi-Step Process Bypass**

---

👉 App enforces access control on **some steps** of a multi-step flow — but not all steps!

### The Vulnerable Pattern

```
Admin function to update user details:
  Step 1: Load form with user details    ← Access control ✅
  Step 2: Submit changes                 ← Access control ✅
  Step 3: Review and confirm             ← Access control ❌ MISSING!
```

**Attack — Skip directly to the unprotected step:**

- The app assumes you must have completed protected steps 1 and 2 to reach step 3
- **The assumption is wrong** — attacker directly submits step 3!

```http
POST /admin/updateUser/confirm HTTP/1.1
Cookie: session=NORMAL_USER_SESSION

action=upgrade&username=carlos&role=admin
→ Step 3 executed with normal user session → privilege escalation!
```

**How to Test:**

1. Complete the full admin flow while logged in as admin → capture all step requests
2. Log out → log in as normal user
3. Replay **step 3 only** (the confirmation/final step) with normal user session
4. If it executes → multi-step bypass!

> ⚠️ Common in: admin user management, order processing, checkout flows, approval workflows

---

## **13. 📨 Referer-Based Access Control Bypass**

---

👉 App checks the `Referer` header to determine if a request "came from" an authorized page — but `Referer` is fully attacker-controllable!

### The Vulnerable Pattern

```
/admin              ← Rigorous access control ✅
/admin/deleteUser   ← Only checks if Referer contains /admin ❌
```

If `Referer: https://target.com/admin` → request is allowed!

**Attack — Forge the Referer header:**

```http
GET /admin/deleteUser?username=carlos HTTP/1.1
Host: target.com
Referer: https://target.com/admin    ← Forged! Attacker controls this!
Cookie: session=NORMAL_USER_SESSION

→ Sub-admin action executes despite lacking admin privileges!
```

**How to Test:**

1. As admin — visit `/admin/deleteUser` — capture the request (includes legitimate `Referer`)
2. Log in as normal user
3. Replay the request with the same `Referer` header
4. If action executes → Referer-based bypass!

> ⚠️ The `Referer` header is just another HTTP header — completely forgeable by the client. NEVER use it for access control!

---

## **14. 🌍 Location-Based Access Control Bypass**

---

👉 Some apps restrict features by **geographic location** (banking, streaming services, legal compliance) — these can often be bypassed.

**Bypass Methods:**

|Method|How It Works|
|---|---|
|🌐 VPN / Proxy|Route traffic through a server in the allowed region|
|🔄 Web Proxy / Burp|Configure upstream proxy in allowed location|
|🗺️ Client-Side Geolocation Manipulation|App uses JS `navigator.geolocation` → intercept and modify coordinates|
|📍 IP Spoofing via Headers|`X-Forwarded-For: [IP_IN_ALLOWED_COUNTRY]` if app trusts this header|
|🌍 Tor Exit Node|Exit nodes in specific countries|

**Testing:**

- Check if the app uses client-side location (JS geolocation API)
- Check if it uses IP-based detection and trusts `X-Forwarded-For`
- Try: `X-Forwarded-For: 8.8.8.8` (US IP) if testing a US-only feature

---

## **15. 🛠️ Tools for Access Control Testing**

---

|Tool|Purpose|Key Usage|
|---|---|---|
|🧱 **Burp Suite Pro**|Core proxy and testing platform|Capture admin requests → replay as low-priv user|
|🔄 **Autorize (BApp)**|Automated access control testing|Auto-replays all requests with low-priv cookie|
|🔍 **Param Miner (BApp)**|Discover hidden parameters|Find hidden admin/role parameters|
|📊 **Burp Intruder**|Enumerate IDs for IDOR|Increment ID numbers → find accessible records|
|🌊 **ffuf**|Brute force admin URL paths|Discover hidden admin endpoints|
|🗺️ **Burp Comparer**|Compare admin vs user responses|Spot differences in what each role can see|
|📋 **Burp Logger++**|Track all requests per session|Compare two sessions side by side|

---

### 🔄 Autorize Extension — The IDOR Automation Tool

```
HOW AUTORIZE WORKS:
1. Login as admin → configure Autorize with admin session
2. Login as normal user → browse application normally
3. Autorize automatically replays EVERY request with admin cookies
   AND without any cookies (unauthenticated)
4. Flags responses where:
   → Same response with low-priv session = ACCESS CONTROL BYPASS!
   → Same response with no session = UNAUTHENTICATED ACCESS!

SETUP:
  BApp Store → Install "Autorize"
  Autorize tab → "Header(s) to replace" → paste normal user's Cookie header
  Browse as admin → Autorize tests every request with normal user cookies!
  Red = same response (bypass!) | Yellow = different length | Green = properly restricted
```

---

### 📊 Manual Burp Workflow for IDOR Testing

```
STEP 1: Login as admin → browse all admin functions → capture in History
STEP 2: Note all requests with ID parameters (user IDs, order IDs, etc.)
STEP 3: Send interesting requests to Repeater
STEP 4: Change session cookie to normal user's cookie
STEP 5: Also change the ID to another user's ID
STEP 6: Send → does it return data? → IDOR confirmed!

FOR ID ENUMERATION:
  Send request to Intruder
  Mark the ID as payload position: /api/users/§123§/data
  Payload: Numbers → From 1, To 1000, Step 1
  Look for: 200 OK responses with data (not 403/404)
  Grep for: username, email, password, or other sensitive fields
```

---

## **16. 🌍 Real-World CVEs & Bug Bounty Cases**

---

### 💰 PayPal — IDOR on Business Accounts ( $10,500 )

- **Type:** IDOR allowing addition of secondary users to any PayPal business account
- **Impact:** Attacker could add themselves as authorized user to any merchant account
- **Root Cause:** API endpoint accepted account ID without verifying ownership
- **Lesson:** Any action that references another account's ID needs strict ownership verification!

---

### 🏅 HackerOne Certification System — IDOR Deletion ( $12,500 )

- **Type:** IDOR allowing deletion of any user's certification
- **Impact:** Attacker could delete earned certifications belonging to any HackerOne user
- **Root Cause:** DELETE endpoint accepted certification ID without checking the requester owns it
- **Lesson:** Destructive actions (delete, modify) need the strictest ownership checks!

---

### 🇦🇺 Optus Data Breach — IDOR → 10 Million Records ( 2023 )

- **Type:** API IDOR — customer records accessible by enumerating customer IDs
- **Impact:** 9.8 million Australians had their driver's license, email, phone, and address stolen
- **Root Cause:** API endpoint returned customer data by ID with no authentication or authorization check
- **Attack:** Attacker sequentially incremented customer IDs → downloaded all records
- **Lesson:** NEVER expose sequential IDs in APIs without strict authorization checks AND authentication!

---

### 📸 Instagram — IDOR on Private Content ( 2019 )

- **Type:** IDOR enabling viewing of private posts and stories by manipulating user IDs in API requests
- **Impact:** Private content exposed to unauthorized users
- **Root Cause:** API didn't verify the requesting user was permitted to see the target user's private content
- **Lesson:** Privacy settings must be enforced at the API level, not just the UI level!

---

### 🚗 Kia & Hyundai — API Access Control Failure ( June 2024 )

- **Type:** Missing authentication + authorization on vehicle control APIs
- **Impact:** Using only a license plate number, attacker could remotely control vehicle functions
- **Root Cause:** APIs designed for authenticated users were accessible without authentication
- **Attack:** Supply VIN or license plate → API returns vehicle control access
- **Lesson:** APIs must enforce authentication AND authorization — exposed APIs with no auth check = critical risk!

---

### 📁 MOVEit Transfer — Unauthenticated Endpoint ( CVE-2023-34362 )

- **Type:** SQL injection reached via an unauthenticated API endpoint
- **Impact:** Cl0p ransomware group stole data from hundreds of organizations
- **Root Cause:** Critical endpoint had no authentication check → anyone could reach the SQL injection
- **Lesson:** Missing access control on ONE critical endpoint can compromise an entire organization!

---

### ☁️ GitHub — Privilege Escalation in Repositories ( 2022 )

- **Type:** Users could gain higher access levels within repositories without authorization
- **Impact:** Regular users accessed functions restricted to repository administrators
- **Root Cause:** Missing authorization check on repository management endpoint
- **Lesson:** Role checks must be on every individual endpoint — not assumed from context!

---

### 🏥 Healthcare Provider — Patient Records ( 2024 )

- **Type:** IDOR on patient portal — lab results accessible by changing document ID
- **Impact:** Patients could access other patients' lab results, diagnoses, prescriptions
- **Discovery:** Found accidentally by a patient who typed the wrong URL!
- **Lesson:** Healthcare data needs the most rigorous ownership checks — HIPAA violations from IDOR!

---

## **17. 📋 Complete Testing Checklist**

---

### ⬆️ Vertical Privilege Escalation

- → Check `robots.txt` for disallowed admin paths
- → Check JavaScript source for leaked admin URLs
- → Try common admin paths: `/admin`, `/administrator`, `/manage`, `/panel`
- → Try URL case variation: `/ADMIN`, `/Admin`
- → Try trailing slash: `/admin/`
- → Try suffix: `/admin/deleteUser.json` (Spring suffix match)
- → Test parameter manipulation: `?admin=true`, `?role=admin`, `?isAdmin=1`
- → Check hidden form fields: `<input type="hidden">` for role/admin values
- → Check cookies: any `role=`, `admin=`, `isAdmin=` values?
- → Test `X-Original-URL` / `X-Rewrite-URL` headers for platform bypass
- → Try HTTP method switching: POST → GET on restricted endpoints

### ↔️ Horizontal Privilege Escalation

- → Change numeric ID to another user's ID in URL parameters
- → Change numeric ID to another user's ID in POST body
- → Change ID in cookies to another user's value
- → Test with IDs of known users (admin is often ID 1!)
- → Try GUIDs — look for them leaked in other responses, messages, reviews
- → Check if 302 redirect still returns data in body (don't follow redirect!)
- → Enumerate sequential IDs with Burp Intruder

### 🎯 IDOR

- → Test ALL parameters that reference objects (IDs, filenames, tokens)
- → Test in: URL path, query string, POST body, JSON body, cookies, headers
- → Check file download endpoints — predictable filename patterns?
- → Check export / PDF generation functions
- → Check API endpoints: REST, GraphQL, WebSocket
- → Compare: can User A access User B's orders/invoices/messages/files?

### 🧩 Multi-Step Process

- → Map entire multi-step admin flow as admin
- → Identify final confirmation/execution step
- → Replay ONLY the final step with normal user session
- → Does it execute without completing prior authorized steps?

### 📨 Referer-Based

- → Find admin sub-pages that check Referer instead of session/role
- → Forge `Referer: https://target.com/admin` header on requests
- → Test with normal user session + forged Referer

### 🌍 Location-Based

- → Test with VPN/proxy from different regions
- → Check if `X-Forwarded-For` with a different country IP bypasses restriction
- → Check for client-side geolocation that can be intercepted

---

## **18. 📊 Access Control Vulnerability Impact Matrix**

---

|Vulnerability|Auth Needed|Impact|Severity|
|---|---|---|---|
|Unprotected admin URL|None|Full admin access|🔴 Critical|
|Parameter-based bypass `?admin=true`|Any auth|Instant privilege escalation|🔴 Critical|
|X-Original-URL platform bypass|Any auth|Admin function access|🔴 Critical|
|IDOR on user data|Same-level auth|Any user's PII exposure|🔴 Critical|
|IDOR on admin account|Any auth|Admin credential theft → full takeover|🔴 Critical|
|Horizontal → Vertical chain|Any auth|Admin takeover via IDOR|🔴 Critical|
|Multi-step process bypass|Any auth|Admin action without authorization|🔴 Critical|
|IDOR on static files|Any auth|Private file/document exposure|🟠 High|
|Referer-based bypass|Any auth|Admin sub-functions access|🟠 High|
|Method switching bypass|Any auth|Restricted function execution|🟠 High|
|URL-matching discrepancy|Any auth|Access control bypass|🟠 High|
|Location-based bypass|None / VPN|Geo-restricted content access|🟡 Medium|
|Admin URL in robots.txt|None|Path disclosure|🟡 Low-Med|

---
---

## **19. 🛡️ Developer Best Practices**

---

### 🔑 Core Principles

|Principle|Implementation|
|---|---|
|🚫 **Deny by default**|Unless a resource is explicitly public, deny access|
|🔒 **Server-side enforcement**|NEVER trust client-side role/permission values|
|🏛️ **Centralize access control**|One consistent mechanism for the whole application|
|👤 **Ownership checks**|Always verify: does this user OWN this resource?|
|🎯 **Least privilege**|Users get only the minimum access they need|
|📋 **Declare access per resource**|Explicitly define who can access what|
|🧪 **Test thoroughly**|Audit and test access controls — don't assume they work|

---

### 🛠️ Secure Code Patterns

**✅ Safe — Ownership Check in Database Query:**

```js
# Most reliable pattern — ownership enforced at DB level
def get_owned_resource(resource_id: int, user):
    resource = db.query(Resource).filter(
        Resource.id == resource_id,
        Resource.owner_id == user.id   # ← Ownership check IN the query!
    ).first()
    if not resource:
        raise HTTPException(status_code=404)  # Same error — no info leak!
    return resource
```

**❌ Vulnerable — Fetch Then Check:**

```r
// Fetches data first → then checks → but data already loaded!
app.get('/api/profile/:id', async (req, res) => {
    const profile = await db.getProfile(req.params.id); // No auth check!
    res.json(profile); // Returns to anyone!
});
```

**✅ Safe — Role-Based Access Control (RBAC):**

```r
// Middleware that checks role before executing function
function requireAdmin(req, res, next) {
    if (!req.session.userId) return res.status(401).json({ error: 'Not authenticated' });
    const user = getUserFromSession(req.session.userId);
    if (user.role !== 'admin') return res.status(403).json({ error: 'Not authorized' });
    next();
}

app.post('/admin/deleteUser', requireAdmin, deleteUserHandler);
// ← Server-side check! Not a URL parameter or cookie value!
```

**✅ Safe — Session State for 2FA / Multi-Step:**

```r
// ❌ VULNERABLE — trusts client cookie for user identity
res.cookie('account', req.body.username);

// ✅ SAFE — server-side session tracks which user is mid-flow
req.session.pendingUserId = user.id;
req.session.stepCompleted = 1;
// No client-controllable values!
```

**✅ Safe — Avoid User-Controllable Roles:**

```r
// ❌ NEVER DO THIS:
const isAdmin = req.query.admin === 'true';        // URL parameter!
const isAdmin = req.cookies.role === 'admin';       // Cookie!
const isAdmin = req.body.isAdmin;                   // POST body!

// ✅ ALWAYS DO THIS — look up role from server-side data:
const user = await getUserFromDatabase(req.session.userId);
const isAdmin = user.role === 'admin';  // From DB, not client!
```

---
