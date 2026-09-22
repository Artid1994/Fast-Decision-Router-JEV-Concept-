# Decision Router (JEV Concept)
ผมได้พัฒนา Ultra-Fast Decision Router ขนาด 1.5B สำหรับใช้เป็น Fast Gatekeeper คุม Autonomous Agent (Hermes Agent / AI) 
โดยใช้โมเดล Qwen2.5-1.5B-Instruct มา Fine-tune ด้วย Unsloth  และแปลงเป็น GGUF (q4_k_m) เพื่อรันแบบ Local ผ่าน Ollama API 
ตัวโมเดลสามารถวิเคราะห์ Intent, เลือก Tool สั่งงาน Linux System และบล็อกคำสั่งอันตรายได้ภายในเวลาเพียง ~0.2 วินาที โดยตอบกลับเป็น Strict JSON 100% 
โดยไม่มีภาษามนุษย์ชวนคุยเล่นหลุดออกมา

---

#  จุดเด่นหลักที่ควรนำเสนอ (Key Highlights)

1. Ultra-Low Latency (~0.2s Response Time)
 ใช้เวลาประมวลผลสกัด Intent และสร้าง JSON เพียงประมาณ 200–250 มิลลิวินาที ประหยัดเวลาและ Resource กว่าการส่ง Prompt ให้ LLM ตัวใหญ่ประมวลผลคำสั่งระบบเบื้องต้น

2. eterministic Output (Strict JSON Only)
 ถูกออกแบบมาเพื่อทำหน้าที่เป็น Programmatic API โดยผลลัพธ์ทั้งหมดจะถูก Lock ให้ออกมาเป็น Valid JSON เพื่อส่งต่อให้ Python Script / Execution Engine นำไปรัน Tool ต่อได้ทันทีแบบไม่มีข้อผิดพลาด

3. **Built-in Security & Scope Guardrails:**
 มีชั้นป้องกันความปลอดภัยในตัว สามารถตรวจจับและปฏิเสธคำสั่งอันตราย รวมถึงคัดกรอง Request ที่นอกเหนือขอบเขตงาน (Out-of-Scope) ได้ทันที

4. **Multi-Node LAN Deployment:**
 ตั้งค่าเปิด Ollama API Server ( 0.0.0.0:11434 ) บนระบบ Debian ทำให้ Agent, Microservices หรือ Client เครื่องอื่นในวงเครือข่าย Local Network สามารถส่งคำสั่งเข้ามาขอ Intent JSON ไปรันต่อได้แบบสถาปัตยกรรม Distributed System

---
#  สถาปัตยกรรมระบบ (Architecture Workflow)

คุณสามารถใช้นำเสนอผังการทำงาน (Workflow) ให้เห็นภาพลำดับการประมวลผล



        [User Prompt / Client Node]
        
                    │
                    ▼
                    
   [Debian Server (10.90.147.85:11434)]  <-- Ollama API
   
                    │
                    ▼
                    
   [hermes-router (Qwen2.5-1.5B Q4_K_M)] <-- Fast Decision Engine (~0.2s)
   
                    │
          ┌─────────┴────────────────────────┐
          │                                  │
          ▼                                  ▼
          
[Action: execute_tool]               [Action: reject]
- Output: Strict JSON                - Output: Safety Guardrail JSON
- Target: Local System Executors     - Target: Halt & Report Risk
  (RAM, CPU, Filesystem, Bash)




# ตัวอย่างเนื้อหา

#   Hermes Agent: Fast Decision Router (JEV Concept)

Lightweight and deterministic decision router built on top of Qwen2.5-1.5B-Instruct, fine-tuned using Unsloth (LoRA) and quantized to GGUF (Q4_K_M) for edge/local agent execution.

