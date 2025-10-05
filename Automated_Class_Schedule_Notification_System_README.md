# 📘 Automated Class Schedule Notification System

This project automates class schedule management and SMS notifications for faculty using a **NodeMCU (ESP8266)**, **SIM800L GSM module**, **RTC DS3231**, and **LCD (I2C)** display.  

It displays current class periods on an LCD and automatically sends SMS alerts to faculty before their scheduled classes.

---

## 🧠 Features

- ✅ Real-time clock tracking using **RTC DS3231**  
- ✅ **LCD (I2C 16x2)** display showing current time, day, and schedule  
- ✅ Automatic **SMS alerts** to faculty for each period using **SIM800L GSM** module  
- ✅ Schedule data stored permanently in **EEPROM**  
- ✅ Easily update or view schedules via **Serial Monitor**  
- ✅ Supports **multiple sections (A, B, C)** and up to **10 periods per section**

---

## 🛠️ Components Used

| Component | Description |
|------------|--------------|
| NodeMCU (ESP8266) | Main controller |
| SIM800L GSM Module | Sends SMS notifications |
| DS3231 RTC Module | Maintains accurate time |
| 16x2 I2C LCD | Displays real-time information |
| Push Button | Switch between modes (Auto/Manual Update) |
| EEPROM (internal) | Stores schedule details |
| Wires, Breadboard, Power Supply | Supporting components |

---

## ⚙️ Pin Connections

| Component | NodeMCU Pin | Description |
|------------|-------------|-------------|
| LCD SDA | D2 | I2C Data |
| LCD SCL | D1 | I2C Clock |
| SIM800L RX | D5 | SoftwareSerial RX |
| SIM800L TX | D6 | SoftwareSerial TX |
| Mode Button | D7 | Switch between run/update modes |
| VCC | 3.3V | Power supply |
| GND | G | Ground |

---

## 💻 Libraries Required

Before uploading, install these libraries from the **Arduino Library Manager**:

1. `Wire.h` (built-in)
2. `LiquidCrystal_I2C.h`
3. `SoftwareSerial.h`
4. `EEPROM.h`
5. `RTClib.h`

---

## 📋 Setup Instructions

1. **Install Libraries**  
   Open Arduino IDE → Sketch → Include Library → Manage Libraries → Search and Install the above libraries.

2. **Connect Components**  
   Use the pin connections as shown in the table above.

3. **Upload the Code**  
   - Choose **NodeMCU 1.0 (ESP-12E Module)** board.  
   - Set correct **COM Port** and **Upload Speed (115200)**.

4. **Power the Device**  
   After uploading, power on your NodeMCU and wait for initialization.

5. **GSM Setup**  
   - Wait for “GSM Ready” message on LCD.  
   - Insert SIM card into SIM800L with sufficient balance/SMS pack.

---

## 🔧 Serial Commands

When the **Mode Button (D7)** is **LOW**, you can interact through Serial Monitor:

| Command | Description |
|----------|--------------|
| `U` | Update or View schedule |
| `T` | Set RTC time/date manually |
| `E` | Exit from manual mode |

### 🧾 Example Serial Flow for Updating Schedule
```
Enter Sec (1-A, 2-B, or 3-C):
1
Enter period number (1 to 10):
2
Do you want to view (v) or update (u) the schedule?
u
Enter subject name:
Data Structures
Enter faculty name:
Dr. Ramesh
Enter faculty mobile number:
9876543210
Schedule updated successfully!
```

---

## ⏰ Daily Automation

- The RTC tracks time continuously.  
- During each class period:
  - LCD shows the **current period and subject**.
  - An **SMS** is sent to the respective faculty indicating their class details.
- Automatically resets period status at the end of the day.

---

## 📲 Example SMS Output

```
Dear Sir/Madam,
You have a class to Sec-A
Subject: Data Structures
```

---

## 🧩 EEPROM Memory Map

| Section | EEPROM Start Address | Description |
|----------|----------------------|--------------|
| Year A | 0 | Section A schedule data |
| Year B | 500 | Section B schedule data |
| Year C | 1000 | Section C schedule data |

Each period stores:
- Subject name (20 chars)
- Faculty name (20 chars)
- Mobile number (15 chars)

---

## ⚡ Troubleshooting

| Issue | Possible Cause | Solution |
|-------|----------------|-----------|
| LCD not displaying | Wrong I2C address | Change `0x27` to `0x3F` in code |
| GSM not initializing | Low voltage | Use dedicated 4V–5V 2A power for SIM800L |
| SMS not sent | No SIM balance or network | Check signal and SMS pack |
| Time not correct | RTC battery weak | Replace CR2032 coin cell |

---

## 🧑‍💻 Author

**Nandu Kumar**  
BSc Computer Science (Honors) with IoT Minor  
Gayatri Vidya Parishad (A)

---

## 📜 License

This project is open-source and free to use for educational and research purposes.
