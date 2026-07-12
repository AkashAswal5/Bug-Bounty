I think Discourse is one of the **best open-source targets** for someone who wants to become a strong web application security researcher. You get a mature codebase, an active development team, and the ability to read the implementation while testing. Their security policy encourages responsible disclosure through their vulnerability reporting process. ([GitHub][1])

One important update: Discourse announced in 2026 that it **paused bounty rewards**, although it still accepts responsible vulnerability reports and continues to work with HackerOne on vulnerability disclosure. So you should verify the current program status before expecting a monetary reward. ([Discourse][2])

---

# 30-Day Discourse Bug Bounty Roadmap

The goal of these 30 days is **not to find a bug immediately**.

The goal is to become an expert on one application.

Think like this:

```
Understand
      ↓
Map
      ↓
Read Code
      ↓
Make Hypotheses
      ↓
Break Assumptions
      ↓
Report
```

---

# Week 1 — Learn the Application

Don't hack.

Just become a power user.

## Day 1

* Install Discourse locally
* Create 3 users
* Create 1 admin
* Learn directory structure
* Run in development mode

Read:
```
README
docs/
config/
```

---

## Day 2

Authentication

Test

* Register
* Login
* Logout
* Forgot password
* Email verification
* Session management

Capture every request.

Questions

```
Cookies?

CSRF?

JWT?

Headers?

Tokens?

Rate limiting?
```

---

## Day 3

Profiles

Explore

* Avatar
* Username
* Bio
* Preferences
* Custom fields

Now search code

```
User
UserUpdater
UsersController
Avatar
```

Read 200 lines only.

---

## Day 4

Topics

Create

Edit

Delete

Pin

Close

Archive

Now locate

```
TopicsController

TopicGuardian

TopicCreator
```

---

## Day 5

Posts

Read

Reply

Edit

Delete

Recover

Revision history

Search

```
Post

PostRevisor

PostCreator
```

---

## Day 6

Messaging

* DM
* Group DM
* Notifications
* Mentions

Read code

```
MessageBus

Notifications

Guardian
```

---

## Day 7

Review

Create a notebook

```
Authentication

Users

Posts

Topics

Messages

Permissions
```

Draw every relationship.

---

# Week 2 — Authorization

This week is all about **who is allowed to do what**.

---

## Day 8

User roles

```
Admin

Moderator

Trust Level 0

Trust Level 1

Trust Level 2

Trust Level 3

Trust Level 4
```

Test everything.

---

## Day 9

IDOR

Try changing

```
Topic ID

Post ID

User ID

Message ID

Upload ID
```

Questions

```
Can another user access this?

Can another user edit?

Delete?

Download?
```

---

## Day 10

API

Find every API endpoint.

Look at

```
config/routes.rb
```

Map everything.

---

## Day 11

Authorization

Read

```
Guardian
```

This file is one of the most important.

Questions

```
Where does authorization happen?

What assumptions exist?
```

---

## Day 12

Admin panel

Test

* Site settings
* Categories
* Users
* Groups

Can any parameter be changed?

---

## Day 13

Search

Search

```
current_user

admin?

staff?

guardian
```

These are gold mines.

---

## Day 14

Review notes

Create attack surface diagram.

---

# Week 3 — Inputs

Every input is suspicious.

---

## Day 15

Uploads

Images

PDF

ZIP

SVG

HTML

Test

* MIME
* Extension
* Content-Type
* Polyglot files

Read upload pipeline.

---

## Day 16

Markdown

Test

```
Markdown

HTML

BBCode

Emoji

Links

Images

Tables
```

Look for XSS.

---

## Day 17

Composer

Read preview code.

Questions

```
Preview server-side?

Client-side?

Sanitization?

Escaping?
```

---

## Day 18

Search feature

Test

```
Wildcards

Unicode

Quotes

Long strings
```

Read search implementation.

---

## Day 19

Notifications

Can you

```
Trigger others?

Spam?

Replay?

Guess IDs?
```

---

## Day 20

Invites

Invite links

Expiration

Reuse

Guessing

---

## Day 21

Review

Write

```
Inputs

Validation

Sanitization

Authorization
```

---

# Week 4 — Think Like a Hunter

Now stop reading code first.

Think.

Then confirm in code.

---

## Day 22

Race Conditions

Test

```
Like twice

Delete twice

Redeem twice

Accept twice
```

---

## Day 23

Business Logic

Questions

