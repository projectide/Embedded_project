### Embedded_project

1. เตรียมเครื่องมือ สำหรับเป็น Cross Compiler
```
$ cd ~
$ git clone https://github.com/raspberrypi/tools.git
$ cp -r tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian/ ~/
$ cd ~
$ mv gcc-linaro-arm-linux-gnueabihf-raspbian/ arm-linux-gnueabihf
```

จากนั้นทำการกำหนด PATH แบบถาวร ใน ~/.bashrc หรือ /etc/profile
```
$ echo "export "PATH=$PATH:~/arm-linux-gnueabihf/bin"" >> ~/.bashrc
```
ทดสอบ PATH ด้วยคำสั่ง (ถ้าไม่ได้ให้ปิด Terminal แล้วเปิดขึ้นมาใหม่)
```
$ arm-linux-gnueabihf-gcc -v
```
2. สร้างโฟลเดอร์สำหรับเก็บงาน
```
$ cd ~
$ mkdir project
```
3. จัดเตรียม wiringPi ในโฟลเดอร์ project
```
$ cd project
$ git clone git://git.drogon.net/wiringPi
```
4. ทำการ config wiringPi เพื่อให้ make ด้วย cross compiler ที่ได้เตรียมไว้ก่อนหน้านี้
```
$ cd wiringPi/wiringPi/
$ nano Makefile
		
		แก้ไขจาก “CC      = gcc” เป็น “CC      = arm-linux-gnueabihf-gcc”
		จากนั้น save และ exit
```
5. ทำการmake ซึ่งจะ compile โดยใช้ cross compiler ให้โดยอัตโนมัติ
```
$ make
```
   จะเห็นว่าได้ไฟล์ libwiringPi.so.x.xx ให้ทำการเปลี่ยนชื่อไฟล์  libwiringPi.so.x.xx เป็น  libwiringPi.so
```	
$ mv libwiringPi.so.x.xx  libwiringPi.so
```
6. เขียนโปรแกรมสำหรับ raspberry pi เพื่อควบควม ACT LED บนบอร์ดโดยเรียกใช้  libwiringPi
```
$ cd ~/project/
$ touch ACT-blink.c
$ nano ACT-blink.c
```
  จากนั้นพิมพ์โค้ดตัวอย่างดังนี้
  ```
	#include <stdio.h>
	#include <wiringPi.h> 
	const int ledPin = 47;
	int main(void){
	    	wiringPiSetupGpio(); 
    		pinMode(ledPin, OUTPUT);    
    	while(1)
    	{
            		digitalWrite(ledPin, HIGH); 
            		delay(1000);
            		digitalWrite(ledPin, LOW);
            		delay(1000);
    	}
    	return 0;
	}
```
จากนั้น save แล้ว compile  โดยใช้ cross compiler 
```
$  arm-linux-gnueabihf-gcc -o ACT-blink ACT-blink.c -I ~/project/wiringPi/wiringPi -L ~/project/wiringPi/wiringPi -lwiringPi
```
  จะได้ไฟล์ executable ชื่อว่า ACT-blink 

##### ต่อไปจะเป็นการสร้าง minimal Linux system สำหรับ Raspberry Pi 2 โดยใช้ Buildroot 

1. จัดเตรียม buildroot โดยใช้ git
```
$ cd ~/project/
$ git clone git://git.buildroot.net/buildroot
```

2. ทำการ config สำหรับ raspberry pi 2 
```
$ cd buildroot
$ make raspberrypi2_defconfig
```
3. ทำการ config สำหรับเลือก Toolchain เป็น Linaro ARM
```
$ make menuconfig
```
    หลังจากนั้นเลือก
```
	Toolchain
		Toolchain type (Buildroot toolchain) –-> เปลี่ยนเป็น Enternal toolchain
		Toolchain (Linaro ARM xxxx.xx)
```
  เสร็จแล้ว Save และ Exit ออกมาหน้า Terminal พิมพ์คำสั่ง
```
$ make
```
จากนั้นรอ (ถ้า make ครั้งแรกจะนานมากๆ เพราะต้องโหลดไฟล์ขนาดใหญ่)
4. จัดเตรียม SD Card 
```
$ sudo fdisk /dev/mmcblk0
		> p 		        #แสดงพาร์ติชันของ SD Card 
		> d 		        #ลบพาร์ติชันของ SD Card (หากมี 2 พาร์ติชัน ให้ใช้คำสั่ง d อีกครั้ง)
		> p 		        #แสดงพาร์ติชัน อีกครั้งเพื่อให้แน่ใจว่าพาร์ติชันทั้งหมดถูกลบแล้ว
		> n		          #สร้างพาทิชั่นใหม่สำหรับ boot
		   > p	        #เลือกชนิดพาร์ติชันป็น primary 
		   > 1	        #กำหนดให้เป็นพาร์ติชันแรก
		   > <enter>       
		   > +50M       #กำหนดขนาดพาร์ติชันเป็น 50MB

		> n		          #สร้างพาทิชั่นใหม่สำหรับ root
		   > p	        #เลือกชนิดพาร์ติชันป็น primary 
		   > 2	        #กำหนดให้เป็นพาร์ติชันที่สอง
		   > <enter>       
		   > <enter>

		> t           	#กำหนดชนิดของพาร์ติชัน
		      > 1       #เลือกพาร์ติชันที่หนึ่ง
		      > e       #พิมพ์ 'e' หมายถึง (FAT16)
		> a         	  #สร้างพาร์ติชันที่เป็น bootable
		     > 1        #เลือกพาร์ติชันที่หนึ่ง
		> p		          #ตรวจสอบความถูกต้อง
		> w		          #ยืนยันการจัดการพาร์ติชัน
```


ทำการฟอร์แมตพาร์ติชัน FAT16 และ EXT4
```
$ sudo mkfs.vfat -F16 -n BOOT /dev/mmcblk0p1
$ mkdir -p ~/mnt/boot
$ sudo mount /dev/mmcblk0p1 ~/mnt/boot

$ sudo mkfs.ext4 -L rootfs /dev/mmcblk0p2
$ mkdir -p ~/mnt/rootfs
$ sudo mount /dev/mmcblk0p2 ~/mnt/rootfs
```

5. คัดลอกไฟล์ที่ได้จากการ make ของ buildroot ไปยัง SD Card ที่เตรียมไว้
```
$ cd ~/project/buildroot
$ sudo cp output/images/rpi-firmware/* ~/mnt/boot
$ sudo cp output/images/*.dtb ~/mnt/boot
$ sudo ./output/host/usr/bin/mkknlimg output/images/zImage ~/mnt/boot/zImage
$ sudo tar xf ./output/images/rootfs.tar -C ~/mnt/rootfs
```
6. คัดลอกไฟล์โปรแกรม ACT-blink และ libwiringPi.so ไปยัง  rootfs
```
$ sudo cp  ~/project/wiringPi/wiringPi/libwiringPi.so ~/mnt/rootfs/usr/lib
$ sudo cp  ~/project/ACT-blink ~/mnt/rootfs/opt

$ sudo umount ~/mnt/*
```
7. boot raspberry pi แล้วทดลองรัน ACT-blink
```
$ cd /opt
$ ./ACT-blink
```
