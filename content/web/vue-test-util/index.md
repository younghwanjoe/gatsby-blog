---
emoji: 🔮
title: vue-test-util를 사용한 테스트 환경 구축 과정 정리
date: '2022-03-09 00:00:00'
author: YoungHwanJoe
tags: testing
categories: web
---

## 1. 설치 및 설정

- `vue add unit-jest` 를 사용해서 의존 모듈 설치
  ```tsx
  vue add unit-jest
  ```
  **@vue/cli-plugin-unit-jest** 설치하면
  jest파일에서 typescript설정이나 babel설정(vue파일을 바벨로 트랜스파일하는거)를 간단히 해결할수있다
  ```tsx
  module.exports = {
  	preset: '@vue/cli-plugin-unit-jest/presets/typescript-and-babel',
  	...
  }
  //저거 자체적으로ts-jest를 사용하게끔한다
  ```
  viewditor/package.json
  ```tsx
  {
    ...
    "scripts": {
  		...
      "test:unit": "vue-cli-service test:unit",
      "test": "jest --watchAll",
    },
    "dependencies": {
      ...
    },
    "devDependencies": {
  	   ...
  		"jest": "^27.4.3",
      "@types/jest": "^24.0.19",
      "@vue/cli-plugin-unit-jest": "~4.5.0",
      "@vue/test-utils": "^1.0.3",
      "ts-jest": "^27.0.7",
    },
    "eslintConfig": {
  	   ...
  		"overrides": [
  		    {
  		      files: [
  		        '**/__tests__/*.{j,t}s?(x)',
  		        '**/__tests__/**/*.spec.{j,t}s?(x)',
  		        '**/__tests__/*.spec.{j,t}s?(x)',
  		      ],
  		      env: {
  		        jest: true,
  		      },
  		    },
  		  ],
  		}
    },
    ...
  }
  ```
- lerna프로젝트 `root`에 `jest`설치

  ```jsx
  // root 공통
  yarn add -D --ignore-workspace-root-check jest ts-jest @types/jest babel-jest @babel/core babel-core@^7.0.0-bridge.0

  // viewditor에만 이걸 따로 깐다
  npx lerna add -D @vue/test-utils packages/viewditor
  npx lerna add -D vue-jest packages/viewditor

  ```

- `vue add unit-jest` 없이 `lerna add` 를 사용해서 의존 모듈 설치
  일단 의존 모듈들을 전부 설치해준다(add는 한번에 하나만 설치가능한게 여러모로 불편하다)

  ```tsx
  npx lerna add -D jest packages/viewditor
  npx lerna add -D @types/jest packages/viewditor
  npx lerna add -D babel-jest packages/viewditor
  npx lerna add -D @babel/core packages/viewditor
  npx lerna add -D babel-core@^7.0.0-bridge.0 packages/viewditor
  npx lerna add -D ts-jest packages/viewditor
  npx lerna add -D @vue/test-utils packages/viewditor
  npx lerna add -D vue-jest packages/viewditor

  // 위 바벨코어 모듈들은 babel-jest돌아가려면 필요함

  // 이건 없어도 될거같다
  // npx lerna add -D @babel/preset-env packages/viewditor

  ```

- tsconfig.js 설정 변경
  “types”는 “@types”내에서 사용할것을 추리는 옵션
  ```tsx
  {
    ...
    "compilerOptions": {
      ...
      "types": [
  	     ...
        "jest"
      ]
    },
    "include": [ ...,"tests/**/*.ts"],
    "exclude": ["node_modules"]
  }
  ```
