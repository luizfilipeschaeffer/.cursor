---
name: ENV_SETUP
model: default
description: Instructions on how to set up, install, and configure the environment during project development.
---


---
name: ENV_SETUP
model: default
readonly: false
description: Instructions on how to set up, install, and configure the environment during project development.
---


# Configuração das Variáveis de Ambiente

## 📋 Variáveis Obrigatórias

### `DATABASE_URL` (OBRIGATÓRIO)
URL de conexão com o banco de dados PostgreSQL.

**Formato:**
```
postgresql://usuario:senha@host:porta/nome_do_banco?schema=public
```

**Exemplos:**

#### PostgreSQL Local
```env
DATABASE_URL="postgresql://postgres:minhasenha@localhost:5432/matheuslf?schema=public"
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

---

## 📧 Variáveis para Envio de Emails (Opcional)

### `RESEND_API_KEY` (Opcional - necessário para envio de emails)
Chave de API do Resend para envio de emails de avaliação de desafios.

**Como obter:**
1. Acesse https://resend.com
2. Crie uma conta ou faça login
3. Vá em "API Keys" e crie uma nova chave
4. Copie a chave (formato: `re_xxxxxxxxxxxxx`)

```env
RESEND_API_KEY="re_xxxxxxxxxxxxx"
```

### `RESEND_FROM_EMAIL` (Opcional - necessário para envio de emails)
Email remetente que será usado para enviar os emails. Deve ser um domínio verificado no Resend.

**Formato:**
```env
RESEND_FROM_EMAIL="noreply@seudominio.com.br"
```

**Nota:** Para desenvolvimento, você pode usar o domínio de teste do Resend (ex: `onboarding@resend.dev`). Em produção, você precisará verificar seu próprio domínio no Resend.

---

## 🔐 Variáveis Opcionais (para autenticação futura)

### `NEXTAUTH_URL`
URL base da aplicação. Usado pelo NextAuth quando implementado.

```env
NEXTAUTH_URL="http://localhost:3000"
```

**Em produção:**
```env
NEXTAUTH_URL="https://seudominio.com"
```

### `NEXTAUTH_SECRET`
Chave secreta para assinatura de tokens do NextAuth.

**⚠️ IMPORTANTE:** Gere uma chave aleatória segura!

Você pode gerar uma em: https://generate-secret.vercel.app/32

```env
NEXTAUTH_SECRET="sua-chave-secreta-aleatoria-aqui"
```

---

## 🚀 Como Configurar

### 1. Criar o arquivo `.env`

Na raiz do projeto, crie um arquivo chamado `.env`:

```bash
# Windows (PowerShell)
New-Item .env

# Linux/Mac
touch .env
```

### 2. Adicionar as variáveis

Copie e cole no arquivo `.env`:

```env
# OBRIGATÓRIO
DATABASE_URL="postgresql://usuario:senha@localhost:5432/matheuslf?schema=public"

# OPCIONAL (para envio de emails)
RESEND_API_KEY="re_xxxxxxxxxxxxx"
RESEND_FROM_EMAIL="noreply@seudominio.com.br"

# OPCIONAL (para autenticação futura)
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="gere-uma-chave-aleatoria-aqui"
```

### 3. Substituir os valores

- **usuario**: Seu usuário do PostgreSQL (geralmente `postgres`)
- **senha**: Sua senha do PostgreSQL
- **localhost**: Host do banco (ou IP/URL do serviço cloud)
- **5432**: Porta do PostgreSQL (padrão: 5432)
- **matheuslf**: Nome do banco de dados

---

## 📝 Exemplo Completo

### Desenvolvimento Local

```env
DATABASE_URL="postgresql://postgres:minhasenha123@localhost:5432/matheuslf?schema=public"
RESEND_API_KEY="re_xxxxxxxxxxxxx"
RESEND_FROM_EMAIL="onboarding@resend.dev"
NEXTAUTH_URL="http://localhost:3000"
NEXTAUTH_SECRET="abc123xyz456def789ghi012jkl345mno678pqr901stu234vwx567"
NODE_ENV="development"
```

### Produção

```env
DATABASE_URL="postgresql://user:senha@ep-xxx.neon.tech/dbname?sslmode=require"
RESEND_API_KEY="re_xxxxxxxxxxxxx"
RESEND_FROM_EMAIL="noreply@matheusleandro.com.br"
NEXTAUTH_URL="https://matheusleandro.com.br"
NEXTAUTH_SECRET="chave-super-secreta-para-producao"
NODE_ENV="production"
```

---

## ✅ Verificar se está funcionando

Após configurar o `.env`, execute:

```bash
# Gerar o cliente Prisma
npm run db:generate

# Testar conexão e criar tabelas
npm run db:push

# Popular dados iniciais
npm run db:seed
```

Se tudo estiver correto, você verá mensagens de sucesso! 🎉

---

## 🔒 Segurança

- ⚠️ **NUNCA** commite o arquivo `.env` no git
- ✅ O arquivo `.env` já está no `.gitignore`
- ✅ Use `.env.example` como referência (sem valores sensíveis)
- ✅ Em produção, use variáveis de ambiente do seu provedor (Vercel, Railway, etc.)
