# JWT Authentication with Next.js and Django REST Framework

## 🔐 What is JWT (JSON Web Token)?

A **JWT (JSON Web Token)** is a compact, URL-safe way of representing claims to be transferred between two parties. It is commonly used for **authentication and authorization**.

A JWT is composed of three parts:
```
header.payload.signature
```

Example:
```
eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9
.
eyJ1c2VyX2lkIjoxLCJ1c2VybmFtZSI6ImpvaG5kb2UiLCJleHAiOjE2MDE2ODgyMDAsImlhdCI6MTYwMTY4NDYwMH0
.
abc123signaturexyz
```

### Header
- Specifies the algorithm and token type (usually `HS256` and `JWT`).

### Payload
- Contains claims (data), such as user ID, username, and expiry.

### Signature
- Used to verify the token's authenticity and integrity using a secret key.

---

## 🔐 What is an MITM Attack?

**Man-in-the-middle (MITM)** attack is when an attacker secretly intercepts and possibly alters the communication between two parties.

Example:
- You think you’re connected to `example.com`, but your connection is intercepted by an attacker who sees your data.

Mitigation:
- Use HTTPS (TLS encryption)
- Use `Secure`, `HttpOnly`, and `SameSite` cookies
- Validate SSL certificates

---

## 🔧 Next.js + Django REST Framework JWT Cookie Auth Setup

### ✅ Backend: Django

#### Install `SimpleJWT`
```bash
pip install djangorestframework-simplejwt
```

#### In `settings.py`
```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'rest_framework_simplejwt.authentication.JWTAuthentication',
    )
}
```

#### In `urls.py`
```python
from rest_framework_simplejwt.views import TokenObtainPairView, TokenRefreshView

urlpatterns = [
    path('api/token/', TokenObtainPairView.as_view(), name='token_obtain_pair'),
    path('api/token/refresh/', TokenRefreshView.as_view(), name='token_refresh'),
]
```

---

### ✅ Frontend: Next.js

#### `app/api/auth/login/route.ts`

```ts
import { NextRequest, NextResponse } from "next/server";

export async function POST(req: NextRequest) {
  const { email, password } = await req.json();

  const res = await fetch("https://your-backend.com/api/token/", {
    method: "POST",
    headers: {
      "Content-Type": "application/json",
    },
    body: JSON.stringify({ email, password }),
  });

  if (!res.ok) {
    return NextResponse.json({ error: "Invalid credentials" }, { status: 401 });
  }

  const data = await res.json();
  const accessToken = data.access;
  const refreshToken = data.refresh;

  const response = NextResponse.json({ success: true });

  response.cookies.set("access_token", accessToken, {
    httpOnly: true,
    secure: true,
    sameSite: "strict",
    path: "/",
    maxAge: 60 * 15,
  });

  response.cookies.set("refresh_token", refreshToken, {
    httpOnly: true,
    secure: true,
    sameSite: "strict",
    path: "/",
    maxAge: 60 * 60 * 24 * 7,
  });

  return response;
}
```

---

## 🔐 How Django Gets the Token from Cookie

### 1. Create `authentication.py`
```python
from rest_framework_simplejwt.authentication import JWTAuthentication

class CustomJWTAuthentication(JWTAuthentication):
    def authenticate(self, request):
        token = request.COOKIES.get("access_token")
        if token is None:
            return None
        validated_token = self.get_validated_token(token)
        return self.get_user(validated_token), validated_token
```

### 2. Use in `settings.py`
```python
REST_FRAMEWORK = {
    'DEFAULT_AUTHENTICATION_CLASSES': (
        'yourapp.authentication.CustomJWTAuthentication',
    )
}
```

### 3. Example Protected View
```python
from rest_framework.permissions import IsAuthenticated
from rest_framework.views import APIView
from rest_framework.response import Response

class ProtectedView(APIView):
    permission_classes = [IsAuthenticated]

    def get(self, request):
        return Response({
            "message": "You are authenticated!",
            "user": request.user.username
        })
```

---

## 🧪 Example Frontend Call to Protected Route

```ts
const res = await fetch("https://your-backend.com/api/protected/", {
  method: "GET",
  credentials: "include",
});
```

---

## ✅ Security Features Summary

| Feature                 | Status |
|------------------------|--------|
| JWT in HttpOnly Cookie | ✅     |
| `Secure` Flag          | ✅     |
| `SameSite=Strict`      | ✅     |
| Custom Auth Handler    | ✅     |
| HTTPS Recommended      | ✅     |


---

# 🕵️‍♂️ MITM Attacks (Man-in-the-Middle Attacks)

## 📌 What is a MITM Attack?

A Man-in-the-Middle (MITM) attack is a type of cyberattack where a malicious actor intercepts and possibly alters the communication between two parties (e.g., between your browser and a server) without them knowing.

### 🧠 In simple terms:

A hacker “sits in the middle” between your device and a website/server, secretly watching or changing the data being transmitted.

---

## 🔁 Example Scenario

You log into a website using a public Wi-Fi:

1. You send your login credentials.
2. An attacker on the same network intercepts the data.
3. If the connection is not secure (HTTPS), they can:
   - Read your credentials (username, password).
   - Steal your JWT access token.
   - Impersonate you.

---

## 🧰 Common Techniques Used in MITM Attacks

| Technique          | Description                                      |
|--------------------|--------------------------------------------------|
| Packet sniffing    | Capturing unencrypted data using tools like Wireshark. |
| DNS spoofing       | Redirecting you to a fake version of a site.     |
| IP spoofing        | Impersonating a legitimate device’s IP address.  |
| SSL stripping      | Downgrading HTTPS to HTTP so traffic can be read. |
| Wi-Fi eavesdropping| Intercepting data over insecure Wi-Fi.           |

---

## 🔓 What Happens to JWT in a MITM Attack?

If a JWT token is:
- Sent over unencrypted HTTP or
- Stored in insecure locations (e.g., localStorage)

An attacker can intercept it and:
- Use it to authenticate as the user
- Unless the token is expired or invalidated

---

## 🛡️ How to Protect Against MITM Attacks

| Protection              | Explanation                                                    |
|-------------------------|----------------------------------------------------------------|
| 🔒 Use HTTPS (TLS)       | Always use HTTPS for all API calls. Never use HTTP.            |
| 🔐 Use httpOnly Cookies  | Store tokens in httpOnly cookies so JavaScript can’t access them. |
| 👀 Certificate Pinning   | Ensures you only trust a specific SSL cert for your domain.     |
| 🧠 Validate Tokens       | Check `exp`, `iss`, `aud`, and signature on every request.      |
| 🧼 Avoid Public Wi-Fi    | Don’t send sensitive data over public networks without VPN.     |
| 🚨 Set Short Expiry      | Keep access tokens short-lived (e.g., 15 minutes).              |
