# LAB CCNA SUBNETTING

## คู่มือสร้าง Network Topology และตั้งค่า Router เบื้องต้น

**เป้าหมาย:** สร้างเครือข่าย 2 LAN เชื่อมต่อกันด้วย Router 2 ตัวผ่าน Serial Link เพื่อเตรียมทำ LAB Subnetting ตามแนวทางของ David Bombal

---

## 1. เตรียมอุปกรณ์ (Devices)

เปิด Cisco Packet Tracer แล้วเพิ่มอุปกรณ์ดังนี้

| อุปกรณ์ | จำนวน | รุ่นที่แนะนำ |
| ------- | ----: | ------------ |
| PC      |     2 | PC-PT        |
| Switch  |     2 | Cisco 2960   |
| Router  |     2 | Cisco 1941   |

## 2. วาง Topology

โครงสร้างเครือข่ายที่เราต้องการคือ

```text
 PC1                                      PC2
  |                                        |
  | Ethernet                               | Ethernet
  |                                        |
Switch1                                  Switch2
  |                                        |
  | Ethernet                               | Ethernet
  |                                        |
Router1 ============ Router2
         Serial Link
```

**หน้าที่ของอุปกรณ์**

- **PC:** อุปกรณ์ปลายทางที่ใช้ทดสอบการสื่อสาร
- **Switch:** เชื่อมต่ออุปกรณ์ภายใน LAN เดียวกัน
- **Router:** เชื่อมต่อเครือข่ายคนละ Subnet
- **Serial Link:** เชื่อมต่อ Router1 กับ Router2 ใน LAB นี้

## 3. เลือกสายให้ถูกต้อง

| การเชื่อมต่อ    | สายที่ใช้ใน Packet Tracer                         |
| --------------- | ------------------------------------------------- |
| PC → Switch     | Copper Straight-Through                           |
| Switch → Router | Copper Straight-Through                           |
| Router → Router | Serial DCE หรือ Serial DTE cable ตามตัวเลือกที่มี |

**Serial DCE คืออะไร?**

DCE ย่อมาจาก Data Communications Equipment ในการเชื่อมต่อ Serial แบบที่ใช้ใน LAB นี้ ฝั่ง DCE เป็นฝั่งที่ให้ Clocking แก่ลิงก์ ส่วน DTE รับ Clocking

ใน Packet Tracer สามารถเลือกสาย Serial แล้วเชื่อม Router ทั้งสองฝั่งได้ โดยอาจต้องตั้ง `clock rate` ที่ฝั่ง DCE ตามสถานการณ์ของ LAB

**จำไว้:** สาย Ethernet กับ Serial เป็นคนละประเภทกัน ต้องใช้ให้ตรงกับ Interface ของอุปกรณ์

## 4. เพิ่ม Serial Interface ให้ Router

Router Cisco 1941 ที่เพิ่มเข้ามาใหม่อาจมี Interface เช่น

- GigabitEthernet0/0
- GigabitEthernet0/1
- Console

แต่ยังไม่มี Serial Interface จึงต้องเพิ่ม Module ก่อน

### ขั้นตอนติดตั้ง Module

1. คลิก Router1
2. เข้าแท็บ **Physical**
3. ปิด Power ของ Router
4. หา Module ที่มี Serial Interface เช่น `HWIC-2T` หากมีให้เลือกในอุปกรณ์รุ่นนั้น
5. ลาก Module ไปใส่ Slot ที่รองรับ
6. เปิด Power กลับขึ้นมา
7. ทำซ้ำกับ Router2

หลังติดตั้งแล้ว ควรตรวจสอบว่า Router มี Serial Interface เพิ่มขึ้นมาหรือไม่ เพราะชื่อและจำนวน Interface อาจแตกต่างกันตามรุ่นและ Module

ตัวอย่าง Interface ที่อาจพบ:

```text
Router1
├── GigabitEthernet0/0
├── GigabitEthernet0/1
└── Serial0/0/0

Router2
├── GigabitEthernet0/0
├── GigabitEthernet0/1
└── Serial0/0/0
```

## 5. ตรวจสอบและเปิดใช้งาน Interface

ก่อนกำหนด IP เราต้องตรวจสอบก่อนว่า Interface ที่ใช้เชื่อมต่อมีสถานะอย่างไร

### 5.1 เข้าโหมดตั้งค่า

```cisco
enable
configure terminal
```

### 5.2 เลือก Interface ที่ต่อสายไว้

ตัวอย่าง: ถ้าสายจาก Switch1 เสียบเข้ากับ Router1 ที่ `GigabitEthernet0/0`

```cisco
interface gigabitEthernet 0/0
no shutdown
```

ถ้าสายเสียบที่ `GigabitEthernet0/1` ก็ต้องเลือก `interface gigabitEthernet 0/1` แทน

**ข้อสำคัญ:** `Gi0/1` ของ Switch สามารถต่อกับ `Gi0/0` ของ Router ได้ ไม่จำเป็นต้องเป็นหมายเลขเดียวกัน เพราะแต่ละอุปกรณ์มีหมายเลข Interface ของตัวเอง

### 5.3 ตรวจสอบสถานะ

ออกจากโหมดตั้งค่าด้วย

```cisco
end
show ip interface brief
```

ตัวอย่างผลลัพธ์:

