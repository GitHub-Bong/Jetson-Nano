# Jetson Nano 를 활용한 Object Detection        

​         

# Day1            

​            

## Jetson Nano 를 배우는 이유?           

__Cloud computing vs Edge computing__         

서버와 무선통신 없이 Jetson Nano에서 모두 해결 -> 높은 반응속도, 보안 확보, 비용 절감 등 장점 보유        

​                   

## Jetson Nano 기자재 연결        

SD 카드에 Nvidia Jetpack(4.5 version)         

<img src = "/image/sd.jpg" width = "400px">     



​             

### 부팅 방법 1 - 기자재 직접 연결         

__장점: 쉽고 빠르고 효육적으로 프로그래밍 가능__         

1. Jetson Nano에 모니터, 키보드, 마우스, 전원, SD 카드 입력      
2. Jetson Nano 전원 ON (녹색불 들어오면 OK)        

<img src = "/image/direct1.jpg" width = "400px">    

<img src = "/image/direct2.jpg" width = "400px">             

​               

### 부팅 방법 2 - 외부 PC로 원격 접속하는 Headless mode(실외에서 사용 가능)               

__장점: 이동식 시스템에 적용 가능__      

__단점: 화면이 작고, 속도가 느림(화면이 클수록 느림)__             

1. Jetson Nano에 전원선, 동글 연결            

2. 마이크로 5핀 USB 케이블 꼽기               

3. 전원선 뺏다가 다시 꼽기            

4. 외부 PC에 Putty 설치 후 실행              

5. 윈도우 우클릭 후 __장치 관리자__ 클릭 -> 아래 __포트__ 클릭 -> USB 직렬 장치(COM3) 확인            

6. Putty 실행 후 __serial line__ 에 COM3 , __speed__ 에 115200 하고 __Open__ 클릭         

   1. 잘 안되면 __ssh__ 누르고 __host name__ 에 192.168.55.1, __port__ 에 22 하고 __Open__ 클릭          

7. 새로 뜬 창에 ID, Password 차례로 입력             

8. VNC 접속을 위해 다음과 같이 입력 (비밀번호 입력 요구 시 입력하면 됨)               

   ~~~~
   cd /usr/lib/systemd/user/graphical-session.target.wants
   
   sudo ln -s ../vino-server.service ./.
   
   gsettings set org.gnome.Vino prompt-enabled false
   
   gsettings set org.gnome.Vino require-encryption false
   
   gsettings set org.gnome.Vino authentication-methods "['vnc']"
   
   gsettings set org.gnome.Vino vnc-password $(echo -n 'sogang'|base64)
   
   sudo reboot
   ~~~~

9. Jetson이 재부팅 후 연결되면, VNC Viewer 설치 후 실행       
10. 주소창에 __192.168.55.1__ 입력, Password 입력        
11. __Jetson Nano 화면 크기가 작을 수록 VNC Viewer의 속도가 빨라짐__ -> 해상도 조절        
    1. 새로운 터미널 열기 (터미널 클릭 or Ctrl + Alt + t)            
    2. xrandr --fb 1000x600 (너비x높이를 의미) 입력       
12. 터미널 창 글씨 크기 조절         
    1. 터미널 창 우클릭 -> __Preferences__ 클릭 -> __Custom font__ 클릭 -> __Monospace Regular__ 클릭 -> 글씨 크기 조절       
13. 외부 PC 해상도 및 글씨 크기 조절     
    1. PC 화면 우클릭 -> 디스플레이 설정 ->  __텍스트, 앱 및 기타 항목의 크기 변경__ 로 조절              

<img src = "/image/undirect.jpg" width = "400px">          

<img src = "/image/port.PNG" width = "400px">             

<img src = "/image/putty1.PNG" width = "400px">

<img src = "/image/putty2.PNG" width = "400px">    

<img src = "/image/vnc1.PNG" width = "400px">

<img src = "/image/vnc2.PNG" width = "400px">

<img src = "/image/vnc3.PNG" width = "400px">

<img src = "/image/vnc4.PNG" width = "400px">

<img src = "/image/vnc5.PNG" width = "400px">

<img src = "/image/vnc6.PNG" width = "400px">

<br/>

## Jetson nano 인터넷 연결           

__유선__ : 랜선을 Jetson Nano에 연결          

__무선__ :             

1. 스마트폰과 usb 연결하여, usb 테더링 사용           
2. Jetson 화면 우측 상단의 Wifi 아이콘 클릭 -> wifi 연결                

​           

__인터넷 연결이 되었는지 확인__              

터미널 창 열기 (Ctrl + Alt + t) -> __iwconfig wlan0__ 입력          

<img src = "/image/wifi1.PNG" width = "600px">

​             



<br/>

## Nano Editor 설치       

인터넷 연결 된 상태에서 터미널 창 열고          

__sudo apt-get install nano__ 입력             

​	(만약 설치 안될 시 $ sudo apt-get update  $ sudo apt-get install nano 입력)            

<img src = "/image/nano1.PNG" width = "600px">

​                

## CSI camera 테스트          

<img src = "/image/camera0.jpg" width = "400px">

