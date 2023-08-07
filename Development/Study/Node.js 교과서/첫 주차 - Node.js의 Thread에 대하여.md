# 발단
- Node.js의 경우 Multi-Thread이지만 하나의 Thread만 유저가 사용가능하게 하고, 나머지는 내부적으로 사용
- 최근에는 worker_thread와 같은 기능으로 유저도 Multi-Thread 환경에서 개발할 수 있도록 하는것으로 알고있음.
- 여기서 Node.js의 Thread는 어떻게 구성되어 있고, 어떻게 작동하는지, 왜 하나의 Thread만 유저가 사용할 수 있게 한건지, worker_thread 기능은 또 뭔지 궁금해졌음.
### 궁금한 점
1. Node.js의 Thread 구조는 어떻게 될 까?
2. Node.js의 Thread는 어떻게 작동할까?
3. 왜 Node.js는 하나의 Thread만 유저가 사용할 수 있게 한걸까?(왜 싱글스레드 모델을 선택했을까?)
4. worker_thread 기능은 무엇일까?

# 진행
## 1. Node.js의 Thread 구조는 어떻게 될까?
---
Node.js는 Event-Driven Architecture를 채용해 <mark style="background: #FFF3A3A6;">(1) 오케스트레이션을 위한 Event Loop</mark>와 <mark style="background: #FFF3A3A6;">(2)고비용 작업을 위한 Worker Pool</mark>을 포함하고 있습니다.
### Event Loop
main loop, main thread, event thread 라고도 부릅니다.
간단히 말하면 Event에 등록된 Javascript callback 함수를 실행하고, Non-Blocking 비동기 요청(ex. 네트워크 I/O) 역시 처리하는 스레드 입니다.
Node.js 애플리케이션이 시작되면 먼저 초기화 phase를 완료하고, module들을 require하고 Event에 대한 callback들을 등록합니다. 
그 다음 Node.js 애플리케이션은 Event Loop에 진입해 들어오는 Client 요청들에 대해 그에 해당하는 callback들을 실행시키므로써 응답합니다. 이러한 callback들은 동기적으로 실행되며, 완료된 후에도 계속 처리하기 위해 비동기 요청들을 등록할 수도 있습니다.
### Worker Pool(Thread Pool)
Task를 처리하는 Worker로 이루어진 Pool 입니다.
Node.js의 Worker Pool은 [libuv](https://docs.libuv.org/en/v1.x/threadpool.html)의 구현체로, libuv는 일반적인 Task Submission API(`uv_queue_work`)를 노출하고 있습니다.
- `int uv_queue_work(*loop, *req, work_cb, after_work_cb)` : threadpool의 스레드에서,  `work_cb`를 실행하는 작업 request를 initialize 합니다. `work_cb`가 완료되고 나면, loop thread에서 `after_work_cb`를 호출합니다.
Node.js에서는 Worker Pool을 "비싼" 작업을 처리하기 위해 사용합니다. "비싼" 작업으로는 OS단에서 non-blocking 버전을 제공하지 않는 I/O 작업(특히 CPU 집약적인 작업)이 있습니다.
Worker Pool을 사용하는 Node.js Module API는 다음과 같습니다.
##### I/O-집약적인 API
1. [DNS](https://nodejs.org/api/dns.html) : Node.js에서 Name Resolution을 가능케 해주는 Module로, host name의 IP 주소를 조회하는 등의 작업을 수행할 때 사용할 수 있습니다. (조회시 항상 DNS 프로토콜을 사용하는 건 아니고, OS 기능을 이용해 이름 확인을 할 수도 있습니다.)
2. [File System](https://nodejs.org/api/fs.html#fs_threadpool_usage) : (`fs.FSWatcher()`와 명시적으로 동기 방식인 API를 제외한) 대부분의 File System API가 libuv의 threadpool을 사용합니다.
	- 참고로 libuv이 제공하는 threadpool은 내부적으로 모든 File System 작업과 `getaddrinfo`, `getnameinfo`와 같은 요청을 실행하는데 사용됩니다.
##### CPU-집약적인 API
1. [Crypto](https://nodejs.org/api/crypto.html): Node.js에서 암호화를 위해 사용하는 Module로, 주로 Hashing같은 고비용 작업을 처리하는데에 libuv의 threadpool을 사용합니다.
	- `crypto.pbkdf2()`, `crypto.scrypt()`, `crypto.randomBytes()`, `crypto.randomFill()`, `crypto.generateKeyPair()`
 2. [Zlib](https://nodejs.org/api/zlib.html#zlib_threadpool_usage): Node.js에서 파일 압축을 위해 사용하는 Module로, 명시적으로 동기적인 몇몇을 제외하면 대부분의 작업이 libuv의 threadpool을 사용합니다.
이러한 API들만이 Worker Pool의 task 소스입니다. [C++ add-on](https://nodejs.org/api/addons.html)을 사용하는 애플리케이션과 모듈은 다른 task들을 Worker Pool에 submit 할 수 있습니다.
## 2. Node.js의 Thread는 어떻게 작동할까?
---
##### API 호출과 task submit
위에서 언급한 API들 중 하나를 Event Loop의 callback 에서 호출한다면, Event Loop는 해당 API에 대한 Node.js C++ bindings에 들어가 task를 submit할 때 약간의 설정 비용(setup cost)를 지불합니다.
설정 비용들은 task의 전체 비용에 비해 무시할 수 있는 수준이라 Event Loop는 이를 `offloading` 합니다.
- `offloading`: 리소스 집약적인 컴퓨팅 작업을 하드웨어 가속기와 같은 별도의 프로세서나 클러스터, 그리드 또는 클라우드와 같은 외부 플랫폼으로 전송하는 것.
task를 submit할 때, Node.js는 Node.js C++ bindings 내부의 해당 C++ 함수에 대한 Pointer를 제공합니다.
##### Node.js가 다음으로 실행할 code를 정하는 방법
Event Loop와 Worker Pool은 각각 대기중인 event, 대기중인 task들 위한 Queue를 추상적으로 유지합니다.
**추상적**이라고 말한 이유는, 사실 Event Loop는 queue를 유지하고 있지 않습니다. 대신 OS에 요청한 file descriptor들의 집합을 갖고 있는데, 이를 통해 OS에 OS별 메커니즘(epoll (linux), kqueue (OSX), event ports (Solaris), IOCP (Windows))을 이용해 모니터링을 요청합니다.
file descriptor로는 **network socket**, **감시중인 파일**들이 있습니다.
OS가 이들 중 하나 준비되었다고 말하면, Event Loop는 이를 적절한 Event로 변환하고 해당 Event와 관련된 callback을 호출합니다.
반면 이와 대조적으로 Worker Pool은 실행되어야 하는 Task들에 대한 실제 Queue를 사용합니다. Worker는 queue에서 task를 pop하여 처리한 뒤, task가 완료되면 "적어도 하나의 작업이 완료되었음(At least one task is finished)" event를 Event Loop에 발생시킵니다.
##### 개발자가 구현시 중요하게 생각해야 할 점
위에서 언급한 (적은 thread로 많은 client를 처리하는)Node.js의 구조상, thread가 어떤 request 때문에 block될 경우 끼치는 피해가 더 막대하므로 공정한 스케줄링(fair scheduling)을 개발자가 보장할 필요가 있게 됩니다.
따라서 개발자는 구현시 requset에 대한 scheduling에 좀 더 신경써야 합니다.
다시 한 번 말하면, 모든 들어오는 요청과 나가는 응답이 Event Loop를 통과하므로 Event Loop가 한 작업에 대해 너무 오래걸리지 않도록, 즉 **Block**되지 않도록 Javascript callback들이 빠르게 처리되도록 해야합니다.
따라서 개발자는 input에 제한을 걸고, input이 너무 긴 작업은 거절하는 것을 고려해야 합니다.
이를 통해 callback의 복잡성이 크더라도, input을 제한하여 callback이 worst-case 작업시간보다 오래 걸리지 않도록 할 수 있습니다.
- 사실 뒤에 block을 유발할 수 있는 작업들에 대한 내용이 있는데, 이러한 내용은 공식 문서를 정리하는 것만 해도 길어질 것 같아 추후 따로 글을 작성하도록 하겠습니다.
## 3. Node.js의 싱글-스레드 모델 선택 이유
---
[Node.js 공식문서](https://nodejs.org/en/about)에 따르면, Node.js는 (싱글 스레드 모델을 선택하므로써) 현대의 일반적인 동시성 모델들과 달리<mark style="background: #FFF3A3A6;"> 비효율적이고 사용하기 어려운 '스레드-기반 네트워킹'</mark>을 피할 수 있으며, lock이 발생하지 않아 사용자들이 dead-lock의 위험으로 부터 자유로울 수 있게 됐다고 말하고 있습니다.
- Node.js에서는 적은 수의 Thread를 사용하므로써, Thread의 Memory, Context Switching으로 인해 발생하는 공간과 시간 Overhead를 줄일 수 있다고 말합니다. 그를 통해 시스템에서는 더 많은 시간과 메모리를 클라이언트들에게 사용할 수 있다는 이점을 얻을 수 있습니다.
### 영향 받은 모델
또한 Node.js의 Event Loop Runtime 구조는 Ruby의 `Event Machine`과 Python의 `Twisted`로 부터 영향을 받았으며, 유사한 설계를 갖고있다고 말합니다.
#### Event Machine
Ruby를 위한 Event-Driven IO 및 경량 동시성 라이브러리로, Node.js, libev와 마찬가지로 **`Reactor`** 패턴을 사용해 Event-Driven I/O를 제공합니다.
Event Machine은 '높은 확장성, 성능, 안정성' 그리고 '개발자가 애플리케이션의 로직에 집중할 수 있도록 하는 API(스레드 네트워크 프로그래밍의 복잡성을 제거한)' 이라는 두 핵심 요구사항을 만족하도록 설계되어 있습니다. 
정리하자면,  '*Scalable해서 규모를 증감*시킬 수 있으면서, **Thread에 대해 신경쓰지 않고 개발**할 수 있는 **동시성을 지닌 API**를 만들도록 돕는 라이브러리' 라고 정리할 수 있을 것 같습니다.
##### **`Reactor 패턴`**  
- 참고 문서
	- [Reactor - An Object Behavioral Pattern for Demultiplexing and Dispatching Handles for Synchronous Events](http://www.dre.vanderbilt.edu/~schmidt/PDF/reactor-siemens.pdf)
	- [Reactor pattern - Wikipedia](https://en.wikipedia.org/wiki/Reactor_pattern)
Service Handler에 *동시에 전달*되는 Request를 처리하기 위한 **Event Handling 패턴**.
Service Handler는 들어오는 요청을 **역다중화**(Demultiplex)한 뒤, 관련된 Request Handler(service provider)에게 동기적으로 전송합니다.
모든 Reactor 시스템은 정의대로라면 싱글-스레드이지만, 멀티스레드 환경에서도 존재할 수 있습니다.
애플리케이션 부분 코드를 Reactor 구현과 분리해 코드를 모듈화해 재사용이 가능하다는 장점이 있지만, 제어의 흐름이 역전되어 있어 절차적 패턴(Procedural Pattern)보다 디버깅하기 힘들고 Request Handler 호출이 동기적이라는 점과 Demultiplexer 때문에 최대 동시성이 제한(*대칭형 멀티프로세싱 하드웨어에서 특히*) 된다는 한계가 존재합니다.
- **역다중화** : 물리적 장치의 효율성을 높이기 위해 최소한의 물리적 요소를 사용해 최대한 데이터를 전달하는 **다중화(Multiplex)** 작업을 거쳐 전달받은 데이터를 다시 합치는 작업. [#](https://dbehdrhs.tistory.com/98)
	- Event Demultiplexer의 경우 `Handles`에서 발생하는 이벤트를 Block하며, Blocking 없이 Handle에 대한 작업을 시작할 수 있을 때 반환됩니다.
	- I/O Event에 대한 일반적인 Demultiplexer는 `select`라는, UNIX 및 WIN32 OS에서 제공되는 Event Demultiplexing System Call 입니다.
		- `select` call은 어떤 handles가 *애플리케이션 프로세스를 Block하지 않고* <mark style="background: #FFF3A3A6;">동시에</mark> 작업을 호출할 수 있는지를 나타내는 작업입니다.
		- `Handles` : 일반적으로 OS에서 관리하는 자원을 식별합니다. Logging Server에서 Socket Endpoint를 식별하여, Synchronous Event Demultiplexer가 Endpoint에서 이벤트가 발생할 때 까지 기다리도록 해줍니다.
- 참고할만한 글
	- [Leveraging Event Multiplexing in Even-Driven Architectures | Guillermo Wrba](https://www.linkedin.com/pulse/leveraging-event-multiplexing-even-driven-guillermo-wrba/)
	- [비동기 서버에서 이벤트 루프를 블록하면 안 되는 이유 1부 - 멀티플렉싱 기반의 다중 접속 서버로 가기까지 (linecorp.com)](https://engineering.linecorp.com/ko/blog/do-not-block-the-event-loop-part1)
	- [[네이버클라우드 기술&경험] IO Multip.. : 네이버블로그 (naver.com)](https://blog.naver.com/n_cloudplatform/222189669084)
	- [12장 IO 멀티플렉싱(Multiplexing) (tistory.com)](https://dbehdrhs.tistory.com/98)
	- [What is a demultiplexer ? - GeeksforGeeks](https://www.geeksforgeeks.org/what-is-a-demultiplexer/)
#### Twisted
Python으로 작성된 Event-Driven Networking Engine으로  Event-Driven Web Server는 물론 mail, SSH Client 등을 포함하고 있어 웹 애플리케이션이 구현 가능합니다. Event Engine과 마찬가지로 `Reactor` 패턴이 적용되어 있습니다.
- 참고할만한 글
	- [네트워크 프로그램 개발을 위한 파이썬 프레임워크 – DATA ON-AIR (dataonair.or.kr)](https://dataonair.or.kr/db-tech-reference/d-lounge/expert-column/?mod=document&uid=53875)
	- [Twisted 22.10.0 documentation](https://docs.twisted.org/en/stable/)

## 4. worker_threads
---
`worker_threads`는 Javascript를 병렬 실행하는 Thread를 사용할 수 있게 해주는 모듈이다.
Node.js에 내장되어 있는 동기 I/O 작업이 더 효율적이므로 I/O-집약적인 작업보다 CPU-집약적인 연산을 수행하는 Javascript 실행에 더 유용합니다.
`isMainThread`, `Worker`, `parentPort`등을 통해 메인 스레드로 부터 Worker들을 만들고, Worker들이 어떤 작업을 수행할 지를 지정할 수 있습니다. 
또한 메인 스레드 <-> Worker 간 `message` 이벤트를 발생시키거나, 메시지를 송수신할 port(`MessageChannel`)를 만들어 데이터를 주고받을 수 있습니다.
- worker_threads 역시 이 문서에서 다루면 너무 길어질 것 같아 추후 따로 글을 작성하겠습니다.
# 정리 후기
- 공식 문서를 찾기보다 검색을 먼저 하는 습관이 어느순간부터 들어있었는데, 이번에 여러 공식 문서들을 찾아보며 공식 문서만 잘 찾아봐도 검색할 일이 많이 줄겠다는 생각이 들었다.
- 개인적으로 역다중화, 다중화와 같은 개념부터 I/O Blocking, Non-Blocking 등 모르던 개념들을 많이 알게 되어서 나중에 이들에 대해 따로 정리하는 글을 작성해보고 싶어졌다.
- FE만 공부하던 입장에서 이런 동시성 관련된 문제에 공부하니 머리가 많이 깨질것 같지만(...) 흥미도 많이 생기게 되었다!
- 다음 주는 주제를 하나만 잡던가 좀 쉬운걸로 잡아야겠다.. 다른 것들도 하면서 글 정리하려니 한 주 가지고는 택도 없을 것 같다.

# 이외 참고한 문서 및 블로그 포스트
- [Worker threads | Node.js v20.5.0 Documentation (nodejs.org)](https://nodejs.org/api/worker_threads.html)
- [Don't Block the Event Loop (or the Worker Pool) | Node.js (nodejs.org)](https://nodejs.org/en/docs/guides/dont-block-the-event-loop)
- [Introduction to Node.js (nodejs.dev)](https://nodejs.dev/en/learn/introduction-to-nodejs/)
- [The Node.js Event Loop, Timers, and process.nextTick() | Node.js (nodejs.org)](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick)
- [Leveraging Event Multiplexing in Even-Driven Architectures | LinkedIn](https://www.linkedin.com/pulse/leveraging-event-multiplexing-even-driven-guillermo-wrba/)
- [비동기 서버에서 이벤트 루프를 블록하면 안 되는 이유 1부 - 멀티플렉싱 기반의 다중 접속 서버로 가기까지 (linecorp.com)](https://engineering.linecorp.com/ko/blog/do-not-block-the-event-loop-part1)
- [비동기 서버에서 이벤트 루프를 블록하면 안 되는 이유 3부 - Reactor 패턴과 이벤트 루프 (linecorp.com)](https://engineering.linecorp.com/ko/blog/do-not-block-the-event-loop-part3)
- [[네이버클라우드 기술&경험] IO Multiplexing (IO 멀티플렉싱) 기본 개념부터 심화까지 -1부- : 네이버블로그 (naver.com)](https://blog.naver.com/n_cloudplatform/222189669084)
- [12장 IO 멀티플렉싱(Multiplexing) (tistory.com)](https://dbehdrhs.tistory.com/98)
- [입출력 다중화 (I/O Multiplexing) (tistory.com)](https://plummmm.tistory.com/68)