# 1장. 도커란?
**도커(Docker)는 리눅스 컨테이너에 여러 기능을 추가함으로써 애플리케이션을 컨테이너로서 좀 더 쉽게 사용할 수 있게 만들어진 오픈소스 프로젝트다.** 도커 자체는 Go 언어로 작성돼있다. 기존에 쓰이던 가상화 방법인 가상 머신과는 달리 도커 컨테이너는 성능의 손실이 거의 없어 차세대 클라우드 인프라 솔루션으로서 많은 개발자들에게 주목받고 있다.

도커와 관련된 프로젝트는 Docker Compose, Private Registry, Docker Hub, Docker for Desktop 등 여러 가지가 있지만 일반적으로 도커라고 하면 Docker Engine 혹은 도커에 관련된 모든 프로젝트를 의미한다.  
도커 엔진은 **컨테이너를 생성하고 관리하는 주체다.**

이 장에선, Docker Engine에 대해 간략하게 짚어보고 책에 나온 설치 과정은 생략하도록 한다.

## 가상 머신과 도커 컨테이너
기존의 가상화 기술은 Hypervisor를 이용해 여러 개의 OS를 하나의 호스트에서 생성해 사용하는 방식이었다. 이러한 여러 개의 OS는 가상 머신이라는 단위로 구별되고, 각 가상 머신에는 Ubuntu, CentOS 등의 OS가 설치되어 사용된다. Hypervisor에 의해 생성되고 관리되는 OS는 Guest OS라고 하며, 각 게스트 OS는 다른 게스트 OS와는 완전히 독립된 공간과 시스템 자원을 할당받아 사용한다. 대표적인 가상화 툴은 VirtualBox, VMware 등이 있다.

![image](https://github.com/alanhakhyeonsong/LetsReadBooks/assets/60968342/5bf410ad-a1fb-45b7-82d6-106d0847ca08)

하지만, 각종 시스템 자원을 가상화하고 독립된 공간을 생성하는 작업은 Hypervisor를 반드시 거치기 때문에 일반 호스트에 비해 **성능의 손실이 발생**한다. 가상 머신은 Guest OS를 사용하기 위한 라이브러리, 커널 등을 모두 포함하기 때문에 가상 머신을 배포하기 위한 이미지로 만들었을 때 이미지의 크기 또한 커진다.  
완벽한 OS를 생성할 수 있다는 장점은 있지만, 일반 호스트에 비해 성능 손실이 있으며, 수 기가바이트에 달하는 가상 머신 이미지를 애플리케이션으로 배포하기엔 부담스럽다는 단점이 있다.

이에 비해 도커 컨테이너는 **가상화된 공간을 생성하기 위해 linux 자체 기능인 chroot, namespace, cgroup을 사용함으로써 프로세스 단위의 격리 환경을 만들기 때문에 성능 손실이 거의 없다.** 컨테이너에 필요한 커널은 호스트의 커널을 공유해 사용하고, 컨테이너 안에는 애플리케이션을 구동하는 데 필요한 라이브러리 및 실행 파일만 존재하기 때문에 컨테이너를 이미지로 만들었을 때 이미지의 용량 또한 가상 머신에 비해 대폭 줄어든다. 따라서 컨테이너를 이미지로 만들어 배포하는 시간이 가상 머신에 비해 빠르며, 가상화된 공간을 사용할 때의 성능 손실도 거의 없다는 장점이 있다.

## 도커를 시작해야 하는 이유
도커는 컨테이너 생태계에서 사실상 표준으로서 사용되고 있다. 쿠버네티스, 메소스와 같은 오픈소스 프로젝트에서도 도커를 기준으로 개발되고 있다.

### 애플리케이션의 개발과 배포가 편해진다.
서버를 부팅할 때 실행되는 OS를 일반적으로 Host OS라고 부르며, 도커 컨테이너는 Host OS 위에서 실행되는 격리된 공간이다. 컨테이너 자체에 특별한 권한을 주지 않는 한, 컨테이너 내부에서 수많은 SW를 설치하고 설정 파일을 수정해도 Host OS엔 영향을 끼치지 않는다. 즉, 독립된 개발 환경을 보장받을 수 있다는 것이다.

**컨테이너 내부에서 여러 작업을 마친 뒤 이를 운영 환경에 배포하려 한다면, 해당 컨테이너를 '도커 이미지'라고 하는 일종의 패키지로 만들어 운영 서버에 전달하기만 하면 된다.**
- 컨테이너에서 사용되던 운영 서버에서 새롭게 패키지를 설치할 필요가 없다.
- 각종 라이브러리 설치 등으로 인한 의존성을 걱정할 필요가 없다.
- 서비스를 개발했을 때 사용했던 환경을 다른 서버에서도 컨테이너로서 똑같이 복제할 수 있기 때문에 개발/운영 환경의 통합이 가능하다.
- 도커 이미지는 가상 머신의 이미지와 달리 커널을 포함하고 있지 않기 때문에, 이미지 크기가 그다지 크지 않다.
- 이미지 내용을 레이어 단위로 구성하며, 중복되는 레이어를 재사용할 수 있어서 애플리케이션의 배포 속도가 매우 빨라진다.

### 여러 애플리케이션의 독립성과 확장성이 높아진다.
모놀리스 방식에선 서비스의 기능이 복잡해지고 거대해질수록 소프트웨어 자체의 확장성과 유연성이 줄어든다는 단점이 있다. 이를 대체하기 위해 마이크로서비스 구조를 사용하곤 한다.

![image](https://github.com/alanhakhyeonsong/LetsReadBooks/assets/60968342/8f2f4a5e-a2e7-4ebb-badb-9eb3c2d1941c)

마이크로서비스 구조는 여러 모듈을 독립된 형태로 구성하기 때문에 언어에 종속되지 않고 변화에 빠르게 대응할 수 있으며, 각 모듈의 관리가 쉬워진다는 장점이 있다.  
**컨테이너는 수 초 내로 생성, 시작이 가능할 뿐만 아니라 여러 모듈에게 독립된 환경을 동시에 제공할 수 있기 때문에 마이크로서비스 구조에서 가장 많이 사용되고 있는 가상화 기술이다.**

컨테이너 기반의 마이크로서비스는 개발자가 그 구조를 직접 구현하기보단 도커 스웜 모드, 쿠버네티스 등의 컨테이너 오케스트레이션 플랫폼을 통해 사용하는 것이 일반적이다.