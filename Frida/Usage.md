# Frida 使用
### Frida版本

```
pip install frida==12.11.18
pip install frida-tools==5.4.0
pip install objection==1.8.4
```

### Frida-Inject

```
/data/local/tmp/frida-inject -n com.taobao.idlefish -s hook.js --runtime=v8
```



### Objection

查看某个方法

```
android hooking watch class_method android.content.Context.startActivity  --dump-args --dump-backtrace --du
mp-return
```



### Android 与 Frida 使用思想

当实例化一个类之后，就可以把当前的变量当做Java来使用了

```
#Android
String str = new String();
str.toString()

# Frida
var JavaString = Java.use("Java.util.String").$new()
JavaString.toString() # 直接像java那样调用即可
```



### 获取当前context

```
var context = Java.use('android.app.ActivityThread').currentApplication().getApplicationContext();
```



### 实例化对象

```
Java.use("Java.util.String").$new()
```



### 打印具体对象名

```
# android
public final MethodChannel channel;

# frida
this.channel # 返回[object object]
this.channel.value # 返回具体对象
this.channel.value.$className # 返回类名
```



### 调用Toast

```
Java.perform(function() {  
    Java.use("com.taobao.fleamarket.home.activity.MainActivity").onCreate.implementation = function(a) {
        console.log("echo")
        var context = Java.use('android.app.ActivityThread').currentApplication().getApplicationContext();
		# 需要在主线程(scheduleOnMainThread)下调用，不然报错
        Java.scheduleOnMainThread(function() {
            var toast = Java.use("android.widget.Toast");
            toast.makeText(Java.use("android.app.ActivityThread").currentApplication().getApplicationContext(), Java.use("java.lang.String").$new("This is works!"), 1).show();
        });
        return this.onCreate(a)
    }
})
```



### 注册一个类实现抽象方法或者集成抽象类

```
# android
public interface Result {
void error(String arg1, String arg2, Object arg3);

void notImplemented();

void success(Object arg1);
}

#frida
var testResultClass = Java.registerClass({
            name: "io.flutter.plugin.common.MethodChannel.MyIncomingMethodCallHandler",
            implements: [Java.use("io.flutter.plugin.common.MethodChannel$Result")],
            fields: {
                
             },
             methods: {
                error: function(arg1, arg2, arg3) {
                   console.log('error')
                },
                notImplemented:function() {
                    console.log('notImplemented')
                },
                success:function(obj) {
                    console.log('success')
                }
             }
          })

```

### 创建byte[]

```
var buffer = Java.array('byte', [ 13, 37, 42 ])
```



### Int.class表示

一般情况下直接用int就行了，frida会自动处理，如果非要弄个为什么就下面

```
Integer.class.getField("TYPE").get(null)
```



### frida生成[Ljava.lang.Object;

```
var newList = Java.use("java.util.ArrayList").$new();
newList.add(Java.use("java.lang.Integer").$new(9));
newList.add(Java.use("java.lang.String").$new("nihao"));
```



### 获取类字段

```
#类名.字段名.对象
class.field.value
MyClass.myField.value
```



### 内部类调用父类的变量

```
# android
MethodCall call = MethodChannel.this.codec.decodeMethodCall(message);

# frida
# this = 内部类对象
# this$0 = MethodChannel.this
# value = 对象实例
this.this$0.value.codec.value
```

### 获取所有重载方法 -- overloads

```
Java.performNow(function () {
  var MainActivity = Java.use('com.example.myapplication.MainActivity');
  var methods = MainActivity.test1.overloads;
  for (var i in methods) {
    var types = methods[i].argumentTypes;
    for (var j in types) {
      console.log(types[j].className);
    }
    console.log('---');
  }
});
```





### 参考

https://yuanbug.github.io/2020/05/06/2020/frida-java-hook