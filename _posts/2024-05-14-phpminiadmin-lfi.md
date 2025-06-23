---
layout: post
title: "phpminiadmin.php LFI Leads to FULL DB Access"
date: 2024-05-14
categories: [bugbounty-writeup, web-security, exploitation]
tags: [LFI, MySQL, phpminiadmin, db-exploit]
---


Websites have different options to manage their DB through GUI. One of the most common tools is phpMyAdmin.

In this case, the target used another option — a standalone script called `phpminiadmin.php`.

---

## What is phpminiadmin.php?

**phpminiadmin** is a lightweight alternative to phpMyAdmin. It’s a single PHP script for quick and easy MySQL access via web interface.

GitHub repo: [https://github.com/osalabs/phpminiadmin](https://github.com/osalabs/phpminiadmin)

You can also detect it using this [Nuclei template](https://github.com/projectdiscovery/nuclei-templates/blob/fc23e21aabb1f48b86b8dc8a444beeabe492a668/http/exposed-panels/phpminiadmin-panel.yaml).

---

## Bug Story

A Bugcrowd public program had a subdomain, `partner.test.com`, running a PHP app.

While fuzzing, I discovered a publicly accessible `phpminiadmin.php` file.

Opening it in the browser, I saw something like this:

![phpminiadmin](https://cdn-images-1.medium.com/max/1024/1*M1co4zuwHpvnUz0xjFB4dQ.png)

It showed a login panel. Interestingly, there were **advanced options** to input a custom SQL host.

---

## SSRF → Public MySQL

I first tested if the script connected to arbitrary DB hosts. I entered a **Burp Collaborator URL** as the MySQL host and received a pingback → SSRF confirmed.

I then registered a **free SQL hosting** account from:

- [https://freemysqlhosting.net](https://freemysqlhosting.net)

Got credentials via email, added them to the form — and it worked! I was inside:

![connected](https://cdn-images-1.medium.com/max/1024/1*WwXmGCwy2WjiudEweb6sSg.png)

---

## Now It’s LFI Time

MySQL supports reading files from the server using `LOAD DATA`.

I created a table named `test`, then ran:

```sql
LOAD DATA LOCAL INFILE "/etc/passwd" INTO TABLE test FIELDS TERMINATED BY '\n';
```

Then:

```sql
SELECT * FROM test;
```

I successfully dumped the `/etc/passwd` file 🎉

---

## Reading Source Code

I wanted more impact. I noticed the script linked to `phpinfo`, which I accessed after login.

From `phpinfo`, I got `DOCUMENT_ROOT` — the base directory for the web app.

I then tried to read source files using:

```sql
LOAD DATA LOCAL INFILE "DOCUMENT_ROOT/admin/config.php" INTO TABLE test FIELDS TERMINATED BY '\n';
```

And this gave me the contents of `config.php`:

![config.php](https://cdn-images-1.medium.com/max/1024/1*Wfook0X-BkewUHGeT1GYGg.png)

It referenced another file: `config_prod.php` containing **MySQL credentials**:

![config_prod](https://cdn-images-1.medium.com/max/1024/1*N9KWpKkXbTfCgmyVTz32ug.png)

I reused the script again to connect with these creds — and got **full database access**:

![DB](https://cdn-images-1.medium.com/max/1024/1*1fOXHsTksmompgOeGRlcZQ.png)

---

### Final Thoughts

- If you see `phpminiadmin.php` in the wild, **investigate it**.
- If custom SQL hosts are allowed, SSRF → LFI → DB takeover is possible.
- Always sanitize and lock down these admin tools.

Thanks for reading!
