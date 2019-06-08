# TDD is more of architecture design and requirement analysis than just let individual developers writes 'as much as possible' tests.
# kkp has problem in requirement anaslysis, i.e., the transaltion from requirement to code.

# a proper tdd should not slowdowns the development cycles, it should be another way around.

# is kkp has tests - yes of course
# what is the tests look like in kkp?
# what is the problem about these test and the process of writing test.
# do we benefit for these tests, honestly!
# we can be safe and employ some best practice on writing tests, but have we every question what benefit do we really get out of them?
# In this article, I explore some best practice and suggestion out there and see what would it like when appling to kkp40 situation.
# then I will conclude with my own thought about what should we do with the test for kkp40 project.

# mandatory tests vs optional tests
# architecture suitable for writing tests.


# What's kkp lack of? Design for testing.
# Well defined interfaces of a layer


--------------------------

สิ่งที่ผมทำได้ตอนนี้ มีแค่การเขียนตามใจ ในบล๊อกส่วนตัว เน้นใส่ความเห็นส่วนตัว แบบที่ไม่ต้องคำนึงถึงความถูกต้องและ reference มากนัก

ถ้าผมจะเขียนเป็น professional คงจะใส่ unproved opinion ความเห็นส่วนตัวลงไปมากไม่ได้ 

ผมลองแล้วรู้สึกว่าตัวเองตื้นเขินเกินไป 

อยากจะขอเวลาศึกษาพื้นฐานของสิ่งที่ควรจะรู้ให้มากกว่านี้ เช่น TDD, DDD

อยากจะลอง 

--------------------------------

พี่โป้งครับ หลังจากที่ได้คุยกับพี่โป้งวันอังคารที่แล้ว ผมได้ลองไปคิดหาหัวข้อที่จะเอามาเขียน -- ผมคิดว่าในฐานะ technology evangelist ที่ต้อง coach คนอื่น -- สิ่งที่เขียน ควรจะต้องมี proof หรือ reference พอสมควร ไม่ควรเป็นไอเดียลมๆ แล้งที่ยังไม่ได้พิสูจน์ ผมพยายามจะหาหนังสือมาอ่านเพื่อทดแทนประสบการณ์ที่ขาดไป แต่ยิ่งอ่านก็ยิ่งรู้ว่าตัวเองมันช่างตื้นเขิน ด้วยความสามารถของผมในปัจจุบัน การเขียนบทความเพื่อ coaching มันจะเต็มไปด้วย ignorance ที่ไม่มีวันถมเต็ม มันจะได้แค่บทความตื้นๆ ที่รีบๆ เขียนให้ได้ตาม commitment และไม่ได้มีอะไรมากไปกว่าบทความอื่นๆ ที่หาอ่านกันได้เองอยู่แล้ว

ผมเลยมาลองนั่งทบทวน ผมพบว่าสิ่งที่ผมอยากทำและน่าจะทำได้ดีกว่า คือการเขียนเพื่อหาคำตอบให้กับปัญหาที่ตัวเองพบเจอ ไม่ได้เป็นการเขียนสอนคนอื่น แต่เพื่อแชร์ให้คนอื่นเห็นว่าผมกำลังเจอปัญหาอะไร ผมคิดยังไงกับมัน และกำลังจะทำอะไรเพื่อแก้ปัญหา และหลังจากทำแล้วเวิร์คหรือไม่อย่างไร

อีกอย่างคือ ผมคิดว่าตัวเองยังอยากที่จะ coding อยู่ครับ และคิดว่ามันเป็นสิ่งจำเป็นเพื่อที่จะสามารถไปอยู่ในตำแหน่งที่ coach น้อง dev คนอื่นได้ ผมคิดว่าจำเป็นจะต้องแก้ปัญหาหลายๆ อย่างของตัวเองให้ได้ก่อน การลงมือ dev จะได้เจอปัญหาจริง และได้ลงมือทดลองเองก่อน

