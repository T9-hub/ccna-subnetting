# LAB CCNA Subnetting (Packet Tracer)

คู่มือสร้าง Network Topology, ตั้งค่า Router, แจก IP ด้วย DHCP และทำ Routing ด้วย EIGRP ตามแนวทางของ David Bombal

**เป้าหมาย:** เครือข่าย 2 LAN เชื่อมกันด้วย Router 2 ตัวผ่าน Serial Link

- LAN 1: ตั้ง IP เอง (Static)
- LAN 2: PC ขอ IP จาก Router2 (DHCP)
- Router ทั้งสองแลกเส้นทางกันด้วย EIGRP

---

## 0. แผน IP (ดูตารางนี้ก่อนทุกครั้ง)

แบ่ง `192.168.1.0/24` เป็น 4 วง วงละ `/26` (Mask `255.255.255.192`)

| Subnet | ใช้กับ       | Network         | IP ที่ใช้ได้ | Broadcast | Router ใช้ (Last usable)           |
| ------ | ------------ | --------------- | ------------ | --------- | ---------------------------------- |
| 1      | LAN 1        | `192.168.1.0`   | .1 – .62     | .63       | Router1 = `.62`                    |
| 2      | LAN 2        | `192.168.1.64`  | .65 – .126   | .127      | Router2 = `.126`                   |
| 3      | WAN (Serial) | `192.168.1.128` | .129 – .190  | .191      | Router1 = `.129`, Router2 = `.130` |
| 4      | ยังว่าง      | `192.168.1.192` | .193 – .254  | .255      | -                                  |

> **หมายเหตุเรื่องชื่อ Interface:** เอกสารนี้ใช้ Cisco 1941 จึงใช้ `GigabitEthernet0/0` (LAN) และ `Serial0/0/0` (WAN) ตลอด
> ในวิดีโอของ David ใช้ Router รุ่นอื่น จึงเห็น `FastEthernet0/0` และ `Serial2/0` ถ้าอุปกรณ์ของคุณไม่ตรง ให้ดูชื่อจริงจาก `show ip interface brief`

---

## ส่วนที่ 1: สร้าง Topology และตั้งค่าพื้นฐาน

### 1. เตรียมอุปกรณ์

| อุปกรณ์ |                                     จำนวน | รุ่นที่แนะนำ |
| ------- | ----------------------------------------: | ------------ |
| PC      | 4 (PC1, PC2 ใน LAN 1 / PC3, PC4 ใน LAN 2) | PC-PT        |
| Switch  |                                         2 | Cisco 2960   |
| Router  |                                         2 | Cisco 1941   |

### 2. วาง Topology

```text
 PC1  PC2                               PC3  PC4
   \  /                                   \  /
  Switch1                                Switch2
     |                                      |
  Router1 ========== Serial Link ========== Router2
  Gi0/0 (.62)    S0/0/0 (.129)  S0/0/0 (.130)    Gi0/0 (.126)
     └── LAN 1 ──┘                          └── LAN 2 ──┘
```

หน้าที่ของอุปกรณ์:

- **PC:** เครื่องปลายทางที่ใช้ทดสอบ
- **Switch:** เชื่อมอุปกรณ์ใน LAN เดียวกัน
- **Router:** เชื่อมเครือข่ายต่าง Subnet
- **Serial Link:** เส้นทางเชื่อม Router1 กับ Router2

### 3. เลือกสายให้ถูกประเภท

| การเชื่อมต่อ    | สายใน Packet Tracer     |
| --------------- | ----------------------- |
| PC → Switch     | Copper Straight-Through |
| Switch → Router | Copper Straight-Through |
| Router → Router | Serial DCE (หรือ DTE)   |

**DCE / DTE คืออะไร?**
DCE (Data Communications Equipment) คือฝั่งที่ **ให้สัญญาณ Clock** แก่ลิงก์ ส่วน DTE เป็นฝั่งที่รับ Clock
ดังนั้นต้องตั้ง `clock rate` ที่ **ฝั่ง DCE** เท่านั้น (ดูหัวข้อ 15)

สาย Ethernet กับ Serial เป็นคนละประเภท ต้องเสียบให้ตรงกับ Interface

### 4. เพิ่ม Serial Interface ให้ Router

Cisco 1941 ใหม่ ๆ มีแค่ `Gi0/0`, `Gi0/1` และ Console จึงต้องเพิ่ม Module ก่อน

