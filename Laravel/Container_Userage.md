
# 容器的使用方式

## 单例容器
```php
$container = Container::setInstance(new Container);
```

## 绑定

### 匿名函数绑定
```php
$container = new Container;
$container->bind('name', function() {
    return 'Taylor';
});
dump('Taylor' == $container->make('name'))
```

### 带参数绑定
```php
$container = new Container;

$container->bind('foo', function($container, $config) {
    return $config;
});


dump($container->make('foo', [1,2,3]));
```

### bindif 如果没有绑定过就绑定，绑定过了就不再绑定
```php
$container = new Container;
$container->bind('name', function() {
    return 'Taylor';
});

$container->bindIf('name', function () {
    return 'Dayle';
});

dump('Taylor' == $container->make('name'))
```

### 单例绑定
```php
$container = new Container;
$class = stdClass::class;
$container->singleton('class', function() use($class) {
    return $class;
});

$otherclass = stdClass::class;
# 如果没有绑定class的话就绑定
$container->singletonIf('class', function() use ($otherclass) {
    return $otherclass;
});

dump($class == $container->make('class'));
```

### bind和singleton的区别
```php
$container->singleton('class', function() {
    return new stdClass;
});

dump($container->make('class') === $container->make('class')); # true

$container->bind('class', function() {
    return new stdClass;
});

dump($container->make('class') === $container->make('class')); # false

```

### 抽象类绑定具体实现类
```php
$container = new Container;

# cache接口由redis实现
$container->bind(CacheInterface::class, \EasyFramework\Cache\RedisCache::class);

# TestClass2的构造函数依赖cache接口
$container->make(\EasyFramework\Http\TestClass2::class);
```

### 把容器传递到解析器
```php
$container = new Container;

$container->bind('something', function($container) {
    return $container;
});

dump($container->make('something'));
```

### ArrayAccess方式绑定
```
$container = new Container;

$container['something'] = function ($container) {
    return $container;
};

dump($container['something']);
```

### 别名 Aliases
```php
$container = new Container;

$container['foo'] = 'bar';
$container->alias('foo', 'baz'); # foo的别名是baz
$container->alias('baz', 'bat'); # baz的别名是bat
# foo = baz = bat
dump('bar' == $container->make('foo'));
dump('bar' == $container->make('baz'));
dump('bar' == $container->make('bat'));
```

### 绑定可以被覆盖
```php
$container = new Container;

$container['foo'] = 'bar';
$container['foo'] = 'baz';
dump('baz' == $container['foo']); # true
dump('bar' == $container['foo']); # false
```

### 绑定实例然后返回实例
```php
$container = new Container;

$bound = new stdClass();
$resolved = $container->instance('foo', $bound);
dump($bound === $resolved);
dump($container->debug());
```

### 解析带有参数类
```php
class ContainerConcreteStub {}
class ContainerDefaultValueStub
{
    public $stub;
    public $default;
    public function __construct(ContainerConcreteStub $stub, $default = 'taylor')
    {
        $this->stub = $stub;
        $this->default = $default;
    }
}

$container = new Container;

$instance = $container->make(ContainerDefaultValueStub::class); # ContainerDefaultValueStub 构造函数依赖 ContainerConcreteStub参数
dump(ContainerConcreteStub::class == get_class($instance->stub));
```

### 解除绑定
```php
$container = new Container;

$container->instance('object', new stdClass());

# unset 隐式调用 offsetUnset
unset($container['object']); 

dump($container->bound('object'));
```


### 通过ArrayAccess绑定实例和别名检查
```php
$container = new Container;

$container->instance('object', new stdClass());
$container->alias('object', 'alias');

# isset 隐式调用 offsetExists
dump(isset($container['object'])); 
dump(isset($container['alias']));
```

### 重新绑定rebinding

#### bing方式
```php
$container = new Container;

unset($_SERVER['__test.rebind']);
$container->bind('foo', function (){
    $_SERVER['__test.rebind'] = false; 
});
dump($_SERVER['__test.rebind']); # null

# 如果foo已经绑定过之后, 会重新绑定并且调用make方法， 并且执行上一次绑定的方法
$container->rebinding('foo', function (){
    $_SERVER['__test.rebind'] = true; 
});
dump($_SERVER['__test.rebind']); # false

$container->bind('foo', function (){

});
dump($_SERVER['__test.rebind']); # true
```

