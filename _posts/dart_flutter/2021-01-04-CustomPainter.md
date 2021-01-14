---
title: CustomPainter, Path - 말풍선 그리기
layout: post
subtitle: message bubble을 CustomPainter와 Path를 이용해 그려보자 
tags:
  - flutter
header-style: text
---

최근 사이드 프로젝트로 만들고 있는 플러터 앱에서 말풍선을 만들 일이 생겼다. 이미 나와있는 라이브러리가
있을 것 같지만 이 기회에 직접 그리고 싶어 `CustomPaint`와 `CustomPainter`, `Path`를 이용해 구현해보았다.
최종적으로 구현되는 말풍선의 형태는 아래와 같다.

![result](/img/in-post/flutter/custom_painter/result.png){: width="250" height="600"}

`CustomPainter`와 `Path`에 대한 단순 예제 및 동작 설명은 아래 참고에서 굉장히 잘 설명되어있다.
<https://medium.com/flutter-community/paths-in-flutter-a-visual-guide-6c906464dcd0>


## 구현

먼저 message bubble을 보여줄 화면 MyHomePage를 main.dart에 홈으로 설정해준다.

```dart
import 'package:flutter/material.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      debugShowCheckedModeBanner: false,
      home: MyHomePage(),
    );
  }
}
```

MyHomePage의 body에서는 앞으로 만들 말풍선을 CustomPaint 위젯을 이용하여 출력한다.
이 때 CustomPaint 위젯이 paint 할 캔버스 사이즈를 알 수 있도록 Container의 height와 width를 지정해준다.

```dart
class MyHomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    Size screenSize = MediaQuery.of(context).size;
    return Scaffold(
      appBar: AppBar(
        title: Text('Message Bubble Page'),
      ),
      body: Container(
        padding: EdgeInsets.all(screenSize.width * 0.05),
        height: screenSize.height * 0.25,
        width: screenSize.width,
        child: CustomPaint(
          painter: BubblePainter(),
        ),
      ),
    );
  }
}
```

`CustomPaint` 위젯에서는 실제로 paint를 할 painter를 받아야한다. 해당 BubblePainter를 만들어준다.

```dart
class BubblePainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    //paint something
  }
  @override
  bool shouldRepaint(CustomPainter oldDelegate) => false;
}
```

painter에는 `CustomPainter`를 상속한(extends) Painter 클래스가 와야한다.
상속한 뒤, paint와 shouldRepaint 메서드를 오버라이딩 해야한다.
paint 메서드에서는 주어진 canvas, size와 함께 그리는 로직을 수행하면 되고, shouldRepaint 메서드에서는 
받은 값에 따라서 다시 그려야하는지 `true`, `false`를 리턴해주면 된다. 이 예제에서는 항상 동일한 형태의
message bubble을 그리므로 `false`를 리턴한다. 아직 아무것도 그리지 않으므로 아래와 같이 출력된다.

![result](/img/in-post/flutter/custom_painter/1.png){: width="250" height="600"}


```dart
  @override
  void paint(Canvas canvas, Size size) {
    final bubbleSize = Size(size.width, size.height * 0.8);
    final tailSize = Size(size.width * 0.1, size.height - bubbleSize.height);
    final fillet = bubbleSize.width * 0.1;
    final tailStartPoint = Point(size.width * 0.7, bubbleSize.height);
  }
```

그리기 전, 주어진 캔버스 사이즈에서 말풍선 바디의 사이즈와 꼬리 사이즈를 정의한다.
또한 말풍선의 모서리 부분에 대한 fillet 값을 정의한다. 마지막으로 꼬리 시작 부분을 정의한다.
다음으로는 캔버스에 무엇을 그릴지 결정한다. `Canvas`의 메서드를 보면 어떤 것을 그릴 수 있는지 알 수 있다.
`drawCircle`, `drawImage` 등등 많지만 Path를 이용하여 구현해보도록 한다.

```dart
  @override
  void paint(Canvas canvas, Size size) {
    final bubbleSize = Size(size.width, size.height * 0.8);
    final tailSize = Size(size.width * 0.1, size.height - bubbleSize.height);
    final fillet = bubbleSize.width * 0.1;
    final tailStartPoint = Point(size.width * 0.7, bubbleSize.height);
    //bubble body
    final bubblePath = Path()
      ..moveTo(0, fillet)
      // 왼쪽 위에서 왼쪽 아래 라인
      ..lineTo(0, bubbleSize.height - fillet);
    // paint setting
    final paint = Paint()
      ..color = Color(0xffFCA311)
      ..strokeWidth = 2
      ..style = PaintingStyle.stroke;
    // draw
    canvas.drawPath(bubblePath, paint);
  }
```
![image](/img/in-post/flutter/custom_painter/2.png){: width="250" height="300" style="flow"}