```text
Interface              IP-Address      Status                  Protocol
GigabitEthernet0/0     unassigned      up                      up
GigabitEthernet0/1     unassigned      administratively down   down
Serial0/0/0            unassigned      administratively down   down
```

ความหมายของสถานะ:

- `up/up` — Interface และ Line Protocol ทำงานอยู่
- `administratively down/down` — Interface ถูกปิดด้วยคำสั่งตั้งค่า โดยทั่วไปคือยังไม่ได้ใช้ `no shutdown`
- `up/down` — Interface ขึ้นแล้ว แต่ Line Protocol ยังไม่ทำงานตามปกติ ต้องตรวจสอบเพิ่มเติม
- `down/down` — ลิงก์หรือ Interface ยังไม่ขึ้น อาจเกิดจากสาย อุปกรณ์ฝั่งตรงข้าม หรือการเชื่อมต่อ

**จำไว้:** `up/up` ไม่ได้แปลว่า IP ถูกต้องทั้งหมด หรือ Ping ได้แน่นอน แต่เป็นสัญญาณว่า Interface พร้อมสำหรับการตรวจสอบขั้นต่อไป

## 6. กำหนด IP Address ให้ Router1

เมื่อเลือก Interface ถูกต้องแล้ว ให้กำหนด IP Address และ Subnet Mask ให้ Interface ที่เชื่อมกับ LAN

ใน LAB นี้ เราแบ่ง `192.168.1.0/24` ออกเป็น Subnet `/26`

Subnet แรกมีข้อมูลดังนี้:

| รายการ            | ค่า               |
| ----------------- | ----------------- |
| Network Address   | `192.168.1.0`     |
| First Usable IP   | `192.168.1.1`     |
| Last Usable IP    | `192.168.1.62`    |
| Broadcast Address | `192.168.1.63`    |
| Subnet Mask       | `255.255.255.192` |

David เลือก IP สุดท้ายที่ใช้งานได้ให้ Router1 เป็น Default Gateway ของ Site 1

### คำสั่งตั้งค่า Router1

```cisco
enable
configure terminal
interface gigabitEthernet 0/0
ip address 192.168.1.62 255.255.255.192
no shutdown
end
```

### ตรวจสอบอีกครั้ง

```cisco
show ip interface brief
```

ควรเห็น IP `192.168.1.62` อยู่ที่ `GigabitEthernet0/0`

หาก Interface ยังไม่เป็น `up/up` ให้แก้ปัญหาสถานะลิงก์ก่อน แล้วจึงทดสอบการสื่อสาร

## 7. ถ้าสายยังเป็นสีแดง ต้องตรวจสอบอะไร?

อย่าเพิ่งสรุปว่าต่อสายผิดทันที ให้ตรวจตามลำดับนี้

1. ตรวจสอบว่าสายเสียบอยู่ที่ Port ไหนจริง
2. ตรวจสอบว่าเลือก Interface ตรงกับ Port ที่เสียบสาย
3. ตรวจสอบว่าใช้สายถูกประเภท
4. ตรวจสอบว่า Router เปิด Power แล้ว
5. ตรวจสอบว่า Interface ใช้ `no shutdown` แล้ว
6. ตรวจสอบ `show ip interface brief`
7. หากเป็น Serial Link ให้ตรวจสอบฝั่งตรงข้ามและ Clocking ของฝั่ง DCE ตามความจำเป็น

เมื่อเปิด Interface แล้ว ลิงก์ใน Packet Tracer อาจใช้เวลาสักครู่กว่าจะเปลี่ยนสถานะ สีของสายบอกสถานะลิงก์เบื้องต้น ไม่ได้ยืนยันว่า IP Address และ Subnet ถูกต้องทั้งหมด

## 8. คำสั่งที่ควรจำใน LAB นี้

| คำสั่ง                          | ใช้ทำอะไร                                            |
| ------------------------------- | ---------------------------------------------------- |
| `enable`                        | เข้า Privileged EXEC Mode                            |
| `configure terminal`            | เข้า Global Configuration Mode                       |
| `interface gigabitEthernet 0/0` | เข้าไปตั้งค่า Interface                              |
| `ip address IP MASK`            | กำหนด IP Address และ Subnet Mask                     |
| `no shutdown`                   | เปิดใช้งาน Interface                                 |
| `show ip interface brief`       | ตรวจสอบ IP และสถานะของ Interface ทั้งหมด             |
| `show running-config`           | ดูค่าตั้งค่าปัจจุบัน                                 |
| `show cdp neighbors`            | ตรวจสอบอุปกรณ์ข้างเคียงและ Interface ที่เชื่อมต่อกัน |

---

## สรุปลำดับการทำงาน

```text
1. วางอุปกรณ์
      ↓
2. เพิ่ม Serial Module
      ↓
3. เชื่อมต่อสายให้ถูกประเภท
      ↓
4. ตรวจสอบว่าเสียบสายเข้าพอร์ตไหน
      ↓
5. เข้า Interface นั้นและใช้ no shutdown
      ↓
6. ตรวจสอบ show ip interface brief
      ↓
7. กำหนด IP Address และ Subnet Mask
      ↓
8. ตรวจสอบสถานะและทดสอบ Ping
```

**หลักสำคัญที่สุด:** เลือก Interface ให้ตรงกับพอร์ตที่เสียบสายก่อน จากนั้นเปิด Interface ด้วย `no shutdown` แล้วจึงกำหนด IP Address ให้ตรงกับ Subnet ที่ออกแบบไว้
