# Shared patterns — centralized components, hooks, and providers

Every project must have these shared patterns instead of duplicating logic per page.

## 1. Global Toast — `AppToastProvider` + `useAppToast()`

Problem: every page creates its own `useRef<Toast>(null)` and `<Toast ref={toast} />`.

Solution: one `<Toast>` in the layout, every page calls `useAppToast().show()`.

```tsx
// src/providers/AppToastProvider.tsx
import { createContext, useContext, useRef, type ReactNode } from "react";
import { Toast, type ToastMessage } from "primereact/toast";

interface AppToastContextValue {
  show: (message: ToastMessage | ToastMessage[]) => void;
  clear: () => void;
}

const AppToastContext = createContext<AppToastContextValue | null>(null);

export function AppToastProvider({ children }: { children: ReactNode }) {
  const toastRef = useRef<Toast>(null);

  const value: AppToastContextValue = {
    show: (message) => toastRef.current?.show(message),
    clear: () => toastRef.current?.clear(),
  };

  return (
    <AppToastContext.Provider value={value}>
      <Toast ref={toastRef} />
      {children}
    </AppToastContext.Provider>
  );
}

export function useAppToast(): AppToastContextValue {
  const context = useContext(AppToastContext);
  if (!context) {
    throw new Error("useAppToast must be used inside AppToastProvider");
  }
  return context;
}
```

Usage in any page:

```tsx
import { useAppToast } from "../../providers/AppToastProvider";

function SomePage() {
  const toast = useAppToast();

  const handleSave = async () => {
    try {
      await apiClient.post("/save", data);
      toast.show({ severity: "success", summary: "Saved", detail: "Done" });
    } catch {
      toast.show({ severity: "error", summary: "Error", detail: "Save failed" });
    }
  };
}
```

Wire in `main.tsx`:

```tsx
// src/main.tsx
import { AppToastProvider } from "./providers/AppToastProvider";

createRoot(document.getElementById("root")!).render(
  <StrictMode>
    <AppToastProvider>
      <App />
    </AppToastProvider>
  </StrictMode>
);
```

---

## 2. Page loading — `<PageLoading />`

Problem: every page repeats `<div className="flex justify-content-center p-5"><ProgressSpinner /></div>`.

```tsx
// src/components/common/PageLoading.tsx
import { ProgressSpinner } from "primereact/progressspinner";

interface PageLoadingProps {
  message?: string;
}

export function PageLoading({ message }: PageLoadingProps) {
  return (
    <div className="flex flex-column align-items-center justify-content-center p-5">
      <ProgressSpinner />
      {message && <p className="mt-3 text-color-secondary">{message}</p>}
    </div>
  );
}
```

---

## 3. API error display — `<ApiErrorMessage />`

Problem: every page repeats Axios error extraction + `<Message severity="error">`.

```tsx
// src/components/common/ApiErrorMessage.tsx
import { Message } from "primereact/message";
import { type AxiosError } from "axios";

interface ApiErrorMessageProps {
  error: unknown;
  fallbackMessage?: string;
  className?: string;
}

export function ApiErrorMessage({
  error,
  fallbackMessage = "An unexpected error occurred",
  className,
}: ApiErrorMessageProps) {
  const message = extractErrorMessage(error, fallbackMessage);
  return <Message severity="error" text={message} className={className} />;
}

function extractErrorMessage(error: unknown, fallback: string): string {
  if (!error) return fallback;

  // Axios error with response body
  const axiosErr = error as AxiosError<{ message?: string }>;
  if (axiosErr.response?.data?.message) {
    return axiosErr.response.data.message;
  }

  // Generic Error
  if (error instanceof Error) {
    return error.message;
  }

  return fallback;
}
```

---

## 4. Form field wrapper — `<FormField />`

Problem: every form repeats `<label>`, input slot, and validation error message layout.

Works with any PrimeReact input passed as `children`. Integrates with `react-hook-form` error messages.

