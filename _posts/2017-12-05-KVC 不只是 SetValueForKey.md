---
layout: post
title: 'KVC 不只是 SetValueForKey'
subtitle: 'KVC访问集合属性, 使用集合运算符以及KVC的搜索模式(逻辑)'
date: 2017-12-05
categories: 技术
cover: 'http://zhaoyanda-blog.oss-cn-qingdao.aliyuncs.com/%E5%8D%9A%E5%AE%A2%E5%B0%81%E9%9D%A2/KVC%20%E4%B8%8D%E5%8F%AA%E6%98%AF%20SetValueForKey.jpg'
tags: Cocoa iOS
---
**KVC** (KeyValueCoding), 做`iOS`开发的几乎无人不知, 哪怕做`iOS`开发不久的小伙伴也能说出个一二三来, 比如: 字典转模型, 访问私有变量啊之类的. 好一点的还能说出`KVC`和`KVO`的关系. 实际上`KVC`是很多`Cocoa`技术中的基础概念, 如`KVO`, `Core Data`, `Cocoa bindings`. 当然最重要的是KVC还可以帮助我们简化代码. 

咱们由浅入深好好聊聊这个`KVC`.

## KVC能干什么?

使用KVC有个前提, 那就是遵守`NSKeyValueCoding`这个非正式协议, 并为方法提供实现, 不过好在`Objective-C`中所有的类都继承自`NSObject`, `NSObject`是已经遵守了这个协议, 并提供了默认的实现. 所以无需我们做太多的工作就可以使用`KVC`.

KVC为提供了一下功能:

- **访问对象属性**
- **操作集合属性**
- **在集合对象上调用集合运算符**
- **访问非对象属性**
- **按键路径访问属性**

简单的属性赋值取值操作(也就是`valueForKey`和`setValueForKey`), 网上的文章一搜一大把. 今天主要讨论的是一下几点:

- 访问集合属性
- 使用集合运算符
- KVC 搜索模式 (`KVC`如何通过`key`值找到对应的`value`)

## 访问集合属性

当访问目标对象中的集合属性时, 依然可以通过`valueForKey:`和`setValue:forKey:`方法获取或者设置集合属性, 比如这样:

<pre><code class="language-objectivec">
@class Animal;
@interface Person : NSObject
@property (nonatomic, copy) NSString *name;
@property (nonatomic, strong) NSArray&lt;Animal *> *pets;
@end

@interface Animal : NSObject
@property (nonatomic, copy) NSString *name;
@property (nonatomic, assign) NSInteger age;
@end

Animal *cat = [[Animal alloc] init];
cat.name = @"cat";
Animal *dog = [[Animal alloc] init];
dog.name = @"dog";
    
Person *p1 = [[Person alloc] init];
p1.pets = @[cat, dog];
    
NSMutableArray *petsM = [[p1 valueForKey:@"pets"] mutableCopy];
[petsM removeLastObject];
[p1 setValue:petsM.copy forKey:@"pets"];
NSLog(@"p1.pets: %@", p1.pets);

/* 打印
 p1.pets: (
    cat
 )
 */
</code></pre>

上面代码可以看到我们声明了 `Animal` 和 `Person` 两个类, `Person` 类中有个 `pets` 的数组属性, 我们一开始给 `p1` 实例的 `pets` 属性赋值为 `@[cat, dog]`, 假设我们有需求, 需要删除数组中的最后一个元素, 首先我们通过`KVC`取出`pets`属性中的值并加以可变拷贝, 删除指定的值, 再使用`KVC`赋值回去. 根据打印结果, 我们完成了需求.

但是, 这种操作仿佛没有那么优雅, 还有一些繁琐, 其实`KVC`提供了更方便的方法.

#### 代理对象

协议为集合对象访问定义了三种不同的代理方法, 每个都具有一个键和一个键路径变体:

- `mutableArrayValueForKey:` 和 `mutableArrayValueForKeyPath:`
这两个方法会返回一个 `NSMutableArray` 代理对象.
- `mutableSetValueForKey:` 和 `mutableSetValueForKeyPath:`
这两个方法会返回一个 `NSMutableSet` 代理对象.
- `mutableOrderedSetValueForKey:` 和 `mutableOrderedSetValueForKeyPath:`
这两个方法会返回一个 `NSMutableOrderedSet` 代理对象.

当对代理对象进行操作, 向其添加对象, 从其中移除对象或替换其中的对象时, 协议的默认实现将相应地修改原本的属性.

刚才的代码现在可以改成这样:

<pre><code class="language-objectivec">
Animal *cat = [[Animal alloc] init];
cat.name = @"cat";
Animal *dog = [[Animal alloc] init];
dog.name = @"dog";
    
Person *p1 = [[Person alloc] init];
p1.pets = @[cat, dog];
    
NSMutableArray *proxyPets = [p1 mutableArrayValueForKey:@"pets"];
[proxyPets removeLastObject];
NSLog(@"p1.pets: %@", p1.pets);
/* 打印
 p1.pets: (
    cat
 )
 */
</code></pre>

因为 `pets` 是一个数组, 所以我们使用了 `mutableArrayValueForKey:` 方法, 反回了一个代理对象. 从代码的打印结果来看, 我们改变了这个 `proxyPets` 这个代理对象, 也同时改变了原本的 `pets` 属性, 更加优雅的完成了需求.

对 `NSSet` 和 `NSOrderedSet` 类型的属性的操作也都是类似的. 也就是说当我们对集合类型的属性进行进行操时, 使用这种方式尤为便利.

## 集合运算符