주어진 canvas에서 Path는 왼쪽 위 (0,0)부터 시작하고 커질수록 x는 오른쪽, y는 아래쪽으로 이동한다.
Path를 하나 만들고, `moveTo` 메서드를 이용하여 (0, fillet)인 점으로 이동한다.
그 다음 (cascade operator ..를 이용하여) `lineTo` 메서드로 왼쪽 위에서 왼쪽 아래로 선을 긋는다.
그 뒤에 실제로 출력하기 위해 Paint를 만들고 color, style 등을 세팅하여 canvas.drawPath 메서드에 path와 같이 던져준다.

![image](/img/in-post/flutter/custom_painter/3.png){: width="250" height="600"}

사진처럼 왼쪽 위부터 아래로 라인이 그려진 것을 확인할 수 있다.

```dart
  //bubble body
  final bubblePath = Path()
    ..moveTo(0, fillet)
    // 왼쪽 위에서 왼쪽 아래 라인
    ..lineTo(0, bubbleSize.height - fillet)
    ..quadraticBezierTo(0, bubbleSize.height, fillet, bubbleSize.height);
```

![image](/img/in-post/flutter/custom_painter/quadraticBezierTo.gif){: width="250" height="600"} *quadraticBezierTo - Source: Wikipedia*

`lineTo` 메서드 뒤에 모서리를 둥글게 말기 위해 `quadraticBezierTo` 메서드를 붙여준다. 이 메서드는 4개의 double 값을 받는데,
(x, y)점 두 개를 주는 것이다. 첫 번째 점은 `제어점`이고, 두 번째 점은 `목적지 점`이다. 이렇게 두 개의 점을 넘겨주면 현재 시작점과 
목적지 점 사이를 제어점을 이용하여 커브로 이어준다.



![image](/img/in-post/flutter/custom_painter/4.png){: width="250" height="600"}
남은 부분들도 마찬가지로 호다닥 구현하면 다음과 같다.

```dart
//bubble body
final bubblePath = Path()
  ..moveTo(0, fillet)
  // 왼쪽 위에서 왼쪽 아래 라인
  ..lineTo(0, bubbleSize.height - fillet)
  ..quadraticBezierTo(0, bubbleSize.height, fillet, bubbleSize.height)
  // 왼쪽 아래에서 오른쪽 아래 라인
  ..lineTo(bubbleSize.width - fillet, bubbleSize.height)
  ..quadraticBezierTo(bubbleSize.width, bubbleSize.height, bubbleSize.width,
      bubbleSize.height - fillet)
  // 오른쪽 아래에서 오른쪽 위 라인
  ..lineTo(bubbleSize.width, fillet)
  ..quadraticBezierTo(bubbleSize.width, 0, bubbleSize.width - fillet, 0)
  // 오른쪽 위에서 왼쪽 아래 라인
  ..lineTo(fillet, 0)
  ..quadraticBezierTo(0, 0, 0, fillet);
```

![image](/img/in-post/flutter/custom_painter/5.png){: width="250" height="600"}

말풍선 몸체는 완성되었으니, 꼬리를 작업하도록 한다. bubblePath 밑에 tailPath를 만들어주고,
moveTo 메서드로 앞서 정의한 `tailStartPoint` 점으로 간다.

```dart
// bubble tail
    final tailPath = Path(
      ..moveTo(tailStartPoint.x, tailStartPoint.y);
```

![image](/img/in-post/flutter/custom_painter/cubicTo.gif){: width="250" height="600"} *cubicTo - Source: Wikipedia*

이제 꼬리 곡선을 그려주면 되는데, `tailStartPoint`에서 시작하여 `cubicTo` 메서드를 2번 사용해 그려주었다.
`cubicTo` 메서드는 6개의 double 파라미터를 받는다. 6개의 double 값은 3개의 점을 의미하고 마지막 값은 목적지 점을,
나머지 2개는 제어점이다. 곡선을 그리는 다양한 방법이 있는데, 맨 위의 참조 링크에서 잘 설명되어있다.

