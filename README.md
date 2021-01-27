# 2021 Kaggle master study

## How to?
-  fork 버튼을 누르셔서 여러분의 git을 만드시고, 그 git 주소로 git clone
  ```git clone ```
- 변경된 파일을 추가
  ```git add . --all```
- 변경사항을 commit
  ```git commit -m "update"```
- 변경사항을 push
  ```git push origin main```

- 여러분의 git에서 이제 공용 git으로 보내야합니다! 
  - https://github.com/Pseudo-Lab/21_kaggle_master 에 가셔서
  - Pull Requests > New Pull Request > compare accross folks > head repository를 여러분의 id/branch로 변경 > create pull request

### 첫 clone 및 push가 아니라, 두번째부터!
- 공용 git으로부터 최신 버전을 받고서 시작하셔야 합니다. 처음에는 upstream이 설정되어있지 않으니 설정이 필요합니다!
  ```git remote add upstream https://github.com/Pseudo-Lab/21_kaggle_master.git```
- git remote 확인
  ```git remote -v```
- 공용 git에서 변경된 사항 가져오기
  ```git fetch upstream https://github.com/Pseudo-Lab/21_kaggle_master.git```
