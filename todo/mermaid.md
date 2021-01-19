AOP包含以下几个分支，注意：spring中的动态代理aop完全没有使用到aspectj，只是语法构成上是一致的



```mermaid
graph LR
A(AOP) -->| |B(静态代理)
B --> | |F(aspectj)
A --> | |C(动态代理)
C --> | |D(jdk动态代理 基于接口)
C --> | |E(cglib)
```

