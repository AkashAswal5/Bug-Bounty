## Let's follow one login request completely

When you log in as Alice:

### Step 1

Browser sends:

```
POST /login
```

with

```
Email
Password
CSRF Token
```

---

### Step 2

BookStack authenticates:

```
routes/web.php
      ↓
LoginController
      ↓
Auth::attempt()
```

Laravel checks the database:

```
SELECT * FROM users
WHERE email='alice@example.com';
```

Verifies:

```
password_verify()
```

---

### Step 3

Laravel creates a session

Internally something like:

```
session()->regenerate();
```

This creates a **new session ID**.

This is important because it protects against **session fixation attacks**.

---

### Step 4

Laravel writes a file

For example:

```
storage/framework/sessions/

A83Jd92Lm...
```

Inside that file you'll find serialized session data.

---

### Step 5

Laravel encrypts the session cookie

It creates:

```
bookstack_session=
eyJpdiI6...
```

Notice something important.

The cookie is **NOT** the session file.

The cookie points to the session.

Think of it like this:

```
Browser
│
│ Cookie
│
▼
bookstack_session
│
│
▼
Server
│
Session File
│
User ID
CSRF
Flash Data
Previous URL
etc...
```

----