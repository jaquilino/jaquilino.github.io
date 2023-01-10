---
layout: post
title:  "Debug logging in Vue3"
date:   2023-01-09 8:00:00 -0600
categories: Tech
---

I had an issue occurring in some Vue code I was trying to get working.
In attempting to debug the logic, I defaulted to a _brute force_ approach
and just inserted a few ```console.log``` statements in my code.
Much to my surprise, the console output was reporting that the ```console``` object
was not defined, and that therefore there was no function named ```log```.
The reason is that Vue comparmentalizes the Javascript code within the ```script``` tag
for the component, and that global javascript variables are hidden from it.

some of you will be quick to tell me I shouldn't rely on the use of ```console.log``` for debugging anyway,
to prefer use of an interactive debugger such as [Vue.js Devtools](https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd?hl=en){:target=_blank}.
But I was foolishly trying to do something that I perceived as simple and straight-forward.
So how to replicate console logging in a Vue component?

I turned to use of a third-party component called ```vuejs3-logger```[Link](https://www.npmjs.com/package/vuejs3-logger){:target=_blank}.
(There are others, this is the one I chose.)
First I import it into my ```main.js``` module (yours might be named differntly).
```
import VueLogger from 'vuejs3-logger';
```
And then add it to the ```App``` by using ```provide``` to associate with an injectable variable
(the following should be inserted after instantiating the app object and before mount-ing it):
```
const logger = app.config.globalProperties.$log;
app.provide('$log', logger);
```

Then each component that you want to write debug output from, you would add the following:
```
        inject:    [ '$log' ],
        methods:  {
            logMessage(msg) {
                this.$log.debug(msg);
            }
        }
```
The logMessage function declareation above is not necessary, it's just to provide an example of outputting a debug message.
You may, of course, use a differnt variable than ```$log``` in the above, it'w what I chose to use.

One can also invoke the module inside of a component's ```<script setup>``` section.
Add the following to ```<script setup>```:
```
    import { inject } from 'vue'; 
    const log = inject('$log');
    function f() {
         log.debug('f invoked');
    }
```
If your component doesn't use ```<script setup>```, there's no need to add it for logging.

The ```vuejs3-logger``` module understands production building and automatically ignores calls to the debug method,
so you don't have to warry about removing debug lines from your production build.
