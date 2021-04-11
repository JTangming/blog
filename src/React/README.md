# todo

React，先从 ReactDOM.render开始看，里面的涉及兼容性， hydrate，Suspense，Hooks 的就可以不看，先整个走一遍，这样 mount 的过程就基本完成了，然后就是 update，自然从 setState开始看，涉及到事件/表单的可以不看，这样可以把整个 update 的流程基本搞懂，会发现哦，原来 mount 和 update 基本没有区别，然后再细化，比如更新队列/batch 是怎么工作的，（简单的）Concurrent 是怎么工作的，Reconciler 是怎么工作的，Scheduler 是怎么工作的，Hooks 是怎么工作的，到了细化的时候，就会越来越发现底层的设计的重要性和关联性，如 fiber 里各字段的含义和用途，各种 Flags 的用途，各种 WorkType，FiberType，EffectType 的含义和用途，各种 Dev 下的 mark，warning 触发的时机，这些又会反过来帮助去理解各个功能，甚至整个体系如生命周期，UI/UX 的思考等等。


比如 React 的体积很大，就不能一直停留在知道他很大的概念上，可以做的还有很多，比如大到底大多少，20k 还是 50k，是都这么大，还是 Production 和 Dev 不同，不同的部分到底是什么引起的，比如一般 development 版会大一些，但是为什么 React 的 development 版会大好几倍，再深入去了解，就会发现，一方面不仅多了很多功能（JIT 优化，user timing，strict mode，track，profiler，fast refresh），另一方面对于 development 下开发的体验的提升（error stack 信息，更良好的开发者可能犯的一些错误/anti pattern 的校验提醒，未来的 breaking change 的提前告知，底层数据结构的变化，启发式算法的策略，钩子单词拼错都有考虑）
