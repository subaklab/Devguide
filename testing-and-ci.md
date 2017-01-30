# 테스팅과 지속적 통합(Testing and Continuous Integration)

PX4는 확장된 단위 테스트와 지속적 통합에 필요한 기능을 제공합니다. 이 페이지는 개요를 제공합니다.

## 로컬 머신에서 테스팅(Testing on the local machine)

다음 명령으로 PX4 posix 포트를 실행시키는 작은 크기의 쉘을 시작시킵니다. :

```
make posix_sitl_shell none
```

다음으로 쉘은 단위 테스트를 실행합니다.:

```
pxh> tests mixer
```

bash에서 바로 전체 단위 테스트를 실행시키는 것도 가능합니다. :

```
make tests
```

## Cloud / CI에서 테스팅 (Testing in the Cloud / CI)
