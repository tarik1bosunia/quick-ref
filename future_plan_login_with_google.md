## The Complete Process of Login with Google

This guide outlines the complete process for implementing Google login, with a focus on a modern stack: Next.js (TypeScript, Tailwind) for the frontend and Django (DRF, Simple JWT) for the backend. The process follows the **OAuth 2.0 Authorization Code Flow**, a secure and standard way to handle third-party authentication.

### Core Concepts

  * **OAuth 2.0:** An authorization framework that allows a user to grant a third-party application access to their information without sharing their password.
  * **Authorization Code:** A temporary, single-use code issued by Google to your backend, which is then exchanged for access and ID tokens. This is more secure than the implicit flow, as tokens are never exposed in the browser's URL.
  * **ID Token (JWT):** A JSON Web Token issued by Google that contains information about the authenticated user (e.g., name, email).
  * **Access Token:** A token used by your backend to make API calls to Google on behalf of the user.
  * **JWT (from your backend):** The token your Django backend issues to the frontend after successfully validating the Google ID token. This is what the frontend will use for all subsequent authenticated requests to your own API.

### 1\. Google Cloud Console Setup 🛠️

This is the first and most critical step, as it sets up the communication between your app and Google.

  * **Create a Google Cloud Project:** Go to the [Google Cloud Console](https://console.cloud.google.com/) and create a new project.
  * **Configure OAuth Consent Screen:**
      * Navigate to **APIs & Services \> OAuth consent screen**.
      * Choose a user type (e.g., "External" for public apps).
      * Fill in the required app information (name, user support email, etc.).
      * Add the necessary **scopes**. For basic user login, you'll need `.../auth/userinfo.email` and `.../auth/userinfo.profile`.
      * Add **test users** if you're still in development.
  * **Create OAuth Client ID:**
      * Navigate to **APIs & Services \> Credentials**.
      * Click **Create Credentials** and select **OAuth client ID**.
      * Select **"Web application"** as the application type.
      * **Authorized JavaScript origins:** Add your frontend domains (e.g., `http://localhost:3000`, `https://your-production-app.com`).
      * **Authorized redirect URIs:** Add your backend's callback URL. This is where Google will send the user back after they sign in. A common format is `http://localhost:8000/api/auth/google/callback/`.
  * **Store Credentials:** Note down the **Client ID** and **Client Secret**. These will be used in both your frontend and backend.

**Google Documentation Exact Link:**

  * [Using OAuth 2.0 for Web Server Applications](https://developers.google.com/identity/protocols/oauth2/web-server)

### 2\. Frontend (Next.js with TypeScript, Tailwind)

The frontend's role is to initiate the OAuth flow and manage the JWT from your backend. A recommended library is `next-auth` as it simplifies a lot of the boilerplate.

#### Key Steps:

1.  **Install `next-auth`:**
    ```bash
    npm install next-auth
    ```
2.  **Configure NextAuth:**
      * Create a file at `pages/api/auth/[...nextauth].ts`. This file will contain your NextAuth configuration.
      * Use the Google Provider and configure it with your `GOOGLE_CLIENT_ID` and `GOOGLE_CLIENT_SECRET`.
      * Crucially, you'll need a **custom callback function** to handle the token exchange with your Django backend.
3.  **Create Login Button:**
      * On your login page, create a button that triggers the sign-in flow.
      * Use `signIn` from `next-auth/react` to initiate the Google login.

<!-- end list -->

```tsx
// Example of a login button component
"use client";

import { signIn } from "next-auth/react";
import { FaGoogle } from "react-icons/fa";

export default function GoogleSignInButton() {
  return (
    <button
      onClick={() => signIn("google")}
      className="flex items-center gap-2 p-3 text-white bg-blue-500 rounded-md"
    >
      <FaGoogle />
      Sign in with Google
    </button>
  );
}
```

### 3\. Backend (Django, Django REST Framework, simple-jwt)

The backend is the core of the authentication logic. It validates the token from Google and issues its own JWT for your API. A very common approach is to use `django-allauth` to handle the Google part and then issue your own tokens.

#### Key Steps:

1.  **Install Dependencies:**
    ```bash
    pip install djangorestframework django-rest-framework-simplejwt django-allauth
    ```
2.  **Configure `settings.py`:**
      * Add `rest_framework`, `rest_framework_simplejwt`, `allauth`, `allauth.account`, `allauth.socialaccount`, and `allauth.socialaccount.providers.google` to `INSTALLED_APPS`.
      * Configure `SIMPLE_JWT` to set the token lifetimes and other security settings.
      * Set up `AUTHENTICATION_BACKENDS` for `allauth`.
      * Define `LOGIN_REDIRECT_URL` and other relevant `allauth` settings.
3.  **Define a Custom View/API Endpoint:**
      * Create a DRF view that will be the target of your Google OAuth redirect.
      * This view will receive the authorization code from Google.
      * It will then make a secure, server-to-server POST request to Google's token endpoint to exchange the code for an ID token.
      * Validate the ID token received from Google.
      * Use the user information from the ID token (e.g., email) to either create a new user or retrieve an existing one.
4.  **Issue JWT:**
      * After successfully authenticating the user, use `simple-jwt` to generate an access and refresh token.
      * Return these tokens to the frontend in a JSON response. The frontend will then store these tokens (e.g., in `localStorage` or `HttpOnly` cookies) for future API requests.

**Documentation Links:**

  * [Django-allauth Google Provider](https://docs.allauth.org/en/latest/socialaccount/providers/google.html)
  * [Django REST Framework Simple JWT](https://django-rest-framework-simplejwt.readthedocs.io/en/latest/)

### Flow Summary

1.  User clicks "Sign in with Google" on the Next.js frontend.
2.  The frontend redirects the user to Google's sign-in page.
3.  Google authenticates the user and asks for their consent.
4.  Upon consent, Google redirects the user back to your Django backend's redirect URI with an `authorization code`.
5.  The Django backend receives this code and exchanges it for an ID token and access token from Google (server-to-server).
6.  The backend validates the ID token, either creates a new user or logs in an existing one based on the email.
7.  The backend generates a new JWT (from `simple-jwt`) for the user.
8.  The backend sends this new JWT to the frontend.
9.  The frontend stores the JWT and uses it to make authenticated requests to your Django API.
