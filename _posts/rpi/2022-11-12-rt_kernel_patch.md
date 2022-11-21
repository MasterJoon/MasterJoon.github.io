---
title:  "Raspberry Pi OS RT kernel patch"
categories:
  - rpi
---
Raspberry Pi에 설치할 수 있는 OS는 많지만, 공식적으로 제공되고 있는 OS 중에서 가장 대표적인 Raspberry Pi OS를 GUI 가 없는 Lite 버전으로 설치 하고, RT kernel patch 까지 진행해 보려 한다.

## (1) Raspberry Pi OS 설치
Raspberry Pi OS를 설치하기 위해서는 [*Raspberry Pi Imager*](https://www.raspberrypi.com/software/)를 다운받아 설치한다.  
설치 후 실행하면 다음과 같은 화면이 뜬다.  
![image](/assets/images/rpi/rpi_imger.jpg){: width="50%" height="50%"}  
여기에서, *"운영체제 선택"* 을 누르고,  
![image](/assets/images/rpi/rpi_imger_2.jpg){: width="50%" height="50%"}  
*"Raspberry Pi OS (other)"* 을 누르면,  
![image](/assets/images/rpi/rpi_imger_3.jpg){: width="50%" height="50%"}  
*"Raspberry Pi OS Lite (32-bit)"* 를 선택할 수 있다.  
운영체제 선택을 하고 나오면, 메인 화면에서 없었던 톱니바퀴 버튼이 생긴다.  
![image](/assets/images/rpi/rpi_imger_4.jpg){: width="50%" height="50%"}  
톱니바퀴를 눌러보면 다음과 같은 고급옵션 창이 뜬다.  
![image](/assets/images/rpi/rpi_imger_5.jpg){: width="50%" height="50%"}  
여기에서, SSH 사용을 선택하고, 비밀번호 인승 사용을 선택한다.  
사용자 이름 및 비밀번호 설정을 선택하고 원하는 내용을 기입한다.  
(필자의 경험으로는 사용자 이름 및 비밀번호 설정을 선택하지 않고 설치하면 비밀번호가 무엇으로 설정되어 설치되는 것인지 찾을 수 없었음. 반드시 설정하자.)  
마지막으로 무선 LAN 설정을 선택하면 집의 wifi ID와 비밀번호를 넣어서 자동으로 인터넷에 연결되게 할 수 있다. SSH를 무선으로 이용하고자 하면 설정하는 것이 좋다.  
이렇게 설정하고, 저장을 누른 뒤 설치할 SD 카드 디스크를 선택하고 쓰기를 누르면 설치가 진행된다.  
설치가 완료되면 Imger 프로그램이 알아서 SD 카드 디스크를 unmount 시키는데, 우리는 추가적인 작업이 필요하기 때문에 뺐다가 다시 꽂자.  
꽂으면 boot 라는 디스크가 잡히는데, 그 곳을 열어보면 config.txt라는 파일을 찾을 수 있다. 그 파일의 끝부분에 다음 내용을 추가해 주자.  
#### config.txt
```
[pi3]
# Disable bluetooth
dtoverlay=pi3-disable-bt

[all]
# Enable UART1
enable_uart=1
```  
이 내용은 Raspberry Pi GPIO 핀 중에서 UART1 핀을 CLI Command 모드로 사용하겠다는 것이다.  
Raspberry Pi 3의 경우는 Bluetooth가 UART1을 통해 통신을 하기 때문에 Bluetooth를 비활성화 해야 UART1을 사용할 수 있다.  
![image](/assets/images/rpi/rpi_GPIO.png)  
이제 설정은 모두 끝났고, SD카드를 Raspberry Pi에 삽입한다.
전원을 연결하기 전에 UART1 TX, RX 핀과 Ground 핀을 USB-to-UART 모듈에 연결하고 PC에 연결한다.  
putty를 실행하고, 해당 COM port를 115,200 bps로 Open 한다.  
이제 Raspberry Pi에 전원을 연결하고 기다린다.  
수많은 텍스트가 나오면서 부팅이 진행되는데, 최초 부팅이 좀 더 오래걸리니 차분하게 기다려 보자.  
부팅이 끝나면 다음과 같이 login 하라고 뜨는데, 여기에 아까 설정했던 사용자 이름과 비밀번호를 입력하면된다.  
이제 SSH를 이용하기 위해서는 IP 주소를 알아야 하니 ifconfig 명령어를 이용하자.  
IP 주소를 알았다면, putty를 새로 실행하여 ssh로 해다 IP 주소를 넣어 실행하자.  
![image](/assets/images/rpi/rpi_ssh.jpg){: width="80%" height="80%"}  
putty를 ssh로 실행하면, 텍스트 색깔까지 표현되는 깔끔한 CLI 환경을 누릴 수 있다.  
여기에서 uname -a 명령어를 입력하여 현재 kernel의 버전을 확인할 수 있다.  
커널 버전은 5.15.61-v71+ #1579 SMP 라고 나온다.
## (2) wifi 연결 및 고정 IP 설정
ssh를 접속하기 위해서는 IP 주소를 알아야 하는데, Raspberry Pi 가 공유기에 연결될 때 마다 IP 가 변경되면 여간 귀찮을 일이 아니다.  
IP 주소를 고정하는 방법은 /etc/dhcpcd.conf 파일에 다음 내용을 기입하는 것이다.
```sh
~ $ sudo nano /etc/dhcpcd.conf

[/etc/dhcpcd.conf 내용 추가]
interface wlan0
sataic ip_address=111.222.333.444/24
static rounters=111.222.333.1

[nano 단축키]
저장: Ctrl + o
끝내기: Ctrl + x
```

우리는 Rapsberry Pi OS 설치 과정에서 wifi 연결을 설정했지만, 혹시 하지 못한 경우는 다음과 같이 wifi 연결 설정을 할 수 있다.  
#### /etc/wpa_supplicant/wpa_supplicant.conf 내용 생성
```sh
~ $ sudo su
~ $ wpa_passphrase "ssid_이름" > /etc/wpa_supplicant/wpa_supplicant.conf
비밀번호 입력 후 Enter
```
/etc/wpa_supplicant/wpa_supplicant.conf 내용이 아래와 같이 생성됨
```sh
network={
	ssid="wifi 이름"
	#psk="입력한 비밀번호"
	psk=생성된 psk
}
```
위 내용으로 wifi 연결을 하기 위해 다음 명령어 입력한다.  
#### wifi 연결
```sh
~ $ sudo wpa_supplicant -B -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf

-B : 백드라운드 실행
-i wlan0 : 무선랜 인터페이스 이름
-c /etc/wpa_supplicant/wpa_supplicant.conf : wifi 설정 정보 파일 경로
```
#### dhcp 할당 받기
```sh
~ $ sudo dhclient wlan0
```  
만약, SD카드에 OS 설치 과정에서 wifi 설정을 하지 않았다면 다음과 같이 뜨면서 연결이 되지 않는다.
```sh
Successfully initialized wpa_supplicant
rfkill: WLAN soft blocked
rfkill: WLAN soft blocked
```  
이는 software에 의해서 인터넷이 block되어 있기 때문이다.
다음과 같이 확인 후 해제 할 수 있다.
```sh
~ $ sudo rfkill list
0: phy0: Wireless LAN
        Soft blocked: yes
        Hard blocked: no
1: hci0: Bluetooth
        Soft blocked: no
        Hard blocked: no
~ $ sudo rfkill unblock wifi
~ $ sudo rfkill list
0: phy0: Wireless LAN
        Soft blocked: no
        Hard blocked: no
1: hci0: Bluetooth
        Soft blocked: no
        Hard blocked: no
```  
부팅하면 설정한 wifi에 자동으로 연결되게 하려면 다음과 같이 설정한다.
#### /etc/network/interfaces 내용 추가
```sh
~ $ sudo nano /etc/network/interfaces
```
```sh
auto lo

iface lo inet loopback
iface eth0 inet dhcp

allow-hotplug wlan0

auto wlan0

iface wlan0 inet dhcp
pre-up wpa_supplicant -Dwext -i wlan0 -c /etc/wpa_supplicant/wpa_supplicant.conf -B

```  

## (3) RT kernel patch
RT kernel Patch는 다음과 같이 두가지 의미를 갖는 것으로 필자는 파악하였다.  
1. kernel 단 Process를 사용자 프로그램에서 Preemption 할 수 있음.
2. 내부 Timer의 분해능 향상  

RT kernel patch를 하는 여러가지 방법이 있지만, 필자는 github.com/raspberrypi/linux에 업로드 되어있는 linux 커널 중에서 4.19.y-rt 버전을 내려받아 Rapsberry Pi 에 맞도록 컴파일하여 kernel을 교체하는 과정으로 진행할 예정이다.  
우선, 컴파일 과정이 Raspberry Pi 에게는 상당히 과부하이고 오래걸리기 때문에 별도의 linux PC 가 필요하다. 필자는 Windows 11의 WSL Ubuntu 상에서 진행하였다.

#### 1) 필요 툴 설치
```sh
~ $ sudo apt-get install git bc bison flex libssl-dev make
~ $ sudo apt-get install build-essential
~ $ sudo apt-get install g++-arm-linux-gnueabihf
~ $ sudo apt-get install gdb-multiarch
```

#### 2) 작업 디렉터리 생성
```sh
~ $ mkdir rpi-kernel
~ $ cd rpi-kernel
~/rpi-kernel $ mkdir rt-kernel
```

#### 3) 커널 소스 및 툴체인 다운로드
```sh
~/rpi-kernel $ git clone https://github.com/raspberrypi/linux.git -b rpi-4.19.y-rt
~/rpi-kernel $ git clone https://github.com/raspberrypi/tools.git
```

#### 4) 크로스 컴파일 환경설정
```sh
~/rpi-kernel $ export ARCH=arm
~/rpi-kernel $ export CROSS_COMPILE=~/rpi-kernel/tools/arm-bcm2708/gcc-linaro-arm-linux-gnueabihf-raspbian-x64/bin/arm-linux-gnueabihf-
```

#### 5) 커널 모듈 설치위치 및 디바이스트리 설치위치 설정
```sh
~/rpi-kernel $ export INSTALL_MOD_PATH=~/rpi-kernel/rt-kernel
~/rpi-kernel $ export INSTALL_DTBS_PATH=~/rpi-kernel/rt-kernel
```

#### 6) 커널빌드 설정
- Raspberry Pi 2, 3B(+)
```sh
~/rpi-kernel $ export KERNEL-kernel7
~/rpi-kernel $ cd linux
~/rpi-kernel/linux $ make bcm2709_defconfig
```  
- Raspberry Pi 4B
```sh
~/rpi-kernel $ export KERNEL-kernel7l
~/rpi-kernel $ cd linux
~/rpi-kernel/linux $ make bcm2711_defconfig
```

#### 7) 커널 컴파일
- 커널 이미지 생성
```sh
~/rpi-kernel/linux $ make -j4 zImage
```  
- 커널 모듈 빌드
```sh
~/rpi-kernel/linux $ make -j4 modules
```  
- 디바이스트리 빌드
```sh
~/rpi-kernel/linux $ make -j4 dtbs
```  
- 빌드된 모듈 설치
```sh
~/rpi-kernel/linux $ make -j4 modules_install
```  
- 디바이스트리 설치
```sh
~/rpi-kernel/linux $ make -j4 dtbs_install
```

#### 8) 커널 배포 이미지 생성
- 부트 디렉터리 생성
```sh
~/rpi-kernel/linux $ mkdir $INSTALL_MOD_PATH/boot
```  
- 커널 이미지 생성
```sh
~/rpi-kernel/linux $ ./scripts/mkknlimg ./arch/arm/zImage $INSTALL_MOD_PATH/boot/$KERNEL.img
```  
- 커널 이미지 디렉터리로 이동 후 파일 이름 변경
```sh
~/rpi-kernel/linux $ cd $INSTALL_MOD_PATH/boot
~/rpi-kernel/rt-kernel/boot $ mv $KERNEL.img kernel7_rt.img
```  

#### 9) 커널 이미지 파일 전송
- 커널 이미지 압축
```sh
~/rpi-kernel/rt-kernel/boot $ cd ..
~/rpi-kernel/rt-kernel $ tar czf ../rt-kernel.tgz *
```
- 커널 이미지 전송
```sh
~/rpi-kernel/rt-kernel $ cd ..
~/rpi-kernel $ scp rt-kernel.tgz pi@<ssh_ip>:/tmp
EX) scp rt-kernel.tgz pi@192.168.0.13:/tmp
```  
파일 전송을 위해 ssh 비밀번호(Fingerprint) yes 입력 후 진행
```sh
~/rpi-kernel $ sudo scp ./rt-kernel.tgz pi@192.168.0.13:/tmp
The authenticity of host '192.168.0.13 (192.168.0.13)' can't be established.
ECDSA key fingerprint is SHA256:Xed7FHW+g51hJbD...(생략)
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '192.168.0.13' (ECDSA) to the list of known hosts.
pi@192.168.0.13's password:
비밀번호 입력
```  

#### 10) Raspberry Pi에 kernel 설치
- 전송된 압축파일 확인
```sh
~ $ cd /tmp
/tmp $ ls
...
rt-kernel.tgz
...
```  
- 압축파일 해제  
　tar 의 -C 인자는 압축을 풀 디렉터리를 지정하는 옵션이다.
```sh
/tmp $ mkdir rt-kernel
/tmp $ tar xzf rt-kernel.tgz -C ./rt-kernel
/tmp $ cd rt-kernel
```  
- 부트관련 파일 복사
```sh
/tmp/rt-kernel $ cd boot
/tmp/rt-kernel/boot $ sudo cp -rd * /boot
/tmp/rt-kernel/boot $ sync
```  
- 라이브러리 관련 파일 복사
```sh
/tmp/rt-kernel/boot $ cd ../lib
/tmp/rt-kernel/lib $ sudo cp -dr * /lib
/tmp/rt-kernel/lib $ sync
```  
- 오버레이 관련 파일 복사
```sh
/tmp/rt-kernel/lib $ cd ../overlays
/tmp/rt-kernel/overlays $ sudo cp -d * /boot/overlays
/tmp/rt-kernel/overlays $ sync
```  
- 기타 부트관련 파일 복사
```sh
/tmp/rt-kernel/overlays $ cd ..
/tmp/rt-kernel $ sudo cp -d bcm* /boot
/tmp/rt-kernel $ sync
```  
- 부트 설정 변경
```sh
/tmp/rt-kernel $ sudo nano /boot/config.txt
```
#### /boot/config.txt 내용 추가
```sh
# Change RT-kernel
kernel=kernel7_rt.img
```  

이제 RT kernel patch의 모든 과정이 끝났고, 재부팅 하고 uname -a 명령어로 kernel 버전을 확인해 보자.  
```sh
/tmp/rt-kernel $ sudo reboot
```  
![image](/assets/images/rpi/rpi_rt_ssh.jpg){: width="80%" height="80%"}  
위와 같이 4.19.71-rt24-v71+ #1 SMP PREEMPT RT 라고 나오는 것을 볼 수 있다.
