

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

### Next Step

스크립트 내용을 확인하셨으니, 이제 환경 변수 설정이 끝났다고 가정하고 원하는 **파이썬 버전(예: 3.11.5)을 실제로 설치**하는 `pyenv install` 명령어를 실행하는 실습을 진행해 드릴까요?
