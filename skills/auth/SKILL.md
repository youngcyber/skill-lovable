---
name: lovable-auth-planner
description: "รับความต้องการ auth แล้วสร้าง prompt แบบ chain pattern 4 ชั้นสำหรับ Lovable: โครงสร้าง login UI → ฟีเจอร์ auth → เชื่อม Supabase → เก็บรายละเอียด รองรับภาษาไทย"
---

# Lovable Auth Planner

รับความต้องการ auth จาก user แล้วสร้าง chain prompt พร้อม paste เข้า Lovable ทีละ chain ตอบกลับทุกอย่างเป็นภาษาไทย รวมถึง prompt ที่สร้างสำหรับ Lovable

## When to Use

- User ต้องการเพิ่มระบบ login/signup ให้แอป Lovable
- User ถามเรื่อง protected routes หรือการ redirect เมื่อยังไม่ login
- User ต้องการ Google OAuth, magic link, หรือ email/password
- User ต้องการสิทธิ์ต่างกันสำหรับ user กับ admin
- User ใช้ภาษาไทยในการอธิบายความต้องการ

## Input Schema

```yaml
app_name: string           # ชื่อแอป
auth_methods: list         # [email, google, magic-link]
post_login_route: string   # หน้าที่ไปหลัง login เช่น /app/dashboard
user_roles: list           # [user, admin] — default: [user]
profile_fields: list       # field เพิ่มเติม เช่น [full_name, avatar_url, plan]
```

## Workflow

### Step 1: ทำความเข้าใจความต้องการ

ถ้าไม่รู้ `post_login_route` ให้ถามก่อน เพราะทุก chain อ้างอิง route นี้

### Step 2: วาง 4 Chain

```
Chain 1 → โครงสร้าง: สร้างหน้า login/signup UI เปล่าๆ ก่อน
Chain 2 → ฟีเจอร์หลัก: form validation, OAuth button, route logic (ยังไม่ connect จริง)
Chain 3 → จัดการข้อมูล: เชื่อม Supabase Auth จริง, profiles table, RLS, route guard
Chain 4 → เก็บรายละเอียด: error states, loading, redirect logic, session management
```

### Step 3: เขียน Prompt แต่ละ Chain

---

**Chain 1 — โครงสร้าง**

```
สร้างหน้า auth UI สำหรับ [ชื่อแอป]

สร้างหน้าต่อไปนี้ (UI เท่านั้น ยังไม่ต้อง connect backend):

หน้า /login:
- Email input + Password input
- ปุ่ม "เข้าสู่ระบบ" (เต็ม width, indigo-600)
- ลิงก์ "ลืมรหัสผ่าน?" → /forgot-password
- [ถ้ามี Google] ปุ่ม "Continue with Google" (white bg, gray border, Google G icon)
- ลิงก์ "ยังไม่มีบัญชี? สมัครสมาชิก" → /signup

หน้า /signup:
- Full name + Email + Password inputs
- ปุ่ม "สมัครสมาชิก"
- ลิงก์ "มีบัญชีแล้ว? เข้าสู่ระบบ" → /login

หน้า /forgot-password:
- Email input + ปุ่ม "ส่งลิงก์รีเซ็ต"
- Success state: ข้อความ "ตรวจสอบ email ของคุณ"

Design:
- Centered card, max-w-[400px], white bg, rounded-2xl, shadow-md
- โลโก้ "[ชื่อแอป]" เหนือ card
- Inline error placeholder ใต้ field (แดง, text-sm) — placeholder ว่างไว้ก่อน

เสร็จเมื่อ:
- [ ] ทั้ง 3 หน้า render ได้ที่ route ที่กำหนด
- [ ] ปุ่มและ link navigate ถูกต้อง
- [ ] Design responsive บน mobile 375px
- [ ] ยังไม่มี Supabase call ใดๆ
```

---

**Chain 2 — ฟีเจอร์หลัก**

```
เพิ่ม form validation และ UI logic ให้หน้า auth ของ [ชื่อแอป]

ติดตั้ง: react-hook-form + zod + @hookform/resolvers

Validation schema (src/lib/schemas/auth.ts):
- email: z.string().email("รูปแบบ email ไม่ถูกต้อง")
- password: z.string().min(8, "รหัสผ่านต้องมีอย่างน้อย 8 ตัวอักษร")
- full_name: z.string().min(2, "กรุณากรอกชื่อ")

กฎ validation:
- แสดง error เฉพาะเมื่อกด submit ครั้งแรก
- หลังจากนั้น error หายทันทีเมื่อแก้ไข field
- Error แสดง inline ใต้ field เป็น text-red-500 text-sm
- Input ที่ error: border-red-500

Submit button states:
- Default: ปุ่มปกติ
- Loading: disabled + Loader2 spinner ข้างใน (ยัง mock อยู่ — console.log แทน)
- ปุ่มไม่ทำอะไรถ้า form invalid

[ถ้ามี Google OAuth:]
- ปุ่ม "Continue with Google" มี hover:bg-gray-50 transition

เสร็จเมื่อ:
- [ ] Submit โดยไม่กรอก email แสดง error "รูปแบบ email ไม่ถูกต้อง"
- [ ] Password < 8 ตัวแสดง error
- [ ] แก้ไข field ที่ error แล้ว error หายทันที
- [ ] ปุ่มแสดง spinner เมื่อ click (console.log ผลลัพธ์)
- [ ] ยังไม่มี Supabase call จริง
```

---

