# Webpack DllPlugin

dll(Dynamic-link library 动态链接库) 其实就是缓存.

具体到 webpack 这块儿，就是事先把常用但又构建时间长的代码提前打包好（例如 react、react-dom），取个名字叫 dll。后面再打包的时候就跳过原来的未打包代码，直接用 dll。这样一来，构建时间就会缩短，提高 webpack 打包速度。

干的活就是空间换时间。

## Reference

- [动态链接库(DLL)](https://zh.wikipedia.org/wiki/%E5%8A%A8%E6%80%81%E9%93%BE%E6%8E%A5%E5%BA%93)
- [你真的需要 Webpack DllPlugin 吗？](https://blog.csdn.net/xiaoyaGrace/article/details/106328441)
