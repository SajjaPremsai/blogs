---
layout: post
title: "CSRF (Cross-Site Request Forgery)"
date: 2025-06-26 20:00:00 +0530
comments: true
excerpt: "CSRF (Cross-Site Request Forgery) is a type of attack that tricks a logged-in user into making unintended requests. Let's explore how it works and how to defend against it."
---

# ⚖️ Understanding CSRF Attacks: Cross-Site Request Forgery

## 🔒 What is a CSRF Attack?

**CSRF (Cross-Site Request Forgery)** is a type of web security vulnerability that allows an attacker to trick a logged-in user into unknowingly submitting malicious requests to a web application where they are authenticated.

When a user logs into a website, they often receive authentication tokens (such as JWT tokens), which are stored in the browser (typically as cookies). These tokens enable persistent sessions without requiring the user to log in repeatedly.

However, if the user clicks on a **malicious link** or visits a malicious website, the attacker can craft a hidden form or script that sends an unauthorized request to the original website. Since the user's browser includes the stored authentication token, the server treats the request as legitimate.

The scary part? The user has **no idea** this action took place.

---

## 🧪 Example of a CSRF Attack

Let’s say there's a trusted and widely-used website:

```
https://popularwebsite.com
```

To keep users logged in, the site stores JWT tokens in the browser cookies.

Now suppose there's a sensitive endpoint on this site:

```
POST https://popularwebsite.com/user/update-email
```

An attacker creates a malicious webpage like the one below:

```html
<!DOCTYPE html>
<html>
  <body onload="document.forms[0].submit()">
    <h1>You've won a prize! 😈</h1>
    <form action="https://popularwebsite.com/user/update-email" method="POST">
      <input type="hidden" name="email" value="hacker@example.com">
    </form>
  </body>
</html>
```

If the user is **logged in** to `popularwebsite.com` and visits this page, their browser will automatically include the JWT token stored in the cookie. As a result, the server will accept the request and update the user’s email address without their knowledge.

---
