# nuxt-ts-issue-multiregister

> Simple project for reproducing issue of @nuxt/typescript about registering ts-node more than once.

## Issue

A typescript compile error occurs when nuxt.config.ts has changed in development mode.

```
 ERROR  ⨯ Unable to compile TypeScript:                                                                                  12:43:28
nuxt.config.ts:2:5 - error TS7022: '__importDefault' implicitly has type 'any' because it does not have a type annotation and is referenced directly or indirectly in its own initializer.

2 var __importDefault = (this && this.__importDefault) || function (mod) {
      ~~~~~~~~~~~~~~~
nuxt.config.ts:2:67 - error TS7006: Parameter 'mod' implicitly has an 'any' type.

2 var __importDefault = (this && this.__importDefault) || function (mod) {
                                                                    ~~~
nuxt.config.ts:5:5 - error TS7022: '__importStar' implicitly has type 'any' because it does not have a type annotation and is referenced directly or indirectly in its own initializer.

5 var __importStar = (this && this.__importStar) || function (mod) {
      ~~~~~~~~~~~~
nuxt.config.ts:5:61 - error TS7006: Parameter 'mod' implicitly has an 'any' type.

5 var __importStar = (this && this.__importStar) || function (mod) {
                                                              ~~~
nuxt.config.ts:8:81 - error TS7017: Element implicitly has an 'any' type because type '{}' has no index signature.

8     if (mod != null) for (var k in mod) if (Object.hasOwnProperty.call(mod, k)) result[k] = mod[k];
                                                                                  ~~~~~~~~~
nuxt.config.ts:9:5 - error TS7017: Element implicitly has an 'any' type because type '{}' has no index signature.

9     result["default"] = mod;
      ~~~~~~~~~~~~~~~~~
nuxt.config.ts:90:16 - error TS7006: Parameter 'config' implicitly has an 'any' type.

90         extend(config, ctx) {
                  ~~~~~~
nuxt.config.ts:90:24 - error TS7006: Parameter 'ctx' implicitly has an 'any' type.

90         extend(config, ctx) {
                          ~~~
nuxt.config.ts:101:70 - error TS7006: Parameter 'rule' implicitly has an 'any' type.

101             config.module.rules.splice(config.module.rules.findIndex(rule => rule.loader === 'vue-loader') + 1, 0, {
                                                                         ~~~~


  nuxt.config.ts:2:5 - error TS7022: '__importDefault' implicitly has type 'any' because it does not have a type annotation and is referenced directly or indirectly in its own initializer.

  2 var __importDefault = (this && this.__importDefault) || function (mod) {
        ~~~~~~~~~~~~~~~
  nuxt.config.ts:2:67 - error TS7006: Parameter 'mod' implicitly has an 'any' type.

  2 var __importDefault = (this && this.__importDefault) || function (mod) {
                                                                      ~~~
  nuxt.config.ts:5:5 - error TS7022: '__importStar' implicitly has type 'any' because it does not have a type annotation and is referenced directly or indirectly in its own initializer.

  5 var __importStar = (this && this.__importStar) || function (mod) {
        ~~~~~~~~~~~~
  nuxt.config.ts:5:61 - error TS7006: Parameter 'mod' implicitly has an 'any' type.

  5 var __importStar = (this && this.__importStar) || function (mod) {
                                                                ~~~
  nuxt.config.ts:8:81 - error TS7017: Element implicitly has an 'any' type because type '{}' has no index signature.

  8     if (mod != null) for (var k in mod) if (Object.hasOwnProperty.call(mod, k)) result[k] = mod[k];
                                                                                    ~~~~~~~~~
  nuxt.config.ts:9:5 - error TS7017: Element implicitly has an 'any' type because type '{}' has no index signature.

  9     result["default"] = mod;
        ~~~~~~~~~~~~~~~~~
  nuxt.config.ts:90:16 - error TS7006: Parameter 'config' implicitly has an 'any' type.

  90         extend(config, ctx) {
                    ~~~~~~
  nuxt.config.ts:90:24 - error TS7006: Parameter 'ctx' implicitly has an 'any' type.

  90         extend(config, ctx) {
                            ~~~
  nuxt.config.ts:101:70 - error TS7006: Parameter 'rule' implicitly has an 'any' type.

  101             config.module.rules.splice(config.module.rules.findIndex(rule => rule.loader === 'vue-loader') + 1, 0, {
                                                                           ~~~~

  at createTSError (node_modules\ts-node\src\index.ts:240:12)
  at reportTSError (node_modules\ts-node\src\index.ts:244:19)
  at getOutput (node_modules\ts-node\src\index.ts:360:34)
  at Object.compile (node_modules\ts-node\src\index.ts:393:11)
  at Module.m._compile (node_modules\ts-node\src\index.ts:439:43)
  at Module.m._compile (node_modules\ts-node\src\index.ts:439:23)
  at Module._extensions..js (internal/modules/cjs/loader.js:810:10)
  at require.extensions.(anonymous function) (node_modules\ts-node\src\index.ts:442:12)
  at Object.require.extensions.(anonymous function) [as .ts] (node_modules\ts-node\src\index.ts:442:12)
  at Module.load (internal/modules/cjs/loader.js:666:32)
  at tryModuleLoad (internal/modules/cjs/loader.js:606:12)
  at Function.Module._load (internal/modules/cjs/loader.js:598:3)
  at Module.require (internal/modules/cjs/loader.js:705:19)
  at require (internal/modules/cjs/helpers.js:14:16)
  at loadNuxtConfig (node_modules\@nuxt\cli\dist\cli-chunk.js:2459:17)
  at NuxtCommand.getNuxtConfig (node_modules\@nuxt\cli\dist\cli-chunk.js:2653:26)
```

## Steps to reproduce

1. clone this repository
2. `npm install`
3. `npm run dev`
4. make some changes to nuxt.config.ts after first compilation finished
   - this will have nuxt rebuild whole app

## The cause of this issue

The cause of this issue is that [registerTsNode](https://github.com/nuxt/nuxt.js/blob/dev/packages/cli/src/utils/typescript.js#L5) is called more than once.  
Because of this, ts-node tries compiling already transpiled .ts code and noImplicitAny error occurs.  
registerTsNode is called everytime in [_startDev](https://github.com/nuxt/nuxt.js/blob/dev/packages/cli/src/commands/dev.js#L39) via [getNuxtConfig](https://github.com/nuxt/nuxt.js/blob/dev/packages/cli/src/command.js#L94).
