---
title: Coding Standards
type: standard
owner: Development Leadership
last_reviewed: 2026-07-10
status: current
tags:
  - development
  - standards
  - code-quality
---

# Coding Standards

> Davis Elen's standards for writing, organizing, and shipping code. These standards apply to all development work: Next.js, Node.js, WordPress, and supporting tools. They're designed to be practical and enforceable, balancing quality with shipping speed.

## Philosophy

Our coding standards are built around these principles:

- **Consistent enough to collaborate.** Teams should be able to understand each other's code without extensive onboarding.
- **Practical over perfect.** We ship real work for real clients. Standards should make shipping faster, not slower.
- **A11y and performance matter.** Accessibility and performance aren't nice-to-haves—they're part of delivering professional work.
- **Security by default.** Especially for WordPress/PHP, we follow security best practices without debate.
- **Testing is an investment.** We're building a testing culture, starting with high-impact areas.

---

## JavaScript/TypeScript Standards

### General Principles

- Use **TypeScript** for all new projects (Next.js, Node.js). It catches errors and makes code more maintainable.
- Use **modern JavaScript** (ES2020+). Transpilation is handled by build tools.
- Prefer **functional components** and hooks in React/Next.js. Class components are legacy.
- Use **const** by default, **let** when you need reassignment, never **var**.

### File & Folder Structure (Next.js App Router)

Structure projects using a **feature-based approach**:

```
src/
├── app/
│   ├── (auth)/                 # Route group for auth pages
│   │   ├── login/page.tsx
│   │   ├── signup/page.tsx
│   │   └── layout.tsx
│   ├── (marketing)/            # Route group for public pages
│   │   ├── page.tsx            # Homepage
│   │   ├── about/page.tsx
│   │   └── layout.tsx
│   ├── dashboard/              # Protected routes
│   │   ├── page.tsx
│   │   ├── settings/page.tsx
│   │   └── layout.tsx
│   ├── api/                    # API routes
│   │   ├── auth/
│   │   ├── users/
│   │   └── posts/
│   └── layout.tsx              # Root layout
├── components/
│   ├── common/                 # Reusable across features
│   │   ├── Header.tsx
│   │   ├── Footer.tsx
│   │   └── Navigation.tsx
│   ├── auth/                   # Feature-specific components
│   │   ├── LoginForm.tsx
│   │   └── SignupForm.tsx
│   └── dashboard/
│       ├── Card.tsx
│       └── Chart.tsx
├── lib/
│   ├── api.ts                  # API client functions
│   ├── auth.ts                 # Auth utilities
│   ├── db.ts                   # Database client
│   └── utils.ts                # General utilities
├── hooks/
│   ├── useAuth.ts
│   ├── useForm.ts
│   └── useFetch.ts
├── types/
│   ├── index.ts                # Shared types
│   ├── auth.ts
│   └── api.ts
├── styles/
│   └── globals.css
└── env.ts                      # Environment variable validation (see below)
```

