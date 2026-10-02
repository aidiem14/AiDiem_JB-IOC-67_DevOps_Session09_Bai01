# Bài tập 1 – Khắc phục sự cố Pipeline CI/CD cơ bản

## 1. Thông tin bài tập

- **Session:** 09
- **Bài tập:** 1 – Khắc phục sự cố Pipeline CI/CD cơ bản
- **Chủ đề:** GitLab CI/CD và Gradle
- **Mục tiêu:** Phân tích lỗi cấu hình `.gitlab-ci.yml` và sửa pipeline để chạy thành công lệnh:

```bash
./gradlew clean build -x test
```

---

## 2. Nội dung file `.gitlab-ci.yml` ban đầu

File cấu hình ban đầu:

```yaml
stages:
  build_app

build_job:
  stage: build_app
  script:
    - ./gradlew clean build -x test
```

---

## 3. Phân tích các lỗi trong file cấu hình ban đầu

### 3.1. Lỗi thứ nhất: Khai báo `stages` sai cú pháp YAML

File ban đầu sử dụng:

```yaml
stages:
  build_app
```

`stages` phải được khai báo dưới dạng danh sách (list) trong YAML. Vì vậy, cần có dấu `-` trước `build_app`.

Cú pháp đúng:

```yaml
stages:
  - build_app
```

Như vậy, `build_app` được xác định là một phần tử của danh sách `stages`.

---

### 3.2. Lỗi thứ hai: Thiếu Docker image chứa môi trường Java/Gradle

Pipeline thực hiện lệnh:

```bash
./gradlew clean build -x test
```

Tuy nhiên, file cấu hình ban đầu không khai báo `image` để cung cấp môi trường thực thi phù hợp.

Bài tập yêu cầu môi trường có **Gradle và JDK 17**, vì vậy cần bổ sung Docker image phù hợp, ví dụ:

```yaml
image: gradle:8.5-jdk17
```

Image này được dùng làm môi trường để job thực hiện các lệnh build Gradle.

---

### 3.3. Lỗi thứ ba: Gradle Wrapper có thể chưa có quyền thực thi

Trong môi trường Linux của GitLab Runner, file `gradlew` cần có quyền thực thi để có thể chạy trực tiếp bằng:

```bash
./gradlew
```

Để tránh lỗi quyền thực thi, bổ sung bước:

```bash
chmod +x ./gradlew
```

trước khi chạy Gradle Wrapper.

---

## 4. File `.gitlab-ci.yml` sau khi sửa

File hoàn chỉnh:

```yaml
image: gradle:8.5-jdk17

stages:
  - build_app

build_job:
  stage: build_app
  script:
    - chmod +x ./gradlew
    - ./gradlew clean build -x test
```

---

## 5. Giải thích file cấu hình sau khi sửa

### 5.1. `image`

```yaml
image: gradle:8.5-jdk17
```

Khai báo Docker image được sử dụng làm môi trường thực thi cho pipeline.

Mục đích là cung cấp môi trường có Gradle và JDK 17 để thực hiện quá trình build ứng dụng.

---

### 5.2. `stages`

```yaml
stages:
  - build_app
```

Khai báo pipeline có một stage:

```text
build_app
```

---

### 5.3. `build_job`

```yaml
build_job:
```

Đây là tên của job thực hiện quá trình build.

---

### 5.4. `stage`

```yaml
stage: build_app
```

Gắn job `build_job` vào stage `build_app`.

Tên stage phải khớp với tên đã khai báo bên dưới `stages`.

---

### 5.5. `script`

```yaml
script:
  - chmod +x ./gradlew
  - ./gradlew clean build -x test
```

Pipeline thực hiện hai lệnh theo thứ tự.

#### Lệnh 1

```bash
chmod +x ./gradlew
```

Cấp quyền thực thi cho Gradle Wrapper.

#### Lệnh 2

```bash
./gradlew clean build -x test
```

Thực hiện build project bằng Gradle Wrapper:

- `clean`: xóa các kết quả build cũ.
- `build`: biên dịch và đóng gói project.
- `-x test`: bỏ qua task kiểm thử `test`.

---

## 6. So sánh trước và sau khi sửa

### File ban đầu

