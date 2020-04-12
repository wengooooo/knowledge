# Controller分析

### 原型
Illuminate\Foundation\Http\Kernel
```php
protected function sendRequestThroughRouter($request)
{
    $this->app->instance('request', $request);

    Facade::clearResolvedInstance('request');

    $this->bootstrap();

    return (new Pipeline($this->app))
                ->send($request)
                ->through($this->app->shouldSkipMiddleware() ? [] : $this->middleware)
                ->then($this->dispatchToRouter());
}
```
Illuminate\Routing\Router
```php
public function dispatch(Request $request)
{
    $this->currentRequest = $request;

    return $this->dispatchToRoute($request);
}

public function dispatchToRoute(Request $request)
{   
    # 先找到route
    # 再执行route
    return $this->runRoute($request, $this->findRoute($request));
}

# 找路由
protected function findRoute($request)
{
    $this->current = $route = $this->routes->match($request);
    # 实例化Route
    $this->container->instance(Route::class, $route);

    return $route;
}
```

Illuminate\Routing\RouteCollection
```php
public function match(Request $request)
{
    $routes = $this->get($request->getMethod());

    // First, we will see if we can find a matching route for this current request
    // method. If we can, great, we can just return it so that it can be called
    // by the consumer. Otherwise we will check for routes with another verb.
    $route = $this->matchAgainstRoutes($routes, $request);

    if (! is_null($route)) {
        return $route->bind($request);
    }

    // If no route was found we will now check if a matching route is specified by
    // another HTTP verb. If it is we will need to throw a MethodNotAllowed and
    // inform the user agent of which HTTP verb it should use for this route.
    $others = $this->checkForAlternateVerbs($request);

    if (count($others) > 0) {
        return $this->getRouteForMethods($request, $others);
    }

    throw new NotFoundHttpException;
}

```

Illuminate\Routing\Route
```php
public function bind(Request $request)
{
    
    $this->compileRoute();

    $this->parameters = (new RouteParameterBinder($this))
                    ->parameters($request);

    $this->originalParameters = $this->parameters;

    return $this;
}
```

Illuminate\Routing\RouteParameterBinder
```php
public function parameters($request)
{
    // If the route has a regular expression for the host part of the URI, we will
    // compile that and get the parameter matches for this domain. We will then
    // merge them into this parameters array so that this array is completed.
    $parameters = $this->bindPathParameters($request);

    // If the route has a regular expression for the host part of the URI, we will
    // compile that and get the parameter matches for this domain. We will then
    // merge them into this parameters array so that this array is completed.
    if (! is_null($this->route->compiled->getHostRegex())) {
        $parameters = $this->bindHostParameters(
            $request, $parameters
        );
    }

    return $this->replaceDefaults($parameters);
}

protected function bindPathParameters($request)
{
    $path = '/'.ltrim($request->decodedPath(), '/');

    preg_match($this->route->compiled->getRegex(), $path, $matches);
    
    return $this->matchToKeys(array_slice($matches, 1));
}
```
Illuminate\Routing\Router
```php
# 执行路由
protected function runRoute(Request $request, Route $route)
{
    $request->setRouteResolver(function () use ($route) {
        return $route;
    });

    $this->events->dispatch(new RouteMatched($route, $request));

    return $this->prepareResponse($request,
        $this->runRouteWithinStack($route, $request)
    );
}

protected function runRouteWithinStack(Route $route, Request $request)
{
    $shouldSkipMiddleware = $this->container->bound('middleware.disable') &&
                            $this->container->make('middleware.disable') === true;

    $middleware = $shouldSkipMiddleware ? [] : $this->gatherRouteMiddleware($route);

    return (new Pipeline($this->container))
                    ->send($request)
                    ->through($middleware)
                    ->then(function ($request) use ($route) {
                        return $this->prepareResponse(
                            $request, $route->run()
                        );
                    });
}
```
Illuminate\Pipeline\Pipeline
```php
public function then(Closure $destination)
{

    $pipeline = array_reduce(

        array_reverse($this->pipes()), $this->carry(), $this->prepareDestination($destination)
    );

    return $pipeline($this->passable);
}

# 循环所有的中间件
# 其中有一个中间件是SubstituteBindings
protected function carry()
{
    return function ($stack, $pipe) {
        return function ($passable) use ($stack, $pipe) {
            try {

                if (is_callable($pipe)) {
                    // If the pipe is a callable, then we will call it directly, but otherwise we
                    // will resolve the pipes out of the dependency container and call it with
                    // the appropriate method and arguments, returning the results back out.
                    return $pipe($passable, $stack);
                } elseif (! is_object($pipe)) {
                    [$name, $parameters] = $this->parsePipeString($pipe);

                    // If the pipe is a string we will parse the string and resolve the class out
                    // of the dependency injection container. We can then build a callable and
                    // execute the pipe function giving in the parameters that are required.
                    $pipe = $this->getContainer()->make($name);

                    $parameters = array_merge([$passable, $stack], $parameters);
                } else {
                    // If the pipe is already an object we'll just make a callable and pass it to
                    // the pipe as-is. There is no need to do any extra parsing and formatting
                    // since the object we're given was already a fully instantiated object.
                    $parameters = [$passable, $stack];
                }

                
                $carry = method_exists($pipe, $this->method)
                                ? $pipe->{$this->method}(...$parameters)
                                : $pipe(...$parameters);

                return $this->handleCarry($carry);
            } catch (Exception $e) {
                return $this->handleException($passable, $e);
            } catch (Throwable $e) {
                return $this->handleException($passable, new FatalThrowableError($e));
            }
        };
    };
}
```


