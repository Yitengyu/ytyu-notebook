# NPM包开发心得

### 1. 选择合适的打包技术

Vue-cli-service并不方便。打包出来会有很多冗余的文件，build lib的可选配置项不多。



### 2. 注意配置`package.json`与`.npmignore`

`package.json`中dev-dependency、peer-dependency、dependency的合理配置。

npm发布默认使用`.gitignore`的配置，而实际上发布npm包的内容与git库维护的内容经常不一致，因此需要配置`.npmignore`文件。



### 3. 日志的维护很重要

日志的维护应该从1.0.0的版本就开始。



### 4. 控制打包结果的文件大小

比如不把node_modules里面的文件打包进来

比如打包成vant那样的结构



### 5. 建立Demo环境

建立demo环境可以方便开发。

但组件的开发还是“开发中抽象——在项目中使用迭代——再整理抽象到npm包”的过程可能更合理。