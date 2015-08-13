node-emscripten
===============
minmal npm installer for emscripten binaries with zero dependencies

[![NPM](https://img.shields.io/npm/v/emscripten.svg?style=flat-square)](https://www.npmjs.org/package/emscripten)



# build-status [![travis-ci.org build-status](https://api.travis-ci.org/kaizhu256/node-emscripten.svg)](https://travis-ci.org/kaizhu256/node-emscripten)

| git-branch : | [master](https://github.com/kaizhu256/node-emscripten/tree/master) | [beta](https://github.com/kaizhu256/node-emscripten/tree/beta) | [alpha](https://github.com/kaizhu256/node-emscripten/tree/alpha)|
|--:|:--|:--|:--|
| build-artifacts : | [![build-artifacts](https://kaizhu256.github.io/node-emscripten/glyphicons_144_folder_open.png)](https://github.com/kaizhu256/node-emscripten/tree/gh-pages/build..master..travis-ci.org) | [![build-artifacts](https://kaizhu256.github.io/node-emscripten/glyphicons_144_folder_open.png)](https://github.com/kaizhu256/node-emscripten/tree/gh-pages/build..beta..travis-ci.org) | [![build-artifacts](https://kaizhu256.github.io/node-emscripten/glyphicons_144_folder_open.png)](https://github.com/kaizhu256/node-emscripten/tree/gh-pages/build..alpha..travis-ci.org)|

#### master branch
- stable branch
- HEAD should be tagged, npm-published package

#### beta branch
- semi-stable branch
- HEAD should be latest, npm-published package

#### alpha branch
- unstable branch
- HEAD is arbitrary
- commit history may be rewritten



# quickstart cli example

#### to run this example, follow the instruction in the script below
- example.sh

```shell
# example.sh

# this shell script will
    # npm install node-emscripten
    # create local test file hello.txt with data "hello"
    # http PUT hello.txt to $GITHUB_CRUD_FILE
    # http GET $GITHUB_CRUD_FILE and print to stdout
    # validate $GITHUB_CRUD_FILE data is "hello"
    # http DELETE $GITHUB_CRUD_FILE
    # validate deleted $GITHUB_CRUD_FILE does not exist

# instruction
    # 1. set env var $GITHUB_CRUD_FILE to test crud operations

        # uncomment line below, and set env var $GITHUB_CRUD_FILE
        # GITHUB_CRUD_FILE=https://github.com/john/my-repo/blob/master/hello.txt

    # 2. goto https://github.com/settings/tokens,
    #    and create env var $GITHUB_TOKEN with access to $GITHUB_CRUD_FILE

        # uncomment line below, and set env var $GITHUB_TOKEN
        # GITHUB_TOKEN=ffffffffffffffffffffffffffffffffffffffff

    # 3. after editing above lines,
    #    copy and paste this entire shell script into a console and press enter
    # 4. watch this script PUT / GET / DELETE $GITHUB_CRUD_FILE

shExampleSh() {
    # npm install node-emscripten
    npm install node-emscripten || return $?
    alias node-emscripten=node_modules/.bin/node-emscripten || return $?

    # create local test file hello.txt with data "hello"
    printf "hello" > hello.txt || return $?

    # http PUT hello.txt to $GITHUB_CRUD_FILE
    node-emscripten contentPutFile $GITHUB_CRUD_FILE hello.txt || return $?

    # http GET $GITHUB_CRUD_FILE and print to stdout
    DATA=$(node-emscripten contentGet $GITHUB_CRUD_FILE) || return $?
    printf "$DATA\n" || return $?

    # validate $GITHUB_CRUD_FILE data is "hello"
    [ "$DATA" = hello ] || return $?

    # http DELETE $GITHUB_CRUD_FILE
    node-emscripten contentDelete $GITHUB_CRUD_FILE || return $?

    # validate deleted $GITHUB_CRUD_FILE does not exist
    [ "$(node-emscripten contentGet $GITHUB_CRUD_FILE)" = "" ] || return $?
}
shExampleSh
```

#### output from shell
[![screen-capture](https://kaizhu256.github.io/node-emscripten/build/screen-capture.testExampleSh.png)](https://travis-ci.org/kaizhu256/node-emscripten)



# package-listing
[![screen-capture](https://kaizhu256.github.io/node-emscripten/build/screen-capture.gitLsTree.svg)](https://github.com/kaizhu256/node-emscripten)



# package.json
```json
{
    "dependencies": {
        "utility2": "git+https://github.com/kaizhu256/node-utility2.git#alpha"
    },
    "name": "kaizhu-node-emscripten",
    "repository" : {
        "type" : "git",
        "url" : "https://github.com/kaizhu256/node-emscripten.git"
    },
    "scripts": {
        "build-ci": "node_modules/.bin/utility2 shRun shReadmeBuild",
        "postinstall": "./npm-postinstall.sh",
        "preinstall": "touch phantomjs slimerjs",
        "test": "node_modules/.bin/utility2 shRun shReadmeExportPackageJson && \
for ARG0 in phantomjs slimerjs; \
do \
printf \"testing $ARG0\n\" || exit $?; \
[ \
$(./index.js $ARG0 eval 'console.log(\"hello\"); phantom.exit();') = 'hello' \
] || exit $?; \
printf \"passed\n\" || exit $?; \
done"
    },
    "version": "2015.8.1"
}
```



# todo
- none



# change since 9fe8c225
- version 2015.8.1
- test emscripten build on linux and osx using codeship.io and travis-ci.org
- none



# changelog of last 50 commits
[![screen-capture](https://kaizhu256.github.io/node-emscripten/build/screen-capture.gitLog.svg)](https://github.com/kaizhu256/node-emscripten/commits)



# internal build-script
- build.sh

```shell
# build.sh

# this shell script will run the build for this package

shBuild() {
    # this function will run the main build
    local TEST_URL || return $?

    # init env
    export npm_config_mode_slimerjs=1 || return $?
    . node_modules/.bin/utility2 && shInit || return $?

    # run npm-test on published package
    shRun shNpmTestPublished || return $?

    # test example js script
    export npm_config_timeout_exit=10000 || return $?
    MODE_BUILD=testExampleJs shRunScreenCapture shReadmeTestJs example.js || return $?
    unset npm_config_timeout_exit || return $?

    # run npm-test
    MODE_BUILD=npmTest shRunScreenCapture npm test || return $?

    # create api-doc
    shDocApiCreate \
        '{moduleDict:{"node-emscripten":'\
        '{alias:"swmg",module:require("./index.js")}}}' || return $?

    # if running legacy-node, then do not continue
    [ "$(node --version)" \< "v0.12" ] && return

    # deploy app to heroku
    shRun shHerokuDeploy hrku01-$npm_package_name-$CI_BRANCH || return $?

    # test deployed app to heroku
    if [ "$CI_BRANCH" = alpha ] ||
        [ "$CI_BRANCH" = beta ] ||
        [ "$CI_BRANCH" = master ]
    then
        TEST_URL="https://hrku01-$npm_package_name-$CI_BRANCH.herokuapp.com" || return $?
        TEST_URL="$TEST_URL?modeTest=phantom&timeExit={{timeExit}}" || return $?
        MODE_BUILD=herokuTest shPhantomTest "$TEST_URL" || return $?
    fi
}
shBuild

# save exit-code
EXIT_CODE=$?
# create package-listing
MODE_BUILD=gitLsTree shRunScreenCapture shGitLsTree || return $?
# create recent changelog of last 50 commits
MODE_BUILD=gitLog shRunScreenCapture git log -50 --pretty="%ai\u000a%B" || return $?
# upload build-artifacts to github, and if number of commits > 16, then squash older commits
COMMIT_LIMIT=16 shBuildGithubUpload || exit $?
exit $EXIT_CODE
```
