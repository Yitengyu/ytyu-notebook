## vue-cli-service

这次在做项目时，很好奇为什么没有webpack的配置文件，猜测是vue-cli-service大概做了这些工作，就试着自己搭建这个脚手架。

#### vue-cli-service inspect

这个命令能够输出当前环境中实际使用的webpack配置。

在只写了入口js文件与安装了`@vue/cli-service`（与其必要peer依赖）之后，就能够用这个命令输出一个庞大详细的webpack配置——证明了的确是vue-cli-service中内置了基础的webpack配置。

这个配置做了什么呢？我简单看了一下，能看到以下的支持

1. 定义了默认的入口`[rootPath]/src/main.js`和默认输出路径`[rootPath]/dist/`
2. 定义了路径别名`@`=>`[rootPath]/src/`
3. 默认加载的文件类型



插件系统，每个插件都按照这些标准做类似的事情，对其进行补充。



以协约的方式，减少了很多人工配置的麻烦。



noParse是不是可以优化。



1. vue: vue-loader -> cache loader
2. images: url-loader
3. css: css-loader -> vue-style-loader
4. scss, sass: sass-loader -> css-loader -> vue-style-loader



插件

```javascript
/* config.plugin('vue-loader') */
/* config.plugin('define') */
/* config.plugin('case-sensitive-paths') */
/* config.plugin('friendly-errors') */
/* config.plugin('hmr') */
/* config.plugin('no-emit-on-errors') */
/* config.plugin('progress') */
/* config.plugin('html') */
/* config.plugin('preload') */
/* config.plugin('prefetch') */
```



对外接口