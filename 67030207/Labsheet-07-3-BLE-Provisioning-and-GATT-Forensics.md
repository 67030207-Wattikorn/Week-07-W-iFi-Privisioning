## กิจกรรมถอดรหัสซอร์สโค้ดและเขียนผังงาน (Code Deconstruction & BLE GATT Architecture Assignment)

ให้นักศึกษาแกะรอยการทำงานของโมดูล BLE Provisioning ใน `main/main.c` แล้วเขียน **ผังโครงสร้างและลำดับเหตุการณ์**:

### ภารกิจที่ 1: ผังโครงสร้าง GATT Tree & Endpoint Mapping
ให้นักศึกษาวาดโครงสร้างต้นไม้ (Tree Diagram / Block Diagram) แสดงความสัมพันธ์ระหว่าง:
- **Primary Service (128-bit UUID: `021a9004-...`)**
  - **Characteristic UUIDs** แต่ละตัว
  - **Descriptor 0x2901 (User Description)** ที่ผูกเข้ากับ Protocomm Endpoints (`prov-session`, `prov-config`, `prov-scan`, `proto-ver`, `custom-data`)

<img width="937" height="136" alt="ภาพถ่ายหน้าจอ 2569-10-31 เวลา 23 48 51" src="https://github.com/user-attachments/assets/c70c7fbb-a407-44ac-a535-13b9723b1fff" />


### ภารกิจที่ 2: ผังลำดับการคืนหน่วยความจำ Bluetooth (BLE Lifecycle & Memory Reclaim Flow)
ให้นักศึกษาวาด Flowchart / Sequence แสดงว่า:
1. การเชื่อมต่อ BLE ถูกตรวจพบผ่าน Event `PROTOCOMM_TRANSPORT_BLE_CONNECTED` (LED 2 กระพริบเร็ว 100ms)
2. เมื่อเชื่อมต่อ Wi-Fi สำเร็จ (`WIFI_PROV_CRED_SUCCESS`) $\rightarrow$ เกิด Event `WIFI_PROV_END`
3. Provisioning Manager สั่งเรียก `esp_bt_mem_release()` เพื่อปล่อย DRAM คืนสู่ระบบอย่างไร

<img width="308" height="645" alt="ภาพถ่ายหน้าจอ 2569-10-31 เวลา 23 49 17" src="https://github.com/user-attachments/assets/5f6e7cb4-4c36-4324-93a9-023aebd3ad76" />


##  ตารางบันทึกผลการทดลอง (Experiment Results)

| รายการตรวจสอบ | ผลการทดลอง / ข้อมูลที่สังเกตได้ |
| :--- | :--- |
| **1. BLE Device Name ที่สแกนเจอ** | `PROV_39C080` |
| **2. Primary Service UUID (128-bit)** | 021a9004-0382-4aea-bff4-6b3f1c5adfb4 |
| **3. Characteristic Endpoint ที่พบ (0x2901)** | 1. prov-session<br/>2. prov-config<br/>3. prov-scan |
| **4. พฤติกรรมไฟ LED 2 (GPIO 4) ช่วงรอ vs ช่วงต่อ BLE** | ช่วงรอ: กะพริบสลับสี/จังหวะ<br/>ช่วงต่อ: ติดนิ่ง |
| **5. พฤติกรรมเมื่อต่อ Wi-Fi สำเร็จ** | มี Log คืนหน่วยความจำ Bluetooth หรือไม่? | มี Log คืนแรม (esp_bt_mem_release) สำเร็จหลังเชื่อมต่อ Wi-Fi |

---

##  คำถามท้ายการทดลอง (Post-Lab Questions)
1. เหตุใด BLE Provisioning จึงไม่ส่งผลให้สัญญาณ Wi-Fi บนสมาร์ตโฟนของผู้ใช้หลุดระหว่างทำรายการ?
```text
เพราะสมาร์ตโฟนเชื่อมต่อกับ ESP32 ผ่านคลื่นวิทยุบลูทูธ (Bluetooth Low Energy) คนละย่านความถี่กับ Wi-Fi หลักที่ใช้งานอินเทอร์เน็ต เน็ตมือถือจึงยังใช้งานได้ต่อเนื่อง
```

2. Descriptor `0x2901` มีความสำคัญอย่างไรต่อการที่แอปพลิเคชันมือถือจะทราบว่า Characteristic แต่ละตัวใช้ทำหน้าที่อะไร?
```text
เป็น Characteristic User Description ที่ช่วยอธิบายชื่อหรือหน้าที่ของ Endpoint นั้นๆ ให้แอปพลิเคชันมือถืออ่านเข้าใจว่า Service นี้ใช้สำหรับตั้งค่าอะไร
```
3. การที่ ESP-IDF มีฟังก์ชัน `esp_bt_mem_release()` มีประโยชน์อย่างไรต่อการทำงานของแอปพลิเคชัน IoT หลังเชื่อมต่อ Wi-Fi สำเร็จ?
```text
ช่วยคืนพื้นที่หน่วยความจำแรม (RAM) ส่วนที่เคยใช้รัน Stack ของ Bluetooth ให้ระบบนำกลับไปใช้งานทั่วไปหลังเชื่อมต่อสำเร็จ เนื่องจากเลิกใช้บลูทูธแล้ว ช่วยเพิ่มพื้นที่หน่วยความจำให้แอปพลิเคชันหลักทำงานได้เสถียรขึ้น
```