```tsx
// src/components/common/FormField.tsx
import { type ReactNode } from "react";

interface FormFieldProps {
  label: string;
  htmlFor: string;
  error?: string;
  required?: boolean;
  className?: string;
  children: ReactNode;
}

export function FormField({
  label,
  htmlFor,
  error,
  required,
  className,
  children,
}: FormFieldProps) {
  return (
    <div className={`field mb-3 ${className ?? ""}`}>
      <label htmlFor={htmlFor} className="block mb-1 font-medium">
        {label}
        {required && <span className="text-red-500 ml-1">*</span>}
      </label>
      {children}
      {error && (
        <small className="p-error block mt-1">{error}</small>
      )}
    </div>
  );
}
```

Usage with `react-hook-form`:

```tsx
import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";
import { InputText } from "primereact/inputtext";
import { FormField } from "../../components/common/FormField";

const schema = z.object({
  name: z.string().min(1, "Name is required"),
  email: z.string().email("Invalid email"),
});

type FormValues = z.infer<typeof schema>;

function ExampleForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<FormValues>({
    resolver: zodResolver(schema),
  });

  const onSubmit = (data: FormValues) => {
    // submit data
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <FormField label="Name" htmlFor="name" error={errors.name?.message} required>
        <InputText id="name" {...register("name")} className="w-full" />
      </FormField>

      <FormField label="Email" htmlFor="email" error={errors.email?.message} required>
        <InputText id="email" {...register("email")} className="w-full" />
      </FormField>
    </form>
  );
}
```

---

## 5. Confirm dialog — `useConfirmDialog()`

Problem: every destructive action repeats `ConfirmDialog` setup.

```tsx
// src/hooks/useConfirmDialog.ts
import { confirmDialog } from "primereact/confirmdialog";

interface ConfirmOptions {
  header?: string;
  message: string;
  acceptLabel?: string;
  rejectLabel?: string;
  icon?: string;
}

export function useConfirmDialog() {
  return {
    confirm: (options: ConfirmOptions): Promise<boolean> => {
      return new Promise((resolve) => {
        confirmDialog({
          header: options.header ?? "Confirm",
          message: options.message,
          icon: options.icon ?? "pi pi-exclamation-triangle",
          acceptLabel: options.acceptLabel ?? "Yes",
          rejectLabel: options.rejectLabel ?? "No",
          accept: () => resolve(true),
          reject: () => resolve(false),
        });
      });
    },
  };
}
```

Wire `<ConfirmDialog />` once in the layout:

```tsx
// src/App.tsx
import { ConfirmDialog } from "primereact/confirmdialog";

function App() {
  return (
    <>
      <ConfirmDialog />
      {/* routes */}
    </>
  );
}
```

Usage in any page:

```tsx
import { useConfirmDialog } from "../../hooks/useConfirmDialog";

function SomePage() {
  const { confirm } = useConfirmDialog();

  const handleDelete = async () => {
    const ok = await confirm({
      message: "Are you sure you want to delete this certificate?",
      header: "Delete Certificate",
    });
    if (ok) {
      await apiClient.delete(`/certreg/${id}`);
    }
  };
}
```

---

## 6. API call state — `useApiCall()`

Problem: every page repeats `const [loading, setLoading] = useState(false); const [error, setError] = useState(null); ...`

```tsx
// src/hooks/useApiCall.ts
import { useState, useCallback } from "react";

interface ApiCallState<T> {
  data: T | null;
  loading: boolean;
  error: unknown;
}

interface UseApiCallReturn<T> extends ApiCallState<T> {
  execute: (...args: unknown[]) => Promise<T | null>;
  reset: () => void;
}

export function useApiCall<T>(
  apiFunction: (...args: unknown[]) => Promise<T>
): UseApiCallReturn<T> {
  const [state, setState] = useState<ApiCallState<T>>({
    data: null,
    loading: false,
    error: null,
  });

  const execute = useCallback(
    async (...args: unknown[]): Promise<T | null> => {
      setState({ data: null, loading: true, error: null });
      try {
        const result = await apiFunction(...args);
        setState({ data: result, loading: false, error: null });
        return result;
      } catch (error) {
        setState({ data: null, loading: false, error });
        return null;
      }
    },
    [apiFunction]
  );

  const reset = useCallback(() => {
    setState({ data: null, loading: false, error: null });
  }, []);

  return { ...state, execute, reset };
}
```

Usage:

