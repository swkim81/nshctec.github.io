---
layout: post
title: Android Exception의 Method Argument 추적 코드 
date: 2021-05-24 17:39:00 +0900
category: Mobile2
---
<br>

# 안녕하세요!
> 첫 포스트입니다. 잘 부탁드립니다.

* * *

<br><br><br><br><br><br><br><br><br><br><br><br><br>

### 목적
[FridaCodeShare]와 같은 사이트에 공개된 일반적인 java.lang.Exception Hooking 코드를 사용할 때 <br> 
콜 트레이스로 반환된 메소드들의 argument 추적을 할 수 있으면 좋겠다는 생각으로 개발했습니다.
<br><br>

* * *

### 코드 다운로드
- Github : [arg_trace.py]
<br><br>

* * *

### 프로그램 구조
큰 틀에서 프로그램의 구조를 그려보면, Python으로 짜여진 (여기서는 Javascript를 감싸 실행한다는 의미로 특별한 용어가 없지만 Wrapper 프로그램이라 부르겠습니다.) Wrapper 프로그램이 주 역할을 합니다. Wrapper 프로그램의 역할은 크게 4가지 입니다.
<br>

##### Wrapper 프로그램의 역할 
1. Frida agent를 제어합니다. (Setup)
2. Frida Javascript 실행 시에 삽입될 정보를 전처리합니다.
3. Frida Javascript를 생성합니다.
3. 실행 후 런타임 때 반환된 정보를 후처리합니다.

Frida agent를 제어하는 것은 사용자가 원하는 클래스를 셋업하는 것을 의미합니다.<br>
또 실행에 필요한 정보들을 런타임 전후에 알맞게 처리하며, 이는 데이터를 정규식을 이용해 파싱하여 적절하게 데이터를 자르고 활용 가능한 정보로 만드는 것을 말합니다.<br>
<br><br>

* * * 

### [arg_trace.py] 사용법
기본적인 동작방식은 Target setup -> Script Setup -> Run Script 입니다.<br>
현재 활성화된 타겟은 메뉴 아래에 세션(pid)와 클래스명으로 확인 가능하며, 현재 삽입된 JScode는 2. Show data -> 1. Show current JS code에서 확인 할 수 있습니다.

<br>
##### 메인 메뉴
```
-------------------------------------------------
        Trace exception argument in runtime
-------------------------------------------------
[*] Status : SESSION : Session(pid=27306), TARGET : test.test.app

1. Setup
2. Show data
3. Setup to script
4. Run script
5. Overloading Calculate --> Not yet no implement.
6. Exit


>
```
<br><br>

* * * 

### 사용 가능한 스크립트
1. Error overloading : 타겟 함수의 오버로딩 값을 얻기 위해 실행하는 스크립트
2. Trace exception : 얻은 Overloading 값을 통해 Exception 발생시켜  Method에 대한 Call Stack을 얻기 위해 사용하는 스크립트
3. Trage argument of exception : 2번 스크립트에서 얻은 Call Stack을 활용하여 Call Stack안 메소드들의 Argument 값을 추적하기 위해 사용하는 스크립트
<br><br><br>

* * * 

### Exception의 Argument 추적을 하기위한 스크립트 사용법

Setup to Script의 메뉴 1.->2.->3.을 순차적으로 Setup하여 실행하면 Argument 추적이 가능합니다.<br><br>

##### 권장하는 스크립트 실행순서
1. 1번 스크립트의 결과로 Overloading 내역이 런타임 이후 반환되며 반환된 값 중 추적하려는 값을 셋업하면 2번 스크립트의 Overloading 값으로 2번의 script.on 실행 전, 삽입되는 JScode에 자동으로 정보가 삽입됩니다.
2. 2번 스크립트 삽입으로 앱을 다시 셋업하여 실행하면 스크립트가 후킹되어 동작하면서 런타임 때 확인된 메소드 인스턴스들을 리스트 형태로 반환합니다. 
  > 리스트안에 담긴 객체는 각자 런타임 때 생성된 Exception Call Stack 정보를 가지고 있습니다.
3. 이를 3번 스크립트에서 활용하기 위해 1. Setup -> 2. Setup to data that haveto trace in exception result 메뉴에서 원하는 Exception 인스턴스를 선택하여 3번 스크립트 실행전에 셋업합니다.
4. 이후 3번 스크립트를 실행합니다.
<br><br>

* * * 

### 코드 설명 주석 부분

> 프로그램 시작 <br>

```python
if __name__ == "__main__":
```
> 프로그램 종료 <br>

``` python
while(1):
  break;
```

##### 함수
> program_log : 프로그램 전역 로깅 함수

##### 클래스 
>Menu : 메인 메뉴 상호작용 및 출력
>>함수
>>>show_menu : 메뉴 출력 <br>
>>>choice_check : 사용자 메뉴 선택값 입력 처리

