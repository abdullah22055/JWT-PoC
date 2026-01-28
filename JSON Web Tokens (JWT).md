### What's JWT:
JSON Web Token (JWT) is a compact, Base64URL-encoded string used for authentication, authorization, and information exchange in modern web applications. It is made up of three parts separated by dots (.): 

`header.payload.signature`--> The order is crucial.
##### Header: 
  Contains metadata of the token (token type, signing algorithm, etc.)
##### Payload:
  Contains user data (claims)
##### Signature:
   Ensures token's integrity
### Sample JWT
```
Base64-Encoded JWT:

eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJ1c2VySWQiOjEsInJvbGUiOiJhZG1pbiJ9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c

Human-readable Format:

Header
{
  "alg": "HS256",   
  "typ": "JWT"
}

Payload
{
  "user_id": 101,
  "role": "admin",
  "exp": 1700000000
}

Signature Metadata is not human-readable

```

### How JWT Works:
- Upon successful login, server creates a JWT containing user-specific claims. 
- This token is sent to the user's browser in the form of a cookie or in local storage.
- For each request in subsequent requests, client includes JWT under `Authorization` header.
- The server validates the request upon valid JWT. Even third-party services can validate JWT without referring to the issuer. This is why JWT provides decentralization.
- JWT expires at `exp` claim time (mentioned in Payload part above).
-  It provides stateless session management, unlike HTTP.

## Pentester's Perspective:

As a pentester, my main focus typically revolves around understanding the behavior of an app or a specific security mechanism, looking for flaws, exploit them and finding remedies to secure it. As for JWT, here is a high level testing methodology on how issues are generally found, followed by a detailed PoC of the **None Algorithm Vulnerability Exploitation**.   
### What to Look For:
- Identify where the JWT is stored. Cookie, LocalStorage, or Authorization header?
- Decode the token and inspect header and all claims.
- Check the algorithm  
- Test for signature bypass. Change `alg` to `none`, remove signature, replay request
### PoC: The None Algorithm Vulnerability: 
#### Cause:
 The None Algorithm Vulnerability typically refers to the setting the algorithm value (in the header) to `none`. This happens when server sets an algorithm value (HS256, RS256, etc.)in the header section of  JWT but fails at the verification part. Simply put, server is not enforcing a fixed, expected algorithm on the server side and blindly trusting user-controlled input.
  An attacker might intercept the token and set the value to none and as a result, the server validates without verifying the token's integrity. This may sound confusing but follow the PoC below. It will definitely help!
#### Relevance of in Modern Pentesting:
In modern pentesting, none algorithm is not that prevalent in well-developed applications yet it still exists in custom web apps. The reason for the PoC is to understand JWT internals and how the JWT verification logic works during penetration tests. 

# PoC: JWT authentication bypass via flawed signature verification (Portswigger)

### Step 1:
Open the lab in the Burp browser:([](https://portswigger.net/web-security/jwt/lab-jwt-authentication-bypass-via-flawed-signature-verification)).
To keep the PoC to the point, I'm providing the link to set JWT Editor Extension:[](https://youtu.be/9xhdK6HHLjk?si=qxvlRVxphpOJsrar).
### Step 2:
Click on `My account` and login with creds provided: wiener:peter 
<p align="center">
<img width="60%" src="./Pasted image 20260128185051.png">
</p>

### Step 3:
Note that when you login, requests in http history containing JWT will be highlighted as below:
<p align="center">
<img width="60%" src="./Pasted image 20260128185409.png">
</p>
 Send the `GET /my-account?id=wiener` request to our *common brother* `Repeater`!

### Step 4:
You will see the JWT under the **Cookie** header. Double clicking on the header part will show the decoded format: id and algorithm (RS256).
Change the RS256 to none and click on Apply changes. 
<p align="center">
<img width="60%" src="./Pasted image 20260128185806.png">
</p>

### Step 5:
Same goes for the payload part of JWT. Change the username wiener to administrator and apply changes.
<p align="center">
<img width="60%" src="./Pasted image 20260128191352.png">
</p>

### Step 6:
Remove the Signature part completely, leaving the last dot. With this change the path to /admin for accessing admin panel as demonstrated below:
<p align="center">
<img width="60%" src="./Pasted image 20260128200501.png">
</p>

### Step 7:
In the response, you will see we have accessed the admin panel.
<p align="center">
<img width="60%" src="./Pasted image 20260128200810.png">
</p>
Change the path to `/admin/delete?username=carlos` and you will see we have successfully deleted user Carlos.
### How to Secure it (Developer's Perspective):
- Never let user-input decide the behavior security mechanisms.
- Treat whole JWT as untrusted, therefore validating all entries.
- The wrong coding practice:
   `jwt.verify(token, key)`
- The correct practice:
   `jwt.verify(token, key, { algorithms: ['RS256']})` 

This PoC is intended for educational and defensive purposes only. All testing was conducted in a controlled lab environment. This brings us to the end of the PoC. Hope it helps!


