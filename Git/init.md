# Github 에 새로운 저장소 추가

## git init

Git 저장소로 만들고자 하는 디렉토리로 이동한 다음 아래 명령어를 타이핑합니다.

`.git` 과 `.gitignore` 폴더가 생긴다면 성공입니다.

`.git` 폴더는 숨겨져있습니다. (`ls -a` 명령어를 사용하면 숨겨진 파일까지 전부 볼 수 있습니다)

```bash
git init
```

## git remote

그리고 Github 로 이동해서 repository 를 생성한 다음 URL 을 복사합니다.

`-v` 옵션으로 제대로 추가되었는지 확인할 수 있습니다.

지울땐 `remove` 명령어를 사용합니다.

```bash
git remote add origin [URL]     # 추가
git remote -v                   # 버전 확인
git remote remove origin        # 삭제
```

## git add / commit / push

```bash
git add .
git commit -m "first commit"
git push -u origin master
```