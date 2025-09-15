# Docker Compose

Docker Compose는 여러 개의 Docker 컨테이너를 하나의 애플리케이션으로 묶어 정의하고 실행하는 도구입니다.

복잡한 요리를 할 때 필요한 '총괄 레시피'에 비유할 수 있습니다.

- Dockerfile: 재료 하나(예: 파스타 면)를 준비하는 레시피
- Docker Image: 준비된 재료(익힌 파스타 면)
- Docker Compose: 이 재료들(파스타 면, 소스, 채소)을 모아 '알리오 올리오'라는 하나의 완성된 요리를 만드는 전체 과정을 지휘하는 총괄 레시피

## 왜 필요한가요?

현대의 많은 애플리케이션은 웹 서버, 데이터베이스, 캐시 서버 등 여러 컴포넌트가 함께 동작해야 합니다. Docker를 사용하면 이들을 각각 컨테이너로 띄울 수 있지만, Docker Compose가 없다면 다음과 같이 매우 번거로운 과정을 거쳐야 합니다.

1. 공용 네트워크 생성: `docker network create my-app-network`
2. DB 컨테이너 실행: `docker run --name mysql-db --network my-app-network -e MYSQL_ROOT_PASSWORD=1234 -v db-data:/var/lib/mysql -d mysql:8.0`
3. 애플리케이션 컨테이너 실행: `docker run --name my-app --network my-app-network -p 8080:8080 -e DB_HOST=mysql-db -d my-app-image`

이 방식은 명령어가 길고 복잡하며, 컨테이너 간의 의존 관계를 한눈에 파악하기 어렵습니다. Docker Compose는 이 모든 설정을 하나의 파일로 깔끔하게 관리하게 해줍니다.

## 어떻게 동작하나요?: `docker-compose.yml` 파일

Docker Compose는 `docker-compose.yml`이라는 YAML 형식의 설정 파일을 사용합니다. 이 파일에 애플리케이션을 구성하는 모든 서비스(컨테이너)의 설정과 관계를 정의합니다.

`docker-compose.yml` 예시

```yaml
# docker-compose.yml

# 사용하는 Docker Compose 파일 버전
version: '3.8'

# 애플리케이션을 구성하는 서비스(컨테이너)들 정의
services:
  # 1. Spring Boot 애플리케이션 서비스
  app:
    # 현재 디렉토리의 Dockerfile을 사용하여 이미지를 빌드
    build: .
    # 컨테이너 이름을 my-app으로 지정
    container_name: my-app
    # app이 db 서비스에 의존함을 명시 (db가 먼저 실행됨)
    depends_on:
      - db
    # 호스트의 8080 포트와 컨테이너의 8080 포트를 연결
    ports:
      - '8080:8080'
    # 환경 변수 설정 (db 서비스의 컨테이너 이름인 'db'를 호스트로 사용)
    environment:
      - DB_HOST=db
      - DB_USERNAME=user
      - DB_PASSWORD=1234

  # 2. MySQL 데이터베이스 서비스
  db:
    # mysql 8.0 버전을 공식 이미지로 사용
    image: mysql:8.0
    container_name: mysql-db
    # 데이터 영속성을 위해 호스트의 볼륨과 컨테이너의 폴더를 연결
    volumes:
      - db-data:/var/lib/mysql
    # DB 초기화에 필요한 환경 변수
    environment:
      - MYSQL_DATABASE=mydatabase
      - MYSQL_USER=user
      - MYSQL_PASSWORD=1234
      - MYSQL_ROOT_PASSWORD=root

# 데이터 영속성을 위한 Docker 볼륨 정의
volumes:
  db-data:
```

**핵심 설정:**

- `services`: 실행하려는 각 컨테이너를 서비스 단위로 정의합니다. 위 예시에서는 `app`과 `db` 두 개의 서비스가 있습니다.
- `build` / `image`: 컨테이너를 만들 이미지를 지정합니다. (`build`: Dockerfile로부터 직접 빌드, `image`: Docker Hub 등에서 이미지 다운로드)
- `ports`: 호스트 PC와 컨테이너 간의 포트를 연결합니다.
- `environment`: 컨테이너 내부에서 사용할 환경 변수를 설정합니다.
- `volumes`: 데이터를 영속적으로 저장하기 위해 컨테이너의 데이터를 호스트 PC에 저장합니다.
  - 최상단 `volumes`: Docker Compose가 관리할 '네임드 볼륨(Named Volume)'을 선언하는 부분입니다.
  - 서비스 내 `volumes`: 해당 서비스 컨테이너에 볼륨을 마운트하는 설정입니다.
- `depends_on`: 서비스 간의 실행 순서를 제어합니다.

## 주요 명령어

이 docker-compose.yml 파일이 있는 디렉토리에서 다음 명령어들을 사용합니다.

