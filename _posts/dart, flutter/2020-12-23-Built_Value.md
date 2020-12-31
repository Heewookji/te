---
title: Built_Value
layout: post
subtitle: 빌트발루를 이용해 만든 모델로 serialization 로직을 자동화하자
tags:
- Dart
header-style: text
---

## 빌트 발루에 대해서

> Built_Value를 사용하면 다음을 얻게 된다.
> - 불변(Immutable) 객체
> - EnumClass(enums처럼 동작하는 클래스)
> - JSON serialization 

최근 사이드 프로젝트에서 모델들을 만들고, json 형식으로 serialize, deserialize를 하면서
자동화된 패키지가 없나 찾아보다가 발견한 패키지(flutter favorite)이다. 
빌트발루는 value type처럼 동작하는 클래스 생성에 도움을 주는 패키지이고, 
따라서 빌트발루를 이용해 만들어진 객체는 불변이다. 이처럼 serialization 로직을 자동화하는 목적
이상의 패키지이지만 여기서는 serialization에 집중해서 정리하고자 한다.

관련내용과 추가정보 : <https://pub.dev/packages/built_value>

코드예제와 내용 원문 : <https://medium.com/flutter/some-options-for-deserializing-json-with-flutter-7481325a4450>

빌트발루보다 가벼운 json_serializable 패키지에 대한 정보 : <https://flutter-ko.dev/docs/development/data-and-backend/json>

## Json 데이터 변환

서버에서 받은 리스폰스의 데이터(주로 Json string)를 위젯에서 사용하기 전
데이터에 대한 몇 가지 작업들을 해야한다.
먼저 스트링을 parse해 관리에 용이한 json 형식으로 만들어야하고, 그 다음 모델로 변환해야한다.
모델로 변환할 때에는 여러 가지 방법이 있고, Built_value는 그 방법 중 하나를 제공한다.

서버에서 받아온 다음 json 포맷에 대해 직접 구현하는 방식과, 
Built_value(이하 빌트발루)를 이용해 구현하는 방법을 알아본다.

```json
{
  "aString": "Blah, blah, blah.",
  "anInt": 1,
  "aDouble": 1.0,
  "aListOfStrings": ["one", "two", "three"],
  "aListOfInts": [1, 2, 3],
  "aListOfDoubles": [1.0, 2.0, 3.0]
}
```

### 직접 구현하는 방식

```dart
class SimpleObject {
  const SimpleObject({
    this.aString,
    this.anInt,
    this.aDouble,
    this.aListOfStrings,
    this.aListOfInts,
    this.aListOfDoubles,
  });

  final String aString;
  final int anInt;
  final double aDouble;
  final List<String> aListOfStrings;
  final List<int> aListOfInts;
  final List<double> aListOfDoubles;

  factory SimpleObject.fromJson(Map<String, dynamic> json) {
    if (json == null) {
      throw FormatException("Null JSON provided to SimpleObject");
    }
    
    return SimpleObject(
        aString: json['aString'],
        anInt: json['anInt'],
        aDouble: json['aDouble'],
        aListOfStrings: json['aListOfStrings'] != null
            ? List<String>.from(json['aListOfStrings'])
            : null,
        aListOfInts: json['aListOfInts'] != null
            ? List<int>.from(json['aListOfInts'])
            : null,
        aListOfDoubles: json['aListOfDoubles'] != null
            ? List<double>.from(json['aListOfDoubles'])
            : null,
    );
  }
}
```

```dart
final myObject = SimpleObject.fromJson(json.decode(aJsonString));
```
위처럼 fromJson이라는 named factory 생성자를 이용하여 구현한다.
처음에 구현할 때 했던 방식인데, 손이 많이 가고 점점 관리하기가 어려웠다.
만약 많은 수의 모델이 있고, 직접 구현해야한다면 끔찍할 것이라 생각했다.

### built_value로 구현하는 방식

#### setting

built_value는 소스코드를 자동생성하는 접근법을 가진다. 따라서 빌트발루를 사용하기 위해서는 우선
pubspec.yaml 파일에 빌트발루뿐 아니라 build_runner와 built_value_generator라는
패키지를 dev_dependencies로 추가해야한다.

```yaml
dependencies:
  flutter:
    sdk: flutter
  built_value: ^7.1.0
  ...
dev_dependencies:
  flutter_test:
    sdk: flutter
  build_runner: ^1.0.0
  built_value_generator: ^7.1.0
```

#### 모델 생성

이후 SimpleObject.dart 파일을 생성하여 필요한 built_value 클래스들을 import하고
정의할 프로퍼티 등에 대해 작성한다.

