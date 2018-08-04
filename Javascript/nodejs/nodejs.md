## Node.js

### Definition

> Node.js is an open-source, cross-platform **JavaScript run-time environment** that executes JavaScript code **server-side**.   	-- WiKi [[1]][wiki-nodejs]

**JavaScript run-time environment** means you can run JavaScript code in Node.js environment like in console window of Chrome debug-tool.

**server-side** : Historically, JavaScript was used primarily for client-side scripting in which scripts written in JS are embedded in a webpage's HTML.But with Node.js, you can get rid of the limitation, write "JavaScript Everywhere".



### Node.js VS V8

> Node.js is built against modern versions of [V8](https://developers.google.com/v8/), and keeps up-to-date with the latest releases of this engine.[[2]][ nodejs-doc-es6 ]
>
> V8 is Google’s open source high-performance JavaScript engine, written in C++ and used in Google Chrome, the open source browser from Google, and in Node.js, among others. [[3]][v8-doc]
>
> [V8](https://www.wikiwand.com/en/Chrome_V8) is the JavaScript execution engine which was initially built for [Google Chrome](https://www.wikiwand.com/en/Google_Chrome). It was then open-sourced by Google in 2008. Written in [C++](https://www.wikiwand.com/en/C%2B%2B), V8 compiles JavaScript source code to native [machine code](https://www.wikiwand.com/en/Machine_code) instead of interpreting it in real time.  [[1]][wiki-nodejs]

As the quotes say,  comparing Node.js with V8 is not appropriate. Node.js uses V8 engine to compile JS code.

So apart from V8, what else makes up Node.js ?



### Node.js Architecture

![nodejs-architecture](./resources/nodejs-architecture.jpg)

> **libuv** (Unicorn Velociraptor Library) is a multi-platform C library that provides support for [asynchronous I/O](https://www.wikiwand.com/en/Asynchronous_I/O) based on [event loops](https://www.wikiwand.com/en/Event_loop).



### Why Asynchronous I/O and Event Loop ?

### How to use Node.js ?

### About NPM





[wiki-nodejs]: https://en.wikipedia.org/wiki/Node.js	"[1]"
[ nodejs-doc-es6 ]: https://nodejs.org/en/docs/es6/	"[2]"
[ v8-doc ]: https://developers.google.com/v8/	"[3]"