```yaml
stages:
  build_app

build_job:
  stage: build_app
  script:
    - ./gradlew clean build -x test
```

Các vấn đề:

1. `stages` khai báo sai cú pháp danh sách YAML.
2. Không khai báo Docker image có Gradle/JDK 17.
3. Chưa đảm bảo `gradlew` có quyền thực thi.

### File sau khi sửa

```yaml
image: gradle:8.5-jdk17

stages:
  - build_app

build_job:
  stage: build_app
  script:
    - chmod +x ./gradlew
    - ./gradlew clean build -x test
```

---

## 7. Kết quả mong muốn

Sau khi cấu hình đúng, GitLab Runner sẽ:

1. Khởi tạo môi trường từ Docker image `gradle:8.5-jdk17`.
2. Thực hiện cấp quyền cho `gradlew`.
3. Chạy:

```bash
./gradlew clean build -x test
```

4. Pipeline hoàn thành thành công nếu project Gradle và mã nguồn không có lỗi build.

---

## 8. Điều kiện để pipeline chạy được

Project cần là một Gradle project và có Gradle Wrapper, thông thường gồm các file như:

```text
gradlew
gradlew.bat
build.gradle
settings.gradle
```

File `.gitlab-ci.yml` cũng cần được đặt ở thư mục gốc của project để GitLab CI có thể sử dụng cấu hình này.

---

## 9. Cách kiểm tra trước khi nộp bài

Trước khi push bài lên GitHub, kiểm tra nội dung file:

```bash
cat .gitlab-ci.yml
```

Nội dung phải tương ứng với:

```yaml
image: gradle:8.5-jdk17

stages:
  - build_app

build_job:
  stage: build_app
  script:
    - chmod +x ./gradlew
    - ./gradlew clean build -x test
```

Nếu môi trường local có thể chạy Gradle Wrapper, có thể kiểm tra bằng:

```bash
./gradlew clean build -x test
```

---

## 10. Cấu trúc bài nộp

Repository GitHub có thể tổ chức như sau:

```text
session09-bai01/
├── .gitlab-ci.yml
└── README.md
```

Trong đó:

- `.gitlab-ci.yml`: file cấu hình GitLab CI/CD đã sửa.
- `README.md`: phần phân tích lỗi và giải thích cách khắc phục.

---

## 11. Quy trình đưa bài lên GitHub

### Bước 1: Khởi tạo Git repository

```bash
git init
```

### Bước 2: Thêm các file

```bash
git add .
```

### Bước 3: Tạo commit

```bash
git commit -m "Complete Session09 Exercise01"
```

### Bước 4: Đổi tên branch thành `main`

```bash
git branch -M main
```

### Bước 5: Thêm remote GitHub

Ví dụ:

```bash
git remote add origin https://github.com/<username>/<repository>.git
```

### Bước 6: Push lên GitHub

```bash
git push -u origin main
```

---

## 12. Lưu ý về GitLab CI và GitHub

File `.gitlab-ci.yml` là file cấu hình cho **GitLab CI/CD**, không phải GitHub Actions.

Vì vậy, việc đưa `.gitlab-ci.yml` lên GitHub chủ yếu là để đáp ứng yêu cầu **nộp bài trên GitHub**. GitHub sẽ không tự động chạy `.gitlab-ci.yml` như một workflow GitHub Actions.

Muốn kiểm tra pipeline GitLab CI thực tế, cần sử dụng GitLab project và GitLab Runner tương ứng.

---

## 13. Kết luận

File `.gitlab-ci.yml` ban đầu gặp các vấn đề chính:

- khai báo `stages` không đúng cú pháp danh sách YAML;
- chưa khai báo môi trường Docker image có Gradle/JDK 17;
- chưa đảm bảo Gradle Wrapper có quyền thực thi.

Sau khi sửa, cấu hình sử dụng:

```yaml
image: gradle:8.5-jdk17

stages:
  - build_app

build_job:
  stage: build_app
  script:
    - chmod +x ./gradlew
    - ./gradlew clean build -x test
```

Cấu hình này đáp ứng yêu cầu của bài tập về việc sửa cú pháp YAML và chuẩn bị môi trường để chạy Gradle với JDK 17.
