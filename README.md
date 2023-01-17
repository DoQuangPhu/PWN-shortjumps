# PWN-shortjumps
Buớc 1: basic file check
![image](https://user-images.githubusercontent.com/93699926/212887319-3577cc60-4700-472b-9677-081fcaf47a3b.png)

  -như chúng ta thấy thì đây là một file 32bit-kiến trúc little endian,các chế độ bảo vệ như canary và pie đều được tắt(chúng ta sẽ lợi dụng việc này để khai thác chương trình)
 Bước 2: reversing
 lúc giải bài này thì ban tổ chức chưa update file source code nên mình đã vứt nó vô ghidra để xem , và thấy có 3 phần khá thú vị :
  func1: main
  ![image](https://user-images.githubusercontent.com/93699926/212888704-f5e69664-8ea4-448e-97d8-c408340028f0.png)
  hàm main chỉ đơn giản là nó sẽ chỉ in ra màn hình câu hỏi "Hi whaat your name ","do tou have any dream?","tell me your dream:" sau đó nó sẽ scan in phần input của chúng ta.
  nhìn qua thì chúng ta có thể dễ dàng phát hiện đây là lỗi buffer overflow,khi mà local_80 chỉ được cấp 80 bytes nhưng khi chương trình scan in input của chúng ta lên đến 140 .
  func2: jmp2
  ![image](https://user-images.githubusercontent.com/93699926/212890562-3b42f45c-4fc8-4482-8446-ed6cfda22c96.png)

  hàm jmp2 ko được gọi trong hàm main nhưng lại vô cùng powerful và chúng ta sẽ lợidụng nó để có thể lấy được shell vì trong hàm jmp2 có gọi trực tiếp hàm system("/bin/sh")
  string "/bin/sh" chính ta local_14và local_10
  +)sau khi xem qua đoạn code asm của jmp2 thì chúng ta cần đáp ứng việc parameter1=0xcafebabe và parameter2+parameter1=0x13371337
  vì 0xcafebabe>0x13371337 nên chúng ta sẽ làm chúng ta sẽ làm parameter2+parameter1=0x113371337 => parameter2=0x48385879
  
  +)việc thứ hai cần làm là đưa biến jmp=1 chúng ta sẽ lợi dụng hàm jmp1
  func3: jmp1
    ![image](https://user-images.githubusercontent.com/93699926/212893624-4d4c1fcb-2b99-423e-8175-476766037c98.png)
  để lợi dụng jmp1 để đưa biến jmp=1 chúng ta cần chuyền tham số cho jmp1 là parameter1=0xdeadbeef
  
   Bước3: exploit
    ý tưởng của mình là chúng ta sẽ lợi dụng buffer overflow của hàm main để nhảy đến lần lượt các hàm jmp1 và jmp2
    vì chế độ bảo vệ pie bị tắt nên chúng ta có thể lấy được đia chỉ của hàm jmp1 jmp2 và main
    jmp1_address=0x080492b4
    jmp2_address=0x080492e0
    main_address=0x08049378
    payload1 mình sẽ dùng dể nhảy đến jmp1 làm biến jmp=1 và quay lại main.
    payload2 mình sẽ nhảy dến jmp2 và gọi hàm system.
    put it all together ta có :
    ![image](https://user-images.githubusercontent.com/93699926/212895883-d0048c7b-cba9-493d-9996-98e5bdeac7a7.png)
chạy chương trình và ta có được shell :
  ![image](https://user-images.githubusercontent.com/93699926/212896199-60ee3f57-0e6c-400b-ad96-03ba1f268a66.png)

flag :KCSC{jump1ng_1s_1nt3r3st1ng_1snt_1t?_7dc4c818f3}
