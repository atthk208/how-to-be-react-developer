# ชั่วโมงที่ 3 · Zustand (Professional Usage)

> เวลา: 60 นาที | ระดับ: Mid-Level

---

## 🎯 เป้าหมาย

เข้าใจการใช้ Zustand อย่าง professional — ตั้งแต่ store พื้นฐาน, Selectors, Slice Pattern, Middleware จนถึง Async Actions และการ Test

---

## 1. ทำไมต้อง Zustand?

| Feature | Zustand | Redux Toolkit | Jotai |
|---------|---------|---------------|-------|
| Boilerplate | น้อยมาก | ปานกลาง | น้อย |
| Bundle size | ~1KB | ~10KB | ~3KB |
| DevTools | ✅ | ✅ | ✅ |
| SSR Support | ✅ | ✅ | ✅ |
| Learning curve | ต่ำ | สูง | ต่ำ |
| Middleware | ✅ | built-in | จำกัด |

---

## 2. Store พื้นฐาน

```bash
npm install zustand
```

```ts
// stores/useCounterStore.ts
import { create } from "zustand";

type CounterState = {
  count: number;
  increment: () => void;
  decrement: () => void;
  reset: () => void;
  incrementBy: (amount: number) => void;
};

export const useCounterStore = create<CounterState>((set) => ({
  count: 0,
  increment: () => set((state) => ({ count: state.count + 1 })),
  decrement: () => set((state) => ({ count: state.count - 1 })),
  reset: () => set({ count: 0 }),
  incrementBy: (amount) => set((state) => ({ count: state.count + amount })),
}));
```

```tsx
// การใช้งานใน Component
function Counter() {
  const count = useCounterStore((state) => state.count);
  const { increment, decrement, reset } = useCounterStore();

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={increment}>+</button>
      <button onClick={decrement}>-</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}
```

---

## 3. Selectors + useShallow (Performance)

```ts
import { useShallow } from "zustand/react/shallow";

// ❌ แบบนี้ re-render ทุกครั้งที่ store เปลี่ยน
const { user, cart } = useStore();

// ✅ ใช้ useShallow ส��หรับ object/array
const { user, cart } = useStore(
  useShallow((state) => ({ user: state.user, cart: state.cart }))
);

// ✅ Select เฉพาะ primitive value — ปลอดภัยไม่ต้อง useShallow
const count = useCounterStore((state) => state.count);

// ✅ Computed selector
const totalItems = useCartStore(
  (state) => state.items.reduce((sum, item) => sum + item.quantity, 0)
);
```

---

## 4. Slice Pattern (Large App)

```ts
// stores/slices/cartSlice.ts
import type { StateCreator } from "zustand";

export type CartItem = {
  id: string;
  name: string;
  price: number;
  quantity: number;
};

export type CartSlice = {
  items: CartItem[];
  addItem: (item: Omit<CartItem, "quantity">) => void;
  removeItem: (id: string) => void;
  updateQuantity: (id: string, quantity: number) => void;
  clearCart: () => void;
  totalPrice: () => number;
};

export const createCartSlice: StateCreator<CartSlice> = (set, get) => ({
  items: [],

  addItem: (newItem) =>
    set((state) => {
      const existing = state.items.find((i) => i.id === newItem.id);
      if (existing) {
        return {
          items: state.items.map((i) =>
            i.id === newItem.id ? { ...i, quantity: i.quantity + 1 } : i
          ),
        };
      }
      return { items: [...state.items, { ...newItem, quantity: 1 }] };
    }),

  removeItem: (id) =>
    set((state) => ({ items: state.items.filter((i) => i.id !== id) })),

  updateQuantity: (id, quantity) =>
    set((state) => ({
      items:
        quantity <= 0
          ? state.items.filter((i) => i.id !== id)
          : state.items.map((i) => (i.id === id ? { ...i, quantity } : i)),
    })),

  clearCart: () => set({ items: [] }),

  totalPrice: () =>
    get().items.reduce((sum, item) => sum + item.price * item.quantity, 0),
});
```

```ts
// stores/slices/userSlice.ts
import type { StateCreator } from "zustand";

export type User = {
  id: string;
  name: string;
  email: string;
  role: "admin" | "user";
};

export type UserSlice = {
  user: User | null;
  isAuthenticated: boolean;
  setUser: (user: User) => void;
  logout: () => void;
};

export const createUserSlice: StateCreator<UserSlice> = (set) => ({
  user: null,
  isAuthenticated: false,
  setUser: (user) => set({ user, isAuthenticated: true }),
  logout: () => set({ user: null, isAuthenticated: false }),
});
```

