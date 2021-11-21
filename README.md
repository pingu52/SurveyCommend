  # 오픈소스SW개론 과제

<div align="right">
  20174188 컴퓨터공학과 김형균
</div>

-------
  ## 목차 
  1. Shell script getopt,getopts 명령어
  2. Linux sed, awk 명령어
  3. 출처
   
   
   
   
   
## 1. Shell script의 getopt, getopts 명령어
------


> `getopts optstring varname [args...]`
  
   

![image](https://user-images.githubusercontent.com/81621996/142755530-5e62acd0-42af-4022-a00b-4ab8150357e9.png)   
쉘 에서 명령을 실행할 때 옵션을 사용하는데요. 스크립트 파일이나 함수를 실행할 때도 동일하게 옵션을 사용할 수 있습니다.   
사용된 옵션은 다른 인수들과 마찬가지로 `$1`, `$2`, ... positional parameters 형태로 전달되므로 스크립트 내에서 직접 옵션을 해석해서 사용해야 됩니다.   
이때 옵션 해석 작업을 쉽게 도와주는 명령이 getopts 입니다.   

옵션에는 short 옵션과 long 옵션이 있는데 getopts 명령은 short 옵션을 처리합니다.

------
### short 옵션의 특징
short 옵션은 다음과 같이 여러 가지 방법으로 사용할 수 있습니다. 그러므로 getopts 명령을 이용하지 않고 직접 옵션을 해석해 처리한다면 옵션 처리에만 스크립트가 복잡해질 수 있습니다.   
```bash
$ command -a -b -c
```   
#
1. 옵션을 붙여서 사용할 수 있으며 순서가 바뀌어도 된다.
```bash
$ command -abc
$ command -b -ca
```   
#
2. 옵션인수를 가질 수 있다.
```bash
$ command -a xxx -b -c yyy
```
#   
3. 옵션인수를 옵션에 붙여 쓸 수가 있다.   
이것은 2.와 같은 예와 동일하게 해석된다.
```bash
$ command -axxx -bcyyy
```
#
4. 옵션 구분자 '--' 가 올경우 우측에 있는 값은 옵션으로 해석하면 안된다.
```bash
$ command -a -b -- -c
```
------
### Option string 과 $OPTIND
명령에서 `-a`, `-b`, `-c` 세 개의 옵션을 사용한다면 getopts 명령에 설정하는 optstring 값은 `abc` 가 됩니다. args 에는 명령 실행시 사용된 인수 값들이 오는데 생략할 경우 `"$@"` 가 사용됩니다.

다음 예제를 통해서 getopts 명령에서 사용되는 optstring, varname 과 $OPTIND 에 대해 알아보겠습니다. shell 이 처음 실행되면 $OPTIND 값은 1 을 가리키고 getopts 명령이 실행될 때마다 다음 옵션의 index 값을 가리키게 됩니다.   

#
- 다음 set 명령은 실제 "command -a -bc hello world" 명령을 실행했을 때와 같이 positional parameters 를 설정합니다.
```bash
$ set -- -a -bc hello world
$ echo "$@"
-a -bc hello world
```
#
- 처음 $OPTIND 값은 1 로 -a 를 가리킵니다.
```bash
$ echo $OPTIND
1
```
#
- getopts 명령을 실행할 때마다 opt 변수값과 OPTIND 값이 변경되는 것을 볼 수 있습니다.   
- 다음 옵션 "b" 의 index 값은 2 가 됩니다.
```bash
$ getopts abc opt "$@"
$ echo $opt, $OPTIND
a, 2
```
# 
- 다음 옵션 "c" 는 "b" 와 붙여 쓰기를 하여 같은 OPTIND 값을 가집니다.
- sh 의 경우에는 b, 3 가 출력됩니다. 
```bash
$ getopts abc opt "$@"
$ echo $opt, $OPTIND
b, 2
```

#
- args 부분을 생략하면 default 값은 "$@" 입니다.
```bash
$ getopts abc opt
$ echo $opt, $OPTIND
c, 3
```

#
- 옵션 스트링을 처리하고 난후 다음과 같이 shift 를 하면 나머지 명령 인수만 남게 됩니다.
```bash
$ echo "$@"
-a -bc hello world

$ shift $(( OPTIND - 1 ))
$ echo "$@"
hello world
```
#
- 옵션을 붙여쓰기 할 경우 sh 과 bash 가 $OPTIND 값을 처리하는 방식이 약간 다릅니다.   
하지만 최종적으로 shift $(( OPTIND - 1 )) 한 값은 둘 다 같게 됩니다.
```bash
#!/bin/bash 의 경우                                #!/bin/sh 의 경우

set -- -abc hello world                          set -- -abc hello world

echo $OPTIND                                     echo $OPTIND

getopts abc opt                                  getopts abc opt 
echo $opt, $OPTIND                               echo $opt, $OPTIND 

getopts abc opt                                  getopts abc opt 
echo $opt, $OPTIND                               echo $opt, $OPTIND

getopts abc opt                                  getopts abc opt  
echo $opt, $OPTIND                               echo $opt, $OPTIND

shift $(( OPTIND - 1 ))                          shift $(( OPTIND - 1 ))
echo "$@"                                        echo "$@"
---------------------------                      ---------------------------

1         (index 1 은 -abc 를 가리킴                1         (index 1 은 -abc 를 가리킴
a, 1      (index 1 은 -abc 를 가리킴                a, 2      (index 2 는 hello 를 가리킴
b, 1      (index 1 은 -abc 를 가리킴                b, 2      (index 2 는 hello 를 가리킴
c, 2      (index 2 는 hello 를 가리킴               c, 2      (index 2 는 hello 를 가리킴
hello world                                       hello world
```
-------
### Option argument   
옵션은 옵션인수를 가질 수 있는데, 이때는 옵션 스트링에서 해당 옵션 문자 뒤에 `:` 을 붙입니다.
그러면 getopts 명령은 옵션인수 값를 `$OPTARG` 변수에 설정해 줍니다.
```bash
$ OPTIND=1
$ set -- -a xyz -b -c hello world

# 옵션 a 가 옵션인수를 가지므로 옵션 스트링으로 a: 를 설정하였습니다.
$ getopts a:bc opt
$ echo $opt, $OPTARG, $OPTIND        
a, xyz, 3                        # 옵션인수가 포함되므로 OPTIND 값이 3 이 되었음.

$ getopts a:bc opt
$ echo $opt, $OPTARG, $OPTIND
b, , 4

$ getopts a:bc opt
$ echo $opt, $OPTARG, $OPTIND
c, , 5
```
-----
### loop 문을 이용해 처리
명령문에서 사용된 모든 옵션을 처리하기 위해서 다음과 같이 while 과 case 문을 이용합니다.
```bash
#!/bin/bash

while getopts "a:bc" opt; do
  case $opt in
    a)
      echo >&2 "-a was triggered!, OPTARG: $OPTARG"
      ;;
    b)
      echo >&2 "-b was triggered!"
      ;;
    c)
      echo >&2 "-c was triggered!"
      ;;
  esac
done

shift $(( OPTIND - 1 ))
echo "$@"
..................................

$ ./test.sh -a xyz -bc hello world
-a was triggered!, OPTARG: xyz
-b was triggered!
-c was triggered!
hello world
```
------
### Error reporting
위 예제에서 옵션 스트링에 없는 문자, 예를 들면 `-d` 를 사용하게 되면 오류 메시지가 출력되는 것을 볼 수 있는데, getopts 명령은 error reporting 과 관련해서 다음과 같은 두 개의 모드를 제공합니다.
|Verbose mode|   |
|------|---|
|invalid 옵션 사용|opt 값을 `?` 문자로 설정하고 OPTARG 값은 unset. 오류 메시지를 출력.|
|옵션인수 값을 제공하지 않음|opt 값을 `?` 문자로 설정하고 OPTARG 값은 unset. 오류 메시지를 출력.|   

|Silent mode|   |
|------|---|
|invalid 옵션 사용|opt 값을 `?` 문자로 설정하고 OPTARG 값은 해당 옵션 문자로 설정|
|옵션인수 값을 제공하지 않음|opt 값을 `:` 문자로 설정하고 OPTARG 값은 해당 옵션 문자로 설정|

default 는 verbose mode 인데 기본적으로 옵션과 관련된 오류메시지가 표시되므로 스크립트를 배포할 때는 잘 사용하지 않고 대신 silent mode 를 이용합니다. silent mode 를 설정하기 위해서는 옵션 스트링의 맨 앞부분에 `:` 문자를 추가해 주면 됩니다.

```bash
#!/bin/bash

# silent mode 를 설정하기 위해 옵션 스트링의 맨 앞부분에 ':' 문자를 추가
while getopts ":a:" opt; do
  case $opt in
    a)
      echo >&2 "MSG: -a was triggered, argument: $OPTARG"
      ;;
    \?)   # \? 는 escape 했으므로 glob 문자가 아님.
      echo >&2 "ERR: Invalid option: -$OPTARG"
      exit 1
      ;;
    :)
      echo >&2 "ERR: Option -$OPTARG requires an argument."
      exit 1
      ;;
  esac
done
------------------------------------------------------

$ ./test.sh -a 
ERR: Option -a requires an argument.

$ ./test.sh -a xyz
MSG: -a was triggered, argument: xyz

$ ./test.sh -d
ERR: Invalid option: -d
```
-------
### 주의할 점   
OPTIND, OPTARG 변수는 local 변수가 아니므로 필요할 경우 함수 내에서 local 로 설정해 사용해야 합니다. getopts 명령은 옵션인수 사용과 관련해서 다음과 같이 주의할 점이 있습니다.   
#
옵션 스트링이 'a:bc' 이면 -a 는 옵션인수를 갖는데, 옵션인수는 어떤 문자도 올 수 있기 때문에 다음과 같이 -a 에 옵션인수가 설정되지 않으면 -b 가 -a 의 옵션 인수가 됩니다.
```bash
$ command -a -b -c
```
#
파일명이나 기타 스트링은 마지막에 와야하는데 그렇지 않을 경우 이후 옵션은 인식되지 않습니다.   
다음의 경우 옵션 스트링이 'abc' 라면 -b -c 옵션은 인식되지 않습니다.
```bash
$ command -a foo.c -b -c
```
-------
### 예제)
```bash
#!/bin/bash

usage() {
    err_msg "Usage: $(basename "$0") -s <45|90> -p <string>"
    exit 1
}

err_msg() { echo "$@" ;} >&2
err_msg_s() { err_msg "-s option argument 45 or 90 required" ;}
err_msg_p() { err_msg "-p option argument required" ;}

while getopts ":s:p:" opt; do
    case $opt in
        s)
            s=$OPTARG
            [ "$s" = 45 ] || [ "$s" = 90 ] || { err_msg_s; usage ;}
            ;;
        p)
            p=$OPTARG    # -p -s 45 또는 -p -s45 일경우
            [[ $p =~ ^-s ]] && { err_msg_p; usage ;}
            ;;
        :)
            case $OPTARG in
                s) err_msg_s ;;
                p) err_msg_p ;;
            esac
            usage
            ;;
        \?)
            err_msg "Invalid option: -$OPTARG"
            usage
            ;;
    esac
done

if [ -z "$s" ] || [ -z "$p" ]; then
    usage
fi

echo "s = $s"
echo "p = $p"
```
------
### Long 옵션의 처리
```bash
$ ./test.sh -a aaa --posix --long 123 -b --warning=2 -- hello world
```
short 옵션 처리에 대해 알아보았는데요. 만약에 위와 같은 명령문을 getopts 으로 처리한다면 옵션 스트링으로 `:a:b-` 를 사용하고 case 문에서는 `-)` 를 사용해야`--` 로 시작하는 long 옵션을 받을 수 있습니다. 그런데 여기서 문제는 short 옵션은 하나의 문자를 옵션으로 보기 때문에 이후에 `p`, `o`, `s`, `i`, `x` 가 모두 붙여쓰기한 옵션명으로 인식을 하게 됩니다. 또 한 가지 문제점은 위의 `--long` 옵션과 같이 `123` 옵션인수를 사용하게 되면 그 이후의 옵션 그러니까 `-b` 는 getopts 에 의해 인식이 되지 않습니다.   

따라서 getopts 명령으로 short, long 옵션을 동시에 처리하는 것은 어려우므로 먼저 long 옵션을 처리하고 난후 나머지 short 옵션만 정리하여 getopts 에 넘겨주면 이전과 동일하게 short 옵션을 처리할 수 있습니다.   
#
- 처리한 long 옵션은 삭제하고 short 옵션만 getopts 명령에 전달.
```bash
-a aaa -b -- hello world
```
#
long 옵션을 `$@` 에서 삭제하는 방법은 while 문을 이용해 인수들을 하나씩 처리하면서 long 옵션이 아닐 경우 특정 변수에 계속해서 append 하는 것입니다. 마지막으로 long 옵션이 제거된 변수를 이용해 set 명령으로 다시 `$@` 값을 설정합니다.
```bash
# shift 명령을 이용해 항상 '$1' 에는 다음 인수가 설정되게 합니다.
while true; do
    case $1 in
        --optionA)
        ...
        shift; continue   # long 옵션일 경우 continue 명령을 사용함으로써
        ;;                # 변수 'A' 에 값이 저장되지 않게 합니다.
        --optionB)
        ...
        ...
    esac

    # long 옵션이 아닐 경우 변수 'A' 에 append 하는 과정입니다.
    # 이때 인수들의 구분자로 non-printing 문자인 '\a' 를 사용합니다.
    # (공백을 사용하게 되면 인수값에 공백이 포함될 경우 인수값도 분리가 됩니다)
    A=$A$([ -n "$A" ] && echo -e "\a")$1
    shift
done

# 마지막으로 set -f 로 globbing 방지 처리를 하고 IFS 값을 '\a' 로 설정하여 
# set -- $A 명령으로 '$@' 값을 다시 설정합니다.
IFS=$(echo -e "\a"); set -f; set -- $A; set +f; IFS=$(echo -e " \n\t")
```
------------------
다음은 전체 코드 내용입니다.
> `sh` 에서 사용하려면 본문의 `echo -e` 를 `echo` 로 변경해 주면 됩니다.
```bash
#!/bin/bash

usage() {
    err_msg "Usage: $0 -a <string> -b --long <string> --posix --warning[=level]"
    exit 1
}

err_msg() { echo "$@" ;} >&2
err_msg_a() { err_msg "-a option argument required" ;}
err_msg_l() { err_msg "--long option argument required" ;}

########################## long 옵션 처리 부분 #############################
A=""
while true; do
    [ $# -eq 0 ] && break
    case $1 in
        --long) 
            shift    # 옵션인수를 위한 shift
            # --long 은 옵션인수를 갖는데 옵션인수가 오지 않고 다른 옵션명(-*) 이 오거나
            # 명령의 끝에 위치하여 옵션인수가 설정되지 않았을 경우 ("")
            case $1 in (-*|"") err_msg_l; usage; esac
            err_msg "--long was triggered!, OPTARG: $1"
            shift; continue
            ;;
        --warning*)
            # '=' 로 분리된 level 값을 처리하는 과정입니다.
            case $1 in (*=*) level=${1#*=}; esac
            err_msg "--warning was triggered!, level: $level"
            shift; continue
            ;;
        --posix) 
            err_msg "--posix was triggered!"
            shift; continue
            ;;
        --)
            # '--' 는 옵션의 끝을 나타내므로 나머지 값 '$*' 을 A 에 append 하고 break 합니다.
            # 이때 IFS 값을 '\a' 로 변경해야 $* 내의 인수 구분자가 '\a' 로 됩니다.
            IFS=$(echo -e "\a")
            A=$A$([ -n "$A" ] && echo -e "\a")$*
            break
            ;;
        --*) 
            err_msg "Invalid option: $1"
            usage;
            ;;
    esac

    A=$A$([ -n "$A" ] && echo -e "\a")$1
    shift
done

# 이후부터는 '$@' 값에 short 옵션만 남게 됩니다.
# -a aaa -b -- hello world 
IFS=$(echo -e "\a"); set -f; set -- $A; set +f; IFS=$(echo -e " \n\t")

########################## short 옵션 처리 부분 #############################

# 이전과 동일하게 short 옵션을 처리하면 됩니다.
while getopts ":a:b" opt; do
    case $opt in
        a)
            case $OPTARG in (-*) err_msg_a; usage; esac
            err_msg "-a was triggered!, OPTARG: $OPTARG"
            ;;
        b)
            err_msg "-b was triggered!"
            ;;
        :)
            case $OPTARG in
                a) err_msg_a ;;
            esac
            usage
            ;;
        \?)
            err_msg "Invalid option: -$OPTARG"
            usage
            ;;
    esac
done

shift $(( OPTIND - 1 ))
echo ------------------------------------
echo "$@"

########################## 실행 결과  #############################

$ ./test.sh -a 'aaa bbb' --posix --long 123 -b --warning=2 -- hello world 
--posix was triggered!
--long was triggered!, OPTARG: 123
--warning was triggered!, level: 2
-a was triggered!, OPTARG: aaa bbb
-b was triggered!
------------------------------------
hello world
```
-------
### getopt 외부 명령
- `getopt` 은 이름이 `getopts` builtin 명령과 비슷한데 `/usr/bin/getopt` 에 위치한 외부 명령입니다. 이 명령은 기본적으로 short, long 옵션을 모두 지원합니다. 옵션 인수를 가질 경우 `:` 문자를 사용하는 것은 getopts builtin 명령과 동일합니다.
#
- short 옵션 지정은 -o 옵션으로 합니다.
- ':' 에 따라서 옵션 -a 는 옵션 인수를 갖습니다. 
```bash
getopt -o a:bc
```
#
- long 옵션 지정은 -l 옵션으로 하고 옵션명은 ',' 로 구분합니다.
- ':' 에 따라서 옵션 --path 와 --name 은 옵션 인수를 갖습니다.
- 명령 라인에서 옵션 인수 사용은 "--name foo" 또는 "--name=foo" 두 가지 모두 가능합니다.
```bash
getopt -l help,path:,name: 
```
#
- 명령 마지막에는 -- 와 함께 "$@" 를 붙입니다.
```bash
getopt -o a:bc -l help,path:,name: -- "$@"
```
#
#
설정하지 않은 옵션이 사용되거나 옵션 인수가 빠질 경우 오류메시지를 출력해줍니다.
```bash
#!/bin/bash

options=$( getopt -o a:bc -l help,path:,name: -- "$@" )
echo "$options"
-----------------------------------------------------

$ ./test.sh -x
getopt: invalid option -- 'x'

$ ./test.sh --xxx
getopt: unrecognized option '--xxx'

$ ./test.sh -a
getopt: option requires an argument -- 'a'

$ ./test.sh --name
getopt: option '--name' requires an argument
```
#
getopt 명령의 특징은 사용자가 입력한 옵션들을 case 문에서 사용하기 좋게 정렬해준다는 것입니다.
```bash
#!/bin/bash

options=$( getopt -o a:bc -l help,path:,name: -- "$@" )
echo "$options"
-----------------------------------------------------

# 1. -a123 옵션이 -a '123' 로 분리.
# 2. -bc 옵션이 -b -c 로 분리.
# 3. 옵션에 해당하지 않는 hello.c 는 '--' 뒤로 이동
$ ./test.sh -a123 -bc hello.c
-a '123' -b -c -- 'hello.c'

# --path=/usr/bin 옵션이 --path '/usr/bin' 로 분리
# '--' 는 항상 끝부분에 붙는다.
$ ./test.sh --name foo --path=/usr/bin
--name 'foo' --path '/usr/bin' --

# '--' ( end of options ) 처리도 해줍니다.
$ ./test.sh -a123 -bc hello.c -- -x --yyy
 -a '123' -b -c -- 'hello.c' '-x' '--yyy'
```
#
getopt 외부 명령의 경우는 옵션에 해당하지 않는 hello.c 를 -- 뒤로 정렬해 줍니다.
```bash
#!/bin/bash

options=$(getopt -o a:bc -- "$@")
echo $options
.................................

$ ./test.sh -a 123 hello.c -bc
-a '123' -b -c -- 'hello.c'
```
#
### 예제)
```bash
#!/bin/bash

if ! options=$(getopt -o hp:n: -l help,path:,name:,aaa -- "$@")
then
    echo "ERROR: print usage"
    exit 1
fi

eval set -- $options

while true; do
    case "$1" in
        -h|--help) 
            echo >&2 "$1 was triggered!"
            shift ;;
        -p|--path)    
            echo >&2 "$1 was triggered!, OPTARG: $2"
            shift 2 ;;   # 옵션 인수를 가지므로 shift 2 를 합니다.
        -n|--name)
            echo >&2 "$1 was triggered!, OPTARG: $2"
            shift 2 ;;
        --aaa)     
            echo >&2 "$1 was triggered!"
            shift ;;
        --)           
            shift 
            break
    esac
done

echo --------------------
echo "$@"
-----------------------------------------------------------------

$ ./test.sh -h hello.c -p /usr/bin --name='foo bar' --aaa -- --bbb
-h was triggered!
-p was triggered!, OPTARG: /usr/bin
--name was triggered!, OPTARG: foo bar
--aaa was triggered!
--------------------
hello.c --bbb
```
****
## Linux sed, awk 명령어
### Awk
+ <b>자료 처리</b> 및 <b>리포트 생성</b>에 사용하는 프로그래밍 언어
+ 사용자가 지정한 <b>패턴 검색</b>이나 <b>특별한 작업</b>을 수행하기 위해 파일을 <b>행 단위로 조사</b>
-----
### awk 명령에 사용되는 NR , NF 변수 와 옵션
+ `NR 변수` - 하나의 레코드를 처리한 후 1이 증가하는 변수. (즉, <b>해당 패턴이 몇 번째 행에 있는지</b> 나타내 주는 변수)
+ `NF 변수` - 필드의 개수를 출력하는 변수.
+ `-F 옵션` - 공백문자를 필드 구분자로 사용하지 않고 필드 구분자를 무엇으로 사용할지 지정하는 옵션.
+ `~` 와 `! - ~`는 특정 레코드나 필드내에서 일치하는 정규 표현식 패턴이 존재하는지 검사.   
ex) `# awk '$1 ~ /[aA]pple/' [파일명]` --> 해당 파일의 첫번째 필드에 apple or Apple이 일치하는 것을 뽑아냄.
+ `!` 는 not이다. 반대되는 개념. 즉, 패턴과 일치하지 않는 것을 뽑아낸다.   
ex) `# awk '$1 !~ /[aA]pple/' [파일명]` --> 해당 파일의 첫번째 필드에 apple or Apple와 일치하지 않는 것을 뽑아냄.
-----
### 예제)
1. NR 변수를 사용하여 해당 awk 에 의해 몇 개의 레코드가 검색되었는지 맨 앞과 맨 뒤에 나타내라.
```vim
# awk '{print NR, $1, $2}' file

# awk '{print $1, $2, NR}' file
```
+ {print 뒤를 순서대로 출력할 것이다. 즉, NR은 { }안의 어느 위치 든 쓸 수 있다.   

#
2.  NF 변수를 사용하여 검색된 해당 레코드에 몇 개의 필드가 있는지 나타내라.
```vim
# awk '{print NF, $1, $3}' file
```
+ 첫 번째와 3번째 필드만 출력하되 NF는 해당 레코드에 몇 개의 필드가 있는지 표시.
#
3. -F옵션을 사용하여 : 를 구분자로 사용 하여 첫번째 필드와 세 번째 필드를 출력하라.
```vim
# awk -F : '{print $1, $3}' passwd --> -F [구분자]
```
#
------
### sed
- sed는 대화형 기능이 없는 편집기이다. 기본적으로 모든 행을 출력한다.
- 한 번에 한 행씩 처리하며, 결과를 화면으로 내보낸다. (리 다이렉션 > 을 통해 저장도 가능)
```vim
# sed '명령어' [파일명]
```
#
### OPTION
|OPTION|   |
|---|-----|
|`-n`|해당 패턴 or 실행 결과만을 출력 (기본적으로 출력되는 모든 행을 막는다.)|
|`-e`|다중 편집을 가능하게 해준다.|
#
### sed 명령어 FORM
|FORM|    |
|---|-----|
|`p`|print 명령 사용자가 지정한 행을 출력 <br><b> ex) `'1,3p'` -> 1행부터 3행까지 출력</b>|
|`d`|delete명령 사용자가 지정한 행을 삭제 <br><b> ex) `'3d'` -> 3번째 행을 삭제 </b>|
|`r`|read명령 사용자가 지정한 행을 읽어온다. <br><b> ex) `'3r phone' [패턴검색파일]` --> 패턴검색할 파일의 3번째 행에 phone의 내용을 읽어와 출력.</b>|
|`w`|write명령 사용자가 지정한 행에 쓴다. <br><b> ex) `'/패턴/w [쓰여질파일]' [패턴검색파일]` --> 패턴에 일치하는 행을 검색하여 [쓰여질 파일]에 쓴다. </b>|
|`a\`|append명령 검색된 패턴 아래 행에 내용을 덧붙인다. <br><b> ex) `'/패턴/a\내용' [파일명]` --> 패턴에 일치하는 행의 다음에 내용을 삽입하여 출력.</b>|
|`i\`|insert명령 검색된 패턴 위에 내용을 덧붙인다. a\와 사용법은 동일.|

  ---
  ### sed와 awk의 차이점
  + 정의   
  `sed`는 간단하고 컴팩트 한 프로그래밍 언어를 사용하여 텍스트를 구문 분석하고 변환하는 명령 줄 유틸리티.   
  `awk`는 효과적인 프로그램을 명령문의 형태로 작성할 수있는 텍스트 처리 용으로 설계된 명령 행 유틸리티.   
#
  + 복잡성   
  `sed`는 간단하고 제한적이지만 `awk`는 더 복잡하고 다재다능.   
#  
  + 결론   
  sed와 awk의 차이점은 sed는 검색, 필터링 및 텍스트 처리를 위해 문자 스트림과 함께 작동하는 명령 유틸리티 인 반면, awk는 if / else, while, do / while 등과 같은 정교한 프로그래밍 구조로보다 강력하고 강력함.

-------
## 출처

+ https://www.ibm.com/docs/ko/aix/7.2?topic=g-getopt-command
+ https://mug896.github.io/bash-shell/getopts.html
+ https://systemdesigner.tistory.com/
+ https://m.blog.naver.com/PostView.naver?isHttpsRedirect=true&blogId=green187&logNo=110129093339
  
