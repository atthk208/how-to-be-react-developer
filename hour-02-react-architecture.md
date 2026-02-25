# ชั่วโมงที่ 2 · React Architecture

> เวลา: 60 นาที | ระดับ: Mid-Level

---

## 🎯 เป้าหมาย

ออกแบบโครงสร้าง React project ให้ scale ได้ บำรุงรักษาง่าย และ reuse code ได้สูง

---

## 1. Feature-Based Folder Structure

```
src/
├── app/                    # Next.js App Router pages
├── components/             # Shared UI components
│   ├── ui/                 # Base: Button, Input, Modal
│   └── layout/             # Header, Footer, Sidebar
├── features/               # Feature modules
│   ├── auth/
│   │   ├── components/     # LoginForm, RegisterForm
│   │   ├── hooks/          # useAuth, useSession
│   │   ├── store/          # authStore.ts
│   │   ├── api/            # auth.api.ts
│   │   └── types/          # auth.types.ts
│   ├── products/
│   └── cart/
├── hooks/                  # Global hooks
├── lib/                    # Utils, helpers, configs
├── store/                  # Global Zustand stores
└── types/                  # Global TypeScript types
```

> **กฎ:** สิ่งที่เกี่ยวกับ feature เดียว → อยู่ใน `features/[name]/`

---

## 2. Container / Presentational Pattern

```tsx
// Presentational - รับ props, render UI, ไม่มี business logic
type UserCardProps = {
  name: string;
  email: string;
  avatarUrl: string;
  onFollow: () => void;
  isFollowing: boolean;
};

function UserCard({ name, email, avatarUrl, onFollow, isFollowing }: UserCardProps) {
  return (
    <div className="card">
      <img src={avatarUrl} alt={name} />
      <h3>{name}</h3>
      <p>{email}</p>
      <button onClick={onFollow}>
        {isFollowing ? "Unfollow" : "Follow"}
      </button>
    </div>
  );
}

// Container - จัดการ data, state, business logic
function UserCardContainer({ userId }: { userId: string }) {
  const { user, isLoading } = useUser(userId);
  const { follow, isFollowing } = useFollowUser(userId);

  if (isLoading) return <UserCardSkeleton />;
  if (!user) return null;

  return <UserCard {...user} onFollow={follow} isFollowing={isFollowing} />;
}
```

---

## 3. Compound Components Pattern

```tsx
type CardContextValue = { isExpanded: boolean; toggle: () => void };
const CardContext = createContext<CardContextValue | null>(null);

function useCardContext() {
  const ctx = useContext(CardContext);
  if (!ctx) throw new Error("useCardContext must be used within Card");
  return ctx;
}

function Card({ children }: { children: React.ReactNode }) {
  const [isExpanded, setIsExpanded] = useState(false);
  return (
    <CardContext.Provider value={{ isExpanded, toggle: () => setIsExpanded((p) => !p) }}>
      <div className="card">{children}</div>
    </CardContext.Provider>
  );
}

Card.Header = function CardHeader({ children }: { children: React.ReactNode }) {
  const { toggle } = useCardContext();
  return <div className="card-header" onClick={toggle}>{children}</div>;
};

Card.Body = function CardBody({ children }: { children: React.ReactNode }) {
  const { isExpanded } = useCardContext();
  if (!isExpanded) return null;
  return <div className="card-body">{children}</div>;
};

// Usage
// <Card>
//   <Card.Header>Click to expand</Card.Header>
//   <Card.Body>Content here</Card.Body>
// </Card>
```

---

## 4. Error Boundary

```tsx
type ErrorBoundaryState = { hasError: boolean; error: Error | null }; 

class ErrorBoundary extends React.Component<
  { children: React.ReactNode; fallback?: React.ReactNode },
  ErrorBoundaryState
> {
  state: ErrorBoundaryState = { hasError: false, error: null }; 

  static getDerivedStateFromError(error: Error): ErrorBoundaryState {
    return { hasError: true, error };
  }

  componentDidCatch(error: Error, info: React.ErrorInfo) {
    console.error("Error boundary caught:", error, info);
  }

  render() {
    if (this.state.hasError) {
      return this.props.fallback ?? <div>Something went wrong.</div>;
    }
    return this.props.children;
  }
}
```

---

## 5. Performance Patterns

```tsx
// memo - ป้องกัน re-render เมื่อ props ไม่เปลี่ยน
const ExpensiveList = React.memo(function ExpensiveList({ items }: { items: Item[] }) {
  return <ul>{items.map((i) => <li key={i.id}>{i.name}</li>)}</ul>;
});

// useCallback - stable function reference
const handleClick = useCallback(() => {
  setCount((c) => c + 1);
}, []);

// useMemo - cache computed value
const filtered = useMemo(
  () => products.filter((p) => p.name.toLowerCase().includes(search.toLowerCase())),
  [products, search]
);
```

---

## 🏋️ Workshop (15 นาที)

1. Refactor `ProductPage` ให้แยก Container / Presentational
2. สร้าง `Accordion` ด้วย Compound Components pattern
3. เพิ่ม Error Boundary รอบ `ProductList`

---

## 📌 สรุป

| Pattern | เมื่อไหร่ควรใช้ |
|---------|---------------|
| Feature-based structure | Project ขนาดกลาง-ใหญ่ |
| Container/Presentational | แยก logic กับ UI |
| Compound Components | Components ที่มี sub-parts |
| Error Boundary | ป้องกัน UI crash |
| memo/useCallback/useMemo | เมื่อมี performance ปัญหา |

---

[← ชั่วโมงที่ 1](./hour-01-typescript.md) | [ต่อไป → ชั่วโมงที่ 3](./hour-03-zustand.md) | [← กลับหน้าหลัก](./README.md)