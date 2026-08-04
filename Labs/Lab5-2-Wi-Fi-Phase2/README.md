# รายงานการทดลอง ใบงานที่ 5.2: Wi-Fi Connection & IP Assignment

**รหัสนักศึกษา:** 67030011  
**ชื่อโปรเจกต์:** `Labs/Lab5-2-Wi-Fi-Phase2`  
**ไฟล์ซอร์สโค้ด:** `main/main.c`

---

## ขั้นตอนการสั่งรันโปรแกรม (ESP-IDF CLI)

### 1. แก้ไข SSID และ Password ในไฟล์ main.c
เปิดไฟล์ `main/main.c` และเปลี่ยนค่าในบรรทัดที่ 22-23 เป็น Wi-Fi จริงที่จะทดสอบ:
```c
#define EXAMPLE_ESP_WIFI_SSID "ชื่อ_WIFI_ของคุณ"
#define EXAMPLE_ESP_WIFI_PASS "รหัสผ่าน_WIFI_ของคุณ"
```

### 2. ตั้งค่า Target และ Build โปรแกรม
```bash
cd Labs/Lab5-2-Wi-Fi-Phase2
idf.py set-target esp32
idf.py build
```

### 3. Flash โค้ดลงบอร์ด ESP32 และเปิด Serial Monitor
```bash
idf.py -p <PORT> flash monitor
```

---

## บันทึกผลการทดลอง (Experiment Results)

### 1. ตารางสรุปเปรียบเทียบผลการทดลองทั้ง 3 สถานการณ์

| ข้อการทดลอง | สถานการณ์ทดสอบ | Event สุดท้ายที่ได้รับ | ผลลัพธ์ (Passed/Failed) | Reason Code (Decimal / Hex) | คำอธิบาย Reason Code |
| :---: | :--- | :---: | :---: | :---: | :--- |
| **5.2.1** | SSID และ Password ถูกต้อง | `IP_EVENT_STA_GOT_IP` | Passed | - | - (เชื่อมต่อและรับ IP สำเร็จ) |
| **5.2.2** | ระบุ SSID ผิด (ไม่มีในระบบ) | `WIFI_EVENT_STA_DISCONNECTED` | Failed | 201 / 0xC9 | `WIFI_REASON_NO_AP_FOUND` |
| **5.2.3** | ระบุ SSID ถูกต้อง แต่ Password ผิด | `WIFI_EVENT_STA_DISCONNECTED` | Failed | 202 / 0xCA | `WIFI_REASON_AUTH_FAIL` |

### 2. บันทึกข้อมูลเครือข่ายจากการเชื่อมต่อสำเร็จ (ข้อ 5.2.1)

| พารามิเตอร์เครือข่าย | ค่าที่ได้รับจริงจาก DHCP |
| :--- | :--- |
| **SSID** | KBBK-IPHONE |
| **BSSID (MAC Address)** | D2:91:E1:1F:52:4B |
| **Channel** | 6 |
| **IP Address** | 172.20.10.2 |
| **Subnet Mask** | 255.255.255.240 |
| **Default Gateway** | 172.20.10.1 |

### 3. ภาพถ่ายผลการรันโปรแกรมบน Serial Console

![Serial Console Forensic Log](../../Images/2termsnap.png)

---

## ตอบคำถามท้ายการทดลอง (Post-Lab Questions)

### ข้อ 1:
เหตุใดการระบุ SSID ผิด (ข้อ 5.2.2) จึงส่งผลให้เกิด Disconnect Event ด้วย Reason Code `201` (`WIFI_REASON_NO_AP_FOUND`) ตั้งแต่เฟส Scan?

**คำตอบ:**  
เมื่อเรียกใช้งาน `esp_wifi_connect()` ESP32 จะเริ่มส่งเฟรม Probe Request เพื่อค้นหา AP บนช่องสัญญาณวิทยุตามชื่อ SSID ที่ระบุ เมื่อไม่มี AP ใดส่งเฟรม Probe Response ตอบกลับมาด้วยชื่อ SSID นั้น (`NON_EXISTENT_SSID_9999`) กระบวนการสแกนจึงไม่พบเป้าหมายและล้มเหลวตั้งแต่ก่อนเข้าสู่เฟส Link Layer Auth/Assoc ส่งผลให้ Wi-Fi Driver แจ้งเตือนด้วย Reason Code `201` (`WIFI_REASON_NO_AP_FOUND`)

