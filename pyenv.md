

## 🐍 pyenv 설치 스크립트 (curl [https://pyenv.run](https://pyenv.run)) 핵심 내용

`$ \text{curl https://pyenv.run}$` 명령을 실행하면 출력되는 내용은 $\text{pyenv}$를 설치하고 환경 변수를 설정하기 위한 **쉘(Bash/Zsh) 스크립트**입니다.

스크립트 전체가 상당히 길지만, 주요 기능 및 단계는 다음과 같습니다.

### 1\. 환경 변수 및 사전 검사

스크립트가 실행되는 쉘 환경을 확인하고, 필요한 기본 도구(`git`, `curl`, `bash` 등)가 시스템에 있는지 확인합니다.

```bash
#!/usr/bin/env bash

# pyenv 설치를 위한 환경 변수 설정
: "${PYENV_ROOT:=$HOME/.pyenv}"

# 필요한 명령어가 있는지 확인
command -v git >/dev/null || {
  echo >&2 "오류: git이 설치되어 있지 않습니다."
  exit 1
}

# pyenv가 이미 설치되어 있는지 확인
if [ -d "$PYENV_ROOT" ]; then
  echo "pyenv가 이미 $PYENV_ROOT 경로에 설치되어 있습니다."
  # 이미 설치된 경우 업데이트 로직을 수행할 수 있습니다.
fi
```

### 2\. pyenv 다운로드 및 복제 (Clone)

$\text{pyenv}$의 GitHub 저장소에서 코드를 로컬 `$PYENV_ROOT` 경로로 복제합니다.

```bash
# pyenv 저장소를 ~/.pyenv 경로에 복제
git clone --depth 1 https://github.com/pyenv/pyenv.git "$PYENV_ROOT"

# pyenv 플러그인 등 초기화
# ...
```

### 3\. 쉘 환경 설정 안내 (핵심)

스크립트의 가장 중요한 역할은, **사용자가 수동으로 쉘 환경 파일에 추가해야 할 설정 내용**을 안내해 주는 부분입니다.

```bash
# 쉘 환경 설정 안내 메시지 출력
echo
echo "경고: pyenv를 사용하려면 쉘 환경을 설정해야 합니다."
echo "--------------------------------------------------------"
echo "다음 내용을 ~/.bashrc 또는 ~/.zshrc 파일의 끝에 추가하십시오:"
echo
echo 'export PYENV_ROOT="$HOME/.pyenv"'
echo 'command -v pyenv >/dev/null || export PATH="$PYENV_ROOT/bin:$PATH"'
echo 'eval "$(pyenv init --path)"'
echo
echo "설정 후, 'source ~/.bashrc' 명령을 실행하여 적용해야 합니다."
echo "--------------------------------------------------------"
```

### 4\. 정리

스크립트가 성공적으로 설치를 완료했음을 알리고 종료됩니다.

-----

### [Expert Tip] 설치 스크립트의 목적

이 스크립트는 **pyenv의 GitHub 저장소를 로컬에 복제**하고, 사용자가 다음 단계에서 **환경 변수 설정을 완료**할 수 있도록 안내하는 것이 주된 목적입니다.

---
`pyenv`는 **쉘 스크립트(Shell Script)**로 작성된 프로그램입니다.


### 1. 📜 pyenv의 정체: 쉘 스크립트

`pyenv`는 **Bash**와 **Zsh**와 같은 유닉스 계열 쉘 환경에서 실행되도록 설계된 일련의 **쉘 스크립트 파일들**로 구성되어 있습니다.

* **실행 방식:** `pyenv` 명령을 입력하면, 시스템은 `$PATH`를 통해 `~/.pyenv/bin/pyenv` 파일을 찾습니다. 이 파일은 복잡한 로직을 수행하는 다른 쉘 스크립트들을 호출하여 명령을 처리합니다.
* **장점:** 쉘 환경에서 직접 작동하기 때문에, `$PATH`와 같은 **환경 변수**를 자유롭게 조작하고, 쉘의 **내부 함수(Shell Function)**를 사용하여 파이썬 버전 전환을 매우 효율적으로 수행할 수 있습니다.

### 2. ⚙️ 실행 파일처럼 작동하는 이유

`pyenv`가 일반적인 컴파일된 **실행 파일(Binary Executable)**처럼 보이는 이유는 다음 두 가지 핵심 기능 덕분입니다.

#### A. 심볼릭 링크 및 래퍼 스크립트
`pyenv`는 `pyenv install`과 같은 명령을 실행할 때, **심볼릭 링크(Symbolic Link)**나 **래퍼 스크립트(Wrapper Script)**를 사용하여 마치 그 버전의 파이썬 실행 파일이 `$PATH`에 있는 것처럼 속입니다.

#### B. 쉘 후크 (Shell Hook)와 Shims
가장 중요한 부분은 **Shims**와 **`eval "$(pyenv init --path)"`** 명령입니다.
1.  **Shims:** `~/.pyenv/shims` 디렉터리에 `python`, `pip` 등 표준 파이썬 명령어 이름을 가진 작은 스크립트들이 생성됩니다.
2.  **`pyenv init`:** 이 명령은 `$HOME/.pyenv/shims` 디렉터리를 시스템의 `$PATH` 맨 앞에 추가합니다.
3.  **작동:** 사용자가 `python`을 실행하면, 시스템은 shim 스크립트를 먼저 찾습니다. 이 shim 스크립트는 `pyenv`의 로직을 호출하여 현재 프로젝트에 맞는 **올바른 Python 버전**의 실제 실행 파일로 연결(redirect)해 줍니다.

