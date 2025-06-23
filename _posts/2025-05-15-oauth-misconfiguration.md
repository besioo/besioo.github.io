---
layout: post
title: "OAuth Misconfiguration: Hijack User Accounts Without Abusing Redirects"
date: 2025-05-15
categories: [penetration-testing, web-security, bug-hunting]
tags: [OAuth, token, auth, misconfiguration, hijacking]
---

![Intro](https://cdn-images-1.medium.com/max/1024/1*v_MYqiBJRuorhswKKhl_KA.png)


OAuth makes logging in easy, but one small misconfiguration can lead to full account takeover. In this post, I’ll show how skipping a proper token issuer (`iss`) validation allows an attacker to hijack user accounts — **without abusing redirect URIs**. All it takes is a malicious OAuth app and a bit of trust in the wrong token.

---

## What is OAuth?

**OAuth (Open Authorization)** is an open standard that allows applications to obtain limited access to user accounts on an HTTP service, such as Google or Facebook, without exposing the user’s credentials.

It enables secure delegated access using access tokens issued by an authorization server. OAuth is widely used for “Log in with” functionality and API authorization.

- Official spec: [https://oauth.net/](https://oauth.net/)
- RFC: OAuth 2.0 RFC 6749

---

## Sharing Access Token

A common OAuth flow skips the code exchange entirely. Instead, the app asks the user to **share an access token** directly.

This token typically includes limited scopes like:
- user email
- basic profile

This is often enough to authenticate a user by validating their email.

This pattern is common across providers like **Google**, **Facebook**, etc.

---

## In Practice

Example app:  
👉 [https://besioo.github.io/google_auth/index.html](https://besioo.github.io/google_auth/index.html)

The app initiates an OAuth request with:

- `response_type=token`  
- Instead of exchanging `code` → `token` on the server

Example auth URL:

```text
https://accounts.google.com/o/oauth2/v2/auth/oauthchooseaccount?
  client_id=502214687020-0arr62kbk6oaiuq9p687sd4ij6fr9mjf.apps.googleusercontent.com
  &redirect_uri=https%3A%2F%2Fbesioo.github.io%2Fgoogle_auth%2Findex.html
  &response_type=token
  &scope=openid%20email%20profile
  &include_granted_scopes=true
  &prompt=select_account
```

After login, Google redirects back to your app with the access token **in the URL fragment**.

Your app uses this token to call:

```bash
curl 'https://www.googleapis.com/oauth2/v3/userinfo' \
  -H "Authorization: Bearer ya29.a0AW4...."
```

![userinfo](https://cdn-images-1.medium.com/max/1024/1*abbpnqiM8IYQG4-sXto5NA.png)

Then authenticate the user by email from the response.

---

## The Misconfiguration

Let’s say:

- App1 = attacker’s OAuth app
- App2 = your web app using “Login with Google”

If App2 doesn’t **validate** the `aud` or `iss` claims in the access token, it might accept **tokens issued to other clients** (like App1).

That’s bad.

It means the attacker can:

1. Create a malicious OAuth app
2. Trick a user into logging in
3. Steal the access token
4. Use it to log in to **your app**, if you're not validating tokens properly

---

## Real World Exploit

A phishing-like scenario:

1. Attacker sets up OAuth app
2. User logs in & grants access
3. Token gets leaked to attacker
4. Attacker logs in to your app using the stolen token

This works if your app trusts **any valid token** — even if it wasn’t issued for your client ID.

---

## Can It Be One Click?

Yes.

Use this trick: `prompt=none`

Example:

```text
https://accounts.google.com/o/oauth2/v2/auth/oauthchooseaccount?
  client_id=502214687020-0arr62kbk6oaiuq9p687sd4ij6fr9mjf.apps.googleusercontent.com
  &redirect_uri=https%3A%2F%2Fbesioo.github.io%2Fgoogle_auth%2Findex.html
  &response_type=token
  &scope=openid%20email%20profile
  &prompt=none
```

This **bypasses all consent screens** and redirects the user automatically.

If the user is logged in and previously authorized the app — they won’t see any prompt.

If logged in to multiple Google accounts, you can add:

```
authuser=0
```

This will default to the first account silently.

---

## Impact

A simple, non-malicious-looking OAuth app can:

- Steal tokens
- Bypass login protections
- Take over accounts in your app

This is a **single-click** account takeover — no redirect abuse needed.

---

## Conclusion

OAuth is powerful, but misconfigurations can **break security** completely.

🔒 Always validate:
- `aud` — the token’s audience
- `iss` — the token issuer

If you trust any token without checking these fields — you’re vulnerable.

Developers: review your token validation logic.  
Users: be careful when logging in to unknown apps.

---

Feel free to reach out or comment if you want to chat about OAuth security.