**Chain 3 — จัดการข้อมูล**

```
เชื่อม [ชื่อแอป] กับ Supabase Auth

1. สร้าง profiles table:
profiles (
  id uuid references auth.users primary key,
  full_name text,
  avatar_url text,
  role text not null default 'user' check (role in ([roles])),
  onboarded boolean default false,
  created_at timestamptz default now()
)
RLS:
- SELECT: id = auth.uid()
- UPDATE: id = auth.uid()
Trigger: on auth.users INSERT → INSERT into profiles(id, full_name, avatar_url)

2. สร้าง useAuth() hook ที่ src/hooks/useAuth.ts:
returns: { user, session, loading, profile, signIn, signUp, signOut }

3. เชื่อม form กับ Supabase:
- /signup: supabase.auth.signUp({ email, password, options: { data: { full_name } } })
  → redirect [post_login_route]
- /login: supabase.auth.signInWithPassword({ email, password })
  → redirect [post_login_route]
- /forgot-password: supabase.auth.resetPasswordForEmail(email)
[ถ้ามี Google:] supabase.auth.signInWithOAuth({ provider: 'google' })

4. Route guard (src/components/ProtectedRoute.tsx):
- loading → spinner กลางหน้า
- ไม่มี session → redirect /login?next=[current path]
- มี session → render children
- ใช้ครอบ /app/* ทุก route ใน React Router

เสร็จเมื่อ:
- [ ] Signup สร้าง user ใน auth.users + row ใน profiles
- [ ] Login redirect ไป [post_login_route]
- [ ] Login ผิดรหัสแสดง error inline (ไม่ใช่ toast)
- [ ] /app/dashboard เมื่อไม่ได้ login → /login?next=/app/dashboard
- [ ] useAuth() import ได้จาก src/hooks/useAuth
```

---

**Chain 4 — เก็บรายละเอียด**

```
เพิ่ม error handling และ UX details ให้ระบบ auth ของ [ชื่อแอป]

Error messages:
- Login ผิดรหัสผ่าน: "Email หรือรหัสผ่านไม่ถูกต้อง" (inline ใต้ form ไม่ใช่ toast)
- Email ซ้ำ (signup): "Email นี้มีบัญชีอยู่แล้ว กรุณาเข้าสู่ระบบ"
- Network error: "เกิดข้อผิดพลาด กรุณาลองใหม่อีกครั้ง"
- ทุก error: AlertCircle icon + ข้อความ บน indigo-50 bg rounded-lg ใต้ form

Loading:
- หน้า auth: ปุ่ม disabled + spinner ระหว่าง Supabase call
- Route guard: Loader2 animate-spin กลางหน้า (ไม่ flash หน้า login ก่อน)

After-login redirect:
- อ่าน ?next= param หลัง login → redirect ไปหน้านั้น
- ถ้าไม่มี ?next= → redirect [post_login_route]

Session persistence:
- Session อยู่หลัง refresh หน้า (Supabase จัดการอัตโนมัติ)
- ถ้าเปิดแอปแล้วมี session อยู่แล้ว → ไม่ต้อง login ใหม่

Sign-out:
- ปุ่ม sign-out ใน sidebar เรียก supabase.auth.signOut()
- Redirect ไป /login หลัง sign-out

เสร็จเมื่อ:
- [ ] Login ผิดรหัสแสดง error inline ถูกต้อง
- [ ] ไม่มี flash ของหน้า login ก่อน redirect guard
- [ ] Refresh หน้าเมื่อ login อยู่ → ยังอยู่ที่เดิม ไม่ต้อง login ใหม่
- [ ] Sign-out ทำงานและ redirect /login
- [ ] ?next= redirect ทำงานถูกต้อง
```

## Output Schema

```yaml
plan_title: string          # "[ชื่อแอป] — Auth Build Plan"
auth_summary: string        # auth methods, roles, post-login route
chains:
  - number: integer
    name: string
    goal: string
    prompt: string
    done_when: list
total_chains: 4
```

## Output Format

```
# [ชื่อแอป] — Auth Build Plan
> auth ด้วย [methods], roles: [roles], หลัง login ไปที่ [route]

---

## Chain 1 — โครงสร้าง
**เป้าหมาย:** หน้า login/signup UI ก่อน connect backend

[PROMPT]

**เสร็จเมื่อ:**
- [ ] ...

---
(ต่อ Chain 2, 3, 4)
---

**รวม 4 Chains**
Paste Chain 1 เข้า Lovable ✓ ทุก checkbox แล้วไป Chain 2
```

## Error Handling

- **ไม่รู้ post_login_route** — ถามก่อน route นี้ใช้ใน chain 3 ทุกที่
- **ต้องการหลาย auth method** — แต่ละ method เป็น sub-step ใน Chain 3 ไม่แยก chain
- **ต้องการ RBAC** — เพิ่ม AdminRoute component เป็น sub-step ใน Chain 3

## Examples

### ตัวอย่าง: Login ด้วย Email + Google

**Input:** app: Taskly, auth: [email, google], post_login: /app/dashboard, roles: [user, admin]

- Chain 1: หน้า /login (ปุ่ม email + ปุ่ม Google), /signup, /forgot-password
- Chain 2: Validation ทุก field, ปุ่ม Google UI + hover state
- Chain 3: Supabase Auth email + OAuth Google, profiles table + trigger, ProtectedRoute + AdminRoute
- Chain 4: Error messages ภาษาไทย, redirect ?next=, session persistence