```tsx
import { useApiCall } from "../../hooks/useApiCall";
import apiClient from "../../api/client";

function CertListPage() {
  const { data: certs, loading, error, execute } = useApiCall(
    () => apiClient.get("/certreg/list").then((r) => r.data)
  );

  useEffect(() => { execute(); }, [execute]);

  if (loading) return <PageLoading />;
  if (error) return <ApiErrorMessage error={error} />;

  return <DataTable value={certs} /* ... */ />;
}
```

---

## 7. Form with react-hook-form + zod — full pattern

Standard form pattern that replaces manual `useState` per field.

```tsx
// src/pages/certreg/CertForm.tsx
import { useForm, Controller } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";
import { InputText } from "primereact/inputtext";
import { Password } from "primereact/password";
import { Button } from "primereact/button";
import { FormField } from "../../components/common/FormField";
import { useAppToast } from "../../providers/AppToastProvider";
import apiClient from "../../api/client";

const certFormSchema = z.object({
  alias: z.string().min(1, "Certificate alias is required"),
  password: z.string().min(1, "Password is required"),
});

type CertFormValues = z.infer<typeof certFormSchema>;

interface CertFormProps {
  onSuccess: () => void;
  file: File;
}

export function CertForm({ onSuccess, file }: CertFormProps) {
  const toast = useAppToast();
  const {
    register,
    handleSubmit,
    control,
    formState: { errors, isSubmitting },
  } = useForm<CertFormValues>({
    resolver: zodResolver(certFormSchema),
    defaultValues: { alias: "", password: "" },
  });

  const onSubmit = async (values: CertFormValues) => {
    const formData = new FormData();
    formData.append("certificate", file);
    formData.append("alias", values.alias);
    formData.append("password", values.password);

    try {
      await apiClient.post("/certreg/upload", formData, {
        headers: { "Content-Type": "multipart/form-data" },
      });
      toast.show({ severity: "success", summary: "Uploaded", detail: "Certificate registered" });
      onSuccess();
    } catch (err) {
      toast.show({ severity: "error", summary: "Error", detail: "Upload failed" });
    }
  };

  return (
    <form onSubmit={handleSubmit(onSubmit)}>
      <FormField label="Certificate Alias" htmlFor="alias" error={errors.alias?.message} required>
        <InputText id="alias" {...register("alias")} className="w-full" />
      </FormField>

      <FormField label="Certificate Password" htmlFor="password" error={errors.password?.message} required>
        <Controller
          name="password"
          control={control}
          render={({ field }) => (
            <Password
              id="password"
              {...field}
              feedback={false}
              toggleMask
              className="w-full"
            />
          )}
        />
      </FormField>

      <Button
        type="submit"
        label="Upload Certificate"
        icon="pi pi-upload"
        loading={isSubmitting}
      />
    </form>
  );
}
```

### When to use `register` vs `Controller`

- **`register`** — works with native HTML inputs and PrimeReact components that accept `name`, `value`, `onChange`, `onBlur` via spread (e.g., `InputText`, `InputTextarea`).
- **`Controller`** — required for PrimeReact components with custom value/onChange signatures (e.g., `Password`, `Dropdown`, `Calendar`, `InputNumber`, `MultiSelect`, `AutoComplete`).

---

## Naming conventions

| Type | Location | Naming |
|---|---|---|
| Shared component | `src/components/common/` | PascalCase: `PageLoading.tsx`, `FormField.tsx` |
| Layout component | `src/components/layout/` | PascalCase: `AppLayout.tsx`, `AppSidebar.tsx` |
| Provider | `src/providers/` | PascalCase: `AppToastProvider.tsx` |
| Custom hook | `src/hooks/` | camelCase with `use` prefix: `useApiCall.ts`, `useConfirmDialog.ts` |
| Page component | `src/pages/<feature>/` | PascalCase: `GgauregPage.tsx`, `CertForm.tsx` |
| API module | `src/api/` | camelCase: `client.ts`, `endpoints.ts` |
| Utility | `src/utils/` | camelCase: `safeUrl.ts`, `constants.ts` |
| Zod schema | Co-located with form or in `src/schemas/` | camelCase with `Schema` suffix: `certFormSchema` |
