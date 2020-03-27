---
title: "[Lena's Reversing for Newbies] tutorial 1"
categories:
  - Hacking
tags:
  - Reversing
  - lena
last_modified_at: 2020-03-27
---

리버싱을 연습하기 좋은 자료로서 유명한 자료 중 하나로 **Lena's Reversing for Newbies** 가 있다.

[tuts4you](https://tuts4you.com/) 라는 사이트에 올라온 것이다.

다운로드는 [여기](https://tuts4you.com/download/2876/reversing-for-newbies-complete) 에서 한꺼번에 40개의 튜토리얼을 다운받을 수 있다.

## 실행
Ollydbg를 이용해 분석해보기 전에 먼저 실행해서 어떤 것이 목표고 어떻게 동작하는지를 알아보자.
![](/assets/images/lena/1/lena1-1.png){: .align-center}

평가 기간의 만료로 새로운 라이선스를 얻으라는 얘기를 하고 있다.
제목 또한 Key File이 들어가는 것으로 보아 Key를 새로이 만들어주어야 할 것 같은 느낌이 든다.

## 분석
이제 Ollydbg를 이용해 첫번째 튜토리얼 파일인 reverseMe.exe를 분석해보도록 하자.
![](/assets/images/lena/1/lena1-2.png ){: .align-center}

'Search for - All intermodular calls'에 들어가 보면 WinApi로 만들어졌다는 것을 알 수 있다.
그리고 Entry Point부터 동작을 살펴보다보면 다음과 같은 부분을 만날 수 있다.
![](/assets/images/lena/1/lena1-3.png ){: .align-center}
해당하는 함수가 무엇인지 알기 위해 검색을 통해 알아봤더니 '파일을 열거나 생성한다.'라고 되어 있었다.
그래서 전달하는 parameters 중에 'FileName' 부분에 'Keyfile.dat'이라는 이름이 보인다.
reverseMe.exe가 존재하는 파일 내에는 저런 이름을 가진 파일이 없어서 만들어내나 싶었지만,
해당 함수의 호출이 종료되어도 새로 생성되지 않고 에러 리턴값인 -1만 떴다.
이후 CMP 명령어를 통해 에러가 뜨면 우리가 처음 본 메시지 창이 뜨는 것을 볼 수 있다.

나는 해당 파일이 필요할 것으로 판단해 reverseMe.com이 존재하는 파일 내부에 'Keyfile.dat'으로 파일을 하나 생성해주었다.
![](/assets/images/lena/1/lena1-4.png ){: .align-center}
이후 다시 CMP 부분까지 실행해보니 조건분기가 넘어가서 ReadFile 함수가 실행되는 부분까지 넘어갔다.

![](/assets/images/lena/1/lena1-5.png ){: .align-center}
처음에는 Keyfile.dat에 아무런 내용 없이 돌려보니 읽기에 실패하여 실패했다는 메시지가 생성되었다.

그리고 통과 조건들을 찾아보았고 다음과 같은 조건이 있었다.
![](/assets/images/lena/1/lena1-6.png ){: .align-center}
조건 1은 위에서 말했듯이 Keyfile.dat이라는 파일이 존재해야 통과할 수 있고,
이제 남은 것은 조건 2를 통과하는 것이다.

**DWORD PTR DS:[402173]** 이 뭘 말하는 것인지 몰랐다가 위에서 ReadFile 함수의 parameter 중에 pBytesRead의 주소와 똑같았다.
설마 파일의 읽은 크기를 저장하는 것이니 싶어 Keyfile.dat 내부에 'he'라고 입력한 후 해당 주소를 살펴보았더니
![](/assets/images/lena/1/lena1-7.png){: .align-center}
글자 크기만큼인 2가 들어있었다.
그렇다면 밑에 `JL` 인 조건이 있었으니 10보다 크게 해야 실패했다는 메시지로 가지 않을 수 있다.

그래서 Keyfile.dat의 내용을 다음과 같이 설정해주고 돌려보니...
![](/assets/images/lena/1/lena1-8.png){: .align-center}
잉? 무한루프를 도네???

그래서 다시 조건을 살펴보니 조건이 2개나 더 있었다.
![](/assets/images/lena/1/lena1-9.png){: .align-center}
첫 바이트가 0(NULL)이어야 한다는 조건이 있었다.
분석해보면 첫 바이트부터 검사해서 G가 8번 이상 나온뒤 NULL을 만나면 루프를 탈출해서 통과할 수 있게 되는 것이었다.
그래서 다음과 같이 Hex Dump를 고치고 돌려보면...
![](/assets/images/lena/1/lena1-10.png){: .align-center}

![](/assets/images/lena/1/lena1-11.png){: .align-center}
여기로 올 수 있다. 딱봐도 여기로 오면 성공인 것 같다. F9를 눌러 끝까지 가보도록 하자.

![](/assets/images/lena/1/lena1-12.png){: .align-center}
성공!

---
다음에는 포스팅을 좀 더 간단히, 알기 쉽게 전달하도록 노력해봐야겠다.

리버싱 도전은 계속된다. 파이팅!
