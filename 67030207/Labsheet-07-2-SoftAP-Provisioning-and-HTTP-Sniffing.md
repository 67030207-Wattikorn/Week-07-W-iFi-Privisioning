## กิจกรรมถอดรหัสซอร์สโค้ดและเขียนผังงาน (Code Deconstruction & Sequence Flow Assignment)

ให้นักศึกษาแกะรอยการทำงานจาก `main/main.c` ในโหมด SoftAP แล้วเขียน **แผนภาพลำดับเหตุการณ์ (Sequence Diagram)**:

### ภารกิจที่ 1: ผังลำดับการสื่อสารผ่าน HTTP Endpoints (SoftAP Scheme Sequence Flow)
ให้นักศึกษาวาด Sequence Diagram แสดงปฏิสัมพันธ์ระหว่าง 3 ฝ่าย:
1. **Smartphone App (ESP SoftAP Prov)**
2. **ESP32 SoftAP Webserver (Protocomm Layer)**
3. **Wi-Fi Router (AP ปลายทาง)**

**จุดที่ต้องระบุในผัง:**
- จังหวะที่มือถือยิง HTTP POST ไปยัง Endpoint แต่ละตัว (`/prov-session`, `/prov-scan`, `/prov-config`)
- Event ของ ESP-IDF ที่ถูก Trigger ใน `event_handler()` เช่น:
  - `WIFI_EVENT_AP_STACONNECTED`
  - `PROTOCOMM_SECURITY_SESSION_SETUP_OK`
  - `WIFI_PROV_CRED_RECV`
  - `WIFI_PROV_CRED_SUCCESS`
  - `IP_EVENT_STA_GOT_IP`
- สถานะจังหวะการกระพริบของ **LED 3 (GPIO 5)** และ **LED 1 (GPIO 2)** ในแต่ละช่วง

<img width="852" height="643" alt="ภาพถ่ายหน้าจอ 2569-10-31 เวลา 23 02 32" src="https://github.com/user-attachments/assets/64ecc5cf-664e-4a00-97d9-f10f9783f3cf" />

--

##  ตารางบันทึกผลการทดลอง (Experiment Results)

| รายการตรวจสอบ | ค่าที่บันทึกได้จากการทดลอง |
| :--- | :--- |
| **1. ชื่อ SoftAP SSID ของ ESP32** | `PROV_39C080` |
| **2. รหัส PoP (Proof of Possession)** | abcd1234 |
| **3. ข้อความใน QR Code Payload (JSON)** | {"ver":"v1","name":"PROV_39C080","pop":"abcd1234","transport":"softap"}	 |
| **4. พฤติกรรมไฟ LED 3 (GPIO 5) ช่วงรอ vs ช่วงส่งข้อมูล** | ช่วงรอ: ติดค้าง (ไม่กระพริบ) <br/>ช่วงส่งข้อมูล: ติดค้างเหมือนเดิม แล้วดับเมื่อ Provisioning สำเร็จ |
| **5. IP Address ที่ ESP32 ได้รับจาก Router** | 192.168.4.1 |
| **6. เวลาที่ใช้ตั้งแต่เริ่มจนจบกระบวนการ (วินาที)** | 35.4 วินาที |

---

##  คำถามท้ายการทดลอง (Post-Lab Questions)
1. ในโหมด SoftAP Scheme สมาร์ตโฟนส่งข้อมูลหา ESP32 ผ่านโปรโตคอลและ IP Address ใด?
```text
ส่งผ่านโปรโตคอล HTTP ผ่านช่องทาง Wi-Fi และเชื่อมต่อไปยัง IP เกตเวย์ของ ESP32 คือ 192.168.4.1
```
2. หากผู้ใช้ป้อนรหัสผ่าน Wi-Fi ผิดในแอปมือถือ จะเกิด Event ใดขึ้นบน ESP32 (`WIFI_PROV_CRED_FAIL` หรือไม่) และ ESP32 มีพฤติกรรมอย่างไร?
```text
จะเกิด Event WIFI_PROV_CRED_FAIL บน ESP32 โดยอุปกรณ์จะส่งสถานะความล้มเหลวกลับไปที่แอป และยังคงค้างสถานะ SoftAP เดิมเพื่อให้ผู้ใช้กรอกรหัสผ่านใหม่อีกครั้ง
```
3. ทำไมผู้ผลิต IoT ส่วนใหญ่จึงมองว่ากระบวนการเชื่อมต่อแบบ SoftAP มีขั้นตอนที่ยุ่งยากสำหรับผู้ใช้ทั่วไปเมื่อเทียบกับ BLE?
```text
เพราะผู้ใช้ต้องกดสลับไปหน้าตั้งค่า Wi-Fi ของมือถือเพื่อเชื่อมต่อกับ SSID ของอุปกรณ์ด้วยตนเองก่อน แล้วค่อยกลับมาที่แอปพลิเคชัน ซึ่งหลายขั้นตอนและทำให้เน็ตมือถือหลุดจากอินเทอร์เน็ตชั่วคราว
```
