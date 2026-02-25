# ชั่วโมงที่ 1 · TypeScript for React (แบบใช้งานจริง)

> เวลา: 60 นาที | ระดับ: Mid-Level

---

## 🎯 เป้าหมาย

เขียน TypeScript ใน React ได้อย่างถูกต้อง ไม่ใช้ `any` โดยไม่จำเป็น และเข้าใจ type patterns ที่ใช้จริงในงาน

---

## 1. Type vs Interface

```ts
// ใช้ interface สำหรับ Object Shapes (extendable)
interface User {
  id: string;
  name: string;
  email: string;
}

interface AdminUser extends User {
  role: "admin" | "super_admin";
  permissions: string[];
}

// ใช้ type สำหรับ Union / Computed Types
type Status = "idle" | "loading" | "success" | "error";
type UserOrAdmin = User | AdminUser;
type UserKeys = keyof User; // "id" | "name" | "email"
```

---

## 2. Generic Components

```tsx
type ListProps<T> = {
  items: T[];
  renderItem: (item: T) => React.ReactNode;
  keyExtractor: (item: T) => string;
  emptyMessage?: string;
};

function List<T>({ items, renderItem, keyExtractor, emptyMessage = "No items" }: ListProps<T>) {
  if (items.length === 0) return <p>{emptyMessage}</p>;
  return (
    <ul>
      {items.map((item) => (
        <li key={keyExtractor(item)}>{renderItem(item)}</li>
      ))}
    </ul>
  );
}
```

---

## 3. Utility Types

```ts
interface Product {
  id: string;
  name: string;
  price: number;
  description: string;
  imageUrl: string;
  category: string;
}

type ProductCard = Pick<Product, "id" | "name" | "price" | "imageUrl">;
type ProductUpdate = Partial<Omit<Product, "id">>;
type ReadonlyProduct = Readonly<Product>;
type ProductMap = Record<string, Product>;
type Status = "idle" | "loading" | "success" | "error";
type StatusColors = Record<Status, string>;

const statusColors: StatusColors = {
  idle: "gray",
  loading: "blue",
  success: "green",
  error: "red",
};
```

---

## 4. Discriminated Union

```ts
type AsyncState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "error"; error: string };

function UserProfile({ state }: { state: AsyncState<User> }) {
  if (state.status === "loading") return <Spinner />;
  if (state.status === "error") return <ErrorMessage message={state.error} />;
  if (state.status === "idle") return null;
  return <div>{state.data.name}</div>;
}
```

---

## 5. Event & Async Typing

```tsx
const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
  setValue(e.target.value);
};

const handleSubmit = (e: React.FormEvent<HTMLFormElement>) => {
  e.preventDefault();
};

async function fetchUser(id: string): Promise<User> {
  const res = await fetch(`/api/users/${id}`);
  if (!res.ok) throw new Error("Failed to fetch user");
  return res.json() as Promise<User>;
}
```

---

## 6. Component Props Patterns

```tsx
type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement> & {
  variant?: "primary" | "secondary" | "danger";
  isLoading?: boolean;
  leftIcon?: React.ReactNode;
};

function Button({ variant = "primary", isLoading, leftIcon, children, ...props }: ButtonProps) {
  return (
    <button disabled={isLoading} {...props}>
      {isLoading ? <Spinner /> : leftIcon}
      {children}
    </button>
  );
}
}
```

---

## 🏋️ Workshop (15 นาที)

1. สร้าง Generic `Table<T>` component ที่รับ `columns` และ `data`
2. สร้าง `AsyncState<T>` และใช้กับ `useQuery` hook
3. Refactor component ที่มี `any` ให้มี type ที่ถูกต้อง

---

## 📌 สรุป

| หัวข้อ | สิ่งที่ควรจำ |
|--------|------------|
| Type vs Interface | interface สำหรับ object, type สำหรับ union |
| Generic Component | `<T>` ทำให้ component ใช้ได้กับทุก type |
| Utility Types | Pick, Omit, Partial, Record |
| Discriminated Union | status field เพื่อ narrow type |
| Event Typing | ใช้ `React.ChangeEvent<HTMLInputElement>` |

---

[ต่อไป → ชั่วโมงที่ 2](./hour-02-react-architecture.md) | [← กลับหน้าหลัก](./README.md)