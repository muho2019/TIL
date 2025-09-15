# Dockerfile

`Dockerfile`은 Docker 이미지를 만들기 위한 상세한 '조리법' 또는 '조립 설명서'입니다.

이 텍스트 파일 안에는 기반이 될 재료(베이스 이미지)부터 시작해서, 어떤 재료를 추가하고(파일 복사), 어떻게 조리할지(명령어 실행) 등 이미지를 완성하기까지의 모든 과정이 순서대로 적혀 있습니다.

## 주요 명령어 (조리법 순서)

Dockerfile은 보통 아래와 같은 명령어들로 구성됩니다.

- `FROM`: (재료 준비)
  - 어떤 베이스 이미지에서 시작할지 지정합니다. 모든 `Dockerfile`의 첫 줄은 반드시 `FROM`으로 시작해야 합니다.
  - `FROM openjdk:17-jdk-slim` → "Java 17이 설치된 가벼운 리눅스 환경을 준비해 줘."
- `WORKDIR`: (작업 공간 확보)
  - 컨테이너 안에서 명령어를 실행할 기본 디렉토리를 설정합니다.
  - `WORKDIR /app` → "이제부터 컨테이너 안의 `/app` 폴더에서 모든 작업을 할 거야."
- `COPY`: (재료 추가)
  - 호스트 PC의 파일이나 폴더를 컨테이너 안(이미지)으로 복사합니다.
  - `COPY build/libs/*.jar app.jar` → "내 PC의 `build/libs/` 폴더에 있는 `.jar` 파일을 컨테이너의 현재 작업 공간(`/app`)에 `app.jar`라는 이름으로 복사해 줘."
- `RUN`: (중간 조리 과정)
  - 이미지를 빌드하는 과정에서 필요한 스크립트나 명령어를 실행합니다. 주로 패키지 설치, 디렉토리 생성 등에 사용됩니다.
  - `RUN apt-get update && apt-get install -y vim` → "(빌드 중에) vim 에디터를 설치해 줘."
- `EXPOSE`: (사용법 안내)
  - 이 컨테이너가 실행될 때, 외부와 통신하기 위해 사용할 포트 번호를 알려주는 '문서' 역할입니다. 실제로 포트를 외부에 개방하지는 않으며, 개발자에게 어떤 포트를 사용해야 하는지 알려주는 가이드 역할을 합니다.
  - `EXPOSE 8080` → "이 컨테이너는 8080 포트를 사용할 예정이야."
- `CMD` 또는 `ENTRYPOINT`: (완성된 요리를 내놓는 방법)
  - 이미지로 컨테이너를 시작할 때 실행될 기본 명령어를 지정합니다. 애플리케이션을 실행하는 명령어가 여기에 들어갑니다.
  - `CMD ["java", "-jar", "app.jar"]` → "컨테이너가 시작되면, java -jar app.jar 명령어를 실행해서 애플리케이션을 구동해."

## Spring Boot 앱을 위한 Dockerfile 예시

아래는 user-service 모듈을 위한 간단한 Dockerfile 예시입니다.

[module-user-service/Dockerfile]

```dockerfile
# 1. 베이스 이미지로 Java 17 버전을 준비한다.
FROM openjdk:17-jdk-slim

# 2. 컨테이너 내부의 작업 디렉토리를 /app으로 설정한다.
WORKDIR /app

# 3. 호스트 PC에서 빌드된 jar 파일을 컨테이너 내부로 복사한다.
#    (Dockerfile을 실행하기 전에 ./gradlew build를 통해 jar 파일이 생성되어 있어야 함)
COPY build/libs/user-service-0.0.1-SNAPSHOT.jar app.jar

# 4. 이 애플리케이션은 8080 포트를 사용한다고 명시한다.
EXPOSE 8080

# 5. 컨테이너가 시작될 때 이 명령어를 실행하여 애플리케이션을 구동한다.
ENTRYPOINT ["java", "-jar", "app.jar"]
```

## 더 나은 방법: 멀티 스테이지 빌드 (Multi-Stage Builds)

위 예시는 내 PC에 Java와 Gradle이 설치되어 있어서 미리 `*.jar` 파일을 빌드해야 한다는 단점이 있습니다. 멀티 스테이지 빌드를 사용하면, 오직 Docker만으로 소스 코드를 빌드하고 최종 이미지를 만드는 모든 과정을 한 번에 처리할 수 있습니다.

```dockerfile
# --- 1단계: 빌드 스테이지 (Builder Stage) ---
# Gradle과 JDK가 설치된 이미지를 빌더로 사용
FROM gradle:jdk17-jammy as builder

# 소스 코드를 컨테이너로 복사
WORKDIR /build
COPY . .

# Gradle을 사용하여 애플리케이션을 빌드 (jar 파일 생성)
RUN gradle build

# --- 2단계: 최종 스테이지 (Final Stage) ---
# 실제 실행에는 JDK가 아닌 JRE만으로 충분하므로 더 가벼운 이미지를 사용
FROM openjdk:17-jre-slim

WORKDIR /app

# 빌더 스테이지에서 생성된 jar 파일만 최종 이미지로 복사
COPY --from=builder /build/build/libs/*.jar app.jar

EXPOSE 8080

ENTRYPOINT ["java", "-jar", "app.jar"]
```
