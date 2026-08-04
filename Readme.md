# รายงานสรุปผลการทำปฏิบัติการสัปดาห์ที่ 5 (Week 05: Wi-Fi ESP32)

**รหัสนักศึกษา:** 67030011  
**Git Branch:** `Lab-67030011`

---

## 1. ปฏิบัติการที่ 5.1: Wi-Fi Connection and Scanning

### 1.1 ตารางสรุปเปรียบเทียบการสแกนทั้ง 4 กรณี

| ข้อการทดลอง | เงื่อนไขการสแกน | สถานะ | จำนวน AP ที่พบ | เวลาที่ใช้ในการสแกน |
| :---: | :--- | :---: | :---: | :---: |
| **5.1.1** | สแกนทั่วไปทุก Channel (1-13) | ESP_OK | 10 | 2,493 ms |
| **5.1.2** | กำหนดสแกนเฉพาะ Channel 1 | ESP_OK | 3 | 200 ms |
| **5.1.3** | กำหนดสแกน SSID ที่มีจริง ("KBBK-IPHONE") | ESP_OK | 1 | 2,499 ms |
| **5.1.4** | กำหนดสแกน SSID ที่ไม่มีจริง ("NON_EXISTENT_AP_9999") | ESP_OK | 0 | 2,498 ms |

### 1.2 ตารางรายละเอียด AP ที่พบจากการสแกนทั่วไป (ข้อ 5.1.1)

| ลำดับ | ชื่อเครือข่าย (SSID) | MAC Address (BSSID) | ความแรงสัญญาณ (RSSI: dBm) | ช่องความถี่ (Channel) | ประเภทการเข้ารหัส (Encryption Type) |
| :---: | :--- | :--- | :---: | :---: | :--- |
| 1 | KBBK-IPHONE | F6:0D:81:F9:A8:48 | -35 dBm | 6 | WPA2_WPA3_PSK |
| 2 | KMITL-WIFI | 78:17:BE:C0:7D:A1 | -48 dBm | 1 | OPEN (No Password) |
| 3 | KMITL-Legacy | 78:17:BE:C0:7D:A0 | -50 dBm | 1 | WPA2_ENTERPRISE |
| 4 | KMITL-IoT | 78:17:BE:C0:7D:A2 | -50 dBm | 1 | WPA2_PSK |
| 5 | KMITL-IoT | 78:17:BE:C0:72:62 | -78 dBm | 11 | WPA2_PSK |
| 6 | KMITL-Legacy | 78:17:BE:C0:72:60 | -79 dBm | 11 | WPA2_ENTERPRISE |
| 7 | KMITL-WIFI | 78:17:BE:C0:72:61 | -80 dBm | 11 | OPEN (No Password) |
| 8 | KMITL-Legacy | 78:17:BE:C0:66:20 | -81 dBm | 1 | WPA2_ENTERPRISE |
| 9 | KMITL-WIFI | 78:17:BE:C0:66:21 | -82 dBm | 1 | OPEN (No Password) |
| 10 | KMITL-IoT | 78:17:BE:C0:66:22 | -82 dBm | 1 | WPA2_PSK |

### 1.3 หลักฐานการทดลอง (Serial Console)

![Serial Console Forensic Log Lab 5.1](Images/1termsnap.png)

### 1.4 คำถามท้ายการทดลอง 5.1

1. **การกำหนดค่าสแกนเจาะจงเฉพาะช่องความถี่ (ข้อ 5.1.2) ช่วยลดเวลาสแกนอย่างไร และมีข้อจำกัดอย่างไร?**  
   **คำตอบ:** ช่วยลดเวลาสแกนจาก ~2,500 ms เหลือ 200 ms เนื่องจาก Driver ไม่ต้องสลับช่องวิทยุและรอฟังในช่องอื่น แต่มีข้อจำกัดคือหาก AP ย้ายไปอยู่แชนแนลอื่น ESP32 จะไม่พบเครือข่ายนั้น

2. **ความแตกต่างระหว่าง Passive Scan และ Active Scan:**  
   **คำตอบ:** Passive Scan คือการเปิดรับฟังเฟรม Beacon จาก AP ฝั่งเดียว ไม่ส่งสัญญาณรบกวน ใช้พลังงานน้อยแต่ใช้เวลานาน ส่วน Active Scan คือการที่ ESP32 ส่ง Probe Request ออกไปเพื่อกระตุ้นให้ AP ส่ง Probe Response กลับมา ใช้เวลาน้อยกว่าแต่ใช้พลังงานสูงกว่า

