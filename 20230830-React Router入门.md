# 20230830-React Router入门

## ScrollRestoration 组件

`ScrollRestoration` 是 React Router v6 中的一个组件，用于在路由切换时恢复滚动位置。它可以在路由切换时自动保存当前滚动位置，并在下一次访问该路由时恢复滚动位置。

要使用 `ScrollRestoration` 组件，需要在 `Router` 组件中将其作为子组件，并设置 `type` 属性为 `'scroll'`。例如：

```jsx
import { ScrollRestoration } from "react-router-dom";

function RootRouteComponent() {
  return (
    <div>
      {/* ... */}
      <ScrollRestoration />
    </div>
  );
}
```

### getKey 

定义 React 路由器应该用来恢复滚动位置的键的可选属性。

```jsx
<ScrollRestoration
  getKey={(location, matches) => {
    // default behavior
    return location.key;
  }}
/>
```

默认情况下，它使用 `location.key` ，模拟浏览器的默认行为，而无需客户端路由。用户可以在堆栈中多次导航到同一 URL，每个条目都有自己的滚动位置进行还原。

## Outlet 组件

`Outlet` 是 React Router v6 中的一个组件，用于渲染当前路由匹配的组件。它通常用于在父组件中渲染子组件，例如在 `App` 组件中渲染不同的页面组件。

在 React Router v6 中，路由的匹配规则发生了变化，不再使用 `<Route>` 组件来定义路由，而是使用 `useRoutes` 钩子来定义路由。`Outlet` 组件则用于在父组件中渲染子组件，它会根据当前路由匹配的结果来渲染对应的组件。

例如，下面是一个使用 `Outlet` 组件的示例：

```jsx
import { useRoutes, Outlet } from 'react-router-dom';

function App() {
  const routes = useRoutes([
    { path: '/', element: <HomePage /> },
    { path: '/about', element: <AboutPage /> },
    { path: '/contact', element: <ContactPage /> },
  ]);

  return (
    <div>
      <header>
        <nav>
          <ul>
            <li>
              <Link to="/">Home</Link>
            </li>
            <li>
              <Link to="/about">About</Link>
            </li>
            <li>
              <Link to="/contact">Contact</Link>
            </li>
          </ul>
        </nav>
      </header>
      <main>
        <Outlet />
      </main>
    </div>
  );
}
```

在这个示例中，我们使用 `useRoutes` 钩子定义了三个路由，分别对应 `/`、`/about` 和 `/contact` 路径。然后在 `App` 组件中，使用 `Outlet` 组件来渲染当前路由匹配的组件。这样，当用户访问不同的路径时，`Outlet` 组件会自动渲染对应的页面组件。

> 相当于vue-router中的router-view，用来渲染视图的。