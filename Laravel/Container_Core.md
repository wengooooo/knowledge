# 容器实现原理

## 绑定的模式支持5种

## 容器重要的变量和方法
```php
# 别名
# 使用alias方法会添加别名, 别名分别有aliases和abstractAliases数组进行维护
alias($abstract, $alias)
$aliases = [];
$abstractAliases = [];

# 已经构建的对象由resolved数组进行维护
$resolved = [];

# instance的绑定由instances维护，都是存储单例的对象
instance($abstract, $instance) # 抽象类与实例化对象进行绑定
$instances = [];


# bind, bindif, singleton, singletonif进行绑定的由bindings数组进行维护
bind($abstract, $concrete = null, $shared = false) # 抽象类与具体类进行绑定
bindIf($abstract, $concrete = null, $shared = false) # 抽象类与具体类进行绑定,前提是从来没有绑定过
singleton($abstract, $concrete = null) # 抽象类与具体类进行单例模式的绑定
singletonIf($abstract, $concrete = null) # 抽象类与具体类进行单例模式的绑定，前提也是从来没有绑定过 
$bindings = [];

# rebinding的绑定由reboundCallbacks数组维护
rebinding($abstract, Closure $callback)
$reboundCallbacks = [];

# 构建对象
make($abstract, array $parameters = []) # 负责调用resolve然后返回构建的对象
resolve($abstract, $parameters = [], $raiseEvents = true)  # 构建前先对行为的检测
build($concrete) # 利用反射或者匿名函数进行构建对象

```
## Instance绑定

```php
# 原型
instance($abstract, $instance)
```
方法必须实现下面的调用
```php
$container = new Container;
$instance = $container->instance('path', 'app/path');                         # 字符串绑定
$container->instance(ContainerConcreteStub::class, new stdclass); # 实例化对象绑定
$container->instance('foo', function(){});                        # 匿名函数绑定
```

### Instance实现原理
> instance绑定的关系由instances数组维护

> instance绑定后不需要用make进行构建

```php
# 移除给定的别名
protected function removeAbstractAlias($searched)
{

    if (! isset($this->aliases[$searched])) {
        return;
    }

    foreach ($this->abstractAliases as $abstract => $aliases) {
        foreach ($aliases as $index => $alias) {
            if ($alias == $searched) {
                unset($this->abstractAliases[$abstract][$index]);
            }
        }
    }
}

public function instance($abstract, $instance)
{
    # 移除别名，防止重复绑定
    $this->removeAbstractAlias($abstract);

    # 检测是否绑定了，防止重复绑定
    $isBound = $this->bound($abstract);

    # 删除别名，防止重复添加
    unset($this->aliases[$abstract]);

    # 维护绑定关系，后续需要使用直接取出即可
    $this->instances[$abstract] = $instance;

    # 如果绑定过了则重新绑定
    if ($isBound) {
        $this->rebound($abstract);
    }

    return $instance;
}

# 检测是否绑定了
public function bound($abstract)
{
    return isset($this->bindings[$abstract]) ||
        isset($this->instances[$abstract]) ||
        $this->isAlias($abstract);
}

```


## bind绑定

bind方法必须实现下面方法的调用
```php
$container = new Container;

# 无参数匿名方法绑定
$container->bind('name', function () {
    return 'Taylor';
});

# 定制实例化方式绑定
$container->bind('ConcreteStub', function () {
    return new ContainerConcreteStub;
}, true);


# 抽象类/接口类与具体类绑定

# 接口由ContainerImplementationStub实现
$container->bind(IContainerContractStub::class, ContainerImplementationStub::class);

#ContainerDependentStub依赖IContainerContractStub接口
$class = $container->make(ContainerDependentStub::class); 

# 带默认参数匿名方法绑定
# 变量container是每次都会传入进去的
$container->bind('something', function ($container) {
    return $containerc;
});

# 带默认参数和自定义参数绑定
# 除了默认参数container，后面还可以带很一个数组参数
$container->bind('foo', function ($container, $config) {
    return $config;
});

# 函数覆盖绑定
$container->bind('foo', function ($app, $config) {
    return $app->make('bar', ['name' => 'Taylor']);
});
$container->bind('bar', function ($app, $config) {
    return $config;
});
$this->assertEquals(['name' => 'Taylor'], $container->make('foo', ['something']));
```

### bind实现原理
```php
public function bind($abstract, $concrete = null, $shared = false)
{
    # 删除已经绑定过的实例和别名
    $this->dropStaleInstances($abstract);

    # 如果没有具体类，那么抽象类就是具体类，具体类不能为空的
    if (is_null($concrete)) {
        $concrete = $abstract;
    }

    # 具体类必须是匿名函数，如果不是匿名函数就转换为匿名函数，从而实现延迟加载
    if (! $concrete instanceof Closure) {
        $concrete = $this->getClosure($abstract, $concrete);
    }

    # 存储绑定关系
    $this->bindings[$abstract] = compact('concrete', 'shared');

    # 如果解析过就重新绑定
    if ($this->resolved($abstract)) {
        $this->rebound($abstract);
    }
}

# 返回一个匿名函数
protected function getClosure($abstract, $concrete)
{
    return function ($container, $parameters = []) use ($abstract, $concrete) {
        # 如果抽象类和实现类相等的话，那么抽象类就等于实现类了，直接就可以实例化
        if ($abstract == $concrete) {
            return $container->build($concrete);
        }

        # 如果抽象类和实现类不想等的话，那么还需要先检查下实现类，然后再build
        return $container->resolve(
            $concrete, $parameters, $raiseEvents = false
        );
    };
}
```

