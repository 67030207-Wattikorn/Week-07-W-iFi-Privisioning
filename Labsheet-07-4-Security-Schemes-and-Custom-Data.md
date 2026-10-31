##  กิจกรรมถอดรหัสซอร์สโค้ดและเขียนผังงาน (Code Deconstruction & Security Flow Assignment)

ให้นักศึกษาแกะรอยการทำงานด้านความปลอดภัยและ Custom Handler ใน `main/main.c` แล้วเขียน **ผังการไหลของข้อมูล (Data Flow & Cryptographic Handshake Flow)**:

### ภารกิจที่ 1: ผังขั้นตอนการตรวจสอบ PoP (Security Handshake Decision Flow)
ให้นักศึกษาวาด Flowchart แสดงการแลกเปลี่ยนคีย์และตรวจสอบสิทธิ์:
1. การสร้าง Session Parameters ด้วยค่า PoP (`abcd1234`)
2. เมื่อ Client ส่ง Public Key + Verification Hash มาให้ ESP32
3. จุดแยกทางเลือก (Branching):
   - หาก PoP ไม่ตรง $\rightarrow$ Trigger Event `PROTOCOMM_SECURITY_SESSION_CREDENTIALS_MISMATCH` และปฏิเสธการเชื่อมต่อ
   - หาก PoP ถูกต้อง $\rightarrow$ Trigger Event `PROTOCOMM_SECURITY_SESSION_SETUP_OK` และสร้าง AES Session Key สำเร็จ
<img width="689" height="659" alt="ภาพถ่ายหน้าจอ 2569-11-01 เวลา 00 25 55" src="https://github.com/user-attachments/assets/fc2e0dd6-1375-487b-b418-1089c7725cbb" />


### ภารกิจที่ 2: ผังการรับส่งข้อมูลผ่าน Custom Endpoint (Custom Data Handler Flow)
ให้นักศึกษาวาด Sequence / Data Flow ของฟังก์ชัน `custom_prov_data_handler()`:
1. ข้อมูล `inbuf` ถูกส่งเข้ามาจากสมาร์ตโฟนผ่าน Protocomm
2. การพิมพ์ข้อความ Log ด้วย `ESP_LOGI()`
3. การจัดสรรหน่วยความจำแบบไดนามิกด้วย `strdup()` ให้กับ `*outbuf`
4. ทำไมตัวแปร `*outbuf` จึงต้องจัดสรรใน Heap Memory (ทำไมจึงใช้ตัวแปร Local Static Array ธรรมดาไม่ได้)?

<img width="969" height="535" alt="ภาพถ่ายหน้าจอ 2569-11-01 เวลา 00 26 22" src="https://github.com/user-attachments/assets/2c3adfcd-965d-4f95-9216-9c8c0e9755a8" />



---
##  ตารางบันทึกผลการทดลอง (Experiment Results)

| สถานการณ์ทดสอบ          | ค่า PoP ที่ป้อน | ผลลัพธ์บนแอปมือถือ                                                        | ข้อความ Log ใน Serial Monitor                                                                                                                                 |
| :---------------------- | :-------------- | :------------------------------------------------------------------------ | :------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| **1. ป้อน PoP ผิดพลาด** | `wrong1234`     |แอปแสดงข้อความสีส้ม “Please retry with correct proof of possession PIN.” และค้างอยู่หน้า Proof of Possession|E (193987) LAB7_4_CUSTOM: [SECURITY ALERT]: INVALID PoP / Unauthorized Access!                                                                      |
| **2. ป้อน PoP ถูกต้อง** | `abcd1234`      |ผ่านไปหน้า “Select Wi-Fi Network” แสดงรายชื่อ Wi-Fi ที่ ESP32 สแกนเจอ|I (218367) LAB7_4_CUSTOM: [SECURITY SUCCESS]: Valid PoP! Secured Session OK!|
| **3. ส่ง Custom Data**  | `TEST_DATA_999` |ส่งด้วย esp_prov.py (แอปมาตรฐานไม่มีช่องส่ง): Custom data sent successfully, ได้ response ACK_FROM_ESP32_SUCCESS|I (22150) app: Received custom data: TEST_DATA_999   |

