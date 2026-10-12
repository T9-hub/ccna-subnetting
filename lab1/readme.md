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

ต่อไปเป็นการทำให้ Router 2 เเจกการตั้งค่าเครือข่ายให้ PC ใน LAN 2
เเละเราจะใช้ DHCP ในการแจก IP แบบอัติโนมัติ

LAN 1 เลือก IP เอง เเต่รอบนี้
PC ขอค่าการตั้งค่าเครือข่าย >> Router2 เเจกค่าให้ PC ได้ IP add sub, default gateway and dns server

                        เราจะเน้นตรง DHCP Pool

ก่อนสร้าง DHCP Pool ต้องตั้ง IP ให้ Router2 ฝั่ง LAN ก่อน
LAN 2 เราใช้

Network: 192.168.1.64 /26
IP ที่ใช้ได้: 192.168.1.65 – 192.168.1.126
Broadcast: 192.168.1.127

เราเลือกใช้ Last IP usable ให้กับ Router = .126

Router2# configure terminal
Router2(config)# interface fastEthernet 0/0
Router2(config-if)# no shutdown
Router2(config-if)# ip address 192.168.1.126 255.255.255.192

ตอนนี้พอร์ต LAN ของ Router2 มี IP แล้ว แต่ ยังไม่ได้สร้างชุดข้อมูลสำหรับแจกให้ PC ขั้นต่อไปจึงเป็น DHCP Pool

DHCP Pool คืออะไร?

      ให้มองว่าเป็น ชุดการตั้งค่าที่ Router ใช้แจกให้ PC ในวงที่กำหนด
      ภายใน Pool เราบอก Router ว่า:
      DHCP Pool: NewYork
      |
      ├── แจก IP จาก Subnet ไหน?
      ├── บอก PC ให้ใช้ Gateway อะไร?
      └── บอก PC ให้ใช้ DNS Server อะไร?

Router2(config-if)# exit
Router2(config)# ip dhcp pool NewYork
Router2(dhcp-config)#

Prompt เปลี่ยนเป็น:

            Router2(dhcp-config)#
            แปลว่า คำสั่งถัดไปกำลังตั้งค่าภายใน DHCP Pool ที่เลือกอยู่


            Pool ว่าจะใช้ Subnet ไหนแจก IP
                  Router2(dhcp-config)# network 192.168.1.64 /26

                        Pool นี้ใช้ Subnet 192.168.1.64/26 สำหรับแจก IP ให้เครื่องลูกข่าย

                        เขียนด้วย Subnet Mask แบบเต็มก็ได้:

                        Router2(dhcp-config)# network 192.168.1.64 255.255.255.192

                  Router จึงรู้ขอบเขตของวง ว่า IP ไหนใช้ได้อันไหนใช้ไม่ได้



            บอก PC ว่า Default Gateway คือใคร
                  Router2(dhcp-config)# default-router 192.168.1.126
                  ความหมายคือ:
                  เมื่อแจกค่าให้ PC ให้บอก PC ด้วยว่า Default Gateway ของมันคือ 192.168.1.126

PC ใน LAN 2
|
├── ติดต่อเครื่องใน Subnet เดียวกัน → ส่งภายใน LAN
|
└── ติดต่อ IP ต่าง Subnet → ส่งไป Default Gateway
192.168.1.126

            บอก PC ว่า DNS Server คือใคร

                  Router2(dhcp-config)# dns-server 192.168.1.126

Excluded Address: กัน IP ที่ไม่ต้องการให้ DHCP แจก
David ใช้ .126 ให้ Router ไปแล้ว จึงระบุว่าไม่ให้แจก IP นี้แก่ PC
ออกจาก Pool ก่อน:
Router2(dhcp-config)# exit
Router2(config)# ip dhcp excluded-address 192.168.1.126

ดูสิ่งที่ตั้งไว้ด้วย do show run
ขณะยังอยู่ใน Pool: Router2(dhcp-config)# do show run

ทดสอบว่า LAN 2 ใช้งานได้
จาก PC4:
ping 192.168.1.65

ตรวจว่า PC4 ติดต่อ PC3 ได้หรือไม่
ping 192.168.1.126
ตรวจว่า PC4 ติดต่อพอร์ต LAN ของ Router2 ซึ่งเป็น Gateway ได้หรือไม่

. บันทึก Configuration
Transcript บอกว่าเขาบันทึก Router ทั้งสองตัว ตัวอย่างคำสั่งเต็มคือ:
Router2(config)# end
Router2# copy running-config startup-config
ความหมายคือบันทึก Configuration ที่กำลังใช้อยู่ ให้ Router นำกลับมาใช้เมื่อเริ่มทำงานใหม่

จากนั้นตั้ง IP ให้ WAN ระหว่าง Router

(ตอนที่เรากำหนด IP ให้กับ router ที่เป็น last usable
เป็นพอร์ตฝั่ง LAN สำคัญมาก)

But now เรากำลังตั้ง พอร์ต serial ที่ต่อระหว่าง Router
ใช้ Subnet ที่สาม:
192.168.1.128/26