3. **การประเมินความเสถียรจากค่า RSSI:**  
   **คำตอบ:** ค่า RSSI -35 dBm (ข้อ 5.1.1 ลำดับ 1) มีสัญญาณแรงมาก มีความเสถียรสูง เหมาะแก่การรับส่งข้อมูล ส่วนค่า RSSI -80 ถึง -82 dBm (ลำดับ 7-10) สัญญาณอ่อนมาก มีโอกาสแพ็กเกจหลุดร่วงและหลุดการเชื่อมต่อสูง

4. **ความสำคัญของการกรอง SSID ปลอม (ข้อ 5.1.4):**  
   **คำตอบ:** การระบุ SSID เพื่อสแกนแบบเฉพาะเจาะจงช่วยให้รับรู้ทันทีหากไม่มี AP เป้าหมาย ให้ระบบหยุดกระบวนการได้ทันที ไม่ต้องเสียพลังงานและเวลาในการพยายามสถาปนาการเชื่อมต่อไปยัง AP ที่ไม่มีจริง

---

## 2. ปฏิบัติการที่ 5.2: Wi-Fi Connection Phase

### 2.1 ตารางสรุปเปรียบเทียบผลการทดลองทั้ง 3 สถานการณ์

| ข้อการทดลอง | สถานการณ์ทดสอบ | Event สุดท้ายที่ได้รับ | ผลลัพธ์ | Reason Code | คำอธิบาย Reason Code |
| :---: | :--- | :---: | :---: | :---: | :--- |
| **5.2.1** | SSID และ Password ถูกต้อง | `IP_EVENT_STA_GOT_IP` | Passed | - | - (เชื่อมต่อและรับ IP สำเร็จ) |
| **5.2.2** | ระบุ SSID ผิด (ไม่มีในระบบ) | `WIFI_EVENT_STA_DISCONNECTED` | Failed | 201 / 0xC9 | `WIFI_REASON_NO_AP_FOUND` |
| **5.2.3** | ระบุ SSID ถูกต้อง แต่ Password ผิด | `WIFI_EVENT_STA_DISCONNECTED` | Failed | 202 / 0xCA | `WIFI_REASON_AUTH_FAIL` |

### 2.2 บันทึกข้อมูลเครือข่ายจากการเชื่อมต่อสำเร็จ (ข้อ 5.2.1)

| พารามิเตอร์เครือข่าย | ค่าที่ได้รับจริงจาก DHCP |
| :--- | :--- |
| **SSID** | KBBK-IPHONE |
| **BSSID (MAC Address)** | D2:91:E1:1F:52:4B |
| **Channel** | 6 |
| **IP Address** | 172.20.10.2 |
| **Subnet Mask** | 255.255.255.240 |
| **Default Gateway** | 172.20.10.1 |

### 2.3 หลักฐานการทดลอง (Serial Console)

![Serial Console Forensic Log Lab 5.2](Images/2termsnap.png)

### 2.4 คำถามท้ายการทดลอง 5.2

1. **เหตุใดระบุ SSID ผิด (ข้อ 5.2.2) จึงเกิด Reason Code 201 (`WIFI_REASON_NO_AP_FOUND`) ตั้งแต่เฟส Scan?**  
   **คำตอบ:** เมื่อเรียก `esp_wifi_connect()` ESP32 จะส่ง Probe Request ค้นหา SSID ที่ระบุ เมื่อไม่มี AP ตอบกลับด้วย Probe Response กระบวนการสแกนจึงล้มเหลวตั้งแต่ก่อนเข้าสู่เฟส Link Layer Auth/Assoc

2. **เหตุใดพิมพ์ Password ผิด (ข้อ 5.2.3) จึงผ่านเฟส Auth/Assoc มาได้ แต่มาล้มเหลวในเฟส 4-Way Handshake?**  
   **คำตอบ:** ขั้นตอน Auth และ Assoc ในระดับ Link Layer (Layer 2) เป็นเพียงการตกลงคุณสมบัติเบื้องต้นแบบ Open System ยังไม่มีการตรวจรหัสผ่าน PSK การตรวจรหัสผ่านจริงทำในขั้นตอน 4-Way Handshake เมื่อรหัสผ่านผิด ค่า MIC จะไม่ตรงกัน ทำให้เกิด Reason Code `202` (`WIFI_REASON_AUTH_FAIL`)

3. **ลำดับการเกิด Event ระหว่าง `WIFI_EVENT_STA_CONNECTED` กับ `IP_EVENT_STA_GOT_IP`:**  
   **คำตอบ:** `WIFI_EVENT_STA_CONNECTED` เกิดขึ้นก่อน (Layer 2 - Data Link Layer สถาปนาการเชื่อมต่อสัญญาณวิทยุสำเร็จ) ส่วน `IP_EVENT_STA_GOT_IP` เกิดขึ้นทีหลัง (Layer 3 - Network Layer ได้รับจัดสรร IP Address จาก DHCP Server)

