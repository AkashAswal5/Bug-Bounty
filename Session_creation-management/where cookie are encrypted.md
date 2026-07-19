# Why is `encrypt => false`?

Another excellent observation.

You saw:

```
'encrypt' => false,
```

Many people think this means

> "The cookie isn't encrypted."

That is **not** what it means.

It means:

> **Laravel will not encrypt the session payload when using certain session drivers.**

But the **cookie itself** still passes through Laravel's cookie encryption middleware.

That's why your cookie looks like:

```
{
 iv,
 value,
 mac
}
```

because of
```
EncryptCookies Middleware
```

---
## Find where cookies are encrypted

Open:

```
app/Http/Kernel.php
```

You'll find middleware similar to:

```
EncryptCookies::class
```

or

```
vendor/laravel/framework/src/
Illuminate/Cookie/
Middleware/
EncryptCookies.php
```

That middleware encrypts outgoing cookies.

---

# Where should you study next?

Since you want to understand BookStack deeply, I'd recommend following this exact path through the source:

```
BookStack
│
├── routes/web.php
│
├── app/Access/Controllers/LoginController.php
│
├── Auth::attempt()
│
└── vendor/laravel/framework
      │
      ├── Illuminate/Auth/SessionGuard.php
      ├── Illuminate/Session/Store.php
      ├── Illuminate/Session/Middleware/StartSession.php
      ├── Illuminate/Cookie/Middleware/EncryptCookies.php
      └── Illuminate/Encryption/Encrypter.php
```

Each layer has a specific responsibility:

- **LoginController**: accepts the login request.
- **SessionGuard**: authenticates the user and logs them in.
- **StartSession**: loads and saves session data for each request.
- **EncryptCookies**: encrypts and decrypts cookie values.
- **Encrypter**: performs the AES encryption and MAC generation.

---

## I recommend one more exercise

Since you have the full source code, pick one `bookstack_session` cookie from your browser, then:

1. Show me the output of:

```
ls -l storage/framework/sessions
```

2. Show me one session file:

```
cat storage/framework/sessions/<filename>
```

(If there's sensitive data, you can redact it.)

Then I'll explain **every field** in that session file, show you how it maps to the `bookstack_session` cookie, and trace exactly how BookStack retrieves the logged-in user on each request. This is one of the best ways to understand Laravel session management from a product security perspective.


