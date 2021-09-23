---
title:  "Miniconda 설치 및 환경설정"
categories:
  - Python
---
# 1. Miniconda 란?
* Minicoda는 Anaconda Python Package에서 최소한의 모듈만을 포함하는 Anaconda의 Lite 버전이다.  
* Anaconda Python 정식 배포 버전의 경우, Heavy한 수많은 라이브러리들이 기본으로 포함되어 있기 때문에 설치도 상당히 오래 걸리고, 용량도 크다

# 2. Miniconda 설치

[https://docs.conda.io/en/latest/miniconda.html](https://docs.conda.io/en/latest/miniconda.html)

![image](/assets/images/conda_install/download.png)

위 홈페이지에서 windows 64-bits 버전을 클릭하여 다운받는다.
설치는 기본 셋팅으로 진행하면 된다.

# 3. 환경변수 추가
Minoconda가 설치되면, conda.exe으로 필요한 라이브러리를 설치 또는 업데이트를 할 수 있다.  
이러한 Anaconda 기본적인 기능을 명령 프롬프트나 Powershell에서 사용하기 위해서는 환경변수에 해당 경로를 등록해야 한다.

![image](/assets/images/conda_install/conda_path.png)

고급 시스템 설정 → '고급' 텝 → 환경변수(N)...  
에서, PATH 변수에 다음 두 경로를 추가한다.
```
C:\Users\[PC 이름]\miniconda3\Scripts
C:\Users\[PC 이름]\miniconda3\Library\bin
```