- `docker-compose up`

파일에 정의된 모든 서비스를 생성하고 시작합니다. `-d` 옵션을 붙이면 백그라운드에서 실행합니다.

```bash
docker-compose up -d
```

- `docker-compose down`

`up`으로 실행한 모든 컨테이너와 네트워크를 중지하고 제거합니다. `-v` 옵션을 붙이면 볼륨까지 함께 삭제합니다.

```bash
docker-compose down -v
```

- `docker-compose ps`

현재 실행 중인 서비스(컨테이너)들의 상태를 보여줍니다.

```bash
docker-compose ps
```

- `docker-compose logs -f [서비스명]`

특정 서비스의 실시간 로그를 확인합니다.

```bash
docker-compose logs -f app
```

## 장점

### 1. 일관된 개발 환경: "내 컴퓨터에선 됐는데..." 문제 해결

가장 큰 장점입니다. 팀원 A는 M1 맥북에 MySQL 8.1을 쓰고, 팀원 B는 윈도우에 MariaDB를 쓴다고 가정해 보세요. 개발 환경이 다르기 때문에, A에게는 잘 되던 코드가 B에게는 에러가 날 수 있습니다.

Docker Compose는 `docker-compose.yml` 파일에 "**이 앱은 Ubuntu 22.04 기반의 Java 17과 MySQL 8.0 환경에서만 동작한다**"고 명시해 버립니다. 그러면 모든 팀원의 컴퓨터에서 `docker-compose up` 명령 한 번으로 완벽하게 동일한 환경이 구축되어, 환경 차이로 인한 버그가 원천적으로 사라집니다.

### 2. 간편한 환경 구축: 명령어 한 줄로 끝!

새로운 팀원이 프로젝트에 합류하면, 보통 복잡한 개발 환경 세팅 가이드를 따라야 합니다. (JDK 설치, DB 설치 및 설정, 환경 변수 등록 등)

Docker Compose를 사용하면 `git clone` 받은 후 `docker-compose up` 명령어 한 줄이면 끝납니다. 10분 안에 앱과 DB가 모두 준비되어 바로 개발을 시작할 수 있죠. 이는 개발 생산성을 극적으로 향상시킵니다.

### 3. 완벽한 격리: 내 PC는 깨끗하게

Docker Compose로 띄운 앱과 DB는 격리된 가상 네트워크 안에서 서로 통신합니다.

- 포트 충돌 방지: A 프로젝트도 3306 포트의 MySQL을 쓰고, B 프로젝트도 3306 포트의 MySQL을 써야 할 때, 내 PC에 MySQL을 직접 설치했다면 둘 중 하나는 포트를 바꿔야 합니다. Docker Compose 환경에서는 각 프로젝트가 자신만의 격리된 DB를 가지므로 전혀 충돌이 없습니다.
- 깨끗한 호스트 환경: 내 PC에 Java, MySQL, Redis 등을 직접 설치할 필요가 없습니다. 프로젝트를 지우고 싶으면 `docker-compose down` 명령 한 번으로 관련된 모든 것을 흔적 없이 삭제할 수 있습니다.

### 4. 손쉬운 이식성 및 협업

`docker-compose.yml` 파일은 실행 가능한 인프라 설명서입니다. 이 파일만 있으면 내 로컬 PC, 동료의 PC, 테스트 서버, 심지어 운영 서버까지 어디서든 동일한 환경을 재현할 수 있습니다. 이는 CI/CD 파이프라인을 구축하거나, 다른 팀과 협업할 때 매우 강력한 장점이 됩니다.

## 한계

Docker Compose는 **근본적으로 단일 호스트(서버 한 대)에서 사용하는 도구**입니다.

서버가 여러 대로 확장되는 순간부터는 Docker Compose만으로는 한계가 있으며, **컨테이너 오케스트레이션**(Container Orchestration)이라는 기술이 필요해집니다.

### Docker Compose의 한계: 단일 레스토랑 점장

Docker Compose는 훌륭한 '**단일 레스토랑 점장**'입니다. 한 레스토랑 안에서 주방(DB), 홀(App) 직원들을 완벽하게 지휘하여 운영할 수 있습니다.

하지만 이 점장에게 "서울, 부산, 광주에 레스토랑 10개를 동시에 운영하고, 서울에 손님이 몰리면 부산 인력을 지원 보내!" 라고 지시할 수는 없습니다. 이것은 점장의 역할을 넘어선 '**프랜차이즈 본사**'의 역할이기 때문입니다.

### 해결책: 컨테이너 오케스트레이션 (프랜차이즈 본사)

여러 서버(클러스터)에 분산된 수많은 컨테이너들을 마치 하나의 시스템처럼 지휘하고 관리하는 기술을 컨테이너 오케스트레이션이라고 합니다. 이 분야의 사실상 표준 기술이 바로 **쿠버네티스**(Kubernetes)입니다.

