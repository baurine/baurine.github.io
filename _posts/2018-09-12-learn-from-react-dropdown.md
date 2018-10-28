---
layout: post
title: "从 Dropdown 的 React 实现中学习到的"
description: ""
category:
tags: [dropdown, react]
---

## Demo

[Demo Link](https://baurine.github.io/react-dropdown/)

![]({{site.img_url}}/react-dropdown/dropdown.gif)

![]({{site.img_url}}/react-dropdown/dropdown.png)

## Note

dropdown 是一种很常见的 component，一般有两种：

1. 展开 dropdown menu 后，点击任意地方都应该收起 menu。
1. 展开 dropdown menu 后，点击 menu 内部，不会收起 menu，只有点击 menu 外部，才收起 menu。

在 jQuery 时代，dropdown 是很好实现的，直接用 `document.addEventListener('click', handler)`，监听 document 的 click 事件，然后让 dropdown 的 menu 隐藏起来。如果想让 menu 内部的点击不收起 menu，则让 menu 内部的点击事件执行 `event.stopPropagation()`。

刚开始做 React 开发的时候，不知道是从哪接收到的思想，觉得 `document.addEventListener()` 的 API 不那么 React，很排斥使用。这样，在实现 dropdown component 时，怎么处理在 menu 以外点击时让 menu 收起来成了一个头疼的问题。

我查了文档，觉得可以用 `onBlur` 这个事件，但为了能够接收到 `onBlur` 事件，menu 内部必须是 input 类型的 component，或者是有 tabIndex 属性，然后加上 tabIndex 后，当 component 处于 onFocus 时，会额外在边框上加上阴影的样式，像下图所示，必须额外再加 css 处理。总之，逻辑变得复杂了。

![]({{site.img_url}}/react-dropdown/outline.png)

后来用 React 做音乐播放器，看别人的实现源码，发现他们都大都使用了 `audioElement.addEventListener('play', handler)` 这种原生 API，而且，有些逻辑如果不用原生事件就没法处理，比如监听 window 的 resize 事件，似乎除了用 `window.addEventListener('resize', handler)` 就没有其它办法了。因此再回过头来看 dropdown 的实现，如果也用 `document.addEventListener('click', handler)` 处理 menu 以后的点击的话，逻辑就简单多了。

但是，也还是有坑的。

坑之一，React 的 `event.stopPropagation()` 无法阻止原生事件冒泡到 document。

看这篇文章的详细介绍：

- [React 合成事件和原生事件的阻止冒泡](http://echizen.github.io/tech/2017/04-02-Reactjquery-event-system)

React 的 issue:

- [e.stopPropagation() seems to not be working as expect.](https://github.com/facebook/react/issues/4335)

React 有两套事件系统，一套是原生事件系统，就是 `document.addEventListener()` 这种 API，另一套是 React 自己定义的，叫 SyntheticEvent (合成事件)，比如下例中的 `onClick`。

    <a onClick={this.clickLink}>Open</a>

实际 React 的所有合成事件都是绑定在 document 上的 (所谓的代理方式)，而不是单独绑在各个 component 上，当你执行合成事件中的 `event.stopPropagation()` 时，实际原生事件已经到达 document 了。

所以 React 的 `event.stopPropagation()` 只能阻止合成事件继续往上冒泡，却不能阻止原生事件往上冒泡到 document。

所以你会发现，为什么我已经在 menu 内部的点击事件 handler 中 stopPropagation 了，为什么全局的 click handler 还是会执行，这就是原因。

但是! React 的合成事件的 stopPropagation 虽然不能阻止事件冒泡到 document，但它可以阻止事件冒泡到 window。

(这件事让我想起，在某个项目中，我用了 React 的 `event.stopPropagation()`，导致 turbolinks 不工作了，当时觉得很理所当然，现在回想，不对，turoblinks 绑定的是原生事件，如果它是绑在 `<a>` tag 上的话，不应该不工作的啊，由此我推断 turbolinks 的 click 事件是绑定在 window 上的，后来看了源码，的确是这样的)

所以，为了在 React 的 dropdown 中实现点击 menu 外部收起 menu，点击内部不收起 menu，有两种办法：

1. 使用 `window.addEventLister('click', handler)` 替代 `document.addEventListener('click', handler)`，同时在 menu 内部点击时，调用合成事件的 `event.stopPropagation()`

1. 不调用 `event.stopPropagation()`，让事件冒泡到 document 的 click handler 中，在 handler 中判断 `event.target` 中在 menu 内部还是外部，使用 `DOMNode.contains()` 方法判断。这种方法需要用 React 的 ref 属性把 menu 的引用保存下来，如下所示：

        <div className="dropdown-body" ref={ref=>this._dropdown_body=ref}>

   判断：

        handleGlobalClick = (event) => {
          console.log('global click')

          // use DOMNode.contains() method to judge click target is in or out of the dropdown body
          if (this._dropdown_body && this._dropdown_body.contains(event.target)) return

          this.setState({dropDownExpanded: false})
          document.removeEventListener('click', this.handleGlobalClick)
        }

坑之二，在原生事件的 handler 中，`this.setState()` 是同步的，不是异步的，让我很惊讶。之前一直以为 `this.setState()` 肯定是异步的。

具体的分析可以看这篇文章 - [你真的理解 setState 吗？](https://juejin.im/post/5b45c57c51882519790c7441)

总结：

> setState 只在合成事件和生命周期函数中是 "异步" 的，在原生事件和 setTimeout 中都是同步的。

但在 twitter 上看 Dan 发推说以后可能会统一成异步操作，拭目以待。

其它细节：

1. 只有在 menu 展开时才注册 document click handler，收起时移除 document click handler，是动态的。

        handleGlobalClick = () => {
          console.log('global click')

          this.setState({dropDownExpanded: false})
          document.removeEventListener('click', this.handleGlobalClick)
        }

1. 为了实现 toggle 的效果，即点击按钮，展开 dropdown menu，再点击按钮，则收到 menu，最简单的办法是，只有在 menu 收起的时候，才给按钮绑定 click handler，menu 展开的时候，按钮没有 click handler，让 document click handler 处理。否则，同时在合成事件的 handler 和原生事件的 handler 中调用 `this.setState()`，一个异步，一个同步，可能会引起麻烦。

        <div className="dropdown-head">
          {
            dropDownExpanded ?
            <button>Collapse dropdown menu - 1</button> :
            <button onClick={this.handleHeadClick}>Open dropdown menu - 1</button>
          }
        </div>

1. 注册 document 的 click handler 时，必须在 setTimeout 回调中执行。

        handleHeadClick = () => {
          console.log('head click')

          this.setState({dropDownExpanded: true})
          setTimeout(()=>{
            // must run in the next tick
            document.addEventListener('click', this.handleGlobalClick)
          }, 0)
        }

1. 在 `componentWillUnmount()` 中要移除 document 的 click handler，以免造成内存泄漏。

        componentWillUnmount() {
          // important! we need remove global click handler when unmout
          document.removeEventListener('click', this.handleGlobalClick)
        }

### Update

自从发现用 `window.addEventListener('click', handler)` 可以很方便地用来实现收起 React 中的 Dropdown 后，我就不亦乐乎的到处用起来了。为了避免写无数遍的 `window.addEventLister('click', handler)`，我封装了一个 NativeClickListener 的 Component，代码没几行，如下所示：

    export default class NativeClickListener extends React.Component {
      static propTypes = {
        onClick: PropTypes.func
      }

      clickHandler = (event) => {
        console.log('NativeClickListener click')
        const { onClick } = this.props
        onClick && onClick(event)
      }

      componentDidMount() {
        window.addEventListener('click', this.clickHandler)
      }

      componentWillUnmount() {
        window.removeEventListener('click', this.clickHandler)
      }

      render() {
        return this.props.children
      }
    }

使用：

    <div className="dropdown-container">
      <div className="dropdown-head">
        <button onClick={this.handleHeadClick}>
          {dropDownExpanded ? 'Collapse' : 'Open'} dropdown menu - 5
        </button>
      </div>
      {
        dropDownExpanded &&
        <NativeClickListener onClick={()=>this.setState({dropDownExpanded: false})}>
          <div className="dropdown-body"
              onClick={this.handleBodyClick}>
              ...
          </div>
        </NativeClickListener>
      }
    </div>

    handleHeadClick = (event) => {
      console.log('head click')
      this.setState(prevState => ({dropDownExpanded: !prevState.dropDownExpanded}))
      event.stopPropagation()
    }
    handleBodyClick = (event) => {
      console.log('body click')
      // just can stop event propagate from document to window
      event.stopPropagation()
    }

后来我想，那其它开源的 React 组件库中的 Dropdown 都是怎么实现的呢，于是探究了一下，果然不出意外，也是用的原生的 addEventListener 实现的，但也有点意外的是，它们并没有用 window.addEventListener，而都是用了 document.addEventListener 和 node.contains 方法实现。

1. [Material Kit React](https://demos.creative-tim.com/material-kit-react/#/)

   这个组件库的 Dropdown 用到了 [@material-ui/core/ClickAwayListener](https://github.com/mui-org/material-ui/blob/master/packages/material-ui/src/ClickAwayListener/ClickAwayListener.js)，来看看它的实现。

        handleClickAway = event => {
          ...
          if (
            doc.documentElement &&
            doc.documentElement.contains(event.target) &&
            !this.node.contains(event.target)
          ) {
            this.props.onClickAway(event);
          }
        }

        render() {
          const { children, mouseEvent, touchEvent, onClickAway, ...other } = this.props;
          const listenerProps = {};
          if (mouseEvent !== false) {
            listenerProps[mouseEvent] = this.handleClickAway;
          }
          if (touchEvent !== false) {
            listenerProps[touchEvent] = this.handleClickAway;
          }

          return (
            <React.Fragment>
              {children}
              <EventListener target="document" {...listenerProps} {...other} />
            </React.Fragment>
          );
        }

    addEventListener 的逻辑看来在 EventListener 中，来自 [react-event-listener](https://github.com/oliviertassinari/react-event-listener/blob/master/src/index.js) 库。而且从 `target="document"` 来看，event 是绑在 document 上的。

        class EventListener extends React.PureComponent {
          componentDidMount() {
            this.applyListeners(on);
          }
          applyListeners(onOrOff, props = this.props) {
            const { target } = props;

            if (target) {
              let element = target;

              if (typeof target === 'string') {
                element = window[target];
              }

              forEachListener(props, onOrOff.bind(null, element));
          }
          ...
        }
        function on(target, eventName, callback, options) {
          // eslint-disable-next-line prefer-spread
          target.addEventListener.apply(target, getEventListenerArgs(eventName, callback, options));
        }
        function off(target, eventName, callback, options) {
          // eslint-disable-next-line prefer-spread
          target.removeEventListener.apply(target, getEventListenerArgs(eventName, callback, options));
        }

1. Ant Design 中的 Dropdown 的实现最终可以追溯到 [react-component/trigger](https://github.com/react-component/trigger/blob/master/src/index.js) 组件。

        // We must listen to `mousedown` or `touchstart`, edge case:
        // https://github.com/ant-design/ant-design/issues/5804
        // https://github.com/react-component/calendar/issues/250
        // https://github.com/react-component/trigger/issues/50
        if (state.popupVisible) {
          let currentDocument;
          if (!this.clickOutsideHandler && (this.isClickToHide() || this.isContextMenuToShow())) {
            currentDocument = props.getDocument();
            this.clickOutsideHandler = addEventListener(currentDocument,
              'mousedown', this.onDocumentClick);
          }
          // always hide on mobile
          if (!this.touchOutsideHandler) {
            currentDocument = currentDocument || props.getDocument();
            this.touchOutsideHandler = addEventListener(currentDocument,
              'touchstart', this.onDocumentClick);
          }
          // close popup when trigger type contains 'onContextMenu' and document is scrolling.
          if (!this.contextMenuOutsideHandler1 && this.isContextMenuToShow()) {
            currentDocument = currentDocument || props.getDocument();
            this.contextMenuOutsideHandler1 = addEventListener(currentDocument,
              'scroll', this.onContextMenuClose);
          }
          // close popup when trigger type contains 'onContextMenu' and window is blur.
          if (!this.contextMenuOutsideHandler2 && this.isContextMenuToShow()) {
            this.contextMenuOutsideHandler2 = addEventListener(window,
              'blur', this.onContextMenuClose);
          }
          return;
        }

        onDocumentClick = (event) => {
          if (this.props.mask && !this.props.maskClosable) {
            return;
          }

          const target = event.target;
          const root = findDOMNode(this);
          if (!contains(root, target) && !this.hasPopupMouseDown) {
            this.close();
          }
        }

1. JetBrain 的 ring-ui 的 [Dropdown](https://jetbrains.github.io/ring-ui/master/dropdown.html) 并没有实现在其它地方点击后让 Dropdown 收起的功能，有点意外...

一开始不是很理解，不过后来我发现，如果用 `window.addEventListener('click', handler)` 的方式收起 Dropdown，在一个页面中，如果有多个 Dropdown，我先展开一个 Dropdown menu (称之为 A)，再点击另一个 Dropdown (称之为 B)，因为在 Dropdown B 的点击事件中调用了 `event.stopPropagation()`，因此 Dropdown A 的 global click handler 将无法触发，因此 Dropdown A 无法收起。

![]({{site.img_url}}/react-dropdown/multiple-dropdown.gif)

即使只有一个 Dropdown，如果页面中有其它任意地方的 event handler 中调用了 `event.stopPropagation()` 都会导致此 Dropdown 有可能无法收起。

但是用 `document.addEventListener('click', handler)` 配合 `node.contains()` 方法却不会有这个问题，因此恍然大悟，终于明白了为什么那些开源组件库并没有采用 `window.addEventListener()` 的方式。

于是实现 NativeClickListener2：

    export default class NativeClickListener extends React.Component {
      static propTypes = {
        onClick: PropTypes.func
      }

      clickHandler = (event) => {
        console.log('NativeClickListener click')
        if(this._container.contains(event.target)) return

        const { onClick } = this.props
        onClick && onClick(event)
      }

      componentDidMount() {
        document.addEventListener('click', this.clickHandler)
      }

      componentWillUnmount() {
        document.removeEventListener('click', this.clickHandler)
      }

      render() {
        return (
          <div ref={ref=>this._container=ref}>
            {this.props.children}
          </div>
        )
      }
    }

使用：

    <div className="dropdown-container">
      <div className="dropdown-head">
        <button onClick={this.handleHeadClick}>
          {dropDownExpanded ? 'Collapse' : 'Open'} dropdown menu - 5
        </button>
      </div>
      {
        dropDownExpanded &&
        <NativeClickListener2 onClick={()=>this.setState({dropDownExpanded: false})}>
          <div className="dropdown-body"
              onClick={this.handleBodyClick}>
              ...
          </div>
        </NativeClickListener2>
      }
    </div>

    handleHeadClick = (event) => {
      console.log('head click')
      this.setState(prevState => ({dropDownExpanded: !prevState.dropDownExpanded}))
      // no need
      // event.stopPropagation()
    }
    handleBodyClick = (event) => {
      console.log('body click')
      // no need
      // event.stopPropagation()
    }

![]({{site.img_url}}/react-dropdown/native-click-listener-2.gif)
