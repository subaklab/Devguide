# Out-of-tree Modules

여기서는 외부 모듈을 PX4 빌드에 추가하는 방법을 설명합니다.

외부 모듈은 내부 모듈과 동일한 include를 사용하고 내부 모듈은 uORB를 통해 상호작용할 수 있습니다.

## 사용

- `EXTERNAL_MODULES_LOCATION`은 Firmware와 동일한 구조를 갖는 디렉토리를 가리켜야 합니다.(따라서 `src`라는 디렉토리를 포함)
- 여기에는 2가지 옵션이 있습니다. : 외부디렉토리에 기존 모듈을 복사(examples/px4_simple_app 예제)하거나 직접 새로운 모듈을 생성하는 것입니다.
- 모듈(CMakeLists.txt 내부에서 `MODULE` 포함)에 새로운 이름을 주거나 기존 Firmware cmake build 설정에서 삭제하는 것입니다. 이렇게 해서  내부 모듈과의 충돌을 피합니다.
- `$EXTERNAL_MODULES_LOCATION/CMakeLists.txt` 파일에 다음과 같은 내용을 추가합니다.

```
set(config_module_list_external
    modules/<new_module>
    PARENT_SCOPE
    )
```
- `px4_add_module` 내부에 있는 `modules/<new_module>/CMakeLists.txt` 파일에 `EXTERNAL` 라인을 추가합니다. 예제는 다음과 같습니다. :

```
px4_add_module(
	MODULE modules__test_app
	MAIN test_app
	STACK_MAIN 2000
	SRCS
		px4_simple_app.c
	DEPENDS
		platforms__common
	EXTERNAL
	)

```
- `make posix EXTERNAL_MODULES_LOCATION=<path>`를 실행합니다. 다른 빌드 타겟을 실행할 수 있지만, 빌드 디렉토리는 아직 존재하면 안됩니다. 만약에 이미 존재하는 경우라면, 빌드 폴더에 cmake 변수를 설정할 수 있습니다. 다음으로 incremental build에 대해서는 더이상 `EXTERNAL_MODULES_LOCATION`를 지정할 필요는 없습니다.
