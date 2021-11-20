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
## sed
## awk
