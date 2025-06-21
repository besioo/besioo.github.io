---
title: "How I Found an IDOR on a Fintech App"
date: 2025-06-21 17:00:00 +0300
categories: [writeups]
tags: [bugbounty, idor, web]
---

In this writeup, I describe how I found an **Insecure Direct Object Reference (IDOR)** vulnerability in a financial web application.

## 🔍 Recon

I was browsing user profile features when I noticed predictable ID patterns in the URL...

## 💥 Exploitation

By changing `user_id=1234` to `user_id=1235`, I was able to access another user's data without authorization.

## ✅ Impact

- Personal data exposure
- Broken access control

## 🎯 Resolution

Reported via Bugcrowd and received a bounty.

---

*Written by Ahmed Basiony (thebee0x)*
