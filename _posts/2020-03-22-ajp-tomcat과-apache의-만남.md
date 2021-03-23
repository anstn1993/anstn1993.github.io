---
layout: post
title: AJP - Tomcat과 Apache의 만남 # Title of the page
hide_title: false # Hide the title when displaying the post, but shown in lists of poststhumbnail: "assets/img/thumbnails/sample-th.png"  # Add
color: brown # Add the specified color as feature image, and change link colors in post
author: "Mun Soo Kim"
tags: [기술]
---

이번 글에서는 Tomcat과 Apache가 어떻게 연결되는지에 대해서 간단하게 살펴보겠습니다. 여기서 Apache는 Tomcat의 앞단에 위치한 Proxy 서버가 됩니다.

## Tomcat의 Connector

---

Tomcat이 외부의 요청을 어떻게 받는지 먼저 살펴보겠습니다. Tomcat은 다양한 컴포넌트들로 구성이 되는데, 그중 Catalina가 외부 요청을 받을 수 있게 처리하는 컴포넌트가 Connector입니다. Connector는 하나의 서비스 내에서 여러 개로 구성될 수 있으며 각각 요청을 받아서 적절한 Context(Web Application)로 요청을 전달해 주고 동적으로 생성된 응답 콘텐츠를 받아서 응답해 주는 역할을 수행합니다.

각각의 Connector들이 위와 같은 역할을 수행하기 위해서는 기본적으로 2가지를 설정해줘야 합니다.

1. 요청을 listen하기 위한 포트 번호
2. 통신 프로토콜

위와 같은 Connector 설정은 Tomcat의 주요 설정 파일 중 하나인 server.xml에서 하게 됩니다.

- server.xml

  ```tex
  <Server>
      <Service>
          <Connector port="8000" protocol="HTTP/1.1"/>
          <Connector port="8001" protocol="HTTP/1.1"/>

          <Engine>
              <Host name="yourhostname">
                  <Context path="/webapp1"/>
                  <Context path="/webapp2"/>
              </Host>
          </Engine>
      </Service>
  </Server>
  ```

  Server 엘리먼트의 하위 엘리먼트로는 Service 엘리먼트가 들어가고 이 엘리먼트의 하위 엘리먼트로 Connector 엘리먼트와 Engine 엘리먼트가 들어가게 됩니다. 위의 설정을 살펴보면 Tomcat 서버의 Connector는 두 개가 되고 8000, 8001번 포트에서 요청을 listen하게 됩니다. 즉 저 두 포트로 요청을 보내면 Connector가 받아서 Engine으로 요청을 전달하게 됩니다.

  아래와 같이 설정할 수도 있습니다.

  ```tex
  <Server>
      <Service name="Catalina8000">
          <Connector port="8000" protocol="HTTP/1.1"/>
          <Engine>
              <Host name="yourhostname">
                  <Context path="/webapp1"/>
              </Host>
          </Engine>
  </Service>
  <Service name="Catalina8001">
          <Connector port="8001" protocol="HTTP/1.1"/>
          <Engine>
              <Host name="yourhostname">
                  <Context path="/webapp2"/>
              </Host>
          </Engine>
  </Service>
  </Server>
  ```

  한 서버에서 두 개의 Service를 구성하고 각각 Connector를 구성하여 포트별로 요청이 전달되는 Engine이 달라지게 구성할 수도 있습니다.

## AJP

---

앞서 Connector의 통신 프로토콜을 설정할 수 있다고 언급했습니다. 대표적으로 두 가지 타입이 있는데 하나가 우리에게 익숙한 HTTP이고, 다른 하나가 AJP라는 녀석입니다.

#### HTTP Connector

Tomcat은 그 자체만으로 독립적 웹 서버로 작동합니다. 그게 가능한 이유가 바로 HTTP Connector라는 컴포넌트가 있기 때문입니다. HTTP Connector는 HTTP/1.1 버전을 지원하면서 웹상의 HTTP 요청을 모두 받아줍니다.

#### AJP Connector

AJP는 Apache JServ Protocol의 약자로 Apache 웹 서버와 통신을 하기 위해서 고안된 HTTP 바이너리 프로토콜입니다. 즉 AJP Connector는 Apache를 통해서 전달되는 요청을 받아서 처리하는 역할을 담당하는 특화된 Connector라고 볼 수 있습니다. 당연히 Apache와의 연동이기 때문에 양쪽 모두에 별도의 설정을 해줘야 합니다.

## AJP Configuration

---

Apache와 Tomcat의 연동을 위한 설정을 살펴보겠습니다.

#### Tomcat Configuration

먼저 상대적으로 간단한 Tomcat부터 살펴보겠습니다.