4. **ประโยชน์ของตัวแปร `reason` ใน `wifi_event_sta_disconnected_t`:**  
   **คำตอบ:** ช่วยให้โปรแกรมเมอร์แยกแยะสาเหตุการหลุดเชื่อมต่อเพื่อกำหนดกลยุทธ์กู้คืนระบบได้อย่างถูกต้อง เช่น หากเป็น `WIFI_REASON_NO_AP_FOUND (201)` ให้ทำ Exponential Backoff Retry แต่หากเป็น `WIFI_REASON_AUTH_FAIL (202)` ให้ตัดเข้าสู่โหมด Provisioning เพื่อรับรหัสผ่านใหม่ทันที

---

## 3. ปฏิบัติการที่ 5.3: Wi-Fi Authentication & Association Phase

### 3.1 ตารางสรุปเปรียบเทียบผลการทดลองในระดับ Link Layer

| ข้อการทดลอง | สถานการณ์ทดสอบ | Event ที่ได้รับ | ผลการผูกสัมพันธ์ Link Layer | ค่า Association ID (AID) ที่ได้ | Reason Code (ถ้ามี) |
| :---: | :--- | :---: | :---: | :---: | :--- |
| **5.3.1** | ร้องขอ Auth & Assoc กับ AP มีอยู่จริง | `WIFI_EVENT_STA_CONNECTED` | สำเร็จ (Passed) | 34680 (หรือ 1) | - |
| **5.3.2** | ร้องขอ Auth & Assoc กับ AP ไม่มีอยู่จริง | `WIFI_EVENT_STA_DISCONNECTED` | ล้มเหลว (Failed) | - | 201 (`WIFI_REASON_NO_AP_FOUND`) |

### 3.2 บันทึกข้อมูล Link Layer จาก Event `WIFI_EVENT_STA_CONNECTED` (ข้อ 5.3.1)

| พารามิเตอร์ Link Layer | ค่าที่อ่านได้จริงจาก Forensic Log |
| :--- | :--- |
| **SSID** | KBBK-IPHONE |
| **BSSID (MAC Address)** | D2:91:E1:1F:52:4B |
| **Channel** | 6 |
| **Auth Mode Enum** | 6 (WPA2_WPA3_PSK / WPA3-SAE H2E) |
| **Association ID (AID)** | 34680 (หรือ 1) |

### 3.3 หลักฐานการทดลอง (Serial Console)

![Serial Console Forensic Log Lab 5.3](Images/3termsnap.png)

### 3.4 คำถามท้ายการทดลอง 5.3

1. **Association ID (AID) คืออะไร มีบทบาทอย่างไร และส่งคืนมาในโครงสร้างตัวแปรใด?**  
   **คำตอบ:** AID คือหมายเลขรหัสประจำตัวการผูกสัมพันธ์ 16-bit ที่ AP ออกให้ ESP32 ในเฟรม 802.11 Association Response มีบทบาทระบุตัวตนในระดับ Link Layer สำหรับจัดการคิวข้อมูลและ Power Saving โดยส่งคืนในโครงสร้าง `wifi_event_sta_connected_t` สมาชิก `aid` (`event->aid`)

2. **เหตุใด WPA2-PSK จึงผ่าน Phase 2 (Auth) และ Phase 3 (Assoc) เกิด Event CONNECTED ได้ แม้ป้อน Password ผิด?**  
   **คำตอบ:** เพราะ Phase 2 และ 3 ในระดับ Link Layer เป็นการตกลงเชื่อมต่อทางวิทยุแบบ Open System ยังไม่มีการตรวจสอบรหัสผ่าน Driver จึงแจ้ง Event CONNECTED ทันทีที่รับ Association Response สำเร็จ ก่อนจะไปตรวจรหัสผ่านจริงใน Phase 4

3. **หาก Router ตั้งค่า MAC Address Filtering ESP32 จะล้มเหลวในเฟสใด และเกิด Reason Code ใด?**  
   **คำตอบ:** จะล้มเหลวตั้งแต่ใน **Phase 2 (Authentication Phase)** หรือ **Phase 3 (Association Phase)** เนื่องจาก AP ปฏิเสธ MAC Address ที่ไม่อยู่ใน Whitelist และส่ง Disconnect Reason Code `202` (`WIFI_REASON_AUTH_FAIL`) หรือ `203` (`WIFI_REASON_ASSOC_FAIL`)