instance方式
```php
$container = new Container;

unset($_SERVER['__test.rebind']);
$container->instance('foo', function (){
    $_SERVER['__test.rebind'] = false;
});

# 实例化对象直接从instances数组返回,  不会继续执行make方法
$container->rebinding('foo', function ($container){
    $_SERVER['__test.rebind'] = true;
});
dump($_SERVER['__test.rebind']);

$container->instance('foo', function (){
    $_SERVER['__test.rebind'] = false;
});
dump($_SERVER['__test.rebind']);
```

### 删除指定实例化的对象
```php
$container = new Container;
$containerConcreteStub = new ContainerConcreteStub;
$container->instance(ContainerConcreteStub::class, $containerConcreteStub);
$container->forgetInstance(ContainerConcreteStub::class); # 删除instances数组的指定对象
```

### 删除所有已经实例化的对象
```php
 $container = new Container;
$containerConcreteStub1 = new ContainerConcreteStub;
$containerConcreteStub2 = new ContainerConcreteStub;
$containerConcreteStub3 = new ContainerConcreteStub;
$container->instance('Instance1', $containerConcreteStub1);
$container->instance('Instance2', $containerConcreteStub2);
$container->instance('Instance3', $containerConcreteStub3);
$container->forgetInstances();
```

### 容器刷新
```php
$container = new Container;

$container->bind('ConcreteStub', function () {
    return new ContainerConcreteStub;
}, true);

$container->alias('ConcreteStub', 'ContainerConcreteStub');
$container->make('ConcreteStub');
dump($container->resolved('ConcreteStub'));
dump($container->isAlias('ContainerConcreteStub'));
dump($container->getBindings());
$container->flush(); # 刷新之后全部都没了
dump($container->resolved('ConcreteStub'));
dump($container->isAlias('ContainerConcreteStub'));
dump($container->getBindings());
```


### 检查对象是否解析了
```php
$container = new Container;

$container->bind('ConcreteStub', function () {
    return new ContainerConcreteStub;
}, true);

$container->alias('ConcreteStub', 'foo');

dump($container->resolved('ConcreteStub'));
dump($container->resolved('foo'));
$container->make('foo');
dump($container->resolved('ConcreteStub'));
dump($container->resolved('foo'));
```

### 查找别名
```php
$container = new Container;

$container->alias('foo', 'foo1');
$container->alias('foo1', 'foo2');
$container->alias('foo2', 'foo3');

dump($container->getAlias('foo3')); # 结果是foo, 可以递归查找最原始的抽象方法
```

### 延迟解析类
```php
$container = new Container;

$container->bind('name', function () {
    return 'Taylor';
});

$factory = $container->factory('name'); # 返回一个延迟的make匿名函数

dump($factory() == $container->make('name'));
```

### 解析带参数的对象
```php
$container = new Container;

$instance = $container->make(ContainerDefaultValueStub::class, ['default' => 'value']);

dump($instance->default);

$container->bind('foo', function($app, $config) {
    return $config;
});

$instance = $container->make('foo', ['default' => 'value']);
dump($instance);
```

### 解析一个抽象类
```php
$container = new Container;

$container->bind(IContainerContractStub::class, ContainerInjectVariableStubWithInterfaceImplementation::class);
$instance = $container->make(IContainerContractStub::class, ['something' => 'laurence']);
dump($instance->something);
```

### 嵌套解析
```php
$container = new Container;

$container->bind("foo", function($container, $config) {
    return $container->make('bar', ['name' => 'dada']);
});

$container->bind('bar', function($container, $config) {
    return $config;
});

dump($container->make('foo', ['something']));
```

### 单例绑定
```php
$container = new Container;

$container->singleton('foo', function ($app, $config){
    return $config;
});

dump($container->make('foo', [1,2,3]));
```

### 无参数的构建
```php
$container = new Container;

dump($container->build(ContainerConcreteStub::class));
```

### 有参数的构
```php
$container = new Container;

$container->bind(IContainerContractStub::class, ContainerImplementationStub::class);
$container->build(ContainerDependentStub::class);
```

### 检索是否有绑定
```php
$container = new Container;

$container->bind(IContainerContractStub::class, ContainerImplementationStub::class);
dump($container->has(IContainerContractStub::class));

```

### get解析对象
```php
$container = new Container;

$container->bind('Taylor', stdClass::class);
dump($container->get('Taylor'));
```

### 动态设置服务
```php
$container = new Container;

$container['name'] = 'dada';
dump($container['name']);
```

### 直接调用对象方法
```php
class  Test {
    public function test2() {
        echo 1234;
    }
}
$inject = $container->make(Test::class);
$container->call([$inject, 'test2']);
# or
$container->bind(Test::class);
$container->call([Test::class, 'test2']);
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTQ4MjQxMjQ0Ml19
-->