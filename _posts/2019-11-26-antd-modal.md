# Ant Design - Modal组件分析

## 组件调用形式

1. [组件形式调用](https://ant.design/components/modal-cn/#components-modal-demo-basic)

```javascript
import { Modal, Button } from 'antd';

class App extends React.Component {
  state = { visible: false };

  showModal = () => {
    this.setState({
      visible: true,
    });
  };

  handleOk = e => {
    console.log(e);
    this.setState({
      visible: false,
    });
  };

  handleCancel = e => {
    console.log(e);
    this.setState({
      visible: false,
    });
  };

  render() {
    return (
      <div>
        <Button type="primary" onClick={this.showModal}>
          Open Modal
        </Button>
        <Modal
          title="Basic Modal"
          visible={this.state.visible}
          onOk={this.handleOk}
          onCancel={this.handleCancel}
        >
          <p>Some contents...</p>
          <p>Some contents...</p>
          <p>Some contents...</p>
        </Modal>
      </div>
    );
  }
}

ReactDOM.render(<App />, mountNode);

```


2. [方法形式调用](https://ant.design/components/modal-cn/#components-modal-demo-confirm)

```javascript
import { Modal, Button } from 'antd';

const { confirm } = Modal;

function showConfirm() {
  confirm({
    title: 'Do you Want to delete these items?',
    content: 'Some descriptions',
    onOk() {
      console.log('OK');
    },
    onCancel() {
      console.log('Cancel');
    },
  });
}

function showDeleteConfirm() {
  confirm({
    title: 'Are you sure delete this task?',
    content: 'Some descriptions',
    okText: 'Yes',
    okType: 'danger',
    cancelText: 'No',
    onOk() {
      console.log('OK');
    },
    onCancel() {
      console.log('Cancel');
    },
  });
}

function showPropsConfirm() {
  confirm({
    title: 'Are you sure delete this task?',
    content: 'Some descriptions',
    okText: 'Yes',
    okType: 'danger',
    okButtonProps: {
      disabled: true,
    },
    cancelText: 'No',
    onOk() {
      console.log('OK');
    },
    onCancel() {
      console.log('Cancel');
    },
  });
}

ReactDOM.render(
  <div>
    <Button onClick={showConfirm}>Confirm</Button>
    <Button onClick={showDeleteConfirm} type="dashed">
      Delete
    </Button>
    <Button onClick={showPropsConfirm} type="dashed">
      With extra props
    </Button>
  </div>,
  mountNode,
);
```

## 组件主要实现功能

1. 按需加载组件，在未点击组件时不产生对话框dom，产生后一直使用同一个dom
2. 组件可以在非root节点产生
3. 动画功能


## 源码分析

### 按需加载的实现
进入antd的modal源码，可以发现整个Modal使用了rc-dialog库，进入[rc-dialog](https://github.com/react-component/dialog)后可以找到源码实现。

![源码目录](http://static.chryseis.cn/%E7%9B%AE%E5%BD%95.jpg)

按需加载的实现在[DialogWrap.tsx](https://github.com/react-component/dialog/blob/master/src/DialogWrap.tsx)中

```javascript

	<Portal
      visible={visible}
      forceRender={forceRender}
      getContainer={getContainer}
    >
      {(childProps: IDialogChildProps) => (
        <Dialog
          {...props}
          {...childProps}
        />
      )}
    </Portal>
```
整个按需的过程通过Portal组件实现。而Portal组件在rc-util中的[PortalWrapper.js](https://github.com/react-component/util/blob/master/src/PortalWrapper.js)，我们由此可以看到实现

```javascript
    if (!IS_REACT_16) {
      return (
        <ContainerRender
          parent={this}
          visible={visible}
          autoDestroy={false}
          getComponent={(extra = {}) => children(
            {
              ...extra,
              ...childProps,
              ref: this.savePortal,
            }
          )}
          getContainer={this.getContainer}
          forceRender={forceRender}
        >
          {({ renderComponent, removeContainer }) => {
            this.renderComponent = renderComponent;
            this.removeContainer = removeContainer;
            return null;
          }}
        </ContainerRender>
      );
    }
    if (forceRender || visible || this._component) {
      portal = (
        <Portal getContainer={this.getContainer} ref={this.savePortal} >
          {children(childProps)}
        </Portal>
      );
    }
    return portal;
  }

```

整个按需的实现按react版本分为两部分：

1. [react 15](https://github.com/react-component/util/blob/master/src/ContainerRender.js)
2. react 16

由于react 15未提供[createPortal](https://zh-hans.reactjs.org/docs/portals.html)方法故只能使用ReactDOM.unstable_renderSubtreeIntoContainer方法

```javascript
  renderComponent = (props, ready) => {
    const { visible, getComponent, forceRender, getContainer, parent } = this.props;
    if (visible || parent._component || forceRender) {
      if (!this.container) {
        this.container = getContainer();
      }
      ReactDOM.unstable_renderSubtreeIntoContainer(
        parent,
        getComponent(props),
        this.container,
        function callback() {
          if (ready) {
            ready.call(this);
          }
        }
      );
    }
  }

```

react16 可以方便的使用[Portal](https://github.com/react-component/util/blob/master/src/Portal.js)

```javascript
 render() {
    if (this._container) {
      return ReactDOM.createPortal(this.props.children, this._container);
    }
    return null;
  }

```

通过上述方法便可以把Modal按需进行加载。

我按antd的思路简单实现了一个按需加载Portal的高阶组件

```javascript
import React from 'react'
import ReactDOM from 'react-dom'

const createPortals = (DecoratorsComponent) => {
    return class Portals extends React.Component {
        static defaultProps = {
            visible: false
        }

        createContainer() {
            if (!this.component) {
                this.dom = document.createElement('div')
                document.body.appendChild(this.dom)
            }
        }

        componentWillUnmount() {
            this.dom && document.body.removeChild(this.dom)
        }

        render() {
            const { visible } = this.props
            if (visible || this.component) {
                this.createContainer()
                return ReactDOM.createPortal(<DecoratorsComponent
                    ref={ref => this.component = ref} {...this.props}/>, this.dom)
            }
            return null
        }
    }
}

export default createPortals

```

那么我们在使用按需加载的时，只需要给组件加一个**@createPortals**便可以实现按需加载的目的