오케스트레이션 도구는 '프랜차이즈 본사'처럼 다음과 같은 일들을 자동으로 처리해 줍니다.

- 스케줄링 (Scheduling): 여러 서버 중 가장 여유 있는 곳을 찾아 알아서 컨테이너를 배치합니다. (가장 효율적인 곳에 신규 매장 오픈)
- 스케일링 (Scaling): 트래픽이 몰리면 앱 컨테이너 수를 자동으로 3개, 5개로 늘리고, 트래픽이 줄면 다시 줄입니다. (점심시간에만 알바생 증원)
- 고가용성 및 자가 치유 (High Availability & Self-Healing): 서버 한 대가 갑자기 다운되어 앱 컨테이너가 죽으면, 즉시 다른 건강한 서버에 컨테이너를 자동으로 다시 띄웁니다. (A매장이 문 닫으면, 자동으로 B매장으로 손님 안내 및 A매장 복구)
- 로드 밸런싱 (Load Balancing): 여러 개의 앱 컨테이너에 들어오는 요청을 골고루 분산하여 과부하를 막아줍니다.

### 그럼 Docker Compose는 쓸모가 없나요?

아닙니다. 두 도구는 사용되는 목적과 환경이 다릅니다.

- Docker Compose: 개발 환경과 단일 서버 운영의 왕입니다.
  - 개발자가 자신의 PC에서 전체 애플리케이션 스택을 가장 빠르고 간편하게 실행하고 테스트하는 데 사용됩니다. 우리가 이야기했던 '일체형 개발 환경 세트'로서의 가치는 여전히 막강합니다.
- Kubernetes: **운영 환경**(프로덕션)과 여러 서버 운영의 표준입니다.
  - 여러 서버에 걸쳐 애플리케이션을 무중단으로, 안정적으로, 확장 가능하게 운영해야 할 때 사용됩니다.

보통 "**개발은 Docker Compose로 편하게 하고, 배포는 Kubernetes로 견고하게 한다**"는 흐름으로 많이 사용됩니다.

## MSA에서의 활용 예시

`user-service`와 `user-db`, 그리고 `order-service`와 `order-db`가 각각 존재하는 `docker-compose.yml` 파일의 예시입니다.

```yaml
# docker-compose.yml (프로젝트 루트 경로)

version: '3.8'

services:
  # --- 유저 도메인 ---
  user-service:
    build: ./module-user-service
    container_name: user-service
    ports:
      - '8081:8080' # 호스트의 8081 포트와 연결
    environment:
      - DB_HOST=user-db # 유저 서비스 전용 DB를 바라보도록 설정
      - DB_USERNAME=user
      - DB_PASSWORD=userpass
    depends_on:
      - user-db

  user-db:
    image: mysql:8.0
    container_name: user-db
    volumes:
      - user-db-data:/var/lib/mysql # 유저 DB 전용 볼륨
    environment:
      - MYSQL_DATABASE=user_db
      - MYSQL_USER=user
      - MYSQL_PASSWORD=userpass
      - MYSQL_ROOT_PASSWORD=root

  # --- 주문 도메인 ---
  order-service:
    build: ./module-order-service
    container_name: order-service
    ports:
      - '8082:8080' # 호스트의 8082 포트와 연결 (포트 충돌 방지)
    environment:
      - DB_HOST=order-db # 주문 서비스 전용 DB를 바라보도록 설정
      - DB_USERNAME=order
      - DB_PASSWORD=orderpass
      - USER_SERVICE_URL=http://user-service:8080 # 서비스 디스커버리
    depends_on:
      - order-db
      - user-service # 주문 서비스는 유저 서비스에 의존

  order-db:
    image: mysql:8.0
    container_name: order-db
    volumes:
      - order-db-data:/var/lib/mysql # 주문 DB 전용 볼륨
    environment:
      - MYSQL_DATABASE=order_db
      - MYSQL_USER=order
      - MYSQL_PASSWORD=orderpass
      - MYSQL_ROOT_PASSWORD=root

volumes:
  user-db-data:
  order-db-data:
```

### env 파일 활용

환경 변수를 `docker-compose.yml` 파일에 직접 쓰는 대신, `.env` 파일을 만들어 관리할 수도 있습니다. 이렇게 하면 민감한 정보(예: DB 비밀번호)를 코드에서 분리할 수 있어 보안에 유리합니다.

`.env` 파일 예시:

```env
# .env
USER_DB_PASSWORD=userpass
ORDER_DB_PASSWORD=orderpass
```

`docker-compose.yml` 파일에서 환경 변수를 참조하는 방법:

```yaml
environment:
  - DB_PASSWORD=${USER_DB_PASSWORD}
```