4. **ความแตกต่างสำคัญระหว่างจุดสิ้นสุด Phase 3 (Link-Layer Connected) กับ Phase 5 (IP Address Assigned):**  
   **คำตอบ:** จุดสิ้นสุด Phase 3 บอร์ดเชื่อมต่อสัญญาณวิทยุระดับ Layer 2 สำเร็จ ได้รับ AID แต่ยังไม่ผ่านการตรวจรหัสผ่าน Handshake และยังไม่มี IP Address จึงส่งข้อมูล IP ไม่ได้ ส่วนจุดสิ้นสุด Phase 5 ผ่าน Handshake และได้รับจัดสรร IP Address ระดับ Layer 3 เรียบร้อยแล้ว พร้อมสำหรับการสื่อสารเครือข่ายเต็มรูปแบบ

---

## 4. ปฏิบัติการที่ 5.4: 4-Way Handshake & IP Assignment Phase

### 4.1 ตารางสรุปเปรียบเทียบผลการทดลองใน Handshake & IP Phase

| ข้อการทดลอง | สถานการณ์ทดสอบ | Event `WIFI_EVENT_STA_CONNECTED` | Event `IP_EVENT_STA_GOT_IP` | ผลการทดลอง | Disconnect Reason Code (ถ้ามี) |
| :---: | :--- | :---: | :---: | :---: | :--- |
| **5.4.1** | Password ถูกต้อง | เกิด | เกิด | สำเร็จ (Passed) | - |
| **5.4.2** | Password ผิด | เกิด (ในเฟส Auth/Assoc) | ไม่เกิด | ล้มเหลว (Failed) | 202 (`WIFI_REASON_AUTH_FAIL`) / 204 |

### 4.2 บันทึกข้อมูล IP Network จาก Event `IP_EVENT_STA_GOT_IP` (ข้อ 5.4.1)

| พารามิเตอร์ Network Layer | ค่าที่จัดสรรได้จริงจาก DHCP Server |
| :--- | :--- |
| **IP Address** | 172.20.10.2 |
| **Subnet Mask** | 255.255.255.240 |
| **Default Gateway** | 172.20.10.1 |

### 4.3 หลักฐานการทดลอง (Serial Console)

![Serial Console Forensic Log Lab 5.4](Images/4termsnap.png)

### 4.4 คำถามท้ายการทดลอง 5.4

1. **เหตุใด 4-Way Handshake จึงพิสูจน์รหัสผ่านได้โดยไม่ต้องส่ง Passphrase ผ่านอากาศเลย?**  
   **คำตอบ:** เพราะทั้งสองฝั่งคำนวณคีย์หลัก **PMK** จาก Passphrase และ SSID ไว้ล่วงหน้า ภายในเครื่อง แล้วแลกเปลี่ยนค่าสุ่ม ANonce/SNonce เพื่อคำนวณคีย์ชั่วคราว **PTK** และค่าตรวจ **MIC** หากค่า MIC ตรงกัน จะเป็นการพิสูจน์ทันทีว่าทั้งสองฝั่งใช้รหัสผ่านเดียวกันโดยไม่ต้องส่ง Passphrase ผ่านอากาศ

2. **บทบาทและความสัมพันธ์ระหว่าง PMK (Pairwise Master Key) และ PTK (Pairwise Transient Key):**  
   **คำตอบ:** PMK คำนวณจาก Passphrase + SSID ด้วยฟังก์ชัน PBKDF2 (คงที่ตลอดเซสชัน) ส่วน PTK คำนวณไดนามิกใน 4-Way Handshake จาก PMK + Nonces + MAC Addresses เพื่อแยกสกัดเป็นคีย์เข้ารหัสและตรวจความถูกต้องของแพ็กเกจข้อมูลจริง

3. **เหตุใดพิมพ์ Password ผิด (ข้อ 5.4.2) จึงเกิด `WIFI_EVENT_STA_CONNECTED` ก่อนเกิด `WIFI_EVENT_STA_DISCONNECTED`?**  
   **คำตอบ:** Event CONNECTED เกิดขึ้นตั้งแต่อนุมาน Phase 3 (Assoc) สำเร็จ ซึ่งยังไม่ได้ตรวจรหัสผ่าน เมื่อเข้าสู่ Phase 4 แล้วตรวจพบว่า MIC ไม่ตรงกัน สแตก Wi-Fi จึงสั่งเลิกเชื่อมต่อและส่ง Event DISCONNECTED พร้อม Reason Code `202` หรือ `204` ตามมาทีหลัง

4. **หากเครือข่ายไม่มี DHCP Server ผลการทดลอง 5.4.1 จะหยุดอยู่ที่ขั้นตอนใด และจะไม่เกิด Event ใด?**  
   **คำตอบ:** กระบวนการจะสำเร็จใน Phase 3 และ 4 แต่จะ **หยุดลงที่ขั้นตอน DHCP Request** ใน Phase 5 เนื่องจากหมดเวลาส่งคำขอ และ **จะไม่เกิด Event `IP_EVENT_STA_GOT_IP` ขึ้น**
