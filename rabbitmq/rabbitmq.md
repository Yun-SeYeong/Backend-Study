## Overview

`RabbitMQ`는 AMQP를 따르는 오픈소스 메세지 브로커 소프트웨어(메시지 지향 미들웨어)로 시작하여 지금은 STOMP, MQTT등의 프로토콜을 지원하기 위해 플러그인 구조와 함께 확장되고 있다.

### AMQP?

AMQP는 메시지 지향 미들웨어로 개방형 표준 응용 계층 프로토콜이다. AMQP는 메시징 제공자와 클라이언트의 동작에 대해 각기 다른 벤더들의 구현체가 상호 운용될 수 있도록한다. 주로 각기 다른 구현체 간의 통신을 표준화하는데 초점을 두었다. 따라서 특정 언어에 관계없이 상호 운용이 가능하다.

### Producer

`producer`는 message를 메세지를 `RabbitMQ`로 전달한다. 다음과 같이 표기된다 :

![Untitled](/Users/yunseyeong/Documents/workspace/intellij/Backend-Study/rabbitmq/1.png)

### Queue

`queue`는 `RabbitMQ` 안에 있으며 message를 전달 받으면 내부적으로 저장하게 된다. host의 memory, disk limit에 따라 제한될 수 있다. 또한 다수의 `producer`로 부터 message를 받아 저장할 수 있고, 다수의 `consumer`에서 message를 가져갈 수 도 있다. 다음과 같이 표기된다:

![Untitled](/Users/yunseyeong/Documents/workspace/intellij/Backend-Study/rabbitmq/2.png)

### Consumer

`consumer`는 `queue`로부터 message를 가져와 처리한다. 다음과 같이 표기한다 :

![Untitled](/Users/yunseyeong/Documents/workspace/intellij/Backend-Study/rabbitmq/3.png)

### Hello world!

다음은 `RabbitMQ` 에서 제공하는 기본예제이다. `producer` 는 message를 만들어 `queue` 에 전달한다. 받은 message 는 `queue` 에 저장되고 `consumer` 는 해당 message를 `queue` 로 부터 받아와 처리하게 된다.

![Untitled](/Users/yunseyeong/Documents/workspace/intellij/Backend-Study/rabbitmq/4.png)

## Publish / Subscribe

위에서는 `producer` 가 특정 `queue`에 직접 message를 전달하고 특정 `consumer`가 직접 가져와 데이터를 처리하는 형태였다. 이외에 다양한 방법으로 `consumer` 에 message를 전달 할 수 있는데, 이 패턴을 publish/subcribe라고 한다.

### Exchange

기본적으로 `RabbitMQ`에서는 절대 `producer`가 `queue`로 message를 전달하지 않고 `exchange`를 통해 전달하게 된다. `exchange` 는 producer가 전달한 message를 다수의 `queue`로 분배하게 된는데 이를 binding이라고 한다. `RabbitMQ`에서는 기본적으로 4가지 방식을 지원하는데 `direct`, `topic`, `headers`, `fanout`이 있다.

exchange는 아래 x와 같이 표기된다:

![Untitled](/Users/yunseyeong/Documents/workspace/intellij/Backend-Study/rabbitmq/5.png)

위의 구조가 생성되는 순서를 보면 :

1. `exchange`, `queue`를 생성한다.
2. `exchange`와 연결할 `queue`를 바인딩한다.
3. `producer`는 특정 `exchange`로 message를 전달한다.

### Direct Exchange

Direct Exchange는 가장 단순한 방식으로 특정 `queue`를 routing_key를 통해 지정한다. 다음과 같이 message가 publish 될때 전달된 routing_key에 대응하는 `queue`에 전달하는 방식이다.

![Untitled](/Users/yunseyeong/Documents/workspace/intellij/Backend-Study/rabbitmq/6.png)

만약 다음과 같이 여러 `queue`에 지정하면 Fanout Exchange를 사용한것 처럼 모든 `queue`에 전달된다.

![Untitled](/Users/yunseyeong/Documents/workspace/intellij/Backend-Study/rabbitmq/7.png)

### Topic Exchange

Topic Exchange는 routing_key을 패턴에 따라 분류하게 된다. routing_key의 사이즈는 255byte로 제한된다.

Topic Exchange는 다음 두가지 문자를 통해 routing_key를 분류한다.

- `*` (star)는 정확히 하나의 단어를 의미한다.
- `#` (hash)는 없거나 여러개의 단어를 의미한다.

![Untitled](/Users/yunseyeong/Documents/workspace/intellij/Backend-Study/rabbitmq/8.png)

예를 들어 카테고리를 `<celerity>.<colour>.<species>` 와 같이 나누었을때 다음과 같이 분류된다.

- Q1은 orange색의 동물들이 들어간다.
- Q2는 모든 rabbit이나 모든 개으른 동물들이 들어가게 된다.

만약 `quick.orange.rabbit` 은 Q1, Q2 모두에 들어가게된다.

### Header Exchange

Header Exchange는 Topic Exchange와 분류방식은 같으나 routing_key가 아닌 header를 통해 분류하게 되다.

header를 비교할 때 두가지 옵션이 있다.

- {x-match: any} : 헤더 값 중 하나이상 일치하면 message 전달
- {x-match: all} : 헤더 값이 모두 일치하면 message 전달

### Fanout Exchange

Fanout Exchange는 모든 `queue`를 대상으로 바인딩을 한다. 따라서 message가 전달되면 모든 `queue`로 데이터를 전달하게 된다.

Reference

- [](https://ko.wikipedia.org/wiki/AMQP)[AMQP - 위키백과, 우리 모두의 백과사전](https://ko.wikipedia.org/wiki/AMQP)
- [](https://www.rabbitmq.com/getstarted.html)https://www.rabbitmq.com/getstarted.html
