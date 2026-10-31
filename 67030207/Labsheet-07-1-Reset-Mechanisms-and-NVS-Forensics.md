### ภารกิจที่ 1  ผังงานการตัดสินใจช่วง Bootstrapping & Reset Decision
ให้นักศึกษาวาด Flowchart แสดงลำดับตรรกะการตรวจสอบเงื่อนไขตั้งแต่เริ่มต้นรันฟังก์ชัน `app_main()` โดยต้องครอบคลุม:
1. การตรวจสอบสถานะปุ่ม **GPIO 18** (ตรวจจับการกดค้าง 3 วินาที)
2. การทำงานของ `nvs_flash_init()` และกรณีที่ต้อง `nvs_flash_erase()`
3. การตรวจสอบ Macro `#ifdef CONFIG_EXAMPLE_RESET_PROVISIONED`
4. การเรียกฟังก์ชัน `wifi_prov_mgr_is_provisioned(&provisioned)`
5. จุดแยกสายการทำงานเข้าสู่โหมด **Provisioning Mode** หรือ **Station Mode**

<img width="186" height="656" alt="ภาพถ่ายหน้าจอ 2569-10-31 เวลา 22 45 35" src="https://github.com/user-attachments/assets/c8dd2ccd-12da-480d-910b-c6d9b4cb60aa" />





### ภารกิจที่ 2 ผังสถานะการเปลี่ยนจังหวะไฟ LED 1 (Wi-Fi STA Indicator)
ให้นักศึกษาวาด State Diagram แสดงการเปลี่ยนสถานะของ **LED 1 (GPIO 2)**:
- เงื่อนไขใดทำให้ LED 1 เข้าสู่สถานะ `LED_STA_MODE_DISCONNECTED` (กระพริบ 200ms Mark / 200ms Space)
- เงื่อนไขหรือ Event ใดทำให้เปลี่ยนเป็น `LED_STA_MODE_CONNECTED` (Heartbeat 200ms ทุก 1s)

<img width="583" height="192" alt="ภาพถ่ายหน้าจอ 2569-10-31 เวลา 22 47 35" src="https://github.com/user-attachments/assets/6ae96837-784c-440d-8b4a-3bbaed048e95" />


---

## ตารางบันทึกผลการทดลอง (Experiment Results)

| รูปแบบการ Reset                  | คำสั่ง / พฤติกรรมที่ทำ               | พฤติกรรมของ LED แต่ละดวงหลังเปิดเครื่อง                                          | สถานะใน Serial Monitor                                                              |
| :------------------------------- | :----------------------------------- | :------------------------------------------------------------------------------- | :---------------------------------------------------------------------------------- |
| **1. CLI Erase**                 | `idf.py erase-flash` | กะพริบช้า   | Flash erased completely. Starting provisioning mode... |
| **2. Menuconfig Flag**           | `CONFIG_EXAMPLE_RESET_PROVISIONED=y` | กะพริบถี่ต่อเนื่อง |Forced reset provisioned data from menuconfig.  |
| **3. Hardware Button (GPIO 18)** | กดปุ่ม GPIO 18 ค้าง 3 วินาที         | ดับแล้วเปลี่ยนเป็นกะพริบ |Button pressed. Erasing NVS credentials and restarting...  |

---

##  คำถามท้ายการทดลอง (Post-Lab Questions)
1. เพราะเหตุใดการกดปุ่ม BOOT (GPIO 0) ค้างไว้ในจังหวะรีเซ็ตบอร์ด จึงทำให้โปรแกรมค้างอยู่ที่ ROM Bootloader และไม่ยอมทำงานต่อ?
```text
เพราะ GPIO 0 ถูกใช้เป็นฮาร์ดแวร์พินสำหรับสลับโหมดการบูตของชิป ESP32 หากดึงลงกราวด์ค้างไว้ ชิปจะเข้าใจว่าต้องการเข้าสู่โหมด Serial Download (Bootloader) เพื่อรอรับแฟลชเฟิร์มแวร์ใหม่ จึงไม่ยอมรันโปรแกรมหลัก
```
2. เพราะเหตุใดคำสั่ง `idf.py erase-flash` จึงทำให้ข้อมูลเฟิร์มแวร์ Application หายไปด้วย ในขณะที่ `nvs_flash_erase()` ไม่ทำให้เฟิร์มแวร์หาย?
```text
เพราะคำสั่ง erase-flash ลบข้อมูลทั้งชิป (รวมถึง Partition Table, Bootloader และ App Partition) แต่ nvs_flash_erase() เป็นการลบข้อมูลเฉพาะพาร์ติชัน NVS (Non-Volatile Storage) ที่เก็บค่า Config และ Wi-Fi credentials เท่านั้น ตัวเฟิร์มแวร์แอปพลิเคชันยังอยู่ครบ
```
3. การออกแบบปุ่ม Factory Reset บนอุปกรณ์ IoT เชิงพาณิชย์ เหตุใดจึงต้องกำหนดให้ผู้ใช้กดปุ่มค้างไว้ 3-5 วินาที แทนที่จะสั่งลบข้อมูลทันทีที่แตะปุ่มเพียงเสี้ยววินาที?
```text
เพื่อป้องกันไม่ให้ผู้ใช้เผลอไปโดนหรือกดโดนปุ่มรีเซ็ตโดยอุบัติเหตุขณะใช้งานปกติ ซึ่งช่วยเพิ่มความปลอดภัยและความเสถียรให้กับอุปกรณ์
```
4. หากอุปกรณ์ IoT ถูกติดตั้งอยู่บนเสาสูงหรือฝังอยู่ในผนัง วิธีการ Reset ทางกายภาพรูปแบบใดเหมาะสมที่สุด?
```text
ควรใช้วิธี Soft-reset ผ่านแอปพลิเคชันมือถือ/เว็บพอร์ทัล หรือออกแบบให้มีกลไกกดรีเซ็ตผ่านคำสั่งบน Cloud / MQTT หรือใช้แม่เหล็กเข้าใกล้ Reed Switch ภายในตัวเครื่องแทนการกดปุ่มตรงๆ
```
