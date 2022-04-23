### typescriptのinstall

`npm install --save-dev typescript @types/node`

### tsconfig.jsonの準備

`npx tsc --init`

- targetをes2020に
- moduleをesnextに
	- node.jsのバージョンでes modulesが使えるならこっち
- moduleResoulution
	- npmでインストールしたモジュールをTypeScriptが認識できるようにする
- outDirを ./distに
	- dist以下にJSファイルが生成される
- includeをcompilerOptionsの外に必要がある
	- `./src/**/*.ts`
	- src以下の全てのtsがコンパイル対象になる