## 构建对象实现原理
三步构建
> make 负责递归解析所有的依赖类
```php
public function make($abstract, array $parameters = [])
{

    $object = $this->resolve($abstract, $parameters);

    return $object;
}
```
> resolve 是 兼容 instance 与 bind 和没有做绑定的直接make的方式
```php
protected function resolve($abstract, $parameters = [], $raiseEvents = true)
{
    # 因为支持别名, 所以无论是原名还是别名都可以实例化，所以先获取别名
    $abstract = $this->getAlias($abstract);

    # 处理上下文构建
    $needsContextualBuild = ! empty($parameters) || ! is_null(
        $this->getContextualConcrete($abstract)
    );

    # 兼容instances维护的就直接返回，不再构建
    if (isset($this->instances[$abstract]) && ! $needsContextualBuild) {
        return $this->instances[$abstract];
    }

    $this->with[] = $parameters;

    // 1. 如果通过bind绑定的就从bindings返回匿名方法
    // 2. 如果没有绑定过直接make的话, abstract必须是具体类
    $concrete = $this->getConcrete($abstract);

    # 开始构建
    if ($this->isBuildable($concrete, $abstract)) {
        $object = $this->build($concrete);
    } else {
        $object = $this->make($concrete);
    }

    # 处理扩展
    foreach ($this->getExtenders($abstract) as $extender) {
        $object = $extender($object, $this);
    }

    # 判断这个type是否是一个单例 如果在绑定的时候定义为单例的话 那么就将其保存在instances数组中
    # 后面其他地方再需要make它的时候直接从instances中取出返回
    if ($this->isShared($abstract) && ! $needsContextualBuild) {
        $this->instances[$abstract] = $object;
    }

    if ($raiseEvents) {
        $this->fireResolvingCallbacks($abstract, $object);
    }

    # 维护已经解析的对象
    # resolved方法可以进行判断
    $this->resolved[$abstract] = true;

    array_pop($this->with);

    return $object;
}

# 获取具体类
protected function getConcrete($abstract)
{
    if (! is_null($concrete = $this->getContextualConcrete($abstract))) {
        return $concrete;
    }

    # 如果绑定了就会返回具体类
    if (isset($this->bindings[$abstract])) {
        return $this->bindings[$abstract]['concrete'];
    }

    # 返回抽象类
    return $abstract;
}

protected function isBuildable($concrete, $abstract)
{
    # 只有抽象类等于具体类或者是一个匿名的方法才能构建
    return $concrete === $abstract || $concrete instanceof Closure;
}
```

> build

```php
public function build($concrete)
{
    // If the concrete type is actually a Closure, we will just execute it and
    // hand back the results of the functions, which allows functions to be
    // used as resolvers for more fine-tuned resolution of these objects.
    // 通过bing方法来绑定的都会执行下面的方法

    if ($concrete instanceof Closure) {
        return $concrete($this, $this->getLastParameterOverride());
    }

    try {
        // 没有通过bing方法来绑定的都会通过反射来实例化对象
        // 如 Container::getInstance()->make($abstract, $parameters)
        $reflector = new ReflectionClass($concrete);
    } catch (ReflectionException $e) {
        throw new BindingResolutionException("Target class [$concrete] does not exist.", 0, $e);
    }

    // If the type is not instantiable, the developer is attempting to resolve
    // an abstract type such as an Interface or Abstract Class and there is
    // no binding registered for the abstractions so we need to bail out.
    if (! $reflector->isInstantiable()) {
        return $this->notInstantiable($concrete);
    }

    $this->buildStack[] = $concrete;

    $constructor = $reflector->getConstructor();

    // If there are no constructors, that means there are no dependencies then
    // we can just resolve the instances of the objects right away, without
    // resolving any other types or dependencies out of these containers.
    if (is_null($constructor)) {
        array_pop($this->buildStack);

        return new $concrete;
    }

    $dependencies = $constructor->getParameters();

    // Once we have all the constructor's parameters we can create each of the
    // dependency instances and then use the reflection instances to make a
    // new instance of this class, injecting the created dependencies in.
    try {
        $instances = $this->resolveDependencies($dependencies);
    } catch (BindingResolutionException $e) {
        array_pop($this->buildStack);

        throw $e;
    }

    array_pop($this->buildStack);

    return $reflector->newInstanceArgs($instances);
}
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMTcwNjgwNzUwXX0=
-->