当给兼容 `KVC` 的对象发送 `valueForKeyPath:` 消息时, 其实还可以在keyPath路径中嵌入集合运算符.
集合运算符指定了数据返回以前操作数据的某种方式.
集合运算符路径的格式如下:

![](https://developer.apple.com/library/content/documentation/Cocoa/Conceptual/KeyValueCoding/art/keypath.jpg)

- left key path(左键路径): 指定要接受运算的集合属性的路径, 如果直接给一个集合属性发送集合运算符, 左键路径可省略.
- collection operator(集合运算符): 指定集合运算的方式.
- right key path(右键路径): 指定运算符应处理的集合中元素的属性.

举个例子:

<pre><code class="language-objectivec">
Animal *cat = [[Animal alloc] init];
cat.name = @"cat";
cat.age = 5;
Animal *dog = [[Animal alloc] init];
dog.name = @"dog";
dog.age = 9;
    
Animal *pig = [[Animal alloc] init];
pig.name = @"pig";
pig.age = 4;
    
Person *p1 = [[Person alloc] init];
p1.pets = @[cat, dog, pig];
    
NSNumber *avgAge = [p1 valueForKeyPath:@"pets.@avg.age"];
NSLog(@"pets中动物的平均年龄: %@", avgAge);
// 打印 pets中动物的平均年龄: 6
</code></pre>

上面代码中 `pets.@avg.age` 就是集合运算符路径.

-  `pets` 是左键路径, 指定了 `p1` 实例中的 `pets` 数组.
-  `@avg` 是集合运算符, 指定了对 `pets` 的运算方式, 也就是求平均值.
-  `age` 是右键路径, 指定要对 `age` 的值进行运算. 

所以这个集合运算符路径可以直接获得p1实例中pets数组里Animal实例中的age属性的平均值.

### 常用的集合运算符
例子中 `@avg` 只是很多集合运算符中的一个, 常用的运算符有这些:

- `@agv`: 获取集合元素中右键路径属性的平均值. 
- `@count`: 可以得出集合属性中元素的数量, `@count` 集合运算符比较特殊, 无需右键运算符.
- `@max`: 获取集合元素中右键路径属性的最大值. 
- `@min`: 获取集合元素中右键路径属性的最小值. 
- `@sum`: 获取集合元素中右键路径属性值的和.
- `@distinctUnionOfObjects`: 返回集合元素中右键路径属性的数组, 并去重, 比如:上面的例子使用`@"pets.@distinctUnionOfObjects.name"`集合运算符路径, 将返回一个`@[@"cat", @"dog", @"pig"]`的数组, 
- `@unionOfObjects`: 和 `@distinctUnionOfObjects` 一样, 但是不去重.

使用集合运算符在某些情况下对简化代码很有帮助.

## KVC 搜索模式

`KVC`的核心在于使用`key`值对`value`的访问, 但是`KVC`是如何通过`key`键来找到我们想要的`value`呢? `KVC`提供了默认的搜索方案, 来搜索相关的方法或者变量值. 虽然很少需要手动改变这些搜索方式, 但是了解`KVC`的搜索方式, 这对于跟踪键值编码对象的行为将是很有帮助的.

### 获取值搜索模式

获取值, 也就是常用的 `valueForKey:` 方法, 我们将给定一个`key`参数作为输入, 返回一个对应的值, 当调用此方法时基本流程如下:

1. 首先在实例中搜索方法, 其名称类似于 `get<Key>`, `<key>`, `is<Key>`或`_<key>`, 按此顺序. 如果找到就调用它, 然后继续执行步骤3以得到结果. 否则继续执行下一步.
2. 如果找不到步骤1中的方法, 那么调用类方法 `accessInstanceVariablesDirectly`, 如果返回`YES`, 那么搜索名为`_<key>`、 `_is<Key>`、 `<key>`或`is<Key>`的实例变量, 按该顺序进行, 找到就进行第3步. 如果没找到就进行第4步. 如果返回`NO`, 那么不进行搜索, 直接执行第4步. 
  > Tips: 如果我们不希望KVC搜索成员变量, 可以重写`accessInstanceVariablesDirectly`类方法, 返回NO.
  
3. 根据找到的方法返回值或属性值. 如果是一个对象指针, 那么直接返回该对象, 如果该值是`NSNumber`所支持的数据类型, 请将其存储在`NSNumber`实例中并返回. 如果结果是`NSNumber`不支持的标量类型, 就转换为`NSValue`对象, 然后返回它.
4. 执行到这一步说明`KVC`并没有根据指定的`Key`值找到对应的方法或成员变量. `KVC`会调用`valueForUndefinedKey:`方法, 这个方法默认实现会抛出异常, 但是我们依然可以通过重写此方法避免崩溃.

### 设置值搜索模式

也是常用的`setValue:forKey:`, 给定`key`和`value`参数作为输入, 尝试将名为`key`的属性设置为`value`, 当调用此方法时基本流程如下:

1. 按顺序查找名为`set<Key>:`或`_set<Key>`的方法, 如果找到, 请用输入值调用它, 然后完成. 如果没有找到则进行下一步.
2. 如果类方法`accessInstanceVariablesDirectly`返回`YES`, 那么按顺序搜索`_<key>`、 `_is<Key>`、 `<key>`或`is<Key>`的名称的实例变量. 如果找到则直接使用输入值赋值. 如果没找到或者类方法返回`NO`.
3. 执行到这一步说明没有找到对应的方法和成员变量, `KVC`会自动调用`setValue:forUndefinedKey:`方法, 默认此方法会导致异常, 不过依然可以通过重写避免.

   
   

