Perfect. This is exactly what we need. Now we can connect the **cookie**, the **session file**, and the **BookStack/Laravel code**.

---

# Overall Architecture

```
                Browser
                   │
                   │ Cookie
                   ▼
 bookstack_session=
 eyJpdiI6...
                   │
                   ▼
         Laravel decrypts cookie
                   │
                   ▼
         Session ID
                   │
                   ▼
storage/framework/sessions/
1uhKEY2dThOmwJd6o2Sw2BU9bS8SSMuck9FmfswV
                   │
                   ▼
Serialized PHP session data
                   │
                   ▼
Authenticated User (Eve)
```

Notice something important:

> **The cookie is NOT the session.**

The cookie is simply a **secure reference** that allows Laravel to locate the correct session on the server.

---

# Step 1: Browser sends

```
Cookie:

bookstack_session=
eyJpdiI6IkZiNFRPZXJx...
```

This cookie is encrypted.

Laravel first decrypts it.

---

# Step 2: What is inside this cookie?

After URL decoding and Base64 decoding you get something like

```json
{
   "iv":"...",
   "value":"...",
   "mac":"..."
}
```

It is **still encrypted**.

Laravel uses

```
APP_KEY
```

from your `.env`

```
APP_KEY=base64:4cvDvTdW...
```

to decrypt it.

The decryption code is in:

```
vendor/laravel/framework/src/6Illuminate/Encryption/Encrypter.php
```

---

# Step 3: After decryption

Laravel obtains something like

```
1uhKEY2dThOmwJd6o2Sw2BU9bS8SSMuck9FmfswV
```

Notice that filename?

```
storage/framework/sessions/

1uhKEY2dThOmwJd6o2Sw2BU9bS8SSMuck9FmfswV
```

#### READ session
`cat $/HOME/Bug-bounty-projects/bookstack/storage/framework/sessions/1uhKEY2dThOmwJd6o2Sw2BU9bS8SSMuck9FmfswV`

```
a:5:{s:6:"_token";s:40:"yWrZEB21tbeIBqhiBcT7x0aYtX0FYNENgCwvXfi7";s:6:"_flash";a:2:{s:3:"old";a:0:{}s:3:"new";a:0:{}}s:3:"url";a:0:{}s:9:"_previous";a:2:{s:3:"url";s:69:"http://127.0.0.1:8000/books/book-c-product-roadmap-by-eve/export/html";s:5:"route";N;}s:55:"login_standard_59ba36addc2b2f9401580f014c7f58ea4e30989d";i:12;}
```
That is **the session ID**.

Now Laravel opens that file.

---

# Step 4: Reading the session file

Your session file contains:

```php
a:5:{
```

This means

```
PHP Serialized Array

5 elements
```

Now let's decode each one.

---

## First

```
s:6:"_token";
```

means

```
string

length = 6

"_token"
```

Value

```
yWrZEB21tbeIBqhiBcT7x0aYtX0FYNENgCwvXfi7
```

This is the **CSRF Token**.

Every form contains it.

Example

```
POST /login

_token=yWrZEB...
```

Laravel compares

```
Request token

vs

Session token
```

If different

```
419 Page Expired
```

---

## Second

```
_flash
```

This stores flash messages.

For example

```
Book created.

Password changed.

Login successful.
```

Flash data exists

```
one request

↓

deleted
```

---

## Third

```
url
```

Stores temporary URL information.

Currently empty.

---

## Fourth

```
_previous
```

Contains

```
http://127.0.0.1:8000/books/book-c-product-roadmap-by-eve/export/html
```

Laravel remembers

```
Last Visited URL
```

Useful for

```
redirect()->back();
```

---

## Fifth (MOST IMPORTANT)

```
login_standard_59ba36addc2b2f9401580f014c7f58ea4e30989d

i:12
```

This is the authentication key.

Value

```
12
```

Remember your users?

```
Admin

Guest

Alice

Bob

Charlie

Dave

Eve

Mallory
```

Earlier we saw

```
Eve

id = 12
```

Exactly.

Laravel now knows

```
Logged in user

↓

User ID

12
```

---

# What happens on every request?

Suppose

```
GET /books
```

Laravel performs

```
Receive Cookie
        │
Decrypt Cookie
        │
Session ID
        │
Open Session File
        │
Read login_standard...
        │
User ID = 12
        │
SELECT * FROM users
WHERE id=12
        │
Auth::user()
```

Now

```
Auth::user()->name

↓

Eve
```

---

# Where in the code?

Now you can follow it.

---

## Login

```
routes/web.php
```

↓

```
LoginController
```

↓

```
Auth::attempt()
```

---

## Session starts

```
vendor/laravel/framework

Illuminate\Session

StartSession.php
```

This middleware

```
Loads session

Reads cookie

Opens session file
```

---

## Authentication

```
Illuminate\Auth

SessionGuard.php
```

This class

```
login()

logout()

user()

attempt()
```

It stores

```
login_standard_xxxxx

↓

User ID
```

inside the session.

---

## Session Storage

```
Illuminate\Session

Store.php
```

This class writes

```
storage/framework/sessions/
```

---

## Cookie Encryption

```
Illuminate\Cookie

Middleware

EncryptCookies.php
```

Encrypts

```
bookstack_session
```

before sending it.

---

## Encryption

```
Illuminate\Encryption

Encrypter.php
```

Uses

```
APP_KEY
```

AES

MAC

IV

---

# Why doesn't the cookie contain the user ID directly?

Because that would be insecure.

Instead

```
Cookie

↓

Encrypted Session ID

↓

Server File

↓

User ID
```

If someone steals only the session file name but not the encrypted cookie (or cannot produce a valid encrypted cookie), they cannot simply impersonate the user because Laravel verifies and decrypts the cookie using the server's `APP_KEY`.

---

# As a Product Security Engineer, what should you test?

Understanding this flow tells you what to look for during testing:

1. **Session fixation**: After a successful login, does the session ID change (`session()->regenerate()`) or stay the same?
    
2. **Session invalidation**: After logout, is the old session destroyed, or can the cookie still be reused?
    
3. **Session hijacking resistance**: If you replay a valid `bookstack_session` cookie in another browser, are you authenticated? (This demonstrates why protecting cookies is critical.)
    
4. **Cookie attributes**:
    
    - `HttpOnly` (prevents JavaScript access)
        
    - `Secure` (sent only over HTTPS)
        
    - `SameSite=Lax` (mitigates many CSRF scenarios)
        

These are exactly the kinds of behaviors a product security engineer would verify when reviewing an application's session management.