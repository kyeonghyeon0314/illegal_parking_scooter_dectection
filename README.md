# ilegal_parking_scooter_dectection
## Reference
- [LeGO-LOAM](https://github.com/RobustFieldAutonomyLab/LeGO-LOAM)
- [Mseg3D](https://github.com/jialeli1/lidarseg3d)
## Contents
- mapping
- Localization
- Semantic Segmantation

# mapping([LeGO-LOAM](https://github.com/RobustFieldAutonomyLab/LeGO-LOAM))
맵핑한 후 pcd 파일로 변환하여 저장

## utility.h 수정
- VLP-16 관련 상수들 주석 처리
- ~~Ouster OS1-64 주석 해제~~
- Ouster OS1-16 주석 해제 (기종에 맞게)
- ```extern const string pointCloudTopic = "/velodyne_points"``` 토픽 ```/ouster/points```로 수정
### 특이사항
- LeGO-LOAM에서는 9-DOF IMU가 필요하므로 ouster 기종은 imu/data를 사용할 수 없음
- ~~pcd 파일을 저장하는 디렉토리 절대 경로를 지정하는 란이 있지만 지정을 하여도 저장되는 모습은 보이지 않아 [다른 방법](#pcd-파일-저장)을 활용~~
```
// Save pcd
extern const string fileDirectory = "/tmp/";
```

## pcd 파일 저장
pcd 파일 저장은 lego-loam 상에서 ```computer/tmp/``` 디렉토리안에 자동으로 저장되는것을 확인하였습니다. ```home/user``` 디렉토리가 아닌 ``` / ``` (최상위 디렉토리)의 ```tmp```디렉토리 안에 저장이 되는 것을 확인 하였습니다. ```utility.h```안에 fileDirctory는 수정하지 않도록 합니다.

<div align="center">
  <div style="margin-bottom: 10px;">
    <img src="/img/cap/root_directory.png" width="70%">
    <p style="text-align: center;">최상위 디렉토리</p>
  </div>
  <div style="margin-bottom: 10px;">
    <img src="/img/cap/save_directory.png" width="70%">
    <p style="text-align: center;">tmp</p>
  </div>
</div>

###  Topic을 활용한 pcd 저장(/laser_cloud_surround)
```
roslaunch lego_loam run.launch
rosbag play lidar_5.bag --clock --topic /ouster/points
rosbag record -O mapping_data.bag /laser_cloud_surround
rosrun pcl_ros bag_to_pcd mapping_data.bag /laser_cloud_surround pcd # 마지막으로 저장되는 pcd 파일이 전체 map
```

## 맵 정보
- [**mapping_01.pcd**](/img/map/mapping_01.png) : 6호관 7호관 사이
- [**mapping_02.pcd**](/img/map/mapping_02.png) : 제2도서관 연구단지? 쪽 길
- [**mapping_03.pcd**](/img/map/mapping_03.png) : 농대 쪽 큰길 -> 중도 정문쪽
- [**mapping_04.pcd**](/img/map/mapping_04.png) : 건지광장
- [**mapping_05.pcd**](/img/map/mapping_05.png) : 7호관 -> 2호관 -> 4호관 -> 3호관 -> 공대입구 -> 내리막
- [**mapping_06.pcd**](/img/map/mapping_06.png) : 자연대 본관과 3호관
- [**mapping_07.pcd**](/img/map/mapping_07.png) : 공대 입구-> 내리막 -> 진수당 한바퀴
- [**mapping_08.pcd**](/img/map/mapping_08.png) : 진수당 주차장 -> 법대 오르막 오른뒤 한바퀴-> 본부별관 앞 주차장 순회
- [**mapping_09.pcd**](/img/map/mapping_09.png) : 법대,글로벌 인재관 사이 오르막 오르고난 후 장소 -> 제2도서관 (loop closer 없음)
- [**mapping_10.pcd**](/img/map/mapping_10.png) : 경상대 2호관 에서 법대내리막
- [**mapping_11.pcd**](/img/map/mapping_11.png) : 공대 공장동 앞 -> 7호관 - > 6호관-> 제2도서관-> 연구실 단지 -> 후생관쪽 문-> 농대길->중도 정문 -> 후생관 -> 경상대 2호관-> 인문대 -> 실크로드 센터 -> 공대 오르막


### 특이사항
아래와 같은 문제점이 발생을 하는데 이는 Loop Closure 기반으로 설계되어진 LeGO-LOAM 특성상 생기는 문제인것으로 추측 중에 있음

- **mapping_11**에서 후생관 앞 교차로 부분이 만나지 않음 (각자 따로 진행을 해보았지만 동일한 증상이 발생)
- **mapping_11**에서 후생관 앞 교차로에서 잘못 mapping 되면서 경상대 2호관 까지 가는 길이 7호관 건물을 가로 질러가는 문제가 발생
<div align="center">
  <div style="margin-bottom: 10px;">
    <img src="/img/map/11_error_2.png" width="70%">
    <p style="text-align: center;">교차로 부분</p>
  </div>
  <div style="margin-bottom: 10px;">
    <img src="/img/map/11_error_1.png" width="70%">
    <p style="text-align: center;">7호관과 윗길</p>
  </div>
</div>