살짝 힘을 줘 열어주고 파란 부분이 바깥 쪽을 향하도록 사이에 넣은 후 다시 딸깍 소리가 나도록 눌러준다       

​          

터미널 창에 아래 코드 입력                          

~~~
gst-launch-1.0 nvarguscamerasrc sensor_mode=0 ! 'video/x-raw(memory:NVMM),width=3820, height=2464, framerate=21/1, format=NV12' ! nvvidconv flip-method=0 ! 'video/x-raw,width=400, height=300' ! nvvidconv ! nvegltransform ! nveglglessink -e
~~~

__(현재 400x300 크기의 카메라 창 열림)__             

​              

종료하기 위해서는 터미널 창에  __Ctrl + c__          

<img src = "/image/camera1.PNG" width = "400px">       

​              

<br/>

## Jetson-inference Docker Container 설치             

Docker 란?        

Linux 컨테이너를 만들고 사용할 수 있도록 하는 컨테이너화 기술       

Docker를 사용하면 컨테이너를 매우 가벼운 모듈식 가상 머신처럼 다룰 수 있다       

​         

Container 란?      

가상 머신과 마찬가지로 애플리케이션을 관련 라이브러리 및 종속 항목과 함께 패키지로 묶어 소프트웨어 서비스 구동을 위한 격리 환경을 마련해준다             

그러나 컨테이너는 가상 머신보다 훨씬 가볍고         

어느 환경에서나 구동되어 탄력적이고          

운영체제 커널이 하나밖에 없기 때문에 운영 체제 수준에서 업데이트 또는 패치 작업을 한 번만 수행하면 변경 사항이 모든 컨테이너에 적용되어 효율적으로 유지 관리할 수 있다        

​                         

새로운 터미널 창을 열고 (Ctrl + Alt + t)     

~~~ 
git clone --recursive https://github.com/dusty-nv/jetson-inference

cd jetson-inference

docker/run.sh
~~~

<img src = "/image/linux1.PNG" width = "300px">

​      

Docker를 실행하면 나오는 화면     

<img src = "/image/linux2.PNG" width = "600px">        

​                      

## Docker container 에서 Camera Streaming               

1. 터미널 창 열기 (Ctrl + Alt + t)
2. Docker container 열기
   1.  cd jetson-inference
   2. docker/run.sh
3. Docker container에서 카메라 구동하기
   1. video-viewer  --input-width=400 --input-height=300 csi://0
   2. 400, 300은 카메라 윈도우 크기로 변경 가능

   <img src = "/image/linux3.PNG" width = "600px">            

​             

<br/>

## 이미지 데이터에서 객체 인식         

1. 터미널 창 열기 (Ctrl + Alt + t)
2. Docker container 열기
   1.  cd jetson-inference 
   2.  docker/run.sh
3. 아래 코드 입력 
   1. detectnet.py --network=**ssd-mobilenet-v2** data/images/**peds_2.jpg** data/images/test/**output.jpg** 
      1. 파이썬으로 detectnet.py 파일 실행
      2. 사용할 딥러닝 모델 ssd-mobilenet-v2 
      3. 사용할 입력 이미지 파일
      4. 객체 인식 결과를 담은 출력 이미지 파일
4. 결과 확인
   1. 새 터미널 창 열기
   2. 출력 이미지 파일 저장된 위치 이동    
   3. cd jetson-inference/data/images/test      
   4. eog output.jpg    (eog 명령어: 이미지 파일 열기)

<img src = "/image/detection1.PNG" width = "600px">

<img src = "/image/detection2.PNG" width = "600px">

<img src = "/image/detection3.PNG" width = "600px">

<img src = "/image/detection4.PNG" width = "600px">

이미지 내에 6가지 객체를 인식 한 것을 알 수 있다         

<img src = "/image/detection5.PNG" width = "600px">



__Single Shot Multibox detection : SSD__       

SSD 모델은, 테두리 상자 조정을 위해 픽셀이나 특징들을 재 추출하지 않음으로서, 기존의 속도와 정확성의 트레이드 오프 문제를 극복하고자 고안된 분류와 조정이 같이 실행하는 단일 신경망 모델     

SSD의 알고리즘은 각 feature map 에서 다른 스케일과 비율로 default box를 생성하고 모델을 통해 계산된 좌표와 클래스 값을 활용해 bounding box 를 생성              

​               

[detectnet.py](https://colab.research.google.com/drive/1G9ecWl7AXFsid4LSholy_I_wgrHkoxAm?usp=sharing)



<br/>

## 카메라 이용 실시간 객체 인식            

Jetson Nano에 연결된 카메라로 촬영되는 영상에 대해 Docker container에서 객체 인식            

1. 터미널 창 열기 (Ctrl + Alt + t)
2. Docker container 접속 코드
   1. cd jetson-inference
   2. docker/run.sh
3. 아래 코드 입력
   1. detectnet.py --network=**ssd-mobilenet-v2** --input-width=**400** --input-height=**300** csi://0

<img src = "/image/detection6.PNG" width = "600px">

<img src = "/image/detection7.PNG" width = "600px">

코드를 보게 되면 default값으로 ssd-mobilenet-v2가 설정되어있음을 알 수 있다