### NPM VS Yarn

#### What is NPM?

#### What is package manager ?

#### What is the relation of NPM and Yarn

### NPM曾经有过的缺点

1. 下载速度慢。  （没有并行？
2. 使用相同的package.json，得到安装结果可能不一样。



  Yarn使用什么办法解决了这个问题？



NPM后来是否借鉴Yarn完善了自己？



#### package-lock.json做什么用？

> `package-lock.json` is automatically generated for any operations where npm modifies either the `node_modules` tree, or `package.json`. It describes the exact tree that was generated, such that subsequent installs are able to generate identical trees, regardless of intermediate dependency updates.

package-lock.json会在使用npm对模块树（node_modules tree）或者package.json修改时进行实时更新。

它能够描述当前产生的模块树的准确特征，因此在拥有它的情况下，可以还原当前安装的node_modules。

因此它有这样一些应用：

1. 准确描述依赖树，保证开发，部署，持续集成时的一致性。
2. 让开发者不需要将node_modules目录提交版本管理系统，也能方便地回滚查看。
3. 让开发者可以通过版本控制的diff功能，清晰看到依赖树的变化。
4. 使npm可以跳过已安装模块的元数据收集以优化安装过程。（And optimize the installation process by allowing npm to skip repeated metadata resolutions for previously-installed packages.）

