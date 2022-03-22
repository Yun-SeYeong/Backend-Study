# Gradle

> Gradle은 오픈소스 기반의 빌드 자동화 툴로 거의 모든 소프트웨어를 빌드할 수 있도록 유연하게 설계되었다.

#### High performance

Gradle은 소프트웨어 가동에 필요한 작업만 하여 불필요한 작업을 피한다. 왜냐하면 입력 출력은 계속해서 변화 할 수 있기 때문이다. 또한 이전에 실행한 task의 output이나 다른 환경에서 실행한 output은 build cache를 통해 재활용이 가능하다.

#### JVM foundation

Gradle은 JVM환경에서 동작하게된다. 따라서 Java Development Kit(JDK)가 설치되어야 한다. 이는 Java API를 통해 개발하는 사용자에 이점이 된다. 예로 task type이나 plugin을 커스텀 할 수 있다. 또한 다른 플렛폼에서도 쉽게 동작할 수 있다는 장점도 된다.

#### Conventions

Gradle은 Maven's book과 프로젝트들의 공통 점들(Java projects)을 벤치마크하여 쉽게 규약들을 이해하고 빌드 할 수 있도록 하였다. 플러그인들에 접근이 많은 프로젝트를 간단한 스크립트로 빌드해 낼 수 있게된다. 하지만 이런 규약들은 제안하고 있지 않다. (즉 테스크를 추가하고 커스터마이징이 모두 허용된다.)

#### Extensibility

Task type과 build model을 간단히 확장이 가능하다. Android build support를 보면 많은 새로운 빌드 컨셉이 추가되고 있다.

#### IDE support

몇몇의 major IDE들에서 Gradle build가 지원되고 있다. Android Studio, IntelliJ IDE, Eclipse, NetBeans 등.

#### Insight

[Build scans]([Getting started with Build Scan™ for Gradle and Apache Maven™ | Gradle Inc.](https://scans.gradle.com/?_ga=2.114868226.442035419.1647824366-1499376163.1639986287))를 통해 Build issue를 점검할 수 있다. 또한 다른 누구와 해당 build를 공유하고 도움을 받을 수 있다. 

## Five things you need to know about Gradle

> Gradle은 처음 다뤄도 쉽게 사용이 가능하여 간단하게 powerful한 빌드가 가능하다. 하지만 core principles를 이해하면 보다 활용도가 좋아질 수 있다.

#### 1. Gradle is a general-pupose build tool

Gradle은 어떤 software와도 호환이 가능하다. 왜냐하면 어떤것을 빌드해야 할지,  어떻게 완료되었는  지에 대해 추상적이기 때문이다. 

주목할 점은 현재는 dependency에 대해서 Maven-과 Ivy-compatible repository들과 filesystem만 사용하도록 제안하고 있다는 것이다. 이는 build하기 위해 많은 일을 하지 않아도 된다는 것을 의미한다. Gradle은 프로젝트에 layer규약을 추가하고 plugins를 prebuilt하는 등 공통 부분을 빌드하여 보다 쉽게 만든다. 또한 만든 custom plugins를 encapsulate하여 공유 할 수 있다.

#### 2. The Core model is based on tasks

Gradle은 Directed Acyclic Graphs (DAGs) of tasks 로 빌드된다. 빌드에 필요한 작업들이 task들과 이들을 있는 선들로 이루어지게 된다 (각 Dependincies에 기반한). Gradle은 어떤 task가 어떤 순서로 실행되어야 하는지를 결정하고 이들을 실행 시키게된다.

![Example task graphs](https://docs.gradle.org/current/userguide/img/task-dag-examples.png)

대부분의 build process들은 이 모델에 해당하게 된다. 이는 왜 Gradle이 유연한지를 설명하게 된다. 따라서 plugin들과 build script를 위 모델에 맞추어 정의하면 된다는 뜻이된다. 

##### Task구성

- Action: 파일을 복사하거나 컴파일하는등 무언가 일을 하는 것을 뜻한다.

- Inputs: 값이나 파일 또는 디렉토리로 action에 사용되거나 실행하게 된다.

- Outputs: action을 통해 수정되거나 생성된 파일이나 디렉토리

#### 3. Gradle has several fixed build phases

Gradle은 다음 3단계를 통해 buildScript를 evaluate와 execute하게 된다.

1. initializaion
   
   먼저, build환경과 어떤 project 선택할지 설정하게 된다.

2. Configuration
   
   task graph에 따라 구조화하고 설정한다. 이때 어떤 task가 실행되고 어떤 순서로 실행될지 유저가 설정한 내용을 기반으로 결정하게된다.

3. Execution
   
   결정된 tasks들을 마지막 단계까지 실행하게된다.

설정은 당연히 configuration단계에서 모두 evaluated된다. 하지만, 많은 build들에 task action(예를 들면 `doLast{}`, `doFirst{}`) 를 가지고 있는데 이것들은 excution단계에서 evaluated된다. 이게 중요한 이유는 configuration단계에서 evaluated된 코드는 execution단계에서 변경되지 않기 때문이다.

#### 4. Gradle is extensible in more ways than one

Gradle에서 제공되는 build logic만 이용해도 좋다. 하지만 그런 경우는 드물다. 만약 특변한 요구조건이 필요하다면 custom build logic이 필요하다.

Gradle은 몇가지 확장할 수 있는 메카니즘을 제공하고 있다.

- Custom task types
  
  만약 이미 있는 task는 만들지 못한다. 일반적으로 가장 좋은 방법은 [buildSrc]([Organizing Gradle Projects](https://docs.gradle.org/current/userguide/organizing_gradle_projects.html#sec:build_sources)) directory 나 packaged plugin안에 있는 소스파일을 통해 custom task를 추가한다. 

- Custom task actions
  
  `Task.doFirst` 또는 `Task.doLast` method를 통해 task에 build logic을 붙일 수 있다.

- Extra properties on projects and tasks
  
  설정이나 프로젝트에 custom action이나 build logic을 추가할 수 있다. 추가된 설정에 적용한 task들이 명확하게 적용이 안될 수 있다. 예를 들면 Gradle의 core plugin들에 추가된 설정들은 유저가 추가한대로 작동 되지 않을 수 있다.

- Custom Conventions
  
  Conventions는 간단하게 강력한 빌드를 할 수 있도록 해준다. 따라서 이해하고 사용하면 보다 쉽게 사용할 수 있다. Convention은 standard project structures와 naming conventions를 통해 빌드하는 것을 봐왔었다. 유저는 제공되는 conventions를 통해 plugin을 만들 수 있다.

- A cusom model
  
  Gradle은 tasks, files, dependency cofigurations를 통해 새로운 컨셉의 build를 만드는것을 허용하고 있다. source sets를 추가함으로써 대부분의 언어들로 빌드할 수 있는 것을 볼 수 있다. 적절한 모델링은 빌드를 쉽게하고 효율적으로 만들어 줄 것이다.

#### 5.Build script operate aginst an API

API 문서는 [Groovy DSL Reference](https://docs.gradle.org/current/dsl/) 와 [Javadocs](https://docs.gradle.org/current/javadoc/) 에 정의되어 있다. 



Reference

- https://docs.gradle.org/current/userguide/what_is_gradle.html#4_gradle_is_extensible_in_more_ways_than_one
