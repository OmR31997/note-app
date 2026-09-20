# Full Stack Engineering Preparation Notes
**Topics Covered:** SOLID Principles | NestJS | Next.js | Docker | Deployment & DevOps

---

## Table of Contents
1. [SOLID Principles](#1-solid-principles)
2. [NestJS Architecture & Best Practices](#2-nestjs-architecture--best-practices)
3. [Next.js & Modern Web Development](#3-nextjs--modern-web-development)
4. [Docker & Containerization](#4-docker--containerization)
5. [Deployment, Infrastructure & CI/CD](#5-deployment-infrastructure--cicd)
6. [System Design & Integration Patterns](#6-system-design--integration-patterns)

---

## 1. SOLID Principles

### 1.1 Single Responsibility Principle (SRP)
* **Concept:** A class/module should have one, and only one, reason to change.
* **Bad Example:** A `UserService` that handles user DB saving, email sending, and JWT token generation.
* **Good Practice:** Break down into distinct services:
  ```typescript
  // User Service handles entity logic
  class UserService {
    constructor(private userRepository: UserRepository) {}
    async createUser(dto: CreateUserDto) { return this.userRepository.save(dto); }
  }

  // Mail Service handles notification delivery
  class MailService {
    async sendWelcomeEmail(email: string) { /* ... */ }
  }
  ```

### 1.2 Open/Closed Principle (OCP)
* **Concept:** Software entities should be open for extension, but closed for modification.
* **Pattern:** Strategy Pattern / Interfaces.
  ```typescript
  interface PaymentProcessor {
    process(amount: number): Promise<void>;
  }

  class StripePayment implements PaymentProcessor {
    async process(amount: number) { /* Stripe API call */ }
  }

  class PayPalPayment implements PaymentProcessor {
    async process(amount: number) { /* PayPal API call */ }
  }

  // Payment Context extends without modifying PaymentService
  class PaymentService {
    constructor(private processor: PaymentProcessor) {}
    execute(amount: number) { return this.processor.process(amount); }
  }
  ```

### 1.3 Liskov Substitution Principle (LSP)
* **Concept:** Subtypes must be substitutable for their base types without altering program correctness.
* **Rule:** Subclasses should never throw unexpected exceptions or break invariants established by parent classes.

### 1.4 Interface Segregation Principle (ISP)
* **Concept:** Clients should not be forced to depend upon interfaces they do not use.
* **Good Practice:** Split fat interfaces into smaller, specific role interfaces.
  ```typescript
  interface Reader {
    read(): string;
  }

  interface Writer {
    write(data: string): void;
  }

  // Read-only client depends only on Reader
  class ReportGenerator {
    constructor(private reader: Reader) {}
  }
  ```

### 1.5 Dependency Inversion Principle (DIP)
* **Concept:** High-level modules should not depend on low-level modules. Both should depend on abstractions.
* **Implementation in NestJS:** Using Symbol/Interface tokens with NestJS Dependency Injection.

---

## 2. NestJS Architecture & Best Practices

### 2.1 Core Pillars
* **Modules (`@Module`):** Encapsulates domain contexts (controllers, providers, imports, exports).
* **Controllers (`@Controller`):** Handles incoming HTTP requests and routes them to service methods.
* **Providers / Services (`@Injectable`):** Handles business logic, data persistence, and external calls.

### 2.2 Request Lifecycle Sequence
1. **Incoming Request**
2. **Middleware** (Global / Module-bound)
3. **Guards** (Authentication / Authorization - `@UseGuards()`)
4. **Interceptors (Pre-controller)** (Transforming request/logging)
5. **Pipes** (Validation / Transformation - `ValidationPipe`, `@UsePipes()`)
6. **Controller Handler** (Business logic invocation)
7. **Interceptors (Post-controller)** (Response formatting / caching)
8. **Exception Filters** (Catching errors - `@Catch()`)
9. **Outgoing Response**

### 2.3 Dependency Injection (DI) & Scopes
* **DEFAULT:** Singleton instance shared across the entire application lifecycle (Best for performance).
* **REQUEST:** A new instance is created per incoming request (Use with caution: performance impact).
* **TRANSIENT:** A new instance is injected into each consumer.

### 2.4 Advanced NestJS Concepts
* **Custom Decorators:**
  ```typescript
  import { createParamDecorator, ExecutionContext } from '@nestjs/common';

  export const CurrentUser = createParamDecorator(
    (data: keyof UserEntity, ctx: ExecutionContext) => {
      const request = ctx.switchToHttp().getRequest();
      return data ? request.user?.[data] : request.user;
    },
  );
  ```
* **Microservices Support:** TCP, Redis, NATS, RabbitMQ, gRPC bindings via `@nestjs/microservices`.
* **Database Management:** TypeORM / Prisma / Drizzle ORM integration using Repository patterns and async module registration (`forRootAsync`).

---

## 3. Next.js & Modern Web Development

### 3.1 App Router architecture (`/app` directory)
* Built on React Server Components (RSC).
* File-system based routing: `page.tsx`, `layout.tsx`, `loading.tsx`, `error.tsx`, `route.ts`.

### 3.2 Server vs. Client Components
| Feature | React Server Components (RSC) | Client Components (`'use client'`) |
| :--- | :--- | :--- |
| **Execution** | Server-only (Zero bundle size impact) | Pre-rendered on server, hydrated on client |
| **Data Access** | Direct DB / File System access | API calls (`fetch` / SWR / TanStack Query) |
| **Interactivity** | None (no state, no effects) | Full (`useState`, `useEffect`, Event Listeners) |
| **Default in App Router** | **Yes** | No (requires explicit `'use client'`) |

### 3.3 Rendering Strategies
* **SSR (Server-Side Rendering):** Dynamic rendering per request (`dynamic = 'force-dynamic'` or `cache: 'no-store'`).
* **SSG (Static Site Generation):** Pre-rendered at build time (`cache: 'force-cache'`).
* **ISR (Incremental Static Revalidation):** Static pages updated periodically on-demand:
  ```typescript
  // Revalidate fetch every 60 seconds
  const res = await fetch('https://api.example.com/data', { next: { revalidate: 60 } });
  ```
* **CSR (Client-Side Rendering):** Standard SPA behavior using React hooks in Client Components.

### 3.4 Server Actions & Data Mutations
* Execute async functions on the server directly from forms or components.
  ```typescript
  // app/actions.ts
  'use server';

  import { revalidatePath } from 'next/cache';

  export async function updateProfile(formData: FormData) {
    const name = formData.get('name');
    await db.user.update({ where: { id: 1 }, data: { name } });
    revalidatePath('/profile');
  }
  ```

### 3.5 Next.js Performance Optimization
* `next/image`: Automatic webp/avif generation, cumulative layout shift (CLS) prevention, responsive sizing.
* `next/font`: Zero-layout-shift web fonts loading self-hosted.
* Core Web Vitals targets: LCP < 2.5s, FID/INP < 200ms, CLS < 0.1.

---

## 4. Docker & Containerization

### 4.1 Core Concepts
* **Image:** Read-only blueprint containing application code, environment, and runtime dependencies.
* **Container:** A running instance of a Docker image (isolated process execution space).
* **Volume:** Persistent storage decoupled from container lifecycle.
* **Bridge Network:** Default isolated network driver for local containers to communicate.

### 4.2 Multi-Stage Dockerfile (Production Best Practice for NestJS)
```dockerfile
# Stage 1: Build stage
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
RUN npm run build

# Stage 2: Production runtime stage
FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

# Security: Non-root user
USER node

COPY package*.json ./
RUN npm ci --only=production

COPY --from=builder /app/dist ./dist

EXPOSE 3000
CMD ["node", "dist/main.js"]
```

### 4.3 Multi-Stage Dockerfile (Next.js Standalone Output)
```dockerfile
FROM node:20-alpine AS builder
WORKDIR /app
COPY package*.json ./
RUN npm ci
COPY . .
# Ensure next.config.js has output: 'standalone'
RUN npm run build

FROM node:20-alpine AS runner
WORKDIR /app
ENV NODE_ENV=production

USER node

COPY --from=builder /app/public ./public
COPY --from=builder /app/.next/standalone ./
COPY --from=builder /app/.next/static ./.next/static

EXPOSE 3000
CMD ["node", "server.js"]
```

### 4.4 Docker Compose (`docker-compose.yml`)
```yaml
version: '3.8'

services:
  api:
    build:
      context: ./backend
      dockerfile: Dockerfile
    ports:
      - "3000:3000"
    environment:
      - DATABASE_URL=postgres://user:pass@db:5432/mydb
    depends_on:
      - db
    restart: always

  frontend:
    build:
      context: ./frontend
      dockerfile: Dockerfile
    ports:
      - "3001:3000"
    depends_on:
      - api

  db:
    image: postgres:16-alpine
    environment:
      POSTGRES_USER: user
      POSTGRES_PASSWORD: pass
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data

volumes:
  pgdata:
```

---

## 5. Deployment, Infrastructure & CI/CD

### 5.1 Nginx Reverse Proxy Setup
```nginx
server {
    listen 80;
    server_name api.example.com;

    location / {
        proxy_pass http://localhost:3000; # NestJS backend
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_cache_bypass $http_upgrade;
    }
}
```

### 5.2 Process Management with PM2
* **Cluster Mode:** Utilizes all CPU cores (`pm2 start dist/main.js -i max`).
* **Zero-Downtime Reload:** `pm2 reload all`.
* `ecosystem.config.js`:
  ```javascript
  module.exports = {
    apps: [{
      name: 'nestjs-app',
      script: './dist/main.js',
      instances: 'max',
      exec_mode: 'cluster',
      env_production: {
        NODE_ENV: 'production'
      }
    }]
  };
  ```

### 5.3 CI/CD Pipeline (GitHub Actions Example)
```yaml
name: Production CI/CD

on:
  push:
    branches: [ main ]

jobs:
  test-and-deploy:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: 20
          cache: 'npm'

      - name: Install dependencies & Test
        run: |
          npm ci
          npm run test
          npm run test:e2e

      - name: Build & Push Docker Image
        run: |
          docker build -t myregistry/app:${{ github.sha }} .
          # Push to container registry...
```

### 5.4 Production Checklist
* **SSL/TLS Configuration:** Certbot / Let's Encrypt automated renewal.
* **Environment Variables:** Never commit `.env` files; use secret managers (AWS Secrets Manager, Vault, GitHub Secrets).
* **Security Headers:** CORS policy enforcement, Helmet middleware (`helmet()` in NestJS), CSP headers in Next.js.
* **Health Checks:** Implement `/health` endpoints using `@nestjs/terminus` for liveness and readiness probes in Kubernetes / Docker Swarm.

---

## 6. System Design & Integration Patterns

### 6.1 Full Stack Architecture Diagram
```
[ Browser / Client ]
       │
       ▼
 [ Cloudflare / CDN ] (Static Assets & Edge Caching)
       │
       ▼
  [ Nginx Proxy ] (SSL Termination, Rate Limiting)
   ┌───┴───────────────────────┐
   ▼                           ▼
[ Next.js Frontend ]    [ NestJS Backend API ]
(App Router RSC)         (Controllers, DI Services)
                               │
               ┌───────────────┼───────────────┐
               ▼               ▼               ▼
         [ PostgreSQL ]    [ Redis Cache ]  [ Message Queue / Kafka ]
```

### 6.2 Common Interview Q&A Cheatsheet
1. **Q: How do Next.js Server Components improve performance over standard SPAs?**
   * *A:* RSCs render purely on the server and send rendered HTML + React Server Component Payload to the client. JS dependencies used inside server components are NOT included in the client bundle, reducing bundle size and improving initial page load (FCP & LCP).
2. **Q: Why use NestJS Guards instead of custom Middleware?**
   * *A:* Middleware has no access to NestJS execution context (`ExecutionContext`). Guards have full access to metadata and reflection, allowing route-level dynamic authorization based on user roles and custom decorators.
3. **Q: How does a multi-stage Docker build reduce image size?**
   * *A:* It separates dev tools, TypeScript compilers, test runners, and raw source code from the final runtime image. Only compiled artifacts (`dist`) and production `node_modules` are kept in the final lightweight base image (e.g. `alpine`).
4. **Q: How to prevent race conditions during concurrent database updates?**
   * *A:* Use database transactions with pessimistic locking (`SELECT ... FOR UPDATE`) or optimistic locking (version column comparison).