```
Can I bypass workflow?

Skip payment?

Skip verification?

Skip confirmation?
```

---

## Day 24

API Fuzzing

Write a small script.

Change

```
IDs

Headers

Methods

JSON

Parameters
```

---

## Day 25

Permission Review

Read every

```
Guardian.can_*

Guardian.ensure_*
```

Ask

```
What if this check is missing?
```

---

## Day 26

Read recent security fixes.

Study:

* the vulnerable code
* the patch
* why the fix works

---

## Day 27

Find TODOs

Search

```
TODO

FIXME

HACK

SECURITY

temporary

later
```

Developers often leave clues.

---

## Day 28

Git history

Look for commits with

```
Security

Escape

Sanitize

Permission

Auth

XSS

CSRF
```

---

## Day 29

Retest every feature.

Many bugs appear only after you understand the application.

---

## Day 30

Write a complete attack surface document.

Include

```
Authentication

Authorization

Uploads

Markdown

Messages

Notifications

API

Admin

Search

Permissions
```

If you can explain the entire application from memory, you're ready to move beyond beginner-level testing.

---

# Daily Schedule (2–3 hours)

* **20 min**: Use one feature normally.
* **40 min**: Intercept and modify requests with Burp Suite.
* **40 min**: Trace the corresponding backend code.
* **20–40 min**: Test hypotheses and record observations.

---

# Previous Discourse Vulnerabilities to Study

Most valid reports are **not publicly disclosed** because Discourse follows coordinated disclosure. However, there are several excellent public sources to learn from:

1. **Security advisories and security commits** in the Discourse repository. These show the actual patches after issues are fixed. ([GitHub][1])
2. **GitHub Security Advisories (GHSA)** for Discourse and its dependencies.
3. **CVE entries** affecting Discourse (often XSS, authorization, or dependency-related).
4. Commits whose message contains:

   * `SECURITY`
   * `XSS`
   * `sanitize`
   * `permission`
   * `escape`
   * `auth`
   * `csrf`

Studying the *before* and *after* versions of these fixes is one of the fastest ways to learn how real vulnerabilities are introduced and patched.

---

# What Bug Classes Should You Focus On?

For Discourse, I'd prioritize these in roughly this order:

| Priority | Bug Class                    | Learn From                                                      |
| -------- | ---------------------------- | --------------------------------------------------------------- |
| ⭐⭐⭐⭐⭐    | Broken Access Control / IDOR | Guardian checks, user roles                                     |
| ⭐⭐⭐⭐⭐    | Stored & Reflected XSS       | Markdown, uploads, previews                                     |
| ⭐⭐⭐⭐     | CSRF                         | State-changing endpoints                                        |
| ⭐⭐⭐⭐     | Business Logic               | Moderation, invitations, trust levels                           |
| ⭐⭐⭐⭐     | File Upload Issues           | Avatar and attachment handling                                  |
| ⭐⭐⭐      | Race Conditions              | Likes, edits, invitations                                       |
| ⭐⭐⭐      | API Authorization            | JSON endpoints                                                  |
| ⭐⭐       | SSRF                         | Imports, webhooks, integrations                                 |
| ⭐⭐       | Cache Poisoning              | Public content and caching                                      |
| ⭐        | SQL Injection                | Less likely due to Rails ORM, but understand query construction |

---

## One More Recommendation

Since you're serious about bug bounty and already studying web penetration testing, C++, Bash scripting, and Security+, I'd recommend treating this like a real research project.

Create a notebook (Markdown or Obsidian) with one page for every feature:

* **How it works**
* **HTTP requests**
* **Important backend files**
* **Trust boundaries**
* **Authorization checks**
* **Interesting parameters**
* **Attack ideas**
* **Code snippets**
* **Potential vulnerabilities**
* **Retest notes**

After 30 days, you'll have your own **Discourse Attack Playbook**, which is much more valuable than randomly scanning dozens of targets.

If you want to take this even further, I can also design a **90-day "Code-Driven Bug Hunting" roadmap** where you'll learn to discover vulnerabilities by reading the source code first and then proving them with live testing—the same workflow many experienced researchers use.

[1]: https://github.com/discourse/discourse/security?utm_source=chatgpt.com "Security Overview · discourse/discourse · GitHub"
[2]: https://blog.discourse.org/2026/04/discourse-is-not-going-closed-source/?utm_source=chatgpt.com "Discourse is Not Going Closed Source"

