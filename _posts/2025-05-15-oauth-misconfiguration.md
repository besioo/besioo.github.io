![](https://cdn-images-1.medium.com/max/1024/1*v_MYqiBJRuorhswKKhl_KA.png)


OAuth makes logging in easy, but one small misconfiguration can lead to full account takeover. In this post, I’ll show how skipping a proper token issuer (iss) validation allows an attacker to hijack user accounts — **without abusing redirect URIs**. All it takes is a malicious OAuth app and a bit of trust in the wrong token.

### What is OAuth?

**OAuth (Open Authorization)** is an open standard that allows applications to obtain limited access to user accounts on an HTTP service, such as Google or Facebook, without exposing the user’s credentials. It enables secure delegated access using access tokens issued by an authorization server. OAuth is widely used for “Log in with” functionality and API authorization. For full specifications, refer to the OAuth 2.0 RFC 6749 and the official site [oauth.net](https://oauth.net/).

### Sharing Access Token

A common behavior when using OAuth is sharing the access token directly without the need to exchange the code **server-side** to get an access token

In this scenario, the app asks the user to share a Google access token, for example, which typically has limited permissions — such as access to the user’s email and basic profile information. This limited scope is usually enough for the app to authenticate the user by verifying that they own the provided email address. This approach also applies to other OAuth providers like Facebook and others.

### In Practice

As an example, I created a simple app at [https://besioo.github.io/google\_auth/index.html](https://besioo.github.io/google_auth/index.html) that allows users to share their Google access token directly. In this case, the OAuth request uses the parameter response\_type set to token instead of code to receive the access token immediately.

When a user clicks a link like this: https://accounts.google.com/o/oauth2/v2/auth/oauthchooseaccount?client\_id=502214687020-0arr62kbk6oaiuq9p687sd4ij6fr9mjf.apps.googleusercontent.com&redirect\_uri=https%3A%2F%2Fbesioo.github.io%2Fgoogle\_auth%2Findex.html&response\_type=token&scope=openid%20email%20profile&include\_granted\_scopes=true&prompt=select\_account&service=lso&o2v=2&flowName=GeneralOAuthFlow  
They will be prompted to select a Google account or grant the app permission to access their token.

Then, the user is redirected to https://besioo.github.io/google\_auth/index.html with the access token included in the URL fragment.

The app uses the access token to call Google’s endpoint https://www.googleapis.com/oauth2/v3/userinfo and retrieve the user’s profile information.

curl 'https://www.googleapis.com/oauth2/v3/userinfo' -H "Authroization: Bearer ya29.a0AW4...."

![](https://cdn-images-1.medium.com/max/1024/1*abbpnqiM8IYQG4-sXto5NA.png)

The app then authenticates the user based on the email address returned in the response.

### Misconfiguration

If we use a token generated by **App1** to authenticate to **App2**, and App2 does **not validate the token’s intended audience (aud) or issuer (iss)**, it may incorrectly accept the token, allowing unauthorized access. This happens when App2 blindly trusts any valid-looking token without verifying that it was issued **for** its client ID.

In this case, an attacker can create their own OAuth app using Google or Facebook, then trick users into logging in and sharing their access tokens. The attacker can then use these collected tokens to authenticate to a **vulnerable app** that doesn’t properly validate the token’s issuer or audience, effectively hijacking the user’s account.

### Real World Exploit

In a real-world scenario, an attacker would create an app that appears legitimate and use it to phish users into logging in. Once users authorize the app, the attacker captures their Google or Facebook access tokens — which can then be used against vulnerable apps that fail to validate token sources properly.

**But…..**

This may sound like a phishing attack, but can it really be done in just one click? The answer is yes.

An optional parameter you can include in an OAuth request is prompt=none, which tells the OAuth service not to show any consent or login prompt to the user. If the user is already logged in and has previously granted permission, the redirect happens automatically.

A link like this:  
https://accounts.google.com/o/oauth2/v2/auth/oauthchooseaccount?client\_id=502214687020-0arr62kbk6oaiuq9p687sd4ij6fr9mjf.apps.googleusercontent.com&redirect\_uri=https%3A%2F%2Fbesioo.github.io%2Fgoogle\_auth%2Findex.html&response\_type=token&scope=openid%20email%20profile&include\_granted\_scopes=true&prompt=select\_account&service=lso&o2v=2&flowName=GeneralOAuthFlow&prompt=none  
will redirect the user to [https://besioo.github.io/google\_auth/index.html](https://besioo.github.io/google_auth/index.html) with an access token included in the URL fragment.

I tested this with Google and found that when the prompt parameter is set to none, the user does **not** need to have previously authorized the app. The authorization happens silently without prompting the user.

If the user is logged in to multiple accounts, the process will fail because Google requires the user to select which account to use. However, this can be bypassed by adding the parameter authuser=0 to force authentication with the first account.

### Impact

This misconfiguration can lead to a **single-click account takeover**. By obtaining the user’s access token, an attacker can use it directly with the vulnerable app, without needing to exchange or validate it properly, and hijack the user’s account.

### **Conclusion**

OAuth is a powerful tool that simplifies authentication, but small misconfigurations can lead to serious security risks like account hijacking. Proper token validation — including verifying the token’s issuer and audience — is essential to secure your applications. Developers should audit their OAuth implementations and ensure they follow best practices to protect users. As users, always be cautious when authorizing third-party apps. If you have questions or want to discuss OAuth security further, feel free to reach out or leave a comment below.

![](https://medium.com/_/stat?event=post.clientViewed&referrerSource=full_rss&postId=e287680a15cb) \]\]>