---

##  คำถามท้ายการทดลอง (Post-Lab Questions)
1. การใช้ **Proof-of-Possession (PoP)** ช่วยป้องกันการโจมตีประเภทใดได้บ้าง?
		**ตอบ:** ช่วยป้องกันการโจมตีประเภท **Man-in-the-Middle (MitM)** และ **Rogue Provisioning** (การที่ผู้ไม่หวังดีในรัศมีสัญญาณบลูทูธพยายามแอบเชื่อมต่อเข้ามาสวมรอย ดักจับข้อมูล หรือแอบส่งค่า Wi-Fi ปลอมเข้ามาเปลี่ยนการตั้งค่าอุปกรณ์) โดย PoP จะทำหน้าที่เป็น Shared Secret เพื่อยืนยันสิทธิ์ระหว่างแอปกับอุปกรณ์ก่อนสร้าง Session
2. หากไม่มีการใช้ PoP (เช่น ใน Security 0) ผู้โจมตีที่อยู่ในรัศมีสัญญาณบลูทูธสามารถทำสิ่งใดกับอุปกรณ์ได้บ้าง?
		สามารถทำได้ 2 กรณีหลัก คือ:
		1. **ดักจับรหัสผ่าน (Eavesdropping):** ดักอ่านข้อมูล SSID และ Password ของ Wi-Fi ได้โดยตรงเนื่องจากไม่มีการเข้ารหัสด้วย Session Key ที่ปลอดภัย
		2. **ก่อกวนระบบ (Unauthorized Control / DoS):** แอบส่ง SSID/Password ของ Access Point ปลอมเข้ามาเพื่อสั่งให้อุปกรณ์เปลี่ยนไปต่อกับ Wi-Fi อื่น จนทำให้อุปกรณ์หลุดออกจากระบบงานหลัก
3. ในการประยุกต์ใช้งานเชิงพาณิชย์จริง เราสามารถนำ **Custom Data Endpoint** ไปใช้ส่งข้อมูลประเภทใดได้อีกบ้าง (ยกตัวอย่าง 2 กรณี)?
		- **ส่ง MQTT / Cloud Server Configuration:** ส่ง URL ของ MQTT Broker, Port, Client ID, หรือ Device Certificate เพื่อให้อุปกรณ์นำไปใช้ต่อเข้ากับ Cloud Platform (เช่น AWS IoT, Azure IoT, NETPIE) ได้ทันที
		- **ส่ง Device Binding / User Account Token:** ส่ง User ID, Email หรือ Activation Token จากแอปพลิเคชันมือถือลงไปยังอุปกรณ์ เพื่อผูกสิทธิ์ความเป็นเจ้าของอุปกรณ์เข้ากับบัญชีผู้ใช้ในระบบ
4. ในฟังก์ชัน `custom_prov_data_handler()` เหตุใดหน่วยความจำที่จัดสรรให้ `*outbuf` จึงถูก Free โดย Protocomm Layer อัตโนมัติหลังจากส่งข้อมูลเสร็จ?
		เพราะตามดีไซน์ของ ESP-IDF Protocomm Framework หน้าที่ของ Handler คือแค่เตรียมและจองพื้นที่ข้อมูล (`malloc` / `strdup`) แล้วส่ง Pointer คืนให้ Protocomm Layer นำไปเข้ารหัสและส่งออกทางฮาร์ดแวร์ และเพื่อป้องกันปัญหา **Memory Leak** (หน่วยความจำรั่วไหล) ทาง Protocomm Layer ซึ่งเป็นตัวผู้จัดการการส่งแพ็กเก็ต จึงต้องรับหน้าที่สั่ง `free(*outbuf)` คืนพื้นที่ Dynamic Memory ให้กับระบบทันทีที่กระบวนการส่งข้อมูลสำเร็จเรียบร้อยแล้ว
--- 
