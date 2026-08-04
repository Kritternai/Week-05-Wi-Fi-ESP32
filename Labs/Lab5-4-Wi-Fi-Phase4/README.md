# รายงานการทดลอง ใบงานที่ 5.4: 4-Way Handshake & IP Assignment Phase

**รหัสนักศึกษา:** 67030011  
**ชื่อโปรเจกต์:** `Labs/Lab5-4-Wi-Fi-Phase4`  
**ไฟล์ซอร์สโค้ด:** `main/main.c`

---

## ขั้นตอนการสั่งรันโปรแกรม (ESP-IDF CLI)

### 1. กำหนดชื่อ Wi-Fi ในไฟล์ wifi_credentials.h
ไฟล์ `main/wifi_credentials.h` ได้รับการจัดตั้งแล้ว สามารถเปิดแก้ไข SSID และ Password ได้ดังนี้:
```c
#define EXAMPLE_ESP_WIFI_SSID "KBBK-IPHONE"
#define EXAMPLE_ESP_WIFI_PASS "12345678"
```

### 2. ตั้งค่า Target และ Build โปรแกรม
```bash
cd Labs/Lab5-4-Wi-Fi-Phase4
idf.py set-target esp32
idf.py build
```

### 3. Flash โค้ดลงบอร์ด ESP32 และเปิด Serial Monitor
```bash
idf.py -p <PORT> flash monitor
```

---

## บันทึกผลการทดลอง (Experiment Results)

### 1. ตารางสรุปเปรียบเทียบผลการทดลองใน Handshake & IP Phase

| ข้อการทดลอง | สถานการณ์ทดสอบ | Event `WIFI_EVENT_STA_CONNECTED` (เกิด/ไม่เกิด) | Event `IP_EVENT_STA_GOT_IP` (เกิด/ไม่เกิด) | ผลการทดลอง | Disconnect Reason Code (ถ้ามี) |
| :---: | :--- | :---: | :---: | :---: | :--- |
| **5.4.1** | Password ถูกต้อง | เกิด | เกิด | สำเร็จ (Passed) | - |
| **5.4.2** | Password ผิด | เกิด (ในเฟส Auth/Assoc) | ไม่เกิด | ล้มเหลว (Failed) | 202 (`WIFI_REASON_AUTH_FAIL`) / 204 |

### 2. บันทึกข้อมูล IP Network จาก Event `IP_EVENT_STA_GOT_IP` (ข้อ 5.4.1)

| พารามิเตอร์ Network Layer | ค่าที่จัดสรรได้จริงจาก DHCP Server |
| :--- | :--- |
| **IP Address** | 172.20.10.2 |
| **Subnet Mask** | 255.255.255.240 |
| **Default Gateway** | 172.20.10.1 |

### 3. ภาพถ่ายผลการรันโปรแกรมบน Serial Console

![Serial Console Forensic Log](../../Images/4termsnap.png)

---

## ตอบคำถามท้ายการทดลอง (Post-Lab Questions)

### ข้อ 1:
เหตุใดกระบวนการ **4-Way Handshake** จึงพิสูจน์ทราบรหัสผ่าน Wi-Fi ได้โดยไม่ต้องส่งรหัสผ่าน (Passphrase) ลอยไปในอากาศเลยแม้แต่แพ็กเกจเดียว?

**คำตอบ:**  
เพราะทั้งสองฝั่ง (ESP32 และ AP) นำรหัสผ่าน (Passphrase) และชื่อ SSID มาคำนวณภายในเครื่องผ่านอัลกอริทึมแฮชชิ่ง (PBKDF2) ได้เป็นคีย์หลัก **PMK (Pairwise Master Key)** อยู่ก่อนแล้ว ในระหว่าง 4-Way Handshake ทั้งสองฝั่งเพียงแค่ส่งค่าสุ่ม ANonce และ SNonce เพื่อนำมาร่วมคำนวณคีย์ชั่วคราว **PTK** และค่าตรวจความถูกต้อง **MIC (Message Integrity Code)** ส่งให้อีกฝั่งตรวจ หากค่า MIC ที่คำนวณได้ตรงกัน จะเป็นการพิสูจน์ทันทีว่าทั้งสองฝั่งมีรหัสผ่านตรงกัน โดยไม่เคยมีการส่งตัวรหัสผ่านจริงผ่านคลื่นวิทยุเลย

