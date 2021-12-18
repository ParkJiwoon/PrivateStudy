# Error [ERR_PACKAGE_PATH_NOT_EXPORTED]: Package subpath './lib/tokenize' is not defined by "exports" in ~~/package.json

# 에러 로그

```bash
$ yarn start  
yarn run v1.22.17
$ react-scripts start
node:internal/modules/cjs/loader:488
      throw e;
      ^

Error [ERR_PACKAGE_PATH_NOT_EXPORTED]: Package subpath './lib/tokenize' is not defined by "exports" in /Users/Documents/my/projects/practice-codes/react-js-hello/node_modules/postcss-safe-parser/node_modules/postcss/package.json
    at new NodeError (node:internal/errors:371:5)
    at throwExportsNotFound (node:internal/modules/esm/resolve:416:9)
    at packageExportsResolve (node:internal/modules/esm/resolve:669:3)
    at resolveExports (node:internal/modules/cjs/loader:482:36)
    at Function.Module._findPath (node:internal/modules/cjs/loader:522:31)
    at Function.Module._resolveFilename (node:internal/modules/cjs/loader:919:27)
    at Function.Module._load (node:internal/modules/cjs/loader:778:27)
    at Module.require (node:internal/modules/cjs/loader:999:19)
    at require (node:internal/modules/cjs/helpers:102:18)
    at Object.<anonymous> (/Users/Documents/my/projects/practice-codes/react-js-hello/node_modules/postcss-safe-parser/lib/safe-parser.js:1:17) {
  code: 'ERR_PACKAGE_PATH_NOT_EXPORTED'
}

Node.js v17.0.1
error Command failed with exit code 1.
info Visit https://yarnpkg.com/en/docs/cli/run for documentation about this command.
```

<br>

# 원인

Mac OS 를 BigSur 로 업그레이드 후 연습중이던 React 프로젝트를 클론받아 실행했더니 에러가 발생했습니다.

복잡한 프로젝트도 아니어서 당황하던 중 검색해보니 [node 17 버전을 사용해서 그렇다는 글](https://exerror.com/error-err_package_path_not_exported-package-subpath-lib-tokenize-is-not-defined-by-exports/)을 봤습니다.

현재 제 노드 버전을 확인해보니 `v17.0.1` 을 사용하고 있었습니다.

<br>

# 해결

`nvm` (Node Version Manager) 을 설치한 후에 노드 버전을 다운그레이드 해서 해결했습니다.

처음에 `brew install nvm` 으로 설치했는데 잘 안되어서 [nvm Github](https://github.com/nvm-sh/nvm#installing-and-updating) 을 보고 다시 설치했습니다.

```sh
# nvm 설치
$ curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
```

<br>

설치 후에 `~/.zshrc` 을 열어 맨 아래에 다음 내용을 붙여넣습니다.

기본 터미널을 bash 로 사용하고 있다면 `~/.bash_profile` 에 추가하면 됩니다.

```sh
export NVM_DIR="$([ -z "${XDG_CONFIG_HOME-}" ] && printf %s "${HOME}/.nvm" || printf %s "${XDG_CONFIG_HOME}/nvm")"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh" # This loads nvm
```

<br>

마지막으로 `~/.zshrc` 를 갱신한 뒤 LTS 버전을 설치하고 다시 실행해보면 정상적으로 동작합니다.

```sh
# nvm command 가 안뜨면 갱신
$ source ~/.zshrc

# nvm 설치 확인
$ nvm -v
0.39.0

# node lts 버전 설치
$ nvm install --lts

# 16 버전이 설치됨
$ node -v
v16.13.0
```

<br>

# Reference

- [[Solved] Error [ERR_PACKAGE_PATH_NOT_EXPORTED]: Package subpath ‘./lib/tokenize’ is not defined by “exports”](https://exerror.com/error-err_package_path_not_exported-package-subpath-lib-tokenize-is-not-defined-by-exports/)