# 1. 블록
* * *
HDFS파일은지정한크기의 블록으로 나누어지고, 각 블록은 독립적으로 저장됩니다. 

HDFS의 블록은 128MB와 같이 매우 큰 단위입니다. 블록이 큰 이유는 탐색 비용을 최소화할 수 있기 때문입니다. 블록이 크면 하드디스크에서 블록의 시작접을 탐색하는 데 걸리는 시간을 줄일 수 있고, 네트워크를 통해 데이터를 전송하는데 더 많은 시간을 할당할 수 있습니다. 따라서 여러개의 블록으로 구성된 대용량 파일을 전송하는 시간은 디스크IO속도에 크게 영향을 받게 됩니다. 

### 블록 크기 파일 분할
***
기본블록 크기를 넘어서는 파일은 블록 크기로 분할하여 저장합니다. 

  * 블록 크기가보다 작은 파일은 단일 블록으로 저장 
    - 실제 파일 크기의 블록이 됨
  * 블록 단위로 나누어 저장하기 때문에 디스크 사이즈보다 더 큰 파일을 보관할 수 있음
    - 블록 단위로 파일을 나누어 저장하기 때문에 700G * 2 =1.4T 크기의 HDFS에 1T 의 파일 저장 가능
    
### 블록 추상화의 이점
***
첫번째의 파일 하나의 크기가 단일 디스크의 용량보다 더 커질수 있습니다. 메일에 첨부파일을 전송할 때 한번에 보낼 수 있는 용량이 제한되어있을 때 분할 압축을 통해 전송 가능한 용량으로 나누어 보낼 수 있습니다. HDFS도 큰 파일을 블록단위로 나누어서 단일 디스크의 용량보다 큰 파일을 저장할 수 있습니다. 

두번째는 파일 단위보다 블록 단위로 추상화를 하면 스토리지의 서브 시스템을 단순하게 만들 수 있다는 것입니다. 파일 탐색 지점이나 메타정보를 저장할 때 사이즈가 고정되어 있으므로 구현이 좀 더 쉽습니다. 

세번째는 내고장성을 제공하는데 필요한 복제을 구현할 때 매우 적합합니다. 블록 단위로 복제를 구현하기 때문에 데이터 복제에도 어려움 없이 처리할 수 있습니다. 같은 노드에 같은 블록이 존재하지 않도록 복제하여 노드가 고장일 경우 다른 노드의 블록으로 복구할 수 있습니다. 

### 블록지역성
***
맵리듀스를 이용한 분산 컴퓨팅은 블록의 지역성을 이용해 성능을 높입니다. 맵리듀스를 처리할 때 현재 노드에 저장되어 있는 블록을 이용하는 블록 지역성을 통해 성능을 높일 수 있습니다. 

   * 네트워크를 이용한 데이터 전송 시간 감소 
   * 대용량 데이터 확인을 위한 디스크 탐색 시간 감소 
   * 적절한 단위의 블록 크기를 이용한 CPU 처리시간 증가 
   
클라우드 저장공간을 이용하는 경우 HDFS를 이용하는 경우보다 속도가 느려집니다. 하지만 클라우드 저장공간을 이용하는 경우 영구적인 데이터 보관 및 HDFS 관리비 절감에 따른 장점이 있습니다. 

### 블록 작업 순서
***
블록 지역성을 위한 작업 우선 순위는 다음과 같습니다. 

    * 같은 노드에 있는 데이터 
    * 같은 랙의 노드에 있는 데이터 
    * 다른 랙의 노드에 있는 데이터 
    
### 블록스캐너
***
데이터노드는 주기적으로 블록 스캐너를 실행하여 블록의 체크섬을 확인하고 오류가 발생하면 수정합니다. 

### 블록 캐싱 
***
데이터노드에 저장된 데이터 중 자주 읽는 블록은 블록 캐시라는 데이터노드의 메모리에 명시적으로 캐싱할 수 있습니다. 파일 단위로 캐싱 할 수도 있어서 조인에 사용되는 데이터들을 등록하여 읽기 성능을 높일 수 있습니다.
```
$ hdfs cacheadmin
Usage: bin/hdfs cacheadmin [COMMAND]
          [-addDirective -path <path> -pool <pool-name> [-force] [-replication <replication>] [-ttl <time-to-live>]]
          [-modifyDirective -id <id> [-path <path>] [-force] [-replication <replication>] [-pool <pool-name>] [-ttl <time-to-live>]]
          [-listDirectives [-stats] [-path <path>] [-pool <pool>] [-id <id>]
          [-removeDirective <id>]
          [-removeDirectives -path <path>]
          [-addPool <name> [-owner <owner>] [-group <group>] [-mode <mode>] [-limit <limit>] [-maxTtl <maxTtl>]
          [-modifyPool <name> [-owner <owner>] [-group <group>] [-mode <mode>] [-limit <limit>] [-maxTtl <maxTtl>]]
          [-removePool <name>]
          [-listPools [-stats] [<name>]]
          [-help <command-name>]

# pool 등록 
$ hdfs cacheadmin -addPool pool1
Successfully added cache pool pool1.

# path 등록
$ hdfs cacheadmin -addDirective -path /user/hadoop/shs -pool pool1
Added cache directive 1

# 캐쉬 확인 
 hdfs cacheadmin -listDirectives
Found 1 entry
 ID POOL    REPL EXPIRY  PATH             
  1 pool1      1 never   /user/hadoop/shs 
```
