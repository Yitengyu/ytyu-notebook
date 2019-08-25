# Babel 7加载配置策略

### webpack babel-loader 将配置loader时的配置传入。

```js
// node_modules/babel-loader/lib/index.js 
const config = babel.loadPartialConfig(injectCaller(programmaticOptions));
```



### babel寻找配置链，将babel.config.js与.babelrc中的配置合并

```js
// node_modules/@babel/core/lib/config/partial.js
const configChain = (0, _configChain.buildRootChain)(args, context);

// node_modules/@babel/core/lib/config/config-chain.js
function buildRootChain(opts, context) {
  const programmaticChain = loadProgrammaticChain({
    options: opts,
    dirname: context.cwd
  }, context);
  if (!programmaticChain) return null;
  let configFile;

  // 寻找配置文件，1.是否配置
  if (typeof opts.configFile === "string") {
    configFile = (0, _files.loadConfig)(opts.configFile, context.cwd, context.envName, context.caller);
  // 2. 若没有禁用，找根目录下的(怎么判断根目录？)
  } else if (opts.configFile !== false) {
    configFile = (0, _files.findRootConfig)(context.root, context.envName, context.caller);
  }

  // 找.babelrc
  // 1. 参数中是否有指定
  let {
    babelrc,
    babelrcRoots
  } = opts;
  let babelrcRootsDirectory = context.cwd;
  const configFileChain = emptyChain();

  if (configFile) {
    const validatedFile = validateConfigFile(configFile);
    const result = loadFileChain(validatedFile, context);
    if (!result) return null;

    // 2. babel.config.js中是否有指定
    if (babelrc === undefined) {
      babelrc = validatedFile.options.babelrc;
    }

    if (babelrcRoots === undefined) {
      babelrcRootsDirectory = validatedFile.dirname;
      babelrcRoots = validatedFile.options.babelrcRoots;
    }

    mergeChain(configFileChain, result);
  }

  // 从当前目录向上查找到package顶层目录(以package.json为界)(只允许单个)
  const pkgData = typeof context.filename === "string" ? (0, _files.findPackageData)(context.filename) : null;
  let ignoreFile, babelrcFile;
  const fileChain = emptyChain();

  if ((babelrc === true || babelrc === undefined) && pkgData && babelrcLoadEnabled(context, pkgData, babelrcRoots, babelrcRootsDirectory)) {
    ({
      ignore: ignoreFile,
      config: babelrcFile
    } = (0, _files.findRelativeConfig)(pkgData, context.envName, context.caller));

    if (ignoreFile && shouldIgnore(context, ignoreFile.ignore, null, ignoreFile.dirname)) {
      return null;
    }

    if (babelrcFile) {
      const result = loadFileChain(validateBabelrcFile(babelrcFile), context);
      if (!result) return null;
      mergeChain(fileChain, result);
    }
  }

  const chain = mergeChain(mergeChain(mergeChain(emptyChain(), configFileChain), fileChain), programmaticChain);
  return {
    plugins: dedupDescriptors(chain.plugins),
    presets: dedupDescriptors(chain.presets),
    options: chain.options.map(o => normalizeOptions(o)),
    ignore: ignoreFile || undefined,
    babelrc: babelrcFile || undefined,
    config: configFile || undefined
  };
}
```