```ts
// stores/useAppStore.ts
import { create } from "zustand";
import { devtools, persist } from "zustand/middleware";
import { createCartSlice, type CartSlice } from "./slices/cartSlice";
import { createUserSlice, type UserSlice } from "./slices/userSlice";

type AppStore = CartSlice & UserSlice;

export const useAppStore = create<AppStore>()(
  devtools(
    persist(
      (...args) => ({
        ...createCartSlice(...args),
        ...createUserSlice(...args),
      }),
      {
        name: "app-storage",
        partialize: (state) => ({
          // persist เฉพาะ cart items และ user
          items: state.items,
          user: state.user,
          isAuthenticated: state.isAuthenticated,
        }),
      }
    ),
    { name: "AppStore" }
  )
);
```

---

## 5. Middleware

### 5.1 persist (บันทึกลง localStorage)

```ts
import { create } from "zustand";
import { persist, createJSONStorage } from "zustand/middleware";

const useSettingsStore = create(
  persist(
    (set) => ({
      theme: "light" as "light" | "dark",
      language: "th",
      setTheme: (theme: "light" | "dark") => set({ theme }),
      setLanguage: (language: string) => set({ language }),
    }),
    {
      name: "settings-storage",
      storage: createJSONStorage(() => sessionStorage), // หรือ localStorage
      version: 1,
      migrate: (persistedState, version) => {
        // handle migration เมื่อ schema เปลี่ยน
        return persistedState;
      },
    }
  )
);
```

### 5.2 devtools

```ts
import { devtools } from "zustand/middleware";

const useStore = create<State>()(
  devtools(
    (set) => ({
      bears: 0,
      addBear: () => set((state) => ({ bears: state.bears + 1 }), false, "addBear"),
      //                                                              ↑ replace  ↑ action name
    }),
    { name: "BearStore", enabled: process.env.NODE_ENV === "development" }
  )
);
```

### 5.3 immer (Mutable Updates)

```bash
npm install immer
```

```ts
import { immer } from "zustand/middleware/immer";

type State = {
  user: { name: string; address: { city: string; zip: string } };
  updateCity: (city: string) => void;
};

const useStore = create<State>()(
  immer((set) => ({
    user: { name: "Alice", address: { city: "Bangkok", zip: "10100" } },

    // ✅ กับ immer เขียนแบบ mutable ได้เลย
    updateCity: (city) =>
      set((state) => {
        state.user.address.city = city;
      }),

    // ❌ ไม่ใช้ immer ต้อง spread ทุกระดับ
    // updateCity: (city) =>
    //   set((state) => ({
    //     user: { ...state.user, address: { ...state.user.address, city } },
    //   })),
  }))
);
```

---

## 6. Async Actions

```ts
// stores/useProductStore.ts
import { create } from "zustand";
import { devtools } from "zustand/middleware";

type Product = {
  id: number;
  title: string;
  price: number;
  image: string;
};

type ProductState = {
  products: Product[];
  selectedProduct: Product | null;
  isLoading: boolean;
  error: string | null;

  fetchProducts: () => Promise<void>;
  fetchProduct: (id: number) => Promise<void>;
  createProduct: (data: Omit<Product, "id">) => Promise<void>;
};

export const useProductStore = create<ProductState>()(
  devtools(
    (set, get) => ({
      products: [],
      selectedProduct: null,
      isLoading: false,
      error: null,

      fetchProducts: async () => {
        set({ isLoading: true, error: null });
        try {
          const res = await fetch("/api/products");
          if (!res.ok) throw new Error("Failed to fetch products");
          const data: Product[] = await res.json();
          set({ products: data, isLoading: false });
        } catch (err) {
          set({
            error: err instanceof Error ? err.message : "Unknown error",
            isLoading: false,
          });
        }
      },

      fetchProduct: async (id) => {
        // ตรวจสอบ cache ก่อน
        const cached = get().products.find((p) => p.id === id);
        if (cached) {
          set({ selectedProduct: cached });
          return;
        }

        set({ isLoading: true, error: null });
        try {
          const res = await fetch(`/api/products/${id}`);
          if (!res.ok) throw new Error("Product not found");
          const data: Product = await res.json();
          set({ selectedProduct: data, isLoading: false });
        } catch (err) {
          set({
            error: err instanceof Error ? err.message : "Unknown error",
            isLoading: false,
          });
        }
      },

      // Optimistic Update
      createProduct: async (data) => {
        const tempId = Date.now();
        const optimisticProduct = { ...data, id: tempId };

        // 1. อัปเดต UI ก่อน
        set((state) => ({ products: [...state.products, optimisticProduct] }));

        try {
          const res = await fetch("/api/products", {
            method: "POST",
            body: JSON.stringify(data),
            headers: { "Content-Type": "application/json" },
          });
          const created: Product = await res.json();

          // 2. แทนที่ด้วยข้อมูลจริง
          set((state) => ({
            products: state.products.map((p) =>
              p.id === tempId ? created : p
            ),
          }));
        } catch {
          // 3. Rollback ถ้า error
          set((state) => ({
            products: state.products.filter((p) => p.id !== tempId),
            error: "Failed to create product",
          }));
        }
      },
    }),
    { name: "ProductStore" }
  )
);
```

