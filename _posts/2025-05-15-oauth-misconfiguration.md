---
layout: post
title: "OAuth Misconfiguration: Hijack User Accounts Without Abusing Redirects"
date: 2025-05-15
categories: [penetration-testing, bug-hunting, web-security]
image: https://cdn-images-1.medium.com/max/1024/1*v_MYqiBJRuorhswKKhl_KA.png
---

![cover](https://cdn-images-1.medium.com/max/1024/1*v_MYqiBJRuorhswKKhl_KA.png)

### بِسْمِ اللَّـهِ الرَّحْمَـٰنِ الرَّحِيمِ

### Intro

OAuth makes logging in easy, but one small misconfiguration can lead to full account takeover. In this post, I’ll show how skipping a proper token issuer (iss) validation allows an attacker to hijack user accounts — **without abusing redirect URIs**. All it takes is a malicious OAuth app and a bit of trust in the wrong token.

### What is OAuth?

**OAuth (Open Authorization)** is an open standard that allows applications to obtain limited access to user accounts on an HTTP service, such as Google or Facebook, without exposing the user’s credentials. It enables secure delegated access using access tokens issued by an authorization server.

### Sharing Access Token

A common behavior when using OAuth is sharing the access token directly without exchanging the code server-side to get an access token.

Example app: [https://besioo.github.io/google_auth/index.html](https://besioo.github.io/google_auth/index.html)

The OAuth request uses the parameter `response_type=token`. The user is redirected to the app with the access token included in the URL fragment, then it's used to call:

```
curl 'https://www.googleapis.com/oauth2/v3/userinfo' -H "Authorization: Bearer ya29.a0AW4..."
```

### Misconfiguration

If App2 does **not** validate the token’s `aud` or `iss`, it may accept tokens from other apps. This allows unauthorized access.

**Exploit scenario:**
- Attacker creates malicious OAuth app
- Tricks users into logging in and sharing tokens
- Uses tokens against vulnerable apps lacking validation

### Real World Exploit

Using `prompt=none`, attackers can silently collect tokens if the user is already logged in. They can also bypass account selection with `authuser=0`.

### Impact

This misconfiguration enables **single-click account takeover**. With the access token, attackers can authenticate on vulnerable apps and hijack accounts.

### Conclusion

OAuth is powerful but easy to misconfigure. Always validate the `iss` and `aud` claims of the token. Be cautious of third-party apps and audit your implementation to avoid these risks.
