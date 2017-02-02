# Intel RealSense 200 드라이버 Ubuntu에 설치하기
이 튜토리얼에서 Intel RealSense R200 카메라를 리눅스 환경에 설치하는 방법을 소개합니다. ROS를 통해 이미지 수집하는 것이 가능합니다. RealSense R200 카메라는 아래 사진을 참고하세요.:

![](images/realsense_intel/realsense.png)

Virtual Box에서 guest OS로 Ubuntu를 실행합니다. 이 Ubuntu에서 드라이버 패키지를 설치합니다. Virtual Box를 실행하는 호스트 컴퓨터의 스펙은 아래와 같습니다. :

- 호스트 OS : Windows 8
- 프로세서 : Intel(R) Core(TM) i7-4702MQ CPU @ 2.20GHz
- Virtual Box : Oracle VM. Version 5.0.14 r105127
- Extensions: Extension package for Virtual Box installed (Needed for USB3 support)
- Guest OS : Linux - Ubuntu 14.04.3 LTS

이 튜터리얼은 다음과 같은 순서로 구성되어 있습니다. : 첫번째 부분은 Virtual Box에서 Ubuntu 14.04 설치하는 방법을 보여줍니다. 두번째 부분은 ROS Indigo와 카메라 드라이버를 설치하는 방법을 알아봅니다. 자주 사용하는 표현에 대한 의미는 다음과 같습니다. :
- Virtual Box (VB) : 다른 가상머신을 실행하는 프로그램. 이 경우 Oracle VM.
- Virtual Machine (VM) : Virtual Box에서 실행되는 OS로 guest system이라고도 하며 여기서는 ubuntu를 말함.

## Virtual Box에서 Ubuntu 14.04.3 LTS 설치하기

- 새 Virtual Machine(VM) 생성 : Linux 64-Bit
- Ubuntu 14.04.3 LTS iso 파일 다운로드 : ([ubuntu-14.04.3-desktop-amd64.iso](http://www.ubuntu.com/download/desktop))
- Ubuntu 설치 :
  - 설치하는 동안 다음 2개 옵션의 체크 해제 :
	- Download updates while installing
	- Install this third party software
- 설치 후, whole desktop으로 Ubuntu를 표시하기 위해 Virtual Box 활성화가 필요하다 :
  -  VM Ubuntu를 시작하고 로그인해서 Virtual Box의 메뉴바에 있는 Devices->Insert Guest Additions CD image를 클릭합니다.
	-  "Run"을 클릭하고 Ubuntu에서 팝업으로 나타난 윈도우에 패스워드를 입력합니다.
	- 설치가 완료될때까지 기다리고 재시작합니다. 이제 whole desktop에서 VM이 보이는지 확인합니다.
	- Ubuntu에서 윈도우가 뜨면서 업데이트 여부를 물어봅니다. 일단 업데이트하지 않고 넘어 갑니다.
- Virtual Box에서 USB 3 컨트롤러 활성화 :
  - Virtual Machine 종료시키기
	- Virtual Machine 설정의 USB 선택 메뉴로 가서 "USB 3.0(xHCI)" 선택합니다. Virtual Box의 확장 패키지를 설치한 경우에만 가능
	- Virtual Machine를 재시작

## ROS Indigo 설치하기
- [ROS indigo installation guide](http://wiki.ros.org/indigo/Installation/Ubuntu)의 지시를 따릅니다.
  - Desktop-Full 버전 설치
	- "Initialize rosdep"와 "Environment setup" 섹션의 설명을 따라 실행

## 카메라 드라이버 설치하기
- git 설치 :
```bash
sudo apt-get install git
```
- 드라이버 다운로드 및 설치
	- Clone [RealSense_ROS repository](https://github.com/PercATI/RealSense_ROS):
	```bash
	git clone https://github.com/PercATI/RealSense_ROS.git
	```
- [여기](https://github.com/PercATI/RealSense_ROS/tree/master/r200_install)를 참고합니다.
	- 아래 설치 패키지 실행 여부를 묻는 창이 뜨면 enter 버튼을 누릅니다. :
		```
		Intel Low Power Subsystem support in ACPI mode (MFD_INTEL_LPSS_ACPI) [N/m/y/?] (NEW)
		```
		```
		 Intel Low Power Subsystem support in PCI mode (MFD_INTEL_LPSS_PCI) [N/m/y/?] (NEW)

		```
		```
		 Dell Airplane Mode Switch driver (DELL_RBTN) [N/m/y/?] (NEW)
		```
	- 설치 과정의 끝에 다음과 같은 에러 메시지가 보이면 드라이버 설치가 정상적으로 처리되지 않은 것입니다.:
	```
	rmmod: ERROR: Module uvcvideo is not currently loaded
	```

- 설치가 완료된 후에, Virtual Machine을 리부팅합니다.

- 카메라 드라이버 테스트:
	- Intel RealSense 카메라를 컴퓨터에 USB3를 지원하는 컴퓨터에 연결합니다. 이때 USB3 케이블을 사용합니다.
	- Virtual Box의 메뉴바에 Devices->USB-> Intel Corp Intel RealSense 3D Camera R200을 클릭합니다. 이는 카메라 USB가 Virtual Machine로 연결되도록 합니다.
	- [unpacked folder]/Bin/DSReadCameraInfo 파일 실행 :
	  - 다음과 같은 에러 메시지가 나타나면, 컴퓨터에서 USB 케이블을 분리합니다. 다시 연결하고 다시 Virtual Box의 메뉴바에서 Devices->USB-> Intel Corp Intel RealSense 3D Camera R200을 클릭합니다. 그리고 다시 [unpacked folder]/Bin/DSReadCameraInfo 파일을 실행합니다.
		```
		DSAPI call failed at ReadCameraInfo.cpp:134!
		```
		- 카메라 드라이버가 동작하고 Intel RealSense R200을 인식하면, Intel RealSense R200 camera에 대한 상세정보를 살펴봅니다.

- ROS nodlet 설치 및 테스팅 :
  - ROS nodlet 설치하기 위해 [여기](https://github.com/PercATI/RealSense_ROS/blob/master/realsense_dist/2.3/doc/RealSense-ROS-R200-nodelet.md)에서 "Installation" 섹션의 지시를 따릅니다.
	  - 모든게 정상적으로 동작하면, Intel RealSense R200 카메라에서 다른 데이터 스트림을 ROS topic으로 publish합니다.
