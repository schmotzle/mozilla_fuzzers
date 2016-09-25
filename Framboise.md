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

### Listener classes

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

* TestcaseListener searches json data within the recorded input and returns it via "get_data" call. "detected_fault" returns always True. Returns data to bucket 'testcase'.
* AsanListener searches for string "ERROR: AddressSanitizer" within the recorded input. "detected_fault" returns True if the string was found and False if not. Returns data to bucket 'crashlog'.
* SyzyListener searches for string "SyzyASAN error:" within the recorded input. "detected_fault" returns True if the string was found and False if not. Returns data to bucket 'crashlog'.

### Monitor classes

![Alt text](http://g.gravizo.com/g?
/**
*@opt all
*@note Thread
*/
class Monitor implements Listener{
        public Void add_listener;
        public Boolean detect_fault;
};
/**
*@opt all
*/
class ConsoleMonitor extends Monitor{};
/**
*@opt all
*/
class WebSocketMonitor extends Monitor{};
/**
*@opt all
*/
interface Listener{};
)


### Plugins

![Alt text](http://g.gravizo.com/g?
/**
*@opt all
*/
class BasePlugin{
        public Void open;
        public Boolean stop;
};
/**
*@opt all
*/
class ExternalProcess extends BasePlugin{};
/**
*@opt all
*/
class DefaultPlugin extends BasePlugin{};
/**
*@opt all
*/
class FirefoxPlugin extends BasePlugin{};
/**
*@opt all
*/
class IexplorerPlugin extends BasePlugin{};
/**
*@opt all
*/
class B2G_EmulatorPlugin extends BasePlugin{};
)

## Pitfalls

### NULL Pointer in libxul

```bash
./framboise.py -fuzzer 1:Canvas2d -debug
[Framboise] 2016-09-24 06:31:41,893 INFO: Calling command: ['mozilla/Framboise/firefox/firefox', '-no-remote', '-CreateProfile', 'tmpJ9jPpS /tmp/tmpJ9jPpS']
Success: created profile 'tmpJ9jPpS /tmp/tmpJ9jPpS' at '/tmp/tmpJ9jPpS/prefs.js'
[Framboise] 2016-09-24 06:31:42,252 INFO: Running command: ['~/mozilla/Framboise/firefox/firefox', '-P', 'tmpJ9jPpS', '-no-remote', '-height', '512', '-width', '512', 'file:///~mozilla/Framboise/framboise.git/index.html?ws-logger=0&seed=None&fuzzer=1:Canvas2d&with-events=False&timeout=0&with-set-timeout=False&debug=True&max-commands=100&with-set-interval=False']
[Framboise] 2016-09-24 06:31:42,257 INFO: Logger FilesystemLogger initialized.
too much recursion
ASAN:DEADLYSIGNAL
=================================================================
==37478==ERROR: AddressSanitizer: SEGV on unknown address 0x000000000000 (pc 0x7fbebc791c74 bp 0x7fff2f74bac0 sp 0x7fff2f74baa0 T0)
#0 0x7fbebc791c73  (~/mozilla/Framboise/firefox/libxul.so+0x3997c73)
#1 0x7fbebc7199c8  (~/mozilla/Framboise/firefox/libxul.so+0x391f9c8)
#2 0x7fbec274d085  (~/mozilla/Framboise/firefox/libxul.so+0x9953085)
#3 0x7fbebad793a3  (~/mozilla/Framboise/firefox/libxul.so+0x1f7f3a3)
#4 0x7fbebad7a812  (~/mozilla/Framboise/firefox/libxul.so+0x1f80812)
#5 0x7fbebad71ae1  (~/mozilla/Framboise/firefox/libxul.so+0x1f77ae1)
#6 0x7fbebae12df1  (~/mozilla/Framboise/firefox/libxul.so+0x2018df1)
#7 0x7fbebadfcb80  (~/mozilla/Framboise/firefox/libxul.so+0x2002b80)
#8 0x7fbec37aeb82  (~/mozilla/Framboise/firefox/libxul.so+0xa9b4b82)
#9 0x7fbec37afa4c  (~/mozilla/Framboise/firefox/libxul.so+0xa9b5a4c)
#10 0x4df89a  (~/mozilla/Framboise/firefox/firefox+0x4df89a)
#11 0x7fbed67635ef  (/lib/x86_64-linux-gnu/libc.so.6+0x205ef)
#12 0x41ba08  (~/mozilla/Framboise/firefox/firefox+0x41ba08)

AddressSanitizer can not provide additional info.
SUMMARY: AddressSanitizer: SEGV (~/mozilla/Framboise/firefox/libxul.so+0x3997c73)
==37478==ABORTING
[Framboise] 2016-09-24 06:31:42,615 INFO: Exit code: 1
[Framboise] 2016-09-24 06:31:42,617 INFO: Stopping Framboise.
[Framboise] 2016-09-24 06:31:42,623 ERROR: [Errno 3] No such process

```