---

## 7.ใช้งานนอก Component (Vanilla)

```ts
// ใช้ได้นอก React Component เช่น ใน API helper, utility function
import { useAppStore } from "@/stores/useAppStore";

// อ่านค่า
const { user, items } = useAppStore.getState();

// อัปเดตค่า
useAppStore.setState({ items: [] });

// Subscribe to changes
const unsubscribe = useAppStore.subscribe(
  (state) => state.items,
  (items) => {
    console.log("Cart updated:", items.length, "items");
  }
);

// อย่าลืม unsubscribe เมื่อไม่ใช้แล้ว
unsubscribe();
```

---

## 8. Testing

```ts
// stores/__tests__/useCartStore.test.ts
import { renderHook, act } from "@testing-library/react";
import { useAppStore } from "../useAppStore";

describe("CartSlice", () => {
  beforeEach(() => {
    // Reset store ก่อนแต่ละ test
    useAppStore.setState({ items: [] });
  });

  it("should add item to cart", () => {
    const { result } = renderHook(() => useAppStore());

    act(() => {
      result.current.addItem({ id: "1", name: "Product A", price: 100 });
    });

    expect(result.current.items).toHaveLength(1);
    expect(result.current.items[0]).toMatchObject({
      id: "1",
      name: "Product A",
      price: 100,
      quantity: 1,
    });
  });

  it("should increase quantity for duplicate item", () => {
    const { result } = renderHook(() => useAppStore());

    act(() => {
      result.current.addItem({ id: "1", name: "Product A", price: 100 });
      result.current.addItem({ id: "1", name: "Product A", price: 100 });
    });

    expect(result.current.items).toHaveLength(1);
    expect(result.current.items[0].quantity).toBe(2);
  });

  it("should calculate total price correctly", () => {
    const { result } = renderHook(() => useAppStore());

    act(() => {
      result.current.addItem({ id: "1", name: "A", price: 100 });
      result.current.addItem({ id: "2", name: "B", price: 200 });
      result.current.addItem({ id: "1", name: "A", price: 100 }); // quantity: 2
    });

    expect(result.current.totalPrice()).toBe(400); // 100*2 + 200*1
  });
});
```

---

## 🏋️ Workshop (15 นาที)

1. สร้าง `useCartStore` ด้วย Zustand ที่มี: `addItem`, `removeItem`, `clearCart`, `totalPrice`
2. เพิ่ม `devtools` และ `persist` middleware
3. สร้าง `CartBadge` component ที่แสดงจำนวน items โดยใช้ Selector แบบ optimized
4. เขียน unit test สำหรับ `addItem` และ `totalPrice`

---

## 📌 สรุป

| หัวข้อ | สิ่งที่ควรจำ |
|--------|------------|
| Selectors | ใช้ `(state) => state.value` เสมอ ลด re-render |
| useShallow | ใช้เมื่อ select object/array หลายค่าพร้อมกัน |
| Slice Pattern | แยก logic ออกเป็น slice เมื่อ store ใหญ่ขึ้น |
| persist | บันทึก state ลง localStorage/sessionStorage |
| devtools | debug ง่ายด้วย Redux DevTools Extension |
| immer | เขียน update แบบ mutable ได้ อ่านง่ายขึ้น |
| Async Actions | จัดการ loading/error ใน action โดยตรง |
| Optimistic Update | อัปเดต UI ก่อน แล้ว rollback ถ้า API ล้มเหลว |
| getState() | ใช้ Zustand นอก React Component ได้ |

---

[← ชั่วโมงที่ 2](./hour-02-react-architecture.md) | [ชั่วโมงที่ 4–5 →](./hour-04-05-nextjs.md) | [← กลับหน้าหลัก](./README.md)