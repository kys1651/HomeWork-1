# HomeWork-1 

## getopt(명령행 플래그와 매개변수를 구문분석)
* getopt는 플래그와 인수를 지정하는 형식으로 토큰 리스트를 구문 분석한다. 플래그는 단일 아스키이며 뒤에 :(콜론)이 올 경우 하나 이상의 탭 또는 공백으로 분리하거나 분리할 수 없는 인수가 있어야 한다. 인수에는 복수 바이트를 포함시킬 수 있지만 플래그 문자로는 포함시킬 수 없다.

* getopt 명령은 모든 토큰을 읽은 후 또는 특수 토큰 -(하이픈)이 발생하는 경우 처리를 완료한다. 그러면 getopt 명령은 처리된 플래그, -(더블 하이픈) 및 남아 있는 토큰을 출력한다.
 
* 설정되지 않은 옵션을 사용하거나 인자가 빠진 경우에는 getopt 명령은 메시지를 출력한다.
 
* /usr/bin/getopt 에 위치한 외부 명령이다.

### getopt의 옵션
***
* **getopt에서는 기본적으로 short와 long옵션을 모두 지원한다.**
```shell
# short 옵션 지정은 -o 옵션으로 한다.
# ':' 에 따라서 옵션 -a 는 옵션 인수를 갖는다. 
$ getopt -o a:bc

# long 옵션 지정은 -l 옵션으로 하고 옵션명은 ',' 로 구분한다.
# ':' 에 따라서 옵션 --path 와 --name 은 옵션 인수를 갖는다.
# 명령 라인에서 옵션 인수 사용은 "--name foo" 또는 "--name=foo" 두 가지 모두 가능하다.
$ getopt -l help,path:,name: 

# 명령 마지막에는 -- 와 함께 "$@" 를 붙인다.
$ getopt -o a:bc -l help,path:,name: -- "$@"
```
* **getopt명령은 사용자가 입력한 옵션들을 case 문에서 사용하기 좋게 정렬해준다.**

```shell
# -a123 옵션이 -a '123' 로 분리.
# -bc 옵션이 -b -c 로 분리.
# 옵션에 해당하지 않는 hello.c 는 '--' 뒤로 이동
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
### 예제
***
```shell
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
            shift 2 ;;   # 옵션 인수를 가지므로 shift 2 를 한다.
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
```

<img src="https://user-images.githubusercontent.com/43926186/142719336-b291a778-801d-4179-b735-e71e12517dc4.PNG" width="90%" height="85%"/>

## getopts(명령행 인수를 처리하고 유효한 옵션을 검사)
* getopts 명령은 매개변수 리스트에서 옵션 및 옵션 인수를 검색하는 쉘 내장 명령이다.
 
* 사용된 옵션은 다른 인수들과 마찬가지로 $1, $2, ... positional parameters 형태로 전달되므로 스크립트 내에서 직접 옵션을 해석해서 사용해야 한다. 이때 옵션 해석 작업을 쉽게 도와주는 명령이다.

* 옵션에는 short 옵션과 long 옵션이 있는데 getopts 명령은 short 옵션을 처리한다.

### Option string 와 $OPTIND
***
* **명령에서 -a, -b, -c 세 개의 옵션을 사용한다면 getopts 명령에 설정하는 optstring 값은 abc 가 된다. args 에는 명령 실행시 사용된 인수 값들이 오는데 생략할 경우 "$@" 가 사용된다.**
**예제

```shell
# 다음 set 명령은 실제 "command -a -bc hello world" 명령을 실행했을 때와 같이
# positional parameters 를 설정한다.
```
<img src="https://user-images.githubusercontent.com/43926186/142719666-db5c0663-100f-4898-94d0-06bff08afd8f.PNG" width="90%" height="85%"/>

```shell
# 처음 $OPTIND 값은 1 로 -a 를 가리킨다.
```
<img src="https://user-images.githubusercontent.com/43926186/142719906-17f0851f-e7fb-45f4-91e7-6585e1ccbfa7.PNG" width="90%" height="85%"/>

```shell
# getopts 명령을 실행할 때마다 opt 변수값과 OPTIND 값이 변경되는 것을 볼 수 있다.
 # 다음 옵션 "b" 의 index 값은 2 가 된다.
