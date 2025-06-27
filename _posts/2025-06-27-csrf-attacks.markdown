---
layout: post
title: "CSRF (Cross-Site Request Forgery)"
date: 2025-06-27 20:00:00 +12:49
comments: true
excerpt: "CSRF (Cross-Site Request Forgery) is a type of attack that tricks a logged-in user into making unintended requests. Let's explore how it works and how to defend against it."
---

# Understanding CSRF Attacks: Cross-Site Request Forgery

## What is a CSRF Attack?

**CSRF (Cross-Site Request Forgery)** is a type of web security vulnerability that allows an attacker to trick a logged-in user into unknowingly submitting malicious requests to a web application where they are authenticated.

When a user logs into a website, they often receive authentication tokens (such as JWT tokens), which are stored in the browser (typically as cookies). These tokens enable persistent sessions without requiring the user to log in repeatedly.

However, if the user clicks on a **malicious link** or visits a malicious website, the attacker can craft a hidden form or script that sends an unauthorized request to the original website. Since the user's browser includes the stored authentication token, the server treats the request as legitimate.

The scary part? The user has **no idea** this action took place.

---

## Example of a CSRF Attack

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
    <h1>You've won a prize!</h1>
    <form action="https://popularwebsite.com/user/update-email" method="POST">
      <input type="hidden" name="email" value="hacker@example.com">
    </form>
  </body>
</html>
```

If the user is **logged in** to `popularwebsite.com` and visits this page, their browser will automatically include the JWT token stored in the cookie. As a result, the server will accept the request and update the user’s email address without their knowledge.

---

## What makes the CSRF possible

For a CSRF attack to be successful, three conditions must be met by the attacker

- There is a sensitive action within the application that the attacker wants to trigger. This action might be a privileged action or any action on user specific data.
    
- The user must be logged into the target website for the malicious request to succeed. Then only whatever action is induced by the attacker will successful.
    
- The required parameters for the request must be known. If a parameter such as a CSRF token is unknown or unpredictable, the attack is likely to fail.

---

## Defenses against CSRF 


#### CSRF Token

- `CSRF Token` is a token generated at the server side and sent to the user while accessing the form. Whenever the form is submitted or any request is sent using methods like POST, PUT, DELETE, etc., the CSRF token is sent along with the request parameters.
    
- You might wonder where this token is stored. Typically, when we access the website, the server generates a CSRF token and sends it to the client. This token is often stored in a hidden form field or sometimes in a cookie.

```html

<form name="change-email-form" action="/my-account/change-email" method="POST"> <label>Email</label> <input required type="email" name="email" value="example@normal-website.com"> <input required type="hidden" name="csrf" value="50FaWgdOhi9M9wyna8taR1k3ODOR8d6u"> <button class='button' type='submit'> Update email </button> </form>

```

- If it's in the HTML form, the attacker **cannot access** that token due to **Same-Origin Policy** — which blocks JavaScript from another domain from reading your page's DOM.
    
- But sometimes the CSRF token is stored in cookies. Here, there's a **chance for vulnerability**. Because cookies are stored in the browser, and if proper restrictions are not applied, they might be accessed by other websites if the browser sends them automatically.
    
- To protect the token in the cookie, we can mark it as `HttpOnly`. This makes the cookie **not accessible to JavaScript**, preventing attackers from reading it even if XSS is possible.
    
- But still, there is another issue. Cookies are **automatically added to every request** going to the origin, even if the request is triggered from a different origin. So if the attacker creates a fake form or script pointing to the target website, the browser might **still include** the cookies (including the CSRF token or session ID).
    
- To prevent this, we also set the cookie property `SameSite=Strict`. This tells the browser to only include cookies in requests that come from **the same origin**. So even if the attacker builds a custom website and tries to trigger a request, the browser **won’t send the cookies**, making the CSRF fail.

## Situations that fails CSRF Validation

Now we will see the situations that lets attackers do the action even with the CSRF token. And these situation was created by the developer without knowing which leads to make the site vulnerable and allows the CSRF attacks. Lets go through each one.

#### Inappropriately using the GET method to data updation or creation.

- CSRF token validation usually depends on the HTTP method. Tokens are **not typically validated** for GET requests because these are expected to be safe and idempotent. Yeah why we need because we create the CSRF token and send in the GET request usually if the request page contains form.
- If the developers uses the appropriate methods to handle request which leads to a vulnerability here. For example, if a website allows users to update their email address using a GET request instead of PATCH or POST, it introduces a vulnerability because CSRF tokens are not generally validated on GET requests. Now we know that CSRF token are usually not validated at the GET method. Which leads to can bypass the method and exploit the attack.

#### Validating the CSRF token based on its presence

- In some applications the CSRF token will be validated based on its presence. This allows attackers to send requests without a valid CSRF token, and the server may still process them because it only checks whether the token is present, not whether it’s correct or tied to the session.
- By sending the request without CSRF token by following makes attack success

```
POST /email/change HTTP/1.1 
Host: iranwebsite.com 
Content-Type: application/x-www-form-urlencoded 
Content-Length: 25 
Cookie: session=2yQIDcpia41WrATfjPqvm9tOkDvkMvLm 

email=trumpwithb2bommer.com
```

- Since the application `iranwebsite.com` cannot validate the CSRF token if the token is doesn't present in the request. The request will fulfilled and let the attacker allows his request.

#### Global token pool issue

- This is another issue where CSRF tokens are stored in a global pool instead of being tied to individual user sessions.For example, if all users' CSRF tokens are stored in a single shared store and the application only checks for token existence, any user’s token might be accepted, leading to vulnerabilities.
- If attacker creates a account and using his csrf token he can access the site since the token is stores in the global pool not user separate session.

#### When CSRF token is  tie to the non-Session cookie

- This is a situation where CSRF token is not tied to session means lets take a example that user A has logged in he gets a session id and csrf token. If server only checks the csrf token and allow the request which allows the attack
- See If a User A logged in and got session ID and csrf token. Now the attacker logged into same website with his credentials now he got his session ID and csrf token now if he replaces the user A csrf token with his token and session ID of user's A. As the server checks the csrf token and session separately makes this like he is valid user but the attacker uses the csrf token and session of different users. 
- This one usually happen when we use different framework or library for session ID and csrf token.
- However, setting a cookie with the `HttpOnly` flag means JavaScript cannot access it. This helps mitigate the risk of token theft via XSS. But it will happens when the sites under same domain

#### CSRF token is simply duplicated in a cookie

- So in this the token is generated in the server and send to the user as cookie and also in the form in hidden. So when we requested the server just check the cookie and form csrf token is same or not.
- If not reject the request. Here is the catch if the attacker get his csrf token and placed in the cookie and form. The changing of cookie value only happens when cookie has property `sameOrigin` as none or the website is in subdomain. Which makes easy to set the cookie value.
- Happening of the attack very rare but have vulnerability allow access based on the property application subdomain.