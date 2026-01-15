---
name: skill-prisma
model: default
description: Instructions on how to set up, install, and configure Prisma during project development.
---




---
name: skill-prisma
model: default
readonly: false
description: Instructions on how to set up, install, and configure the prisma during project development.
---

# Guia Completo de Configuração do Prisma - Padrão de Projeto

## Objetivo

Este documento serve como guia de referência completo para configurar o Prisma em novos projetos, baseado na configuração padrão estabelecida no projeto matheuslf-site.

## Informações da Configuração Atual

### Versões

- **@prisma/client**: `^6.1.0`
- **prisma** (CLI): `^6.1.0`
- **TypeScript**: `^5.5.4`
- **Node.js**: Requerido (versão compatível com Next.js 16.1.1)

### Banco de Dados

- **Provider**: PostgreSQL
- **Formato da URL**: `postgresql://usuario:senha@host:porta/nome_do_banco?schema=public`

## Estrutura de Arquivos do Prisma

```
projeto/
├── prisma/
│   ├── schema.prisma      # Schema principal do banco de dados
│   └── seed.ts            # Script de seed para popular dados iniciais
├── lib/
│   └── prisma.ts          # Configuração do Prisma Client singleton
├── package.json           # Dependências e scripts do Prisma
└── .env                   # Variáveis de ambiente (NÃO versionado)
```

## 1. Instalação e Dependências

### Instalar o Prisma

```bash
# Instalar CLI do Prisma como devDependency
npm install -D prisma

# Instalar Prisma Client como dependency
npm install @prisma/client
```

### Dependências Adicionais Necessárias

```bash
# Para executar scripts TypeScript (seed, sync, etc)
npm install -D tsx

# Para hash de senhas (se necessário)
npm install bcryptjs
npm install -D @types/bcryptjs
```

## 2. Configuração do Schema (`prisma/schema.prisma`)

### Estrutura Básica

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
```

**Características importantes:**

- Generator usa `prisma-client-js` (padrão para JavaScript/TypeScript)
- Datasource aponta para PostgreSQL
- URL lida de variável de ambiente `DATABASE_URL`

### Tipos de Campos Comuns

- `@id @default(cuid())` - ID único usando CUID
- `@unique` - Campo único
- `@db.Text` - Texto longo (TEXT no PostgreSQL)
- `@default(now())` - Data/hora padrão atual
- `@updatedAt` - Atualiza automaticamente na modificação
- `@relation` - Relacionamentos entre models

## 3. Configuração do Prisma Client (`lib/prisma.ts`)

### Padrão Singleton para Next.js

```typescript
import { PrismaClient } from '@prisma/client'

const globalForPrisma = globalThis as unknown as {
  prisma: PrismaClient | undefined
}

export const prisma = globalForPrisma.prisma ?? new PrismaClient()

if (process.env.NODE_ENV !== 'production') globalForPrisma.prisma = prisma
```

**Por que usar este padrão:**

- Previne múltiplas instâncias do Prisma Client no desenvolvimento (hot-reload)
- Reutiliza a mesma conexão
- Evita problemas de conexões esgotadas

## 4. Variáveis de Ambiente (`.env`)

### Variável Obrigatória

```env
DATABASE_URL="postgresql://usuario:senha@host:porta/nome_do_banco?schema=public"
```

### Formatos por Provedor

#### PostgreSQL Local

```env
DATABASE_URL="postgresql://postgres:minhasenha@localhost:5432/nome_do_banco?schema=public"
```

#### Neon (Cloud PostgreSQL)

```env
DATABASE_URL="postgresql://usuario:senha@ep-xxx-xxx.region.aws.neon.tech/dbname?sslmode=require"
```

#### Supabase

```env
DATABASE_URL="postgresql://postgres:senha@db.xxx.supabase.co:5432/postgres"
```

#### Railway

```env
DATABASE_URL="postgresql://postgres:senha@containers-us-west-xxx.railway.app:5432/railway"
```

### Segurança

- **NUNCA** commite o arquivo `.env` no git
- Adicione `.env` ao `.gitignore`
- Use `.env.example` como referência (sem valores sensíveis)

## 5. Scripts do package.json

### Scripts Essenciais

```json
{
  "scripts": {
    "db:generate": "prisma generate",
    "db:push": "prisma db push",
    "db:migrate": "prisma migrate dev",
    "db:studio": "prisma studio",
    "db:seed": "tsx prisma/seed.ts",
    "db:sync": "tsx scripts/sync-prisma.ts"
  },
  "prisma": {
    "seed": "tsx prisma/seed.ts"
  }
}
```

### Descrição dos Scripts

- `db:generate` - Gera o Prisma Client baseado no schema
- `db:push` - Sincroniza schema com banco (desenvolvimento rápido)
- `db:migrate` - Cria e aplica migrações (produção)
- `db:studio` - Abre interface visual do Prisma Studio
- `db:seed` - Executa script de seed
- `db:sync` - Sincroniza schema e regenera client (script customizado)

### Diferença: `db:push` vs `db:migrate`

- **`db:push`**: Ideal para desenvolvimento. Não cria histórico de migrações. Atualiza o banco diretamente.
- **`db:migrate`**: Ideal para produção. Cria histórico de migrações em `prisma/migrations/`. Permite rollback.

## 6. Script de Seed (`prisma/seed.ts`)

### Estrutura Básica

```typescript
import { PrismaClient } from "@prisma/client";

