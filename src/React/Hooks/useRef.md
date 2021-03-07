[useRef 官网](https://reactjs.org/docs/hooks-reference.html#useref) 的定义如下:

useRef returns a mutable ref object whose .current property is initialized to the passed argument (initialValue). The returned object will persist for the full lifetime of the component.

whats the difference between useref and createref?

- useRef 仅能用在 FunctionComponent；
- createRef 仅能用在 ClassComponent，因为 createRef 并没有 Hooks 的效果，其值会随着 FunctionComponent 重复执行而不断被初始化；
- createRef 每次渲染都会返回一个新的引用，而 useRef 每次都会返回相同的引用。

知道了两者的区别，举个例子来运用一下：

```js
function App() {
  const [renderIndex, setRenderIndex] = useState(1);
  const refFromUseRef = useRef();
  const refFromCreateRef = createRef();
  if (!refFromUseRef.current) {
    refFromUseRef.current = renderIndex;
  }
  if (!refFromCreateRef.current) {
    refFromCreateRef.current = renderIndex;
  }
  return (
    <div className="App">
      Current render index: {renderIndex}
      <br />
      First render index remembered within refFromUseRef.current:
      {refFromUseRef.current}
      <br />
      First render index unsuccessfully remembered within
      refFromCreateRef.current:
      {refFromCreateRef.current}
      <br />
      <button onClick={() => setRenderIndex(prev => prev + 1)}>
        Cause re-render
      </button>
    </div>
  );
}
```

例子中，refFromUseRef.current 会随着每次渲染加 1，而 refFromCreateRef.current 不会。
可以在 [codeSandbox](https://codesandbox.io/s/1rvwnj71x3) 实践一下。

如何使用 useRef？

看下面的例子：
```js
function App() {
  const[count, setCount] =useState(0);
  const handleAlertClick = () => {
    setTimeout(() => {
      console.log("You clicked on：" + count);
    }, 3000);
  }

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count+1)}>Click me</button>
      <button onClick={handleAlertClick}>Show alert</button>
    </div>
  )
}
```
打印日志的 count 和 <p> 标签里边展示的是一样的吗？

答案如下：
![test-1](../../../images/hooks/useRef-1.gif)

每次点击，获取到新的 count 状态，React 重新渲染组件，handleAlertClick 使用的 count 是当下渲染是值。

使用 useRef 来实现打印的 count 与 <p> 标签里边渲染的一样：

```js
function App() {
  const [count, setCount] = useState(0);
  const latestCount = useRef(count);
    
  useEffect(() => {
    latestCount.current=count;
  });
    
  const handleAlertClick = () => {  
    setTimeout(() => {
        alert("You clicked on："+ latestCount.current);
    }, 3000);
  }
  
  return(
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count+1)}>Click me</button>
      <button onClick={handleAlertClick}>Show alert</button>
    </div>
  );
}
```
![test-2](../../../images/hooks/useRef-2.gif)

因为 useRef 每次渲染都返回同一个引用，所以在 useEffect 中修改后，日志打印的也就一样了。

### Reference
- [精读《useRef 与 createRef 的区别》](https://zhuanlan.zhihu.com/p/110247813)
- [What's the difference between `useRef` and `createRef`?](https://stackoverflow.com/questions/54620698/whats-the-difference-between-useref-and-createref)