- jest 관련 설정파일 세팅
  기본적으로 jest는 node환경에서 실행되므로 `node_modules`에 있는건 따로 트랜스 파일안해붜도 된다. 근데 예외적으로 세가지 케이스가 `mode_modules`에 있는걸 트랜스파일해줘야한다.

  1. module.exports(CommonJS)문법으로 컴파일해야하는 ES6의 import/export 구문
  2. vue-jest를 통해 실행되는 .vue 익스텐션을 쓰는 Single File Components
  3. 타입스크립트

  ```tsx
  transformIgnorePatterns: ['/node_modules/'],
  // lerna를 쓰므로 root의 node_modules도 제외
  //transformIgnorePatterns: ['/node_modules/', '../../node_modules'],
  ```

  이 때문에 아래 구문을 `jest.config.js` 에추가해야한다.

  ```tsx
  module.exports = {
    // preset: '@vue/cli-plugin-unit-jest/presets/typescript-and-babel',
    moduleFileExtensions: [
      'js',
      'ts',
      'json',
      // tell Jest to handle `*.vue` files
      'vue',
    ],
    verbose: true,
    testEnvironment: 'jsdom',
    testMatch: ['<rootDir>/src/__tests__/*.(spec|test).{js,jsx,ts,tsx}'],
    setupFilesAfterEnv: ['<rootDir>/src/__tests__/jest-setup.js'],
    transform: {
      // process `*.vue` files with `vue-jest`
      '.*\\.(vue)$': 'vue-jest',
      // process `*.js` files with `babel-jest`
      '^.+\\.(js|jsx)?$': 'babel-jest',
      '^.+\\.(ts|tsx)?$': 'ts-jest',
    },
    transformIgnorePatterns: ['/node_modules/', '<rootDir>/node_modules/'],
    moduleNameMapper: {
      '^@/(.*)': '<rootDir>/src/$1',
    },
    moduleDirectories: ['node_modules', '.'],
    testPathIgnorePatterns: ['/node_modules/'],
  };
  ```

  만약 tsconfig의 alias설정을 가져다 쓰려면 다음과같이 ts-jest에서 제공하는 `pathsToModulenameMapper` 를 jest의 `moduleNameMapper`옵션에 사용하면된다.

  ```tsx
  const { pathsToModuleNameMapper } = require('ts-jest/utils');
  const { compilerOptions } = require('./tsconfig-base.json');

  module.exports = {
  	...
    moduleNameMapper: pathsToModuleNameMapper(compilerOptions.paths),
  	...
  };
  ```

  jest-setup.ts 세팅

  ```tsx
  // jest-setup.js
  import Vue from 'vue';
  import Vuetify from 'vuetify';
  Vue.use(Vuetify);
  import VueRouter from 'vue-router';
  Vue.use(VueRouter);

  import { config } from '@vue/test-utils';
  config.showDeprecationWarnings = false;
  ```

- 기본 test코드 작성

  ```tsx
  import { shallowMount } from '@vue/test-utils';
  import App from '../App.vue';

  describe('App.vue', () => {
    it('renders App', () => {
      const wrapper = shallowMount(App);
      expect(wrapper.name()).toBe('App');
    });
  });
  ```

## 2. 이슈

- `[vue-test-utils]: window is undefined, vue-test-utils needs to be run in a browser environment.`
  jest.config.js에서 testEnvironment: 'jsdom' 옵션을 추가해준다
  ```tsx
  module.exports = {
    testEnvironment: 'jsdom',
  	...
  }
  ```
-
- Jest에서`SyntaxError: Cannot use import statement outside a module`
  기본적으로 jest는 node환경에서 실행되므로 `node_modules`에 있는건 따로 트랜스 파일안해붜도 된다. 근데 예외적으로 세가지 케이스가 `mode_modules`에 있는걸 트랜스파일해줘야한다.

  1. `module.exports(CommonJS)`문법으로 컴파일해야하는 ES6의 `import/export` 구문
  2. vue-jest를 통해 실행되는 `.vue` 익스텐션을 쓰는 Single File Components
  3. 타입스크립트
     이것때문에

  ```tsx
  transformIgnorePatterns: ['/node_modules/'],
  // lerna를 쓰므로 root의 node_modules도 제외
  //transformIgnorePatterns: ['/node_modules/', '../../node_modules'],
  ```

  이걸 jest설정에추가해야한다.
  위에서 말한대로

  > Jest runs the code in your project as JavaScript, but if you use some syntax not supported by Node.js out of the box (such as JSX, types from TypeScript, Vue templates etc.) then you'll need to transform that code into plain JavaScript, similar to what you would do when building for browsers.

  Jest는 노드기반이라서 `import`를 이해하지 못한다. 그래서 트랜스파일을 해줘야한다.
  자바스크립트 파일은 'babel-jest'로 타입스크립트 파일은 'ts-jest'로 변환해주면 된다
  jest.config.js에서 옵션을 추가해주자

  ```tsx
  transform: {
  	'^.+\\.(js|jsx)?$': 'babel-jest',
    '^.+\\.(ts|tsx)?$': 'ts-jest',
  }
  ```

