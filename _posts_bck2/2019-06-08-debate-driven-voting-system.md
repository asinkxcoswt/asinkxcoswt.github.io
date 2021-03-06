---
layout: post
title:  "Debate-Driven Voting System - ระบบโหวตที่น่าจะแก้ปัญหาการเมืองไทยได้"
categories: Software Idea
image: /public/images/debate-driven-voting-system.jpg
---

จากการติดตามการเมืองไทย ผมคิดว่าระบบการโหวตปัจจุบันที่เราใช้ในสภามันมีปัญหา คือมันเป็นการโหวดที่ซึ่งผลลัพธ์ไม่สมเหตุสมผลกับข้ออภิปราย ไม่ว่าผู้ร่วมประชุมจะหยิบยกเหตุผลร้อยแปดใดๆ มาอภิปราย สุดท้ายผลโหวตกลับถูกลิขิตไว้แล้วจะกลไกภายนอก

ขออนุญาตไม่โจมตีวาระใดเป็นพิเศษ แต่ขอพูดโดยรวม เท่าที่ลองสังเกตดู ที่ผ่านมามันเป็นแบบนี้ และมันก็จะเป็นต่อไปหากไม่มีการแก้ไข ซึ่งระบบแบบนี้ที่จริงไม่ต้องมีการอภิปรายให้เสียเวลา คือต่างฝ่ายต่างไปล๊อบบี้กันนอกสภาแล้วพอเข้าสภาปุ๊บก็ vote กันไปเลยดีกว่า บางคนบอกว่าการอภิปรายมีเพื่อให้ประชาชนได้ฟัง แต่ประชาชนฟังไปก็ทำอะไรไม่ได้ กว่าจะได้ใช้สิทธิ์อีกทีก็อีกหลายปี หรือถ้ารอไม่ไหวก็ต้องไปใช้สิทธิ์บนถนน เพียงเพื่อให้มีรัฐประหารวนไป

ในฐานะที่เป็นโปรแกรมเมอร์ ผมก็เลยจินตนาการถึงระบบโหวตที่ควรจะเป็นในมุมมองของโปรแกรมเมอร์คนหนึ่ง ตั้งชื่อว่า Debate-Driven Voting System เป็นระบบที่ทำให้การอภิปรายในสภากลายเป็นกลไกหลักที่มีผลต่อผลการโหวต ซึ่งจะส่งผลให้สภากลายเป็นการต่อสู้กันด้วยหลักการและเหตุผลอย่างแท้จริง

ระบบนี้เป็นเพียงเวอร์ชั่นแบบง่ายๆ ตัดส่วนที่ซับซ้อนออกไปก่อน และเป็นเพียงไอเดียเอาไว้ทดลองต่อ ยังไม่แน่ใจว่าเทคโนโลยีปัจจุบันจะสามารถทำได้แค่ไหน

![debate-driven-voting-system](/public/images/debate-driven-voting-system.jpg)

## ระบบจะประกอบด้วยขั้นตอนด้งนี้

1. มีการประชุมเพื่อโหวตเลือก choice A หรือ B
2. ในช่วงแรก ผู้ร่วมประชุมอภิปรายเพื่อเสนอข้อสนับสนุนต่างๆ สำหรับ choice A หรือ B
   
   ในระหว่างการอภิปราย จะต้องมีระบบที่สามารถฟัง (Speech Recognition System) และสรุปใจความ (Content Summarization System) ข้อสนับสนุนต่างๆ และแสดงหัวข้อขึ้น dash board

3. ในช่วงที่สอง ผู้ร่วมประชุมอภิปรายเพื่อคัดค้านข้อสนับสนุนในขั้นตอนที่แล้ว
    
    dash board จะแสดงหัวข้อของข้อสนับสนุนทั้งหมด และ highligh ว่าผู้อภิปรายกำลังคัดค้านข้อใด
    
    ผลลัพธ์ของการอภิปรายคัดค้านนี้ คือการที่ระบบจะปรับค่าน้ำหนัก (quality weight) ให้แก่ข้อสนับสนุนแต่ละหัวข้อ โดยค่าน้ำหนักจะมีค่าอยู่ระหว่าง 0 ถึง 1 เช่น ข้อเสนอที่ถูกคัดค้านด้วยเหตุผลที่ชัดเจนที่สุดจะมีค่าน้ำหนักเท่ากับ 0 ส่วนข้อเสนอที่ไม่มีใครคัดค้านจะมีค่าน้ำหนักเท่ากับ 1 และข้อเสนอที่มีผู้คัดค้านแต่เหตุผลไม่ชัดเจนมากนัก อาจมีค่าน้ำหนัก 0.5 เป็นต้น
    
    โดยการที่จะทำเช่นนี้ได้จะต้องมีระบบตรวจสอบเหตุผล (Fallacy Detection System) ที่สามารถปรับค่าน้ำหนักโดยอิงจากคำอภิปรายได้โดยอัตโนมัต

4. ผู้ร่วมประชุมโหวตเลือก choice A หรือ B
5. ทำการตรวจสอบผลโหวตว่าสมเหตุสมผลกับการอภิปรายหรือไม่ 
   
   โดยนำผลรวมของค่าน้ำหนักของข้อสนับสนุนของแต่ละ choice มาเทียบกับผลโหวต เช่น choice A มีน้ำหนักสนับสนุน 11.5 ส่วน choice B มีน้ำหนักสนับสนุน 5.1 
    
    ถ้าคำนวนแล้วพบว่ามีผู้โหวตเลือก choice B มากกว่า choice A จะตีความได้ว่าผลโหวตไม่สมเหตุสมผล และจะต้องให้ที่ประชุมทำการอภิปรายเพิ่มเติมเพื่อปรับน้ำหนัก เช่นเดียวกับในขั้นตอนที่ 3.
    
    การที่มีผู้เลือก choice B มีมาก สวนทางกับค่าน้ำหนัก เป็นตัวบ่งชี้ว่าคนที่เลือก chioce B ยังทำหน้าที่ไม่เพียงพอในการอภิปรายสนับสนุน choice A และคัดค้าน choice B ระบบจึงต้องให้ผู้ร่วมประชุมอภิปรายเพิ่มเติม
6. ทำซ้ำข้อ 3. 4. 5. จนกว่าค่าน้ำหนักจะสอดคล้องกับผลโหวต
