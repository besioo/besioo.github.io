---
layout: post
title: "Recon Methods To Expand Your Attack Surface (Using SSL Certificates)"
date: 2024-11-30
categories: [bugbounty-writeup, cybersecurity, recon]
tags: [ssl, recon, bugbounty, nuclei, subdomains]
---

![SSL Wildcard](https://cdn-images-1.medium.com/max/1024/1*9tfRAXaFHfyDCtIWM-pZcQ.png)

### بِسْمِ اللَّـهِ الرَّحْمَـٰنِ الرَّحِيمِ

## Intro

In recon, a key focus is asset enumeration—collecting as many of a company’s assets as possible. The more assets you gather, the higher your chances of discovering potential vulnerabilities and bugs.

Subdomains are often the gateway to a company’s hidden assets. By thoroughly enumerating subdomains, you increase your chances of discovering valuable entry points and potential vulnerabilities.

## SSL Certificate

SSL certificates secure communication by verifying a server’s identity and include fields like the **Common Name (CN)** and **Subject Alternative Name (SAN)**. The CN usually lists the primary domain, while the SAN can cover multiple domains or subdomains, sometimes using wildcards (e.g., `*.example.com`).

---

## Using SSL Certificate Data to Find More Assets

After performing a normal subdomain enumeration process, I usually use some methods to expand my attack service.

### Discover Wildcard Subdomains

A common method is to pass the discovered subdomains to a subdomain enumeration tool (such as `subfinder`). While this may seem like a good idea, it can often be noisy, leading to unuseful results.

Another method I see as more effective is to use SSL certificate data for those subdomains to discover wildcards within this domain.

**First**, enumerate subdomains for `comcast.com`:

```bash
subfinder -d comcast.com -all -o comcast.com.subs
```

**Second**, grab the SSL certificate data (SAN, CN) with the `nuclei` template `ssl-dns-names`.

```bash
cat comcast.com.subs | nuclei -id ssl-dns-names -o comcast.com.subs.ssl
```

Now, extract wildcard subdomains from the output:

```bash
cat comcast.com.subs.ssl | cut -d " " -f 5 | sed 's/\[//' | sed 's/\]//' | tr "," "\n" | tr -d "\"" | grep -I "comcast.com$" | grep "*" | sort | uniq
```

![Wildcard Extraction](https://cdn-images-1.medium.com/max/1024/1*9tfRAXaFHfyDCtIWM-pZcQ.png)

You can treat patterns like `*.xap.comcast.com` as separate domains and run subdomain enumeration tools or DNS fuzzing to uncover related assets.

---

### Bruteforce Virtualhosts Within IPs With Wildcard CN

To identify virtual hosts behind an IP with wildcard subdomains:

**First**, resolve subdomains to IPs:

```bash
cat comcast.com.subs | dnsx -a -resp-only | anew ips.txt
```

**Then**, extract SSL cert data from these IPs:

```bash
cat ips.txt | nuclei -id ssl-dns-names | anew ips.ssl.txt
```

![SSL Output](https://cdn-images-1.medium.com/max/1024/1*q1W1T2Yb-31qSsESM8VZJA.png)

Look for a wildcard pattern:

![Wildcard CN](https://cdn-images-1.medium.com/max/1024/1*1TpJM777NbGXbAsm6knilQ.png)

Notice that `xfinity.com` and `comcat.com` are in scope.

---

### Brute Force Host Headers

Use a list of potential hostnames to uncover hidden services:

```bash
ffuf -u https://162.150.57.78:443 -H "Host: FUZZ.aws-np.identity.xfinity.com" -w ~/wordlists/2m-subdomains.txt -mc all -c -ac
```

Another valuable resource for wildcard subdomains is:

- **[crt.sh](https://crt.sh)**

![crt.sh](https://cdn-images-1.medium.com/max/1024/1*xUpQA_YatTZ2-iALLp73lw.png)

---

Thanks for reading! 🚀
