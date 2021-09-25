---
title:  "PyQt5 설치 및 환경설정"
categories:
  - Python
---
# 1. PyQt5 설치
명령 프롬프트나 Powershell에 다음 명령어를 입력하여 설치한다.
```
pip install PyQt5
Pip install PyQt5-tools
```

# 2. 환경설정
환경변수에 다음과 같이 두 개의 변수를 추가해 준다.  
![image](/assets/images/conda_install/pyqt_path.png)

**QT_PLUGIN_PATH**
```
C:\Users\[PC 이름]\miniconda3\Lib\site-packages\PyQt5\Qt5\plugins
```  
**QT_QPA_PLATFORM_PLUGIN_PATH**
```
C:\Users\[PC 이름]\miniconda3\Lib\site-packages\PyQt5\Qt5\plugins\platforms
```
PyQt5를 설치하면  
`C:\Users\[PC 이름]\miniconda3\Lib\site-packages\qt5_applications\Qt\bin\designer.exe`  
경로에 Qt Designer를 사용하여 UI를 구성할 수 있다.
![image](/assets/images/conda_install/qt_designer.jpg)  
.ui 확장자 파일의 연결 프로그램을 등록해 놓자.