```
<img src="https://user-images.githubusercontent.com/43926186/142719910-0c14d7b1-cd87-4881-8d96-20d21063758b.PNG" width="90%" height="85%"/>

```shell
# 다음 옵션 "c" 는 "b" 와 붙여 쓰기를 하여 같은 OPTIND값을 가진다.
# ( sh 의 경우에는 b, 3 가 출력된다. )
```

<img src="https://user-images.githubusercontent.com/43926186/142719914-971c340c-9dd7-41cf-b752-82981d1f27cc.PNG" width="90%" height="85%"/>
```shell
# args 부분을 생략하면 default 값은 "$@" 이다.
```
<img src="https://user-images.githubusercontent.com/43926186/142719920-4fd53818-a61c-4779-bc89-5f89f39fac7c.PNG" width="90%" height="85%"/>

```shell 
# 옵션 스트링을 처리하고 난후 다음과 같이 shift 를 하면 나머지 명령 인수만 남게 된다.
```
<img src="https://user-images.githubusercontent.com/43926186/142719923-e526eb06-eed9-401e-91d0-c634f00d305e.PNG" width="90%" height="85%"/>

### getopts의 옵션
***
1) **short 옵션**
* short 옵션은 다음과 같이 여러 가지 방법으로 사용할 수 있다. getopts 명령을 이용하지 않고 직접 옵션을 해석해 처리한다면 옵션 처리에만 스크립트가 복잡해질 수 있다.**
```shell
$ command -a -b -c

# 옵션을 붙여서 사용할 수 있으며 순서가 바뀌어도 된다.
$ command -abc
$ command -b -ca

# 옵션인수를 가질 수 있다.
$ command -a xxx -b -c yyy

# 옵션인수를 옵션에 붙여 쓸 수가 있다.
# 그러므로 다음은 위의 예와 동일하게 해석된다.
$ command -axxx -bcyyy

# 옵션 구분자 '--' 가 올경우 우측에 있는 값은 옵션으로 해석하면 안된다.
$ command -a -b -- -c
```
2) **long 옵션**
* --posix, --warning level 와 같은 형태로 사용되는 long 옵션은 short 옵션과는 달리 붙여 쓸 수가 없기 때문에 사용방법이 간단하여 직접 해석해서 처리하는 것이 어렵지 않다.

* 명령문을 getopts 으로 처리한다면 옵션 스트링으로 :a:b- 를 사용하고 case 문에서는 -) 를 사용해야 -- 로 시작하는 long 옵션을 받을 수 있다.

* hort 옵션은 하나의 문자를 옵션으로 보기 때문에 이후에 p, o, s, i, x 가 모두 붙여쓰기한 옵션명으로 인식을 하게 된다.

> 따라서 getopts 명령으로 short, long 옵션을 동시에 처리하는 것은 어려우므로 먼저 long 옵션을 처리하고 난후 나머지 short 옵션만 정리하여 getopts 에 넘겨주면 이전과 동일하게 short 옵션을 처리할 수 있다.


### 오류 출력
***
위 예제에서 옵션 스트링에 없는 문자, 예를 들면 -d 를 사용하게 되면 오류 메시지가 출력되는 것을 볼 수 있는데, getopts 명령은 error reporting 과 관련해서 다음과 같은 두 개의 모드를 제공한다.

|**Verbose mode**||
|:---:|:----:|
|invalid 옵션 사용|opt 값을 ? 문자로 설정,OPTARG 값은 unset, 오류 메시지 출력|
|옵션 인수 값 제공 x|opt 값을 ? 문자로 설정,OPTARG 값은unset, 오류 메시지 출력|

|**Silent mode**||
|:----:|:----:|
|invalid 옵션 사용|opt 값을 ? 문자로 설정,OPTARG 값은 해당 옵션 문자로 설정|
|옵션 인수 값 제공 x|opt 값을 ? 문자로 설정,OPTARG 값은 해당 옵션 문자로 설정|

> default 는 verbose mode 인데 기본적으로 옵션과 관련된 오류메시지가 표시되므로 스크립트를 배포할 때는 잘 사용하지 않고 대신 silent mode 를 이용한다. silent mode 를 설정하기 위해서는 옵션 스트링의 맨 앞부분에 : 문자를 추가해 주면 된다.

### 주의해야하는 점
***
* OPTIND, OPTARG 변수는 local 변수가 아니므로 필요할 경우 함수 내에서 local 로 설정해 사용해야 한다.
```shell
# 옵션 스트링이 'a:bc' 이면 -a 는 옵션인수를 갖는데, 옵션인수는 어떤 문자도 올 수 있기 때문에
# 다음과 같이 -a 에 옵션인수가 설정되지 않으면 -b 가 -a 의 옵션 인수가 된다.
$ command -a -b -c

