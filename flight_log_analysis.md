# 비행 로그 분석 (Flight Log Analysis)

PX4 flight log를 분석하는 다양한 소프트웨어 패키지가 있습니다. 아래에서 소개합니다.

## [Log Muncher](http://logs.uaventure.com)

주의 : Log Muncher는 이전 log format (.px4log)만 사용가능합니다.

### 업로드

사용자는 아래 웹페이지에 방문해서 직접 log 파일을 업로드할 수 있습니다.: [http://logs.uaventure.com/](http://logs.uaventure.com/)

![](images/flight_log_analysis/logmuncher.png)

### 결과

![](images/flight_log_analysis/log-muncher-result.png)

[예제 Log](http://logs.uaventure.com/view/KwTFDaheRueMNmFRJQ3huH)

### 장점
* 웹 기반이라 일반 사용자에게 적합
* 사용자가 업로드와 로드 후에 다른 사람들과 결과를 공유 가능

### 단점
* 분석이 제한적이고 커스터마이즈가 불가능

## [Flight Review](http://logs.px4.io)

Flight Review는 Log Muncher의 차기 버전으로 새로운 ULog 포맷을 조합해서 사용합니다.

### 예제
![](images/flight_log_analysis/flight-review-example.png)

### 장점
* 웹 기반이라 일반 사용자에게 적합
* 사용자가 업로드와 로드 후에 다른 사람들과 결과를 공유 가능
* 상호작용하는 plot

### 단점
* 커스터마이즈가 불가능


## [FlightPlot](https://github.com/PX4/FlightPlot)

![](https://pixhawk.org/_media/dev/flightplot-0.2.16-screenshot.png)

* [FlightPlot 다운받기](https://s3.amazonaws.com/flightplot/releases/latest.html)

### 장점
* 자바 기반으로 다양한 플랫폼 대응
* 직관적인 GUI로 프로그래밍 지식이 필요없이 사용 가능

### 단점
* 기본 제공하는 기능으로 분석에 제한

## [PX4Tools](https://github.com/dronecrew/px4tools)

![](images/flight_log_analysis/px4tools.png)

### 설치

* 추천하는 방식은 anaconda3를 사용하는 것입니다. 자세한 내용은 [px4tools github page](https://github.com/dronecrew/px4tools)을 참고하세요.

```bash
conda install -c https://conda.anaconda.org/dronecrew px4tools
```

### 장점
* 공유가 쉽고 사용자는 github에 (e.g. [https://github.com/jgoppert/lpe-analysis/blob/master/15-09-30%20Kabir%20Log.ipynb](https://github.com/jgoppert/lpe-analysis/blob/master/15-09-30%20Kabir%20Log.ipynb)) notebooks을 볼 수 있음
* 파이썬 기반으로 다양한 플랫폼을 지원하며 anaconda 2와 anaconda3에서 동작
* ipython/ jupyter notebooks은 쉽게 분석한 내용을 공유할 수 있습니다.
* 상세한 분석을 위한 고급 도표 기능

### 단점
* 사용자가 파이썬을 알고 있어야 함
* 사용하기 전에 sdlog2_dump.py 나 px4tools embedded px42csv 프로그램을 사용해서 log 파일을 csv로 변환이 필요
