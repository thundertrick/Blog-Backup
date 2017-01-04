---
title: 4种 Java 多参数构建模式
date: 2017-01-03 14:37:40
tags: [java, Android]
---
最近在实际项目中遇到了多参数初始化问题。如何在模块初始化时传入多个参数（5个以上）？如何更安全？如何更灵活？如何可扩展？如何简化调用代码？如何增强可读性？我在面试中也常提出这样的问题。以上内容如果读过《Efective Jave》会很容易在第二章找到答案。

《Efective Jave》中主要介绍了 3 种多参数构建方法，我在项目里还用到了第 4 种。下面一一介绍。

<!-- more -->

##0x00 简单、常用：重叠构造器
代码示例：
~~~java
public class A {
    private int v1;
    private int v2;
    private int v3;
    private int v4;
    private int v5;
    
    public A(int v1) { this(v1, DEFUALT_V2)}
    public A(int v1, int v2) { this(v1, v2, DEFUALT_V3)}
    public A(int v1, int v2, int v3) { this(v1, v2, v3, DEFUALT_V4)}
    public A(int v1, int v2, int v3, int v4) { this(v1, v2, v3, v4, DEFUALT_V5)}
    public A(int v1, int v2, int v3, int v4, int v5) {
        this.v1 = v1;
        this.v2 = v2;
        this.v3 = v3;
        this.v4 = v4;
        this.v5 = v5;
    }
}
~~~
优点：
1. 逻辑、层级清晰明了。适合参数层级关系明确的类，如 Android 里的 View。

缺点：
1. 参数增多时，可读性变差
2. 如示例所示，如果连续传入多个同样类型的变量，容易把顺序弄错
3. 灵活性不足，无法指定初始化特定几个参数

##0x01 简单、灵活：重复 set 方法
代码示例：
~~~java
public class B {
    private int v1 = DEFAULT_V1;
    private int v2 = DEFAULT_V2;
    private int v3 = DEFAULT_V3;
    private int v4 = DEFAULT_V4;
    private int v5 = DEFAULT_V5;
    
    public B() {}
    public void setV1(int val) { v1 = val; }
    public void setV2(int val) { v2 = val; }
    public void setV3(int val) { v3 = val; }
    public void setV4(int val) { v4 = val; }
    public void setV5(int val) { v5 = val; }
}
~~~
优点:
1. 简单
2. 灵活，支持定制式初始化，即参数可选，如果不需要自定义的变量可以直接不给初始化方法
3. 支持动态改变参数，set 方法也可以在程序运行的任何阶段调用

缺点：
1. 如果变量的需要按一定顺序初始化，那么这种调用方式将带来风险。实际项目被这个坑过。提供外部的接口最好在编写时排除这种漏洞的隐患
2. set 方法可在任何时间调用，更改变量的值。和缺点 1 同理，如果变量耦合了其他逻辑，那么随时可调用 set 方法就是不安全的。

## 0x02 安全、可读：Builder 构建器
代码示例
~~~java
public class C {
    private int v1;
    private int v2;
    private int v3;
    private int v4;
    private int v5;
    
    public static class Builder {
        private int v1 = DEFAULT_V1;
        private int v2 = DEFAULT_V2;
        private int v3 = DEFAULT_V3;
        private int v4 = DEFAULT_V4;
        private int v5 = DEFAULT_V5;

        public Builder(int v1) { this.v1 = v1; return this;}
        public Builder v2(int val) { this.v2 = val; return this; }
        public Builder v3(int val) { this.v3 = val; return this; }
        public Builder v4(int val) { this.v4 = val; return this; }
        public Builder v5(int val) { this.v5 = val; return this; }
        public C build() { return new C(this); }
    }

    private C(Builder builder) {
        this.v1 = builder.v1;
        this.v2 = builder.v2;
        this.v3 = builder.v3;
        this.v4 = builder.v4;
        this.v5 = builder.v5;
    }
}
// Usage:
// C c = new C.Builder(1).v2(2).v3(3).v4(4).v5(5).build();
~~~
优点：
1. 灵活，支持定制式初始化
2. 可读性强，参见 Android 的 Animator 类，一个语句完成类的初始化和调用
3. 可以在 `build()` 方法内统一进行参数校验，为《Ecfective Java》推荐方法

缺点：
1. 复杂，构建所需代码行数增加
2. 参数在传入时为确定参数，不支持动态改变

## 0x03 实时参数：Provider 模式
该方法借鉴了 `Builder` 构建器，但是将构建器独立于类外，同时提供实时获取的接口。
~~~java
public class D {
    private int v1;
    private int v2;
    private int v3;
    private int v4;
    private int v5;
    
    public interface Provider {
        public int getV1();
        public int getV2();
        public int getV3();
        public int getV4();
        public int getV5();
    }

    public D(Provider provider) {
        this.v1 = provider.getV1();
        this.v2 = builder.v2;
        this.v3 = builder.v3;
        this.v4 = builder.v4;
        this.v5 = builder.v5;
    }
}
// Usage:
// public class IProvider implement D.Provider {
//        @Override
//        public int getV1() { return val; }
//        @Override
//        public int getV2() { return val; }
//        @Override
//        public int getV3() { return val; }
//        @Override
//        public int getV4() { return val; }
//        @Override
//        public int getV5() { return val; } 
// }
// D d = new D(new IProvider());
~~~
优点：
1. 借鉴了 `Builder` 构建器的优点，隔离了参数传入方法和初始化方法
2. `Provider` 由调用方实现，使动态参数的传入变为可能
3. 调用节点只需要调用一行语句，初始化参数的准备移至他处。这种场景需求可能发生在一个核心类的 `initAll()` 方法中，调用者需要在该方法内先后初始化十几个模块。如果每个模块都要写十行代码那就非常恐怖了。
