# <p> 🐳 Spring Boot Docker 이미지 최적화 
"스프링 부트 기반 채팅 서버의 **도커라이즈 및 성능 개선**"

<br>

## 👨‍👨‍👧 개발팀원  

| <img src="https://avatars.githubusercontent.com/u/65991884?v=4" width="80"> | <img src="https://avatars.githubusercontent.com/u/90691610?v=4" width="80"> | <img src="https://avatars.githubusercontent.com/u/79312705?v=4" width="80"> |
|:---:|:---:|:---:|
| [류채현](https://github.com/RyuChaeHyun) | [이석철](https://github.com/SeokCheol-Lee) | [김상민](https://github.com/isshomin) |


<br>

## 📌 개요

이 프로젝트는 Docker 이미지를 최적화하여 **빌드 시간 단축과 이미지 크기 감소**를 목표로 진행되었습니다. **다양한 최적화 기법을 적용**하여 배포 과정의 효율성을 극대화하고, 최종적으로 더 가볍고 빠른 이미지를 생성했습니다.

<br>

## 🎖️ 주요 기능

1. **이미지 크기 감소**
   - 무겁고 불필요한 파일을 제거하여 최종 이미지 크기를 최소화
3. **빌드 시간 단축**
   - 최적화된 빌드 프로세스를 통해 빠른 개발 및 배포
4. **최적화된 캐시 사용**
   - 멀티 스테이지 빌드와 캐시 전략을 활용하여 반복적인 빌드에서 성능을 개선
5. **보안 강화**
   - 불필요한 빌드 단계와 파일을 제거함으로써 보안도 함께 강화

<br>

## 📊 프로젝트 과정
**Docker 이미지 최적화**를 목표로, 다양한 최적화 방법들을 적용해보고 성능을 측정하여 최적의 결과를 도출했습니다.

<br>

### 1️⃣ 최초 빌드
기본적으로 `openjdk:21` 이미지를 사용하여 Spring Boot 애플리케이션을 배포합니다. 초기 이미지의 빌드 시간과 크기는 다음과 같습니다.

#### Dockerfile
```dockerfile
FROM openjdk:21

RUN mkdir -p deploy
WORKDIR /deploy

COPY ./build/libs/chat-server-0.0.1-SNAPSHOT.jar chat-server.jar

ENTRYPOINT ["java","-jar","/deploy/chat-server.jar"]

```

**빌드 시간**: 14.2s
  <p align="left"><img src="https://github.com/user-attachments/assets/673819c3-bc00-4ea1-933d-b8d9efaa5187"></p>
  
**이미지 크기**: 289MB
  <p align="left"><img src="https://github.com/user-attachments/assets/7b0c879f-285d-4509-bb87-39eefc3fe8ea"></p>

<br>

### 2️⃣ 경량 JDK 이미지로 변경

기존 `openjdk:21` 이미지는 전체 JDK를 포함하고 있어 크기가 큽니다. **JRE**만 필요한 경우, 더 작은 이미지를 사용해 최적화할 수 있습니다.

#### Dockerfile
```dockerfile
FROM eclipse-temurin:21-jre-alpine
```

**변경 후 빌드 시간**: 7.0s
<p align="left"><img src="https://github.com/user-attachments/assets/8633a31b-e0b5-478c-ad07-13c699615669"></p>
  
**변경 후 이미지 크기**: 89MB
<p align="left"><img src="https://github.com/user-attachments/assets/e4180cf6-125a-458c-b5c8-5c24569a4eff"></p>

<br>

**이미지 크기 변화**
<p align="left"><img src="https://github.com/user-attachments/assets/11956879-b62c-45f1-a278-5bc316dc29e3"></p>
이미지 크기가 589mb 에서 277mb 로 약 53% 감소하였습니다.

**빌드 속도 변화**
빌드 속도가 14.2s 에서 7.0s로 약 50% 감소하였습니다.


<br>
<br>

### 3️⃣ 멀티 스테이지 빌드 도입

**빌드 단계와 실행 단계를 분리**하여 빌드 성능을 최적화하고 불필요한 파일을 제거합니다.

#### Dockerfile
```dockerfile

# 빌드 스테이지
FROM gradle:7.2.0-jdk17 AS build
COPY --chown=gradle:gradle . /home/gradle/src
WORKDIR /home/gradle/srcRUN gradle build --no-daemon

# 실행 스테이지
FROM eclipse-temurin:21-jre-alpine
EXPOSE 8080RUN mkdir /app
COPY --from=build /home/gradle/src/build/libs/*.jar /app/spring-boot-application.jar
ENTRYPOINT ["java","-jar","/app/spring-boot-application.jar"]

```
<br>

**스테이지 적용 직후 빌드 시간**: 22.3s <br>
  prune으로 이전 데이터들을 다 날리고 동일한 조건 하에서 스테이지 적용 후 도커 파일을 실행하였습니다. 
  <p align="left"><img src="https://github.com/user-attachments/assets/5dbd9426-2928-49e7-b17b-d4654c617c66"></p>
  
  <br>

  **이미지 크기**: 88MB <br>
  
 <p align="left"><img src="https://github.com/user-attachments/assets/6908b95b-0080-4bf7-8275-3e90233ed49a"></p>
 
**코드 수정 후 빌드 시간**: 2.5s <br>
  <p align="left"><img src="https://github.com/user-attachments/assets/5d3e9fe7-2429-4168-9ed3-9b794f2318af"></p>
  
 멀티스테이지에서 코드를 변경한 후에 빌드한 결과입니다.
 
<br>

**빌드 및 이미지 크기 변화**
<p align="left"><img src="https://github.com/user-attachments/assets/c7024965-6fad-4551-a6ff-550a007d9b24"></p>
빌드 속도가 14.2s-> 22.s-> 8.6s로 감소하였습니다. 
<br>

---

### **❓ 멀티 스테이지 빌드 시 처음에 시간이 더 걸리는 이유?**

멀티 스테이지 빌드는 초기에는 이미지 레이어 생성, 캐시 미사용, 복잡한 빌드 프로세스 등으로 시간이 더 걸릴 수 있습니다. 하지만 **최종 이미지 크기 감소**, **보안 강화**, **구조화된 빌드 프로세스** 등의 장점을 제공하며, **빌드 캐시 최적화**, **.dockerignore 사용**, **경량 베이스 이미지 활용** 등을 통해 성능을 개선할 수 있습니다. 결과적으로, 초기 시간 증가는 있지만 장기적으로 **개발 및 배포 효율성**을 높이는 방법입니다.

---

### 4️⃣ 스프링 부트 레이어드 JAR 사용

스프링 부트의 **레이어드 JAR** 기능을 사용하여 애플리케이션의 빌드 성능을 더욱 최적화할 수 있습니다. 이 방식은 애플리케이션을 여러 레이어로 나누어 캐싱할 수 있게 해줍니다.

#### Dockerfile
```dockerfile
# 빌드 스테이지
FROM eclipse-temurin:21-jre-alpine as builder
WORKDIR application
ARG JAR_FILE=build/libs/*.jar
COPY ${JAR_FILE} application.jar
RUN java -Djarmode=layertools -jar application.jar extract

# 실행 스테이지
FROM eclipse-temurin:21-jre-alpine
WORKDIR application
COPY --from=builder application/dependencies/ ./
COPY --from=builder application/spring-boot-loader/ ./
COPY --from=builder application/snapshot-dependencies/ ./
COPY --from=builder application/application/ ./
ENTRYPOINT ["java", "org.springframework.boot.loader.JarLauncher"]
```


**레이어드 적용 직후 빌드 시간**: 8.6s
  <p align="left"><img src="https://github.com/user-attachments/assets/94ea760d-8387-4a01-a217-443f427457b9"></p>
  prune으로 기존 데이터를 모두 날린 후 진행하였으며, 레이어드를 적용한 직후 빌드한 화면입니다.


  <br>
  <br>
  
  **이미지 크기**: 88MB
  <p align="left"><img src="https://github.com/user-attachments/assets/60b97b54-cf32-4196-948d-73307d560377"></p>

**코드 수정 후 빌드 시간**: 0.8s
  
  <p align="left"><img src="https://github.com/user-attachments/assets/a9ccc166-040c-4a15-bc62-e3d6a8d72602"></p>
  레이어드를 적용한 상태에서 코드를 수정하고 다시 빌드한 화면입니다.
<br>
<br>

**빌드 및 이미지 크기 변화**
<p align="left"><img src="https://github.com/user-attachments/assets/7b9cbeef-6265-40c3-b2aa-2b99ae2a8d70"></p>
빌드속도가 2.5s -> 0.8s로 줄어들었습니다. 

<br>

## 🧐 결론

이번 프로젝트를 통해 Docker 이미지 크기를 최적화하는 과정에서 멀티 스테이지 빌드, 이미지 변경, 레이어드 JAR 방식을 도입하여 빌드 시간을 **14.2초에서 0.8초**로 대폭 줄일 수 있었습니다. 

특히, 멀티 스테이지 빌드를 활용하여 불필요한 파일과 패키지를 배제하고 경량 베이스 이미지를 선택함으로써 이미지 크기와 빌드 시간이 획기적으로 감소했습니다. 결과적으로 배포 과정이 더 효율적이고 빠르게 이루어졌으며, 리소스 사용량 또한 최적화되었습니다.


<br>

## 🤔 아쉬웠던 점
- **.dockerignore 파일**: 로그 파일이나 기타 불필요한 파일을 빌드에서 제외하기 위해 .dockerignore 파일을 시도해 보았으나, 현재 프로젝트에서 로그나 기타 불필요한 파일들이 많지 않아서 크리티컬한 영향을 미치지 않았습니다. 따라서 최종 빌드에서는 제외했지만, 향후 로그가 많아질 경우 적용을 고려할 필요가 있습니다.
- **초기 빌드 시간**: 멀티 스테이지 빌드를 적용함으로써 처음 빌드 시간은 다소 길어졌습니다. 그러나 최적화된 캐시 전략을 통해 반복적인 빌드에서 큰 효과를 기대할 수 있습니다.

<br>
