# คู่มือการใช้งาน Cloudflare Wrangler CLI

> **Wrangler** คือ Command-Line Interface (CLI) อย่างเป็นทางการของ Cloudflare สำหรับพัฒนาและ deploy Workers, Pages, KV, D1, R2 และบริการอื่น ๆ บน Cloudflare Platform

---

## สารบัญ

1. [ความต้องการของระบบ](#1-ความต้องการของระบบ)
2. [ติดตั้ง Wrangler](#2-ติดตั้ง-wrangler)
3. [Login เข้า Cloudflare](#3-login-เข้า-cloudflare)
4. [สร้างโปรเจกต์ใหม่](#4-สร้างโปรเจกต์ใหม่)
5. [โครงสร้างไฟล์โปรเจกต์](#5-โครงสร้างไฟล์โปรเจกต์)
6. [ไฟล์ wrangler.toml](#6-ไฟล์-wranglertoml)
7. [พัฒนาในเครื่อง (Local Development)](#7-พัฒนาในเครื่อง-local-development)
8. [Deploy ขึ้น Cloudflare](#8-deploy-ขึ้น-cloudflare)
9. [จัดการ Environments](#9-จัดการ-environments)
10. [Secret และ Environment Variables](#10-secret-และ-environment-variables)
11. [KV Namespace](#11-kv-namespace)
12. [D1 Database](#12-d1-database)
13. [R2 Bucket](#13-r2-bucket)
14. [Cloudflare Pages](#14-cloudflare-pages)
15. [ดู Logs](#15-ดู-logs)
16. [Rollback และการจัดการ Deployments](#16-rollback-และการจัดการ-deployments)
17. [Pull โปรเจกต์จาก Cloudflare](#17-pull-โปรเจกต์จาก-cloudflare)
18. [คำสั่งที่ใช้บ่อย (Cheat Sheet)](#18-คำสั่งที่ใช้บ่อย-cheat-sheet)

---

## 1. ความต้องการของระบบ

| สิ่งที่ต้องมี | เวอร์ชันขั้นต่ำ |
|---|---|
| Node.js | >= 18.0.0 |
| npm / yarn / pnpm | ล่าสุด |
| บัญชี Cloudflare | ฟรี (cloudflare.com) |

ตรวจสอบเวอร์ชัน Node.js:

```bash
node --version
npm --version
```

---

## 2. ติดตั้ง Wrangler

### ติดตั้งแบบ Global (แนะนำ)

```bash
npm install -g wrangler
```

### ติดตั้งแบบ Project (devDependency)

```bash
npm install --save-dev wrangler
```

### ตรวจสอบเวอร์ชัน

```bash
wrangler --version
# หรือ
wrangler version
```

### อัปเดต Wrangler

```bash
npm update -g wrangler
```

---

## 3. Login เข้า Cloudflare

### Login ผ่าน Browser

```bash
wrangler login
```

> เปิด browser อัตโนมัติ → กด **Allow** → กลับมาที่ Terminal ✅

### ตรวจสอบสถานะ Login

```bash
wrangler whoami
```

ผลลัพธ์:

```
👋 You are logged in with an OAuth Token, associated with the email: you@example.com
┌──────────────────────┬──────────────────────────────────┐
│ Account Name         │ Account ID                       │
├──────────────────────┼──────────────────────────────────┤
│ My Cloudflare Acct   │ abc123def456...                  │
└──────────────────────┴──────────────────────────────────┘
```

### Logout

```bash
wrangler logout
```

### Login ด้วย API Token (CI/CD)

สร้าง API Token ที่ [dash.cloudflare.com/profile/api-tokens](https://dash.cloudflare.com/profile/api-tokens) แล้วตั้ง environment variable:

```bash
export CLOUDFLARE_API_TOKEN="your_api_token_here"
```

---

## 4. สร้างโปรเจกต์ใหม่

### สร้าง Worker ใหม่

```bash
# สร้างแบบ Interactive (แนะนำ)
wrangler init my-worker

# หรือใช้ npm create
npm create cloudflare@latest my-worker
```

เลือก template ที่ต้องการ:

```
? What would you like to start with?
  ❯ Hello World example
    Framework Starter
    Demo application
    ...
```

### Template ยอดนิยม

```bash
# Workers + TypeScript
npm create cloudflare@latest my-worker -- --type=hello-world --lang=ts

# Workers + Hono Framework
npm create cloudflare@latest my-app -- --type=hello-world --framework=hono

# Cloudflare Pages
npm create cloudflare@latest my-pages -- --type=pages
```

---

## 5. โครงสร้างไฟล์โปรเจกต์

```
my-worker/
├── src/
│   └── index.ts          # Entry point หลัก
├── test/
│   └── index.spec.ts     # Unit tests
├── wrangler.toml          # Config ของ Wrangler
├── package.json
├── tsconfig.json
└── .dev.vars              # Environment variables สำหรับ local (ห้าม commit!)
```

### ตัวอย่าง src/index.ts เบื้องต้น

```typescript
export default {
  async fetch(request: Request, env: Env, ctx: ExecutionContext): Promise<Response> {
    return new Response('Hello, World!');
  },
};
```

---

## 6. ไฟล์ wrangler.toml

ไฟล์ config หลักของโปรเจกต์:

```toml
name = "my-worker"                    # ชื่อ Worker
main = "src/index.ts"                 # Entry point
compatibility_date = "2024-01-01"     # Compatibility date
compatibility_flags = ["nodejs_compat"]

# Account (ถ้ามีหลาย account)
# account_id = "your_account_id"

# Routes (ถ้าต้องการผูกกับ domain)
# routes = [
#   { pattern = "example.com/api/*", zone_name = "example.com" }
# ]

# Workers.dev subdomain
workers_dev = true

# Environment variables (ไม่ใช่ secret)
[vars]
ENVIRONMENT = "production"
API_URL = "https://api.example.com"

# KV Namespace
[[kv_namespaces]]
binding = "MY_KV"
id = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# D1 Database
[[d1_databases]]
binding = "DB"
database_name = "my-database"
database_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"

# R2 Bucket
[[r2_buckets]]
binding = "MY_BUCKET"
bucket_name = "my-bucket"
```

---

## 7. พัฒนาในเครื่อง (Local Development)

### เริ่ม Dev Server

```bash
wrangler dev
```

Worker จะรันที่ `http://localhost:8787` โดย default

### ตั้ง Port เอง

```bash
wrangler dev --port 3000
```

### Remote Mode (ใช้ Cloudflare infra จริง)

```bash
wrangler dev --remote
```

> ⚠️ Remote mode จะใช้ KV, D1, R2 จริงบน Cloudflare

### ตั้ง Environment Variables สำหรับ Local

สร้างไฟล์ `.dev.vars` (ไม่ต้อง commit):

```
MY_SECRET=local_secret_value
DATABASE_URL=postgresql://localhost/mydb
```

---

## 8. Deploy ขึ้น Cloudflare

### Deploy ครั้งแรก / อัปเดต

```bash
wrangler deploy
```

หรือ alias เก่า:

```bash
wrangler publish   # deprecated แต่ยังใช้ได้
```

### Deploy พร้อมระบุ Environment

```bash
wrangler deploy --env staging
wrangler deploy --env production
```

### Deploy แบบ Dry Run (ทดสอบโดยไม่ deploy จริง)

```bash
wrangler deploy --dry-run
```

### ผลลัพธ์หลัง Deploy

```
Total Upload: 12.34 KiB / gzip: 4.56 KiB
Uploaded my-worker (1.23 sec)
Published my-worker (0.45 sec)
  https://my-worker.your-subdomain.workers.dev
Current Deployment ID: abc123xyz...
```

---

## 9. จัดการ Environments

### กำหนด Environment ใน wrangler.toml

```toml
name = "my-worker"
main = "src/index.ts"
compatibility_date = "2024-01-01"

[vars]
ENVIRONMENT = "production"

# Staging Environment
[env.staging]
name = "my-worker-staging"
workers_dev = true

  [env.staging.vars]
  ENVIRONMENT = "staging"

# Production Environment
[env.production]
name = "my-worker-prod"
workers_dev = false
routes = [{ pattern = "example.com/api/*", zone_name = "example.com" }]

  [env.production.vars]
  ENVIRONMENT = "production"
```

### ใช้งาน Environment

```bash
# Dev ด้วย staging config
wrangler dev --env staging

# Deploy ไป staging
wrangler deploy --env staging

# Deploy ไป production
wrangler deploy --env production
```

---

## 10. Secret และ Environment Variables

### เพิ่ม Secret (Encrypted)

```bash
wrangler secret put MY_SECRET_KEY
# พิมพ์ค่าแล้วกด Enter
```

### เพิ่ม Secret สำหรับ Environment เฉพาะ

```bash
wrangler secret put MY_SECRET_KEY --env production
```

### ดูรายการ Secrets

```bash
wrangler secret list
wrangler secret list --env production
```

### ลบ Secret

```bash
wrangler secret delete MY_SECRET_KEY
```

### Bulk Upload Secrets จากไฟล์

```bash
# สร้างไฟล์ .secrets
echo 'KEY1=value1
KEY2=value2' | wrangler secret bulk
```

---

## 11. KV Namespace

### สร้าง KV Namespace

```bash
# Production
wrangler kv namespace create "MY_KV"

# Preview (สำหรับ dev)
wrangler kv namespace create "MY_KV" --preview
```

นำ ID ที่ได้ไปใส่ใน `wrangler.toml`:

```toml
[[kv_namespaces]]
binding = "MY_KV"
id = "xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
preview_id = "yyyyyyyyyyyyyyyyyyyyyyyyyyyyyyyy"
```

### จัดการ KV ผ่าน CLI

```bash
# เพิ่ม/อัปเดตค่า
wrangler kv key put --namespace-id=<ID> "my-key" "my-value"

# อ่านค่า
wrangler kv key get --namespace-id=<ID> "my-key"

# ลบค่า
wrangler kv key delete --namespace-id=<ID> "my-key"

# ดูรายการ keys
wrangler kv key list --namespace-id=<ID>

# ดูรายการ Namespaces ทั้งหมด
wrangler kv namespace list
```

---

## 12. D1 Database

### สร้าง D1 Database

```bash
wrangler d1 create my-database
```

นำ ID ไปใส่ใน `wrangler.toml`:

```toml
[[d1_databases]]
binding = "DB"
database_name = "my-database"
database_id = "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx"
```

### รัน SQL

```bash
# รันคำสั่ง SQL ตรง ๆ
wrangler d1 execute my-database --command "SELECT * FROM users LIMIT 10"

# รันจากไฟล์ .sql
wrangler d1 execute my-database --file schema.sql

# Local mode (ทดสอบในเครื่อง)
wrangler d1 execute my-database --local --command "SELECT * FROM users"
```

### Migrations

```bash
# สร้าง migration ไฟล์
wrangler d1 migrations create my-database add-users-table

# Apply migrations
wrangler d1 migrations apply my-database

# ดูสถานะ migrations
wrangler d1 migrations list my-database
```

---

## 13. R2 Bucket

### สร้าง R2 Bucket

```bash
wrangler r2 bucket create my-bucket
```

### จัดการไฟล์ใน R2

```bash
# อัปโหลดไฟล์
wrangler r2 object put my-bucket/path/to/file.txt --file ./local-file.txt

# ดาวน์โหลดไฟล์
wrangler r2 object get my-bucket/path/to/file.txt --file ./downloaded.txt

# ลบไฟล์
wrangler r2 object delete my-bucket/path/to/file.txt

# ดูรายการ buckets
wrangler r2 bucket list
```

---

## 14. Cloudflare Pages

### สร้าง Pages Project

```bash
# สร้างใหม่
wrangler pages project create my-pages-app

# Deploy จาก directory
wrangler pages deploy ./dist --project-name=my-pages-app

# Deploy พร้อมระบุ branch
wrangler pages deploy ./dist --project-name=my-pages-app --branch=main
```

### Pages Functions (API Routes)

```bash
# Dev server สำหรับ Pages
wrangler pages dev ./dist

# ระบุ port
wrangler pages dev ./dist --port 3000
```

---

## 15. ดู Logs

### Real-time Logs (Tail)

```bash
# ดู logs แบบ real-time
wrangler tail

# ดู logs ของ Worker เฉพาะ
wrangler tail my-worker

# Filter เฉพาะ errors
wrangler tail --status error

# Filter ตาม IP
wrangler tail --ip-address 1.2.3.4

# Format แบบ pretty
wrangler tail --format pretty

# Format แบบ JSON
wrangler tail --format json
```

### ดู Deployment Logs

```bash
wrangler deployments list
wrangler deployments view <deployment-id>
```

---

## 16. Rollback และการจัดการ Deployments

### ดูประวัติ Deployments

```bash
wrangler deployments list
```

### Rollback ไป Deployment ก่อนหน้า

```bash
wrangler rollback
# หรือระบุ deployment ID
wrangler rollback <deployment-id>
```

### ดู Worker ที่ Deploy แล้วทั้งหมด

```bash
wrangler workers list
```

### ลบ Worker

```bash
wrangler delete my-worker
wrangler delete my-worker --env production
```

---

## 17. Pull โปรเจกต์จาก Cloudflare

### Download Worker Script

```bash
# Download source code ของ Worker ที่ deploy แล้ว
wrangler worker download my-worker

# ระบุ output directory
wrangler worker download my-worker --outdir ./downloaded-worker
```

### Fetch Config จาก Cloudflare Pages

```bash
# ดู pages projects ทั้งหมด
wrangler pages project list

# ดู deployments ของ project
wrangler pages deployment list --project-name=my-pages-app
```

### Clone การตั้งค่าด้วย wrangler.toml

ถ้าต้องการดึงการตั้งค่าจาก Worker ที่มีอยู่มาสร้างโปรเจกต์ใหม่:

```bash
# 1. สร้างโปรเจกต์ใหม่เปล่า ๆ
mkdir my-worker-clone && cd my-worker-clone
npm init -y
npm install --save-dev wrangler

# 2. Download source จาก deployed worker
wrangler worker download my-existing-worker --outdir .

# 3. ดู KV namespaces ที่มีอยู่
wrangler kv namespace list

# 4. ดู D1 databases ที่มีอยู่
wrangler d1 list

# 5. ดู R2 buckets ที่มีอยู่
wrangler r2 bucket list

# 6. สร้าง wrangler.toml ใหม่โดยอิงจากข้อมูลที่ได้
wrangler init --from-dash my-existing-worker
```

### `--from-dash` Flag (แนะนำ!)

คำสั่งนี้ดึงการตั้งค่าทั้งหมดจาก Dashboard มาสร้าง `wrangler.toml` ให้อัตโนมัติ:

```bash
wrangler init my-worker --from-dash my-existing-worker-name
```

> ✅ จะดึงมาให้: bindings (KV, D1, R2), routes, environment variables, compatibility settings

---

## 18. คำสั่งที่ใช้บ่อย (Cheat Sheet)

```bash
# ── Authentication ──────────────────────────────────
wrangler login                    # เข้าสู่ระบบ
wrangler logout                   # ออกจากระบบ
wrangler whoami                   # ดู account ที่ login อยู่

# ── Project ─────────────────────────────────────────
wrangler init <name>              # สร้างโปรเจกต์ใหม่
npm create cloudflare@latest      # สร้างด้วย wizard

# ── Development ─────────────────────────────────────
wrangler dev                      # เริ่ม local dev server
wrangler dev --remote             # dev บน Cloudflare infra
wrangler dev --env staging        # dev ด้วย staging config

# ── Deploy ──────────────────────────────────────────
wrangler deploy                   # deploy ขึ้น Cloudflare
wrangler deploy --env production  # deploy ไป production
wrangler deploy --dry-run         # ทดสอบโดยไม่ deploy จริง

# ── Workers ─────────────────────────────────────────
wrangler workers list             # ดูรายการ workers
wrangler worker download <name>   # download worker source
wrangler delete <name>            # ลบ worker
wrangler init --from-dash <name>  # pull config จาก dashboard

# ── Deployments ─────────────────────────────────────
wrangler deployments list         # ดูประวัติ deployments
wrangler rollback                 # rollback การ deploy
wrangler tail                     # ดู real-time logs

# ── Secrets & Vars ──────────────────────────────────
wrangler secret put <KEY>         # เพิ่ม secret
wrangler secret list              # ดูรายการ secrets
wrangler secret delete <KEY>      # ลบ secret

# ── KV ──────────────────────────────────────────────
wrangler kv namespace create <name>
wrangler kv namespace list
wrangler kv key put --namespace-id=<ID> <key> <value>
wrangler kv key get --namespace-id=<ID> <key>

# ── D1 ──────────────────────────────────────────────
wrangler d1 create <name>
wrangler d1 list
wrangler d1 execute <name> --command "<SQL>"

# ── R2 ──────────────────────────────────────────────
wrangler r2 bucket create <name>
wrangler r2 bucket list
wrangler r2 object put <bucket>/<key> --file <path>

# ── Pages ────────────────────────────────────────────
wrangler pages dev ./dist
wrangler pages deploy ./dist --project-name=<name>
wrangler pages project list
```

---

## Tips & Best Practices

- **ห้าม commit** ไฟล์ `.dev.vars` และ `wrangler.toml` ที่มี sensitive data ใส่ `.gitignore` เสมอ
- ใช้ **`--env`** เพื่อแยก staging และ production ออกจากกันอย่างชัดเจน
- ใช้ **`wrangler deploy --dry-run`** ก่อน deploy จริงทุกครั้งใน production
- ตั้ง **`CLOUDFLARE_API_TOKEN`** ใน CI/CD แทนการ login แบบ interactive
- ใช้ **`wrangler tail --format json`** เพื่อส่ง logs ไปยังระบบ monitoring
- ตรวจสอบ **`compatibility_date`** ใน `wrangler.toml` ให้เป็นเวอร์ชันที่รองรับ feature ที่ต้องการ

---

## อ้างอิง

- [Cloudflare Workers Docs](https://developers.cloudflare.com/workers/)
- [Wrangler CLI Reference](https://developers.cloudflare.com/workers/wrangler/commands/)
- [Cloudflare Pages Docs](https://developers.cloudflare.com/pages/)
- [D1 Database Docs](https://developers.cloudflare.com/d1/)
- [R2 Storage Docs](https://developers.cloudflare.com/r2/)
