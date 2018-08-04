## config

file path ~/.gitconfig



### set proxy

1.首先第一步前提是已经打开了SS代理。

2.如果要设置全局代理，可以依照这样设置

```
git config --global http.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080
```

3.完成上面一步后，此时输入git clone xxxxxxx就可以利用代理进行下载了

**但同时，也请注意，这里指的是https协议，也就是**

```
git clone https://www.github.com/xxxx/xxxx.git
```

这种

对于SSH协议，也就是

```
git clone git@github.com:xxxxxx/xxxxxx.git
```

这种，依旧是无效的

**4.以上为总结，但我不推荐直接用全局代理**

因为如果挂了全局代理，这样如果需要克隆coding之类的国内仓库，会奇慢无比

所以我建议使用这条命令，只对github进行代理，对国内的仓库不影响

```
git config --global http.https://github.com.proxy https://127.0.0.1:1080
git config --global https.https://github.com.proxy https://127.0.0.1:1080
```

同时，如果在输入这条命令之前，已经输入全局代理的话，可以输入进行取消

```
git config --global --unset http.proxy
git config --global --unset https.proxy
```

 

 

 

