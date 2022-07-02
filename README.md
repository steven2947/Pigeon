# Pigeon

This is a fork of [Pigeon](https://github.com/flutter/packages/tree/main/packages/pigeon).

Pigeon is a code generator tool to make communication between Flutter and the
host platform type-safe, easier and faster.

## Supported Platforms

Currently Pigeon supports generating Objective-C code for usage on iOS, Java
code for Android, and has experimental support for C++ for Windows. The
Objective-C code is
[accessible to Swift](https://developer.apple.com/documentation/swift/imported_c_and_objective-c_apis/importing_objective-c_into_swift)
and the Java code is accessible to Kotlin.

## Runtime Requirements

Pigeon generates all the code that is needed to communicate between Flutter and
the host platform, there is no extra runtime requirement.  A plugin author
doesn't need to worry about conflicting versions of Pigeon.

## Usage

1) Add Pigeon as a `dev_dependency`.
1) Make a ".dart" file outside of your "lib" directory for defining the
   communication interface.
1) Run pigeon on your ".dart" file to generate the required Dart and 
   host-language code: `flutter pub get` then `flutter pub run pigeon` 
   with suitable arguments (see [example](./example)).
1) Add the generated Dart code to `./lib` for compilation.
1) Implement the host-language code and add it to your build (see below).
1) Call the generated Dart methods.

### Flutter calling into iOS steps

1) Add the generated Objective-C code to your Xcode project for compilation
   (e.g. `ios/Runner.xcworkspace` or `.podspec`).
1) Implement the generated iOS protocol for handling the calls on iOS, set it up
   as the handler for the messages.

### Flutter calling into Android Steps

1) Add the generated Java code to your `./android/app/src/main/java` directory
   for compilation.
1) Implement the generated Java interface for handling the calls on Android, set
   it up as the handler for the messages.

### Flutter calling into Windows Steps

1) Add the generated C++ code to your `./windows` directory for compilation, and
   to your `windows/CMakeLists.txt` file.
1) Implement the generated C++ abstract class for handling the calls on Windows,
   set it up as the handler for the messages.

**Note:** Windows C++ is experimental while we get more usage and add more
testing.  Not all features may be supported.

### Calling into Flutter from the host platform

Flutter also supports calling in the opposite direction.  The steps are similar
but reversed.  For more information look at the annotation `@FlutterApi()` which
denotes APIs that live in Flutter but are invoked from the host platform.

### Rules for defining your communication interface

1) The file should contain no method or function definitions, only declarations.
1) Custom classes used by APIs are defined as classes with fields of the
   supported datatypes (see the supported Datatypes section).
1) APIs should be defined as an `abstract class` with either `HostApi()` or
   `FlutterApi()` as metadata.  The former being for procedures that are defined
   on the host platform and the latter for procedures that are defined in Dart.
1) Method declarations on the API classes should have arguments and a return
   value whose types are defined in the file, are supported datatypes, or are
   `void`.
1) Generics are supported, but can currently only be used with nullable types
   (example: `List<int?>`).

## Supported Datatypes

Pigeon uses the `StandardMessageCodec` so it supports any datatype Platform
Channels supports
[[documentation](https://flutter.dev/docs/development/platform-integration/platform-channels#codec)].
Nested datatypes are supported, too.

## Features

### Asynchronous Handlers

By default Pigeon will generate synchronous handlers for messages.  If you want
to be able to respond to a message asynchronously you can use the `@async`
annotation as of version 0.1.20.

Example:

```dart
class Value {
  int? number;
}

@HostApi()
abstract class Api2Host {
  @async
  Value calculate(Value value);
}
```

Generates:

```objc
// Objc
@protocol Api2Host
-(void)calculate:(nullable Value *)input 
      completion:(void(^)(Value *_Nullable, FlutterError *_Nullable))completion;
@end
```

```java
// Java
public interface Result<T> {
   void success(T result);
}

/** Generated interface from Pigeon that represents a handler of messages from Flutter.*/
public interface Api2Host {
   void calculate(Value arg, Result<Value> result);
}
```

```c++
// C++

/** Generated class from Pigeon that represents a handler of messages from Flutter.*/
class Api2Host {
public:
    virtual void calculate(Value value, flutter::MessageReply<Value> result) = 0;
}
```

### Null Safety (NNBD)

Pigeon supports generating null-safe code, but it doesn't yet support:

1) Nullable generics type arguments
1) Nullable enum arguments to methods

It does support:

1) Nullable and Non-nullable class fields.
1) Nullable return values
1) Nullable method parameters

The default is to generate null-safe code but in order to generate non-null-safe
code run Pigeon with the extra argument `--no-dart_null_safety`. For example:
`flutter pub run pigeon --input ./pigeons/messages.dart --no-dart_null_safety --dart_out stdout`.

### Enums

As of version 0.2.2 Pigeon supports enum generation in class fields.  For
example:
```dart
enum State {
  pending,
  success,
  error,
}

class StateResult {
  String? errorMessage;
  State? state;
}

@HostApi()
abstract class Api {
  StateResult queryState();
}
```

### Primitive Data-types

Prior to version 1.0 all arguments to API methods had to be wrapped in a class, now they can be used directly.  For example:

```dart
@HostApi()
abstract class Api {
   Map<String?, int?> makeMap(List<String?> keys, List<String?> values);
}
```

### TaskQueues

When targeting a Flutter version that supports the
[TaskQueue API](https://docs.flutter.dev/development/platform-integration/platform-channels?tab=type-mappings-kotlin-tab#channels-and-platform-threading)
the threading model for handling HostApi methods can be selected with the
`TaskQueue` annotation:

```dart
@Configure()
class Api2Host {
  @TaskQueue(type: TaskQueueType.serialBackgroundThread)
  int add(int x, int y);
}
```

### 三端全局配置表

```dart
@Configure()
class Api2Host {
   String test = '123456';
   int test1 = 100;
   double test2 = 0.123;
   List<int> test3 = <int>[1, 2, 3, 4, 5];
   List<String> test4 = <String>['1', '2', '3'];
}
```
自动生成三端代码
```dart
@Configure()
class Api2Host {
   String test = '123456';
   int test1 = 100;
   double test2 = 0.123;
   List<int> test3 = <int>[1, 2, 3, 4, 5];
   List<String> test4 = <String>['1', '2', '3'];
}
```
```java
 public class Info {
   public static final String test = '123456';
   public static final Long test1 = 100;
   public static final Double test2 = 0.123;
   public static final List<Long> test3 = <int>[1, 2, 3, 4, 5];
   public static final List<String> test4 = <String>['1', '2', '3'];
}
```
```objectivec
@protocol Info
NSString *const test = @'123456';
NSNumber *const test1 = @100;
NSNumber *const test2 = @0.123;
NSArray<NSNumber *> *const test3 = @<int>[1, 2, 3, 4, 5];
NSArray<NSString *> *const test4 = @<String>['1', '2', '3'];
```


## Feedback

File an issue in [flutter/flutter](https://github.com/flutter/flutter) with the
word 'pigeon' clearly in the title and cc **@gaaclarke**.
