---
layout: post
title: 'Bash Shell script 기초'
subtitle: '쉘 스크립트 기초를 닦아보자'
categories: programing
tags: bash shell script
comments: true
---

## 쉘 스크립트 

쉘은 사용자가 입력한 명령을 해석해 커널에 전달하거나, 커널의 처리 결과를 사용자에게 전달하는 역할을 합니다. <br>


## 쉘 스크립트 만들기 
1. 'vim 파일이름.sh'로 만들 수 있습니다. <br>
   Ex) vim test.sh 
2. vim 환경에서 가장 첫 줄은 '#!/bin/sh'을 적어줌으로써 어떤 명령어 해석기를 사용하는지 명시합니다. 
3. 이후 사용자가 원하는데로 프로그래밍을 해줍니다. 
  - 쉘 스크립트에서 문자열을 출력하고 싶을 때는 **echo** 명령어를 사용합니다. <br>
  Ex) `echo "Hello world!"`
4. sh 파일을 생성했다면 아직 '실행 가능'한 상태가 아닙니다. '실행 가능' 속성을 추가해야 합니다.
  - 권한을 부여할 때는 **chmod** 명령어를 사용합니다. <br>
  Ex) `chmod +x test.sh` 
5. 권한을 부여받은 sh파일을 실행시킵니다.
  Ex) ```python 
       ./test.sh
      ```


## 변수 사용 
- '변수명=변수 값' 으로 변수를 선언합니다. <br>
  Ex) 
  ```python
          A=5 
          B=10 
          C=hello 
          D="Hello world!" 
  ``` 
  ### 주의 사항 
  - 변수에 넣는 값은 문자열로 취급하기 때문에 공백이 있을 경우 ' 혹은 "로 묶어야 합니다.
  - 변수 이름은 대소문자를 구분합니다. 변수 이름은 대문자로 작성하는 것을 추천합니다.  
  - 변수를 대입할 때 '=' 기호 좌우에 공백이 없어야 합니다.
  - 변수를 사용하려면 $ 문자를 앞에 붙여야 합니다. <br>
    Ex) `echo "A=$A"`
    > A=5
    
## 사칙 연산 
- 변수에 입력된 값들은 전부 문자열입니다. 따라서 연산식으로 바꾸게 해주는 **expr** 명령어를 사용합니다. 
- expression 식에서 각 숫자와 연산자 마다도 띄어 써야 합니다.
- Ex) ```python
      #!/bin/bash
      A=5
      B=15
      echo $(expr $A+$B)
      echo $(expr $A + $B)
      exit 0
      ```
    > 5+15<br>
    > 20  
- 곱셈 연산자는 '\*'을 사용합니다. 괄호 앞에도 역슬레쉬(\)를 붙여줍니다. 
- Ex) ```python
      #!/bin/bash
      A=5
      B=15
      echo $(expr \( $A + $B \) \* $A)
      echo $(expr \( $B - $A \) / $A)
      exit 0
      ```
     > 100 <br>
     > 2

## 파라미터 변수
- 파라미터 변수는 $0, $1, $2 와 같은 형식을 가집니다. 
- 명령을 실행할 때 지정되며 sh파일에서 필요한 입력 파라미터를 전달받습니다. 
- Ex)  
```python 
#!/bin/bash
echo "실행 파일 이름: $0"
echo "\$1: $1"
echo "\$2: $2"
echo "\$3: $3"
echo "전체 파라미터: $*"
exit 0
```
    > sh test.sh value1 value2 value3 <br>
    > 실행 파일 이름: test.sh <br>
    > $1: value1 <br>
    > $2: value2 <br>
    > $3: value3 <br>
    > 전체 파라미터: value1 value2 value3 

## 조건문 
- if 문은 아래와 같이 사용합니다. 
```c 
if [ 조건 ]; then
  code
else 
  code
fi
```

- case 문은 아래와 같이 사용합니다.
- 각 case 마지막에 `;;`을 붙여줍니다. 
```c 
case 변수 in 
   case1)
     code
     ;;
   case2)
     code
     ;;
 esac
 
## 값 입력 받기 
- `read` 명령어를 사용해서 사용자로부터 값을 입력 받을 수 있습니다. 
- `-n'은 개행을 의미합니다. 
- `-n 1`은 `enter` 입력 없이 한 문자의 입력을 즉시 받고 싶을 때 사용합니다.
- Ex) ```c
      #!/bin/bash
      echo -n "input"
      read input
      echo "input: $input"
      ```

## 반복문 
- for 문은 아래와 같이 사용합니다. 
```c
for 변수 in 값1, 값2 
do
   code 
done 
```
- Ex) ```c
      #!/bin/bash
      for var in 1 2 3
      do
        ehco $var
      done
      ```
   > 1 <br>
   > 2 <br> 
   > 3 <br>

참고자료 
1. https://mug896.github.io/bash-shell/functions.html
2. https://kangsecu.tistory.com/54
3. https://velog.io/@jinyongp/%EC%89%98-%EC%8A%A4%ED%81%AC%EB%A6%BD%ED%8A%B8-%ED%94%84%EB%A1%9C%EA%B7%B8%EB%9E%98%EB%B0%8D
