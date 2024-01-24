# React Router 实现原理

December 11, 2023

**Anonymous**

react-router 的实现原理（为何改变了页面地址却没有向服务端发起请求），请提供一个示例


**ChatGPT**

React Router 是一个用于构建单页面应用的库，它利用了 HTML5 History API 中的 `pushState` 和 `replaceState` 方法来管理 URL，并通过监听 URL 变化来渲染不同的组件，而不会向服务器发送请求。

其实现原理可以概括为：

1. **History API**: React Router使用HTML5 History API中的`pushState`和`replaceState`方法来在不刷新页面的情况下改变浏览器地址栏的URL，并将对应的路由信息保存到浏览器的历史记录中。
    
2. **监听URL变化**: React Router监听URL的变化，当URL发生改变时，它会匹配定义的路由规则，找到与当前URL匹配的路由，并渲染相应的组件。
    
3. **渲染对应组件**: 一旦匹配到对应的路由，React Router会渲染与该路由关联的组件，而不会向服务器发送请求，因为所有路由信息已经在前端定义好了。
    

示例：
```js
// 引入 React 相关模块和 React Router
import React from 'react';
import { BrowserRouter as Router, Route, Link } from 'react-router-dom';

// 定义两个组件
const Home = () => <h1>Home</h1>;
const About = () => <h1>About</h1>;

// 创建一个使用 React Router 的组件
const App = () => {
  return (
    <Router>
      <div>
        {/* 创建导航链接 */}
        <nav>
          <ul>
            <li>
              {/* 导航链接 */}
              <Link to="/">Home</Link>
            </li>
            <li>
              <Link to="/about">About</Link>
            </li>
          </ul>
        </nav>
        
        {/* 定义路由对应的组件 */}
        <Route path="/" exact component={Home} />
        <Route path="/about" component={About} />
      </div>
    </Router>
  );
};

export default App;
```


在这个示例中，当用户点击导航链接时，URL会改变但页面不会刷新，React Router会根据URL的变化渲染对应的组件（`Home` 或 `About`），这些组件是在前端定义好的，不需要向服务器发送请求获取页面内容。

**Anonymous**

怎样手动实现一个 react router``

**ChatGPT**

手动实现一个简单版本的 React 路由可以通过监听 URL 变化，并根据匹配的路由渲染相应的组件。这里我会提供一个非常基础的例子，用于演示这种实现方式：


```js
import React, { useState, useEffect } from 'react';

const Router = ({ routes }) => {
  const [currentRoute, setCurrentRoute] = useState(window.location.pathname);

  useEffect(() => {
    const handleRouteChange = () => {
      setCurrentRoute(window.location.pathname);
    };

    // 添加监听器，监听URL变化
    window.addEventListener('popstate', handleRouteChange);

    return () => {
      // 移除监听器
      window.removeEventListener('popstate', handleRouteChange);
    };
  }, []);

  const Component = routes.find(route => route.path === currentRoute)?.component;

  return Component ? <Component /> : null;
};

const Home = () => <h1>Home</h1>;
const About = () => <h1>About</h1>;

const App = () => {
  const routes = [
    { path: '/', component: Home },
    { path: '/about', component: About },
  ];

  return (
    <div>
      {/* 导航链接 */}
      <nav>
        <ul>
          <li>
            <a href="/">Home</a>
          </li>
          <li>
            <a href="/about">About</a>
          </li>
        </ul>
      </nav>

      {/* 路由组件 */}
      <Router routes={routes} />
    </div>
  );
};

export default App;
```

这个例子创建了一个非常简化的手动路由实现。`Router` 组件监听 `popstate` 事件（即浏览器的前进后退按钮或通过 JavaScript 调用 `history.pushState()` 和 `history.replaceState()` 方法）来检测 URL 变化，根据当前 URL 找到对应的路由，并渲染相应的组件。

注意，这个例子非常基础，实际的 React Router 还有许多功能和优化，比如嵌套路由、路由参数、重定向等。但这个简化版本可以让你更好地理解 React 路由的基本原理。