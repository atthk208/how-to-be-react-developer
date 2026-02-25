# ชั่วโมงที่ 8 · Production & Code Quality

> เวลา: 60 นาที | ระดับ: Mid-Level

---

## 🎯 เป้าหมาย

Deploy Next.js app สู่ production อย่างมั่นใจ พร้อม code quality tools, error monitoring, performance optimization และ CI/CD pipeline

---

## 1. Code Quality Toolchain

### 1.1 ESLint Configuration

```bash
npm install -D eslint @typescript-eslint/eslint-plugin @typescript-eslint/parser
npm install -D eslint-plugin-react eslint-plugin-react-hooks
npm install -D eslint-plugin-import eslint-config-next
```

```js
// .eslintrc.js
module.exports = {
  extends: [
    "next/core-web-vitals",
    "plugin:@typescript-eslint/recommended",
    "plugin:react-hooks/recommended",
  ],
  rules: {
    "@typescript-eslint/no-explicit-any": "error",
    "@typescript-eslint/no-unused-vars": ["error", { argsIgnorePattern: "^_" }],
    "@typescript-eslint/consistent-type-imports": "warn",
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn",
    "import/order": [
      "warn",
      {
        groups: ["builtin", "external", "internal", "parent", "sibling"],
        "newlines-between": "always",
      },
    ],
    "no-console": ["warn", { allow: ["warn", "error"] }],
  },
};
```

### 1.2 Prettier Configuration

```json
// .prettierrc
{
  "semi": true,
  "singleQuote": false,
  "tabWidth": 2,
  "trailingComma": "es5",
  "printWidth": 100,
  "arrowParens": "always",
  "endOfLine": "lf"
}
```

### 1.3 Husky + lint-staged (Pre-commit hooks)

```bash
npm install -D husky lint-staged
npx husky init
```

```json
// package.json
{
  "lint-staged": {
    "*.{ts,tsx}": [
      "eslint --fix",
      "prettier --write",
      "jest --findRelatedTests --passWithNoTests"
    ],
    "*.{json,md,css}": "prettier --write"
  }
}
```

### 1.4 Conventional Commits

```bash
npm install -D @commitlint/cli @commitlint/config-conventional
```

```
# Format: <type>(<scope>): <subject>
feat(auth): add Google OAuth login
fix(cart): correct total calculation when removing items
docs(readme): update setup instructions
test(button): add accessibility tests
refactor(api): extract error handling to middleware
chore(deps): update dependencies
```

---

## 2. Environment Variables

```bash
# .env.local (ไม่ commit ลง git!)
DATABASE_URL="postgresql://user:pass@localhost:5432/mydb"
NEXTAUTH_SECRET="your-secret-key-min-32-chars"
NEXTAUTH_URL="http://localhost:3000"

# .env.example (commit เพื่อให้คนอื่นรู้ว่าต้องมี)
DATABASE_URL=""
NEXTAUTH_SECRET=""
NEXTAUTH_URL=""
NEXT_PUBLIC_API_URL=""
```

```ts
// lib/env.ts - Validate env vars at startup
import { z } from "zod";

const envSchema = z.object({
  DATABASE_URL: z.string().min(1),
  NEXTAUTH_SECRET: z.string().min(32),
  NEXTAUTH_URL: z.string().url(),
  NEXT_PUBLIC_API_URL: z.string().url(),
});

function validateEnv() {
  const result = envSchema.safeParse(process.env);
  if (!result.success) {
    console.error("❌ Invalid environment variables:");
    console.error(result.error.flatten().fieldErrors);
    throw new Error("Invalid environment variables");
  }
  return result.data;
}

export const env = validateEnv();
```

---

## 3. Error Monitoring (Sentry)

```bash
npm install @sentry/nextjs
npx @sentry/wizard@latest -i nextjs
```

```ts
// sentry.client.config.ts
import * as Sentry from "@sentry/nextjs";

Sentry.init({
  dsn: process.env.NEXT_PUBLIC_SENTRY_DSN,
  environment: process.env.NODE_ENV,
  tracesSampleRate: process.env.NODE_ENV === "production" ? 0.1 : 1.0,
  replaysOnErrorSampleRate: 1.0,
  replaysSessionSampleRate: 0.1,
  integrations: [Sentry.replayIntegration()],
});

// Manual error capture
try {
  await riskyOperation();
} catch (error) {
  Sentry.captureException(error, {
    tags: { section: "checkout" },
    extra: { userId, cartId },
  });
  throw error;
}
```

---

## 4. Performance Optimization

### 4.1 Image Optimization

```tsx
import Image from "next/image";

<Image
  src="/hero.jpg"
  alt="Hero banner"
  width={1200}
  height={600}
  priority                    // LCP image - load ก่อน
  quality={85}
  sizes="(max-width: 768px) 100vw, (max-width: 1200px) 50vw, 33vw"
  placeholder="blur"
  blurDataURL="data:image/jpeg;base64,..."
/>
```

### 4.2 Bundle Analysis

