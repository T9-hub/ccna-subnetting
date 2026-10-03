LAB CCNA SUBNETTING

ฺฺฺBEFORE START 🔴
interface ของ Router ถูก shutdown อยู่
เราจะเริ่ม เปิด no shut

    : enable
    : configure terminal

    จากนั้นเข้า interface ที่เราเสียบสายไว้ (เสียบขาเข้า)

    : interface fast or gig 0/0/0
    : no shut

สถานะควรจะเป็น

    : %LINK-5-CHANGED: Interface GigabitEthernet0/0, changed state to up
    : %LINEPROTO-5-UPDOWN: Line protocol on Interface GigabitEthernet0/0, changed state to up

    รอสักครู่ แล้วสายควรเปลี่ยนเป็นเขียว 🟢


    ถ้าเกิดเราเข้าสายผิด เเละ no shut ไปสายจะไม่เขียวเเละ
    %LINK-3-UPDOWN: Interface GigabitEthernet0/1, changed state to down

เช็คสถานะ

    ออกมาจาก config mode

    : show ip interface brief
        คำสั่งนี้จะแสดง พอร์ต (Interface) ทั้งหมดที่มีอยู่บนตัว Router

    ดู Gi0/0

    ถ้าเห็น: GigabitEthernet0/0 unassigned YES unset up up

    ถือว่าตอนนี้ Port เปิดและ Link switch เข้ากับ Router แล้ว

    up / up
    │     │
    │     └── Protocol ขึ้น
    └──────── Interface ขึ้น
