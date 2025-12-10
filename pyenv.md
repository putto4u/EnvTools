

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

