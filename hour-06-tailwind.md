# ชั่วโมงที่ 6 · Tailwind (Component System Level)

> เวลา: 60 นาที | ระดับ: Component System

---

## 🎯 เป้าหมาย

เรียนรู้การออกแบบระบบคอมโพเนนต์ด้วย Tailwind: utility-first mindset, การสร้าง component primitives, design tokens integration, theming, responsive patterns, และการเขียน CSS ที่ maintainable สำหรับทีมงาน

---

## 1. แนวคิด Utility-first และ Atomic CSS

- Tailwind เป็น utility-first: เขียน class เล็ก ๆ เพื่อประกอบเป็น UI
- ข้อดี: ความสอดคล้อง, ลด CSS ที่ไม่ถูกใช้, ความเร็วในการพัฒนา
- ข้อควรระวัง: ชื่อคลาสยาวๆ ใน JSX — จัดเป็น component หรือใช้ helper เพื่อปรับปรุง readability

```tsx
// ตัวอย่าง Button component แบบ simple
import React from 'react';

type Props = React.ButtonHTMLAttributes<HTMLButtonElement> & { variant?: 'primary' | 'ghost' };

export function Button({ variant = 'primary', className = '', ...rest }: Props) {
  const base = 'inline-flex items-center gap-2 px-4 py-2 rounded-md text-sm font-medium';
  const variants: Record<string, string> = {
    primary: 'bg-indigo-600 text-white hover:bg-indigo-500',
    ghost: 'bg-transparent text-gray-700 hover:bg-gray-100',
  };
  return <button className={`${base} ${variants[variant]} ${className}`} {...rest} />;
}
```

---

## 2. Design Tokens & Tailwind Config

- เก็บสี, ขนาดตัวอักษร, spacing เป็น design tokens ใน `tailwind.config.js` เพื่อให้เป็น single source of truth
- ตั้งค่า custom colors, spacing scale, และ fonts ใน theme.extend
- ใช้ `@apply` ในไฟล์ CSS สำหรับกลุ่ม utilities ที่ใช้บ่อย (component primitives)

```js
// tailwind.config.js
module.exports = {
  content: ['./src/**/*.{ts,tsx,js,jsx}', './app/**/*.{ts,tsx}'],
  theme: {
    extend: {
      colors: {
        brand: {
          DEFAULT: '#4f46e5',
          50: '#eef2ff',
          700: '#4338ca',
        },
      },
      spacing: { 72: '18rem', 84: '21rem' },
    },
  },
  plugins: [],
};
```

---

## 3. Component Primitives & Reusability

- สร้าง primitive components ที่มี API ชัดเจน (Button, Card, Container, Stack, IconButton)
- หลีกเลี่ยงการกระจาย class ซ้ำ ปรับมาเป็น component หรือใช้ `cn()` helper (เช่น class-variance-authority หรือ clsx)

```tsx
// utils/cn.ts
export function cn(...classes: Array<string | false | null | undefined>) {
  return classes.filter(Boolean).join(' ');
}
// components/Stack.tsx
export const Stack = ({ children, space = '4', vertical = true }) => (
  <div className={vertical ? `flex flex-col gap-${space}` : `flex flex-row gap-${space}`}>{children}</div>
);
```

---

## 4. Variants & class-variance-authority (CVA)

- ใช้ CVA เพื่อจัดการ variants, sizes, states ใน component แบบ typed (สำหรับ TypeScript)
- ตัวอย่างการใช้ `class-variance-authority` เ���ื่อสร้าง Button API ที่ปลอดภัยต่อการพิมพ์

```ts
import { cva, type VariantProps } from 'class-variance-authority';

const button = cva('inline-flex items-center rounded-md', {
  variants: {
    intent: {
      primary: 'bg-brand text-white hover:bg-brand-700',
      ghost: 'bg-transparent text-gray-700',
    },
    size: {
      sm: 'px-2 py-1 text-sm',
      md: 'px-4 py-2 text-base',
    },
  },
  defaultVariants: { intent: 'primary', size: 'md' },
});

type ButtonProps = React.ButtonHTMLAttributes<HTMLButtonElement> & VariantProps<typeof button>;

export function Button({ intent, size, className, ...props }: ButtonProps) {
  return <button className={`${button({ intent, size })} ${className ?? ''}`} {...props} />;
}
```