# 파일명이나 기타 스트링은 마지막에 와야하는데 그렇지 않을 경우 이후 옵션은 인식되지 않는다.
# 다음의 경우 옵션 스트링이 'abc' 라면 -b -c 옵션은 인식되지 않는다.
$ command -a foo.c -b -c
```
### 예제
***
```shell
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
            # '=' 로 분리된 level 값을 처리하는 과정
            case $1 in (*=*) level=${1#*=}; esac
            err_msg "--warning was triggered!, level: $level"
            shift; continue
            ;;
        --posix) 
            err_msg "--posix was triggered!"
            shift; continue
            ;;
        --)
            # '--' 는 옵션의 끝을 나타내므로 나머지 값 '$*' 을 A 에 append 하고 break 
            # 이때 IFS 값을 '\a' 로 변경해야 $* 내의 인수 구분자가 '\a' 로 됨.
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

# 이후부터는 '$@' 값에 short 옵션만 남는다.
# -a aaa -b -- hello world 
IFS=$(echo -e "\a"); set -f; set -- $A; set +f; IFS=$(echo -e " \n\t")

########################## short 옵션 처리 부분 #############################

# 이전과 동일하게 short 옵션을 처리한다.
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
```
<img src="https://user-images.githubusercontent.com/43926186/142720564-f5502965-c86a-4d3a-bb3b-d9033a9650c6.PNG" width="90%" height="85%"/>

## sed(스트림 편집기)
* ed명령어와 grep명령어 기능의 일부를 합친 것이 sed(stream editor)명령어이다.

* sed는 각 라인을 읽을 때마다 ed에서 사용하던 형식의 대치작업을 실행한다. 일치하는 문자열이 있으면 그 문자열을 대치한 후 출력하고 일치하는 문자열이 없으면 그 라인은 수정되지 않고 그대로 출력된다.

* sed 명령어를 호출하는 형식은 grep명령어와 같지만 완전한 형식의 대치 연산자를 사용한다는 점만이 다르다.

### sed 사용법
***
* 기본


`$ sed [옵션] 스크립트 입력파일1 [입력파일2 ... ]`
* 옵션


|**옵션**|**별칭**|**기능**|
|:---|:----|:-----|
|**-n**|–quiet, –silent|읽어들인 라인을 암시적으로 자동출력하는 것을 중단한다.|
|**-e**|–expression=script|실행될 명령에 스크립트를 추가한다|
|**-f script_file**|–file=script_file|스크립트 파일의 내용을 가져와서 추가로 실행한다.|
|–follow-symlinks||제자리 처리시에 심볼릭 링크를 따르도록 한다. 하드링크는 깨진다.|
|-i[SUFFIX]|–in-place[=SUFFIX]|파일을 제자리 처리한다. (즉 변경된 내용을 파일에 적용한다.)|
|-c||제자리 처리시에 사본을 이용한다.|
|-l N|–copy|한줄의 길이를 정의한다.|
||–line-length=N|모든 GNU확장을 제외한다.–line-length=N|
|-r|–posix|확장된 정규식 패턴을 사용한다.|
|-s|–regexp-extended|파일을 하나의 긴 스트림이 아닌 분리된 데이터들로 처리한다.|
|-u|	–separate|입력으로부터 최소한의 내용만 읽고 더 자주 플러시한다.|
> 진하게 표시된것은 자주 사용하는것


### sed 예시
***

1) 치환 예시
```shell
#특정 문자열 바꾸기
$ sed 's/hello/goodbye/' tempfile.txt
 
#특정 문자열이 포함된 행에서 특정 문자만 제거 하기
$ sed 's/hello//g' tempfile.txt
 
#파일의 일부만 편집해야 할 경우 라인의 범위를 지정하여 사용가능
$ sed '3,7s/hello//g' tempfile.txt
 
#라인 번호 대신 문맥을 범위로 지정한 경우
$ sed '/hello/,/goodbye/s/bad/good' tmepfile.txt
$ sed '/hello/,/goodbye/s/bad/good/g' tmepfile.txt
 
#특정라인의 문자 치환
$ sed -n '5s/hello/####/g' tmepfile.txt
```
2) 삭제 예시
```shell
#특정 문자열 포함한 라인 제거
$ sed 's/hello/d' tempfile.txt
 
