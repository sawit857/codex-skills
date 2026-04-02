# React + Vite + PrimeReact Coding Standards & Examples

## 1. Project Structure มาตรฐาน

```
src/
├── api/              # API client — centralized fetch/axios
│   ├── client.ts     # Axios instance + interceptors
│   └── endpoints.ts  # API endpoint constants
├── auth/             # Auth logic — token management
│   ├── AuthContext.tsx
│   ├── useAuth.ts
│   └── tokenStore.ts # In-memory token store (ห้ามใช้ localStorage)
├── providers/        # Context providers
│   └── AppToastProvider.tsx  # Global Toast context
├── components/       # Shared PrimeReact components
│   ├── layout/
│   │   ├── AppLayout.tsx
│   │   └── AppSidebar.tsx
│   └── common/
│       ├── ApiErrorMessage.tsx  # Centralized API error display
│       ├── FormField.tsx        # Label + input + error wrapper
│       ├── PageLoading.tsx      # Centered ProgressSpinner
│       ├── SafeHtml.tsx         # Sanitized HTML renderer
│       └── ProtectedRoute.tsx
├── pages/            # Page-level components
│   ├── ggaureg/
│   ├── certreg/
│   └── signdoc/
├── hooks/            # Custom hooks
│   ├── useApiCall.ts         # loading/error/data state for async ops
│   └── useConfirmDialog.ts   # Promise-based confirm wrapper
├── utils/            # Pure utilities
│   ├── safeUrl.ts    # URL validation utility
│   └── constants.ts
├── App.tsx
├── main.tsx
└── vite-env.d.ts
```

---

## 2. API Client — Centralized & Secure (REACT-NET-001)

```typescript
// src/api/client.ts
import axios, { type InternalAxiosRequestConfig } from "axios";
import { getAccessToken } from "../auth/tokenStore";

// ✅ Fixed baseURL — ห้ามให้ user input กำหนด origin
const apiClient = axios.create({
  baseURL: "/api",
  timeout: 30_000,
  headers: {
    "Content-Type": "application/json",
  },
});

// ✅ Attach bearer token automatically (REACT-AUTH-001)
apiClient.interceptors.request.use(
  (config: InternalAxiosRequestConfig) => {
    const token = getAccessToken(); // อ่านจาก in-memory เท่านั้น
    if (token) {
      config.headers.Authorization = `Bearer ${token}`;
    }
    return config;
  },
  (error) => Promise.reject(error)
);

// ✅ Handle auth errors centrally
apiClient.interceptors.response.use(
  (response) => response,
  (error) => {
    if (error.response?.status === 401) {
      // redirect to login or refresh token
      window.location.href = "/login";
    }
    return Promise.reject(error);
  }
);

export default apiClient;
```

---

## 3. Token Store — In-Memory Only (REACT-AUTH-001)

```typescript
// src/auth/tokenStore.ts

// ✅ ห้ามใช้ localStorage/sessionStorage เก็บ token
// ✅ เก็บใน memory เท่านั้น — หายเมื่อปิด tab
let accessToken: string | null = null;

export function getAccessToken(): string | null {
  return accessToken;
}

export function setAccessToken(token: string): void {
  accessToken = token;
}

export function clearAccessToken(): void {
  accessToken = null;
}

// ❌ ห้ามทำแบบนี้:
// localStorage.setItem("token", token);      // XSS สามารถขโมยได้
// sessionStorage.setItem("token", token);    // XSS สามารถขโมยได้
```

---

## 4. URL Validation Utility (REACT-URL-001, JS-URL-001)

```typescript
// src/utils/safeUrl.ts

const ALLOWED_PROTOCOLS = new Set(["https:", "http:"]);

/**
 * Validate URL — ป้องกัน open redirect & javascript: injection
 * ✅ อนุญาตเฉพาะ relative path หรือ same-origin URL
 */
export function safeRedirectUrl(
  input: string,
  fallback = "/"
): string {
  // Allow relative paths starting with /
  if (input.startsWith("/") && !input.startsWith("//")) {
    return input;
  }

  try {
    const parsed = new URL(input, window.location.origin);
    if (
      !ALLOWED_PROTOCOLS.has(parsed.protocol) ||
      parsed.origin !== window.location.origin
    ) {
      return fallback;
    }
    return parsed.pathname + parsed.search + parsed.hash;
  } catch {
    return fallback;
  }
}

// ❌ ห้ามทำแบบนี้:
// window.location.href = params.get("next");   // Open redirect
// <a href={userInput}>                          // javascript: injection
```

