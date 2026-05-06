# Next.js + PrimeReact coding standards example

Use these examples as patterns, not as copy-paste mandates. They translate the team structure into production-oriented code.

## 1. Top-level route-group layout for PrimeReact

```tsx
// src/app/(app-shell)/layout.tsx
import { PrimeReactProvider } from "primereact/api";
import "primereact/resources/themes/lara-light-blue/theme.css";
import "primereact/resources/primereact.min.css";
import "primeicons/primeicons.css";
import "primeflex/primeflex.css";

export default function AppShellLayout({
  children,
}: Readonly<{ children: React.ReactNode }>) {
  return <PrimeReactProvider value={{}}>{children}</PrimeReactProvider>;
}
```

Keep PrimeReact provider and theme ownership in one layout entrypoint rather than importing theme resources ad hoc across pages.

## 2. Centralized API client in the team structure

```ts
// src/resources/customer/lib/api-client.ts
import axios from "axios";

export const apiClient = axios.create({
  baseURL: process.env.NEXT_PUBLIC_API_BASE_URL,
  timeout: 30_000,
  headers: {
    "Content-Type": "application/json",
  },
});
```

Use environment variables instead of hardcoded `http://localhost:8080` values in page files.

## 3. Service module separated from route files

```ts
// src/resources/customer/services/customer.service.ts
import { apiClient } from "../lib/api-client";
import type { Customer } from "../models/customer";

export async function findCustomers(params?: {
  firstName?: string;
  lastName?: string;
}): Promise<Customer[]> {
  const response = await apiClient.get<Customer[]>("/customers", {
    params: params ?? {},
  });
  return response.data;
}
```

Use services for domain operations. Keep route files thin.

## 4. Route file as orchestration layer

```tsx
// src/app/customer/page.tsx
import { CustomerPage } from "@/resources/customer/components/customer-page";
import { findCustomers } from "@/resources/customer/services/customer.service";

export default async function Page() {
  const customers = await findCustomers();
  return <CustomerPage initialCustomers={customers} />;
}
```

This keeps route composition in `src/app` and domain behavior in `src/resources/customer`.

## 5. Keep the client boundary small

```tsx
// src/resources/customer/components/customer-page.tsx
"use client";

import { DataTable } from "primereact/datatable";
import { Column } from "primereact/column";
import type { Customer } from "../models/customer";

type CustomerPageProps = {
  initialCustomers: Customer[];
};

export function CustomerPage({ initialCustomers }: CustomerPageProps) {
  return (
    <DataTable value={initialCustomers}>
      <Column field="firstName" header="First name" />
      <Column field="lastName" header="Last name" />
    </DataTable>
  );
}
```

Keep the route entrypoint on the server and move only the interactive PrimeReact surface into a client component.

## 6. Production form with RHF + zod + PrimeReact

```tsx
// src/resources/customer/components/customer-form.tsx
"use client";

import { zodResolver } from "@hookform/resolvers/zod";
import { Button } from "primereact/button";
import { InputText } from "primereact/inputtext";
import { useForm } from "react-hook-form";
import { z } from "zod";

const customerSchema = z.object({
  firstName: z.string().min(1, "First name is required"),
  lastName: z.string().min(1, "Last name is required"),
});

type CustomerFormData = z.infer<typeof customerSchema>;

export function CustomerForm() {
  const {
    register,
    handleSubmit,
    formState: { errors },
  } = useForm<CustomerFormData>({
    resolver: zodResolver(customerSchema),
  });

  return (
    <form onSubmit={handleSubmit(async (data) => console.log(data))}>
      <label htmlFor="firstName">First name</label>
      <InputText id="firstName" {...register("firstName")} />
      {errors.firstName?.message}

      <label htmlFor="lastName">Last name</label>
      <InputText id="lastName" {...register("lastName")} />
      {errors.lastName?.message}

      <Button type="submit" label="Save" />
    </form>
  );
}
```

In production code, replace repeated field layout with a shared `FormField` once the pattern appears more than once.

## 7. Mutation path guidance

```ts
// src/resources/customer/services/create-customer.ts
"use server";

import { apiClient } from "../lib/api-client";
import type { Customer } from "../models/customer";

export async function createCustomer(input: {
  firstName: string;
  lastName: string;
}): Promise<Customer> {
  const response = await apiClient.post<Customer>("/customers", input);
  return response.data;
}
```

Keep mutation logic in a domain-oriented service or server action instead of embedding it directly in page files.

## 8. What this repo gets right

- App Router routes stay inside `src/app`
- PrimeReact provider is owned by a route-group layout
- reusable models and services already exist under `src/resources/standard`
- later chapters move toward stronger form patterns

## 9. What this repo should not normalize into the standard

- direct `axios` calls inside multiple page files
- repeated hardcoded backend URLs
- duplicated inline message and error extraction logic
- manual field-by-field state used outside tutorial contexts

## 10. Refactor direction

When you see a page doing too much:

1. move API and domain logic into `src/resources/<group>/services`
2. move client and mapping helpers into `src/resources/<group>/lib`
3. move models and DTO types into `src/resources/<group>/models`
4. keep the route file focused on composition and page assembly