```dart
    // bubble tail
    final tailPath = Path()
      ..moveTo(tailStartPoint.x, tailStartPoint.y)
      ..cubicTo(
        tailStartPoint.x + (tailSize.width * 0.2),
        tailStartPoint.y,
        tailStartPoint.x + (tailSize.width * 0.6),
        tailStartPoint.y + (tailSize.height * 0.2),
        tailStartPoint.x + tailSize.width / 2, // 목적지 x
        tailStartPoint.y + tailSize.height, // 목적지 y
      )
      ..cubicTo(
        (tailStartPoint.x + tailSize.width / 2) + (tailSize.width * 0.2),
        tailStartPoint.y + tailSize.height,
        tailStartPoint.x + tailSize.width,
        tailStartPoint.y + (tailSize.height * 0.3),
        tailStartPoint.x + tailSize.width, // 목적지 x
        tailStartPoint.y, // 목적지 y
      );
    // add tail to bubble body
    bubblePath.addPath(tailPath, Offset(0, 0));
```
첫 번째 곡선은 (꼬리 넓이의 반, 꼬리 높이의 끝)이 `목적지 점`이고, 두 번째 곡선은 
(꼬리 넓이의 끝, 꼬리 높이의 처음)이 `목적지 점`이며 나머지는 모두 세세한 `제어점` 조정 값이다.
이렇게 해서 tailPath를 마무리하고, 아까 만들었던 bubblePath에 addPath 메서드를 이용해
붙여준다. offset은 필요없으므로 0,0 고정. 이러면 아래와 같이 나온다. 

![image](/img/in-post/flutter/custom_painter/6.png){: width="250" height="600"}

마지막으로 이제 stroke는 필요없으므로 paint style을 fill로 바꿔주면 최종 형태가 된다.

### 최종 코드

```dart
import 'dart:math';
import 'dart:ui';
import 'package:flutter/material.dart';

void main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      debugShowCheckedModeBanner: false,
      home: MyHomePage(),
    );
  }
}

class MyHomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    Size screenSize = MediaQuery.of(context).size;
    return Scaffold(
      appBar: AppBar(
        title: Text('Message Bubble Page'),
      ),
      body: Container(
        padding: EdgeInsets.all(screenSize.width * 0.05),
        height: screenSize.height * 0.25,
        width: screenSize.width,
        child: CustomPaint(
          painter: BubblePainter(),
        ),
      ),
    );
  }
}

class BubblePainter extends CustomPainter {
  @override
  void paint(Canvas canvas, Size size) {
    final bubbleSize = Size(size.width, size.height * 0.8);
    final tailSize = Size(size.width * 0.1, size.height - bubbleSize.height);
    final fillet = bubbleSize.width * 0.1;
    final tailStartPoint = Point(size.width * 0.7, bubbleSize.height);
    //bubble body
    final bubblePath = Path()
      ..moveTo(0, fillet)
      // 왼쪽 위에서 왼쪽 아래 라인
      ..lineTo(0, bubbleSize.height - fillet)
      ..quadraticBezierTo(0, bubbleSize.height, fillet, bubbleSize.height)
      // 왼쪽 아래에서 오른쪽 아래 라인
      ..lineTo(bubbleSize.width - fillet, bubbleSize.height)
      ..quadraticBezierTo(bubbleSize.width, bubbleSize.height, bubbleSize.width,
          bubbleSize.height - fillet)
      // 오른쪽 아래에서 오른쪽 위 라인
      ..lineTo(bubbleSize.width, fillet)
      ..quadraticBezierTo(bubbleSize.width, 0, bubbleSize.width - fillet, 0)
      // 오른쪽 위에서 왼쪽 아래 라인
      ..lineTo(fillet, 0)
      ..quadraticBezierTo(0, 0, 0, fillet);
    // bubble tail
    final tailPath = Path()
      ..moveTo(tailStartPoint.x, tailStartPoint.y)
      ..cubicTo(
        tailStartPoint.x + (tailSize.width * 0.2),
        tailStartPoint.y,
        tailStartPoint.x + (tailSize.width * 0.6),
        tailStartPoint.y + (tailSize.height * 0.2),
        tailStartPoint.x + tailSize.width / 2, // 목적지 x
        tailStartPoint.y + tailSize.height, // 목적지 y
      )
      ..cubicTo(
        (tailStartPoint.x + tailSize.width / 2) + (tailSize.width * 0.2),
        tailStartPoint.y + tailSize.height,
        tailStartPoint.x + tailSize.width,
        tailStartPoint.y + (tailSize.height * 0.3),
        tailStartPoint.x + tailSize.width, // 목적지 x
        tailStartPoint.y, // 목적지 y
      );
    // add tail to bubble body
    bubblePath.addPath(tailPath, Offset(0, 0));
    // paint setting
    final paint = Paint()
      ..color = Color(0xffFCA311)
      ..style = PaintingStyle.fill;
    // draw
    canvas.drawPath(bubblePath, paint);
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) => false;
}
```


