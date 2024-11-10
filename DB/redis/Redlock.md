## 분산락 (Distributed lock)
* 분산환경에 공유 데이터를 제어하기 위한 기술 입니다 
## Redis에서 분산락 
* Redis는 Single Thread 기반인것을 잊지 말자
    * 싱글 쓰레드라 동시성 문제를 좀더 가볍게 생각할 수 있다 
## SETNX
* Redis에서 가장 쉽게 분산락을 구현할 수 있는 방법은 `SETNX`를 활용하는 것이다 
    * `SETNX` : `SET if Not eXist`이다
* 즉, 값이 없을 때 값을 넣어라 라는 것이다 
* 이를 통해 Spin lock을 구현할 수 있다
### Example
* Client A가 `SETNX`로 Key, value에 특정 값을 넣는다 
* Client B가 `SETNX`로 Key, Value를 넣으려하는데 실패한다 (반복적으로 Key를 확인)
* Client A가 작업을 마치고 `DEL`로 키를 삭제한다 
* Clinet B가 작업을 하기 위해 키에 다시 값을 넣는다 
### 단점 
* SPOF(Single point of failure)가 될 수 있다 
* spin lock이므로, 계속 while 문으로 락이 해제되었는지 확인하므로 자원을 더 쓴다 
* Redis의 Replica는 비동기다 
    * 상황에 따라서, Replica가 있을경우는 race condition 문제가 발생할 수 있다 
## RedLock
* Message Broker를 사용해, Lock 획득을 위한 채널을 구독하고 있다가, Lock이 해제되었다는 메시지를 받았을 때 Lock을 획득하는 방식이다 
    * 여러 Cluster가 있을때도 동작가능하다 
### 작동 방식 
* 시스템에 배포된 N개의 Redis 서버에 동시에 Lock 요청을 한다 
* Redis에서 Lock을 얻기위해 `SETNX`명령을 보낸다 
* Lock을 성공적으로 몇개의 서버에서 얻었는지 count합니다
    * N / 2 + 1개 이상의 서버에서 락을 얻었다면 성공적으로 락을 얻는것으로 판단한다
    * key, value저장할때, **만료시간을 넣는다**
* 모든 N개의 서버에서 락을 해제한다 
