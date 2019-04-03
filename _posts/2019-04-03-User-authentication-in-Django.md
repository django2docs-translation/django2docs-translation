---
big-title: "Common Web application tools"
middle-title: "Authentication"
small-title: "Overview"
author: 
  - 김선규(Kim,seon-kyu)
field:
  - Authentication
relate:
  - Authentication
toc: true
toc-head-level-choice: false
#do this if head level choice is true
# toc-head-max:
# toc-head-min:
---

# User authentication in Django

장고는 유저 인증 시스템을 지원합니다. 이는 유저 계정, 그룹, 권한 그리고 쿠키 기반의 유저 세션을 관리합니다. 이 문서 섹션에서는 기본 기능이 어떻게 작동 하는지 뿐만 아니라, 프로젝트의 필요에 맞게 `extend and customize`하는 방법을 알려 줍니다.

# Overview

장고 인증 시스템은 인증(authentication)과 권한부여(authorization)을 다룹니다. 간략하게, 인증 시스템은 사용자가 가능한지 확인하고, 권한부여 시스템은 인증된 사용자가 무엇을 할 수 있는가를 결정합니다. 여기서 인증 시스템 용어는 두 가지 작업에서 모두 사용됩니다.

auth system은 다음과 같이 구성되어 있습니다:

- 사용자(User)
- 권한: 사용자가 특정 작업을 수행할 수 있는지 지정해주는  바이너리(yes/no) 플래그
- 그룹: 둘 이상의 사용자에게 이름과 permissions를 부여하는 generic 방법
- A configurable password hashing system(구성 가능한 암호 해싱 시스템)
- 사용자 로그 또는 콘텐츠 제한을 위한 양식 및 보기 도구
- A pluggable backend system

장고에서의 인증 시스템은 generic한데 초점을 맞추고 다른 웹 인증 시스템에서는 쉽게 찾을 수 있는 기능들을 제공하지 않습니다. 이러한 문제점들의 해결방안은 third-party packages를 구현합니다:

- Password strength checking
- Throttling of login attempts
- Authentication against third-parties (OAuth, for example)

---

# Installations

인증 시스템의 지원은 **django.contrib.auth**에 있는 Django contrib module에 번들로 있습니다. 기본적으로, 요구되는 configuration은 **`django-admin startproject`**를 실행함으로써 이미 **settings.py**에 포함되어 있습니다. 이들은 당신의 **`INSTALLED_APPS`** setting에 두 가지 항목으로 구성되어 있습니다:

1. **'django.contrib.auth'**는 중요한 인증 시스템 프레임워크와 그것의 기본 모델들을 포함하고 있습니다.
2. **'django.contrib.contenttypes'**는 장고 `content type system`으로  당신이 만든 모델과 권한들을 연결할 수 있게 해줍니다.

또한 당신의 **`MIDDLEWARE`** setting에 다음과 같은 항목으로 구성되어 있습니다:

1. **`SessionMiddleware`**은 요청 전체에 걸쳐 세션들을 관리합니다.
2. **`AuthenticationMiddleware`**은 세션들을 사용하여 요청과 사용자를 연결시켜 줍니다.

이와 같은 settings들과 함께, manage.py migrate 명령어를 실행시키면 델과 관련된 auth와 당신의 installed apps에 정의된 모든 모델에 대한 권한들을 위해 필요한 데이터베잇 테이블을 생성합니다.