>ClassFrida : Frida 후킹 클래스
>>클래스
>>>ClassExceptionData
>>함수 : 기능별 분류
>>>기본
>>>> run_script : Frida 스크립트 실행 <br>
>>>> select_script : 실행할 스크립트 선택 <br>
>>>> show_selected_exception : show_exception_data에서 선택한 exception의 call stack을 출력 <br>
>>>> show_script : 현재 설정된 스크립트를 출력 <br>
>>>> show_exception_data : Exception 정보를 출력 <br>
>>>> setup_is_init : Setup하는 Target의 메소드가 생성자인지 확인 <br>
>>>> submenu_show_data : Show_data의 서브 메뉴 <br>
>>>> submenu_setup_frida : Setup의 서브 메뉴 <br>

>>>설정 셋업
>>>> setup_frida : Target을 셋업하는 함수 <br>
>>>> setup_exception_data : Trace할 Exception을 선택 <br>
>>>> setup_selected_exception_data : 2번 jscode를 실행 후 만들어진 exception 정보를 trace하기 위해 setup하는 함수 <br>

>>>JScode 생성
>>>> setup_exception_trace_code : Exception을 Implementation한 jscode 생성 <br>
>>>> setup_err_js_code : Overloading 값을 출력하기 위한 Error 발생 jscode jscode 생성 <br>
>>>> create_tracing_exception_args_jscode : argument trace jscode 생성 함수 <br>
>>>> template_jscode : Argument Trace jscode를 생성하는 클로저 반환 함수 <br>
>>>> template_check_object_jscode : argument값이 object인지 확인하는 jscode 삽입 함수 <br>
>>>> error_wrapper_setup_jscode : setup_err_js_code에서 생성된 jscode를 target에 삽입하는 함수 <br>

>>> 검사 <br>
>>>> check_exception_run_count : exception을 저장하기위한 g_count 검사 <br>
>>>> check_send : exception 배열을 다음 인덱스로 증가시키기 위한 arguemnt 갯수 확인 <br>
>>>> check_setup_script : script가 비어있는지 검사 <br>
>>>> check_setup_target : target이 비어있는지 검사 <br>
 
>>>핸들 == 모니터링 // frida.script.on <br>
>>>> monitor_error : 1번 error jscode의 모니터링 함수 <br>
>>>> monitor_exception : 2번 exception implementation jscode를 모니터링하는 함수 <br>
>>>> monitor_exception_trace : 3번 argument trace jscode 모니터링 함수 <br>

>>>실행 후 처리 <br>
>>>> parse_from_parameter : exception jscode에 삽입될 파라미터를 overloading 갯수를 측정하여 param1 , param2 의 형식으로 만들어주는 함수 <br>
>>>> parse_selected_exception_data : exception의 callstack을 파싱하는 함수 <br>

<br><br>

* * *
### 차후 개선 방안
- 3번 스크립트 특정 스레드에 대해서만 추적하는 기능 필요
- Exception Call Stack의 argument에 대한 평문 처리 필요
- 다른 앱들에서 활용 가능한 범용성 확장
- iOS에서 활용 가능하도록 코드 구현 필요
<br><br>

* * *

### 참고
- [Frida Javascript API]
- [Java.lang.Exception Reference]
- [Frida Snippets]
<br><br><br><br><br><br><br><br><br><br><br><br><br>

<a href="https://github.com/HyeonBell"><img align="left" src="https://avatars.githubusercontent.com/u/22285792?v=4" width="180" height="180"></a> 
<br><br>

### &nbsp;&nbsp;&nbsp;Shin Hyeon Jong

<p align="left">&nbsp;&nbsp;안녕하세요! 리버스엔지니어링과 자동화를 좋아합니다.</p> 
 
&nbsp;&nbsp; <a href="https://github.com/HyeonBell"><img src="https://img.shields.io/badge/github-181717?style=for-the-badge&logo=Github&logoColor=white"/></a> 
&nbsp; [![Gmail Badge](https://img.shields.io/badge/Gmail-d14836?style=for-the-badge&logo=Gmail&logoColor=white&link=mailto:hyeonbells@gmail.com)](mailto:hyeonbells@gmail.com)
<br>

* * *


<br>
<br>

[arg_trace.py]: <https://github.com/HyeonBell/Tool/tree/master/frida-script>
[FridaCodeShare]: <https://codeshare.frida.re/>
[Frida Javascript API]:  https://frida.re/docs/javascript-api/>
[Java.lang.Exception Reference]: <https://docs.oracle.com/javase/7/docs/api/java/lang/Exception.html?is-external=true>
[Frida Snippets]: <https://github.com/iddoeldor/frida-snippets>