1. คลิก Router → แท็บ **Physical**
2. **ปิด Power** ของ Router
3. ลาก Module ที่มี Serial เช่น `HWIC-2T` ไปใส่ Slot ที่ว่าง
4. เปิด Power กลับ
5. ทำซ้ำกับ Router อีกตัว

ผลที่ควรได้ (ชื่ออาจต่างตามรุ่น):

```text
Router1                      Router2
├── GigabitEthernet0/0       ├── GigabitEthernet0/0
├── GigabitEthernet0/1       ├── GigabitEthernet0/1
└── Serial0/0/0              └── Serial0/0/0
```

### 5. ตรวจสอบและเปิดใช้งาน Interface

เลือก Interface ให้ **ตรงกับพอร์ตที่เสียบสายจริง** แล้วเปิดด้วย `no shutdown`

```cisco
enable
configure terminal
interface gigabitEthernet 0/0
no shutdown
end
show ip interface brief
```

พอร์ตของแต่ละอุปกรณ์เป็นเลขของตัวเอง เช่น `Gi0/1` ของ Switch ต่อกับ `Gi0/0` ของ Router ได้ ไม่ต้องหมายเลขเดียวกัน

**อ่านสถานะ Interface:**

| สถานะ                          | ความหมาย                                                       |
| ------------------------------ | -------------------------------------------------------------- |
| `up / up`                      | Interface และ Line Protocol ทำงาน                              |
| `administratively down / down` | ถูกปิดด้วยคำสั่ง ยังไม่ได้ `no shutdown`                       |
| `up / down`                    | Interface ขึ้นแล้ว แต่ Line Protocol ยังไม่ทำงาน ต้องตรวจเพิ่ม |
| `down / down`                  | ลิงก์ยังไม่ขึ้น (สาย, อีกฝั่ง, การเชื่อมต่อ)                   |

`up/up` แปลว่าพร้อมตรวจขั้นต่อไป ไม่ได้รับประกันว่า IP ถูกหรือ Ping ได้

### 6. ตั้ง IP ให้ Router1 (ฝั่ง LAN 1)

```cisco
enable
configure terminal
interface gigabitEthernet 0/0
ip address 192.168.1.62 255.255.255.192
no shutdown
end
show ip interface brief
```

ควรเห็น `192.168.1.62` ที่ `GigabitEthernet0/0` และสถานะ `up/up`

### 7. สายยังเป็นสีแดง? ตรวจตามลำดับนี้

1. สายเสียบพอร์ตไหนจริง
2. Interface ที่ตั้งค่าตรงกับพอร์ตนั้นหรือไม่
3. ใช้สายถูกประเภทหรือไม่
4. Router เปิด Power แล้วหรือยัง
5. ใช้ `no shutdown` แล้วหรือยัง
6. ดู `show ip interface brief`
7. ถ้าเป็น Serial: ตรวจอีกฝั่งและ `clock rate` ฝั่ง DCE

หลังเปิด Interface สายอาจใช้เวลาสักครู่กว่าจะเปลี่ยนสี สีสายบอกแค่สถานะลิงก์ ไม่ได้ยืนยันว่า IP ถูก

### 8. คำสั่งที่ควรจำ

| คำสั่ง                          | ใช้ทำอะไร                           |
| ------------------------------- | ----------------------------------- |
| `enable`                        | เข้า Privileged EXEC Mode           |
| `configure terminal`            | เข้า Global Configuration Mode      |
| `interface gigabitEthernet 0/0` | เลือก Interface ที่จะตั้งค่า        |
| `ip address IP MASK`            | กำหนด IP และ Subnet Mask            |
| `no shutdown`                   | เปิดใช้งาน Interface                |
| `show ip interface brief`       | ดู IP และสถานะทุก Interface         |
| `show running-config`           | ดู Config ปัจจุบัน                  |
| `show cdp neighbors`            | ดูอุปกรณ์ข้างเคียงและพอร์ตที่ต่อกัน |

---

## ส่วนที่ 2: DHCP บน Router2 (LAN 2)

**LAN 1** เราตั้ง IP เองบน PC แต่ **LAN 2** เปลี่ยนเป็น PC ขอค่าจาก Router2 โดย Router2 จะแจก

- IP Address และ Subnet Mask
- Default Gateway
- DNS Server

### 9. ตั้ง IP ให้ Router2 ฝั่ง LAN ก่อน

ต้องมี IP บนพอร์ต LAN ก่อน Router ถึงรู้ว่ากำลังแจกให้วงไหน

