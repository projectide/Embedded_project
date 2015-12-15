### Embedded_project

1 เตรียมเครื่องมือ สำหรับเป็น Cross Compiler

```sh
$ cd ~
$ git clone https://github.com/raspberrypi/tools.git
$ cp -r tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian/ ~/
$ cd ~
$ mv gcc-linaro-arm-linux-gnueabihf-raspbian/ arm-linux-gnueabihf
```

จากนั้นทำการกำหนด PATH แบบถาวร ใน ~/.bashrc หรือ /etc/profile

```sh
$ echo "export "PATH=$PATH:~/arm-linux-gnueabihf/bin"" >> ~/.bashrc
```
หรือกำหนด PATH แบบชั่วคราว
```sh
$ export "PATH=$PATH:~/arm-linux-gnueabihf/bin"
```
ทดสอบ PATH ด้วยคำสั่ง (ถ้าไม่ได้ให้ปิด Terminal แล้วเปิดขึ้นมาใหม่)

```sh
$ arm-linux-gnueabihf-gcc -v
```

2 สร้างโฟลเดอร์สำหรับเก็บงาน

```sh
$ cd ~
$ mkdir project
```

3 จัดเตรียม wiringPi ในโฟลเดอร์ project
```sh
$ cd project
$ git clone git://git.drogon.net/wiringPi
```
4 ทำการ config wiringPi เพื่อให้ make ด้วย cross compiler ที่ได้เตรียมไว้ก่อนหน้านี้
```sh
$ cd wiringPi/wiringPi/
$ nano Makefile
		
		แก้ไขจาก “CC      = gcc” เป็น “CC      = arm-linux-gnueabihf-gcc”
		จากนั้น save และ exit
```
5 ทำการ make ซึ่งจะ compile โดยใช้ cross compiler ให้โดยอัตโนมัติ
```sh
$ make
```
จะเห็นว่าได้ไฟล์ libwiringPi.so.x.xx ให้ทำการเปลี่ยนชื่อไฟล์  libwiringPi.so.x.xx เป็น  libwiringPi.so
```sh
$ mv libwiringPi.so.x.xx  libwiringPi.so
```
6 เขียนโปรแกรมสำหรับ raspberry pi เพื่อควบควม ACT LED บนบอร์ดโดยเรียกใช้  libwiringPi
```sh
$ cd ~/project/
$ touch ACT-blink.c
$ nano ACT-blink.c
```
จากนั้นพิมพ์โค้ดตัวอย่างดังนี้ (หรือแก้ไขโค้ดตามที่ต้องการ)
```C
	#include <stdio.h>
	#include <wiringPi.h> 
	
	const int ledPin = 47;   //pin ACT LED
	
	int main(void){

	wiringPiSetupGpio(); 
	pinMode(ledPin, OUTPUT);
        
    	while(1)
    	{
            		digitalWrite(ledPin, HIGH); 
            		delay(1000);			//delay time ~ millisecond
            		digitalWrite(ledPin, LOW);
            		delay(1000);
    	}
    	return 0;
	}
```
จากนั้น save แล้ว compile  โดยใช้ cross compiler 
```sh
$  arm-linux-gnueabihf-gcc -o ACT-blink ACT-blink.c -I ~/project/wiringPi/wiringPi -L ~/project/wiringPi/wiringPi -lwiringPi
```
จะได้ไฟล์ executable ชื่อว่า ACT-blink 

##### ต่อไปจะเป็นการสร้าง minimal Linux system สำหรับ Raspberry Pi 2 โดยใช้ Buildroot 

1 จัดเตรียม buildroot โดยใช้ git
```sh
$ cd ~/project/
$ git clone git://git.buildroot.net/buildroot
```

2 ทำการ config สำหรับ raspberry pi 2 
```sh
$ cd buildroot
$ make raspberrypi2_defconfig
```
3 ทำการ config สำหรับเลือก Toolchain เป็น Linaro ARM
```sh
$ make menuconfig
```
หลังจากนั้นเลือก
```sh
	Toolchain
		Toolchain type (Buildroot toolchain) –-> เปลี่ยนเป็น Enternal toolchain
		Toolchain (Linaro ARM xxxx.xx)
```
เสร็จแล้ว Save และ Exit ออกมาหน้า Terminal พิมพ์คำสั่ง
```sh
$ make
```
จากนั้นรอ (ถ้า make ครั้งแรกจะนานมากๆ เพราะต้องโหลดไฟล์ขนาดใหญ่)

4 จัดเตรียม SD Card 

```
$ sudo fdisk /dev/mmcblk0
		> p		#แสดงพาร์ติชันของ SD Card 
		> d		#ลบพาร์ติชันของ SD Card (หากมี 2 พาร์ติชัน ให้ใช้คำสั่ง d อีกครั้ง)
		> p		#แสดงพาร์ติชัน อีกครั้งเพื่อให้แน่ใจว่าพาร์ติชันทั้งหมดถูกลบแล้ว
		> n		#สร้างพาทิชั่นใหม่สำหรับ boot
		   > p		#เลือกชนิดพาร์ติชันป็น primary 
		   > 1		#กำหนดให้เป็นพาร์ติชันแรก
		   > <enter>       
		   > +50M	#กำหนดขนาดพาร์ติชันเป็น 50MB

		> n		#สร้างพาทิชั่นใหม่สำหรับ root
		   > p		#เลือกชนิดพาร์ติชันป็น primary 
		   > 2		#กำหนดให้เป็นพาร์ติชันที่สอง
		   > <enter>       
		   > <enter>	#ใช้พื้นที่ ที่เหลือทั้งหมด

		> t		#กำหนดชนิดของพาร์ติชัน
		      > 1	#เลือกพาร์ติชันที่หนึ่ง
		      > e	#พิมพ์ 'e' หมายถึง (FAT16)
		> a		#สร้างพาร์ติชันที่เป็น bootable
		     > 1	#เลือกพาร์ติชันที่หนึ่ง
		> p		#ตรวจสอบความถูกต้อง
		> w		#ยืนยันการจัดการพาร์ติชัน
```


ทำการฟอร์แมตพาร์ติชัน FAT16 และ EXT4
```sh
$ sudo mkfs.vfat -F16 -n BOOT /dev/mmcblk0p1
$ mkdir -p ~/mnt/boot
$ sudo mount /dev/mmcblk0p1 ~/mnt/boot

$ sudo mkfs.ext4 -L rootfs /dev/mmcblk0p2
$ mkdir -p ~/mnt/rootfs
$ sudo mount /dev/mmcblk0p2 ~/mnt/rootfs
```

5 คัดลอกไฟล์ที่ได้จากการ make ของ buildroot ไปยัง SD Card ที่เตรียมไว้

```sh
$ cd ~/project/buildroot
$ sudo cp output/images/rpi-firmware/* ~/mnt/boot
$ sudo cp output/images/*.dtb ~/mnt/boot
$ sudo ./output/host/usr/bin/mkknlimg output/images/zImage ~/mnt/boot/zImage
$ sudo tar xf ./output/images/rootfs.tar -C ~/mnt/rootfs
```

6 คัดลอกไฟล์โปรแกรม ACT-blink และ libwiringPi.so ไปยัง  rootfs

```sh
$ sudo cp  ~/project/wiringPi/wiringPi/libwiringPi.so ~/mnt/rootfs/usr/lib
$ sudo cp  ~/project/ACT-blink ~/mnt/rootfs/opt

$ sudo umount ~/mnt/*
```

7 boot raspberry pi แล้วทดลองรัน ACT-blink

```sh
$ cd /opt
$ ./ACT-blink
```