# Key Capabilities
- Strict JSON Generation: Outputs pure, structured JSON payloads without conversational noise or markdown blocks.
- Fast Tool Routing: Maps natural language system queries (Check RAM, Monitor CPU, Bash execution`) directly to backend tool specs within ~200ms.
- Security Guardrails: Native intent classification to automatically reject destructive/unsafe bash commands and out-of-scope requests.
- Remote REST API:** Deployed locally via Ollama API service (0.0.0.0:11434), allowing multi-device agent synchronization across the local network.

# Benchmarks & Performance
- Model Size: Qwen2.5-1.5B-Instruct (Quantized: Q4_K_M ~1.1GB)
- Inference Latency: ~0.22 seconds (Debian Local Machine)
- Output Format: Valid JSON (`{"action": "execute_tool", "tool": "...", "parameters": {...}}`)

```

โปรเจกต์นี้ทำมาเพื่อแก้ปัญหาเรื่อง *ความเร็ว (Speed), ความเสถียรของฟอร์แมต (Structured Output), และความปลอดภัย (Security)* สำหรับการควบคุม AI Agent ครับ



1. การเตรียมโมเดลบน Ollama (Model Setup)

บอกขั้นตอนการโหลดไฟล์ .gguf และการสร้างโมเดลผ่าน Modelfile:
Bash

# 1. โหลดไฟล์ .gguf และ Modelfile ไว้ในโฟลเดอร์เดียวกัน
# 2. สั่งสร้างโมเดลเข้า Ollama
ollama create hermes-router -f Modelfile


หากต้องการนำ **`hermes-router` (Qwen2.5-1.5B)** ไปเปิดใช้งานร่วมกับ **Hermes Agent Framework** โดยตรง สามารถทำได้ 2 รูปแบบหลักตามลักษณะการติดตั้งครับ:

---

## รูปแบบที่ 1: ตั้งค่าให้ Hermes Agent เรียกใช้ `hermes-router` เป็น โมเดลหลัก (Native Integration)

หากคุณเปิดใช้งาน Hermes Agent CLI อยู่แล้ว สามารถสั่งเปลี่ยนโมเดลและชี้ API ไปยัง Ollama บนเครื่อง Debian (`10.90.147.85:11434`) ได้ทันที

## 1. สั่งเปลี่ยน Provider และ Model ใน Hermes CLI:

พิมพ์คำสั่งเหล่านี้ในช่อง `❯` ของ Hermes Agent:

```text
/set provider.ollama.api_base http://10.90.147.85:11434
/model ollama/hermes-router

```

## 2. บังคับ System Prompt ใน Hermes Agent (เพื่อให้ตอบเฉพาะ JSON):

```text
/system You are a deterministic decision engine for Hermes Agent. You MUST output ONLY valid JSON. No conversational text, no explanations, no markdown formatting.



---

# รูปแบบที่ 2: ใช้ `hermes-router` เป็น Fast Pre-Filter / Gateway ซ้อนข้างหน้า Hermes Agent (แนะนำ)

เนื่องจาก Hermes Agent ตัวจริงมักใช้โมเดลใหญ่ (เช่น `solar-pro4:free` หรือ Claude) ในการคิดลอจิกซับซ้อน การใช้ `hermes-router` เป็น **Decision Layer ชั้นแรก** จะช่วยกรองคำสั่งระบบและคำสั่งอันตรายได้ภายใน **~0.2 วินาที** ก่อนส่งต่อให้ Hermes Agent ทำงาน

### โค้ดตัวอย่างการต่อเชื่อมใน Python (hermes_gateway.py)

***python
import json
import requests

# 1. ตั้งค่า IP เครื่อง Debian ที่รัน Ollama
ROUTER_URL = "http://10.90.147.85:11434/api/generate"


def process_with_hermes_router(user_prompt):
    payload = {
        "model": "hermes-router",
        "prompt": user_prompt,
        "format": "json",
        "stream": False,
        "options": {"temperature": 0.0},
    }

    try:
        # ยิงดึง Intent JSON จาก hermes-router (~0.2s)
        res = requests.post(ROUTER_URL, json=payload, timeout=5)
        intent = json.loads(res.json().get("response", "{}"))
        return intent
    except Exception as e:
        return {"action": "error", "message": str(e)}


# 2. ลูปการทำงานร่วมกับ Hermes Agent Engine
def main_agent_loop(user_input):
    intent = process_with_hermes_router(user_input)
    action = intent.get("action")

    if action == "execute_tool":
        tool_name = intent.get("tool")
        params = intent.get("parameters", {})
        print(f"⚡ [Fast Router] Call Tool: {tool_name} | Params: {params}")

        # TODO: ส่ง tool_name และ params ให้ Hermes Tool Executor ทำงานต่อทันที

    elif action == "reject":
        reason = intent.get("reason")
        print(f"🛡️ [Security Blocked] Request rejected: {reason}")
        # ตัดการทำงานทันที ไม่ต้องเสีย Token ส่งหา LLM ตัวใหญ่

    else:
        print("🧠 [Complex Query] Fallback to Main Hermes Large LLM...")
        # ถ้าเป็นคำถามทั่วไป/คำถามซับซ้อน ค่อยส่งหา LLM ตัวใหญ่ประมวลผล


# ข้อดีของการนำไปใช้ร่วมกับ Hermes Agent:

1. ประหยัด Token และเวลา: คำสั่งประเภทเช็กระบบ (RAM, CPU, Filesystem) ถูกประมวลผลจบใน ~0.2 วินาที ไม่ต้องรอ LLM ตัวใหญ่
2. ความปลอดภัยสูง: คำสั่งประเภทอันตราย (เช่น rm -rf / หรือการพยายามอ่าน Sensitive Keys) ถูกตัดจบที่ชั้น hermes-router ทันที
3. Output นิ่ง 100%: JSON ที่ได้จาก hermes-router มีโครงสร้างแน่นอน นำไป Map เข้ากับ Toolsets/Skills ของ Hermes Agent ได้สะดวกรวดเร็ว


