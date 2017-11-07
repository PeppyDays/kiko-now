---
layout: post
title: nc, pv 명령어를 통해 파일 전송하기
tags: [linux, nc, pv, file, copy]
---

DB 운영을 하다보면 OS 에서 이런 저런 작업들을 할 일이 많다. 그 중 한가지로, 백업이나 복구를 위해 데이터 파일을 다른 서버로 옮긴다던지 하는 일이 있다.

단순하게는 scp 를 통해 옮기면 되겠지만, 여기에는 아래와 같은 몇 가지 한계가 있다.

- 서버들끼리 ssh 가 되어야 한다.
- 서버의 계정/비밀번호 를 알아야 한다.
- 네트워크 트래픽량을 조절할 수 없다.
- 여러 파일을 보내거나, 또는 압축해서 보내는 경우 여러번 작업해야 해서 매우 번거롭다.

서버들끼리 ssh 나 인증정보를 왜 모르냐고 할 수도 있겠지만, 이런 것들을 직접 관리하지 않고 켈베로스 등을 통해 인증해서 들어가거나 하는 경우도 있다.

그리고 정말 큰 데이터 파일들을 여러 서버들 사이에서 전송시키면 그 망에 있는 스위치 과부하로 다른 서버들에 영향을 줄 수도 있다.

그래서 이런 문제들을 해결하기 위해 nc, pv 명령어를 알아보자.

## nc

nc 는 Netcat 의 줄임말로, TCP/UDP 커넥션을 만들거나 받으면서 소켓통신을 할 수 있는 간단한 유틸리티 프로그램이다.

아래 내용들은 아래 2개 사이트에서 내용을 참고해서 정리했다.

