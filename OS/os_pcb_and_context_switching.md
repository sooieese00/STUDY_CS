### Process Management

cpu가 각 프로세스들을 구분할 수 있어야 관리가 가능

`Process Metadata`는 각 프로세스의 특징을 저장한 데이터

<br>


**Process Metadata의 구성요소**

- Process ID
- Process State : 프로세스가 현재 어떤 상태에 있는지 (실행 중, 대기 중 등)
- Process Priority : 프로세스의 우선순위
- CPU registers : CPU 레지스터 값
- Owners :  프로세스 소유자 정보
- CPU Usage :  CPU 사용정보
- Memory Usage : 메모리 사용 정보

<br>


### PCB (Process Control Block)

> 운영체제가 각 프로세스를 관리하기 위해 사용하는 데이터 구조
> 

![image.png](./img/os_pcb.png)

**생성과정**

1. 프로그램이 실행되어 메모리에 올라감 (프로세스 생성)
2. 프로세스의 주소공간에 [코드&데이터&스택] 생성
    - 프로세스가 실행될 때 CPU가 직접 접근하는 메모리 공간
3. 해당 프로세스의 메타데이터가 PCB에 저장됨
    - 프로세스의 상태정보, 주소공간에 대한 정보

<br>


**PCB 관리방식**

- linkedList형태로 프로세스테이블을 만들어 PCB들을 관리
    
     ⇒ 새로운 PCB가 생성되거나 삭제될 때 삽입과 삭제가 용이
    
- PCB List Head에 PCB들이 생성될 때마다 하나씩 이어붙음
    
     ⇒ 프로세스 교체 작업에서 효율적
    

![image.png](./img/os_process_table.png)

<br>


 **PCB 사용 예시 : 프로세스 교체 작업**

```jsx
대기중인 프로세스의 정보를 잃어버리면 프로그램을 처음부터 다시 시작해야함. 
=> 
대기 상태로 바뀌기 직전의 실행정보를 PCB에 저장해 
다시 실행상태로 돌아왔을 때 
이전 상태에서 작업을 이어서 실행
```

1. A 프로세스 실행중, B 프로세스에서 인터럽트 발생
2. A의 현재 실행정보를 PCB에 저장
3. A는 대기상태로 전환, B의 PCB 정보를 실행상태로 전환
4. B가 작업을 완료하면 B의 실행 정보를 PCB에 저장하고, B를 대기 상태로 전환
5.  A의 PCB정보를 기반으로 A를 실행상태로 전환

<br>


### context switching

> CPU가 한 작업(프로세스 또는 스레드)에서 다른 작업으로 전환하는 과정
> 

`context : CPU가 특정 작업을 실행하기 위해 필요한 정보(레지스터 값, 프로그램 카운터 등)의 집합`

|  | **메타데이터** | **context** |
| --- | --- | --- |
| 정의 | 특정 프로세스에 대한 전체적인 정보를 담은 데이터 | CPU가 프로세스를 실행하기 위해 필요한 즉각적인 정보 |
| 사용범위 | 프로세스 관리와 운영체제의 다양한 작업에서 사용 | 주로 context switching 시에 사용 |
| 저장소 | PCB에 저장 | PCB에 저장 |
| 주요내용 | Process MetaData |  CPU레지스터 값 ,프로그램 카운터 : 현재 실행 중인 명령어의 주소,  프로그램 카운터 : 현재 실행 중인 명령어의 주소,  스택 포인터 : 함수 호출 시 필요한 스택 정보 |
|  |  | >> context는 메타데이터의 일부 |

<br>


**context switching 과정**

1. 현재 실행 중인 프로세스의 context를 PCB에 저장.
2. 새로운 프로세스의 context를 PCB에서 읽어와 레지스터에 적재
3. 새로운 프로세스 (이전에 중단된 지점부터) 실행

![image.png](./img/os_context_switching.png)

<br>


**운영체제에서 context switching이 필요한 이유**

- 멀티태스킹 지원
    
    CPU는 하나의 프로세스만을 실행할 수 있음
    
    여러 프로세스를 번갈아가며 실행하는 과정에서 context switching 필요
    
- 공정한 CPU 자원분배
    
    각 프로세스는 CPU시간을 할당받고, 일정시간이 지나면 다른 프로세스에게 CPU를 넘겨줌
    
    선점형 스케줄링 방식에서는 CPU시간을 다 소모한 프로세스는 대기 상태로 전환되고, 다른 프로세스가 실행될 수 있도록 context switching이 일어남
    
- 입출력 대기
    
     프로세스가 I/O 작업을 기다려야 할 때 실행이 중단됨
    
    나중에 다시 실행할 때 저장된 상태에서 이어서 실행
    

<br>


### context switching overhead

> context switching 시 발생하는 성능 손실


**발생 요인**

1. 상태 저장 및 복구 : 
    
    이전 프로세스 상태를 저장하고 새로운 프로세스 상태를 복구하는 데 시간 소모
    
2. 캐시 메모리 무효화 : 
    
    캐시 메모리는 특정 프로세스의 데이터를 저장하고 있음. 
    
    context switching 후 새로운 프로세스가 실행되면 기존 캐시에 저장된 데이터가 무효화되며, 
    
    새로운 데이터로 채워지는 과정에서 추가 시간 소모
    
3. 유저모드 ↔ 커널모드 전환 
    
    유저모드에서 커널모드로 전환해야하는 경우가 많아, 이 과정에서 추가적인 비용 발생
    

**영향**

빈번한 context switching이 발생하면 오버헤드가 잦아져 시스템 성능 저하

<br>



### 예상 면접 질문

- PCB는 무엇이고, 왜 사용하는지 말해주세요.
- context switching의 과정에서 PCB를 어떻게 사용하는지 설명해주세요
- context switching이 빈번하게 발생하면 어떤 영향이 있는지 말해주세요

<br>


### 참고자료

[컨텍스트 스위칭과 그 영향력 이해하기](https://f-lab.kr/insight/understanding-context-switching?gad_source=1&gclid=CjwKCAjwx4O4BhAnEiwA42SbVENgcwWuHVxK6iiZGpY8ecH1El97wWw-wImTYCakfAc7PQBV7Gr7HxoCA8wQAvD_BwE)

[PCB & Context Switching](https://gyoogle.dev/blog/computer-science/operating-system/PCB%20&%20Context%20Switching.html)

[문맥교환contextswitch알아보기](https://yoongrammer.tistory.com/53?category=961743)
