# Start git

## Agenda

- shell, vim commands
- git basic - clone, add, commit, push
- pre-commit

### Shell, Vim Command

1. shell command

shell은 운영체제의 커널과 사용자를 이어주는 소프트웨어
bash: UNIX shell로 다양한 운영체제에서 기본 쉘로 채택
zsh: bash의 단점을 보완한 최근 UNIX 쉘

```shell
(-) 는 option을 의미
$ ls 파일, 디렉토리 안에 있는 것들 전체보기
$ ls -a (.)으로 시작하는 숨긴 파일까지 전체보기
$ ls -al 더 상세하게 보기

cd 는 변경을 의미
$ cd dev
$ cd .. 전 디렉토리로 이동
$ pwd 내가 어느 디렉토리에 위치해 있는지 절대 경로를 출력

$ touch newfile.md 새로운 파일 만들기

$ mv main.py temp/ move 메인 파일을 temp/로
$ cp newfile.md newfile_copy.md 파일이나 디렉토리 복사
$ rm newfile.md remove
$ rm -rf temp recursive false 
sudo(super user do) rm -rf 모든 데이터 다 지워라
cat newfile.md concatenate 파일 내용을 출력, 이어붙이기
```

2. vim command

Free and Open source text editor 

```text
mode:
normal, insert, visual, command-line

- normal mode(default): 모든 키가 명령으로 동작
- insert mode(i): 입력, 수정 모드
- visual mode(v): 블록설정
- command-line mode(:): 패턴 검색, 필터, 줄이동, 종료 등의 액션 수행 모드

:q - quit
:q! - override and quit
:w - write
:wq - write and quit
:{num} - Go to {num}th line
```  

```
<!— 주석 —>
# heading text(h1~h6: #~######)

<!— 문단 텍스트 —>

She sells seashells by the seashore

<!— 순서가 없는 아이템 —>

- Item 1
- Item 2
- Item 3

<!— 순서가 있는 아이템 —>
1. Order 1
2. Order 2
3. Order 3

<!— 하이퍼텍스트 —>

[link text](link url)
<!— 이미지 —>

![alt text](img src)

<!— 강조 표기 —>

_italic_, **Bold**, `mono`
~~strikethrough~~

<!— code block —>

```python
print('hello')```
```
### Git Basic

```shell
Set Configuration

$ git config --global user.name “{username}”

$ git config --global user.email “{emailaddr}”

$ git config --global core.editor “vim”

$ git config --global core.pager “cat”

$ git config --list

# 수정이 필요할 경우, $ vi ~/.gitconfig 에서 값 수정
```

```shell
How to start

$ git clone {username/repo-addr}
$ cd {repo-addr}
$ vi README.md
$ git status
$ git add README.md
$ git commit
$ git push origin main push 부터 local을 벗어남
```
```text
1. commit의 제목은 commit을 설명하는 문장형이 아닌 구나 절의 형태로 작성

2. importanceofcapitalize `Importance of Capitalize`

3. prefix 꼭 달기

- feat: 기능 개발 관련

- fix: 오류 개선 혹은 버그 패치

- docs: 문서화 작업

- test: test 관련

- conf: 환경설정 관련

- build: 빌드 작업 관련

- ci: Continuous Integration 관련

- chore: 패키지 매니저, 스크립트 등

- refactor: 코드의 기능은 그대로 두면서, 내부 구조를 더 깔끔하고 효율적으로 바꾸는 것

- style: 코드 포매팅 관련
```

```shell
.gitignore
특정 파일이나 디렉토리를 추적하지 않도록 명시하기 위한 파일
[gitignore](https://gitignore.io/)
```

```shell
Pre-commit
- commit 수행 전 체크해야 할 것들을 자동 수행하도록 도와주는 도구

[pre_commit](https://pre-commit.com/)

$ pip install pre-commit (or $ pip3 install pre-commit)
$ pre-commit --version
$ touch .pre-commit-config.yaml

# after setting configuration
$ pre-commit install
$ pre-commit run --all-files
```


