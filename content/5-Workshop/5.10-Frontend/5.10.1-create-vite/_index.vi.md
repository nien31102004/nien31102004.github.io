---
title : "Tạo ứng dụng Vite"
date : 2024-01-01
weight : 1
chapter : false
pre : " <b> 5.10.1 </b> "
---

Tạo single-page application bằng React, TypeScript và [Vite](https://vite.dev/guide/). Tài liệu Vite hiện tại yêu cầu Node.js `20.19+` hoặc `22.12+`; hãy dùng một bản Node.js LTS còn được hỗ trợ và thỏa điều kiện này.

## Khởi tạo dự án

Chạy:

```bash
node --version
npm create vite@latest secure-doc-frontend -- --template react-ts
cd secure-doc-frontend
npm install
npm install aws-amplify react-router-dom
npm run dev
```

Mở `http://localhost:5173` và xác nhận trang khởi tạo hiển thị.

Tạo cấu trúc ứng dụng:

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

## Cấu hình biến môi trường công khai

Tạo `.env.example`:

```text
VITE_AWS_REGION=ap-southeast-1
VITE_COGNITO_USER_POOL_ID=ap-southeast-1_example
VITE_COGNITO_CLIENT_ID=example-client-id
VITE_COGNITO_DOMAIN=example.auth.ap-southeast-1.amazoncognito.com
VITE_API_BASE_URL=https://<api-id>.execute-api.ap-southeast-1.amazonaws.com
```

![env systems](/images/5-Workshop/5.10/5.10.1/5.10.1.2.png)

Sao chép thành `.env.local`, thay các giá trị mẫu và bảo đảm Git bỏ qua `.env.local`. Không thêm `https://` vào `VITE_COGNITO_DOMAIN`; cấu hình Amplify chỉ nhận tên domain.

## Kết nối Cognito user pool hiện có

Tạo `src/services/authService.ts`:

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

Cấu hình này kết nối trực tiếp Amplify client library với user pool và app client đã tạo ở phần 5.2; nó không tạo thêm backend mới.

Import cấu hình trước khi render React trong `src/main.tsx`:

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

## Thêm hàm đăng nhập và phiên làm việc

Dùng modular API của Amplify Auth:

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
Code minh họa cho auth
![Auth systems](/images/5-Workshop/5.10/5.10.1/5.10.1.4.png)

## Cấu hình route

Tạo các route:

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

Yêu cầu phiên đã xác thực cho các route tài liệu. Chỉ hiển thị menu Admin khi group trong access token chứa `Admin` và bảo vệ mọi route `/admin/*` bằng cùng điều kiện ở frontend.

{{% notice warning %}}
Kiểm tra group ở frontend chỉ bảo vệ giao diện. API Gateway và các Lambda quản trị ở phần 5.9 vẫn phải xác minh token cùng nhóm `Admins`.
{{% /notice %}}

Trước khi tiếp tục, hãy xác nhận đăng nhập chuyển đến Cognito, callback quay lại `/callback`, refresh vẫn giữ phiên, logout trở về `/` và người dùng thường không mở được route Admin.
