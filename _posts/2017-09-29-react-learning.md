---

layout: post

title: "react学习"

date: 2017-09-29

categories: [JavaScript]

---

* content
{:toc}

## JSX In Depth
### 可以使用``.``来引用JSX的组件
```js
import React from 'react';
const MyComponents = {
  DatePicker: function DatePicker(props) {
    return <div>Imagine a {props.color} datepicker here.</div>;
  }
}
function BlueDatePicker() {
  return <MyComponents.DatePicker color="blue" />;
}
```

### 用户定义的组件名字必须大写、

### 可以在运行时决定渲染的类型
 这样做的前提是，把希望运行时决定真正值的表达式赋值给一个大写的变量
  ```js
  import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // Correct! JSX type can be a capitalized variable.
  const SpecificStory = components[props.storyType];
  return <SpecificStory story={props.story} />;
}
  ```
### 属性的用法
- js表达式
- 属性值默认为true
  ```js
  <MyTextBox autocomplete />
  <MyTextBox autocomplete={true} />
  ```
- 可以将对象展开为属性
  ```js
  function App2() {
    const props = {firstName: 'Ben', lastName: 'Hector'};
    return <Greeting {...props} />;
  }
  ```
### JSX中的子元素
  JSX中，开标签与闭标签之间的东西以``props.children``来表示,并且传入其中的文本将会对html字符进行z转义

### ``Booleans, Null, and Undefined``被忽略

## 使用PropTypes进行类型检查
使用``prop-types``库引入
```js
import PropTypes from 'prop-types';
class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

Greeting.propTypes = {
  name: PropTypes.string
};
```
### 只有一个子元素
使用``props.children``来保证z还有一个子元素
```js
import PropTypes from 'prop-types';

class MyComponent extends React.Component {
  render() {
    // This must be exactly one element or it will warn.
    const children = this.props.children;
    return (
      <div>
        {children}
      </div>
    );
  }
}

MyComponent.propTypes = {
  children: PropTypes.element.isRequired
};
```

### 属性的默认值
```js
class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}
// Specifies the default values for props:
Greeting.defaultProps = {
  name: 'Stranger'
};
```

## 引用和操作DOM
React使用了``ref``属性来定义一个回调函数，定义了``ref``属性的DOM元素将会作为参数传入该函数中。这个函数在元素``mount``和``unmount``后，会被执行。当``unmount``时，传入函数的参数为``null``.
```js
 <input
  type="text"
  ref={(input) => { this.textInput = input; }} />
```
**注意**: 
1. 不能把它用在函数型组建中，只能用在``Class Component``上。
2. 当用在``Class Component``上时，传入函数的参数是组件实例，而不是DOM节点

### 暴露DOM节点给父元素
子元素可以暴露某个Node给父元素，方法是，父元素将某个函数传递给子元素，在这个函数中，父元素会把参数绑定到自己的变量中。
然后，子元素接收到这个函数类型的参数后，把它赋值给某个DOM的``ref``属性，这样，子元素挂载时，就会把这个DOM传给父元素的属性。
```js
  function CustomTextInput(props) {
    return (
      <div>
        <input ref={props.inputRef} />
      </div>
    );
  }
  class Parent extends React.Component {
    render() {
      return (
        <CustomTextInput
          inputRef={el => this.inputElement = el}
        />
      );
    }
  }
```

## Portals
Portals可以挂载元素到任意的DOM节点之下。
```js
render() {
  // React does *not* create a new div. It renders the children into `domNode`.
  // `domNode` is any valid DOM node, regardless of its location in the DOM.
  return ReactDOM.createPortal(
    this.props.children,
    domNode,
  );
}
```
即使使用``ReactDOM.createPortal()``创建的元素实际的DOM树种并不是父元素的子元素，但是事件冒泡依然会冒到在React中规定的父元素上。

## 表单
React中没有双向绑定的概念，需要利用``state``和事件机制来绑定
```js
class Reservation extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      isGoing: true,
      numberOfGuests: 2
    };

    this.handleInputChange = this.handleInputChange.bind(this);
  }

  handleInputChange(event) {
    const target = event.target;
    const value = target.type === 'checkbox' ? target.checked : target.value;
    const name = target.name;

    this.setState({
      [name]: value
    });
  }

  render() {
    return (
      <form>
        <label>
          Is going:
          <input
            name="isGoing"
            type="checkbox"
            checked={this.state.isGoing}
            onChange={this.handleInputChange} />
        </label>
        <br />
        <label>
          Number of guests:
          <input
            name="numberOfGuests"
            type="number"
            value={this.state.numberOfGuests}
            onChange={this.handleInputChange} />
        </label>
      </form>
    );
  }
}
```