---

## 5. Responsive & Mobile-first Patterns

- ใช้ breakpoints ใน `tailwind.config.js` หรือ default (`sm`, `md`, `lg`, `xl`)
- Prefer mobile-first: write base styles for mobile and add `md:` for larger screens

```tsx
<div className="p-4 grid grid-cols-1 md:grid-cols-3 gap-4">...</div>
```

---

## 6. JIT Mode & Purging (Tree-shaking)

- Tailwind v3 มี JIT by default — สร้าง utilities ตาม class ที่ใช้จริง
- ตรวจสอบ `content` path ให้ครอบคลุมไฟล์ทั้งหมด (รวม .mdx, .tsx ในโฟลเดอร์ app/ หรือ src/)
- ถ้าสร้าง class แบบ dynamic ให้ระวัง purge: ใช้วิธีที่ปลอดภัยเช่น mapping หรือ safelist
---

## 7. Theming & Dark Mode

- Tailwind รองรับ dark mode โดยตั้งค่า `darkMode: 'class'` (หรือ media)
- สร้าง context/provider สำหรับ theme switching และเพิ่ม class `dark` บน <html> หรือ root container

```tsx
// theme-provider.tsx
'use client';
import { useEffect, useState } from 'react';
export function ThemeProvider({ children }: { children: React.ReactNode }) {
  const [theme, setTheme] = useState<'light' | 'dark'>('light');
  useEffect(() => {
    document.documentElement.classList.toggle('dark', theme === 'dark');
  }, [theme]);
  return (
    <div>
      <button onClick={() => setTheme((t) => (t === 'light' ? 'dark' : 'light'))}>Toggle</button>
      {children}
    </div>
  );
}
```
---

## 8. Accessibility (A11y) with Tailwind

- Tailwind ไม่ได้แทนที่ A11y — ต้องใส่ role, aria-* และ semantic HTML
- ตัวอย่าง: focus styles, skip links และ color contrast

```tsx
<button className="focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-brand">Save</button>
```
---

## 9. Integration with Component Libraries & Headless UI

- ใช้ Headless UI / Radix UI สำหรับ accessible primitives (Dialog, Popover, Tabs) แล้ว style ด้วย Tailwind
- ตัวอย่าง: Combobox/Popover ของ Headless UI จะให้ accessible behavior และเราเพิ่ม class ของ Tailwind เพื่อ custom look
---

## 10. Performance Best Practices

- ตรวจสอบขนาด bundle: ใช้ `@tailwindcss/typography` หรือ `forms` ตามต้องการ อย่าเพิ่ม plugin เกินจำเป็น
- ใช้ CDN หรือ prebuilt CSS สำหรับ rapid prototyping แต่ใน production ควร build ด้วย PostCSS และ Purge content paths ถูกต้อง
---

## 11. Testing Components with Tailwind

- ใน unit tests ให้ assert ผ่าน role/text/aria มากกว่าตรวจสอบ class names
- E2E tests (Playwright) สามารถตรวจสอบ visual regression โดยใช้ Percy หรือ Chromatic
---

## Workshop (20 นาที)

1. สร้าง component `Card` ที่รับ props: title, description, actions และรองรับ theme `elevated` / `flat`
2. สร้าง `Stack` และ `Grid` primitive ที่ใช้ utility classes และใช้งานร่วมกับ `Card`
3. เพิ่ม variants ให้ `Button` ด้วย CVA แล้วใช้งานใน `Card` actions
4. เขียนตัวอย่าง responsive layout ที่มี 1 คอลัมน์บน mobile และ 3 คอลัมน์บน md+

---

## 🏁 สรุป

- Tailwind ช่วยเราสร้างระบบคอมโพเนนต์ที่ maintainable ถ้าจัดการ design tokens และ primitives ให้ดี
- ใช้ CVA / cn helper เพื่อจัดการ variants และ maintain types ใน TypeScript
- รวม Headless UI/Radix สำหรับ accessible primitives และทดสอบผ่าน role/text แทนการพึ่งพา class names
---

[← ชั่วโมงที่ 4–5](./hour-04-05-nextjs.md) | [ชั่วโมงที่ 7 →](./hour-07-jest.md) | [← กลับหน้าหลัก](./README.md)