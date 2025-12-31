# Configuração do Prisma - Guia de Reprodução

Este documento descreve todas as configurações do Prisma utilizadas neste projeto, permitindo reproduzir o mesmo padrão em outros projetos.

## 📦 Versões

### Dependências Principais

```json
{
  "@prisma/client": "^6.1.0",
  "prisma": "^6.1.0"
}
```

**Versões utilizadas:**
- `@prisma/client`: `^6.1.0` (dependência de produção)
- `prisma`: `^6.1.0` (dependência de desenvolvimento)

### Dependências Auxiliares

```json
{
  "tsx": "^4.16.2"
}
```

O `tsx` é necessário para executar scripts TypeScript diretamente (usado no seed e scripts de sincronização).

---

## 🗄️ Configuração do Banco de Dados

### Provider

O projeto utiliza **PostgreSQL** como banco de dados.

### String de Conexão

A conexão é configurada através da variável de ambiente `DATABASE_URL` no arquivo `.env`.

**Formato:**
```
postgresql://usuario:senha@host:porta/nome_do_banco?schema=public
```

**Exemplo:**
```env
DATABASE_URL="postgresql://postgres:minhasenha@localhost:5432/matheuslf?schema=public"
```

Para mais detalhes sobre configuração de variáveis de ambiente, consulte o arquivo `ENV_SETUP.md`.

---

## 📝 Schema do Prisma

### Localização

O arquivo de schema está localizado em:
```
prisma/schema.prisma
```

### Configuração do Generator

```prisma
generator client {
  provider = "prisma-client-js"
}
```

Esta configuração gera o Prisma Client em JavaScript/TypeScript.

### Configuração do Datasource

```prisma
datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}
```

- **Provider**: PostgreSQL
- **URL**: Lida da variável de ambiente `DATABASE_URL`

---

## 🔧 Configuração do Prisma Client

### Arquivo Singleton

O Prisma Client é configurado como um singleton para evitar múltiplas instâncias durante o desenvolvimento (especialmente importante no Next.js com Hot Module Replacement).

**Localização:** `lib/prisma.ts`

```typescript
import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const prisma = globalForPrisma.prisma ?? new PrismaClient()

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
```

### Como Funciona

1. **Singleton Pattern**: Reutiliza a mesma instância do Prisma Client quando já existe
2. **Global Storage**: Em desenvolvimento, armazena a instância no `globalThis` para persistir entre hot reloads
3. **Produção**: Em produção, cria uma nova instância a cada importação (comportamento padrão)

### Uso no Projeto

```typescript
import { prisma } from '@/lib/prisma'

// Usar em qualquer lugar do projeto
const users = await prisma.user.findMany()
```

---

## 📜 Scripts NPM

### Scripts Disponíveis

Os seguintes scripts estão configurados no `package.json`:

```json
{
  "scripts": {
    "db:generate": "prisma generate",
    "db:push": "prisma db push",
    "db:migrate": "prisma migrate dev",
    "db:studio": "prisma studio",
    "db:seed": "tsx prisma/seed.ts",
    "db:sync": "tsx scripts/sync-prisma.ts"
  }
}
```

### Descrição dos Scripts

| Script | Comando | Descrição |
|--------|---------|-----------|
| `db:generate` | `npm run db:generate` | Gera o Prisma Client baseado no schema |
| `db:push` | `npm run db:push` | Sincroniza o schema com o banco (sem criar migrations) |
| `db:migrate` | `npm run db:migrate` | Cria e aplica uma nova migration |
| `db:studio` | `npm run db:studio` | Abre o Prisma Studio (interface visual) |
| `db:seed` | `npm run db:seed` | Executa o script de seed do banco |
| `db:sync` | `npm run db:sync` | Sincroniza schema + regenera client (script customizado) |

### Configuração do Seed

O Prisma está configurado para executar o seed automaticamente:

```json
{
  "prisma": {
    "seed": "tsx prisma/seed.ts"
  }
}
```

Isso permite executar `npx prisma db seed` diretamente.

