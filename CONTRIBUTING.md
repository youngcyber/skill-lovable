# Contributing

ยินดีรับการมีส่วนร่วมครับ คู่มือนี้อธิบายวิธีเพิ่ม skill ใหม่หรือปรับปรุง skill ที่มีอยู่

## สิ่งที่ต้องการ

- **Skill ใหม่** — chain prompt pattern สำหรับ use case ที่ยังไม่มีใน `skills/`
- **ปรับปรุง Skill ที่มีอยู่** — เพิ่ม example, แก้ prompt template, หรืออัปเดต tech stack
- **Bug fix** — แก้ prompt ที่ให้ผลลัพธ์ผิดใน Lovable

## สิ่งที่ไม่ต้องการ

- Prompt ที่ใช้ได้เฉพาะ project ของตัวเองเท่านั้น
- Skill ที่ไม่ได้ทดสอบกับ Lovable จริง
- ไฟล์ที่มี TODO placeholder ค้างอยู่

## วิธีเพิ่ม Skill ใหม่

1. Fork repository นี้
2. สร้าง directory ใหม่ใน `skills/` เช่น `skills/blog/`
3. สร้าง `skills/blog/SKILL.md` ตาม format ด้านล่าง
4. อัปเดตตาราง Skills ใน `README.md`
5. เปิด Pull Request ชื่อเช่น `Add: blog platform skill`

## Format ของ SKILL.md

ทุก SKILL.md ต้องมี YAML frontmatter และ sections ต่อไปนี้:

```markdown
---
name: lovable-[domain]-planner
description: "รับ idea [ประเภท] แล้วสร้าง prompt แบบ chain pattern 4 ชั้นสำหรับ Lovable: โครงสร้าง → ฟีเจอร์หลัก → จัดการข้อมูล → เก็บรายละเอียด รองรับภาษาไทย"
---

## When to Use
## Input Schema
## Workflow
## Output Schema
## Output Format
## Error Handling
## Examples
```

**กฎที่ต้องทำตาม:**
- `description` ต้องเป็น single-line string ใน `" "`
- prompt template ในส่วน Workflow ต้องเป็นภาษาไทย
- ทุก chain ต้องมี checklist "เสร็จเมื่อ" ที่ตรวจสอบได้จริง
- ต้องมีอย่างน้อย 1 ตัวอย่างใน Examples section

## Chain Pattern ที่ใช้

skill ทุกตัวต้องเรียงตาม 4 chain นี้:

| Chain | ชื่อ | สิ่งที่ห้ามทำ |
|-------|------|--------------|
| Chain 1 | โครงสร้าง | ห้าม connect database หรือ add logic |
| Chain 2 | ฟีเจอร์หลัก | ห้าม call Supabase — ใช้ mock data เท่านั้น |
| Chain 3 | จัดการข้อมูล | ทำทีละ feature ห้าม connect ทุกอย่างพร้อมกัน |
| Chain 4 | เก็บรายละเอียด | ต้องทำหลัง chain 3 เสร็จเท่านั้น |

## Commit Style

```
Add: blog platform skill
Update: saas-apps chain 3 Supabase schema
Fix: auth skill redirect logic after login
```
