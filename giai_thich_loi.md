# PHÂN TÍCH VÀ KHẮC PHỤC SỰ CỐ PIPELINE GITLAB CI

## 1. Phân tích nguyên nhân lỗi trong file cấu hình ban đầu:
Đoạn mã cấu hình ban đầu:
```yaml
stages:
build_app
build_job:
stage:     build_app
script:
- ./gradlew clean build -x test
```

### Các lỗi và thiếu sót nghiêm trọng:
1. **Lỗi cú pháp YAML ở từ khóa `stages`:**
   - Trong cú pháp YAML của GitLab CI, `stages` phải được định nghĩa dưới dạng một danh sách (list/array). Cú pháp đúng phải có dấu gạch đầu dòng (`-`) và thụt lề chuẩn:
     ```yaml
     stages:
       - build_app
     ```
   - Việc viết `build_app` ngay dưới `stages:` không thụt dòng và không có `-` khiến parser báo lỗi cú pháp YAML (YAML Syntax Error / Invalid CI Configuration).

2. **Thiếu định nghĩa môi trường chạy (`image`):**
   - GitLab Runner mặc định sử dụng image cơ sở như `ruby:latest` hoặc container rỗng không cài đặt sẵn môi trường Java Development Kit (JDK) và Gradle.
   - Khi chạy lệnh `./gradlew clean build -x test`, hệ thống không tìm thấy lệnh `java` hoặc trình thông dịch thực thi, dẫn đến lỗi:
     `bash: line 1: ./gradlew: command not found` (hoặc `ERROR: JAVA_HOME is not set`).
   - Cần bổ sung `image: gradle:8.5-jdk17-alpine` hoặc `eclipse-temurin:17-jdk-alpine`.

3. **Thiếu cấp quyền thực thi cho Gradle Wrapper:**
   - Để tránh lỗi `Permission denied (exit code 126)`, cần bổ sung bước cấp quyền `chmod +x gradlew` trong `before_script`.

---

## 2. File cấu hình `.gitlab-ci.yml` chuẩn xác sau khi sửa:
```yaml
image: gradle:8.5-jdk17-alpine

stages:
  - build_app

before_script:
  - chmod +x gradlew

build_job:
  stage: build_app
  script:
    - ./gradlew clean build -x test
  artifacts:
    paths:
      - build/libs/*.jar
    expire_in: 1 week
```
