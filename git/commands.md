# Git commands

## 삭제 관련

### Staged 된 파일들 되돌리기

`git reset <option> <commit>`

**<option>**
  - `--hard`: staged 된 내용을 전부 삭제한다. Resets the index and working tree. Any changes to tracked files in the working tree since <commit> are discarded.
  - `--soft`: Does not touch the index file or the working tree at all (but resets the head to <commit>, just like all modes do). This leaves all your changed files "Changes to be committed", as git
               status would put it.
  - `--mixed`: Resets the index but not the working tree (i.e., the changed files are preserved but not marked for commit) and reports what has not been updated. This is the default action.
  
**<commit>**
  - `HEAD~3`: HEAD 에서 3번째 이전 커밋으로 되될리기.
  
  
### Untracked 파일 삭제
  
`git clean -rd`
  - `-d`: 디렉토리 포함
  - `-f`: 강제
