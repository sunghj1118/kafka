카프카에서 CLI (command-line tool)을 통해 카프카 브로커 운영에 필요한 다양한 명령을 내릴 수 있다. 

CLI에서는 필수 옵션과 선택 옵션이 있는데, 선택 옵션은 선택을 않을시 기본 설정값을 따른다. 파티션 수는 만들어진 이후 바꿀 수 없기 때문에 설정값을 제대로 맞추는것이 중요하다. 

## 로컬 카프카 설치 및 실행
![[Screenshot 2023-08-09 at 2.29.27 PM.png]]
![[Screenshot 2023-08-09 at 2.31.30 PM.png]]

설정을 다음과 같이 변경한다: 기존 -> 최신
![[Screenshot 2023-08-09 at 2.34.30 PM.png]]
![[Screenshot 2023-08-09 at 2.39.41 PM.png]]
![[Screenshot 2023-08-09 at 2.39.18 PM.png]]

### 주키퍼 실행

`bin/zookeeper-server-start.sh config/zookeeper.properties`
를 CLI에 입력하면 주키퍼가 실행된다. sh는 shellscript이며 .properties 파일의 설정을 따른다.

*The operation couldn’t be completed. Unable to locate a Java Runtime.
Please visit http://www.java.com for information on installing Java.*

놀랍게도 자바가 안 깔려있어서 실행이 안되니 brew install openjdk로 설치해줬다.
![[Screenshot 2023-08-09 at 2.46.21 PM.png]]
자바를 설치하고 PATH 설정을 해주니 잘 작동한다.
![[Screenshot 2023-08-09 at 2.52.41 PM.png]]
![[Screenshot 2023-08-09 at 2.53.43 PM.png]]

-it 

### 카프카 브로커 실행

`bin/kafka-server-start.sh config/server.properties`
위 명령어로 카프카 브로커를 실행한다.

![[Screenshot 2023-08-09 at 2.54.41 PM.png]]

카프카 브로커도 잘 작동한다. 카프카 서버가 브로커다.



### 카프카 정상 실행 여부 확인
카프카 내에 여러 shell script가 존재하는데 이는 카프카의 상황을 조회하는데 쓰인다.

로컬 호스트가 잘 작동하는지 확인:
`bin/kafka-broker-api-versions.sh --bootstrap-server localhost:9092`
![[Screenshot 2023-08-09 at 2.58.44 PM.png]]

카프카 토픽의 리스트를 조회:
`bin/kafka-topics.sh --bootstrap-server localhost:9092 --list`

![[Screenshot 2023-08-09 at 2.59.42 PM.png]]

만든게 없어서 정상.



## kafka-topics.sh

카프카 클러스터 정보와 토픽 이름마능로 토픽을 생성할 수 있다.
클러스터 정보와 토픽 이름은 토픽을 만들기 위한 필수 값이다.

 --create 옵션을 써서 토픽을 만들 수 있다.
`bin/kafka-topics.sh --create --bootstrap-server my-kafka:9092 --topic test` 
이름은 test다.

파티션을 개수를 늘리기 위해서는 --alter 옵션을 사용하면 된다.
그러나, 파티션을 줄이는 명령은 불가능하다.
파티션을 줄여야 한다면 차라리 새로 만드는것이 더 빠르다.

![[Screenshot 2023-08-09 at 3.17.38 PM.png]]

![[Screenshot 2023-08-09 at 3.20.26 PM.png]]

기본 값을 3으로 해놨기 때문에 3개의 파티션이 생성되었다.
![[Screenshot 2023-08-09 at 3.21.38 PM.png]]

 --alter로 파티션이 10개 늘어났다.
![[Screenshot 2023-08-09 at 3.23.01 PM.png]]

## kafka-configs.sh

토픽의 일부 옵션을 설정하기 위해서는 kafka-configs.sh 명령어를 사용해야 한다.

server.properties에 저장되어 있는 기본값들 또한 조회할 수 있다.
![[Screenshot 2023-08-09 at 3.28.02 PM.png]]

## kafka-console-producer.sh

데이터를 넣을 수 있는 명령어다.
![[Screenshot 2023-08-09 at 3.39.23 PM.png]]

참고로 kafka console producer을 나오려면 Ctrl+C를 누르면 된다.

레코드 키가 있는 경우에는 한개의 파티션에 지정되서 들어가고, 그렇지 않을 경우 RR으로 기본으로 들어간다.

`bin/kafka-console-producer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka`
이때는 parsekey를 안 줬기 때문에 null 값으로 들어간다.
![[Screenshot 2023-08-09 at 3.44.16 PM.png]]



## kafka-console-consumer.sh

조회하기 위한 명령이다.

`bin/kafka-console-consumer.sh --bootstrap-server my-kafka:9092 --topic hello.kafka --from-beginning`
![[Screenshot 2023-08-09 at 3.46.43 PM.png]]
메시지 값


메시지 키와 함께 조회:
![[Screenshot 2023-08-09 at 3.48.22 PM.png]]

 --max-messages 옵션은 메시지 개수를 설정한다.
 --partition 옵션을 사용하면 특정 파티션만 컨슘할 수 있다.
 --group 컨슈머 그룹을 기반으로 kafka-console-consumer이 동작한다. 어디 읽었는지 확인할 수 있다.


  __ consumer_offsets 컨슈머 그룹을 사용하면 어디까지 읽었는지의 커밋이 저장되는 곳.


## kafka-consumer-groups.sh

컨슈머 그룹을 조회하기 위해 쓰이는 명령어.

컨슈머 랙: 파티션 마지막 레코드와 현재까지 가져간 레코드의 차이.


