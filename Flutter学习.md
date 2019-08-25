# Flutter官方基础教程体验

## 现状了解

一个由谷歌推出的多端统一开发的解决方案。在18年12月推出了第一个release版本。目前国内阿里巴巴的闲鱼团队采取了相当规模的应用，详见[闲鱼技术博客](https://yq.aliyun.com/teams/290)；美团也贡献了一系列入门文章[Flutter的原理及美团的实践](https://zhuanlan.zhihu.com/p/41731412)

看了国内外一些对其介绍的博客的评论，有以下两点总结：

* 最能打的特点是： 统一，快。
* 被吐槽比较多的点是： 层叠的回调式写法，‘另一个RN’的唱衰论调。

## 安装

按照官网提示按部就班。

```
1. 下载macOS的安装包，解压，添加路径到PATH
2. 运行flutter docotr，缺什么就补什么
3. 选用VSCode作为IDE，根据提示启动一个项目。项目自带Demo代码。
4. 在状态栏指定iPhone的模拟器，debug运行程序（或者跑flutter run），顺利地在模拟器上看到了效果。
5. 改动字符串，可以看到非常迅速的热加载效果。
```



## 学习Dart语言概述（以ts为基础样板）

[Dart英文教程](https://www.dartlang.org/guides/language/language-tour)

[Dart中文教程](http://dart.goodev.org/guides/language/language-tour)

1. Dart有什么好处？

```
参考： https://juejin.im/entry/5b0f94a36fb9a009fb32cd08
1. 能AOT编译成native的机器码，避免了如js等动态语言通过bridge与平台通信的overhead。
2. 支持JIT编译，因此可以很好地支持热编译。
3. 是单线程的，显式让出CPU避免竞争。
4. 良好的对象分配和垃圾回收机制。分代垃圾回收和对象分配方案
5. 统一布局。(就像是把html js css糅在一块儿写)让人专注。
```

2. 以ts的基础，有什么需要注意学习的？

```
// 粗略看一遍的总结
1. final关键字，与更复杂的const
2. 具有顶级入口函数（top-level function）main()
3. 蛮特别的级联调用(cascade notation) ..
4. 相等符号在对象语义上的差别；没有===
5. 多了 ~/ 运算符
6. 更强更复杂的class, 近似C++
7. 广泛使用的可选命名参数(optional named parameter)
8. metadata
...
```



## 按教程搭建一个小页面

[教程part1](https://flutter.dev/docs/get-started/codelab), [教程part2](https://codelabs.developers.google.com/codelabs/first-flutter-app-pt2)

按照教程指导，可以实现一个无限下拉的列表，支持触摸选中列表中的项，并在另一个页面集中展示。通过教程能了解到[Material](https://flutter.dev/docs/development/ui/widgets/material)的简单使用，对`flutter`和`dart`有一个初步的印象。



## 坑点

1. [package get执行慢怎么办？](https://flutter.dev/community/china)。需要配置一下国内可用的镜像源。

2. 如果遇上Waiting for another flutter command to release the startup lock...， 手动kill dart的进程。



## 个人总结

```
1. 热编译快。这一点体验很好，甚至比前端项目热编译更快(也可能只是因为教程项目规模小)。
2. Material有很多的API需要记忆，而且很多的名字真的很长(Dart实践中就建议不要用缩写)。在VSCode的扩展中就看到了一些snippet类的缩写支持插件。
3. Material组件库内容比较丰富。对于定制组件需求不高的项目（比如EHR）而言可能是个能提高生产力的方案？
4. VSCode对Flutter调试的体验很顺畅。而一些代码补全的体验还不太好。
5. 又要写分号了...非常不习惯。
```



## 附：完成教程后的代码

食用方法： 

1. 生成dart项目后替换main.dart文件中的代码
2. 在`pubspec.yaml`中添加依赖englissh_words: ^3.1.0后更新package。
3. flutter run

```dart
// main.dart
import 'package:flutter/material.dart';
import 'package:english_words/english_words.dart';

void main() => runApp(MyApp());

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Hello World',
      home: RandomWords()
    );
  }
}

class RandomWords extends StatefulWidget {
  @override
  RandomWordsState createState() => new RandomWordsState();
}

class RandomWordsState extends State<RandomWords> {
  final _suggestions = <WordPair>[];
  final _saved = new Set<WordPair>();

  Widget _buildSuggestions() {
    return ListView.builder(
      itemBuilder: (context, i) {
        if (i.isOdd) return Divider();

        final index = i ~/ 2;
        if (index >= _suggestions.length) {
          _suggestions.addAll(generateWordPairs().take(10));
        }
        return _buildRow(_suggestions[index]);
      },
    );
  }

  Widget _buildRow(WordPair pair) {
    final hasSaved =_saved.contains(pair);
    return ListTile(
      title: Text(
        pair.asPascalCase,
      ),
      trailing: Icon(
        hasSaved ? Icons.favorite : Icons.favorite_border,
        color: hasSaved ? Colors.red : null
      ),
      onTap: () {
        setState(() {
          hasSaved
          ? _saved.remove(pair)
          : _saved.add(pair);
        });
      },
    );
  }

  void _pushSaved() {
    Navigator.of(context).push(
      MaterialPageRoute(
        builder: (BuildContext context) {
          final Iterable<ListTile> tiles = _saved.map(
            (WordPair pair) {
              return ListTile(
                title: Text(pair.asPascalCase),
              );
            }
          );

          final List<Widget> divided = ListTile
          .divideTiles(
            context: context,
            tiles: tiles
          )
          .toList();

          return Scaffold(
            appBar: AppBar(
              title: Text('favorited pairs'),
            ),
            body: ListView(children: divided)
          );

        }
      )
    );
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(
        title: Text('Startup Name Generator'),
        actions: <Widget>[
          IconButton(icon: Icon(Icons.list), onPressed: _pushSaved,)
        ],
      ),
      body: _buildSuggestions(),
    );
  }
}
```

