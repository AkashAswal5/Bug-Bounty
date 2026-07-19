This is exactly the kind of deep understanding a **Product Security Engineer** should develop. Let's use BookStack as a case study and walk through **how Laravel creates and manages sessions**, where the code lives, and how to analyze the `bookstack_session` cookie.

---

# 1. Login Flow in BookStack

When you log in:

```text
Browser
    │
POST /login
    │
    ▼
routes/web.php
    │
    ▼
Login Controller
    │
Authenticate credentials
    │
    ▼
Laravel Auth::attempt()
    │
Creates authenticated session
    │
    ▼
Session stored (file/database/redis)
    │
    ▼
Encrypted session ID sent back
    │
Set-Cookie: bookstack_session=...
```

Notice that **BookStack itself does not implement session management from scratch**. It relies on **Laravel's session system**.

---

# 2. Where is this implemented?

## Step 1: Routes

Start here:

```text
routes/web.php
```

Look for:

```php
Route::post('/login', ...);
```

or search:

```bash
grep -R "login" routes app
```

---

## Step 2: Login Controller

Eventually you'll reach something like:

```
app/Access/Controllers/LoginController.php
```

or a similarly named controller (depends on version).

Inside you'll find methods similar to:

```php
public function login(...)
```

or

```php
Auth::attempt(...)
```

---

## Step 3: Laravel Authentication

Once credentials are correct:

```php
Auth::attempt(...)
```

Laravel internally does:

```
User authenticated
↓

Create Session

↓

session()->regenerate()

↓

Store session

↓

Send Cookie
```

---

# 3. Where are sessions configured?

Open

```
config/session.php
```

This file controls almost everything.

You'll see things like:

```php
'driver' => env('SESSION_DRIVER', 'file'),
```
This means:
> "Read the environment variable `SESSION_DRIVER`. If it **doesn't exist**, use `'file'` as the default."

Possible values
```
file
database
redis
cookie
array
```


### How can we verify

Run:
```
php artisan tinker
```

Then:
```
config('session.driver');
```

You should get:
```
=> "file"
```

Also check:
```
config('session.files');
```

Expected output:
```
=> "/path/to/bookstack/storage/framework/sessions"
```




---

Check your `.env`

```
SESSION_DRIVER=file
```

If it's `file`, session files are stored in:

```
storage/framework/sessions/
```

Look there:

```bash
ls storage/framework/sessions
```

You'll likely see many files with random names.

---

# 4. How the Cookie Works

You showed:

```text
Cookie: 
bookstack_session=

eyJpdiI6IkZiNFRPZXJxWmExMFo2dkxyc0Z3OGc9PSIsInZhbHVlIjoiVzY3alRxcWhXcnRLZXd6b2d1LzZER0FVdFducFo3a3JRT3J2VndvTm9OQmd3U3RsMzVFT0F0VGtDZXdONmhCRUR0TGRHcUtnV25ManFrMzM4L1kvUENvbk1tWGtwVUJvR1g2VFdXMDA2QURFQWNjdlpSaHlMYW1tZ2ExS0ZDemgiLCJtYWMiOiI0ZmZjMjkwYWU4NTVlNDQyOGE4YWQ5YjIxNDBkMmUwMTMyYTNhMWRlOTgwYmRmOWQzMjk4MGU1MWU2MTQzZDc2IiwidGFnIjoiIn0%3D
```

It looks like Base64.

decode from `Base64`
```
{
"iv":"Fb4TOerqZa10Z6vLrsFw8g==",

"value":"W67jTqqhWrtKewzogu/6DGAUtWnpZ7krQOrvVwoNoNBgwStl35EOAtTkCewN6hBEDtLdGqKgWnLjqk338/Y/PConMmXkpUBoGX6TWW006ADEAccvZRhyLammga1KFCzh",

"mac":"4ffc290ae855e4428a8ad9b2140d2e0132a3a1de980bdf9d32980e51e6143d76",

"tag":""
}
7
```

If you URL-decode it first, you'll get JSON like:

```json
{
  "iv": "...",
  "value": "...",
  "mac": "...",
  "tag": ""
}
```

Laravel encrypts cookies using its encryption service.

---

## Structure

```json
{
   "iv": "...",
   "value": "...",
   "mac": "...",
   "tag": ""
}
```

---

### iv

Initialization Vector

Example

```
Fb4TOerqZa10Z6vLrsFw8g==
```

Random bytes.

Used for AES encryption.

It should be different every encryption.

---

### value

```
W67jTqqhWrtK...
```

Encrypted session payload.

You cannot read it.

Even if you Base64 decode it,

it is still encrypted.

---

### mac

```
4ffc290ae855...
```

Message Authentication Code.

Purpose:

If someone changes even **one bit** in the cookie,

MAC verification fails.

Laravel rejects it.

---

### tag

Only used by certain encryption modes.

Often empty.

---

# 5. What is actually stored?

Many people think:

```
Cookie

↓

Contains User ID
```

Not true.

Usually:

```
Cookie

↓

Contains encrypted session identifier

↓

Server reads session

↓

Gets User ID
```

The important authentication data lives on the **server**, not in the cookie.

---

# 6. Where is the session stored?

If using:

```
SESSION_DRIVER=file
```

look here:

```
storage/framework/sessions/
```

Open one:

```bash
cat storage/framework/sessions/<filename>
```

You'll see serialized session data.

Often containing values like:

```
login_web

csrf_token

previous_url

user_id
```

---

# 7. Cookie Creation

Laravel eventually does something like:

```
Session Data

↓

serialize()

↓

encrypt()

↓

generate MAC

↓

Base64 encode

↓

Set-Cookie
```

Browser receives

```
bookstack_session=...
```

---

# 8. What happens on every request?

Suppose

```
GET /books
```

Browser sends

```
Cookie:

bookstack_session=...
```

Laravel

```
↓

Decrypt

↓

Verify MAC

↓

Read Session

↓

Find logged-in user

↓

Auth::user()

↓

Continue request
```

---

# 9. Why does the cookie look random?

Because

```
AES Encryption
```

Random IV

Encryption

MAC

Every login generates a fresh encrypted value.

---

# 10. How to trace it in the code

Start in BookStack:

```
routes/web.php
```

↓

Login controller

↓

Auth::attempt()

↓

Then step into Laravel:

```
vendor/laravel/framework/
```

Useful files include:

```
Illuminate/Auth/

Illuminate/Session/

Illuminate/Cookie/

Illuminate/Encryption/
```

Some particularly relevant classes are:

```
SessionGuard.php

Store.php

StartSession.php

EncryptCookies.php

Encrypter.php
```

Follow the flow from the login controller into these framework classes to see how the session is created, regenerated, stored, and how the cookie is encrypted and attached to the response.

---