```bash
npm install -D @next/bundle-analyzer

# next.config.ts
const withBundleAnalyzer = require("@next/bundle-analyzer")({
  enabled: process.env.ANALYZE === "true",
});

# Run analysis
ANALYZE=true npm run build
```

### 4.3 Lazy Loading

```tsx
import dynamic from "next/dynamic";

const RichTextEditor = dynamic(() => import("./RichTextEditor"), {
  loading: () => <Skeleton />,
  ssr: false,               // client-only library
});
```

---

## 5. CI/CD Pipeline (GitHub Actions)

```yaml
# .github/workflows/ci.yml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  quality:
    name: Code Quality
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"

      - run: npm ci

      - name: TypeScript check
        run: npx tsc --noEmit

      - name: ESLint
        run: npm run lint

      - name: Run tests
        run: npm test -- --coverage --ci

      - name: Upload coverage
        uses: codecov/codecov-action@v3

  build:
    name: Build
    needs: quality
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"
      - run: npm ci
      - name: Build
        run: npm run build
        env:
          DATABASE_URL: ${{ secrets.DATABASE_URL }}
          NEXTAUTH_SECRET: ${{ secrets.NEXTAUTH_SECRET }}

  deploy:
    name: Deploy to Vercel
    needs: build
    runs-on: ubuntu-latest
    if: github.ref == 'refs/heads/main'
    steps:
      - uses: actions/checkout@v4
      - name: Deploy
        uses: amondnet/vercel-action@v25
        with:
          vercel-token: ${{ secrets.VERCEL_TOKEN }}
          vercel-org-id: ${{ secrets.ORG_ID }}
          vercel-project-id: ${{ secrets.PROJECT_ID }}
          vercel-args: "--prod"
```

---

## 6. Security Headers

```ts
// next.config.ts
const securityHeaders = [
  { key: "X-DNS-Prefetch-Control", value: "on" },
  { key: "X-Frame-Options", value: "SAMEORIGIN" },
  { key: "X-Content-Type-Options", value: "nosniff" },
  { key: "Referrer-Policy", value: "strict-origin-when-cross-origin" },
  {
    key: "Permissions-Policy",
    value: "camera=(), microphone=(), geolocation=()",
  },
];

const config: NextConfig = {
  async headers() {
    return [{ source: "/(.*)", headers: securityHeaders }];
  },
};
```

---

## 7. Production Checklist

```markdown
## Pre-Launch Checklist

### Code Quality
- [ ] TypeScript strict mode enabled (no `any`)
- [ ] ESLint passing with 0 errors
- [ ] Test coverage > 80%
- [ ] No console.log in production code

### Performance
- [ ] Lighthouse score > 90
- [ ] Images optimized with next/image
- [ ] Fonts loaded with next/font
- [ ] Bundle size analyzed and acceptable

### Security
- [ ] Security headers configured
- [ ] Secrets in environment variables only
- [ ] Input validation with Zod on all endpoints
- [ ] Rate limiting on API routes

### Monitoring
- [ ] Sentry configured and tested
- [ ] Error alerting set up
- [ ] Uptime monitoring (UptimeRobot / Checkly)

### SEO
- [ ] Metadata on all pages
- [ ] Open Graph tags
- [ ] sitemap.xml generated
- [ ] robots.txt configured

### Deployment
- [ ] .env.example updated
- [ ] Database migrations run
- [ ] CI/CD pipeline passing green
- [ ] Rollback plan documented
```

---

## 🏋️ Workshop (15 นาที)

1. Setup ESLint + Prettier + Husky ในโปรเจกต์
2. สร้าง GitHub Actions workflow ที่ run lint + test + build
3. สร้าง `lib/env.ts` ที่ validate environment variables ด้วย Zod

---

## 📌 สรุป

| หัวข้อ | สิ่งที่ควรจำ |
|--------|------------|
| ESLint + Prettier | ทำให้ codebase consistent |
| Husky + lint-staged | ป้องกัน bad code เข้า git |
| Conventional Commits | commit history ที่อ่านได้ |
| Sentry | Error monitoring ใน production |
| GitHub Actions | Automate test + deploy |
| Security Headers | ป้องกัน common attacks |
| Production Checklist | ลด risk ก่อน launch |

---

## 🎉 จบหลักสูตร!

คุณได้เรียนรู้:

| ชั่วโมง | สิ่งที่ได้ |
|--------|----------|
| 1 | TypeScript ที่ถูกต้องและปลอดภัย |
| 2 | React Architecture ที่ scale ได้ |
| 3 | Zustand สำหรับ State Management |
| 4–5 | Next.js App Router แบบ mid-level |
| 6 | Tailwind Design System |
| 7 | Unit Testing ด้วย Jest |
| 8 | Production-ready Code Quality |

> **ขั้นต่อไป:** ลงมือทำ side project ที่ใช้ทุก skill ที่เรียนมา 🚀

---

[← ชั่วโมงที่ 7](./hour-07-jest.md) | [← กลับหน้าหลัก](./README.md)