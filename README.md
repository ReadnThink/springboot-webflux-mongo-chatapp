# springboot-webflux-mongo-chatapp
채팅 프로그래밍을 만들어보며 발생한 이슈 및 고민

<img src="https://github.com/ReadnThink/pocoapoco-refectoring/assets/103480627/ebd6e114-8b63-45fc-9aa1-010c69412c7f" width="800" height="500"/>

# MongoDB 사용 이유

RDB는 스키마가 있고 연관관계 맵핑을 통해 Join을 하여 쿼리를 한번 더 날린다.

MongoDB는 Collection을 저장할 수 있어 쿼리를 2번 날리지 않아도 된다.

- 입력시에 Collection을 넣기위해 시간이 많이 소요되지만
- 조회시에 바로 읽을 수 있어 조회에 특화되어 있다.
- 단점은 데이터가 변경되면 그에따른 Collection값들도 직접 바꿔야한다는 것이다.

# Netty 사용 이유
## 서블릿 기반 스프링 - `Context Switching O`

스레드 풀에서 스레드를 꺼내서 사용합니다.

- 스레드는 DB에 접근하여 데이터가 전달될때까지 스레드는 기다리게 됩니다.
    - 스레드 기반 서버는 스레드가 많으면 Context Switching이 높아져 요청이 더 느려집니다.
        - Thread 생성시 OS에 1:1로 매칭이되어 OS에 권한을 넘겨야하기 때문
        - 채팅서버는 응답이 빨라야 합니다.

## 비동기 (싱글 스레드) 기반 - `Context Switching X`

스레드 혼자서 요청과 반환을 담당합니다.

- 스레드 하나로 Context Switching없이 요청이 가능합니다.
    - 완료된 요청들은 스레드가 쉴때 반환하게 됩니다.
    - Context Switching이 일어나지 않아 속도가 빠릅니다.
- MongoDB를 사용해야 하는 이유
    - 서버가 비동기면 DB도 비동기여야 합니다.
 
# Flux
흐름을 갖고 있습니다.
HTTP처럼 연결하고 끊는게 아닌 SSE를 통해 지속적인 흐름을 유지할 수 있습니다.

* TEXT_EVENT_STREAM_VALUE : SSE 프로토콜
* Flux : 지속적인 리턴
* Mono : 한번만 리턴
* @CrossOrigin : javascript 통신하기 위함

# CrossOrigin
어떠한 오리진에서 작동하고 있는 웹 어플리케이션이 다른 오리진 서버로의 엑세스를 오리진 사이의 HTTP 요청에 의해 허가를 할 수 있는 체계라고 할 수 있습니다.
즉, java와 javascript의 통신시 HTTP가 막혀있었음. <br/>
@CrossOrigin 어노테이션을 사용해 해결 <br/>
![image](https://github.com/ReadnThink/springboot-webflux-mongo-chatapp/assets/103480627/ab836963-78ca-4f8b-832a-c6e068c879a9)

# MongoDB Capped collection 버퍼 크기 에러
tailable cursor가 capped collection이 아닌 것을 요청했다는 것입니다.  <br/>
확인해보니 tailable는 capped collection설정을 해주어야 했습니다. <br/>

[해결 블로그](https://sol-b.tistory.com/124)