Router1 มันคือสายที่เชื่อมกันใน WAN Router2
192.168.1.129 ================= 192.168.1.130
Serial Link

ตอนนั้นเราเลือกพอร์ต Gig แล้วตั้ง .126
interface Gig 0/0

      ip address 192.168.1.126 255.255.255.192

ความหมายคือ:
ให้พอร์ต FastEthernet0/0 ใช้ IP .126 สำหรับเชื่อมกับ LAN 2

ครั้งนี้ เราเลือกอีกพอร์ต คือ Serial แล้วตั้ง .130
interface serial 2/0

ip address 192.168.1.130 255.255.255.192

ความหมายคือ:
ให้พอร์ต Serial2/0 ใช้ IP .130 สำหรับเชื่อมกับ Router1

ตอนนี้ พอร์ต Serial ทั้งสองฝั่งอยู่ในวงเดียวกัน:

WAN: 192.168.1.128/26 (subnet ที่ 3 , .0 (ใช้กับ LAN 1 ไปแล้ว), .64 (ใช้กับ LAN2) , .128(ใช้กับ Router) , .192 (ยังว่างให้ใช้) )

├── Router1 Serial: 192.168.1.129
└── Router2 Serial: 192.168.1.130

เมื่อ Router1 สั่ง:
ping 192.168.1.130
จึงกำลังทดสอบว่า ติดต่อพอร์ต Serial ของ Router2 ได้หรือยัง ไม่ใช่ทดสอบ PC ฝั่ง LAN 2 ครับ

คำสั่ง interface ... เลือกพอร์ต ส่วน ip address ... ตั้ง IP ให้พอร์ตที่เลือกอยู่

จำเอาไว้
Router เชื่อมหลาย Network ได้ จึงมี IP บนแต่ละ Interface ที่ใช้เชื่อม Network เหล่านั้น ใน Lab นี้ Router แต่ละตัวใช้สองพอร์ต จึงตั้งสอง IP ครับ

ขั้นต่อไปคือให้ Router รู้ทางไป LAN ฝั่งตรงข้าม ซึ่งเป็นช่วง EIGRP
Router แต่ละตัวต้องรู้ว่า จะส่งข้อมูลไป LAN ฝั่งตรงข้ามทางไหน
เปิด EIGRP เพื่อให้ Router1 และ Router2 แลกเปลี่ยนข้อมูลเส้นทาง แล้วตรวจด้วย show และทดสอบด้วย Ping ครับ

Router รู้จัก Network ที่ต่อกับ Interface ของตัวเองโดยตรงอยู่แล้ว
ทำไมตั้ง IP แล้ว ยังต้องมี EIGRP?

Router รู้จัก Network ที่ต่อกับ Interface ของตัวเองโดยตรงอยู่แล้ว
ก่อนแลกเปลี่ยนเส้นทาง:

      Router	Network ที่รู้จักโดยตรง
      Router1	LAN 1: 192.168.1.0/26 และ WAN: 192.168.1.128/26
      Router2	LAN 2: 192.168.1.64/26 และ WAN: 192.168.1.128/26

      แต่ Router1 ยังต้องรู้ทางไป LAN 2 และ Router2 ยังต้องรู้ทางกลับไป LAN 1

EIGRP ช่วยให้เกิดการเรียนรู้แบบนี้:
Router2 บอก Router1: ก็คือบอก R1 ว่าให้เข้ามารู้จัก .64 ได้
“วง 192.168.1.64/26 ไปผ่านฉันได้”

      Router1 บอก Router2: ก็คือบอก R2 ว่าให้เข้ามารู้จัก  .0 ได้
      “วง 192.168.1.0/26 ไปผ่านฉันได้”

ต้องมีทางไปและทางกลับ เพราะ Ping ต้องส่งคำขอไปถึงปลายทาง และได้รับคำตอบกลับมาด้วย

เปิด EIGRP บนทั้งสอง Router
ตัวอย่างคำสั่ง:
Router1# configure terminal
Router1(config)# router eigrp 1
Router1(config-router)# network 192.168.1.0
Router1(config-router)# no auto-summary
Router2 ตั้งแบบเดียวกัน:
Router2# configure terminal
Router2(config)# router eigrp 1
Router2(config-router)# network 192.168.1.0
Router2(config-router)# no auto-summary

                  เลข 1 เป็นตัวอย่าง

            router eigrp 1
            คือเข้าโหมดตั้งค่า EIGRP สังเกต Prompt:

                                    (config-router)#

ทำไมใช้ network 192.168.1.0 ทั้งที่มีหลาย Subnet?
เพราะ Classful network
คำสั่งที่เขาใช้:
network 192.168.1.0
เมื่อเขียนแบบนี้โดยไม่ระบุ Wildcard Mask จะเลือก Interface ตามขอบเขต Class เดิม ในกรณีนี้คือ Class C ของ 192.168.1.0 ซึ่งครอบคลุม IP 192.168.1.x

