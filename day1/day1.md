
---
#### Authentication
Test
* Register
* Login
* Logout
* Forgot password
* Email verification
* Session management

---

login: https://try.discourse.org/login

create [[multiple CSRF Token]]: https://id.discourse.com/session/csrf 

---
request: https://id.discourse.com/manifest.webmanifest
Response: 
	- user can : send/accept file type   [ .jpg, .jpeg, .png, .gif, .heic, .heif, .webp, .avif, .svg, .jxl ]

```response
{
  "name": "Discourse ID",
  "short_name": "Discourse ID",
  "description": "Discourse ID is a single sign-on service that lets you use one account across multiple Discourse forums. Instead of creating separate accounts for each community, you can sign up once here and use those credentials anywhere that supports Discourse ID.",
  "display": "standalone",
  "start_url": "/",
  "background_color": "#ffffff",
  "theme_color": "#333333",
  "icons": [
    {
      "src": "https://d1v8sgyxuzxz1b.cloudfront.net/optimized/1X/b33be9538df3547fcf9d1a51a4637d77392ac6f9_2_512x512.png",
      "sizes": "512x512",
      "type": "image/png"
    },
    {
      "src": "https://d1v8sgyxuzxz1b.cloudfront.net/optimized/1X/b33be9538df3547fcf9d1a51a4637d77392ac6f9_2_512x512.png",
      "sizes": "512x512",
      "type": "image/png",
      "purpose": "maskable"
    }
  ],
  "share_target": {
    "action": "/share-target",
    "method": "POST",
    "enctype": "multipart/form-data",
    "params": {
      "title": "title",
      "text": "text",
      "url": "url",
      "files": [
        {
          "name": "files",
          "accept": [
            ".jpg",
            ".jpeg",
            ".png",
            ".gif",
            ".heic",
            ".heif",
            ".webp",
            ".avif",
            ".svg",
            ".jxl"
          ]
        }
      ]
    }
  },
  "shortcuts": [
    {
      "name": "Create a new topic",
      "short_name": "New Topic",
      "url": "/new-topic"
    },
    {
      "name": "Inbox",
      "short_name": "Inbox",
      "url": "/my/messages"
    },
    {
      "name": "Bookmarks",
      "short_name": "Bookmarks",
      "url": "/my/activity/bookmarks"
    },
    {
      "name": "Top",
      "short_name": "Top",
      "url": "/top"
    }
  ]
}


```

---
