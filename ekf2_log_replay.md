# EKF2 Log Replay
여기서는 실제 flight log에 replay 기능을 사용해서 EKF2 estimator의 파라미터를 튜닝하는 방법을 보여줍니다. `sdlog2` logger를 기반으로 합니다.

## 소개
개발자는 EKF2 estimator 입력으로 들어오는 센서 데이터의 로그를 뽑을 수 있습니다. 이렇게 하면 개발자가 로그 데이터를 기반으로 replay를 실행하면 오프라인으로 estimator의 parameter를 튜닝해서 다른 parameter 집합에 대해서도 실험해 볼 수 있습니다. 이 페이지의 나머지 부분은 이 기능으로 효과를 얻기 위해서 어떤 parameter를 설정해야하는지 그리고 올바로 기능을 사용하는 방법에 대해서 알아봅니다.

## 전제 조건
* **EKF2_REC_RPL** 파라미터를 1로 설정. 이렇게 하면 estimator로 하여금 logging에서 특정 replay 메시지를 publish하도록 합니다.
* **SDLOG_PRIO_BOOST** parameter를 {0, 1, 2, 3} 중에 하나의 값으로 설정합니다. 0인 경우는 onboard logging app의 우선순위가 기본(low) 스케쥴링 우선순위를 가지게 됩니다. low 스케쥴링 우선순위인 경우 logging 메시지 손실이 일어날 수 있습니다. log 파일에 건너뛴 메시지가 있는 경우 'gaps'이라고 표시되므로 이런 경우 parameter 값을 최대치인 3으로 설정합니다. 테스팅에서는 데이터 손실을 피하기 위해서 최소 2 값으로 설정합니다.

## Deployment
실제 flight log가 있다면 PX4 Firmware의 root 디렉토리에서 다음과 같은 명령을 이용해서 replay를 실행시킵니다.
```
make posix_sitl_replay replay logfile=absolute_path_to_log_file/my_log_file.px4log
```
'absolute_path_to_log_file/my_log_file.px4log'는 replay 실행시 사용할 log 파일의 절대경로입니다.
일단 명령이 실행되면 해당 위치와 replay할 log 파일의 이름을 검사합니다.

## replay를 위해 parameter 튜닝 변경하기
replay에 대한 estimator parameter 값을 설정할 수 있습니다. 이 parameter는 log 파일을 replay했던 동일 디렉토리에 존재하며 파일이름은 **replay_params.txt** 입니다.(예 **build_posix_sitl_replay/src/firmware/posix/rootfs/replay_params.txt**) 처음으로 replay를 실행할때, 이 파일은 자동 생성되며 flight log에서 가져온 기본 EKF2 parameter 값으로 채워집니다. 이후에 EKF2 parameter 값을 변경할려면, text 파일에 관련 숫자를 변경하면 됩니다. gyro bias를 위한 noise 값 설정은 다음과 같이 합니다.
```
EKF2_GB_NOISE 0.001
```