- server.xml
  ```tex
  <Server>
      <Service name="Catalina">
          <Connector port="8001" protocol="AJP/1.3"/>
          <Engine name="Catalina">
              <Host name="localhost"/>
                  <Context path="/"/>
              </Host>
          </Engine>
      </Service>
  </Server>
  ```
  Connector의 프로토콜을 AJP로 설정해 줬고 포트는 8001번으로 설정했습니다. 간단하죠?

#### Apache Configuration

Apache에서 필요한 설정을 살펴보겠습니다.

##### 1. mod_jk 다운로드

mod_jk는 Apache의 HTTPD가 Tomcat과 AJP 프로토콜로 통신을 할 수 있게 해주는 Apache 모듈입니다. 이 모듈을 다운로드합니다. ([mod_jk 다운로드 링크](http://tomcat.apache.org/download-connectors.cgi))

##### 2. mod_jk 설치

다운로드된 패키지에 있는 mod_jk 모듈을 찾아서 그 파일을 Apache의 modules 디렉터리로 옮겨줍니다.(리눅스의 경우 mod_jk.so, 윈도우의 경우 mod_jk.dll)

##### 3. httpd.conf 설정

모듈을 옮겨줬으면 설정 파일에서 그 모듈을 사용하겠다고 선언을 해야 합니다. 아래에 나오는 설정은 mod_jk 모듈이 정상적으로 동작하는 데 필요한 설정들입니다.(각종 파일들의 경로는 예시로 작성된 것이기 때문에 실제 자신의 파일의 경로를 넣어주면 됩니다.)

- httpd.conf

  ```tex
  # Load the mod_jk module.
  LoadModule    jk_module  path/to/mod_jk.so

  # Declare the module for use with the <IfModule directive> element.  (This only applies to versions of HTTPD below 2.x.  For 2.x and above, REMOVE THIS LINE.)
  AddModule     mod_jk.c

  # Set path to workers.properties. We will create this file in the next step.  The file will be placed in the same directory as httpd.conf.
  JkWorkersFile /path/to/httpd/conf/workers.properties

  # Set path to jk shared memory.  Generally, you'll want this to point to your local state or logs directory.
  JkShmFile     /path/to/log/httpd/mod_jk.shm

  # Set path to jk logs.  This path should point to the same logs directory as the HTTPD access_log.
  JkLogFile     /path/to/log/httpd/mod_jk.log

  # Set the jk log level.  Valid values for this setting are 'debug', 'error', or 'info'.
  JkLogLevel    level

  # Set timestamp log format.  Use provided variables to customize.
  JkLogStampFormat "[%a %b %d %H:%M:%S %Y] "

  # Map a worker to a namespace.  Workers represent Tomcat instances that are listening for requests.  We'll configure these in the next section.  For the sake of this example, Tomcat's "examples" context is used, and a default worker named 'worker1', which we will create in Step 4, is designated.  Multiple JkMount attributes can be used simultaneously.
  JkMount  /examples/* worker1
  ```

  위의 설정에서 JkMount를 보면 worker에 대한 mapping 설정이 존재합니다. Worker는 Tomcat 인스턴스를 의미합니다. 위의 설정을 해석해보자면 /examples/ 이하로 들어오는 요청은 모두 worker1로 라우팅을 해주라는 의미가 됩니다. 즉 요청이 들어오면 ajp로 Tomcat에게 요청을 전달하라는 의미입니다.

##### 4. Worker 설정

3번에서 JkMount 설정으로 worker1을 지정해 줬지만 그 Worker에 대한 정보가 아직 존재하지 않습니다. Worker에 대한 정보는 workers.properties에서 설정하게 됩니다.

- workers.properties
  ```tex
  worker.list=worker1
  worker.worker1.type=ajp13
  worker.worker1.host=localhost
  worker.worker1.port=8001
  ```
  - worker.list: Worker의 namespace를 지정합니다. 복수의 Worker를 지정하고 싶다면 ','를 쓰고 이어나가면 됩니다.
  - worker.worker1.type: worker1과의 통신 프로토콜을 지정합니다.
  - worker.worker1.host: worker1의 주소를 지정합니다.
  - worker.worker1.port: worker1의 포트번호를 지정합니다. 앞선 Tomcat설정에서 AJP Connector의 포트 번호를 8001로 설정했습니다.

여기까지가 Apache와 Tomcat이 AJP를 통해서 통신하기 위한 기본적인 설정 입니다.

## 마치며

---

기술교육 프로젝트의 웹 서버 구성이 Apache + Tomcat이었는데 정확히 어떻게 연동이 되는지 잘 모르고 사용을 해왔던 터라 공부 차원에서 이 글을 작성합니다. 위의 설정들은 정말 기본적인 설정들이고 그 외에도 사용자의 니즈에 따른 다양한 설정들이 가능합니다. 이런 설정들도 앞으로 개발을 하면서 하나하나 익혀야겠다는 생각이 듭니다.

읽어주셔서 감사합니다!