Illuminate\Routing\Middleware\SubstituteBindings
# 负责处理方法参数的orm处理
```php
public function handle($request, Closure $next)
{
    $this->router->substituteBindings($route = $request->route());

    $this->router->substituteImplicitBindings($route);

    return $next($request);
}
```

Illuminate\Routing\Router
```php
public function substituteImplicitBindings($route)
{
    ImplicitRouteBinding::resolveForRoute($this->container, $route);
}
```

Illuminate\Routing\ImplicitRouteBinding
```php
public static function resolveForRoute($container, $route)
{

    $parameters = $route->parameters();

    foreach ($route->signatureParameters(UrlRoutable::class) as $parameter) {
        if (! $parameterName = static::getParameterName($parameter->name, $parameters)) {
            continue;
        }

        $parameterValue = $parameters[$parameterName];

        if ($parameterValue instanceof UrlRoutable) {
            continue;
        }

        $instance = $container->make($parameter->getClass()->name);

        # 根据参数查询数据库的值
        if (! $model = $instance->resolveRouteBinding($parameterValue)) {
            throw (new ModelNotFoundException)->setModel(get_class($instance), [$parameterValue]);
        }

        $route->setParameter($parameterName, $model);
    }
}
```


Illuminate\Routing\Route
```php
public function signatureParameters($subClass = null)
{
    return RouteSignatureParameters::fromAction($this->action, $subClass);
}
```

namespace Illuminate\Routing\RouteSignatureParameters
```php
public static function fromAction(array $action, $subClass = null)
{
    $parameters = is_string($action['uses'])
                    ? static::fromClassMethodString($action['uses'])
                    : (new ReflectionFunction($action['uses']))->getParameters();

    return is_null($subClass) ? $parameters : array_filter($parameters, function ($p) use ($subClass) {
        return $p->getClass() && $p->getClass()->isSubclassOf($subClass);
    });
}
```

```php
protected static function fromClassMethodString($uses)
{
    [$class, $method] = Str::parseCallback($uses);

    if (! method_exists($class, $method) && is_callable($class, $method)) {
        return [];
    }
    
    # 通过反射方法获取到参数名字
    return (new ReflectionMethod($class, $method))->getParameters();
}
```


<!--stackedit_data:
eyJoaXN0b3J5IjpbNzU0OTM1NzM0XX0=
-->