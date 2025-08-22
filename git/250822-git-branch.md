# Git Branch

## Agenda
- Git Branch
- Trouble-shoot
- Make new project

## Branch
분기점을 생성하여 독립적으로 코드를 변경할 수 있도록 도와주는 모델

```shell
# on main branch
print(‘hello’)

# on develop branch
print(''.join(['h', 'e', 'l', 'l', 'o',]))
```

```shell
# Branch list
(local) $ git branch
(remote) $ git branch -r
(all) $ git branch -a

# Create new branch
$ git branch {branch name}

# switch to branch
$ git switch {branch name}

# Delete branch
$ git branch -D stem

# Push commits on specific branch
# 첫 push에는 remote와 local의 branch 연결(-u)이 필요 (이후 수행 필요 x)

(first) $ git push -u origin stem
$ git push -u origin stem

# See the differences
$ git diff main stem
```

## Merge Conflict
Merge 중 발생하는 변경사항 끼리의 충돌
git은 auto-merge로 대부분을 해결

```shell
1. Merge 상황에서 해결
# 충돌 일으키기
$ git merge main

# 충돌 발생한 파일 열어서 문제해결
$ vi {filename}

# 수정 시 (=====, <<<<<, >>>> 는 꼭 삭제)

# 수정 완료한 변경사항 add, commit
$ git add {filename}
$ git commit
```
```shell
2. rebase의 방법으로 해결(git flow에서는 사용하지 않는 방식)
다른 사람의 코딩을 가져올 때 보통 rebase로 해결

$ git rebase main

# conflict 발생 시 conflict 해결
$ vi {filename}

# conflict 해결 후 add, continue

# 해당 변경사항 commit 넘기려면 skip

# rebase를 취소하려면 abort
$ git add {filename}
$ git rebase --continue
```

```shell
Conflict 해결 방법
# 1. Merge 시 해결(이미 merge 명령을 내린 경우)
$ git branch -D stem

# 2. 미리 변경사항을 적용

# 첫 push에는 remote와 local의 branch 연결(-u)이 필요
(first) $ git push -u origin stem
$ git push -u origin stem

# See the differences
$ git diff main stem
```
## Trouble-shoot

```shell
1. stash
- 작업 중 브랜치 이동이 필요할 때, 작업사항을 잠시 미뤄둘 때 사용
- 작업사항을 임시 저장소에 쌓기

$ git stash or $ git stash save “{message}”
- 쌓아둔 작업사항 복구하기(index: stash number)

$ git stash pop or $ git stash pop {index}
or $ git stash apply
- 쌓아둔 작업사항 리스트 확인하기

$ git stash list
- {index} 번째 작업사항 삭제하기

$ git stash drop {index}

2. rename
$ mv a.md to b.md

3. undo
$ git restore {filename} or .(whole changes)

4. upstaging
$ git reset HEAD {filename}

5. upstaging과 remove 동시에 삭제
$ git rm -f {filename}

6. edit commit message 직전 commit 메세지 삭제
$ git commit --amend

7. 이전 commit 메세지 삭제
$ git rebase -i <commit>

$ git rebase --continue (rebase 취소: git rebase --abort)

8. reset commit 없애버리기 (절대 하지 말 것)
$ git reset --hard HEAD~{nums of commit}

$ git push -f origin <branch>

- {nums of commit}개의 커밋을 삭제 후 remote <branch>에 강제 push
- 협업 시에는 나의 local과 clone한 remote repo에서 지워졌다고 해도 다른 곳에 남아있던 이력으로 인해 살아나거나, 충돌이 발생함

9. revert commit 잘못 알린 후 특점 시점으로 돌리기
$ git revert --no-commit HEAD~{nums of commit}..
$ git commit
$ git push origin <branch>
- {nums of commit}개의 커밋을 되돌린 후 remote <branch>에 push
- 잘못하기 직전 시점으로 되돌리고 해당 되돌림의 이력을 팀원들에게 전달하여 어떤 문제가 있었고, 어떻게 수행되는지에 대해 뚜렷한 설명가능
- commit을 따로 하지 않을 땐 ‘—no-edit’
- merge commit을 되돌릴 땐 ‘-m’
(ex) $ git revert -m {1 or 2} {merge commit id})

```

## Team project
```text
new repo 생성하기
code html 복사해서 clone하기
팀원은 isuue에서 작업사항 정리하고 체킹
fork하고 clone하기
각자의 컴퓨터에 fork한 나의 repo를 clone하기
만약, 다른 사람이 업로드 했을 시 upstream 설정
내 repo에 push하기
open pull request에서 작업 사항 전달하기 위한 PR 열기
code review 서로 해주기
branch에서 수정할 사항 수정하기
```