สำหรับปัญหาหลายๆ อย่างที่พบเจอกับโปรเจค kkp ในฐานะ dev -- ไม่ว่าจะเป็นในเรื่องของ hard skill ที่เกี่ยวกับการ design software, development process หรือ soft skill ที่เกี่ยวกับการวางแผนและประเมินงาน และการจัดการความเครียด -- พอพ้นจากอาการเมาเรือแล้ว ผมกลับมีความรู้สึกว่าอยากจะเจอปัญหาแบบนี้อีกครั้ง เพื่อที่จะได้ทำให้มันดีกว่าเดิม 

ดังนั้น ถ้าพี่โป้งโอเค ผมอยากจะขอโอกาสอีกครั้งในตำแหน่งเดิม (software engineer) แต่จะเน้นตัวเองไปในส่วนของการ design และ planning ให้มากขึ้น และลดในส่วนของ detailed implementation ให้น้อยลง และเอาเวลามาเขียนบทความและหาวิธีแก้ปัญหาต่างๆ เพิ่มขึ้น ดีไหมครับ โดยในช่วงนี้ผมคงจะช่วย   project kkp แบบที่ทำอยู่ไปก่อน เอาไว้งานซาลงแล้ว พี่โป้งอาจจะ release น้องไป project อื่น เหลือแค่ผมซัพพอร์ต kkp ต่อ พลางทำ parallel โปรเจคอื่นไปด้วย ดีไหมครับ

ส่วนไอเดียคร่าวๆ สำหรับสิ่งที่ผมอยากจะ achieve ให้ได้ (และจะเขียนเป็นบทความ) ตอนนี้มีดังนี้ครับ

1. software design ที่ visible สำหรับทุกคนในทีม
    การที่ทุกคนในทีมรวมทั้ง PM, BA, SA และ DEV เข้าใจโครงสร้างของ software จะทำให้ทุกคนสามารถมีส่วนร่วมในการประเมินได้ว่าจะต้อง change ตรงไหน อย่างไร ใช้เวลาเท่าไหร่ และมี impact กับระบบแค่ไหน หรือเป็นการบิด design โดยไม่ควรหรือไม่

    ปัญหาที่พบเจอใน project kkp การมี design ที่ไม่ชัดเจน ทำให้มีแต่เฉพาะ developer ที่คุ้นเคยกับ code เท่านั้นที่จะเข้าใจโครงสร้างและประเมินได้ ส่งผลให้
    
    - ความเครียดและความรับผิดชอบตกไปอยู่กับ developer มากเกินไป 
    - คนที่คุยกับลูกค้าเพื่อรับ requirement มีความลำบากในการประเมินว่าสิ่งใดทำได้ หรือไม่ได้ หรือใช้เวลาแค่ไหน
    - การวางแผนก่อนเริ่ม implement มักจะขาดตกเสมอ เนื่องจากเราไม่เห็นภาพที่ชัดเจน ทำให้มีงานแทรถโผล่ขึ้นมาอยู่เสมอในระหว่าง Implement และทำให้ project delay

2. การทำ code design ให้ testable and extensible และง่ายต่อการแบ่งงานให้ developer หลายๆ คนช่วยกันทำอย่างมีประสิทธิภาพและมี quality
    - ใน project kkp -- เราใช้ policy ว่า ให้ developer "เขียนเทสให้มากที่สุดเท่าที่ทำได้ในกรอบเวลาที่มี" ทำให้ test ถูกเขียนอย่างสะเปะสะปะ -- บางจุดเขียน test ยาก ใช้เวลานาน แต่ไม่ค่อยมีประโยชน์ -- บางจุดที่น่าจะต้องมีเทสกลับไม่ได้เขียนเท่าที่ควร
    - หาวิธีที่เหมาะสมในการนำ TDD มาใช้กับ spring boot project (with Domain Drive Design) ได้อย่างไร โดยที่ไม่ทำให้ project ล่าช้า
  
3. บริษัทอื่นๆ เขามี software development process อย่างไรกันบ้าง (Google, Netflix, Agoda, etc ...) และถ้าเปรียบเทียบกับ project kkp -- มีอะไรบ้างที่ project kkp ขาดไป

ปล. เรื่องจำนวนในการเขียนบทความ อยากจะขออนุญาต ตั้งเป็นเดือนละบทความเป็นอย่างน้อย ก่อนได้ไหมครับ เพราะช่วงแรกๆ น่าจะต้องอ่านหนังสือเยอะ และทำ POC หลายๆ อย่าง 