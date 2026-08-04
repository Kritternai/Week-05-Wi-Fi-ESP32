# รายงานการทดลองและบันทึกผลใบงาน สัปดาห์ที่ 5 (Wi-Fi ESP32)
**รหัสนักศึกษา:** 67030011  
**Branch:** `Lab-67030011`

---

## 📁 โครงสร้างโฟลเดอร์สำหรับทำใบงาน (Workspace Setup)

- `Labs/Lab5-1-Wi-Fi-Phase1/` - โค้ดและรายงานการทดลอง ใบงานที่ 5.1 (Wi-Fi Scan Phase)
- `Labs/Lab5-2-Wi-Fi-Phase2/` - โค้ดและรายงานการทดลอง ใบงานที่ 5.2 (Wi-Fi Connection Phase)
- `Labs/Lab5-3-Wi-Fi-Phase3/` - โค้ดและรายงานการทดลอง ใบงานที่ 5.3 (Wi-Fi Auth & Assoc Phase)
- `Labs/Lab5-4-Wi-Fi-Phase4/` - โค้ดและรายงานการทดลอง ใบงานที่ 5.4 (Wi-Fi 4-Way Handshake & IP Phase)

---

## 📝 ภาพรวมใบงานที่ 5.1 - 5.4

### 1. ใบงานที่ 5.1: Wi-Fi Connection and Scanning (`Labs/Lab5-1-Wi-Fi-Phase1`)
- **วัตถุประสงค์:** ทดสอบการสแกนหาสัญญาณ Wi-Fi (General Scan, Channel-Specific Scan, Targeted SSID Scan)
- **ไฟล์หลัก:** `Labs/Lab5-1-Wi-Fi-Phase1/main/main.c`

### 2. ใบงานที่ 5.2: Wi-Fi Connection & IP Assignment (`Labs/Lab5-2-Wi-Fi-Phase2`)
- **วัตถุประสงค์:** ทดสอบการสถาปนาการเชื่อมต่อและรับหมายเลข IP Address จาก Access Point
- **ไฟล์หลัก:** `Labs/Lab5-2-Wi-Fi-Phase2/main/main.c`

### 3. ใบงานที่ 5.3: Wi-Fi Authentication & Association Phase (`Labs/Lab5-3-Wi-Fi-Phase3`)
- **วัตถุประสงค์:** วิเคราะห์ Forensic Log ขั้นตอน Authentication และ Association (IEEE 802.11)
- **ไฟล์หลัก:** `Labs/Lab5-3-Wi-Fi-Phase3/main/main.c`

### 4. ใบงานที่ 5.4: 4-Way Handshake & IP Assignment Phase (`Labs/Lab5-4-Wi-Fi-Phase4`)
- **วัตถุประสงค์:** วิเคราะห์ Forensic Log การทำ 4-Way Handshake (EAPOL) และการรับ IP ผ่าน DHCP
- **ไฟล์หลัก:** `Labs/Lab5-4-Wi-Fi-Phase4/main/main.c`