---

## 🚀 Passo a Passo para Reproduzir em Outros Projetos

### 1. Instalar Dependências

```bash
npm install @prisma/client@^6.1.0
npm install -D prisma@^6.1.0 tsx@^4.16.2
```

### 2. Inicializar o Prisma

```bash
npx prisma init
```

Isso criará:
- Pasta `prisma/` com `schema.prisma`
- Arquivo `.env` (se não existir)

### 3. Configurar o Schema

Edite `prisma/schema.prisma`:

```prisma
// This is your Prisma schema file,
// learn more about it in the docs: https://pris.ly/d/prisma-schema

generator client {
  provider = "prisma-client-js"
}

datasource db {
  provider = "postgresql"
  url      = env("DATABASE_URL")
}

// Adicione seus models aqui
model User {
  id        String   @id @default(cuid())
  email     String   @unique
  name      String?
  createdAt DateTime @default(now())
  updatedAt DateTime @updatedAt
}
```

### 4. Configurar a String de Conexão

No arquivo `.env`, configure a `DATABASE_URL`:

```env
DATABASE_URL="postgresql://usuario:senha@localhost:5432/nome_do_banco?schema=public"
```

### 5. Criar o Singleton do Prisma Client

Crie o arquivo `lib/prisma.ts` (ou `src/lib/prisma.ts`):

```typescript
import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const prisma = globalForPrisma.prisma ?? new PrismaClient()

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
```

### 6. Adicionar Scripts ao package.json

Adicione os scripts:

```json
{
  "scripts": {
    "db:generate": "prisma generate",
    "db:push": "prisma db push",
    "db:migrate": "prisma migrate dev",
    "db:studio": "prisma studio",
    "db:seed": "tsx prisma/seed.ts"
  },
  "prisma": {
    "seed": "tsx prisma/seed.ts"
  }
}
```

### 7. Gerar o Prisma Client

```bash
npm run db:generate
```

### 8. Sincronizar com o Banco de Dados

**Opção A - Push (desenvolvimento rápido):**
```bash
npm run db:push
```

**Opção B - Migrations (produção):**
```bash
npm run db:migrate
```

### 9. (Opcional) Criar Script de Seed

Crie `prisma/seed.ts`:

```typescript
import { PrismaClient } from '@prisma/client'

const prisma = new PrismaClient()

async function main() {
  // Seu código de seed aqui
  console.log('Seed executado com sucesso!')
}

main()
  .catch((e) => {
    console.error(e)
    process.exit(1)
  })
  .finally(async () => {
    await prisma.$disconnect()
  })
```

Execute o seed:
```bash
npm run db:seed
```

### 10. Usar o Prisma Client no Projeto

```typescript
import { prisma } from '@/lib/prisma'

// Exemplo em uma API Route (Next.js)
export async function GET() {
  const users = await prisma.user.findMany()
  return Response.json(users)
}
```

---

## 🔄 Script de Sincronização Customizado

Este projeto inclui um script customizado para sincronizar o schema e regenerar o client de forma segura.

### Localização

`scripts/sync-prisma.ts`

### Funcionalidades

1. Sincroniza o schema com o banco (`prisma db push`)
2. Regenera o Prisma Client (`prisma generate`)
3. Trata erros específicos do Windows (EPERM)
4. Detecta processos Node.js rodando
5. Retry automático em caso de arquivo em uso

### Uso

```bash
npm run db:sync
```

### Quando Usar

- Após modificar o `schema.prisma`
- Quando o Prisma Client precisa ser regenerado
- Para sincronizar mudanças sem criar migrations

### Nota Importante

No Windows, se o erro `EPERM` ocorrer, pare o servidor Next.js antes de executar o script.

---

## 📁 Estrutura de Arquivos

A estrutura de arquivos relacionada ao Prisma deve ser:

