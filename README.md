  # 오픈소스SW개론 과제

<div align="right">
  20174188 컴퓨터공학과 김형균
</div>

-------
  ## 목차 
  1. Shell script getopt,getopts 명령어
  2. sed
  3. awk
  4. 출처
   
   
   
   
   
## 1. Shell script의 getopt, getopts 명령어
------

> `getopts optstring varname [args...]`   
   

![image](https://user-images.githubusercontent.com/81621996/142755530-5e62acd0-42af-4022-a00b-4ab8150357e9.png)   
쉘 에서 명령을 실행할 때 옵션을 사용하는데요. 스크립트 파일이나 함수를 실행할 때도 동일하게 옵션을 사용할 수 있습니다.   
사용된 옵션은 다른 인수들과 마찬가지로 `$1`, `$2`, ... positional parameters 형태로 전달되므로 스크립트 내에서 직접 옵션을 해석해서 사용해야 됩니다.   
이때 옵션 해석 작업을 쉽게 도와주는 명령이 getopts 입니다.   

옵션에는 short 옵션과 long 옵션이 있는데 getopts 명령은 short 옵션을 처리합니다.


### short 옵션의 특징
short 옵션은 다음과 같이 여러 가지 방법으로 사용할 수 있습니다. 그러므로 getopts 명령을 이용하지 않고 직접 옵션을 해석해 처리한다면 옵션 처리에만 스크립트가 복잡해질 수 있습니다.   
```C
$ command -a -b -c
```   
------
1. 옵션을 붙여서 사용할 수 있으며 순서가 바뀌어도 된다.
```C
$ command -abc
$ command -b -ca
```   
------
2. 옵션인수를 가질 수 있다.
```C
$ command -a xxx -b -c yyy
```


## 출처
-------
+ https://www.ibm.com/docs/ko/aix/7.2?topic=g-getopt-command
+ https://mug896.github.io/bash-shell/getopts.html
+ https://systemdesigner.tistory.com/
