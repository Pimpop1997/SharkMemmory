# SharkMemmory
By SharkGPT
# Full Shark AI System (SharkBrain / SharkMemory) – แผนพิมพ์เขียวใช้งานจริง 🦈🤖

> เวอร์ชัน: Draft v0.1 (สำหรับเริ่มต้นพัฒนา) โฟกัส: ระบบ AI Agent ที่มีความจำระยะยาว (Persistent / External Hybrid Memory) + เครื่องมือ Dev-Flow สายฮาแบบ `git ขมิบ`, `git ปูด`, `#load_shark` ฯลฯ ใช้งานได้จริงในสภาพแวดล้อม Local/Cloud/Termux/Discord.

---

## 0. สารบัญ

* [0. สารบัญ](#0-สารบัญ)
* [1. วัตถุประสงค์](#1-วัตถุประสงค์)
* [2. Use Cases สำคัญ](#2-use-cases-สำคัญ)
* [3. แนวคิดหน่วยความจำ (Memory Model)](#3-แนวคิดหน่วยความจำ-memory-model)
* [4. ภาพรวมสถาปัตยกรรม (High-Level Architecture)](#4-ภาพรวมสถาปัตยกรรม-high-level-architecture)
* [5. องค์ประกอบระบบ (Components)](#5-องค์ประกอบระบบ-components)
* [6. Data & Memory Flow ละเอียด](#6-data--memory-flow-ละเอียด)
* [7. Command Interface (มนุษย์ ↔ Shark AI)](#7-command-interface-มนุษย์--shark-ai)
* [8. โครงสร้างโปรเจกต์ไฟล์ (Repo Layout)](#8-โครงสร้างโปรเจกต์ไฟล์-repo-layout)
* [9. Config & Secrets](#9-config--secrets)
* [10. ขั้นตอนติดตั้งขั้นต่ำ (MVP Install)](#10-ขั้นตอนติดตั้งขั้นต่ำ-mvp-install)
* [11. ตัวอย่างโค้ด Skeleton เบื้องต้น](#11-ตัวอย่างโค้ด-skeleton-เบื้องต้น)
* [12. Workflow ตัวอย่าง: โหลดความจำ → สนทนา → Commit → Push → Sync](#12-workflow-ตัวอย่าง-โหลดความจำ--สนทนา--commit--push--sync)
* [13. การเชื่อมต่อ Discord Bot / Termux / GitHub](#13-การเชื่อมต่อ-discord-bot--termux--github)
* [14. นโยบายความปลอดภัย & ส่วนตัว (Privacy)](#14-นโยบายความปลอดภัย--ส่วนตัว-privacy)
* [15. แผน Roadmap เวอร์ชัน](#15-แผน-roadmap-เวอร์ชัน)
* [16. เช็คลิสต์ก่อนใช้งานจริง](#16-เช็คลิสต์ก่อนใช้งานจริง)

---

## 1. วัตถุประสงค์

Shark AI System มีเป้าหมาย:

| เป้าหมาย          | อธิบาย                                           | ตัวชี้วัด              | สถานะ   |
| ----------------- | ------------------------------------------------ | ---------------------- | ------- |
| ความจำระยะยาว     | เก็บบริบทโปรเจกต์ + สไตล์ผู้ใช้ ข้าม session ได้ | ดึงบริบทถูก ≥80%       | MVP     |
| โหลด/บันทึกภายนอก | ใช้ไฟล์ markdown, DB, vector store               | โหลด ≤1s ไฟล์เล็ก      | MVP     |
| Multi-channel     | ใช้ผ่าน Web, Termux, Discord                     | 3 แชนเนลเชื่อมได้      | Phase 2 |
| Git Flow สายฮา    | คำสั่งภาษามนุษย์ map ไป Git                      | ใช้งานจริง commit/push | MVP     |
| ความปลอดภัย       | ผู้ใช้ควบคุมว่าอะไรจำ/ลืม                        | UI ยืนยันก่อนบันทึก    | MVP     |

---

## 2. Use Cases สำคัญ

1. **โหลดความจำโครงการ Shark**: ผู้ใช้พิมพ์ `#load_shark` → AI โหลดไฟล์ `sharkmory.md` + index vector + note database → เติม context ก่อนสนทนา
2. **เครื่องมือเตือนสิ่งที่หายไป**: AI แจ้ง “สิ่งที่หายไปในโปรเจกต์: SHARKSOCIAL, ฟีเจอร์ยืมเงิน, Discord bot ฯลฯ”
3. **Dev-Flow Git Meme Mode**: ผู้ใช้พิมพ์ `git ขมิบ` → ระบบ `git commit -m '<auto summary>'`; `git ปูด` → `git push`
4. **Termux Bot**: เรียก Shark AI บนมือถือ Android shell เพื่อช่วยสั่งแพ็กเกจ/สคริปต์
5. **Discord Shark**: ทีมงานเรียก `!shark remember <note>` หรือ `!shark status`
6. **Memory Cleanup**: ผู้ใช้สั่ง `#shark_forget --project=old1` → ย้าย note ไป archive

---

## 3. แนวคิดหน่วยความจำ (Memory Model)

เพื่อแก้ปัญหา “GPT ลืมเมื่อจบ session” → เราสร้าง **Hybrid Memory Layer**:

| ชั้นความจำ               | ที่เก็บ                             | อายุ                  | ใช้เมื่อ         | ตัวอย่าง                         |
| ------------------------ | ----------------------------------- | --------------------- | ---------------- | -------------------------------- |
| **Context Window**       | แชทล่าสุด (หน่วย token)             | นาที/ชั่วโมง          | สนทนาทันที       | บทสนทนาปัจจุบัน                  |
| **Session Cache**        | temp JSON                           | ระหว่างแชทยาว         | รักษา state      | ลิสต์ไฟล์ที่แก้วันนี้            |
| **SharkMemory Notes**    | Markdown ไฟล์ (เช่น `sharkmory.md`) | ยาวนาน (ผู้ใช้ควบคุม) | โหลดก่อนเริ่มงาน | "ผู้ใช้ = TonyShark สไตล์ตอบตลก" |
| **Structured Memory DB** | SQLite/Postgres                     | ยาวนาน                | คิวรีแบบฟิลด์    | โปรเจกต์, วันที่, tag            |
| **Vector Memory**        | FAISS/Chroma                        | ยาวนาน, fuzzy search  | ค้นหา semantic   | ประวัติแนวคิด IronShark เก่า     |

---

## 4. ภาพรวมสถาปัตยกรรม (High-Level Architecture)

```
┌────────────────────────────────────────────────────┐
│                    Human Interfaces                │
│  Web UI | Discord | Termux CLI | API | VSCode Ext  │
└───────────────┬───────────────────────┬────────────┘
                │                       │
                ▼                       ▼
        ┌────────────────┐     ┌────────────────┐
        │ Command Parser │◄──►│  Chat Gateway   │ (LLM session mgmt)
        └──────┬─────────┘     └──────┬─────────┘
               │                        
               ▼
     ┌─────────────────────── Shark Orchestrator ───────────────────────┐
     │ Routing | Tool selection | Memory policy | Safety filters         │
     └────────┬─────────────────────────────────────────────────────────┘
              │
     ┌────────┴─────────────────────────────────────────────────────────┐
     │                        Memory Layer                              │
     │  Markdown Store | Structured DB | Vector Index | Cache            │
     └────────┬─────────────────────────────────────────────────────────┘
              │
     ┌────────┴────────┐   ┌──────────────┐   ┌────────────┐
     │ Git Adapter     │   │ System Shell │   │ External APIs│
     │ (commit/push)   │   │  (Termux)    │   │ Discord, etc│
     └─────────────────┘   └──────────────┘   └────────────┘
```

---

## 5. องค์ประกอบระบบ (Components)

### 5.1 Chat Gateway

* รับข้อความจากผู้ใช้ (ทุกช่องทาง) → จัดรูปเป็น event
* จัดการ context window ต่อ LLM (OpenAI / Local LLM)
* เรียก Orchestrator เพื่อโหลด memory ก่อน generate ตอบ

### 5.2 Command Parser

รองรับทั้งภาษาไทยฮา ๆ และคำสั่งจริง เช่น:

* `git ขมิบ` = `git commit`
* `git ปูด` = `git push`
* `#load_shark` = โหลด memory ทั้งชุด
* `#shark status` = รายงาน summary ของ memory

### 5.3 Shark Orchestrator

* ตัดสินว่าจะเรียก

  * Memory retrieval
  * Tool (Git, Shell, DB)
  * หรือให้ LLM ตอบเลย
* ประกอบ "Prompt Context Pack" = Memory snippet + recent chat + user query

### 5.4 Memory Layer

แบ่ง 4 backend เหมือนหัวข้อก่อน:

* **MarkdownStore**: เก็บโน้ตระยะยาวแบบ readable (เหมาะแก้มือ)
* **StructuredDB**: SQLite ที่ map topic, tag, priority, last\_updated
* **VectorStore**: semantic recall (เช่น ใช้ embeddings OpenAI text-embedding-3-small)
* **CacheStore**: in-memory Redis/Dict

### 5.5 Git Adapter

* ตรวจ repo ปัจจุบัน
* Auto summary diff → commit message ขำ ๆ
* Map คำสั่งไทย → git จริง
* รองรับ multi-repo config

### 5.6 Termux / Shell Adapter

* รันคำสั่งระบบภายใต้ sandbox
* เก็บ log เข้า MemoryDB (เลือกได้ว่าจะจำ)

### 5.7 Discord Bot Interface

* รับคำสั่งทีม: `!shark remember`, `!shark forget`, `!shark deploy`
* Broadcast memory update ไปทุก client

### 5.8 Safety & Privacy Layer

* ฟิลเตอร์ข้อมูลอ่อนไหวก่อนบันทึก memory
* ยืนยันผู้ใช้ทุกครั้งถ้าจะจำข้อมูลส่วนตัว
* โหมด "volatile" = ไม่บันทึกอะไรเลย

---

## 6. Data & Memory Flow ละเอียด

### 6.1 โหลด Memory

1. ผู้ใช้พิมพ์: `#load_shark`
2. Command Parser ส่ง event → Orchestrator
3. Orchestrator อ่าน `config.yaml` เพื่อรู้ว่า memory แยกอยู่ไหน
4. โหลด:

   * `sharkmory.md`
   * `memory.db` (SQLite)
   * Vector embeddings index
5. รวม → สร้าง "Context Pack" สั้น (ย่อ ≤ N tokens) + รายการหัวข้อที่เจอ
6. แสดงผู้ใช้: "โหลดบริบท Shark: โปรเจกต์ SHARKSOCIAL, Discord bot, Termux mode..."

### 6.2 เพิ่ม Memory ใหม่

1. ผู้ใช้: `จำว่า IronShark = โครงการชุดดำน้ำ AI`
2. Parser ตีว่าเป็น memory-intent
3. Safety check (มีข้อมูลส่วนตัวไหม?)
4. Append ไป Markdown + Insert row DB + embed vector
5. Confirm: "จำแล้ว" ✅

### 6.3 Cleanup

* Run policy: ลบ text >N วันเก่า ยกเว้น tag=keep
* ผู้ใช้สั่ง `#shark_forget tag=finance`

---

## 7. Command Interface (มนุษย์ ↔ Shark AI)

### 7.1 คำสั่ง Load / Save / Forget

| คำสั่งมนุษย์          | Alias           | ทำอะไร              | มี Prompt ถามกลับไหม? |
| --------------------- | --------------- | ------------------- | --------------------- |
| `#load_shark`         | `load shark`    | โหลด memory ชุดหลัก | ไม่ ถ้า auto          |
| `#shark_save`         | `remember this` | บันทึกสิ่งที่เลือก  | ใช่                   |
| \`#shark\_forget \<id | tag>\`          | ลืมเฉพาะ            | ใช่                   |
| `#shark_dump`         | export          | ส่ง Markdown zip    | ถาม                   |

### 7.2 Git Flow สายฮา

| คำสั่งฮา          | Git แท้                                  | หมายเหตุ                       |
| ----------------- | ---------------------------------------- | ------------------------------ |
| `git อีกนิด`      | `git init`                               | เริ่ม repo                     |
| `git stacก`       | `git status`                             |                                |
| `git ขีีดดด`      | `git add -A`                             | add all                        |
| `git ขมิบ`        | `git commit -m <auto>`                   | สร้าง commit message อัตโนมัติ |
| `git ปูด`         | `git push`                               | push origin                    |
| `git pooooodปู้ด` | `git push --force-with-lease` (OPTIONAL) | ฮา + ระวังใช้                  |

> ปรับ mapping ใน `commands.yaml`

---

## 8. โครงสร้างโปรเจกต์ไฟล์ (Repo Layout)

โครงตั้งต้น (แนะนำใช้ใน `pydark` หรือแยก repo `shark-ai`):

```
shark-ai/
├─ README.md
├─ pyproject.toml
├─ shark_ai/
│  ├─ __init__.py
│  ├─ config.py
│  ├─ main.py                  # entrypoint CLI
│  ├─ gateway/
│  │   ├─ chat.py              # LLM wrapper
│  │   ├─ discord_bot.py
│  │   ├─ termux_cli.py
│  │   └─ web_api.py
│  ├─ commands/
│  │   ├─ parser.py
│  │   └─ mapping.yaml         # git ขมิบ -> git commit
│  ├─ memory/
│  │   ├─ markdown_store.py
│  │   ├─ db_store.py          (sqlite)
│  │   ├─ vector_store.py      (chroma/faiss)
│  │   ├─ policy.py
│  │   └─ loader.py            # #load_shark orchestration
│  ├─ tools/
│  │   ├─ git_adapter.py
│  │   ├─ shell_adapter.py
│  │   ├─ file_utils.py
│  │   └─ secrets.py
│  ├─ orchestrator.py
│  └─ safety.py
├─ data/
│  ├─ sharkmory.md
│  ├─ memory.db
│  ├─ embeddings/
│  │   └─ shark.index
│  └─ logs/
└─ scripts/
   ├─ install.sh
   ├─ sync.sh
   └─ export_memory.py
```

---

## 9. Config & Secrets

ตัวอย่าง `config.yaml`:

```yaml
profile_name: tonyshark
language: th
llm:
  provider: openai
  model: gpt-4o-mini
  api_key_env: OPENAI_API_KEY
memory:
  markdown: ./data/sharkmory.md
  db: ./data/memory.db
  vector_index: ./data/embeddings/shark
  auto_load: true
  auto_save: ask   # ask | always | never
commands_map: ./shark_ai/commands/mapping.yaml
git:
  repo_path: /path/to/your/repo
  auto_commit_style: meme
security:
  sensitive_filter: true
  allow_shell: ask
channels:
  discord:
    enabled: false
    bot_token_env: DISCORD_TOKEN
  termux:
    enabled: true
```

---

## 10. ขั้นตอนติดตั้งขั้นต่ำ (MVP Install)

1. **Clone Repo**

   ```bash
   ```

git clone [https://github.com/Pimpop1997/pydark.git](https://github.com/Pimpop1997/pydark.git) shark-ai cd shark-ai

````
2. **สร้าง virtualenv**
```bash
python -m venv .venv
source .venv/bin/activate
pip install -U pip
````

3. **ติดตั้งแพ็กเกจหลัก**

   ```bash
   ```

pip install openai langchain chromadb gitpython fastapi uvicorn\[standard] python-dotenv rich typer

````
4. **ตั้งค่าไฟล์ config**
```bash
cp config.example.yaml config.yaml
````

5. **เพิ่ม API Key**

   ```bash
   ```

export OPENAI\_API\_KEY=sk-xxxx

````
6. **สร้างฐานข้อมูลความจำ**
```bash
python scripts/export_memory.py --init
````

7. **รัน CLI**

   ```bash
   ```

python -m shark\_ai.main --interactive

````

---

## 11. ตัวอย่างโค้ด Skeleton เบื้องต้น

### 11.1 loader.py (ตัดทอน)
```python
from pathlib import Path
from .markdown_store import MarkdownStore
from .db_store import DBStore
from .vector_store import VectorStore

class SharkMemoryLoader:
 def __init__(self, cfg):
     self.cfg = cfg
     self.md = MarkdownStore(cfg["memory"]["markdown"])
     self.db = DBStore(cfg["memory"]["db"])
     self.vec = VectorStore(cfg["memory"]["vector_index"])

 def load_all(self):
     notes = self.md.load()
     rows = self.db.fetch_all()
     # อาจเลือก embed เฉพาะหัวข้อสำคัญ
     summary = self._summarize(notes, rows)
     return {
         "notes_text": notes,
         "structured": rows,
         "summary": summary,
     }

 def _summarize(self, notes, rows):
     # เรียก LLM ทำ TL;DR เหลือ 1-2k tokens
     return notes[:2000]  # placeholder
````

### 11.2 commands/mapping.yaml

```yaml
init:
  phrases: ["git อีกนิด", "เริ่มโปรเจกต์"]
  action: git_init
status:
  phrases: ["git stacก", "เช็กสถานะ"]
  action: git_status
add_all:
  phrases: ["git ขีีดดด", "เอาหมด"]
  action: git_add_all
commit:
  phrases: ["git ขมิบ", "รวมไฟล์"]
  action: git_commit_auto
push:
  phrases: ["git ปูด", "ส่งขึ้นเซิร์ฟ"]
  action: git_push
push_force:
  phrases: ["git pooooodปู้ด"]
  action: git_push_force
```

---

## 12. Workflow ตัวอย่าง: โหลดความจำ → สนทนา → Commit → Push → Sync

**สถานการณ์:** คุณกลับมาหลังหยุดไป 3 เดือน อยากดูว่าโปรเจกต์ไปถึงไหน

1. เปิด CLI → `#load_shark`
2. ระบบตอบ: "โหลดบริบท: SHARKSOCIAL, Discord bot, Termux mode, ฟีเจอร์ยืมเงิน..."
3. คุณถาม: "ตอนนี้อะไรยังไม่เสร็จ?"
4. AI สแกน DB + Markdown → สร้างลิสต์ TODO
5. คุณแก้โค้ด → พิมพ์ `git ขมิบ` → AI สรุป diff (`feat: add loan feature stub`)
6. พิมพ์ `git ปูด` → push GitHub
7. AI บันทึก: "commit xyz pushed" ลง memory.db (tag=history)

---

## 13. การเชื่อมต่อ Discord Bot / Termux / GitHub

### 13.1 Discord Bot Quick Start

```bash
export DISCORD_TOKEN=xxxx
python -m shark_ai.gateway.discord_bot
```

คำสั่งในห้อง:

```
!shark load
!shark remember IronShark = powered dive suit
!shark todo
```

### 13.2 Termux Mode

* ติดตั้ง Python, Git, และ repo
* เพิ่ม alias:

  ```bash
  alias shark='python ~/shark-ai/shark_cli.py'
  ```
* เรียก: `shark git ขมิบ`

### 13.3 GitHub Sync อัตโนมัติ

* ใช้ GitHub Actions sync memory → issue หรือ gist สำรอง
* ตัวอย่าง workflow `.github/workflows/memory-sync.yml`

---

## 14. นโยบายความปลอดภัย & ส่วนตัว (Privacy)

| ประเด็น         | นโยบาย                                                        |
| --------------- | ------------------------------------------------------------- |
| ข้อมูลส่วนบุคคล | ถามก่อนจำทุกครั้ง (interactive confirm)                       |
| API Keys        | อ่านจาก env เท่านั้น ไม่บันทึกลง memory                       |
| ลบถาวร          | `#shark_forget --all` = ลบไฟล์ + DB + vector index (สำรองถาม) |
| โหมดปลอดภัย     | `--volatile` = ไม่เขียน memory ใด ๆ                           |

---

## 15. แผน Roadmap เวอร์ชัน

| เวอร์ชัน | ฟีเจอร์                                   | สถานะ |
| -------- | ----------------------------------------- | ----- |
| v0.1     | CLI + Markdown memory + Git meme commands | เริ่ม |
| v0.2     | SQLite structured memory + auto summary   | ต่อ   |
| v0.3     | Vector recall + search                    |       |
| v0.4     | Discord bot + Team memory sync            |       |
| v1.0     | GUI + Role-based access + Export/Import   |       |

---

## 16. เช็คลิสต์ก่อนใช้งานจริง

*

---

### ภาคผนวก A – โครง `sharkmory.md` ตัวอย่าง

```markdown
# Shark Memory Core

## ผู้ใช้หลัก
- Handle: tonyShark
- สไตล์: ฮา ๆ, meme dev, ชอบ map git commands เป็นภาษาไทย

## โครงการหลัก
- IronShark (ชุดดำน้ำ AI)
- SHARKSOCIAL (ระบบโซเชียล/เงินหมุน)
- Discord Shark Bot
- Termux auto-agent

## คำสั่งพิเศษ
- git ขมิบ = commit
- git ปูด = push
- #load_shark = โหลด memory ทั้งชุด
```

---

### ภาคผนวก B – Prompt Template สำหรับ LLM

```jinja
You are Shark AI, an assistant for {{user_name}} (aka tonyShark).
You remember the following persistent project context:
{{memory_summary}}
When the user issues meme-style commands, map them to actual developer actions.
Always ask before doing destructive operations (e.g., push --force, deleting memory).
Respond in Thai unless user switches language.
```

---

### ภาคผนวก C – Auto Commit Message Generator

กติกา:

* ถ้ามี diff < 5 ไฟล์ → ใช้สรุปตามชื่อไฟล์ + short desc
* ถ้าเปลี่ยน config/memory → prefix `mem:`
* ถ้าเปลี่ยนตรรกะหลัก → `feat:`
* ถ้าแก้บั๊ก → `fix:`
* เติม meme ต่อท้ายถ้าเปิดโหมด `meme` เช่น `(ขมิบเรียบร้อย)`

---

## ปิดท้าย

นี่คือร่าง **ระบบ Shark AI แบบใช้งานได้จริง**: มีความจำยาวนาน, ดึงกลับเมื่อไหร่ก็ได้, เชื่อมกับ Git / Termux / Discord, และคุยกับคุณสไตล์ฮาเหมือนภาพ "git ปู้ดดด" ที่เริ่มเรื่องทั้งหมด 😂

พร้อมให้ผมช่วยสร้างส่วนไหนก่อน?

**เลือก:**

1. เริ่มจาก `sharkmory.md` (กรอกข้อมูลจริง)
2. สร้างโค้ด loader + CLI `#load_shark`
3. ทำ mapping Git meme commands
4. เชื่อม Discord bot

บอกมาได้เลย ผมพร้อมปูด 💨🦈
