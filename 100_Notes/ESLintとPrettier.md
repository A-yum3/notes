1. Prettier入れる
	2. `npm install prettier -D`
2. ESLint入れる
	1. https://qiita.com/notakaos/items/85fd2f5c549f247585b1
		1. `prettier/@typescript-eslint`は`prettier`にmergedなのでいらない


`npm install -D prettier eslint @typescript-eslint/parser @typescript-eslint/eslint-plugin  eslint-config-prettier`

```sh
#!/bin/bash  
  
####################################################  
# ESLint + Prettierの設定ファイルを作成する  
####################################################  
  
PATH_DIR_PARENT="$(dirname "$(cd "$(dirname "${BASH_SOURCE:-$0}")" && pwd)")"  
cd ${PATH_DIR_PARENT}  
green='\e[32;1m'  
  
touch tsconfig.eslint.json .eslintrc.js .prettierignore .prettierrc.json  
  
cat <<EOS > tsconfig.eslint.json  
{  
  "extends": "./tsconfig.json",  
  "include": [  
    "src/**/*.ts",  
    ".eslintrc.js"  
  ],  
  "exclude": [  
    "node_modules",  
    "dist"  
  ]  
}  
EOS  
  
printf "${green}Generated tsconfig.eslint.json\n"  
  
cat <<EOS > .eslintrc.js  
module.exports = {  
  root: true,  
  env: {  
    es6: true,  
    node: true,  
  },  
  parser: '@typescript-eslint/parser',  
  parserOptions: {  
    sourceType: 'module',  
    ecmaVersion: 6,  
    tsconfigRootDir: __dirname,  
    project: ['./tsconfig.eslint.json']  
  },  
  plugins: [  
    '@typescript-eslint',  
  ],  
  extends: [  
    'eslint:recommended',  
    'plugin:@typescript-eslint/recommended',  
    'plugin:@typescript-eslint/recommended-requiring-type-checking',  
    'prettier'  
  ],  
  rules: {  
  },  
};  
EOS  
  
printf "${green}Generated .eslintrc.js\n"  
  
cat <<EOS > .prettierignore  
# Ignore artifacts:  
/dist  
node_modules  
package.json  
package-lock.json  
tsconfig.json  
tsconfig.eslint.json  
EOS  
  
printf "${green}Generated .prettierignore\n"  
  
cat <<EOS > .prettierrc.json  
{  
  "tabWidth": 4,  
  "useTabs": false,  
  "prettier:base": "prettier --parser typescript --single-quote",  
  "prettier:check": "npm run prettier:base -- --list-different \"src/**/*.{ts,tsx}\"",  
  "prettier:write": "npm run prettier:base -- --write \"src/**/*.{ts,tsx}\""  
}  
EOS  
  
printf "${green}Generated .prettierrc.json\n"
```