결론적으로, `pyenv` 자체는 쉘 스크립트이지만, 이 스크립트가 시스템 환경을 조작하여 **버전 전환을 마치 네이티브 실행 파일처럼 매끄럽게** 만들어주는 것입니다.



---
`pyenv`의 실제 사용 예시를 보여드리면서, 이것이 어떻게 개발 환경을 깔끔하게 만드는지 쉽게 이해할 수 있도록 설명해 드리겠습니다.

**3vm구축실습예제**와 같은 프로젝트에서 **Flask 백엔드 개발 환경**을 구축한다고 가정해 봅시다.

-----

## 1\. 🔍 현재 환경 확인: 기본 상태 파악

`pyenv`를 설치했다면, 먼저 현재 시스템에 어떤 Python 버전이 설치되어 있고, 어떤 버전이 사용되고 있는지 확인합니다.

```bash
# 설치된 모든 Python 버전 목록 확인
pyenv versions
```

**[실행 예시]**

```
* system  <-- 현재 시스템의 기본 Python
  3.10.12
```

현재는 시스템 Python과 $\text{3.10.12}$만 설치되어 있고, `*` 표시를 통해 **시스템 버전**이 활성화되어 있음을 알 수 있습니다.

-----

## 2\. 🛠️ 원하는 Python 버전 설치

Flask 프로젝트를 위해 $\text{Python 3.11.5}$를 사용하기로 결정했습니다. `pyenv`를 사용하여 시스템에 이 버전을 설치합니다.

```bash
# 3.11.5 버전 설치 (컴파일 시간이 소요됩니다)
pyenv install 3.11.5

# 설치 후 다시 목록 확인
pyenv versions
```

**[실행 예시]**

```
* system
  3.10.12
  3.11.5   <-- 새 버전 설치됨
```

이제 원하는 버전이 설치되었지만, 아직 활성화된 버전은 \*\*`system`\*\*입니다.

-----

## 3\. 📂 프로젝트 환경 격리: `pyenv local` 사용 (핵심)

이제 프로젝트 폴더를 만들고, 이 폴더에서만 $\text{Python 3.11.5}$를 사용하도록 설정합니다. 이것이 바로 **`pyenv`의 가장 중요한 기능**입니다.

### A. 프로젝트 폴더 생성 및 이동

```bash
mkdir flask_backend_project
cd flask_backend_project
```

### B. 로컬 버전 설정 (격리)

```bash
# 현재 디렉터리에서만 Python 3.11.5를 사용하도록 설정
pyenv local 3.11.5
```

**[결과 확인]**

1.  폴더 내에 \*\*`.python-version`\*\*이라는 파일이 생성되고, 그 안에 `3.11.5`가 기록됩니다.
2.  `pyenv versions`를 다시 실행하면 **`3.11.5`** 앞에 `*`가 붙으며 활성화됩니다.

<!-- end list -->

```bash
pyenv versions
```

```
  system
  3.10.12
* 3.11.5  <-- 현재 이 폴더에서 활성화됨!
```

### C. 자동 전환 확인

이제 이 폴더 안에서는 **`python`** 명령어를 입력하기만 하면 자동으로 $\text{3.11.5}$가 실행됩니다.

```bash
# 이 폴더 안에서는 3.11.5가 실행됨
python --version
# 출력: Python 3.11.5

# 폴더 밖으로 나가서 확인하면 다시 시스템 버전이 활성화됨
cd ..
python --version
# 출력: Python 3.x.x (시스템 버전)
```

-----

## 4\. 📦 패키지 관리: 가상 환경(venv) 구축

`pyenv`로 버전 격리를 완료했다면, 이제 이 버전 안에서 **패키지 종속성**을 격리하기 위해 \*\*`venv` (Virtual Environment)\*\*를 사용합니다.

```bash
cd flask_backend_project

# 1. 3.11.5 환경에 myvenv라는 가상 환경 생성
python -m venv myvenv

# 2. 가상 환경 활성화 (이제부터 설치되는 모든 패키지는 이 myvenv에만 저장됨)
source myvenv/bin/activate
```

**[터미널 변화]**

```bash
(myvenv) user@vm:~/flask_backend_project$ 
```

프롬프트 앞에 `(myvenv)`가 붙어 **패키지 격리**가 완료되었음을 알립니다.

### 5\. Flask 설치 및 실행

이제 안전하게 Flask를 설치하고 프로젝트를 시작할 수 있습니다.

```bash
(myvenv) pip install Flask

# 패키지는 myvenv 내에만 설치됨
(myvenv) pip freeze
# 출력: Flask==x.x.x

# 작업 완료 후 가상 환경 비활성화
(myvenv) deactivate
```

### 💡 최종 정리

`pyenv`를 사용함으로써, 우리는 시스템을 건드리지 않고 **프로젝트 폴더별로 필요한 Python 버전을 명확히 격리하고 전환**하는 안정적인 개발 환경을 완성했습니다. 이것이 `pyenv`를 사용하는 가장 큰 이유이자 장점입니다.