```cisco
Router2# configure terminal
Router2(config)# interface gigabitEthernet 0/0
Router2(config-if)# ip address 192.168.1.126 255.255.255.192
Router2(config-if)# no shutdown
Router2(config-if)# exit
```

เราใช้ **Last usable IP (.126)** ให้ Router เหมือนที่ทำกับ Router1 (.62)

### 10. DHCP Pool คืออะไร?

Pool คือ "ชุดการตั้งค่า" ที่ Router ใช้แจกให้ PC ในวงที่กำหนด ภายใน Pool เราบอก Router ว่า

```text
DHCP Pool: NewYork
├── แจก IP จาก Subnet ไหน?        → network
├── ให้ PC ใช้ Gateway อะไร?      → default-router
└── ให้ PC ใช้ DNS Server อะไร?   → dns-server
```

### 11. ยกเว้น IP ที่ไม่ให้แจก (Excluded Address)

`.126` เป็นของ Router แล้ว จึงต้องกันไม่ให้ DHCP แจกให้ PC ทำใน Global Config **ก่อนสร้าง Pool**

```cisco
Router2(config)# ip dhcp excluded-address 192.168.1.126
```

### 12. สร้าง DHCP Pool

```cisco
Router2(config)# ip dhcp pool NewYork
Router2(dhcp-config)# network 192.168.1.64 255.255.255.192
Router2(dhcp-config)# default-router 192.168.1.126
Router2(dhcp-config)# dns-server 192.168.1.126
Router2(dhcp-config)# end
```

หลังสั่ง `ip dhcp pool NewYork` Prompt จะเปลี่ยนเป็น `Router2(dhcp-config)#` แปลว่าคำสั่งถัดไปตั้งค่าภายใน Pool

| คำสั่ง                                 | ความหมาย                                                                             |
| -------------------------------------- | ------------------------------------------------------------------------------------ |
| `network 192.168.1.64 255.255.255.192` | ใช้ Subnet นี้แจก IP (เขียนสั้นเป็น `/26` ก็ได้) Router จึงรู้ขอบเขตว่า IP ไหนแจกได้ |
| `default-router 192.168.1.126`         | บอก PC ว่า Gateway คือ .126                                                          |
| `dns-server 192.168.1.126`             | บอก PC ว่า DNS Server คือ .126                                                       |

**Default Gateway ทำงานอย่างไร:**

```text
PC ใน LAN 2
├── ติดต่อเครื่องใน Subnet เดียวกัน → ส่งภายใน LAN
└── ติดต่อ IP ต่าง Subnet            → ส่งไป Default Gateway (192.168.1.126)
```

> **ข้อควรรู้:** ใน Lab นี้ชี้ DNS ไปที่ Router เพื่อให้เห็นว่า DHCP แจกค่านี้ได้ แต่ Router ไม่ได้เป็น DNS Server จริง ถ้าต้องการ Resolve ชื่อจริง ให้ใส่ IP ของ DNS Server ที่ใช้งานได้ เช่น `8.8.8.8`

ดูค่าที่ตั้งไว้: `show running-config` (หรือ `do show run` ถ้ายังอยู่ใน config mode)

### 13. ให้ PC ขอ IP และทดสอบ LAN 2

1. คลิก PC3 และ PC4 → Desktop → IP Configuration → เลือก **DHCP**
2. ตรวจด้วย `ipconfig` ว่าได้ IP, Mask, Gateway, DNS ครบ (PC แรกควรได้ `.65`, ถัดไป `.66`)

จาก PC4:

```text
ping 192.168.1.65     # PC4 ติดต่อ PC3 ได้ไหม
ping 192.168.1.126    # PC4 ติดต่อ Gateway (พอร์ต LAN ของ Router2) ได้ไหม
```

### 14. บันทึก Configuration

```cisco
Router2(config)# end
Router2# copy running-config startup-config
```

บันทึก Config ที่ใช้อยู่ ให้ Router นำกลับมาใช้ตอนเริ่มทำงานใหม่ ทำทั้งสอง Router

---

## ส่วนที่ 3: ตั้ง IP ให้ WAN (Serial) ระหว่าง Router

พอร์ต LAN ใช้ Last usable ของ Subnet ตัวเองไปแล้ว ตอนนี้ตั้งพอร์ต Serial ซึ่งใช้ **Subnet ที่ 3: `192.168.1.128/26`**

