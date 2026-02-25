# ชั่วโมงที่ 4–5 · Next.js (Mid-Level Understanding)

> เวลา: 120 นาที (สองชั่วโมง) | ระดับ: Mid-Level

---

## 🎯 เป้าหมาย

เข้าใจ Next.js App Router, ความต่างระหว่าง Server & Client components, กลยุทธ์ data fetching (SSG/SSR/ISR), API Routes, Server Actions, middleware และแนวปฏิบัติเมื่อ deploy ใน production

---

## 1. สถาปัตยกรรม App Router

- โฟลเดอร์ `app/` เป็น root ของ routing
- ทุกไฟล์ `page.tsx` ในโฟลเดอร์ย่อยจะกลายเป็น route
- `layout.tsx` ใช้สำหรับ shared UI (persistent layout)
- `loading.tsx`, `error.tsx`, `not-found.tsx` สำหรับ states ของ route

```text
app/
├─ layout.tsx        # shared layout
├─ page.tsx          # / (root)
├─ dashboard/
│  ├─ layout.tsx     # dashboard layout
│  └─ page.tsx       # /dashboard
└─ api/
   └─ hello/route.ts  # App Router API route
```

---

## 2. Server vs Client Components

- Default: component เป็น Server Component (render ที่เซิร์ฟเวอร์)
- ถ้าต้องใช้ state, effect, browser API ให้เพิ่ม `'use client'` บน top ของไฟล์
- ข้อดีของ Server Components: smaller bundles, better SEO, can use server-only libs (DB, secret)

```tsx
// Server component (default)
export default async function Home() {
  const data = await fetch('https://api.example.com/products', { cache: 'no-store' });
  const products = await data.json();
  return <ProductList products={products} />;
}

// Client component
'use client';
import { useState } from 'react';
export default function LikeButton() {
  const [liked, setLiked] = useState(false);
  return <button onClick={() => setLiked((s) => !s)}>{liked ? '♥' : '♡'}</button>;
}
```

---

## 3. Data Fetching Patterns

- Server Components สามารถเรียก fetch/DB โดยตรง (recommended)
- fetch options: `cache: 'no-store'` (dynamic), `next: { revalidate: 60 }` (ISR-like)
- `getStaticProps`/`getServerSideProps` เป็นของ Pages Router; ใน App Router ใช้ `generateStaticParams`, `generateMetadata`, และการตั้ง `revalidate`

```ts
// Server component with revalidate
export default async function Page() {
  const res = await fetch('https://api.example.com/posts', {
    next: { revalidate: 60 }, // ISR: revalidate every 60s
  });
  const posts = await res.json();
  return <PostsList posts={posts} />;
}
```

- Edge Caching: ใช้ header `Cache-Control` หรือ Vercel edge
- Stale-while-revalidate: ใช้ `revalidate` + appropriate headers for fast responses

---

## 4. API Routes (app/api)

- App Router ใช้ `route.ts` exports for handlers

```ts
// app/api/hello/route.ts
import { NextResponse } from 'next/server';

export async function GET(req: Request) {
  return NextResponse.json({ message: 'Hello from App Router API' });
}

export async function POST(req: Request) {
  const body = await req.json();
  // validate body with zod
  return NextResponse.json({ ok: true, data: body });
}
```

- API routes run in Node or Edge runtime depending on config (`export const runtime = 'edge'`)
- Use Zod to validate body and return proper status codes

---

## 5. Server Actions

- Server Actions (experimental/stable depending on version) ให้เรียก server-side functions จาก client โดยตรงผ่าน form/button
- ใช้งานเมื่ออยากให้ logic รันที่เซิร์ฟเวอร์ (เช่น เขียน DB) โดยไม่ต้องสร้าง API route แยก

```tsx
// app/actions.ts (server)
export async function addComment(formData: FormData) {
  'use server';
  const content = formData.get('content')?.toString();
  // validate
  await db.comment.create({ data: { content } });
}

// app/post/page.tsx (server component)
import { addComment } from '../actions';

export default function PostPage() {
  return (
    <form action={addComment}>
      <textarea name="content" />
      <button type="submit">Add</button>
    </form>
  );
}
}
```