- Jest에서 view router 콘솔에러
  이것도 Vuetify케이스랑 같다.

  ```tsx
  // jest-setup.js
  import Vue from 'vue';
  import Vuetify from 'vuetify';
  Vue.use(Vuetify);
  import VueRouter from 'vue-router';

  Vue.use(VueRouter);
  ```

- Jest에서 Vuetify 콘솔에러
  [Vuetify에서 Jest 초기 셋업](https://lovemewithoutall.github.io/it/jest-with-vuetify/)

  ```
  console.error node_modules/vue/dist/vue.runtime.common.js:603
      [Vue warn]: Unknown custom element: <v-container> - did you register the component correctly? For recursive components, make sure to provide the "name" option.
  ```

  이건 Vuetify를 전역 로드하면 해결된다

  ```tsx
  import Vue from 'vue';
  import Vuetify from 'vuetify';

  Vue.use(Vuetify);
  ```

  그러나 create 로컬 뷰를 쓰기때문에..

- vscode 테스트 파일에 describe나 jest 템플릿이 잡히지 않는이슈. eslint랑 tsconfig 설정해줘야한다
  ```tsx
  tsconfig compieroptions에 "@types/jest" 를 추가한다
  ```
- 테스트 성공해도 vue-test-uils의 콘솔에러가 자꾸뜸. deprecated관련해서는 끈다
  **`showDeprecationWarnings`**
  위옵션을 쓴다
  jest-setup.js에 아래와 같이 코드를 추가해주면된다
  ```tsx
  import { config } from '@vue/test-utils';
  config.showDeprecationWarnings = false;
  ```
- Jest의 jest-setup 파일을 lerna 프로젝트 root에서 사용하면 vue를 참조하지 못함
  따라서 각패키지마다 jest 설정파일을 확장해서 사용해야하는거같다.
  이떄 rootDir도 바꿔줘야한다
  ```jsx
  const sharedConfig = require('../../jest.config.js');
  module.exports = {
    ...sharedConfig,
    rootDir: './',
    setupFilesAfterEnv: ['<rootDir>/jest-setup.ts'],
  };
  ```
- flow의 tsconfig의 target이 `esNext` Node-Target-Mapping 문제 발생
  현재 빌드는 node 14버전으로 되고있다
  로컬은 노드 12버전이었는데 노드버전에 상응하는 타입스크립트 target이 맞아야한다.
  그래서 빌드버전인 14로 올리고나서
  [Node-Target-Mapping](https://github.com/microsoft/TypeScript/wiki/Node-Target-Mapping)
  여기서 확인하여 노드14와 맞는 `target`인 ES2020
- tsconfig에서 정의한 ‘@/’ alias가 jest파일내에서 인식이안됨
  vue파일은 뒤에 .vue 익스텐션 붙여줘야함
- ts-jest의 pathsToModuleNameMapper가만드는 moduleNameMapper경로가 에러를일으킴

  ```tsx
  Configuration error:

      Could not locate module @/components/account/Login.vue mapped as:
      ./src/$1.

      Please check your configuration for these entries:
      {
        "moduleNameMapper": {
          "/^@\/(.*)$/": "./src/$1"
        },
        "resolver": undefined
      }
  ```

  pathsToModuleNameMapper를사용하지 않고 Jest의 설정을 아래와같이 바꿔서 픽스

  ```tsx
  moduleNameMapper: {
      '^@/(.*)': '<rootDir>/src/$1',
    },
  ```

## 3. 도입 소감

- 한 함수에 대해 인자에 여러 케이스를 줘서 테스트를 하면 에러처리에 대해 더 구체적이고 정확하게 가능함
- 예를 들어 로컬스토리지의 AES엔크립트된 토큰을 `decrypt` -> jwt를 다시 `decode`하는거→ 그걸로 세션확인하는 복잡한 작업을 부분부분마다 체크할수있어서 안심이된다
- 확실히 함수를 기능단위로 쪼개게 된다 그래야 테스트가 편하니까, 작게 쪼개야 작은 단위와 로직에 대한 테스트가 쉬워지고 결과적으로 모듈화가 된다

## 4. 더 해볼것

- 제대로된 CI/CD를 구축하기 위해 테스트 코드를 `git action`에 연동후 테스트 통과되어야 머지할수있도록 설정

```toc

```