```text
Router1 S0/0/0                         Router2 S0/0/0
192.168.1.129 ================ 192.168.1.130
              Serial Link  (192.168.1.128/26)
```

### 15. คำสั่ง

**Router1**

```cisco
Router1# configure terminal
Router1(config)# interface serial 0/0/0
Router1(config-if)# ip address 192.168.1.129 255.255.255.192
Router1(config-if)# no shutdown
Router1(config-if)# end
```

**Router2**

```cisco
Router2# configure terminal
Router2(config)# interface serial 0/0/0
Router2(config-if)# ip address 192.168.1.130 255.255.255.192
Router2(config-if)# no shutdown
Router2(config-if)# end
```

**ตั้ง Clock ที่ฝั่ง DCE เท่านั้น** (ดูว่าฝั่งไหนเป็น DCE ด้วย `show controllers serial 0/0/0`)

```cisco
Router1(config-if)# clock rate 64000
```

### 16. ทดสอบ

```cisco
Router1# ping 192.168.1.130
```

นี่ทดสอบว่า **Router1 ติดต่อพอร์ต Serial ของ Router2 ได้ไหม** ไม่ใช่ทดสอบ PC ใน LAN 2

**จำไว้:** `interface ...` เลือกพอร์ต, `ip address ...` ตั้ง IP ให้พอร์ตที่เลือกอยู่ Router เชื่อมได้หลายเครือข่าย จึงมี IP แยกบนแต่ละ Interface ใน Lab นี้ Router แต่ละตัวใช้ 2 พอร์ต จึงมี 2 IP

---

## ส่วนที่ 4: EIGRP (ให้ Router รู้ทางไป LAN ฝั่งตรงข้าม)

### 17. ทำไมตั้ง IP แล้วยังต้องมี EIGRP?

Router รู้จักเฉพาะ Network ที่ **ต่ออยู่กับ Interface ตัวเองโดยตรง**

| Router  | Network ที่รู้จักโดยตรง                            |
| ------- | -------------------------------------------------- |
| Router1 | LAN 1 `192.168.1.0/26` และ WAN `192.168.1.128/26`  |
| Router2 | LAN 2 `192.168.1.64/26` และ WAN `192.168.1.128/26` |

Router1 ยังไม่รู้ทางไป LAN 2 และ Router2 ยังไม่รู้ทางกลับ LAN 1 EIGRP ช่วยให้ Router บอกกันเอง:

```text
Router2 → Router1 : "วง 192.168.1.64/26 ไปผ่านฉันได้"
Router1 → Router2 : "วง 192.168.1.0/26 ไปผ่านฉันได้"
```

**ต้องมีทั้งขาไปและขากลับ** เพราะ Ping ต้องส่งคำขอไปถึงปลายทางและรับคำตอบกลับมาด้วย

### 18. เปิด EIGRP บนทั้งสอง Router

ตั้งค่าเหมือนกันทั้งสองตัว

```cisco
Router(config)# router eigrp 1
Router(config-router)# network 192.168.1.0
Router(config-router)# no auto-summary
```

- `router eigrp 1` เข้าโหมดตั้งค่า EIGRP (Prompt เป็น `(config-router)#`) เลข `1` คือหมายเลข AS ต้อง **ตรงกันทั้งสอง Router**
- `no auto-summary` ปิดการสรุปเส้นทางอัตโนมัติตามขอบเขต Class เดิม ตอนนี้ให้รู้แค่หน้าที่ของคำสั่ง ยังไม่ต้องลงลึกเรื่อง Summarization

### 19. ทำไมใช้ `network 192.168.1.0` ได้ทั้งที่มีหลาย Subnet?

เมื่อเขียน `network` โดยไม่ใส่ Wildcard Mask EIGRP จะเลือก Interface ตามขอบเขต **Classful** ในที่นี้คือ Class C ของ `192.168.1.0` ซึ่งครอบคลุมทุก IP `192.168.1.x`

| Router  | Interface | IP            | ถูกเลือกไหม |
| ------- | --------- | ------------- | ----------- |
| Router1 | Gi0/0     | 192.168.1.62  | ใช่         |
| Router1 | S0/0/0    | 192.168.1.129 | ใช่         |
| Router2 | Gi0/0     | 192.168.1.126 | ใช่         |
| Router2 | S0/0/0    | 192.168.1.130 | ใช่         |

คำสั่งนี้ **ไม่ได้เปลี่ยน Mask เป็น /24** ทุกวงยังเป็น /26 เหมือนเดิม

