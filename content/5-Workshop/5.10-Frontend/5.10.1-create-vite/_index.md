---
title : "Create a Vite Application"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.10.1 </b> "
---

Create a single-page application using React, TypeScript, and [Vite](https://vite.dev/guide/). The current Vite documentation requires Node.js `20.19+` or `22.12+`. Use a supported Node.js LTS version that meets this requirement.

## Initialize the project

Run:

```bash
node --version
npm create vite@latest secure-doc-frontend -- --template react-ts
cd secure-doc-frontend
npm install
npm install aws-amplify react-router-dom
npm run dev
```

Open `http://localhost:5173` and verify that the default Vite page is displayed.

Create the following application structure:

```text
src/
├── app/
│   ├── App.tsx
│   ├── queryClient.ts
│   └── router.tsx
├── components/
│   ├── admin/
│   ├── common/
│   ├── documents/
│   ├── layout/
│   └── upload/
├── config/
│   └── env.ts
├── contexts/
│   ├── AuthContext.tsx
│   └── ToastContext.tsx
├── hooks/
├── lib/
│   ├── apiClient.ts
│   ├── authorization.ts
│   └── jwt.ts
├── pages/
│   ├── admin/
│   ├── AuthCallbackPage.tsx
│   ├── DocumentsPage.tsx
│   ├── DocumentDetailPage.tsx
│   ├── LoginPage.tsx
│   └── UploadDocumentPage.tsx
├── services/
│   ├── authService.ts
│   ├── adminDocumentService.ts
│   ├── adminService.ts
│   └── documentService.ts
├── types/
└── main.tsx
```

![structure systems](/images/5-Workshop/5.10/5.10.1/5.10.1.1.png)

## Configure public environment variables

Create a `.env.example` file:

```text
VITE_AWS_REGION=ap-southeast-1
VITE_COGNITO_USER_POOL_ID=ap-southeast-1_example
VITE_COGNITO_CLIENT_ID=example-client-id
VITE_COGNITO_DOMAIN=example.auth.ap-southeast-1.amazoncognito.com
VITE_API_BASE_URL=https://<api-id>.execute-api.ap-southeast-1.amazonaws.com
```

![env systems](/images/5-Workshop/5.10/5.10.1/5.10.1.2.png)

Copy the file to `.env.local`, replace the sample values, and ensure that Git ignores `.env.local`. Do not include `https://` in `VITE_COGNITO_DOMAIN`; the Amplify configuration accepts only the domain name.

## Connect to the existing Cognito User Pool

Create `src/services/authService.ts`:

```ts
const params = new URLSearchParams({
  response_type: "code",
  client_id: env.VITE_COGNITO_CLIENT_ID,
  redirect_uri: env.VITE_COGNITO_REDIRECT_URI,
  scope: "openid email",
  state,
  code_challenge_method: "S256",
  code_challenge: base64Url(await sha256(verifier)),
  prompt: "login",
});

window.location.assign(
  `${env.VITE_COGNITO_DOMAIN}/oauth2/authorize?${params.toString()}`,
);
```

![cognito service](/images/5-Workshop/5.10/5.10.1/5.10.1.3.png)

This configuration directly connects the Amplify client library to the Cognito User Pool and App Client created in Section 5.2. It does not create an additional backend.

Import the configuration before rendering React in `src/main.tsx`:

```tsx
import "./lib/amplify";
import { StrictMode } from "react";
import { createRoot } from "react-dom/client";
import { BrowserRouter } from "react-router-dom";
import App from "./App";

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <BrowserRouter>
      <App />
    </BrowserRouter>
  </StrictMode>,
);
```

## Add sign-in and session management

Use the Amplify Auth modular API:

```ts
import { authService } from "../services/authService";
import { isTokenExpired, userFromAccessToken } from "./jwt";

export const startSignIn = (): Promise<void> =>
  authService.login();

export const endSession = (): void =>
  authService.logout();

export async function loadSession() {
  let accessToken = authService.getToken();

  if (accessToken && isTokenExpired(accessToken)) {
    if (!authService.getRefreshToken()) {
      authService.clearSession();
      accessToken = null;
    } else {
      try {
        accessToken = await authService.refreshAccessToken();
      } catch {
        authService.clearSession();
        accessToken = null;
      }
    }
  }

  const user = accessToken
    ? userFromAccessToken(accessToken)
    : null;

  return {
    user,
    accessToken,
    groups: user?.groups ?? [],
  };
}
```

Example authentication code:

![Auth systems](/images/5-Workshop/5.10/5.10.1/5.10.1.4.png)

## Configure routes

Create the following routes:

```text
/
/login
/auth/callback
/documents
/documents/:documentId
/upload
/forbidden
/admin
/admin/dashboard
/admin/users
/admin/users/:username
/admin/documents
/admin/documents/:documentId/review
```

Require an authenticated session for all document-related routes. Display the **Admin** menu only when the user's access token contains the `Admin` group, and protect all `/admin/*` routes using the same condition on the frontend.

{{% notice warning %}}
Checking the user group on the frontend only protects the user interface. Amazon API Gateway and the administrative Lambda functions implemented in Section 5.9 must still validate the JWT token and verify membership in the `Admin` group.
{{% /notice %}}

Before continuing, verify that signing in redirects the user to Amazon Cognito, the callback returns to `/callback`, refreshing the page preserves the session, signing out redirects to `/`, and standard users cannot access the Admin routes.