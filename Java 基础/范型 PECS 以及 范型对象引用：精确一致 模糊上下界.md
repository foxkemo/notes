这是一个非常精准的切入点。简单来说：**不一定完全一致，但必须“兼容”**。

这种兼容性取决于你使用的是**精确类型引用**，还是**通配符引用**。我们将这个关系拆解为以下三种场景：

---

## 1. 精确类型引用：必须严格一致

如果你声明的引用没有 `?`，那么赋值号左边的 `T` 必须与右边 `new` 出来的 `T` **完全相同**。

Java

```
List<Number> list = new ArrayList<Integer>(); // 编译报错！
```

- **为什么？**：虽然 `Integer` 是 `Number` 的子类，但 `List<Integer>` 并不是 `List<Number>` 的子类。这叫泛型的**不协变性（Invariance）**。
    
- **风险点**：如果允许这样赋值，你可以通过 `list.add(1.1)` 往一个本应全是整数的 `ArrayList<Integer>` 里存入 `Double`，这会破坏类型安全。
    

---

## 2. 上界通配符引用：`new` 的对象必须是 `T` 或其子类

当你使用 `<? extends T>` 时，左边的 `T` 是右边的**上限**。

Java

```
List<? extends Number> list = new ArrayList<Integer>(); // 合法
```

- **关系**：右边的 `Integer` 只要在 `Number` 的继承树下方（包括 `Number` 本身），就是合法的。
    
- **代价**：此时你失去了往 `list` 写入数据的权利（除了 `null`），因为编译器无法确定右边到底 `new` 的是 `Integer`还是 `Float`。
    

---

## 3. 下界通配符引用：`new` 的对象必须是 `T` 或其父类

当你使用 `<? super T>` 时，左边的 `T` 是右边的**下限**。

Java

```
List<? super Integer> list = new ArrayList<Number>(); // 合法
```

- **关系**：右边的 `Number` 必须是 `Integer` 的父类（包括 `Object` 和 `Integer` 本身）。
    
- **逻辑**：这种引用保证了“我这个容器至少能装下 `Integer`”。
    

---

## 4. 深度总结：引用类型 vs 对象类型

我们可以把这个关系看作是**“合同”与“工厂实体”**的关系：

|**引用（合同）声明**|**右侧 new 的对象要求**|**核心限制原因**|
|---|---|---|
|**`Base<T>`**|必须是 **`new Base<T>`**|为了保证双向（读/写）的绝对安全，必须严丝合缝。|
|**`Base<? extends T>`**|可以是 **`new Base<Sub_of_T>`**|你只打算读，所以只要对象是 `T` 的子类，读出来转成 `T` 永远安全。|
|**`Base<? super T>`**|可以 is **`new Base<Parent_of_T>`**|你只打算写，所以只要对象能包容 `T`，存进去永远安全。|

---

## 5. 字节码层面的真相

不管你左边写的是 `List<String>` 还是 `List<? extends Number>`，字节码里 `new` 出来的对象永远只有一种形态：**原始类型（Raw Type）**。

- **源码**：`List<? extends Number> list = new ArrayList<Integer>();`
    
- **字节码**：`NEW java/util/ArrayList` (没有类型信息)
    
- **编译器的坚持**：编译器在编译那一行赋值语句时，会去查符号表。它会比对引用类型的“范围”是否完全覆盖了对象类型的“范围”。如果对象类型超出了引用的承诺范围，它就拒绝生成字节码。
    

---

## 最后的关键点

如果你使用 **Java 7+ 的钻石语法 `<>`**：

Java

```
List<Number> list = new ArrayList<>(); 
```

这时编译器会自动根据左边的引用 `List<Number>` 推断出右边应该是 `new ArrayList<Number>()`。这实际上是编译器在帮你**强制保持一致**。

**既然聊到了类型推断，你想了解一下在 Java 8 之后，Lambda 表达式是如何利用这种“引用类型”来反向推导出参数类型的吗？**