**อย่าสับสนคำสั่ง `network` สองที่:**

| อยู่ในโหมด         | `network` ทำหน้าที่             |
| ------------------ | ------------------------------- |
| `(dhcp-config)#`   | ระบุ Subnet ที่ DHCP ใช้แจก IP  |
| `(config-router)#` | เลือก Interface ที่จะเปิด EIGRP |

---

## ส่วนที่ 5: ตรวจสอบผล

### 20. มี Neighbor แล้วหรือยัง

```cisco
Router1# show ip eigrp neighbors   → ควรเห็น 192.168.1.130 (Router2 ฝั่ง Serial)
Router2# show ip eigrp neighbors   → ควรเห็น 192.168.1.129 (Router1 ฝั่ง Serial)
```

Neighbor คือ Router ที่ EIGRP ติดต่อและสร้างความสัมพันธ์ด้วย ใน Lab นี้ทั้งสองพบกันผ่าน Serial Link

### 21. เรียนรู้ Network ฝั่งตรงข้ามแล้วหรือยัง

```cisco
Router1# show ip route
```

Routing Table คือตารางที่ Router ใช้เลือกเส้นทางส่ง Packet บน Router1 ควรเห็น:

| Network            | รู้ได้อย่างไร          | ส่งไปทาง             |
| ------------------ | ---------------------- | -------------------- |
| `192.168.1.0/26`   | ต่ออยู่โดยตรง (C)      | Gi0/0                |
| `192.168.1.128/26` | ต่ออยู่โดยตรง (C)      | Serial0/0/0          |
| `192.168.1.64/26`  | เรียนรู้ผ่าน EIGRP (D) | ผ่าน `192.168.1.130` |

### 22. ทดสอบ Ping

จาก Router1:

```cisco
Router1# ping 192.168.1.65
Router1# ping 192.168.1.66
```

จาก PC1 (อย่าลืมเช็ค `ipconfig` ก่อน):

```text
ping 192.168.1.65
ping 192.168.1.66
```

อันนี้ทดสอบ **จาก PC ข้าม Router ไปถึง PC อีก LAN**

### 23. Trace ดูเส้นทาง

ใน Packet Tracer PC ใช้ `tracert` (บน Router ใช้ `traceroute`)

```text
PC1> tracert 192.168.1.65
```

```text
PC1
 │ ปลายทางอยู่ต่าง Subnet → ส่งให้ Gateway
 ▼
192.168.1.62      Router1 ฝั่ง LAN 1
 │ Router1 รู้ทางไป LAN 2 ผ่าน Router2
 ▼
192.168.1.130     Router2 ฝั่ง Serial
 │ Router2 ต่อกับ LAN 2 โดยตรง
 ▼
192.168.1.65      PC3
```

---

## สรุป

### คำสั่งตรวจสอบ

| คำสั่ง                    | ตอบคำถามว่า                       |
| ------------------------- | --------------------------------- |
| `show ip eigrp neighbors` | พบ Router เพื่อนบ้านหรือยัง?      |
| `show ip route`           | รู้เส้นทางไปวงฝั่งตรงข้ามหรือยัง? |
| `ping`                    | ติดต่อปลายทางได้หรือไม่?          |
| `tracert` / `traceroute`  | ระหว่างทางผ่าน Router ไหนบ้าง?    |

### ภาพรวมการทำงาน

```text
PC ได้ Gateway จาก DHCP
        ↓
PC ส่งข้อมูลต่าง Subnet ให้ Gateway
        ↓
Router ดู Routing Table (ได้มาจาก EIGRP) แล้วเลือกทางส่งต่อ
```

**DHCP** แจกค่าตั้งค่าให้ PC ส่วน **EIGRP** แลกเปลี่ยนเส้นทางระหว่าง Router สองหน้าที่นี้แยกกันชัดเจน

### ลำดับการทำงานทั้ง Lab

```text
1. วางอุปกรณ์ + เพิ่ม Serial Module
2. เชื่อมสายให้ถูกประเภท
3. ตั้ง IP พอร์ต LAN ของ Router1 (.62) และ Router2 (.126)
4. ตั้ง DHCP Pool บน Router2 (+ excluded-address)
5. ตั้ง IP พอร์ต Serial (.129 / .130) + clock rate ฝั่ง DCE
6. เปิด EIGRP บนทั้งสอง Router
7. ตรวจ neighbors → route → ping → tracert
8. copy running-config startup-config
```
