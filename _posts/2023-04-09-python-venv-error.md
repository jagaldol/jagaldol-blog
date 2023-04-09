---
layout: single
title: "파이썬 가상환경 활성화했는데 시스템 파이썬 사용 중인 문제"
date: 2023-04-09 21:47:00 +0900
categories:
    - development
---

학교에서 진행 중인 랩 인턴 프로그램 중 머신러닝 연구실의 실습을 정리할려고 파이썬 가상환경을 구축하고 있었어요. 그런데 자꾸 가상환경에서 시스템 파이썬을 사용하는 거에요\...

```sh
> python -m venv .venv
> .venv\Scripts\activate.bat
(.venv) >
```

파이썬 가상환경을 만드는 방법입니다... 원래 이렇게 가상환경을 만들고 실행하면 되야하는데! (.venv)이 되며 가상환경에 들어왔지만 `python`과 `pip` 명령어가 만든 `.venv`의 `python`을 사용하지 않고 시스템의 `python`을 사용하고 있더라고요???💦

## 문제 발견
### 비정상 환경
일단 이걸 발견한 코드입니다.

```sh
> python -c "import sys; print(sys.executable)"
C:\Users\hyejun\AppData\Local\Programs\Python\Python311\python.exe

> .venv\Scripts\activate.bat
(.venv) > python -c "import sys; print(sys.executable)"
C:\Users\hyejun\AppData\Local\Programs\Python\Python311\python.exe

```

보이시나요?? 해석하자면 `python -c "import sys; print(sys.executable)"`이 코드는 실행한 python의 실행 경로를 확인하는 명령어입니다. 가상환경 진입 이후에도 여전히 시스템의 appdata내부의 python을 사용하는 거에요!🙃

### 정상 환경
이게 바로 정상적인 출력입니당..🥲

```sh
> .venv\Scripts\activate.bat
(.venv) > python -c "import sys; print(sys.executable)"
C:\Users\hyejun\Downloads\.venv\Scripts\python.exe

```

Download 폴더에다가 만든 가상환경이고 정상적으로 나오죠??

## 해결

일단 결론부터 말하면 겨우겨우 해결했어요! 설마설마한 폴더명 때문이더라고요.. 폴더명에 특수문자가 들어가면 안돼요(저의 경우 & 였어요).

### 과정
다시 같이 볼까요? 제가 가상환경을 만들던 폴더의 절대 경로입니다. `C:\Users\hyejun\github\lab-internship\Machine Learning & Bioinformatics Lab` 여기에다가 가상환경을 만들었거든요. 원래 가상환경을 진입하고 `pip list`를 하면

```sh
Package    Version
---------- -------
pip        22.3.1
setuptools 65.5.0
```

이것만 나와야한단 말이죠? 그런데 만들고 나서 pip list를 하니까 원래 시스템에 깔려있던 패키지가 다 뜨는 거에요. 그래서 ChatGPT한테도 물어봤죠. 그러니까 이녀석이 가상환경이지만 원래 시스템꺼도 사용할 수 있다! 이러길래 그런갑다~ 하고 필요한 패키지를 전부 설치했거든요.

그런 후 주피터 노트북으로 가상환경 선택하고 실행하니 패키지가 아무것도 설치 안되있다고 하는거에요. 알고보니 설치한 수많은 패키지가 기본 시스템에 설치가 된거에요.🙃🙃

그 이후 가상환경 재설치도 해보고 `activate.bat` script도 뜯어봤는데 멀쩡해보였단 말이죠. script에도 폴더명 잘 나타내고 있구..

그러고 다른 폴더에서도 설치해보고 테스트해봤는데 하필 그 폴더만 안되면서, 스택오버플로우에서 경로를 바꾸거나 하여 `hard coding`된 경로 때문에 시스템 python을 사용할 수가 있다는 걸 발견했어요. 하지만 script를 뜯어봤을 때 멀쩡했단 말이죠. 

```bat
...

set VIRTUAL_ENV=C:\Users\hyejun\github\lab-internship\Machine Learning & Bioinformatics Lab\.venv

...
```

그러던 중, 왠지 폴더 명이 문제 일거 같아 특수 기호를 지웠어요. &를 and로 바꿔 `Machine Learning and Bioinformatics Lab`로 폴더 명을 바꾸자 정상적으로 인식이 됐어요!

```sh
> python -m venv .venv
> .venv\Scripts\acitvate.bat
(.venv) > python -c "import sys; print(sys.executable)"
C:\Users\hyejun\Downloads\lab-internship\Machine Learning and Bioinformatics Lab\.venv\Scripts\python.exe
```

드디어!! 설마 폴더명에 한국어가 있던 것도 아니였는데 경로 문제일 줄은 상상도 못했네요. 아무튼 특수문자도 위험하니 자제를 해야겠습니다. &는 확실하게 쓰시면 안돼요!!😅

이건 문제가 있던 제 [레포](https://github.com/jagaldol/lab-internship)입니다. 여기서 Machine Learnig & Bioinformatics Lab의 실습을 정리하다가 정식 명칭으로 폴더명 만든다고 &까지 그대로 넣다가 문제가 생겼네요. 이젠 해결했어요~

[https://github.com/jagaldol/lab-internship](https://github.com/jagaldol/lab-internship)

## 참고
* ["python" still runs the system version after virtualenv activate - stack overflow](https://stackoverflow.com/questions/41003859/python-still-runs-the-system-version-after-virtualenv-activate)
* [ChatGPT](https://chat.openai.com/chat)