---

## 6. Middleware

- `middleware.ts` อยู่ที่ root และรันบน Edge runtime
- เหมาะสำหรับ authentication redirects, A/B testing, i18n routing

```ts
// middleware.ts
import { NextResponse } from 'next/server';
import type { NextRequest } from 'next/server';

export function middleware(req: NextRequest) {
  const url = req.nextUrl.clone();
  const token = req.cookies.get('token');
  if (!token && url.pathname.startsWith('/dashboard')) {
    url.pathname = '/login';
    return NextResponse.redirect(url);
  }
  return NextResponse.next();
}
}
```

- Be mindful: middleware size and sync constraints — avoid large libs

---

## 7. Runtime & Edge

- `export const runtime = 'edge'` for edge functions (faster cold starts, limited 1st-party Node APIs)
- Server Components default runtime is Node.js unless configured
- Choose edge for low-latency global responses; choose Node when you need heavier Node APIs (DB drivers)

---

## 8. Authentication Patterns

- Recommended: server-side session + HttpOnly cookies or JWT with refresh flow
- For App Router, do auth check in server components or middleware

```ts
// example server component auth
export default async function DashboardPage() {
  const token = cookies().get('token')?.value;
  const user = await getUserFromToken(token);
  if (!user) redirect('/login');
  return <Dashboard user={user} />;
}
}
```

- Use NextAuth or custom auth. For NextAuth, prefer server actions or API routes for auth flows

---

## 9. Testing Next.js App Router

- Use React Testing Library + Vitest/Jest for components
- For server components, test rendered HTML by mocking fetch/DB
- Integration: Playwright or Cypress for full flows

---

## 10. Performance & SEO

- Use Server Components for SEO-critical pages
- LCP optimization: mark hero image with priority and use next/image
- Use `generateMetadata` to set dynamic metadata per route

```ts
// app/blog/[slug]/page.tsx
export async function generateMetadata({ params }) {
  const post = await getPost(params.slug);
  return { title: post.title, description: post.excerpt };
}
}
```

---

## 11. Deployment Considerations

- Vercel is the reference platform; ensure environment variables set in project
- Choose appropriate Node version and region
- Monitor build cache and image optimization settings

---

## 12. Security Best Practices

- Validate inputs in API routes and server actions with Zod
- Use HttpOnly cookies for session tokens
- Use CSP headers and other security headers in `next.config.js`
- Rate-limit API routes if public

---

## 🏋️ Workshop (15 นาที)

1. สร้าง route `app/products/page.tsx` เป็น Server Component ที่ดึงรายการสินค้าจาก API และแสดงเป็น list (ใช้ `revalidate: 30`)
2. สร้าง `app/api/products/route.ts` ที่ตอบ JSON แบบ paginated และ validate request body ด้วย Zod
3. สร้าง client component `AddProductForm` (ใช้ `use client`) ที่ส่งข้อมูลผ่าน Server Action `addProduct` และทำ optimistic UI update
4. เพิ่ม `middleware.ts` ที่ redirect ผู้ใช้ที่ไม่ได้ล็อกอินออกจาก `/dashboard`

---

## 📌 สรุป

| หัวข้อ | สิ่งที่ควรจำ |
|--------|------------|
| App Router | layout, nested routes, loading/error pages |
| Server vs Client | default server; use 'use client' for hooks/browser API |
| Data Fetching | fetch with `next: { revalidate }` หรือ `cache: 'no-store'` |
| Server Actions | ลด boilerplate ของ API routes เมื่อต้องการ server-side logic |
| Middleware | รันบน Edge, ใช้สำหรับ redirect/auth/i18n |
| Edge vs Node | edge = low-latency, Node = full Node APIs |

---

[← ชั่วโมงที่ 3](./hour-03-zustand.md) | [ชั่วโมงที่ 6 →](./hour-06-tailwind.md) | [← กลับหน้าหลัก](./README.md)