const prisma = new PrismaClient();

async function main() {
  // Seu código de seed aqui
  console.log("🌱 Iniciando seed...");
  
  // Exemplo: criar dados iniciais
  await prisma.modelName.create({
    data: {
      // dados...
    }
  });
  
  console.log("🎉 Seed concluído!");
}

main()
  .catch((e) => {
    console.error("❌ Erro no seed:", e);
    process.exit(1);
  })
  .finally(async () => {
    await prisma.$disconnect();
  });
```

### Boas Práticas

- Use `upsert` para evitar duplicação
- Trate erros adequadamente
- Sempre desconecte o Prisma Client no final
- Use logs informativos

## 7. Fluxo de Trabalho Inicial

### 1. Instalar Dependências

```bash
npm install
```

### 2. Configurar `.env`

Criar arquivo `.env` na raiz com `DATABASE_URL` configurada.

### 3. Gerar Prisma Client

```bash
npm run db:generate
```

### 4. Sincronizar Schema com Banco

```bash
# Desenvolvimento (rápido)
npm run db:push

# OU Produção (com migrações)
npm run db:migrate
```

### 5. Popular Dados Iniciais (Opcional)

```bash
npm run db:seed
```

### 6. Iniciar Desenvolvimento

```bash
npm run dev
```

## 8. Fluxo de Trabalho Durante Desenvolvimento

### Ao Modificar o Schema

1. Edite `prisma/schema.prisma`
2. Sincronize com banco:
   ```bash
   npm run db:sync  # Script customizado que faz push + generate
   # OU manualmente:
   npm run db:push
   npm run db:generate
   ```

3. Reinicie o servidor Next.js se estiver rodando

### Usando Migrações (Produção)

1. Edite `prisma/schema.prisma`
2. Crie migração:
   ```bash
   npm run db:migrate
   ```

3. Dê um nome descritivo à migração
4. Migração será criada em `prisma/migrations/`

## 9. Script de Sincronização Personalizado

O projeto inclui `scripts/sync-prisma.ts` que:

- Faz `db:push` (sincroniza schema)
- Faz `generate` (regenera client)
- Trata erros no Windows (EPERM)
- Fornece feedback visual

**Uso:**

```bash
npm run db:sync
```

## 10. Checklist de Configuração para Novos Projetos

- [ ] Instalar `prisma` e `@prisma/client`
- [ ] Instalar `tsx` para executar scripts TypeScript
- [ ] Criar diretório `prisma/`
- [ ] Criar `prisma/schema.prisma` com generator e datasource
- [ ] Criar `lib/prisma.ts` com padrão singleton
- [ ] Criar `.env` com `DATABASE_URL`
- [ ] Adicionar `.env` ao `.gitignore`
- [ ] Adicionar scripts no `package.json`
- [ ] (Opcional) Criar `prisma/seed.ts`
- [ ] Executar `npm run db:generate`
- [ ] Executar `npm run db:push` ou `npm run db:migrate`
- [ ] (Opcional) Executar `npm run db:seed`

## 11. Troubleshooting Comum

### Erro: "Prisma Client não encontrado"

**Solução:** Execute `npm run db:generate`

### Erro: "Can't reach database server"

**Solução:** Verifique se `DATABASE_URL` está correta e se o banco está rodando

### Erro: "EPERM" no Windows ao gerar client

**Solução:** Pare o servidor Next.js e tente novamente, ou use `npm run db:sync`

### Erro: "Migration failed"

**Solução:** Verifique conflitos no schema. Use `db:push` para desenvolvimento ou resolva conflitos manualmente

## 12. Arquivos de Referência do Projeto

- `prisma/schema.prisma` - Schema completo com todos os models
- `lib/prisma.ts` - Configuração do Prisma Client
- `prisma/seed.ts` - Exemplo completo de seed
- `package.json` - Scripts e dependências
- `ENV_SETUP.md` - Documentação de variáveis de ambiente
- `scripts/sync-prisma.ts` - Script de sincronização customizado

## 13. Recursos Adicionais

- [Documentação Oficial do Prisma](https://www.prisma.io/docs)
- [Prisma Schema Reference](https://www.prisma.io/docs/reference/api-reference/prisma-schema-reference)
- [Prisma Migrate Guide](https://www.prisma.io/docs/concepts/components/prisma-migrate)

---

**Última atualização:** Baseado na configuração do projeto matheuslf-site

**Versão do Prisma documentada:** 6.1.0
