# Guzzle Promoise 

promise 是一个状态机，它有三种状态：

 - 等待 Wait
 - 满足 Fulfilled
 - 拒绝  Reject


## Then 回调
一个Promoise对象主要跟then来交互，一旦完成，要么是成功，要么是失败，主意一点就是如果一旦成功了，状态就不能变更了

    use GuzzleHttp\Promise\Promise;
    $promise = new Promise();
    $promise->then(
	    // $onFulfilled 成功
	    function ($value) {
	        echo 'The promise was fulfilled.';
	    },
	    // $onRejected 失败
	    function ($reason) {
	        echo 'The promise was rejected.';
	    }
	);
	# resolve 和 reject只能设置一个，不能同时执行
	$promise->resolve('ok.'); # 告诉promise这个是成功的
	$promise->reject('not ok'); # 告诉promise这个是失败的

## Resolving Promise
```
$promise = new Promise();  
$promise  
  ->then(function ($value) {  
  // 返回一个值，但是不退出这个链式，返回值作为参数传递给下一个链  
  return "Hello, " . $value;  
  })  
  // 接收第一个链式(then)的值作为参数  
  ->then(function ($value) {  
  echo $value;  
  });  
// 输出 "Hello, reader."  
$promise->resolve('reader.');
```

## Resolving Promoise转发
```
$promise = new Promise();  
$nextPromise = new Promise();  
  
$promise  
  ->then(function ($value) use ($nextPromise) {  
  echo $value;  
  return $nextPromise; # 返回一个新的promiese  
  })  
  # 转发了nextPromise, 等同于 $nextPromise->then()了  
  ->then(function ($value) {  
  echo $value;  
  });  
  
// 第一个回调返回A  
$promise->resolve('A');  
// 第二个回调返回B  
$nextPromise->resolve('B');
```

## Promise rejection

```
$promise = new Promise();  
$promise->then(null, function ($reason) {  
  echo $reason;  
});  
  
$promise->reject('Error!');
```

## Rejection 转发
```
$promise = new Promise();
$promise->then(null, function ($reason) {
 throw new \Exception($reason);
})->then(null, function ($reason) {
 assert($reason->getMessage() === 'Error!');
});
$promise->reject('Error!');
```

还可以在回调里面返回一个RejectionPromise或者一个FulfilledPromise

```
use GuzzleHttp\Promise\Promise;
use GuzzleHttp\Promise\RejectedPromise;
$promise = new Promise();
$promise->then(null, function ($reason) {
 return new RejectedPromise($reason);
})->then(null, function ($reason) {
 assert($reason === 'Error!');
});
$promise->reject('Error!');
```

在拒绝回调返回一个成功promise

```
$promise = new Promise();  
$promise->then(null, function ($reason) {  
  return new FulfilledPromise($reason);  
})->then(function($response) {  
 var_dump('success');  
}, function ($reason) {  
 var_dump(assert($reason === 'Error!'));  
});  
  
$promise->reject('Error!');
```
如果拒绝回调链式没有定义接收拒绝回调，默认会调用成功的回调

```
$promise = new Promise();  
$promise  
  ->then(null, function ($reason) {  
  return "It's ok";  
  })  
 ->then(function ($value) {  
  echo assert($value === "It's ok");  
  });  
  
$promise->reject('Error!');
```

## 异步等待

```
# 传递一个等待结束的方法  
$promise = new Promise(function () use (&$promise) {  
  $promise->resolve('foo');  
});  
$promise->then(function($response) {  
 var_dump($response);  
}, function($response) {  
 var_dump($response);  
});  
// Calling wait will return the value of the promise.  
echo $promise->wait(); // outputs "foo  
// Outputs "Error!"
```

如果一个等待回调抛出异常, wait的时候会直接抛出异常
```
$promise = new Promise(function () use (&$promise) {  
  throw new \Exception('foo');  
});  
  
$promise->wait(); // throws the exception.
```

如果在wait之前调用了resolve，则不会执行wait方法
```
$promise = new Promise(function () { die('this is not called!'); });  
$promise->resolve('foo');  
echo $promise->wait(); // outputs "foo",上面的等待函数不会执行
```

如果在wait之前调用了reject， 则会抛出异常
```
$promise = new Promise(function () {});  
$promise->reject('foo');  
$promise->wait();
```
如果不想抛出异常，则别Unwrapping promise
```
$promise = new Promise();
$promise->reject('foo');
// This will not throw an exception. It simply ensures the promise has
// been resolved.
$promise->wait(false); #设置false
```

## 取消
```
$waitFn = function() use(&$promise) {  
  $promise->resolve('foo');  
};  
$cancelFn = function () {  
 var_dump('cancel');  
};  
$promise = new Promise($waitFn, $cancelFn);  
$promise->cancel();
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEwNjQ4MzE0MzhdfQ==
-->