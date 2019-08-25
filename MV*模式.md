# MV*架构

> <[Learning JavaScript Design Patterns](https://addyosmani.com/resources/essentialjsdesignpatterns/book/)>

## MVC

1. MVC原则
   1. Model层不知道View和Controller。当Model层改变，通知其观察者
   2. View表示着Model的一个状态。利用观察者模式，view知道Model层的改变。
   3. View层控制着表现。不仅仅一对View-Controller
   4. Controller负责处理用户交互，指导View层
   5. 所以View并不直接知道Model的变化？



尽管现在发布订阅模式广泛应用在框架中，但开发者会惊讶于观察者模式在数十年前就已经应用于MVC架构了——SmallTalk中，就有model改变，view就跟着改了。

MVC的例子：Backbone，Ember.js， AngularJS

### Models

代表了业务数据。

### Views

> Note that templates are not themselves views. Developers coming from a Struts Model 2 architecture may feel like a template *is* a view, but it isn't. A view is an object which observes a model and keeps the visual representation up-to-date. A template *might* be a declarative way to specify part or even all of a view object so that it can be generated from the template specification.

模板不是视图层，视图层是观察数据层并保持更新的一个对象。

模板只是其中一种生成视图层的方法。

一个典型的例子。

```js
var buildPhotoView = function ( photoModel, photoController ) {
 
  var base = document.createElement( "div" ),
      photoEl = document.createElement( "div" );
 
  base.appendChild(photoEl);
 
  var render = function () {
          // We use a templating library such as Underscore
          // templating which generates the HTML for our
          // photo entry
          photoEl.innerHTML = _.template( "#photoTemplate", {
              src: photoModel.getSrc()
          });
      };
 
  // view层构建时，订阅了model的变化
  photoModel.addSubscriber( render );
 
  // ui交互交给控制层
  photoEl.addEventListener( "click", function () {
    photoController.handleEvent( "click", photoModel );
  });
 
  // 可以看到这里的视图层控制并没有完全数据化。
  var show = function () {
    photoEl.style.display = "";
  };
 
  var hide = function () {
    photoEl.style.display = "none";
  };
 
  return {
    showView: show,
    hideView: hide
  };
 
};
```

### Controllers

负责对用户交互做出响应，去修改model层数据。

1. 数据绑定是MVVM的一个常见实现方法

