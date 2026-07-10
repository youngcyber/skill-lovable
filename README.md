# Lovable Vibe Coder Skills

คอลเลกชัน Manus Skills สำหรับสร้าง prompt แบบ chain pattern เพื่อ build แอปด้วย [Lovable](https://lovable.dev) — รับ idea เป็นภาษาไทย ออก prompt พร้อม paste ทีละ chain

## โครงสร้าง

```
├── SKILL.md                      # Root skill — รับ idea แล้วแบ่ง 4 chain
└── skills/
    ├── saas-apps/SKILL.md        # SaaS app (auth, features, billing)
    ├── auth/SKILL.md             # ระบบ login, OAuth, protected routes
    ├── landing-pages/SKILL.md    # Landing page, hero, pricing, SEO
    ├── dashboards/SKILL.md       # Dashboard, KPI cards, charts
    ├── e-commerce/SKILL.md       # ร้านค้า, cart, Stripe checkout
    ├── forms/SKILL.md            # Form, validation, file upload
    ├── data/SKILL.md             # CRUD table, kanban, CSV import/export
    └── sub-chain/SKILL.md        # v2 — sub-chain pattern, 1 prompt = 1 component
```

## Chain Pattern (v1)

แบ่ง build เป็น 4 chain ตามลำดับนี้เสมอ:

| Chain | ชื่อ | เป้าหมาย |
|-------|------|----------|
| Chain 1 | โครงสร้าง | Layout shell, sidebar, routes เปล่า — ไม่มี logic |
| Chain 2 | ฟีเจอร์หลัก | UI ทุกฟีเจอร์ด้วย mock data — ไม่มี database |
| Chain 3 | จัดการข้อมูล | Supabase schema + auth + เชื่อม UI กับ database จริง |
| Chain 4 | เก็บรายละเอียด | Loading, error, empty states, mobile, polish |

## Sub-Chain Pattern (v2)

แตก prompt ย่อยกว่า v1 — **1 prompt = 1 component** เพื่อลด regression

```
Chain 1     — Layout shell
Chain 2.1   — Component แรก
Chain 2.2   — Component สอง
...
Chain 3.1   — Auth
Chain 3.2   — เชื่อม feature แรกกับ Supabase
...
Chain 4     — Polish
```

ใช้ `skills/sub-chain/SKILL.md` สำหรับแอปซับซ้อนหรือเมื่อ v1 ทำให้ Lovable regression

## Skills

| Skill | ใช้เมื่อ |
|-------|---------|
| [SaaS Apps](skills/saas-apps/SKILL.md) | แอปที่ต้องการ login, user data, dashboard |
| [Auth](skills/auth/SKILL.md) | ระบบ login, Google OAuth, protected routes, RBAC |
| [Landing Pages](skills/landing-pages/SKILL.md) | หน้าเว็บสาธารณะ hero, pricing, testimonials |
| [Dashboards](skills/dashboards/SKILL.md) | Analytics, KPI cards, Recharts, data tables |
| [E-Commerce](skills/e-commerce/SKILL.md) | Product catalog, cart, Stripe, admin |
| [Forms](skills/forms/SKILL.md) | Form ทุกประเภท, validation, file upload, email |
| [Data Management](skills/data/SKILL.md) | CRUD table, kanban, drag-to-reorder, CSV |
| [Sub-Chain (v2)](skills/sub-chain/SKILL.md) | ลด regression — 1 prompt ต่อ 1 component |

## License

MIT
