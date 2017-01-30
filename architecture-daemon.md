# Daemons

daemon은 백그라운드로 실행되는 프로세스를 말합니다. NuttX에서 daemon 프로세스는 task가 되며, POSIX(Linux / Mac OS)에서 daemon은 thread입니다.

`px4_task_spawn()` 명령을 이용해서 새로운 daemon이 생성됩니다.

```C++
daemon_task = px4_task_spawn_cmd("commander",
			     SCHED_DEFAULT,
			     SCHED_PRIORITY_DEFAULT + 40,
			     3600,
			     commander_thread_main,
			     (char * const *)&argv[0]);
```


사용한 인자들을 살펴보면:

  * arg0: 프로세스 이름으로 `commander` 사용
  * arg1: 스케쥴링 타입(PR or FIFO)
  * arg2: 스케쥴링 우선순위
  * arg3: 새로운 프로세스나 thread의 스택 사이즈
  * arg4: task / thread main 함수
  * arg5: void pointer를 새 task로 전달하는데 여기서는 commandline 인자를 가짐
