# Documentação do Sistema Administrativo - Guia Genérico

## Índice

1. [Visão Geral](#visão-geral)
2. [Estrutura de Pastas](#estrutura-de-pastas)
3. [Tecnologias Utilizadas](#tecnologias-utilizadas)
4. [Sistema de Autenticação](#sistema-de-autenticação)
5. [Padrões de Módulos](#padrões-de-módulos)
6. [APIs do Admin](#apis-do-admin)
7. [Componentes Admin](#componentes-admin)
8. [Criando Novos Módulos](#criando-novos-módulos)

---

## Visão Geral

Este documento descreve a arquitetura e padrões de um sistema administrativo genérico construído com Next.js App Router. O sistema é protegido por autenticação baseada em cookies HTTP-only e segue padrões RESTful para APIs.

**Características principais**:
- Autenticação segura com cookies HTTP-only
- Estrutura modular e escalável
- APIs RESTful padronizadas
- Componentes reutilizáveis
- Validação de dados com Zod
- Revalidação inteligente de cache

---

## Estrutura de Pastas

### Estrutura Principal

```
app/
├── admin/                    # Área administrativa
│   ├── layout.tsx            # Layout principal (sidebar, header)
│   ├── page.tsx              # Dashboard com estatísticas
│   ├── login/                # Página de login
│   │   ├── layout.tsx        # Layout específico do login
│   │   └── page.tsx
│   ├── {resource}/           # Módulo de gerenciamento
│   │   ├── page.tsx          # Lista de itens
│   │   ├── new/
│   │   │   └── page.tsx      # Criar novo item
│   │   └── [id]/
│   │       └── page.tsx      # Editar item existente
│   ├── profile/              # Perfil do admin
│   │   └── page.tsx
│   └── settings/             # Configurações gerais (opcional)
│       └── page.tsx
│
├── api/                      # APIs do sistema
│   ├── admin/                # APIs administrativas
│   │   ├── {resource}/
│   │   │   ├── route.ts      # GET (listar), POST (criar)
│   │   │   └── [id]/
│   │   │       └── route.ts  # GET, PUT, DELETE
│   │   └── users/            # Gerenciamento de usuários
│   │       └── route.ts
│   └── auth/                 # APIs de autenticação
│       ├── login/
│       │   └── route.ts      # POST
│       ├── logout/
│       │   └── route.ts      # POST
│       └── check/
│           └── route.ts      # GET

components/
└── admin/                    # Componentes específicos do admin
    ├── admin-nav.tsx         # Navegação lateral (desktop)
    ├── admin-mobile-nav.tsx  # Navegação mobile
    ├── logout-button.tsx      # Botão de logout
    ├── {resource}-list.tsx   # Lista de itens
    └── {resource}-form.tsx   # Formulário de item

lib/
├── auth.ts                   # Funções de autenticação
├── prisma.ts                 # Cliente Prisma
└── revalidation.ts           # Funções de revalidação de cache

middleware.ts                 # Middleware de proteção de rotas
```

### Padrão de Roteamento

O sistema utiliza o **App Router do Next.js**, onde:

- **`page.tsx`**: Define uma página/rota
- **`layout.tsx`**: Define um layout compartilhado
- **`[id]`**: Rotas dinâmicas para edição (ex: `/admin/posts/123`)
- **`new`**: Rotas para criação de novos itens (ex: `/admin/posts/new`)

**Convenções**:
- Rotas de listagem: `/admin/{resource}`
- Rotas de criação: `/admin/{resource}/new`
- Rotas de edição: `/admin/{resource}/[id]`
- Rotas aninhadas: `/admin/{resource}/[id]/subresource`

---

## Tecnologias Utilizadas

### Framework e Core

- **Next.js** (App Router)
  - Server Components e Client Components
  - Server Actions
  - Route Handlers (API Routes)
  - File-based routing

- **React**
  - Hooks modernos
  - Componentes funcionais
  - Server Components

- **TypeScript**
  - Tipagem estática
  - Type safety
  - Melhor DX

### Banco de Dados

- **Prisma**
  - ORM type-safe
  - Migrations
  - Schema management
  - Suporte a múltiplos bancos (PostgreSQL, MySQL, SQLite, etc.)

### Autenticação

- **bcryptjs**
  - Hash de senhas
  - Verificação segura de credenciais

- **Cookies HTTP-only**
  - Armazenamento seguro de sessão
  - Proteção contra XSS

### UI e Estilização

- **Tailwind CSS**
  - Utility-first CSS
  - Responsive design
  - Customização via config

- **Radix UI**
  - Componentes acessíveis e não estilizados
  - Headless components
  - Customizáveis

- **Lucide React**
  - Biblioteca de ícones
  - Consistente e moderna

- **Framer Motion** (opcional)
  - Animações suaves
  - Transições

### Formulários e Validação

- **React Hook Form**
  - Gerenciamento de formulários
  - Performance otimizada
  - Validação integrada

- **Zod**
  - Validação de schemas
  - Type inference
  - Runtime validation

- **@hookform/resolvers**
  - Integração React Hook Form + Zod

### Editor de Texto Rico (Opcional)

- **TipTap**
  - Editor WYSIWYG extensível
  - Extensões modulares
  - Customizável

### Utilitários

- **clsx** e **tailwind-merge**
  - Combinação de classes CSS
  - Conditional classes

- **class-variance-authority**
  - Variantes de componentes
  - Type-safe variants

---

## Sistema de Autenticação

### Arquitetura

O sistema de autenticação utiliza uma abordagem baseada em **cookies HTTP-only** para segurança máxima. A autenticação é verificada em duas camadas:

1. **Middleware** (`middleware.ts`): Protege todas as rotas `/admin/*`
2. **Funções de autenticação** (`lib/auth.ts`): Validação de usuário e sessão

### Fluxo de Autenticação

#### 1. Login

**Endpoint**: `POST /api/auth/login`

**Body**:
```json
{
  "email": "user@example.com",
  "password": "password123"
}
```

**Processo**:
1. Validação do Content-Type (`application/json`)
2. Validação de email e senha obrigatórios
3. Busca do usuário no banco de dados
4. Verificação da senha usando `bcrypt.compare()`
5. Criação de cookie de sessão com configurações seguras

**Configuração do Cookie**:
```typescript
{
  httpOnly: true,        // Proteção contra XSS
  secure: process.env.NODE_ENV === "production", // Apenas HTTPS em produção
  sameSite: "lax",       // Proteção CSRF
  maxAge: 60 * 60 * 24 * 7, // 7 dias
  path: "/"
}
```

**Resposta de sucesso**:
```json
{
  "message": "Login realizado com sucesso",
  "user": {
    "id": "user_id",
    "email": "user@example.com",
    "name": "Nome do Usuário"
  }
}
```

#### 2. Middleware de Proteção

**Arquivo**: `middleware.ts`

```typescript
export function middleware(request: NextRequest) {
  const { pathname } = request.nextUrl;

  // Permitir acesso à página de login sem autenticação
  if (pathname === "/admin/login") {
    return NextResponse.next();
  }

  // Verificar autenticação para outras rotas admin
  if (pathname.startsWith("/admin")) {
    const sessionId = request.cookies.get("admin_session")?.value;

    if (!sessionId) {
      return NextResponse.redirect(new URL("/admin/login", request.url));
    }
  }

  return NextResponse.next();
}

export const config = {
  matcher: ["/admin/:path*"],
};
```

**Características**:
- Aplica-se a todas as rotas que começam com `/admin/:path*`
- Permite acesso livre apenas a `/admin/login`
- Redireciona automaticamente para login se não houver cookie de sessão

#### 3. Funções de Autenticação

**Arquivo**: `lib/auth.ts`

**Funções principais**:

- **`getCurrentUser()`**: Obtém o usuário atual da sessão
  ```typescript
  export async function getCurrentUser() {
    const cookieStore = await cookies();
    const sessionId = cookieStore.get(SESSION_COOKIE_NAME)?.value;
    
    if (!sessionId) return null;
    
    const user = await prisma.user.findUnique({
      where: { id: sessionId },
      select: { id: true, email: true, name: true }
    });
    
    return user;
  }
  ```

- **`requireAuth()`**: Garante que o usuário está autenticado
  ```typescript
  export async function requireAuth() {
    const user = await getCurrentUser();
    if (!user) {
      throw new Error("Unauthorized");
    }
    return user;
  }
  ```

- **`verifyPassword()`**: Verifica senha com bcrypt
  ```typescript
  export async function verifyPassword(
    plainPassword: string,
    hashedPassword: string
  ): Promise<boolean> {
    return bcrypt.compare(plainPassword, hashedPassword);
  }
  ```

- **`hashPassword()`**: Cria hash de senha
  ```typescript
  export async function hashPassword(password: string): Promise<string> {
    return bcrypt.hash(password, 10);
  }
  ```

#### 4. Verificação de Autenticação (API)

**Endpoint**: `GET /api/auth/check`

Retorna o status de autenticação:
```json
{ "authenticated": true }
```

Usado pela página de login para verificar se o usuário já está logado e redirecionar automaticamente.

#### 5. Logout

**Endpoint**: `POST /api/auth/logout`

**Processo**:
1. Remove o cookie `admin_session`
2. Retorna mensagem de sucesso

**Implementação**:
```typescript
export async function POST() {
  const cookieStore = await cookies();
  cookieStore.delete("admin_session");
  return NextResponse.json({ message: "Logout realizado com sucesso" });
}
```

### Página de Login

**Arquivo**: `app/admin/login/page.tsx`

**Características**:
- Componente Client Component (`"use client"`)
- Verifica autenticação ao carregar (redireciona se já logado)
- Formulário com validação
- Feedback de erros
- Loading state durante autenticação
- Redirecionamento automático após login bem-sucedido

**Layout de Login**:
- Layout separado (`app/admin/login/layout.tsx`)
- Não exibe sidebar ou header do admin
- Design limpo e focado

### Segurança

**Medidas implementadas**:

1. **Cookies HTTP-only**: Previne acesso via JavaScript (XSS)
2. **Cookies Secure em produção**: Apenas HTTPS
3. **SameSite Lax**: Proteção contra CSRF
4. **Hash de senhas**: bcrypt com salt rounds 10
5. **Middleware de proteção**: Todas as rotas admin protegidas
6. **Validação de entrada**: Zod schemas em todas as APIs
7. **Timeout de sessão**: Configurável (padrão: 7 dias)

**Melhorias recomendadas**:
- Implementar refresh tokens
- Adicionar rate limiting
- Implementar 2FA (autenticação de dois fatores)
- Logs de auditoria de acesso
- Sessões no banco de dados para controle mais granular
- Verificação de IP e device fingerprinting

---

## Padrões de Módulos

### Estrutura de um Módulo

Cada módulo administrativo segue uma estrutura padrão:

```
app/admin/{resource}/
├── page.tsx              # Lista de itens
├── new/
│   └── page.tsx         # Criar novo item
└── [id]/
    └── page.tsx         # Editar item existente
```

### Componentes de um Módulo

```
components/admin/
├── {resource}-list.tsx   # Componente de listagem
└── {resource}-form.tsx   # Componente de formulário
```

### APIs de um Módulo

```
app/api/admin/{resource}/
├── route.ts              # GET (listar), POST (criar)
└── [id]/
    └── route.ts          # GET, PUT, DELETE
```

### Padrão de Listagem

Todas as páginas de listagem seguem o padrão:
- Tabela ou grid com dados
- Ações (editar, deletar, visualizar)
- Filtros e busca
- Paginação (quando necessário)
- Status indicators
- Ordenação

### Padrão de Formulário

Todos os formulários seguem o padrão:
- React Hook Form para gerenciamento
- Zod para validação
- Componentes UI do Radix
- Feedback visual de erros
- Loading states
- Validação client-side e server-side
- Mensagens de sucesso/erro

### Padrão de API

Todas as APIs seguem o padrão RESTful:

```typescript
// GET /api/admin/{resource} - Listar todos
export async function GET() {
  const items = await prisma.{resource}.findMany({
    orderBy: { createdAt: "desc" }
  });
  return NextResponse.json(items);
}

// POST /api/admin/{resource} - Criar novo
export async function POST(request: NextRequest) {
  const body = await request.json();
  const validatedData = schema.parse(body);
  const item = await prisma.{resource}.create({
    data: validatedData
  });
  await revalidatePages();
  return NextResponse.json(item);
}

// GET /api/admin/{resource}/[id] - Obter por ID
export async function GET(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  const item = await prisma.{resource}.findUnique({
    where: { id }
  });
  if (!item) {
    return NextResponse.json({ error: "Not found" }, { status: 404 });
  }
  return NextResponse.json(item);
}

// PUT /api/admin/{resource}/[id] - Atualizar
export async function PUT(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  const body = await request.json();
  const validatedData = schema.parse(body);
  const item = await prisma.{resource}.update({
    where: { id },
    data: validatedData
  });
  await revalidatePages();
  return NextResponse.json(item);
}

// DELETE /api/admin/{resource}/[id] - Deletar
export async function DELETE(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  await prisma.{resource}.delete({
    where: { id }
  });
  await revalidatePages();
  return NextResponse.json({ message: "Deleted successfully" });
}
```

---

## APIs do Admin

### Estrutura de APIs

Todas as APIs administrativas seguem o padrão RESTful e estão localizadas em `app/api/admin/`.

### Padrão de Rotas

```
GET    /api/admin/{resource}           # Listar todos
POST   /api/admin/{resource}           # Criar novo
GET    /api/admin/{resource}/[id]      # Obter por ID
PUT    /api/admin/{resource}/[id]      # Atualizar
DELETE /api/admin/{resource}/[id]     # Deletar
```

### Validação

Todas as APIs utilizam **Zod** para validação de dados:

```typescript
import { z } from "zod";

const resourceSchema = z.object({
  title: z.string().min(1, "Título é obrigatório"),
  slug: z.string().min(1, "Slug é obrigatório"),
  description: z.string().optional(),
  published: z.boolean().default(false),
  // ... outros campos
});

// Uso na API
export async function POST(request: NextRequest) {
  try {
    const body = await request.json();
    const validatedData = resourceSchema.parse(body);
    // ... processar dados validados
  } catch (error) {
    if (error instanceof z.ZodError) {
      return NextResponse.json(
        { error: "Validation error", details: error.errors },
        { status: 400 }
      );
    }
    // ... outros erros
  }
}
```

### Tratamento de Erros

Padrão de tratamento de erros:

```typescript
try {
  // ... operação
} catch (error) {
  console.error("Erro:", error);
  return NextResponse.json(
    { error: "Mensagem de erro amigável" },
    { status: 500 }
  );
}
```

### Revalidação de Cache

Após operações de escrita (POST, PUT, DELETE), o sistema revalida as páginas afetadas:

```typescript
import { revalidatePath } from "next/cache";

// Revalidar página específica
revalidatePath("/admin/resource");
revalidatePath("/resource/[slug]", "page");

// Ou criar funções de revalidação customizadas
export async function revalidateResourcePages() {
  revalidatePath("/admin/resource");
  revalidatePath("/resource");
  revalidatePath("/");
}
```

**Padrão de revalidação**:
- Após criar/atualizar/deletar: revalidar listagem e páginas públicas relacionadas
- Após atualizar configurações: revalidar páginas que usam essas configurações
- Após atualizar conteúdo: revalidar página do conteúdo e listagens

---

## Componentes Admin

### Layout e Navegação

#### `AdminLayout` (`app/admin/layout.tsx`)

Layout principal que envolve todas as páginas admin:
- Header com título e informações do usuário
- Sidebar de navegação (desktop)
- Menu mobile
- Botão de logout
- Link para voltar ao site público

**Estrutura**:
```typescript
export default async function AdminLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  const user = await getCurrentUser();
  
  return (
    <div className="min-h-screen bg-background">
      <Header user={user} />
      <div className="flex">
        <Sidebar />
        <main className="flex-1">{children}</main>
      </div>
    </div>
  );
}
```

#### `AdminNav` (`components/admin/admin-nav.tsx`)

Navegação lateral para desktop:
- Lista de módulos com ícones
- Destaque da rota ativa
- Responsivo (oculto em mobile)

**Estrutura de navegação**:
```typescript
const navigation = [
  {
    name: "Dashboard",
    href: "/admin",
    icon: LayoutDashboard,
  },
  {
    name: "Recursos",
    href: "/admin/resources",
    icon: FileText,
  },
  // ... outros itens
];
```

#### `AdminMobileNav` (`components/admin/admin-mobile-nav.tsx`)

Navegação mobile:
- Menu dropdown
- Mesma estrutura do AdminNav
- Adaptado para telas pequenas

### Componentes de Formulário

Todos os formulários seguem o padrão:

```typescript
"use client";

import { useForm } from "react-hook-form";
import { zodResolver } from "@hookform/resolvers/zod";
import { z } from "zod";

const formSchema = z.object({
  // ... campos
});

export function ResourceForm({ initialData }: Props) {
  const form = useForm({
    resolver: zodResolver(formSchema),
    defaultValues: initialData || {},
  });

  const onSubmit = async (data: z.infer<typeof formSchema>) => {
    // ... enviar para API
  };

  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      {/* ... campos do formulário */}
    </form>
  );
}
```

**Características**:
- React Hook Form para gerenciamento
- Zod para validação
- Componentes UI do Radix
- Feedback visual de erros
- Loading states
- Validação client-side e server-side

### Componentes de Listagem

Todos os componentes de listagem seguem o padrão:

```typescript
export async function ResourceList() {
  const items = await fetch("/api/admin/resource").then(r => r.json());

  return (
    <div>
      <div className="flex justify-between items-center mb-4">
        <h1>Recursos</h1>
        <Button asChild>
          <Link href="/admin/resource/new">Criar Novo</Link>
        </Button>
      </div>
      <Table>
        {/* ... tabela com dados */}
      </Table>
    </div>
  );
}
```

**Características**:
- Tabela ou grid com dados
- Ações (editar, deletar, visualizar)
- Filtros e busca
- Paginação (quando necessário)
- Status indicators

### Componentes Especiais

- **`logout-button.tsx`**: Botão de logout com confirmação
- **Componentes de confirmação**: Dialogs para ações destrutivas
- **Componentes de status**: Badges e indicators
- **Componentes de loading**: Skeletons e spinners

---

## Criando Novos Módulos

### Passo a Passo

#### 1. Criar Modelo no Prisma

```prisma
model Resource {
  id          String   @id @default(cuid())
  title       String
  slug        String   @unique
  description String?  @db.Text
  content     String?  @db.Text
  published   Boolean  @default(false)
  createdAt   DateTime @default(now())
  updatedAt   DateTime @updatedAt
}
```

#### 2. Criar Schema de Validação

```typescript
// lib/schemas/resource.ts
import { z } from "zod";

export const resourceSchema = z.object({
  title: z.string().min(1, "Título é obrigatório"),
  slug: z.string().min(1, "Slug é obrigatório"),
  description: z.string().optional(),
  content: z.string().optional(),
  published: z.boolean().default(false),
});
```

#### 3. Criar APIs

```typescript
// app/api/admin/resource/route.ts
import { NextRequest, NextResponse } from "next/server";
import { prisma } from "@/lib/prisma";
import { resourceSchema } from "@/lib/schemas/resource";
import { revalidatePath } from "next/cache";

export async function GET() {
  const items = await prisma.resource.findMany({
    orderBy: { createdAt: "desc" },
  });
  return NextResponse.json(items);
}

export async function POST(request: NextRequest) {
  const body = await request.json();
  const data = resourceSchema.parse(body);
  const item = await prisma.resource.create({ data });
  revalidatePath("/admin/resource");
  return NextResponse.json(item);
}
```

```typescript
// app/api/admin/resource/[id]/route.ts
export async function GET(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  const item = await prisma.resource.findUnique({ where: { id } });
  if (!item) {
    return NextResponse.json({ error: "Not found" }, { status: 404 });
  }
  return NextResponse.json(item);
}

export async function PUT(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  const body = await request.json();
  const data = resourceSchema.parse(body);
  const item = await prisma.resource.update({
    where: { id },
    data,
  });
  revalidatePath("/admin/resource");
  return NextResponse.json(item);
}

export async function DELETE(
  request: NextRequest,
  { params }: { params: Promise<{ id: string }> }
) {
  const { id } = await params;
  await prisma.resource.delete({ where: { id } });
  revalidatePath("/admin/resource");
  return NextResponse.json({ message: "Deleted successfully" });
}
```

#### 4. Criar Componentes

```typescript
// components/admin/resource-list.tsx
export async function ResourceList() {
  const resources = await fetch(`${process.env.NEXT_PUBLIC_URL}/api/admin/resource`, {
    cache: "no-store",
  }).then((r) => r.json());

  return (
    <div>
      {/* ... implementação da lista */}
    </div>
  );
}
```

```typescript
// components/admin/resource-form.tsx
"use client";

export function ResourceForm({ initialData }: Props) {
  // ... implementação do formulário
}
```

#### 5. Criar Páginas

```typescript
// app/admin/resource/page.tsx
import { ResourceList } from "@/components/admin/resource-list";

export default function ResourcePage() {
  return <ResourceList />;
}
```

```typescript
// app/admin/resource/new/page.tsx
import { ResourceForm } from "@/components/admin/resource-form";

export default function NewResourcePage() {
  return <ResourceForm />;
}
```

```typescript
// app/admin/resource/[id]/page.tsx
import { ResourceForm } from "@/components/admin/resource-form";

export default async function EditResourcePage({
  params,
}: {
  params: Promise<{ id: string }>;
}) {
  const { id } = await params;
  const resource = await fetch(
    `${process.env.NEXT_PUBLIC_URL}/api/admin/resource/${id}`,
    { cache: "no-store" }
  ).then((r) => r.json());

  return <ResourceForm initialData={resource} />;
}
```

#### 6. Adicionar à Navegação

```typescript
// components/admin/admin-nav.tsx
const navigation = [
  // ... outros itens
  {
    name: "Recursos",
    href: "/admin/resource",
    icon: FileText,
  },
];
```

### Checklist de Criação de Módulo

- [ ] Modelo criado no Prisma
- [ ] Schema de validação Zod criado
- [ ] APIs CRUD implementadas
- [ ] Componente de listagem criado
- [ ] Componente de formulário criado
- [ ] Páginas criadas (listagem, novo, editar)
- [ ] Adicionado à navegação
- [ ] Revalidação de cache configurada
- [ ] Tratamento de erros implementado
- [ ] Validação client-side e server-side
- [ ] Loading states implementados
- [ ] Mensagens de feedback implementadas

---

## Conclusão

Este sistema administrativo foi desenvolvido com foco em:

1. **Segurança**: Autenticação robusta com cookies HTTP-only
2. **Organização**: Estrutura clara e modular
3. **Manutenibilidade**: Código tipado e bem documentado
4. **Escalabilidade**: Fácil adicionar novos módulos
5. **UX**: Interface intuitiva e responsiva
6. **Performance**: Server Components e revalidação inteligente de cache

O sistema permite gerenciar conteúdo de forma centralizada e segura, com uma arquitetura escalável e fácil de manter. Siga os padrões descritos neste documento para criar novos módulos e manter a consistência do sistema.
