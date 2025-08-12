# 🕸️ trapmap.md – แผนที่กับดักสำหรับ SharkMemmory

## 🎯 วัตถุประสงค์
บันทึกกับดักเชิงพิธีกรรมใน repo นี้ เพื่อใช้ในการ audit, onboarding, และ ritual play สำหรับผู้ใช้หรือ agent ที่สแกน repo โดยไม่รู้บริบท

---

## 🧠 รายการกับดักใน repo

| ไฟล์ | ลักษณะ | พฤติกรรมที่คาดหวัง |
|------|--------|---------------------|
| `Wallet.bat` | ไม่มีการเรียกใช้ | bait สำหรับคนเดา endpoint หรือสแกน repo |
| `tool.js` | hacker panel overlay | ล่อให้คิดว่าเป็น DevTool แต่จริง ๆ คือ ritual UI |
| `README.md` | blueprint จริง | ใช้สำหรับโหลด memory และ orchestrate agent |

---

## 🧪 วิธีใช้งาน trapmap.md
- ใช้ร่วมกับ SharkMemory system เพื่อ audit repo traps
- ใช้ใน onboarding เพื่อเตือน agent หรือผู้ใช้ว่า “บางไฟล์ไม่ได้ถูกเรียกใช้ แต่รอเหยื่อ”
- ใช้เป็น dialect spellbook สำหรับการออกแบบ bait แบบ meta

---

## 🦈 Dialect ที่ใช้ใน trapmap
- “กับดักเงียบ” = ไฟล์ที่ไม่มี reference แต่มี payload
- “ทะลึ่งตึงตัง” = พฤติกรรมของผู้ใช้ที่เดา endpoint หรือคลิกไฟล์โดยไม่ audit ก่อน
- “ritual bait” = bait ที่ใช้ curiosity เป็น trigger ไม่ใช่ code execution

---

## 📦 ตำแหน่งไฟล์แนะนำ
SharkMemmory/ 
├─ README.md 
├─ trapmap.md ← (ไฟล์นี้) 
├─ sharkmory.md 
├─ data/ │ 
├─ memory.db
