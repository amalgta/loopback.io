---
title: Using ESLint with LoopBack repositories
tags: [contributing]
keywords: ESLint
last_updated: Sept 27, 2016
summary: "ESLint is a pluggable linting utility for JavaScript. Here are the guidelines for using it with LoopBack. "
sidebar: contrib_sidebar
permalink: /doc/en/contrib/eslint-guide.html
---

To begin with, clone the repo you want to fix and create a new branch: `feature/eslint` or name it as you find appropriate.
Following are the steps to introduce ESLint to LoopBack repositories:

To make reviews easier, it is encouraged to send three separate commits:

- [Add ESLint infrastructure](#add-eslint-infrastructure)
- [Auto-update by ESLint --fix](#auto-update-by-eslint---fix)
- [Fix linting errors](#fix-linting-errors)

## Add ESLint infrastructure

*Refer to [this commit](https://github.com/strongloop/strong-remoting/pull/288/commits/17b3bc3e7c17d1595dab1422c1052f8d90d2c27f) for an example*

- Remove all `jshint` and/or `jscs` files, i.e. `.jshintrc`, `.jscsrc` or `.jshintignore` files
- If the repo:
  - Is not using Grunt, install `eslint` as a `devDependencies` via `npm install eslint --save-dev`
  - Is using Grunt and depends on `grunt-jshint`, replace it with `grunt-eslint` (no need to install `eslint` as it's already installed as a dep from `grunt-eslint`)
- Update `devDependencies` by installing `eslint-config-loopback`
  - Run `npm install --save-dev eslint-config-loopback@latest` to ensure the latest version is installed
- In `package.json`'s `devDependencies` section, set `eslint` version to `^2.13.1`
  - Please refrain from using version `3.x.x` until we decide to drop support for versions of Node < 4
  - See http://eslint.org/blog/2016/07/eslint-v3.0.0-released#requires-nodejs-4-or-higher

- Add `.eslintrc` file with following content:

  ```json
   {
      "extends": "loopback"
    }
  ```

- Add `.eslintignore` file as well, if applicable
- If the repository is using `Grunt`, update the `Gruntfile.js` to use `eslint` _(refer to the  example commit)_
- Scan code base for any occurances of `jshint/jscs` and replace with `eslint` where appropriate. _(note: Mac users can run `grep -r jshint <dir/file> <dir/file> ...` to find occurances of word `jshint` in a dir/file)_
- Modify `package.json` and/or Gruntfile to ensure that linter (`eslint`) is run *after* the tests pass. This typically means adding a `"posttest": "eslint"` script. If the original package contains a `pretest` script to run the linter, then remove the `pretest` script in favour of `posttest`.

Commit with message `Add eslint infrastructure`

## Auto-update by eslint --fix

Once `eslint` infrastructure is setup, following steps will eliminate most linting errors, such as comma-dangle errors, automatically:

- Copy this file: https://raw.githubusercontent.com/maury91/eslint/b58cd73c2825a1f71f07a2869c1968472509cc83/lib/rules/comma-dangle.js to `/node_modules/eslint/lib/rules/comma-dangle.js` _(it will enable `eslint --fix` for `comma-dangle` rule)_
- from terminal, go to repo and run following command:
  `./node_modules/.bin/eslint --fix .`

Above command will auto-fix linting errors as per linter rules and will also show the errors it didn't fix.
Commit these changes with message: `Auto-update by eslint --fix`

## Fix linting errors

_When fixing errors, **DO NOT** change the original code because you might break things and it makes it harder for us to review! If you must change logic, do them in a **SEPARATE** PR._

_(disclaimer: this can be a tedious task)_

- from terminal, go to repo and run following command:
  `./node_modules/.bin/eslint .`
  it will display list of errors with a line-number that needs fixup. Take a note of number of errors reported

- If you see a high number of `max-len` violation, then add a custom `max-len` settings to `.eslintrc` as follows:
    ```json
    {
      "extends": "loopback",
      "rules": {
        "max-len": ["error", 90, 4, {
          "ignoreComments": true,
          "ignoreUrls": true,
          "ignorePattern": "^\\s*var\\s.+=\\s*(require\\s*\\()|(/)"
        }]
      }
    }
    ```
  and run `eslint` as in previous step.
  Try different values (90, 100, 110, 120) and find the lowest one where the number of failing lines is still reasonable.
  For example, in `strong-remoting`, `eslint` was reporting 76 errors for `max-len` of `100` and changing it to `90` was reporting `78` errors, thus lowest number `90` would make sense to set as max line length.
- usage of `'use strict';` may cause this error: `Uncaught SyntaxError: In strict mode code, functions can only               be declared at top level or immediately within another function`
  - Cause: function is declared inside a If/switch...etc block.
  - Solution: Move the function outside the block. For example:
  ```
    function funcName(parameter) {    
    }
    switch() {
      case 1: function() {
        funcName (parameter);
      }
    }
  ```
- Go to the files and fix errors manually and commit.
  - **Special case:**
    There is an issue with `one-var` rule in `eslint` which reports an error for a variable declared in a `for-loop` after an uninitialized variable declaration. for example:
      ```
        function reproduce(candidates) {
          var err;
          for (var ix in candidates) {
            err = ix > 0;
          }
          console.log(err);
        }
      ```
      `eslint` will complain as:
       `3:8  error  Combine this with the previous 'var' statement with uninitialized variables  one-var`

    (see filed bug [here](https://github.com/eslint/eslint/issues/5744))

    **solution _(for now)_**:

    The solution is to disable `one-var` rule where required.
    e.g.
       ```
         /* eslint-disable one-var */
          for (var key in body.error) {
            err[key] = body.error[key];
          }
         /* eslint-enable one-var */
      ```
      (need an example?! - have a look at [this commit](https://github.com/strongloop/strong-remoting/pull/288/commits/5ede708a0017cb87ae9fbe20da682a1f372c5044))

     _Don't forget to enable the rule as these enable/disable directives work at file level (not at method/scope level)_

    Push the commit with message: `Fix linting errors`

That's it, open a PR for review!