```
projeto/
├── prisma/
│   ├── schema.prisma          # Schema do banco de dados
│   ├── seed.ts                # Script de seed (opcional)
│   └── migrations/            # Migrations (se usar migrate)
│       └── ...
├── lib/
│   └── prisma.ts             # Singleton do Prisma Client
├── scripts/
│   └── sync-prisma.ts        # Script de sincronização (opcional)
├── .env                      # Variáveis de ambiente (NÃO commitar)
└── package.json              # Scripts e dependências
```

## 🔒 Configuração do .gitignore

Certifique-se de que o `.gitignore` inclui:

```gitignore
# local env files
.env*.local
.env

# prisma
/prisma/migrations
```

**Importante:**
- **NUNCA** commite o arquivo `.env` (contém credenciais)
- Se usar `db:push`, ignore a pasta `migrations`
- Se usar `db:migrate`, **commite** a pasta `migrations` (histórico de mudanças)

---

## 🎯 Boas Práticas Implementadas

### 1. Singleton Pattern
- Evita múltiplas instâncias do Prisma Client
- Importante para desenvolvimento com HMR

### 2. Variáveis de Ambiente
- Conexão configurada via `DATABASE_URL`
- Não hardcoded no código

### 3. Scripts Organizados
- Scripts npm com prefixo `db:`
- Fácil de lembrar e usar

### 4. TypeScript
- Prisma Client totalmente tipado
- Autocomplete e type safety

### 5. Seed Configurado
- Seed automático via `prisma.seed` no package.json
- Facilita popular dados iniciais

---

## 🔍 Verificação da Configuração

Para verificar se tudo está configurado corretamente:

```bash
# 1. Verificar se o Prisma Client foi gerado
npm run db:generate

# 2. Verificar conexão com o banco
npm run db:push

# 3. Abrir Prisma Studio para visualizar dados
npm run db:studio
```

Se todos os comandos executarem sem erros, a configuração está correta! ✅

---

## 📚 Recursos Adicionais

- [Documentação oficial do Prisma](https://www.prisma.io/docs)
- [Prisma com Next.js](https://www.prisma.io/docs/guides/performance-and-optimization/connection-management#prismaclient-in-serverless-environments)
- [Prisma Migrate](https://www.prisma.io/docs/concepts/components/prisma-migrate)
- [Prisma Studio](https://www.prisma.io/studio)

---

## ⚠️ Notas Importantes

1. **Desenvolvimento vs Produção**
   - Em desenvolvimento, o singleton persiste entre hot reloads
   - Em produção, cada instância é independente

2. **Migrations vs Push**
   - Use `db:push` para desenvolvimento rápido (este projeto usa esta abordagem)
   - Use `db:migrate` para produção (cria histórico de migrations)
   - **Nota**: Este projeto usa `db:push`, então a pasta `prisma/migrations` está no `.gitignore`

3. **Windows**
   - Se encontrar erros `EPERM`, pare o servidor Next.js antes de regenerar o client

4. **Versões**
   - Mantenha `@prisma/client` e `prisma` na mesma versão
   - Atualize ambos simultaneamente

5. **Gitignore**
   - A pasta `prisma/migrations` está ignorada no `.gitignore` (quando usar `db:push`)
   - O arquivo `.env` também está ignorado (nunca commite credenciais)

---

## 📝 Checklist de Configuração

Use este checklist ao configurar o Prisma em um novo projeto:

- [ ] Instalar `@prisma/client` e `prisma`
- [ ] Instalar `tsx` (para scripts TypeScript)
- [ ] Executar `npx prisma init`
- [ ] Configurar `schema.prisma` (generator e datasource)
- [ ] Configurar `DATABASE_URL` no `.env`
- [ ] Criar `lib/prisma.ts` com singleton
- [ ] Adicionar scripts ao `package.json`
- [ ] Configurar `prisma.seed` no `package.json`
- [ ] Executar `npm run db:generate`
- [ ] Executar `npm run db:push` ou `npm run db:migrate`
- [ ] (Opcional) Criar `prisma/seed.ts`
- [ ] Testar importação do Prisma Client no código

---

**Última atualização:** Baseado na configuração do projeto matheuslf-site
**Versão do Prisma:** 6.1.0