---

## 5. Safe HTML Renderer (REACT-XSS-001)

```typescript
// src/components/common/SafeHtml.tsx
import DOMPurify from "dompurify";

interface SafeHtmlProps {
  html: string;
  className?: string;
}

/**
 * ✅ ใช้ DOMPurify sanitize ก่อน render HTML
 * ❌ ห้ามใช้ dangerouslySetInnerHTML โดยตรงกับ untrusted content
 */
export function SafeHtml({ html, className }: SafeHtmlProps) {
  const clean = DOMPurify.sanitize(html, {
    ALLOWED_TAGS: ["b", "i", "em", "strong", "a", "p", "br", "ul", "ol", "li"],
    ALLOWED_ATTR: ["href", "target", "rel"],
  });

  return (
    <div
      className={className}
      dangerouslySetInnerHTML={{ __html: clean }}
    />
  );
}

// ❌ ห้ามทำแบบนี้:
// <div dangerouslySetInnerHTML={{ __html: apiResponse.html }} />
// element.innerHTML = userInput;
```

---

## 6. PrimeReact Component Usage — ggaureg-ui Example

```tsx
// src/pages/ggaureg/GgauregPage.tsx
import { Card } from "primereact/card";
import { Button } from "primereact/button";
import { Divider } from "primereact/divider";
import { useState, useEffect } from "react";
import { PageLoading } from "../../components/common/PageLoading";
import { ApiErrorMessage } from "../../components/common/ApiErrorMessage";
import apiClient from "../../api/client";

interface GgauregPageProps {
  urlToken: string; // ✅ url token ใช้เปิด flow เท่านั้น ไม่ใช่ bearer auth
}

export function GgauregPage({ urlToken }: GgauregPageProps) {
  const [status, setStatus] = useState<"loading" | "valid" | "invalid">("loading");
  const [error, setError] = useState<unknown>(null);

  useEffect(() => {
    // ✅ ใช้ url token เพื่อ validate flow context เท่านั้น
    // ✅ API call ใช้ Bearer token แยกต่างหาก (ผ่าน interceptor)
    apiClient
      .post("/ggaureg/validate", { token: urlToken })
      .then(() => setStatus("valid"))
      .catch((err) => {
        setStatus("invalid");
        // ✅ ห้าม log token หรือ sensitive data
        setError(err);
      });
  }, [urlToken]);

  if (status === "loading") {
    return <PageLoading />;
  }

  return (
    <Card title="Google Authenticator Registration">
      {status === "invalid" && (
        <ApiErrorMessage error={error} fallbackMessage="Validation failed" className="mb-3" />
      )}

      {status === "valid" && (
        <>
          <p>QR Code and enrollment instructions here.</p>
          <Divider />
          <Button
            label="Continue to OTP Verification"
            icon="pi pi-check"
            onClick={() => {
              /* navigate to OTP step */
            }}
          />
        </>
      )}
    </Card>
  );
}
```

---

## 7. FileUpload — ตรวจ Backend Contract ก่อนใช้ (certreg-ui)