---

### ข้อ 2:
เหตุใดการพิมพ์ Password ผิด (ข้อ 5.2.3) จึงผ่านเฟส Auth และ Assoc มาได้ แต่มาล้มเหลวในเฟส 4-Way Handshake (Reason Code `15` หรือ `204` / `202`)?

**คำตอบ:**  
ในมาตรฐาน IEEE 802.11 WPA2/WPA3 ขั้นตอน Authentication และ Association ในระดับ Link Layer (Layer 2) เป็นเพียงการตกลงคุณสมบัติและสถาปนาการเชื่อมต่อเบื้องต้นแบบ Open System (ยังไม่มีการตรวจยืนยันรหัสผ่าน PSK) การตรวจสอบรหัสผ่านจริงจะเกิดขึ้นในขั้นตอน **4-Way Handshake (EAPOL Exchange)** โดยใช้รหัสผ่านคำนวณเป็น PMK และแลกเปลี่ยนคีย์ PTK/GTK เมื่อใส่ Password ผิด ค่า MIC (Message Integrity Code) ที่แลกเปลี่ยนจะไม่ถูกต้อง ทำให้การทำ 4-Way Handshake ล้มเหลวและเกิด Reason Code `202` (`WIFI_REASON_AUTH_FAIL`) หรือ Handshake Timeout

---

### ข้อ 3:
ลำดับการเกิด Event ระหว่าง **`WIFI_EVENT_STA_CONNECTED`** กับ **`IP_EVENT_STA_GOT_IP`** Event ใดเกิดขึ้นก่อนกัน และมีความหมายทางกายภาพของ Layer Network ต่างกันอย่างไร?

**คำตอบ:**  
**`WIFI_EVENT_STA_CONNECTED` เกิดขึ้นก่อน `IP_EVENT_STA_GOT_IP`**  
- **`WIFI_EVENT_STA_CONNECTED` (Layer 2 - Data Link Layer):** มีความหมายว่า ESP32 สามารถสถาปนาการเชื่อมต่อคลื่นวิทยุ ยืนยันตัวตน และแลกเปลี่ยนคีย์ความปลอดภัยกับ Access Point สำเร็จแล้ว (ระดับ MAC Address / Wi-Fi Link)  
- **`IP_EVENT_STA_GOT_IP` (Layer 3 - Network Layer):** มีความหมายว่า ESP32 สามารถสื่อสารผ่านโปรโตคอล DHCP Client กับ DHCP Server เพื่อรับหมายเลข IP Address, Subnet Mask และ Gateway เรียบร้อยแล้ว พร้อมสำหรับการรับส่งข้อมูล TCP/IP และเชื่อมต่ออินเทอร์เน็ต

---

### ข้อ 4:
สมาชิกตัวแปร `reason` ในโครงสร้าง `wifi_event_sta_disconnected_t` มีประโยชน์อย่างไรต่อการออกแบบระบบค้นหาสาเหตุและกู้คืนการเชื่อมต่อ (Auto-Reconnection Mechanism) ในแอปพลิเคชัน IoT?

**คำตอบ:**  
ตัวแปร `reason` ช่วยให้โปรแกรมเมอร์สามารถวิเคราะห์สาเหตุการหลุดเชื่อมต่อที่แท้จริงเพื่อตัดสินใจดำเนินกลยุทธ์กู้คืนการเชื่อมต่อ (Recovery Policy) ได้อย่างถูกต้อง:
- หากเป็น `WIFI_REASON_NO_AP_FOUND (201)` หรือสัญญาณขาดหายชั่วคราว: ระบบควรรอเวลาสั้นๆ แล้วลองเชื่อมต่อใหม่ (Exponential Backoff Retry)
- หากเป็น `WIFI_REASON_AUTH_FAIL (202)`: แสดงว่ารหัสผ่านผิด การพยายาม retry ซ้ำด้วยรหัสเดิมจะสูญเสียพลังงานและล็อกเครื่องเปล่าประโยชน์ ระบบควรรีบแจ้งเตือนผู้ใช้หรือตัดเข้าสู่โหมด Provisioning (SoftAP/BLE) เพื่อให้ผู้ใช้ป้อนรหัสผ่านใหม่ทันที

