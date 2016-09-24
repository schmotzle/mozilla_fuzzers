# Framboise

## Introduction

Framboise is a fuzzer for WebAPIs. WebAPIs provide a way for javascript programms to retrieve system information from the device running the WebAPI.
Various APIs are implemented by both desktop and mobile browsers. A list of existing WebAPIs can be found on [https://developer.mozilla.org/de/docs/Web/WebAPI](https://developer.mozilla.org/de/docs/Web/WebAPI). Framboise uses [Address Sanitizer](https://developer.mozilla.org/en-US/docs/Mozilla/Testing/Firefox_and_Address_Sanitizer) to detect memory corruption in Firefox. 



## Installation

1. Clone framboise from github.

   ```bash
   git clone https://github.com/MozillaSecurity/framboise framboise.git
   ```


2. Download Firefox with address sanitizer support

   ```bash
   wget https://index.taskcluster.net/v1/task/gecko.v2.mozilla-central.latest.firefox.linux64-asan-opt/artifacts/public/build/target.tar.bz2
   ```

3. Unpack Firefox 

   ```bash
   tar xf target.tar.bz2
   ```

4. Adjust "<PATH>/" in settings file 

   ```bash
   vim framboise.git/settings/framboise.linux2.yaml
   ```

   ```yaml
   targets:
     firefox:
       plugin: firefox
       setups:
         default:
           environment: *environment
           application: <PATH>/firefox/firefox
           arguments: -no-remote -height 512 -width 512
           preferences: settings/firefox/prefs.js
           monitors: [[console, asan, testcase]]
           buckets:
             FilesystemLogger:
               <<: *FilesystemLogger
         inbound64-release:
           environment: *environment
           application: <PATH>/firefox/firefox
           arguments: -no-remote -height 512 -width 512
           preferences: settings/firefox/prefs.js
           monitors: [[console, asan, testcase]]
           buckets:
             FilesystemLogger:
               <<: *FilesystemLogger
         inbound64-debug:
           environment:
             <<: *environment
           application: <PATH>/firefox/firefox
           arguments: -no-remote -height 512 -width 512
           preferences: settings/firefox/prefs.js
           monitors: [[console, asan, testcase]]
           buckets:
             FilesystemLogger:
               <<: *FilesystemLogger
   ```

   ​

## Running

```bash
cd framboise.git
./framboise.py -fuzzer 1:Canvas2D -debug
```

## Internals

Listener classes

![Alt text](http://g.gravizo.com/g?
/**
*@opt all
*@note Object
*/
class Listener {
        public Void process_line;
        public String get_data;
        public Boolean detect_fault;
}
/**
*@opt shape activeclass
*@opt all
*/
class TestcaseListener extends Listener{};
/**
*@opt shape activeclass
*@opt all
*/
class AsanListener extends Listener{};
/**
*@opt shape activeclass
*@opt all
*/
class SyzyListener extends Listener{};
)


Monitor classes

```Dot
class Monitor{
  public void add_listener;
  
  public boolean detect_fault;
};

class ConsoleMonitor extends Monitor;
class WebSocketMonitor extends Monitor;

```

Plugins

```Dot

```