#3번째 라인을 삭제 
$ sed '3d' tempfile.txt
 
#princess 또는 Princess를 포함하고 있는 라인들을 삭제함
$ sed '/[hH]ello/d' tempfile.txt
 
#라인 1부터 hello를 포함하고 있는 첫번째 라인까지 모든 라인을 삭제함
$ sed '1,/hello/d' tempfile.txt
 
#tempfile.xtxt 안의 데이터로부터 처음 세 개의 문자열을 삭제함
$ sed 's/...//' tempfile.txt
 
#tempfile.xtxt 안의 데이터로부터 마지막 세 개의 문자열을 삭제함
$ sed 's/...$//' tempfile.txt
 
#모든 공백라인 제거
$ sed '^$/d' tempfile.txt
 
#여기서부터는 실제 cli창에 테스트 해보세요!
#각 라인의 첫번째 공백에서 마지막까지 삭제함
$ who | sed 's/ .*$//'
 
#각 라인의 처음부터 맨 마지막 공백까지 삭제
$ who | sed 's/^.*$//'
 
#각 라인의 처음부터 : 문자가 있는 곳(:문자포함)까지 삭제
$ who | sed 's/^.*://'
```

3) 입/출력 예시
```shell
#각 라인마다 뒤에 Hello World! 문자를 입력
$ sed 'a\\Hello World\!' tempfile.txt
 
#3번째라인 뒤에 Good Morning 문자 삽입
$ sed '3a\\Good Morning' tempfile.txt
 
#각 라인마다 공백라인 추가
$ sed 'a\\' tempfile.txt
 
#hello 문자가 들어있는 line만 프린트
$ sed -n '/hello/p' tempfile.txt
 
#hello 문자가 들어있는 라인 중복 출력?
$ sed '/hello/p' tempfile.txt
```


## awk(파일에서 패턴이 일치하는 행을 찾아서 그러한 행에 지정한 조치를 수행)
* awk는 파일로부터 레코드(record)를 선택하고, 선택된 레코드에 포함된 값을 조작하거나 데이터화하는 것을 목적으로 사용한다.

* awk 명령의 입력으로 지정된 파일로부터 데이터를 분류한 다음, 분류된 텍스트 데이터를 바탕으로 패턴 매칭 여부를 검사하거나 데이터 조작 및 연산 등의 액션을 수행하고, 그 결과를 출력한다.

* 패턴, 분류, 텍스트, 조작, 연산, 액션. 뭔가 복잡한 작업을 하는 프로그램이다.

### awk 명령으로 할 수 있는 일
***
1) 텍스트 파일의 전체 내용 출력한다.
2) 파일의 특정 필드만 출력한다.
3) 특정 필드에 문자열을 추가해서 출력한다/.
4) 패턴이 포함된 레코드 출력한다.
5) 특정 필드에 연산 수행 결과 출력한다.
6) 필드 값 비교에 따라 레코드 출력한다.
### awk 명령어 옵션
***
```shell
awk [OPTION...] [awk program] [ARGUMENT...]
      OPTION
        -F        : 필드 구분 문자 지정.
        -f        : awk program 파일 경로 지정.
        -v        : awk program에서 사용될 특정 variable값 지정.
      awk program
        -f 옵션이 사용되지 않은 경우, awk가 실행할 awk program 코드 지정.
      ARGUMENT
        입력 파일 지정 또는 variable 값 지정.
```
### awk 사용 예시
***
> 1) 파일의 모든 내용 출력
<img src="https://user-images.githubusercontent.com/43926186/142721829-905a6dca-6394-4c23-91a6-551fc51809b8.PNG" width="90%" height="85%"/>

> 2) 파일의 특정 필드 부분만 출력
<img src="https://user-images.githubusercontent.com/43926186/142721851-892a371d-97c2-476f-8355-7b848dd544da.PNG" width="90%" height="85%"/>

> 3) 필드 값에 임의 문자열을 같이 출력.
<img src="https://user-images.githubusercontent.com/43926186/142721867-f726b673-1cff-4604-91a0-0004c0068997.PNG" width="90%" height="85%"/>

> 4) 지정된 문자열을 포함하는 레코드만 출력.
<img src="https://user-images.githubusercontent.com/43926186/142721877-06dd21f1-7120-4cdb-a7b0-fce92944a7d7.PNG" width="90%" height="85%"/>

