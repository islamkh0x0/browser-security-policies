### Same-Origin Policy (SOP) and Cross-Origin Resource Sharing (CORS)

#### **1. Same-Origin Policy (SOP)**

Same-Origin Policy (SOP) is a security mechanism implemented by web browsers to prevent malicious websites from accessing data on other websites without permission. It restricts how scripts loaded from one origin can interact with resources from another origin.

##### **1.1  What Defines an "Origin"?**

An **origin** consists of three parts:

1. **Scheme (Protocol)** – like: `http`, `https`
2. **Domain** – like: `example.com`
3. **Port** – like: `80`, `443`

If any of these components differ between two URLs, they are considered **different origins**.

##### **1.2 Example: How SOP Applies**

Consider a webpage hosted at:

```
http://islam.com/example/example.html
```

The following table shows whether the page can access different URLs:

| URL Attempted                    | Access Allowed? | Reason                                      |
| -------------------------------- | --------------- | ------------------------------------------- |
| `http://islam.com/example/`      | ✅ Yes           | Same scheme, domain, and port               |
| `http://islam.com/example2/`     | ✅ Yes           | Same scheme, domain, and port               |
| `https://islam.com/example/`     | ❌ No            | Different scheme (`https` vs `http`)        |
| `http://en.islam.com/example/`   | ❌ No            | Different domain (`en.normal-website.com`)  |
| `http://www.islam.com/example/`  | ❌ No            | Different domain (`www.normal-website.com`) |
| `http://islam.com:8080/example/` | ❌ No            | Different port (`8080` vs `80`)             |

##### **1.3 The importance of SOP**

SOP prevents unauthorized access between websites, protecting users from attacks such as:

- A malicious website reading a user's **emails from Gmail**.
- A hacker site accessing **private messages from Facebook**.
- An attacker stealing **session cookies** and impersonating the user.

Without SOP, visiting a malicious site while logged into another service could expose sensitive data.

##### **1.4 How SOP is Enforced?**

JavaScript running in a webpage is **not allowed** to:

- Read data from another origin (e.g., making `XMLHttpRequest` or `fetch` to another domain).
- Access properties of iframes or windows loaded from a different origin.

However, some cross-origin resource loading is **allowed**:

- **Images (`<img>`)**
- **Videos (`<video>`)**
- **Scripts (`<script>`)**
- **Stylesheets (`<link>` for CSS)**

Even though these resources can be loaded, JavaScript **cannot** read their content if they originate from a different domain.

---

## **2. Overriding SOP with CORS**

Since SOP is restrictive, sometimes developers need a way to **safely** allow cross-origin requests. This is where **CORS (Cross-Origin Resource Sharing)** comes in.

### **2.1 What is CORS?**

CORS is a mechanism that **allows servers** to specify which origins are permitted to access their resources. It is implemented using HTTP headers.

Without CORS, a JavaScript request from `http://test.com` to `https://api.example.com` would be blocked by SOP.

With CORS, the server at `https://api.example.com` can **explicitly allow** requests from `http://test.com` by including the appropriate headers.

### **2.2 How CORS Works**

When a browser makes a cross-origin request, it follows these steps:

1. **The browser sends an HTTP request** to the target server.
2. **The server responds with CORS headers** indicating whether the request is allowed.

The most important CORS header is:

```mathematica
Access-Control-Allow-Origin: <allowed-origin>
```

Example response allowing `http://test.com`:

```mathematica
Access-Control-Allow-Origin: http://test.com
```

If the header is missing or set incorrectly, the browser **blocks the request**.

---

## **3. Types of CORS Requests**

There are three types of CORS requests:

### **3.1 Simple Requests**

A request is considered **simple** if it meets **ALL** of these conditions:

- Uses one of these HTTP methods:
    - `GET`, `POST`, or `HEAD`
- Only includes **safe headers**:
    - `Accept`, `Content-Type`, `Origin`
- `Content-Type` is one of:
    - `text/plain`, `multipart/form-data`, `application/x-www-form-urlencoded`

Example:

```js
fetch("https://api.example.com/data", {
  method: "GET"
})
```

If the server responds with:

```mathematica
Access-Control-Allow-Origin: *
```

The browser **allows** the request and any request from other domains

---

### **3.2 Preflight Requests**

If a request does **not** meet the "simple request" criteria, the browser **sends a preflight request** before making the actual request.

- Preflight uses an `OPTIONS` request.
- It asks the server whether the actual request is allowed.
- If the server permits it, the browser proceeds with the request.

Example:

```js
fetch("https://api.example.com/update", {
  method: "PUT",
  headers: {
    "Content-Type": "application/json",
    "Authorization": "Bearer token"
  }
})
```

Since this request:

- Uses `PUT` (not a simple method)
- Includes `Authorization` (not a simple header)

The browser first sends:

```mathematica
OPTIONS /update HTTP/1.1
Origin: http://test.com
Access-Control-Request-Method: PUT
Access-Control-Request-Headers: Authorization
```

If the server allows it, it responds with:

```mathematica
HTTP/1.1 200 OK
Access-Control-Allow-Origin: http://test.com
Access-Control-Allow-Methods: PUT
Access-Control-Allow-Headers: Authorization
```

Then, the browser makes the actual `PUT` request.

---

### **3.3 Credentialed Requests**

By default, CORS **does not** allow sending cookies or authentication headers. To allow them:

- The client must set `credentials: "include"`:
    
    ```js
    fetch("https://api.example.com/user", {
      credentials: "include"
    })
    ```
    
- The server must include:
    
    ```mathematica
    Access-Control-Allow-Credentials: true
    ```


**But what if the server does not allow it?**

- If the browser sets `credentials: "include"` but the server does **not** include `Access-Control-Allow-Credentials: true`, the browser will block the request from accessing the protected data.
- If the server sets `Access-Control-Allow-Origin: *` along with `Access-Control-Allow-Credentials: true`, the browser will **still block the request** because this combination is **not allowed for security reasons**.

Note --> If you intercept the request using **Burp Suite or any external tool**, you can view the full response without any restrictions. This is because **SOP and CORS are browser-enforced policies**, not server-side restrictions.

---
## So, thts Mean That:

**SOP**: Prevents cross-origin access to protect users from attacks.  
**CORS**: Allows controlled cross-origin requests using HTTP headers.  
**Simple Requests**: Allowed if they use safe methods/headers.  
**Preflight Requests**: Sent before non-simple requests for security approval.  
**Credentialed Requests**: Require `Access-Control-Allow-Credentials: true`.  