- [Tutorials Point](http://www.tutorialspoint.com/unix_commands/nc.htm)
- [The Geek Stuff](http://www.thegeekstuff.com/2012/04/nc-command-examples/?utm_source=feedburner)

테스트한 내용들은 로컬에서 Session #1 (Server), Session #2 (Client) 2개의 세션 사이에서 통신한다.

### Server-Client Architecture

한 쪽에서는 특정 포트로 소켓통신 서버 역할을 하고, 다른 쪽에서는 서버에 접속하는 클라이언트 역할을 한다. 우선 서버 소켓을 열어보자.

```
# Session #1 (Server), 명령어를 실행하면 소켓으로 들어올 데이터를 기다린다.
# 1234 포트로 소켓을 연다.
$ nc -l 1234
```

이제 다른 터미널을 하나 더 열어서 위에서 만든 서버에 접속한다.

```
# Session #2 (Client), 1234 포트로 접속하고, 사용자로부터의 데이터 입력을 기다린다.
$ nc localhost 1234
```

여기서 Session #2 (Client) 에서 이런 저런 텍스트들을 입력해보자.

```
# Session #2 (Client)
$ nc localhost 1234
hello,
stranger?
```

같은 내용이 Session #1 (Server) 에서 나타난다는 것을 확인할 수 있다.

```
# Session #1 (Server)
$ nc -l 1234
hello,
stranger?
```

그리고 Session #2 (Client) 에서 Ctrl+C 를 눌러서 명령을 종료시키면 Session #1 (Server) 의 명령도 자동으로 같이 종료된다. 즉, 1번 통신을 시작해서 완료되면 서버-클라이언트 모두 종료된다.

위와 같은 간단한 명령어로 서버간 통신을 할 수 있고, 기본적으로는 서버에서 표준출력으로 데이터를 넘기는 것을 알 수 있다.

### Data (or File) Transfer

이제 nc 를 이용해서 파일을 전송해보자. Session #2 (Client) 에서 아래처럼 텍스트 파일을 하나 만들고, 이걸 전송하자.

```
# Session #2 (Client)
$ echo "hi, there?" > nc_test
$ cat nc_test
hi, there?
```

파일이 준비되었으므로, Session #1 (Server) 에서 서버를 열어보자.

```
# Session #1 (Server)
$ nc -l 2381
```

이제 Session #2 (Client) 에서 위에 열린 서버에 접속해서, 파일의 내용의 내용을 cat 으로 표준출력으로 만들고, nc 명령어의 표준입력으로 보내주도록 한다.

```
# Session #2 (Client)
$ cat nc_test | nc localhost 2381
```

명령은 바로 종료되고, Session #1 (Server) 을 확인해보면 테스트 파일의 내용이 표준출력으로 나오고 종료되었다.

```
# Session #1 (Server)
$ nc -l 2381
hi, there?
```

그런데 우리는 이 내용이 파일로 저장되기를 바라므로, 표준출력을 파일로 리다이렉션 시켜주도록 Session #1 (Server) 의 명령을 아래처럼 수정하고 다시 테스트해보자.

```
# Session #1 (Server)
$ nc -l 2381 > nc_test
```

위의 명령을 통해 표준출력을 파일로 저장하고, 클라이언트에서 보낸 뒤 읽어보면 잘 나오는 것을 확인할 수 있다.

위의 방법을 사용하면 서버간 ssh 통신이 안되도, 계정정보를 몰라도, 포트간 소켓통신을 통해 데이터를 전송시킬 수 있다.

### Other Options

nc 명령에는 그 외에도 여러가지 쓸만한 옵션들이 있다.

예를 들어 포트 스캐닝은 아래 명령을 통해 확인할 수 있다. -z 옵션은 포트번호구간 지정 시 하나씩 스캔하는 옵션이고, -w 옵션은 접속 시의 timeout(s) 시간이다.

```
$ nc -z -w 1 localhost 27000-27100
Connection to localhost port 27001 [tcp/*] succeeded!
Connection to localhost port 27002 [tcp/*] succeeded!
Connection to localhost port 27003 [tcp/*] succeeded!
Connection to localhost port 27017 [tcp/*] succeeded!
```

위의 파일 전송에서 테스트 해보면 서버를 열어도 1번 통신이 완료되면 명령이 종료되는데, -k 옵션을 통해 이걸 종료시키지 않을 수도 있다.

```
$ nc -k -l 1234
```

다른 옵션들의 내용은 [Tutorials Point](http://www.tutorialspoint.com/unix_commands/nc.htm) 에서 확인할 수 있다.

## pv

pv 는 프로세스 사이에 파이프라인을 통해 데이터 교환 시 그 진행상태 정도를 알려주는 유틸리티 명령어이다. 그리고 pv 를 통해 데이터 교환 속도도 조정을 할 수 있다!

기본적인 개념은 파이프라인을 통해 프로세스간 데이터를 전송할 때, 그 중간에 아래처럼 파이프라인으로 pv 명령을 추가해서 진행상태를 확인하는 것이다.

또는 pv {파일명} 을 통해 파일을 직접 표준출력으로 만들어서 파이프라인으로 다음 프로세스로 진행할 수도 있다.

```
$ cat aaa.txt | pv | scp root@client:/root
```

상세한 내용은 [pv Manual Page](https://linux.die.net/man/1/pv) 를 참고하자.

이제 nc 명령어와 조합해서 서버간 파일 전송 시 pv 를 통해 데이터를 전송해보자. Session #1 (Server) 에서 nc 로 소켓을 하나 열고, 받는 데이터를 test 라는 파일로 저장하도록 하자.

```
# Session #1 (Server)
$ nc -l 9999 > test
```

이제 Session #2 (Client) 에서 위에 만든 서버 소켓으로 파일을 전송할 때, 가운데 pv 명령어를 넣어서 보내자.

```
# Session #2 (Client)
$ pv install_flash_player_osx.dmg | nc localhost 9999
18.2MiB 0:00:00 [ 164MiB/s] [========================================================>] 100%
```

로컬에서 데이터를 보내는 거라서 너무 순식간에 전송되지만, 결과를 보면 전송하는 중간중간 1초마다 보낸 파일의 크기, 걸린 시간, 속도, 전체 중 몇 % 나 진행되었는지, 얼마나 후에 종료될지 표시되는 것을 확인할 수 있다.

다른 적용할 옵션들도 많지만, 표시하는 부분에서는 따로 옵션을 주지 않아도 충분한 정보들이 보인다. 이제 -L 옵션을 써서 전송하는 속도를 제어해보자.

```
# Session #1 (Server)
$ nc -l 9999 > test
```

```
# Session #2 (Client)
$ pv -L 2m install_flash_player_osx.dmg | nc localhost 9999
18.2MiB 0:00:09 [1.99MiB/s] [========================================================>] 100%
```

속도를 2 MiB/s 로 조정했더니 총 전송시간이 9초가 걸리고, 속도는 2 MiB/s 근처인 것을 알 수 있다.

이제 pv 를 사용해서 파일 복사의 진행 정도와 전송 속도를 제어할 수 있게 되었다.

## 파일 압축 후 전송

전송할 파일이 여러개라면 파일 개수만큼 nc 명령어를 통해 전송하고 저장해야 한다. 그래서 tar 로 한번에 묶어서 전송한다면 더욱 편할 것이다.

그리고 만약 압축하는데 걸리는 시간보다 네트워크 전송 속도로 인해 지연되는 시간이 더욱 길다면 압축해서 전송하고 압축 해제하는 것이 더욱 효율적일 것이다.

> 압축해서 전송하고 압축을 풀려고 하는 방법이 항상 효율적이지 않다. 압축하고 해제 시에 걸리는 시간과, 압축해서 줄어드는 용량만큼의 네트워크 전송 시 걸리는 시간을 비교해야 한다. 그리고 압축하고 해제 시에 발생하는 CPU 와 메모리의 사용, 디스크 I/O 가 서비스에 영향이 될 수 있다는 것도 생각해야 한다.

이러한 경우도 tar 명령을 이용해서 압축의 결과를 표준출력으로 보내 nc, pv 를 사용해서 전송하도록 한다면 작업할 때 매우 편할 것 같다.

우선 tar 명령을 통해 단순히 데이터를 묶어서 전송하는 경우를 생각해보자.

클라이언트에서는 tar 명령으로 여러 파일을 묶어서 표준출력으로 보내고, pv 로 표준출력을 받아서 진행상황과 전송속도를 제어하고, nc 명령으로 pv 의 표준출력을 받아서 서버로 전송한다.

서버에서는 전송받은 내용을 tar 명령으로 원래 파일로 해제시켜서 저장하면 된다. 서버는 아래의 명령어로 시작시킨다.

```
# Session #1 (Server)
$ nc -l 9999 | tar -xvf -
```

클라이언트는 아래와 같은 명령으로 데이터를 서버로 보낸다.

```
# Session #2 (Client)
$ ls -al
total 24
drwxr-xr-x   5 kakao  staff  170 Nov  7 14:28 .
drwx------+ 19 kakao  staff  646 Nov  7 12:01 ..
-rw-r--r--   1 kakao  staff    2 Nov  7 14:28 a.txt
-rw-r--r--   1 kakao  staff    2 Nov  7 14:28 b.txt
-rw-r--r--   1 kakao  staff    2 Nov  7 14:28 c.txt

$ tar -cvf - * | pv -L 1m | nc localhost 9999
a a.txt
a b.txt
a c.txt
10.0KiB 0:00:00 [5.38MiB/s] [ <=>                             ]
```

이제 서버 쪽에서 전송되어서 압축이 해제되었는지 확인해보자.

```
# Session #1 (Server)
$ ls -al
total 37376
drwxr-xr-x  6 kakao  staff       204 Nov  7 15:11 .
drwx------+ 9 kakao  staff       306 Nov  7 12:02 ..
-rw-r--r--  1 kakao  staff         2 Nov  7 14:28 a.txt
-rw-r--r--  1 kakao  staff         2 Nov  7 14:28 b.txt
-rw-r--r--  1 kakao  staff         2 Nov  7 14:28 c.txt
```

생각한대로 잘 동작하는 것을 확인할 수 있다. 또한 위의 테스트에서 tar 의 옵션으로 -z 를 추가시켜서 gzip 압축을 해도 동일하게 동작할 것이라고 예측할 수 있다.

압축에는 다양한 알고리즘이 존재하고, linux 에서도 다양한 명령어들이 존재한다. 보통 압축률과 압축속도 간 트레이드 오프가 있고, [네이버 기술 블로그](https://m.blog.naver.com/PostView.nhn?blogId=naverdev&logNo=120114840646&proxyReferer=https%3A%2F%2Fwww.google.co.kr%2F) 에 매우 잘 정리가 되어있으니 참고하자.

일반적으로 네트워크로 압축해서 전송하려고 하면 보통 압축속도가 높고 압축률은 낮은 것을 사용해서 속도를 높이려고 한다. 그래서 lzop 이라는 유틸리티를 사용해서 위의 작업을 하고자 한다면 아래처럼 할 수 있다.

> tar 명령의 --use-compress-program 옵션을 사용하면 위와 동일한 명령어를 통해 작업을 진행할 수 있는데, 왜인지는 모르겠지만 Mac 에서 압축 해제 시에 --use-compress-program 옵션을 사용하면 계속 에러가 난다. 그래서 lzop 의 -c 옵션을 사용해서 표준출력으로 내보내는 방법을 사용했다.

위와 동일하게 서버와 클라이언트에서 해야할 일들을 정리해보자.

클라이언트는 파일을 tar 로 묶어서 표준출력으로 lzop 에게 전달해주고, lzop 은 표준출력의 데이터를 압축해서 pv 로 전달하고, pv 에서는 nc 를 통해서 서버로 파일을 전송한다.

서버는 nc 가 파일을 받고, lzop 이 압축을 해제해서 표준출력으로 tar 로 전송하고, tar 는 묶음을 해제하면 된다. 이제 서버를 올려보자.

```
# Session #1 (Server)
$ nc -l 9999 | lzop -d | tar -xf -
```

클라이언트에서 아래 명령을 실행하자.

```
# Session #2 (Client)
$ tar -cf - *.txt | lzop -c | pv | nc localhost 9999
 273 B 0:00:00 [ 494KiB/s] [ <=>                             ]
```

서버와 클라이언트에서 데이터가 잘 왔는지 확인해보자.

## 마치며

결론은 서버와 클라이언트에서 아래 명령을 수행해서 파일을 옮기면 된다.

```
# Server (압축 사용 여부에 따라 1가지 선택)
$ nc -l 9999 | tar -xf -
$ nc -l 9999 | tar -xzf -
$ nc -l 9999 | lzop -d | tar -xf -
```

```
# Client (압축 사용 여부에 따라 1가지 선택)
$ tar -cf - * | pv -L 50m | nc localhost 9999
$ tar -czf - * | pv -L 50m | nc localhost 9999
$ tar -cf - * | lzop -c | pv -L 50m | nc localhost 9999
```
