# 1. Firefox

A Dockerfile to dockerize any version of Mozilla Firefox and push it within a Selenium grid for automation tests.

<!-- TOC -->

- [1. Firefox](#1-firefox)
    - [1.1. tl;dr](#11-tldr)
    - [1.2. Getting started](#12-getting-started)
        - [1.2.1. Build](#121-build)
            - [1.2.1.1. Basic build](#1211-basic-build)
            - [1.2.1.2. Complete build](#1212-complete-build)
            - [1.2.1.3. Example with Firefox 52.2.1-ESR](#1213-example-with-firefox-5221-esr)
        - [1.2.2. Run](#122-run)
            - [1.2.2.1. Basic run](#1221-basic-run)
            - [1.2.2.2. Complete run](#1222-complete-run)

<!-- /TOC -->

## 1.1. tl;dr

To run everything in a single terminal, use _detached mode_, with the _-d_ argument.

```shell
$ docker build -t gigouni/firefox53 .
$ docker run \
    -it \
    --rm \
    -p 4444:4444 \
    --net=host \
    --name selenium-hub \
    gigouni/hub-3.4.0-dysprosium
$ docker run \
    -it \
    --rm \
    -e HUB_PORT_4444_TCP_ADDR=172.17.0.1 \
    -e HUB_PORT_4444_TCP_PORT=4444 \
    -e NODE_PORT=5558 \
    gigouni/firefox53
```

## 1.2. Getting started

### 1.2.1. Build

You can build Docker images of Firefox using the version you need [from the list here](https://ftp.mozilla.org/pub/firefox/releases/).

#### 1.2.1.1. Basic build

```shell
$ docker build -t gigouni/firefoxYOUR_VERSION .
```

#### 1.2.1.2. Complete build

Build a Docker image following your needed version

```shell
$ docker build --build-arg FIREFOX_VERSION=YOUR_VERSION -t gigouni/firefoxYOUR_VERSION .
```

#### 1.2.1.3. Example with Firefox 52.2.1-ESR

```shell
$ # Build the Firefox 52.2.1-ESR Docker image
$ docker build \
    --build-arg FIREFOX_VERSION=52.2.1esr \
    -t gigouni/firefox52.2.1-esr \
    .

$ # Run the gigouni/firefox52.2.1-esr image
$ docker run \
    -it \
    --rm \
    -e HUB_PORT_4444_TCP_ADDR=172.17.0.1 \
    -e HUB_PORT_4444_TCP_PORT=4444 \
    -e NODE_PORT=5558 \
    gigouni/firefox52.2.1-esr
```

__For FF < 48__

_Geckodriver_, the Mozilla Firefox driver, is suitable [since the 48.0](https://github.com/mozilla/geckodriver/issues/85). Before this version, you need to disable Marionette by adding

```javascript
let firefoxCap = Capabilities.firefox();
// use legacy drivers (by setting marionette: false)
// https://github.com/angular/protractor/issues/4170
firefoxCap.set('marionette', false);
```

but you'll get another error

```shell
Error: You may not use a custom command executor with the legacy FirefoxDriver
```

It's a [known issue](https://stackoverflow.com/questions/42220507/protractor-cant-run-in-firefox-works-fine-in-chrome/42430278#42430278) and there are two solutions.

- First: running tests on higher versions of Firefox --> we want to test on FF41, not interesting
- Second: downgrade the current version of Selenium from 3.4.0 to 2.5x.y --> we would have two Selenium grids, not interesting.

The problem remains unresolved. From now on, we're considering impossible to build tests on Firefox < 48 images.

### 1.2.2. Run

Run your Selenium hub (_if it's not already running_)

```shell
$ docker run -it --rm -p 4444:4444 --name selenium-hub selenium/hub:3.4.0-dysprosium
```

#### 1.2.2.1. Basic run

Run your new Firefox image

```shell
$ docker run \
    -it \
    --rm \
    -e HUB_PORT_4444_TCP_ADDR=172.17.0.1 \
    -e HUB_PORT_4444_TCP_PORT=4444 \
    -e NODE_PORT=5558 \
    gigouni/firefoxYOUR_VERSION
```

#### 1.2.2.2. Complete run

```shell
$ docker run \
    -it \
    --rm \
    -e FIREFOX_VERSION=53.0 \
    -e GECKODRIVER_VERSION=0.18.0 \
    -e HUB_PORT_4444_TCP_ADDR=172.17.0.1 \
    -e HUB_PORT_4444_TCP_PORT=4444 \
    -e NODE_MAX_INSTANCES=1 \
    -e NODE_APPLICATION_NAME=node-firefox \
    -e NODE_MAX_SESSION=1 \
    -e NODE_PORT=5558 \
    -e NODE_REGISTER_CYCLE=5000 \
    -e NODE_POLLING=5000 \
    -e NODE_UNREGISTER_IF_STILL_DOWN_AFTER=60000 \
    -e NODE_DOWN_POLLING_LIMIT=2 \
    gigouni/firefoxYOUR_VERSION
```

Check if it's working [here](http://localhost:4444/grid/console).