```tsx
// src/pages/certreg/CertUpload.tsx
import { useRef, useState } from "react";
import { Button } from "primereact/button";
import { Password } from "primereact/password";
import { Message } from "primereact/message";
import { useAppToast } from "../../providers/AppToastProvider";
import apiClient from "../../api/client";

/**
 * ✅ ไม่ใช้ PrimeReact FileUpload auto-upload
 *    เพราะ backend contract ต้อง:
 *    - Bearer token ใน header
 *    - multipart/form-data format เฉพาะ
 *    - custom error mapping
 *
 * ✅ ใช้ manual upload ควบคุม request เอง
 */
export function CertUpload() {
  const toast = useAppToast();
  const fileInputRef = useRef<HTMLInputElement>(null);
  const [pfxPassword, setPfxPassword] = useState("");
  const [selectedFile, setSelectedFile] = useState<File | null>(null);
  const [uploading, setUploading] = useState(false);

  const handleUpload = async () => {
    if (!selectedFile) return;

    const formData = new FormData();
    formData.append("certificate", selectedFile);
    formData.append("password", pfxPassword);

    setUploading(true);
    try {
      // ✅ Bearer token ถูก attach โดย interceptor อัตโนมัติ
      await apiClient.post("/certreg/upload", formData, {
        headers: { "Content-Type": "multipart/form-data" },
      });

      toast.show({
        severity: "success",
        summary: "Uploaded",
        detail: "Certificate registered successfully",
      });
    } catch (err: unknown) {
      // ✅ แสดง error message จาก backend ไม่ log sensitive data
      const message =
        err instanceof Error ? err.message : "Upload failed";
      toast.show({
        severity: "error",
        summary: "Error",
        detail: message,
      });
    } finally {
      setUploading(false);
    }
  };

  return (
    <div>
      <div className="mb-3">
        <label htmlFor="cert-file" className="block mb-1">
          Certificate File (.pfx / .p12)
        </label>
        <input
          id="cert-file"
          ref={fileInputRef}
          type="file"
          accept=".pfx,.p12"
          onChange={(e) => setSelectedFile(e.target.files?.[0] ?? null)}
        />
      </div>

      <div className="mb-3">
        <label htmlFor="pfx-password" className="block mb-1">
          Certificate Password
        </label>
        <Password
          id="pfx-password"
          value={pfxPassword}
          onChange={(e) => setPfxPassword(e.target.value)}
          feedback={false}
          toggleMask
        />
      </div>

      <Button
        label="Upload Certificate"
        icon="pi pi-upload"
        loading={uploading}
        disabled={!selectedFile || !pfxPassword}
        onClick={handleUpload}
      />
    </div>
  );
}
```

---

## 8. Environment Variables — ห้ามเก็บ Secret (REACT-CONFIG-001)

```bash
# .env — ✅ ค่าที่เปิดเผยได้เท่านั้น
VITE_API_BASE_URL=/api
VITE_APP_TITLE=My App

# ❌ ห้ามทำแบบนี้:
# VITE_API_SECRET=sk-1234567890       # จะถูก bundle เข้า client!
# VITE_JWT_SIGNING_KEY=my-signing-key # ผู้ใช้ทุกคนเห็นได้!
```

---

## 9. Offline-Only Asset Loading

เครื่อง client อาจออก internet ไม่ได้ ทุก asset ต้อง bundle อยู่ใน build output

### ✅ วิธีที่ถูกต้อง — import จาก npm package

```typescript
// src/main.tsx
import 'primereact/resources/themes/lara-light-indigo/theme.css';
import 'primereact/resources/primereact.min.css';
import 'primeicons/primeicons.css';  // ✅ จาก node_modules
import 'primeflex/primeflex.css';    // ✅ จาก node_modules (ถ้าใช้)
```

### ✅ Self-host fonts (ถ้าต้องการ custom font)

```css
/* src/assets/fonts.css */
@font-face {
  font-family: 'Sarabun';
  src: url('/fonts/Sarabun-Regular.woff2') format('woff2');
  font-weight: 400;
  font-style: normal;
  font-display: swap;
}
```

วางไฟล์ font ไว้ที่ `public/fonts/Sarabun-Regular.woff2`

### ❌ ห้ามทำแบบนี้

```html
<!-- ❌ ห้าม load จาก CDN -->
<link href="https://fonts.googleapis.com/css2?family=Sarabun" rel="stylesheet">
<link href="https://unpkg.com/primeicons/primeicons.css" rel="stylesheet">
<script src="https://cdn.jsdelivr.net/npm/primereact/primereact.min.js"></script>
```

```css
/* ❌ ห้าม import จาก external URL */
@import url('https://fonts.googleapis.com/css2?family=Inter');
```

### ✅ ตรวจสอบหลัง build

```bash
# ตรวจว่า dist/ ไม่มี external URL
grep -rn 'https://' dist/ --include='*.html' --include='*.css' --include='*.js' || echo "✅ No external URLs found"

# ทดสอบ offline ด้วย browser DevTools → Network → Offline
```

---

## 10. CSP Meta Tag ขั้นต่ำ (REACT-CSP-001, JS-CSP-001)