**Key principles:**
- Group by feature, not by type (don't separate all components into one folder)
- Keep related code together (component + hooks + types)
- Use `index.ts` only when exporting multiple items from a folder
- `lib/` is for utilities, helpers, and reusable logic
- `types/` contains all TypeScript type definitions

### Naming Conventions

- **Files:** camelCase for utilities (`apiClient.ts`), PascalCase for components (`UserCard.tsx`)
- **React components:** PascalCase (`MyComponent.tsx`)
- **Hooks:** camelCase, prefixed with `use` (`useAuth.ts`)
- **Types/Interfaces:** PascalCase (`User`, `ApiResponse`)
- **Constants:** SCREAMING_SNAKE_CASE (`MAX_FILE_SIZE`, `API_TIMEOUT`)
- **Functions:** camelCase (`fetchUser`, `formatDate`)
- **CSS classes:** kebab-case (if using CSS classes or Tailwind)

### Code Style & Formatting

**Use Prettier for formatting:**

```json
// .prettierrc.json
{
  "semi": true,
  "singleQuote": true,
  "trailingComma": "es5",
  "printWidth": 100,
  "useTabs": true
}
```

Run `prettier --write .` before committing, or integrate with your editor.

**Tabs, not spaces.** All indentation uses tabs for consistency.

**Use ESLint for linting:**

Base config for Next.js projects:

```javascript
// .eslintrc.json
{
  "extends": ["next/core-web-vitals"],
  "rules": {
    "react/react-in-jsx-scope": "off",
    "react-hooks/rules-of-hooks": "error",
    "react-hooks/exhaustive-deps": "warn",
    "@next/next/no-html-link-for-pages": "warn"
  }
}
```

This is **moderate** linting—catches issues without being overly strict.

### Imports

- Group imports: external libraries first, then internal modules, then types
- Use absolute imports (configure in `tsconfig.json` or `next.config.js`)
- Order: `import type` statements last

```typescript
// ✅ Good
import React, { useState } from 'react';
import Link from 'next/link';
import { useRouter } from 'next/navigation';

import { Button } from '@/components/common/Button';
import { useAuth } from '@/hooks/useAuth';
import { formatDate } from '@/lib/utils';

import type { User } from '@/types';
```

```typescript
// ❌ Bad
import type { User } from '@/types';
import { formatDate } from '@/lib/utils';
import { useAuth } from '@/hooks/useAuth';
import { Button } from '@/components/common/Button';
import Link from 'next/link';
```

### Null/Undefined Handling

Use optional chaining and nullish coalescing:

```typescript
// ✅ Good
const email = user?.email ?? 'no-email@example.com';
const name = user?.profile?.name || 'Unknown';

// ❌ Avoid
const email = user && user.email ? user.email : 'no-email@example.com';
```

### Error Handling

Handle errors in obvious places. You don't need try/catch everywhere, but use it when appropriate:

```typescript
// ✅ Good: Catch API errors
async function fetchUser(id: string) {
  try {
    const response = await fetch(`/api/users/${id}`);
    if (!response.ok) throw new Error('Failed to fetch user');
    return await response.json();
  } catch (error) {
    console.error('Error fetching user:', error);
    throw error; // Let caller handle
  }
}

// ✅ Good: Error boundary for React
export function ErrorBoundary({ error }: { error: Error }) {
  return <div>Something went wrong: {error.message}</div>;
}

// ❌ Avoid: Try/catch everywhere for normal flow
try {
  const name = user.profile.name;
} catch (e) {
  // Normal flow, not an error
}
```

### Async/Await

- Prefer async/await over `.then()` chains
- Use `await` for readability, but don't await unnecessarily

```typescript
// ✅ Good
async function processUsers(userIds: string[]) {
  const users = await Promise.all(userIds.map(id => fetchUser(id)));
  return users;
}

// ❌ Less clear
userIds.map(id => fetchUser(id))
  .then(results => results)
```

---

## React / Next.js Standards

### Components

**Functional components only.** No class components.

```typescript
// ✅ Good
export function UserCard({ user }: { user: User }) {
  const [isLoading, setIsLoading] = useState(false);
  
  return (
    <div className="user-card">
      <h3>{user.name}</h3>
      <p>{user.email}</p>
    </div>
  );
}

// ❌ Avoid (legacy)
export class UserCard extends React.Component {
  // ...
}
```

**Keep components small.** If a component is >200 lines, break it into smaller pieces.

**Props should be typed:**

```typescript
// ✅ Good
interface UserCardProps {
  user: User;
  onDelete?: (id: string) => void;
  isLoading?: boolean;
}

export function UserCard({ user, onDelete, isLoading = false }: UserCardProps) {
  // ...
}

// ❌ Avoid
export function UserCard(props: any) {
  // ...
}
```

### Hooks

**Use built-in hooks:** `useState`, `useEffect`, `useContext`, `useCallback`, `useMemo`

**Custom hooks should start with `use`:**

```typescript
// ✅ Good
export function useAuth() {
  const [user, setUser] = useState<User | null>(null);
  
  useEffect(() => {
    // fetch user
  }, []);
  
  return { user };
}

// Use it
const { user } = useAuth();
```

**Keep effects simple.** If an effect does too much, break it into multiple effects.

```typescript
// ✅ Better: Two focused effects
useEffect(() => {
  fetchUser(); // Effect 1: Fetch on mount
}, []);

useEffect(() => {
  updateUserProfile(user); // Effect 2: Update when user changes
}, [user]);

// ❌ Avoid: One large effect
useEffect(() => {
  fetchUser();
  updateUserProfile(user);
  // ... more logic
}, []);
```

### Rendering

**Use conditional rendering properly:**

```typescript
// ✅ Good
{isLoading && <Spinner />}
{!isLoading && <Content />}

{user && <UserProfile user={user} />}
{!user && <LoginPrompt />}

// ❌ Avoid
{isLoading ? <Spinner /> : !user ? <LoginPrompt /> : <Content />}  // Too nested
```

**Don't inline complex logic in JSX:**

```typescript
// ✅ Good: Logic in a function
function renderContent() {
  if (isLoading) return <Spinner />;
  if (error) return <ErrorMessage error={error} />;
  return <Data data={data} />;
}

return <div>{renderContent()}</div>;

// ❌ Avoid: Complex logic in JSX
return (
  <div>
    {isLoading ? (
      <Spinner />
    ) : error ? (
      <ErrorMessage error={error} />
    ) : (
      <Data data={data} />
    )}
  </div>
);
```

### Performance

**Use `useCallback` and `useMemo` intentionally**, not everywhere:

```typescript
// ✅ Good: Memoize callback passed to child component
const handleClick = useCallback(() => {
  updateUser(userId);
}, [userId]);

return <Button onClick={handleClick} />;

// ✅ Good: Memoize expensive computation
const expensiveResult = useMemo(() => {
  return computeExpensiveData(items);
}, [items]);

// ❌ Don't memoize everything—it adds overhead
const memoizedString = useMemo(() => 'hello', []);
```

**Lazy load heavy components:**

```typescript
import dynamic from 'next/dynamic';

const HeavyChart = dynamic(() => import('@/components/Chart'), {
  loading: () => <div>Loading...</div>,
});

export function Dashboard() {
  return <HeavyChart data={data} />;
}
```

---

## Next.js Specific

### Server vs Client Components

**Default to Server Components.** Use Client Components only when you need interactivity, hooks, or browser APIs.

```typescript
// ✅ Good: Server component (default)
export async function UserList() {
  const users = await db.user.findMany();
  return (
    <ul>
      {users.map(user => <li key={user.id}>{user.name}</li>)}
    </ul>
  );
}

// ✅ Good: Client component (only when needed)
'use client';

export function UserFilter() {
  const [filter, setFilter] = useState('');
  return <input onChange={e => setFilter(e.target.value)} />;
}
```

### API Routes

Keep API routes simple and focused:

```typescript
// ✅ Good: api/users/[id].ts
import { NextRequest, NextResponse } from 'next/server';

export async function GET(
  request: NextRequest,
  { params }: { params: { id: string } }
) {
  try {
    const user = await db.user.findUnique({
      where: { id: params.id },
    });
    
    if (!user) {
      return NextResponse.json({ error: 'Not found' }, { status: 404 });
    }
    
    return NextResponse.json(user);
  } catch (error) {
    return NextResponse.json(
      { error: 'Internal server error' },
      { status: 500 }
    );
  }
}
```

### Metadata & SEO

Set metadata on pages for SEO:

```typescript
import type { Metadata } from 'next';

export const metadata: Metadata = {
  title: 'Users',
  description: 'View all users',
  openGraph: {
    title: 'Users',
    description: 'View all users',
    type: 'website',
  },
};

export default function UsersPage() {
  return <UserList />;
}
```

---

## Accessibility (a11y)

**Accessibility is non-negotiable.** All projects should meet WCAG 2.1 AA standards.

### Semantic HTML

```typescript
// ✅ Good: Semantic HTML
<nav>
  <ul>
    <li><a href="/about">About</a></li>
    <li><a href="/contact">Contact</a></li>
  </ul>
</nav>

// ❌ Avoid: div soup
<div className="nav">
  <div className="nav-list">
    <div className="nav-item"><span>About</span></div>
  </div>
</div>
```

### ARIA Labels

When semantic HTML isn't enough, use ARIA:

```typescript
// ✅ Good
<button aria-label="Close dialog" onClick={closeDialog}>
  ×
</button>

<div role="alert" aria-live="polite">
  Error message here
</div>

// Focus management
const dialogRef = useRef<HTMLDivElement>(null);
useEffect(() => {
  dialogRef.current?.focus();
}, []);
```

### Form Accessibility

```typescript
// ✅ Good
<label htmlFor="email">Email</label>
<input
  id="email"
  type="email"
  aria-describedby="email-help"
  required
/>
<p id="email-help">We'll never share your email</p>

// ❌ Avoid
<input placeholder="Email" />
```

### Color & Contrast

- Never rely on color alone to convey meaning
- Maintain 4.5:1 contrast ratio for text
- Test with tools like WAVE or axe DevTools

---

## Performance

### Image Optimization

Use Next.js `Image` component:

```typescript
import Image from 'next/image';

export function UserAvatar({ user }: { user: User }) {
  return (
    <Image
      src={user.avatar}
      alt={user.name}
      width={40}
      height={40}
      className="rounded-full"
    />
  );
}
```

### Font Optimization

Use `next/font`:

```typescript
import { Inter } from 'next/font/google';

const inter = Inter({ subsets: ['latin'] });

export default function RootLayout({ children }: { children: React.ReactNode }) {
  return (
    <html className={inter.className}>
      <body>{children}</body>
    </html>
  );
}
```

### Bundle Size

- Lazy load heavy dependencies
- Check bundle size with `npm run build` and review `.next/static`
- Avoid importing entire libraries when you only need one function

```typescript
// ❌ Avoid: Imports entire lodash
import _ from 'lodash';
const sorted = _.sortBy(items, 'name');

// ✅ Good: Import only what you need
import sortBy from 'lodash/sortBy';
const sorted = sortBy(items, 'name');

// ✅ Better: Use native JavaScript
const sorted = [...items].sort((a, b) => a.name.localeCompare(b.name));
```

---

## Node.js/Backend Standards

### Environment Variables

Store all configuration in environment variables. Use a validation library like `zod`:

```typescript
// lib/env.ts
import { z } from 'zod';

const envSchema = z.object({
  DATABASE_URL: z.string().url(),
  API_SECRET: z.string().min(32),
  NODE_ENV: z.enum(['development', 'production', 'test']),
  PORT: z.coerce.number().default(3000),
});

export const env = envSchema.parse(process.env);

// Usage
import { env } from '@/lib/env';
const dbUrl = env.DATABASE_URL;
```

### Database

- Use an ORM (Prisma, TypeORM) or query builder
- Never write raw SQL with user input (SQL injection risk)
- Version migrations and commit them to version control

### API Design

- Use REST conventions or GraphQL
- Return consistent response formats
- Use appropriate HTTP status codes
- Validate all input

```typescript
// ✅ Good: Consistent API response
{
  success: true,
  data: { user: { id: '123', name: 'John' } },
  error: null
}

{
  success: false,
  data: null,
  error: { code: 'USER_NOT_FOUND', message: 'User not found' }
}
```

---

## WordPress/PHP Standards

### Theme Development

- Use a modern framework (Roots/Sage, Underscores, or custom) if building custom
- Keep PHP out of templates; move logic to functions or classes
- Use child themes when extending existing themes (don't modify the parent)

### Security (WordPress)

**This is non-negotiable:**

- **Escape output:** `esc_html()`, `wp_kses_post()`, `esc_attr()`
- **Sanitize input:** `sanitize_text_field()`, `sanitize_email()`, `wp_verify_nonce()`
- **Use nonces:** For any form submission
- **Capabilities:** Check user capabilities with `current_user_can()`

```php
// ✅ Good: Secure form handling
if ( ! wp_verify_nonce( $_POST['_wpnonce'], 'update_user' ) ) {
    wp_die( 'Security check failed' );
}

if ( ! current_user_can( 'edit_users' ) ) {
    wp_die( 'Insufficient permissions' );
}

$email = sanitize_email( $_POST['email'] );
echo esc_html( $user->name );
```

### Code Organization

```
wp-content/themes/theme-name/
├── functions.php          # Main hooks and setup
├── style.css              # Theme header
├── inc/
│   ├── cpt.php            # Custom post types
│   ├── taxonomies.php     # Custom taxonomies
│   ├── enqueue.php        # Scripts and styles
│   └── settings.php       # Theme settings
├── template-parts/
│   ├── header.php
│   ├── footer.php
│   └── content/
│       ├── content.php
│       └── content-single.php
├── templates/
│   ├── home.php
│   └── single.php
├── assets/
│   ├── css/
│   ├── js/
│   └── images/
└── languages/             # Translation files
```

### WordPress Hooks

Prefer actions/filters over direct function calls:

```php
// ✅ Good: Use hooks
add_action( 'wp_footer', 'my_custom_footer' );

function my_custom_footer() {
    echo 'Custom footer content';
}

// ✅ Good: Add custom hooks for extensibility
do_action( 'my_theme_before_footer' );
$filtered_content = apply_filters( 'my_theme_footer_text', 'Default text' );
echo wp_kses_post( $filtered_content );
```

### jQuery (Use Sparingly)

Modern WordPress should prefer vanilla JavaScript or Alpine.js. If using jQuery:

```php
// ✅ Good: Enqueue jQuery properly
wp_enqueue_script( 'my-script', get_template_directory_uri() . '/assets/js/my-script.js', array( 'jquery' ), filemtime( get_template_directory() . '/assets/js/my-script.js' ), true );

// ✅ Inline script with localization
wp_localize_script( 'my-script', 'myObj', array(
    'ajaxUrl' => admin_url( 'admin-ajax.php' ),
    'nonce'   => wp_create_nonce( 'my_nonce' ),
) );
```

---

## Testing

We're building a testing culture. Start with high-impact areas.

### Unit Tests (Jest for JavaScript)

Test utilities and business logic:

```typescript
// lib/utils.test.ts
import { formatDate } from './utils';

describe('formatDate', () => {
  it('formats a date correctly', () => {
    const date = new Date('2024-01-15');
    expect(formatDate(date)).toBe('Jan 15, 2024');
  });
  
  it('handles null gracefully', () => {
    expect(formatDate(null)).toBe('');
  });
});
```

### Integration Tests (Prisma + Jest)

Test database interactions:

```typescript
// lib/db.test.ts
import { db } from '@/lib/db';

describe('User database', () => {
  it('creates a user', async () => {
    const user = await db.user.create({
      data: { name: 'John', email: 'john@example.com' },
    });
    expect(user.id).toBeDefined();
  });
});
```

### E2E Tests (Playwright)

Test critical user flows:

```typescript
// tests/login.spec.ts
import { test, expect } from '@playwright/test';

test('user can login', async ({ page }) => {
  await page.goto('http://localhost:3000/login');
  await page.fill('input[name="email"]', 'user@example.com');
  await page.fill('input[name="password"]', 'password123');
  await page.click('button[type="submit"]');
  await expect(page).toHaveURL('/dashboard');
});
```

**Start small:** Pick one critical path (authentication, checkout, etc.) and write tests for it. Expand from there.

---

## Git & Version Control

### Commit Messages

Write clear commit messages:

```
✅ Good:
Add user authentication
- Implement login/signup flow
- Add JWT token handling
- Add session management

❌ Bad:
fix stuff
update
wip
```

### Branches

- `main` — Production-ready code
- `develop` — Integration branch
- `feature/user-auth` — Feature branches
- `bugfix/login-error` — Bug fixes
- `chore/update-deps` — Chores

### Pull Requests

Keep PRs focused and reviewable:
- One feature per PR
- Smaller PRs are reviewed faster
- Include context in the description
- Link to related issues

---

## Documentation

We don't require extensive code documentation, but do document:

- **Complex logic:** If it's not obvious why something works that way, explain it
- **APIs:** Document function signatures and parameters
- **Configuration:** How to set up environment, run the project

Example:

```typescript
/**
 * Formats a date into a human-readable string
 * @param date - The date to format
 * @param locale - The locale for formatting (default: 'en-US')
 * @returns Formatted date string, e.g., 'Jan 15, 2024'
 */
export function formatDate(date: Date | null, locale = 'en-US'): string {
  if (!date) return '';
  return new Intl.DateTimeFormat(locale, {
    year: 'numeric',
    month: 'short',
    day: 'numeric',
  }).format(date);
}
```

---

## Code Review Checklist

When reviewing code, check for:

- ✅ Code follows these standards
- ✅ Types are correct (TypeScript)
- ✅ No console.logs left in (except logging libraries)
- ✅ Accessibility is considered
- ✅ Performance is reasonable
- ✅ Error handling is in place
- ✅ Security (especially for WordPress/PHP)
- ✅ Tests pass (if applicable)

---

## Common Mistakes to Avoid

| Mistake | Fix |
|---------|-----|
| Hardcoding secrets in code | Use environment variables |
| No TypeScript types | Type everything |
| Huge components (>200 lines) | Break into smaller pieces |
| Not handling async errors | Add try/catch or .catch() |
| Using `any` type | Define proper types |
| Inline styles | Use CSS or Tailwind |
| Props drilling too deep | Use Context or state management |
| No accessibility labels | Add alt text, aria-labels, etc. |
| Ignoring performance | Use Lighthouse, check bundle size |
| No git commits (one giant commit) | Commit logical units |

---

## Setup Checklist for New Projects

- ✅ TypeScript configured
- ✅ Prettier and ESLint set up
- ✅ Environment variables validated
- ✅ Database connection set up
- ✅ Authentication (if needed) implemented
- ✅ Error handling in place
- ✅ Basic tests written
- ✅ README with setup instructions
- ✅ `.gitignore` configured
- ✅ Accessibility audit passed

---

## Questions?

These standards are a guide, not gospel. If something doesn't make sense for your project, discuss it. Standards should evolve based on what we learn.

For questions or suggestions, raise an issue or discussion in the team Slack.

---

## Related Documents

- [Client Engagement Quick Reference](/standards/client-engagement-quick-reference.md)
- [Development Deployment Checklist](/playbooks/deployment-checklist.md) (future)
- [Testing Strategy](/playbooks/testing-strategy.md) (future)