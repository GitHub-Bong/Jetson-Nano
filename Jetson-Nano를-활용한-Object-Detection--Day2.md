# Jetson Nano 를 활용한 Object Detection             

​                

# Day2              

​               



## Transfer learning ?             

[참고자료](https://towardsdatascience.com/a-comprehensive-hands-on-guide-to-transfer-learning-with-real-world-applications-in-deep-learning-212bf3b2f27a)          

기존의 머신러닝, 딥러닝 알고리즘은 특정 task를 풀기 위해 독립적으로 설계되었다       

Transfer learning(전이학습)은 task를 풀기 위해 비슷한 task를 해결하며 얻은 내용을 활용하는 것이다       

'Andrew Ng' 교수는 2016년에 지도 학습 이후로 전이 학습이 ML 성공에 핵심 포인트라고 말씀했다       

​              

지도 학습은 엄청난 양의 labeled data를 필요로 한다 이는 많은 시간과 노력이 필요함을 의미한다       

또한 비슷한 task일지라도 심각한 성능 저하를 보일 수 있다       

이것이 전이 학습의 필요성이다

![image](https://miro.medium.com/max/700/1*9GTEzcO8KxxrfutmtsPs3Q.png)

​             

<br/>

## 메모리 SWAP (가상으로 메모리 늘리기)              

터미널 창에 아래 코드 입력           

~~~
git clone https://github.com/JetsonHacksNano/installSwapfile

cd installSwapfile

./installSwapfile.sh

sudo reboot
~~~

재시작 후, 터미널에서 아래 코드 입력    

free -m

<img src = "/image/swap1.PNG" width = "400px">

또는 아래 코드 입력             

gnome-system-monitor               

<img src = "/image/swap.PNG" width = "400px">                   

​                  

<br/>

## 외장 메모리에 데이터 저장      

딥러닝 성능 향상을 위해서는 많은 데이터가 필요      

현재 사용하고 있는 64GB SD카드 외에 외장 하드디스크, USB 메모리 등을 Jetson nano에 인식하는 방법     

1. Jetson nano에서는 한글 사용이 불가하므로 외장 메모리 이름을 영문으로 설정
2. 터미널 창에서 아래 코드 입력
   1. sudo apt-get install exfat-fuse exfat-utils (외장 메모리 인식을 위한 파일 설치)
3. Jetson nano에 외장 메모리 연결
4. 외장 메모리 연결 확인
   1. Docker 밖에서 터미널 창에 아래 코드 입력
   2. ls /media/mobis/**BONG** (본인 외장 메모리 이름) 
5. Docker Container에 외장 메모리 마운트
   1. Docker 밖에서 터미널에 아래 코드 입력
   2. cd jetson-inference
   3. docker/run.sh --volume /media/mobis/**BONG**:/media/mobis/**BONG**
6. docker container에 외장 메모리 마운트 확인
   1. ls /media/mobis/**BONG** 

<img src = "/image/mount1.PNG" width = "400px">  

​              

<br/>

## 학습에 사용할 이미지 데이터 다운로드          

[Re-training SSD-Mobilenet](https://github.com/dusty-nv/jetson-inference/blob/master/docs/pytorch-ssd.md)             

[Open Images Dataset (600 classes)](https://storage.googleapis.com/openimages/web/visualizer/index.html?set=train&type=detection&c=%2Fm%2F0kpt_)               

<img src = "/image/openimage1.PNG" width = "400px">

​             

외장 메모리에 데이터 다운로드        

1. 새 터미널 창에서 아래 코드 입력

   1. ~~~~
      sudo apt-get install python3-pip
      
      sudo pip3 install boto3
      
      cd jetson-inference/python/training/detection/ssd
      
      wget https://nvidia.box.com/shared/static/djf5w54rjvpqocsiztzaandq1m3avr7c.pth -O models/mobilenet-v1-ssd-mp-0_675.pth
      
      pip3 install -v -r requirements.txt
      ~~~~

   2. ssd/models에 base model 설치되게끔 해준다

   3. Base model은 이미 pre-trained 되어 있기 때문에 transfer learning을 사용해 우리가 고른 새로운 classes들을 예측하도록 할 것 이다

2. __--stats-only__       

   1. <img src = "/image/openimage2.PNG" width = "400px">

   2. '--stats-only' 옵션은 우리가 선택한 classes들마다 얼마나 많은 사진이 있는지 알려준다. (실제로 이미지를 다운받지는 않는다)

   3. 어떤 클래스들은 (people, vehicles)은 엄청난 양의 이미지들을 가지고 있기 때문에 --stats-only 옵션을 통해 먼저 이미지의 수를 파악하는 것을 추천

   4. ~~~ 
      cd jetson-inference/python/training/detection/ssd
      python3 open_images_downloader.py --stats-only  --class-names "Apple,Orange,Banana" --data=/media/mobis/BONG
      ~~~

3. __--max-images, --max-annotations-per-class__          

   1. '--max-images' 는 다운 받을 총 이미지의 수를 정한다. 클래스마다 균등한 개수를 맞춰준다

   2. '--max-annotations-per-class'는  클래스마다 다운 받을 이미지의 수를 정한다. 클래스에 이미지가 적을 경우 그 이미지들 전체를 다운받게 한다. 클래스마다 분포가 불균형할 경우 유용하다

   3. ~~~
      cd jetson-inference/python/training/detection/ssd
      python3 open_images_downloader.py  --max-annotations-per-class=10 --class-names "Apple" --data=/media/mobis/BONG
      ~~~

   4. default로 다운받은 이미지들은 jetson-inference/python/training/detection/ssd 에 들어가게 되는데 --data= 로 위치를 변경 가능하다

   5. <img src = "/image/openimage3.PNG" width = "400px">

   6. <img src = "/image/openimage4.PNG" width = "400px">

4. 외장 메모리에 다운 받은 이미지 확인

   1. <img src = "/image/openimage5.PNG" width = "400px">

   2. <img src = "/image/openimage6.PNG" width = "400px">

   3. ~~~
      cd /media/mobis/BONG/train
      ls
      eog 9218c9cbb81f1b78.jpg
      ~~~

5. Docker container에서 외장 메모리에 다운 받은 사진 확인

   1. 새로운 터미널에 아래 코드 입력

   2. ~~~
      cd jetson-inference
      docker/run.sh --volume /media/mobis/BONG:/media/mobis/BONG
      ls /media/mobis/BONG/train
      ~~~

   3. <img src = "/image/openimage7.PNG" width = "400px">

   4. <img src = "/image/openimage8.PNG" width = "400px">



<br/>

## 다운 받은 이미지를 활용해 Re-training         

1. 이미지가 다운로드 된 외장메모리를 Docker container에 마운트
   1. cd jetson-inference
   2. docker/run.sh --volume /media/mobis/**BONG**:/media/mobis/**BONG**
2. Transfer learning을 위해 폴더로 이동
   1. cd /jetson-inference/python/training/detection/ssd
   2. <img src = "/image/retraining1.PNG" width = "400px">
3. 다운로드 된 이미지를 활용해 Re-training 수행
   1. python3 train_ssd.py **--data=/media/mobis/BONG** --model-dir=models --batch-size=4 --epochs=3
   2. <img src = "/image/retraining2.PNG" width = "400px">
4. 모델 생성 확인
   1. ls  models
   2. <img src = "/image/retraining3.PNG" width = "400px">

​      

<br/>

## ONNX로 네트워크 모델 이전       

Pytorch -> ONNX -> TensorRT            

1. ~~~
   python3 onnx_export.py --model-dir=models
   ls models
   ~~~

<img src = "/image/onnx1.PNG" width = "400px"> 

<img src = "/image/onnx2.PNG" width = "400px">           

<img src = "/image/onnx3.PNG" width = "400px">

​           

<br/>

## Re-trained 딥 네트워크 테스트          

__test 이미지 파일로 테스트__            

1. 새 터미널 창에 아래 코드 입력 ()

   1. ~~~
      cd /media/mobis/mobis_usb/test
      ls
      eog 2f7a596e42ad0bbc.jpg
      ~~~

   2. <img src = "/image/onnx4.PNG" width = "400px">

   3. <img src = "/image/onnx5.PNG" width = "400px">

2. Docker 에서 'IMAGES' 변수에 테스트 이미지 저장된 폴더 지정 

   1. IMAGES=**/media/mobis/BONG/test** 

3. Docker에서 아래 코드 입력

   1. Re-trained 된 모델 실행

   2. ~~~
      detectnet.py --model=models/ssd-mobilenet.onnx --labels=models/labels.txt --input-blob=input_0 --output-cvg=scores --output-bbox=boxes "$IMAGES/2f7a596e42ad0bbc.jpg" $IMAGES/output.jpg
      ~~~

   3.  <img src = "/image/onnx6.PNG" width = "400px">

4. 새 터미널 창에서 결과 확인

   1. eog /media/mobis/BONG/test/output.jpg
   2. <img src = "/image/onnx7.PNG" width = "400px">       

​           

__카메라로 촬영되는 영상으로 테스트__       

1. 외장 메모리 마운트 된 Docker 에 아래 코드 입력        

2. ~~~
   detectnet.py --model=models/ssd-mobilenet.onnx --labels=models/labels.txt --input-blob=input_0 --output-cvg=scores --output-bbox=boxes --input-width=300 --input-height=300 csi://0
   ~~~

3. <img src = "/image/onnx8.PNG" width = "400px">

4. <img src = "/image/onnx9.PNG" width = "400px">

