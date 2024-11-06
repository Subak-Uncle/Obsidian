
1. .git 디렉터리 삭제 / 상태확인
   $ rm -rf .git
   $ git status (fatal: not a git repository)

1. git 초기화 후 새로운 git 설정
   $ cd <생성할 디렉터리>
   $ git init
   $ git add .
   $ git commit -m "commit message"

1. github 저장소 연결후 강제 push
   $ git remote add origin <연결할 url>
   $ git push --force --set-upstream origin master