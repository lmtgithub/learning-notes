1. `@Builder`不能直接使用字段默认值，需要结合`@Default`一起使用。详情请见[@Builder编译代码详解](@Builder编译代码详解.md)
2. `@Builder`会生成一个全参构造函数，所以不能直接使用new构造对象。需要另一个注解：`@NoArgsConstructor`
3. `@Builder`在继承关系上使用不是很顺畅。（todo：详细分析）