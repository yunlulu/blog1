# H5 - WebApp 路由机制
> 使用react-router的路由设计。
> 
> 针对单页WebApp的路由机制设计(hash-history)，使路由的声明更加友好，路由与页面组件一一对应。
> 
> hash-history可以利用浏览器的history和pushState特性。

## 路由声明
react-router是声明式路由，声明形式如下代码：

    <Router history={hashHistory}>
        <Route path="/" component={Index}>
            <IndexRoute component={Feeds} />
            <Route path="plans" component={Plans} />
            <Route path="approves" component={Approves} />
        </Route>
        <Route path="feeds/:id" component={Feed} />
        <Route path="feeds/:id/detail" component={Detail} onEnter={requireAuth} />
        <Redirect from="old/:id" to="new/:id" />
    </Router>

    // 权限认证拦截
    function requireAuth(nextState, replace) {
        if (!auth.loggedIn()) {
            replace({
                pathname: '/login',
                state: { nextPathname: nextState.location.pathname }
            });
        }
    }

1. Router - 路由规则器
    * history - 历史记录类型，这里指定为`hashHistory`，另外还有`browserHistory`等
2. Route - 一条路由规则
    * path - 定义路由路径，首页则指定为"/"即可
    * component - 该路由规则对应的React组件(页面)
    * components - 该路由规则对应的多个React组件
    * onEnter - 进入该路由页面之前的拦截hook，一般用来做权限拦截
    * onLeave - 离开该路由页面之前的拦截hook
3. IndexRoute - 给上级路由指定默认的下级路由，不必指定`path`属性
4. Redirect - 定义重定向关系，重定向废弃路由规则到新的有效路由
    * from - 废弃路由
    * to - 新的有效路由

## 路由与React组件的关系

    const Index = React.createClass({
        render: function() {
            return (
                <div className="demo-view">
                    <TitleBar title="纷享销客 - 首页" />
                    <div className="demo-body">
                        <Nav data={require('data/nav.json')} />
                        <div className="demo-content">
                            {this.props.children}
                        </div>
                        <footer><img src={require('assets/images/logo.png')} /></footer>
                    </div>
                </div>
            );
        }
    });

路由与React组件是一一对应的关系，正因此，react-router被设计成声明式路由机制。

上面例子中：

`/`的Route对应的组件是`Index`，这个路由下定义的所有下级路由，都会被渲染到`<div className="demo-content">`所指定的`{this.props.children}`位置。

__这个就是路由嵌套！__

与`/`路由平级的路由页面会覆盖`Index`页面。

关于`components`的情况，用下面代码说明：

    // 路由规则声明
    <Route path="abc" components={{sidebar:SideBar, content: Content}} />

    // Abc页面的UI结构
    // props上的sidebar和content属性分别对应于route components定义的SideBar和Content
    const Abc = React.createClass({
        render() {
            return (
                <div>
                    <aside>
                        {this.props.sidebar}
                    </aside>
                    <article>
                        {this.pros.content}
                    </article>
                </div>
            );
        }
    });

## 路由规则的使用

    let Link = require('react-router').Link;
    const nav = React.createClass({
        render: function() {
            return (
                <Link to="/">首页</Link>
                <Link to="/plans">Plans</Link>
                <Link to="/approves">Approves</Link>
                <ul>
                    <li><Link to={{pathname: '/feeds/1', query: {title: 'Feed 1'}}}>Feed 1</Link></li>
                </ul>
            );
        }
    });

1. 使用`Link`而不是`a`标签，最终`Link`还是渲染为真实`a`标签

2. `Link`的`to`属性其实就等于是`a`标签的`href`，注意：`to`的链接url不用带`#`，就指定为`pathname`即可

3. 可以用字面量对象定义除`pathname`以外的`query`、`state`等属性

例如`to={{pathname: '/feeds/1', query: {title: 'Feed 1'}}}`的最终结果是：`href="#/feeds/1?title=Feed 1`。

## 路由的params
路由匹配规则如下：

    <Route path="/hello/:name">         // matches /hello/michael and /hello/ryan
    <Route path="/hello(/:name)">       // matches /hello, /hello/michael, and /hello/ryan
    <Route path="/files/*.*">           // matches /files/hello.jpg and /files/hello.html
    <Route path="/**/*.jpg">            // matches /files/hello.jpg and /files/path/to/file.jpg

其中`/hello/:name`规则，`:name`部分即是param(params = { name: 'xxx' } )。可以有多个param，如`/hello/:name/:id`。

可在React组件内通过`this.props.params`调用params键值对。

## 路由的query
通过url传递的querystring部分，在使用链接处传递：

    <Link to={{pathname: '/feeds/1', query: {title: 'Feed 1'}}}>Feed 1</Link>

跳转链接后的url是：

    #/feeds/1?title=Feed 1

`?`后面部分就是querystring。

可在React组件内通过`this.props.location.query`调用query键值对（注意：query是已被解析为键值对，如{ title: "Feed 1" }）。

## 路由的state
通过 `state`属性可以传递复杂数据到目标路由内，实际上调用的是浏览器的`pushState`或`replaceState`传递的第一个参数`state`。

在使用链接处传递：

    <Link to={{pathname: '/feeds/1', state: {name: 'abc', id: 1111, value: '1235'}}}>Feed 1</Link>

可在目标路由对应的React组件内通过`this.props.location.state`调用`state`对象。

这个属性可以用在路由跳转时携带数据和路由返场时回传数据，用来在多页面间本地传递数据，而不用从服务器频繁获取。

## 如何手动调用路由跳转？
有一种场景需求就是在做某个动作之后，需要通过js脚本手动跳转到别的路由，这个时候的做法是：

    // 在组件内定义router上下文
    contextTypes: {
        router: React.PropTypes.object.isRequired
    }

    // 然后调用router上下文
    this.context.router.goBack();

`router`上下文其实就是一个react-router封装的`history`实例，其中包含的方法有：
* go - 跳转到指定历史记录页，正数等于goForward，负数等于goBack
* goBack - 回到上一页
* goForward - 回到下一页
* push - 转到指定页，产生一条新的历史记录，push('/abc')
* replace - 转到指定页，覆盖当前历史记录，不产生新的历史记录，replace('/abc')
* pushState - 类似于push，但可传递`state`
* replaceState - 类似于replace，但可传递`state`
* isActive - 检测指定页是否是当前活动页，isActive('/abc')

## 动态路由
动态路由的意思是指，起初并不明确路由规则对应的React组件是什么，这个时候就需要先定义路由规则，并指定未来有可能返回的React component。

    const CourseRoute = {
      path: 'course/:courseId',
      getComponent(location, callback) {
        require.ensure([], function (require) {
          callback(null, require('./components/Course'))
        })
      }
    }

比如这个例子中，`CourseRoute`的`component`是异步模块，在声明路由规则的时候并不存在（未引入），这种情况，使用XML形式声明路由就做不到了，因此react-router提供了这种动态声明路由的方式。
