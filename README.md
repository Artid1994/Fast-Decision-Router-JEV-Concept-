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