```html
<!-- index.html — ใส่ก่อน <script> ทุกตัว -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8" />
    <!-- ✅ CSP baseline — ห้าม unsafe-inline/unsafe-eval ถ้าเป็นไปได้ -->
    <meta
      http-equiv="Content-Security-Policy"
      content="
        default-src 'self';
        script-src 'self';
        style-src 'self' 'unsafe-inline';
        img-src 'self' data:;
        connect-src 'self';
        font-src 'self';
        object-src 'none';
        base-uri 'self';
        form-action 'self';
      "
    />
    <meta name="viewport" content="width=device-width, initial-scale=1.0" />
    <title>My App</title>
  </head>
  <body>
    <div id="root"></div>
    <script type="module" src="/src/main.tsx"></script>
  </body>
</html>
```

> **หมายเหตุ:** `style-src 'unsafe-inline'` จำเป็นสำหรับ PrimeReact styled mode ที่ inject inline styles หากใช้ CSP header จริงให้เพิ่ม `frame-ancestors 'none'` ด้วย (ไม่รองรับใน meta tag)

---

## 11. Review Checklist รวม (React + PrimeReact + Security)

### Stack & Setup
- [ ] ใช้ React + Vite + TypeScript + PrimeReact
- [ ] PrimeReact ใช้ styled mode เท่านั้น (ไม่ซ้อน Tailwind)
- [ ] Pin package versions ใน `package.json`
- [ ] Commit lock file
- [ ] ผ่าน `npm audit` ไม่มี critical/high

### PrimeReact-First
- [ ] ทุก UI element ใช้ PrimeReact component เมื่อมีอยู่ (ไม่เขียน custom ทับ)
- [ ] Custom components follow PrimeReact design tokens
- [ ] ไม่มี thin wrapper ที่แค่ re-export PrimeReact โดยไม่มี logic เพิ่ม

### Code Reuse / DRY
- [ ] Global Toast ใช้ `AppToastProvider` + `useAppToast()` (ไม่มี `useRef<Toast>` ซ้ำทุกหน้า)
- [ ] Loading state ใช้ `<PageLoading />` กลาง
- [ ] Error display ใช้ `<ApiErrorMessage />` กลาง
- [ ] Form fields ใช้ `<FormField />` wrapper
- [ ] Confirm dialog ใช้ `useConfirmDialog()`
- [ ] API call state ใช้ `useApiCall()` hook
- [ ] Form validation ใช้ `react-hook-form` + `zod` (ไม่ใช้ manual useState ต่อ field)
- [ ] ไม่มี code ซ้ำข้ามหน้าที่ควร extract เป็น shared

### Offline-Only Asset Loading
- [ ] ไม่มี `<script src>` หรือ `<link href>` ที่ชี้ไป external CDN
- [ ] PrimeIcons import จาก npm ไม่ใช่ CDN
- [ ] Font files อยู่ใน project (`public/fonts/`) ไม่ใช่ Google Fonts
- [ ] หลัง build ไม่มี `https://` reference ใน dist/
- [ ] ทดสอบ offline mode ใน browser แล้ว app load ได้สมบูรณ์

### Token & Auth
- [ ] `accessToken` เก็บใน memory เท่านั้น (ไม่ใช่ localStorage)
- [ ] `url token` ใช้เปิด flow context เท่านั้น ไม่ใช่ bearer auth
- [ ] `signingSessionId` ใช้เฉพาะ signdoc flow
- [ ] Protected API ส่ง `Authorization: Bearer <token>` ตาม backend contract
- [ ] ไม่ log/persist token, password, OTP ใน console หรือ storage

### XSS Prevention
- [ ] ไม่ใช้ `dangerouslySetInnerHTML` กับ untrusted data โดยไม่ sanitize
- [ ] ไม่ใช้ `innerHTML`, `document.write`, `eval`, `new Function`
- [ ] ใช้ React JSX escaping เป็นหลัก
- [ ] URL จาก user input ผ่าน validation ก่อน navigate

### PrimeReact Components
- [ ] Network-capable components (FileUpload, DataTable lazy) ตรวจ backend contract
- [ ] ไม่ใช้ component library 2 ตัวในหน้าเดียวกัน
- [ ] Auth/token logic อยู่ใน app code ไม่ใช่ใน component

### CSP & Headers
- [ ] มี CSP อย่างน้อย meta tag หรือ HTTP header
- [ ] ไม่ใช้ `unsafe-eval` ใน CSP
- [ ] Third-party scripts มี SRI หรือ self-host