```dart
import 'package:built_collection/built_collection.dart';
import 'package:built_value/built_value.dart';
import 'package:built_value/serializer.dart';

part 'simple_object.g.dart';

abstract class SimpleObject
    implements Built<SimpleObject, SimpleObjectBuilder> {
  static Serializer<SimpleObject> get serializer =>
      _$SimpleObjectSerializer;

  @nullable
  String get aString;

  @nullable
  int get anInt;

  @nullable
  double get aDouble;

  @nullable
  BuiltList<String> get aListOfStrings;

  @nullable
  BuiltList<int> get aListOfInts;

  @nullable
  BuiltList<double> get aListOfDoubles;

  SimpleObject._();

  factory SimpleObject([updates(SimpleObjectBuilder b)]) =
      _$SimpleObject;
}
```

#### source generation

위처럼 주어진 형식에 맞게 abstract class를 만들고(형식에 대한 자세한 내용, 튜토리얼 등은 위에 링크된 패키지 
주소에 잘 설명되어있음) pubspec.yaml 파일이 있는 곳에서 아래 명령어를 실행하면
위 part에 적어놓은 simple_object.g.dart 파일이 생성된다.

> flutter packages pub run build_runner build

이렇게 생성된 simple_object.g.dart 파일 안에는 SimpleObject를 extend 하는 _$SimpleObject 클래스가 있다.
이 _$SimpleObject 클래스는 해쉬코드, 불변성 관련 메서드, toString 등등에 대한 많은 기능을 제공한다.
그리고 SimpleObject에 대한 새 인스턴스를 생성할 때마다 실제로는 _$SimpleObject 인스턴스를 받는다.
새 인스턴스를 생성할 때에는 다음과 같이 Builder를 이용하여 생성한다.

```dart
final SimpleObject myObject = SimpleObject((b) => b
  ..aString = 'Blah, blah, blah'
  ..anInt = 1
  ..aDouble = 1.0
  ..aListOfStrings = ListBuilder<String>(['one', 'two', 'three'])
  ..aListOfInts = ListBuilder<int>([1, 2, 3])
  ..aListOfDoubles = ListBuilder<double>([1.0, 2.0, 3.0])
);
```
SimpleObject의 기존 생성자는 underscore(_) 처리로 private이 되었으므로, 사용자는 factory 생성자를
이용하도록 제한받고 결과적으로 SimpleObject의 인스턴스가 직접적으로 인스턴스화되지 않음을 보장한다.

#### serializer setting

소스코드가 자동 생성되었지만, deserialization을 구현하기 위해서는 몇 가지 작업이 남았다.
먼저 앱 어딘가에 serializers.dart 파일을 하나 생성하고 다음과 같이 작성한다.

```dart
import 'package:built_collection/built_collection.dart';
import 'package:built_value/serializer.dart';
import 'package:built_value/standard_json_plugin.dart';
import 'simple_object.dart';

part 'serializers.g.dart';

@SerializersFor(const [
  SimpleObject,
])

final Serializers serializers =
    (_$serializers.toBuilder()..addPlugin(StandardJsonPlugin())).build();
```

이 파일은 두 가지 일을 한다. 
1. @SerializersFor 어노테이션을 통해 리스트에 있는 Class들을 위한 serializer를 생성하도록 빌트발루에게 알려준다.
2. 빌트발루 모델 클래스들의 serialization을 담당하는 Serializers 객체에 대한 serializers라는 전역 변수를 생성한다.

빌트발루는 기본적으로 map-based JSON format을 사용하지 않고, list-based format을 사용한다.
아래는 두 가지 포맷의 차이.

```json
//list-based format
[
  "SimpleObject",
  "aString",
  "Blah, blah, blah",
  "anInt",
  1,
  "aDouble",
  2.0
]
//map-based format
{
  "$": "SimpleObject",
  "aString": "Blah, blah, blah",
  "anInt": 1,
  "aDouble": 2.0
}
```

때문에 주로 쓰는 map-based JSON format을 사용하기 위해서는 serializers.dart 파일의 맨 아래에
나와있듯이 다음과 같이 플러그인을 부착해야한다.

```dart
final Serializers serializers =
    (_$serializers.toBuilder()..addPlugin(StandardJsonPlugin())).build();
```

이제 작성을 완료한 뒤, 다시 명령어를 이용하여 소스 제네레이션을 하면, serializers.g.dart 파일이 생성된다.

#### deserialization

이제 데이터를 받아와서 deserialization을 하려면 아래와 같이 사용하면 된다.

```dart
final myObject = serializers.deserializeWith(
    SimpleObject.serializer, json.decode(aJsonString));
```