ลองดู IP ของ

      Router1:
      FastEthernet0/0:  192.168.1.62   → อยู่ในช่วงที่เลือก
      Serial2/0:        192.168.1.129  → อยู่ในช่วงที่เลือก

      Router2:
      FastEthernet0/0: 192.168.1.126   → อยู่ในช่วงที่เลือก
      Serial2/0:       192.168.1.130   → อยู่ในช่วงที่เลือก

จึงเปิด EIGRP บน Interface เหล่านี้ และนำ Network ที่เกี่ยวข้องไปประกาศให้เพื่อนบ้านเรียนรู้
คำสั่งนี้ไม่ได้เปลี่ยน Subnet Mask ของ Interface เป็น /24
วงที่เราแบ่งไว้ยังคงเป็น:

      LAN 1 → /26
      WAN   → /26
      LAN 2 → /26

แยกให้ออกว่า network อยู่ในโหมดไหน:

      ตำแหน่ง	            ความหมายของ network
      (dhcp-config)#	      ระบุ Subnet ที่ DHCP ใช้แจก IP
      (config-router)#        ของ EIGRP เลือก Interface
      ที่จะเปิดใช้งานEIGRP

no auto-summary ทำอะไร?
Router(config-router)# no auto-summary

ใช้เพื่อ ปิดการสรุปเส้นทางอัตโนมัติตามขอบเขต Class เดิม
ตอนนี้ให้รู้หน้าที่ของคำสั่งก่อน ส่วนวิธี Summarization ทำงานอย่างไร ยังไม่ต้องลงรายละเอียดใน Lab นี้

ตรวจว่า Router เป็นเพื่อนบ้าน EIGRP กันแล้วหรือยัง

Router1(config-router)# end
Router1# show ip eigrp neighbors

บน Router1 ควรเห็นเพื่อนบ้าน:
192.168.1.130 → Router2 ฝั่ง Serial

บน Router2:
Router2# show ip eigrp neighbors
ควรเห็น:
192.168.1.129 → Router1 ฝั่ง Serial

Neighbor หมายถึง Router ที่ EIGRP ติดต่อและสร้างความสัมพันธ์ด้วย ใน Lab นี้ทั้งสองตัวพบกันผ่าน Serial Link

ตรวจว่าเรียนรู้ Network ฝั่งตรงข้ามแล้วหรือยัง
Router1# show ip route
คำสั่งนี้แสดง Routing Table —
ตารางที่ Router ใช้เลือกเส้นทางส่ง Packet

บน Router1 เห็น:
Network ปลายทาง รู้จักได้อย่างไร ส่งไปทางไหน
192.168.1.0/26 ต่ออยู่โดยตรง Gig 0/0
192.168.1.128/26 ต่ออยู่โดยตรง Serial 2/0
192.168.1.64/26 เรียนรู้ผ่าน EIGRP ผ่าน 192.168.1.130

ทดสอบ Ping ไป PC ฝั่ง LAN 2
จาก Router1:
Router1# ping 192.168.1.65
Router1# ping 192.168.1.66

ipconfig อย่าลืมเช็ค

จากนั้น David ทดสอบจาก PC1 ไปยัง PC3 และ PC4:
ping 192.168.1.65
ping 192.168.1.66
คราวนี้เป็นการทดสอบ จาก PC ข้าม Router ไปถึง PC อีก LAN

Trace ดูว่าผ่านใครบ้าง
David Trace จาก PC1 ไป 192.168.1.65 แล้วเห็น:
PC1
|
| ปลายทางอยู่ต่าง Subnet → ส่งให้ Gateway
v
192.168.1.62 Router1 ฝั่ง LAN 1
|
| Router1 รู้ทางไป LAN 2 ผ่าน Router2
v
192.168.1.130 Router2 ฝั่ง Serial
|
| Router2 ต่อกับ LAN 2 โดยตรง
v
192.168.1.65 PC3

ใน Packet Tracer PC ใช้:

tracert 192.168.1.65

คำสั่ง ช่วยตรวจอะไร
ping ปลายทางตอบกลับได้หรือไม่
tracert / traceroute (มีหน้าที่ในการตรวจสอบเส้นทางเหมือนกัน)
ระหว่างทางผ่าน Router ไหนบ้าง

สรุปคำสั่ง
show ip eigrp neighbors → พบ Router เพื่อนบ้านหรือยัง?
show ip route → รู้เส้นทางไปวงฝั่งตรงข้ามหรือยัง?
ping → ติดต่อปลายทางได้หรือยัง?
tracert / traceroute → ผ่าน Router ไหนบ้าง?

จำเอาไว้
DHCP แจกค่าการตั้งค่าให้ PC ส่วน EIGRP แลกเปลี่ยนเส้นทางระหว่าง Router
PC รู้ Gateway จาก DHCP
↓
PC ส่งข้อมูลต่าง Subnet ให้ Gateway
↓
Router ใช้ Routing Table เลือกว่าจะส่งต่อทางไหน