---

### ข้อ 2:
อธิบายบทบาทและที่มาของคีย์ **PMK (Pairwise Master Key)** และ **PTK (Pairwise Transient Key)** ว่ามีความสัมพันธ์กันอย่างไรในการเข้ารหัสเฟรมข้อมูล?

**คำตอบ:**  
- **PMK (Pairwise Master Key):** เกิดจากการนำ Passphrase และ SSID มาผ่านฟังก์ชัน PBKDF2 (HMAC-SHA1 ทำซ้ำ 4096 รอบ) เป็นคีย์ระดับ Master ที่คงที่ตราบใดที่รหัสผ่านไม่เปลี่ยน  
- **PTK (Pairwise Transient Key):** เกิดจากการนำ **PMK** มารวมกับค่าสุ่ม ANonce (จาก AP), SNonce (จาก ESP32), MAC Address ของ AP และ MAC Address ของ ESP32 สกัดออกมาเป็นกลุ่มคีย์ชั่วคราว (KCK, KEK, TK) เพื่อนำไปใช้เข้ารหัสแพ็กเกจข้อมูลที่รับส่งจริง และตรวจสอบความถูกต้องของข้อมูล (Integrity) ตลอดเซสชันนั้น

---

### ข้อ 3:
เหตุใดเมื่อเราพิมพ์ Password ผิด (ข้อ 5.4.2) ESP32 จึงยังคงได้รับ Event **`WIFI_EVENT_STA_CONNECTED`** ก่อนที่จะเกิด Event **`WIFI_EVENT_STA_DISCONNECTED`** ตามมาในภายหลัง?

**คำตอบ:**  
ในมาตรฐาน IEEE 802.11 Event `WIFI_EVENT_STA_CONNECTED` จะเกิดขึ้น ณ จุดสิ้นสุดของ Phase 3 (Association Phase) ซึ่งเป็นขั้นตอนตกลงความสามารถทางวิทยุในระดับ Link Layer แบบ Open System (ยังไม่มีการตรวจรหัสผ่าน) เมื่อ Association สำเร็จ Driver จึงแจ้ง Event CONNECTED ทันที จากนั้นจึงเข้าสู่ Phase 4 (4-Way Handshake) เมื่อตรวจพบว่ารหัสผ่านผิด ค่า MIC จะไม่ตรงกัน AP หรือ ESP32 จึงทำการยกเลิกการเชื่อมต่อและปล่อย Event `WIFI_EVENT_STA_DISCONNECTED` พร้อม Reason Code `202` หรือ `204` ตามมาทีหลัง

---

### ข้อ 4:
หากเครือข่าย Wi-Fi ไม่มี DHCP Server (ไม่มีการแจก IP อัตโนมัติ) ผลการทดลองในข้อ 5.4.1 จะหยุดอยู่ที่ขั้นตอนใด และจะไม่เกิด Event ใดขึ้น?

**คำตอบ:**  
กระบวนการจะสามารถทำ Phase 3 (`WIFI_EVENT_STA_CONNECTED`) และ Phase 4 (4-Way Handshake แลกเปลี่ยนคีย์สำเร็จ) ได้สมบูรณ์ แต่เมื่อเข้าสู่ Phase 5 ตัวโปรโตคอล DHCP Client บน ESP32 จะส่งคำขอ IP แต่ไม่ได้รับคำตอบกลับมาจนหมดเวลา (Timeout) ทำให้กระบวนการ **หยุดลงที่ขั้นตอน DHCP Request** และ **จะไม่เกิด Event `IP_EVENT_STA_GOT_IP` ขึ้น** (บอร์ดจะไม่ได้รับหมายเลข IP อัตโนมัติ หากต้องการสื่อสารต้องกำหนด Static IP แมนนวล)

