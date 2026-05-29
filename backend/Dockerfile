# 첫 번째 스테이지: 빌드 스테이지
FROM gradle:jdk-21-and-23-graal-jammy AS builder

# 작업 디렉토리 설정
WORKDIR /app

# 1. 설정 파일만 먼저 복사 (캐싱 활용)
COPY build.gradle.kts settings.gradle.kts ./

# 2. Gradle 래퍼 생성
RUN gradle wrapper

# 3. [수정] 소스 코드 없이 의존성 라이브러리만 미리 다운로드하여 캐싱
# (bootJar가 실행되지 않도록 compileJava 관련 명령만 수행하며, 실패해도 무시하도록 || true 처리)
RUN ./gradlew compileJava compileKotlin --no-daemon || true

# 4. [수정] 실제 자바 소스 코드를 빌드 직전에 복사!
COPY src src

# 5. [수정] 메인 클래스가 포함된 최종 애플리케이션 빌드 (테스트는 제외)
RUN ./gradlew bootJar --no-daemon -x test

# 이후 명령어가 편하도록 불필요한 파일 삭제 (bootJar만 쓰면 plain은 보통 안 생기지만 안전상 유지)
RUN rm -rf /app/build/libs/*-plain.jar


# 두 번째 스테이지: 실행 스테이지
FROM container-registry.oracle.com/graalvm/jdk:23

# 작업 디렉토리 설정
WORKDIR /app

# 첫 번째 스테이지에서 빌드된 JAR 파일 복사
COPY --from=builder /app/build/libs/*.jar app.jar

# 실행할 JAR 파일 지정
ENTRYPOINT ["java", "-jar", "-Dspring.profiles.active=prod", "app.jar"]