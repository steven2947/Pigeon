# PigeonPro

This is a fork of [Pigeon](https://github.com/flutter/packages/tree/main/packages/pigeon).


## [怎样使用Pigeon](https://github.com/flutter/packages/blob/main/packages/pigeon/README.md)

Pigeon 是一个代码生成器工具，用于在 Flutter 和
主机平台类型安全，更轻松，更快捷。

## 支持的平台

目前 Pigeon 支持在 iOS、Java 上生成 Objective-C 代码
Android 的代码，并为 Windows 的 C++ 提供实验性支持。这
Objective-C 代码是
[Swift](https://developer.apple.com/documentation/swift/imported_c_and_objective-c_apis/importing_objective-c_into_swift)
并且 Kotlin 可以访问 Java 代码。


## How to use PigeonPro

与Pigeon相比，PigeonPro增加了全平台[Dart, Android, iOS]的数据同步，让团队可以同步配置信息，提高效率

```dart
@Configure()
class Info {
   String test = '123456';
   int test1 = 100;
   double test2 = 0.123;
   List<int> test3 = <int>[1, 2, 3, 4, 5];
   List<String> test4 = <String>['1', '2', '3'];
}
```
### 通过脚本生成的代码

Dart Code:
```dart
@Configure()
class Info {
   String test = '123456';
   int test1 = 100;
   double test2 = 0.123;
   List<int> test3 = <int>[1, 2, 3, 4, 5];
   List<String> test4 = <String>['1', '2', '3'];
}
```
Java Code:
```java
 public class Info {
   public static final String test = '123456';
   public static final Long test1 = 100;
   public static final Double test2 = 0.123;
   public static final List<Long> test3 = <int>[1, 2, 3, 4, 5];
   public static final List<String> test4 = <String>['1', '2', '3'];
}
```
OC Code:
```objectivec
@protocol Info
NSString *const test = @'123456';
NSNumber *const test1 = @100;
NSNumber *const test2 = @0.123;
NSArray<NSNumber *> *const test3 = @<int>[1, 2, 3, 4, 5];
NSArray<NSString *> *const test4 = @<String>['1', '2', '3'];
```


