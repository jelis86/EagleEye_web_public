# EagleEye Web — 제품·SDK·펌웨어 배포 포털

> **공개 범위:** 이 저장소에는 프로젝트 설명과 화면 캡처만 포함되어 있습니다. 소스 코드, 프로젝트 파일, 실행 바이너리는 포함되어 있지 않습니다. 아래 설정과 API는 구현 설명이며, 이 저장소만으로 빌드하거나 동작을 검증할 수 없습니다.

ASP.NET Core MVC 프레임워크를 기반으로 구축된 웹 애플리케이션입니다. 제품소개, 시뮬레이션, 소프트웨어, SDK 파일 다운로드 기능을 제공합니다.

---

## 기술 스택 (Tech Stack)

### Backend & Frontend
* **Framework:** .NET 8.0 (ASP.NET Core MVC)
* **Language:** C#
* **UI Framework:** Bootstrap 5 (Responsive Web Design)
* **Database:** SQLite / Entity Framework Core 8

### Infrastructure
* **Web Hosting:** 자체 Windows 서버 / Nginx
* **Database:** SQLite 영속 데이터베이스
* **Firmware Storage:** 서버 영속 데이터 디렉터리

---

## 핵심 기능 (Key Features)

1. **제품 소개 (Product Introduction)**
   * 런치 모니터 종류 소개

2. **소프트웨어 (software)**
   * 시뮬레이터, 스윙분석 프로그램 소개 및 다운로드

3. **SDK**
   * C#, C++ 개발가이드 및 라이브러리
   * Zynq Firmware 공개 다운로드와 관리자 전용 업로드

4. **SUPPORT**
   * 사용설명서, FAQ, 회사 위치

---

## 배포 파일 관리 설정

센서 펌웨어와 E-Simulator Connect 제어툴은 별도 릴리스로 관리합니다. 파일은 DB에 직접 저장하지 않고 `EAGLE_EYE_DATA_PATH` 아래의 `firmware`, `setting-tool` 폴더에 각각 저장합니다. SQLite DB에는 버전, 파일명, SHA-256, 릴리스 노트, 업로더, 공개 상태, 최신 지정 상태, 다운로드 횟수를 기록합니다.

관리자는 `/Home/Connect`에서 두 종류의 릴리스를 업로드, 수정, 공개/비공개, 최신 지정, 다운로드, 삭제할 수 있습니다. 최신 버전은 파일 종류별로 하나씩 관리자가 직접 지정하며, 공개 릴리스만 최신으로 지정할 수 있습니다.

운영 환경 변수:

* `EAGLE_EYE_DATA_PATH`: SQLite DB, 펌웨어 및 제어툴 파일을 보관할 영속 디렉터리
* `EAGLE_EYE_ADMIN_SECRETS_PATH`: 최초 관리자 계정 설정 JSON 경로

최초 관리자 설정 예시:

```json
{
  "AdminBootstrap": {
    "Username": "admin",
    "Password": "replace-with-a-strong-temporary-password"
  }
}
```

애플리케이션 최초 실행 시 비밀번호를 해시하여 DB에 저장합니다. 관리자 생성 후에는 설정 파일에서 평문 `Password` 값을 제거할 수 있습니다.

### 공개 버전 API

런처와 제어툴은 인증 없이 최신 릴리스 정보를 JSON으로 조회하고 파일을 다운로드할 수 있습니다.

```text
GET /api/releases/setting-tool/latest
GET /api/releases/firmware/latest
GET /api/releases/setting-tool/latest/download
GET /api/releases/firmware/latest/download
```

메타데이터 API에 `currentVersion`을 전달하면 대소문자를 구분하지 않는 정확한 문자열 비교 결과를 `isLatest`로 반환합니다.

```text
GET /api/releases/firmware/latest?currentVersion=1.2.0
```

```json
{
  "releaseType": "firmware",
  "version": "1.3.0",
  "currentVersion": "1.2.0",
  "isLatest": false,
  "releaseNotes": "센서 초기화 안정성 개선",
  "sha256": "0123456789abcdef...",
  "sizeBytes": 7340032,
  "publishedAtUtc": "2026-07-24T03:00:00Z",
  "downloadUrl": "https://rbiotech.kr/eagleeye/api/releases/firmware/latest/download"
}
```

공개된 최신 릴리스가 지정되지 않았거나 실제 파일이 없으면 HTTP 404를 반환합니다.

---

## 화면 예시 (Screenshots)
<img width="1915" height="904" alt="스크린샷 2026-07-15 154933" src="https://github.com/user-attachments/assets/2e404035-9b83-433f-8bb1-6141583e9859" />
<img width="1906" height="828" alt="스크린샷 2026-07-15 155018" src="https://github.com/user-attachments/assets/6768522a-6ad2-468d-bbc9-006691b87557" />
<img width="1913" height="902" alt="스크린샷 2026-07-15 155004" src="https://github.com/user-attachments/assets/c4912a13-4888-43f8-b6c4-b9848c5a5f68" />
<img width="1911" height="904" alt="스크린샷 2026-07-15 154949" src="https://github.com/user-attachments/assets/eee1ebcf-019a-4b47-aba2-e2700fbb9e20" />

## 실행 및 연동 범위

실행에는 별도로 제공되는 애플리케이션과 배포 구성이 필요합니다. 위 JSON은 응답 구조를 설명하는 예시이며 실제 릴리스 정보나 유효한 SHA-256 값이 아닙니다. 주소는 배포 환경에 맞게 확인해야 합니다.

관련 [SettingTool 프로젝트](https://github.com/jelis86/SettingTool_Release_public)는 정적 버전 파일과 ZIP 다운로드 방식을 설명합니다. 해당 방식과 이 문서의 JSON API가 같은 클라이언트 버전에서 호환되는지는 공개 자료만으로 확인할 수 없습니다.
