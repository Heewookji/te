---
title: AnimatedWidget, ClipPath - 애니메이션 구현
layout: post
subtitle: ClipPath를 이용한 애니메이션을 구현해보자
tags:
  - flutter
header-style: text
---

최근 사이드 프로젝트로 만들고 있는 플러터 앱에서 누른 위치에 원이 커지면서 화면을 가득 채우는 애니메이션을 
구현해야 할 일이 생겼다. 해당 목적을 위해서 여러가지로 알아보다가, AnimatedWidget과 ClipPath를 이용하여 구현하였다.

![result](/img/in-post/flutter/animation/result.gif){: width="250" height="600"}

구현을 위해 플러터 애니메이션 공식 youtube 채널 동영상을 참고하여 진행하였다.
<https://www.youtube.com/watch?v=GXIJJkq_H8g&list=PLjxrf2q8roU2v6UqYlt_KPaXlnjbYySua>


## 애니메이션 구현 방식 결정

먼저, 플러터에서 제공하는 다양한 방법들 중 하나를 결정해야했다. 결정을 위해서 아래 로직표를 참고하였다. 
목표로 하는 애니메이션은 사용자가 버튼을 탭할 때마다 forward 한 뒤 reset하고 다시 forward 하는 식으로 구현되어야했다.
따라서 내가 생각하기에는 implicit보다는 explicit 쪽의 방식이 맞아보였고, 누른 위치에서 애니메이션이 시작되어야하니
AnimatedBuilder나 AnimatedWidget이 맞아보였다. 마지막으로 재사용을 위해 AnimatedWidget을 사용하기로 결정!

![logic](/img/in-post/flutter/animation/logic.png)


## 구현

StateManagement가 필요하기 때문에 Stateful 위젯으로 MyHomePage를 만들어준다. 그 다음 Scaffold의 body에
Stack을 놓고, 버튼 3개를 Row로 배치한다. Stack을 이용하여 버튼 아래에 애니메이션이 구현될 배경을 깔아놓을 것이다.

```dart
class MyHomePage extends StatefulWidget {
  @override
  _MyHomePageState createState() => _MyHomePageState();
}
class _MyHomePageState extends State<MyHomePage> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Stack(
        children: [
          Center(
            child: Row(
              children: [
                GestureDetector(
                  child: Icon(
                    Icons.play_arrow,
                    color: Colors.cyan,
                    size: 60,
                  ),
                ),
                GestureDetector(
                  child: Icon(
                    Icons.play_arrow,
                    color: Colors.red,
                    size: 60,
                  ),
                ),
                GestureDetector(
                  child: Icon(
                    Icons.play_arrow,
                    color: Colors.amber,
                    size: 60,
                  ),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}
```

![image](/img/in-post/flutter/animation/1.png){: width="250" height="300" style="flow"}

애니메이션을 위해 `AnimationController`와 `Animation`을 설정해준다. `AnimationController`를 만들기 위해
`_MyHomePageState` 클래스에 `TickerProviderStateMixin`를 mixin 해준다. 
initState에서는 `AnimationController`의 duration과 vsync, `Animation`의 parent와 curve를 설정해준다.
애니메이션 실행 시 원의 크기가 변화되도록 `Tween<double>`을 이용해 0부터 1.0까지의 value를 주기로 하였다.
이렇게 되면 parent인 _controller의 Duration(600밀리초)동안, Curve에 따라 변화된 double value를 받는다.
또한 lifecycle 자원 관리를 위해 controller.dispose를 호출해준다.

다음 링크에서 curve들의 모션을 참고해볼 수 있다.
<https://api.flutter.dev/flutter/animation/Curves-class.html>


```dart
class _MyHomePageState extends State<MyHomePage> with TickerProviderStateMixin {
  AnimationController _controller;
  Animation<double> _animation;

  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: Duration(milliseconds: 600),
      vsync: this,
    );
    _animation = Tween<double>(begin: 0, end: 1.0).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.easeInOutQuart,
      ),
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }
```

이제 애니메이션에 따라 그려질 위젯 클래스를 만들어준다. `CustomFillAnimation`에서는 `AnimatedWidget`을 상속하고
super에게 listenable객체(animation)를 그대로 넘기면서 build 메서드에서는 animation의 `value`를
이용해 그릴 위젯을 리턴한다. 일단은 간단하게 value에 따라 높이가 커지는 빨간 원을 그리도록 한다.
얼마나 커질 것인지 정해주기 위해 생성자로 Size를 넣어주었다.

```dart
class CustomFillAnimation extends AnimatedWidget {
  final Size _size;
  final Animation<double> _animation;

  CustomFillAnimation(
    this._size,
    this._animation,
  ) : super(listenable: _animation);

  @override
  Widget build(BuildContext context) {
    return Container(
      height: _animation.value * _size.height,
      decoration: BoxDecoration(color: Colors.red, shape: BoxShape.circle),
    );
  }
}
```

이제 다시 `_MyHomePageState`로 돌아와, Stack의 맨 밑에 방금 만든 `CustomFillAnimation`를 깔아준다.
`MediaQuery`를 이용해 핸드폰 화면의 size를 넘겨주고, 아까 설정해둔 `_animation`도 넣어준다.

```dart
@override
  Widget build(BuildContext context) {
    Size _screenSize = MediaQuery.of(context).size;
    return Scaffold(
      body: Stack(
        children: [
          CustomFillAnimation(
            _screenSize,
            _animation,
          ),
          ...
```

버튼을 누르면 애니메이션을 진행할 수 있도록 _onTap 메서드를 만들어준다. 간단하게 forward만 실행한다.


```dart
void _onTap(Color color, Offset tappedLocation) async {
    await _controller.forward();
}
```

3개의 `GestureDetector`에 `onTapUp` 프로퍼티로 방금 만든 _onTap 메서드를 넣어준다.

```dart
GestureDetector(
  child: Icon(
    Icons.play_arrow,
    color: Colors.cyan,
    size: 60,
  ),
  onTapUp: (detail) => _onTap(),
),
```

![image](/img/in-post/flutter/animation/2.gif){: width="250" height="600"}

이제 버튼을 누르면 위와 같은 결과를 볼 수 있는데, 몇 가지 문제가 보인다. 우선 계속 forward 할 수
있도록 reset 구문을 _onTap 메서드에 추가하고, 해당 버튼의 색과 누른 위치를 `CustomFillAnimation`에서
사용할 수 있도록 만들어줘야한다.

```dart
class _MyHomePageState extends State<MyHomePage> with TickerProviderStateMixin {
  AnimationController _controller;
  Animation<double> _animation;
  Color _pickedColor;
  Offset _pickedLocation;

  @override
  void initState() {
    ...
  }

  @override
  void dispose() {
    ...
  }

  void _onTap(Color color, Offset pickedLocation) async {
    _controller.reset();
    setState(() {
      _pickedColor = color;
      _pickedLocation = pickedLocation;
    });
    await _controller.forward();
  }

  @override
  Widget build(BuildContext context) {
    Size _screenSize = MediaQuery.of(context).size;
    return Scaffold(
      body: Stack(
        children: [
          CustomFillAnimation(
            _screenSize,
            _animation,
            _pickedColor,
            _pickedLocation,
          ),
          Center(
            child: Row(
              children: [
                GestureDetector(
                  child: Icon(
                    Icons.play_arrow,
                    color: Colors.cyan,
                    size: 60,
                  ),
                  onTapUp: (detail) =>
                      _onTap(Colors.cyan, detail.globalPosition),
                ),
                GestureDetector(
                  child: Icon(
                    Icons.play_arrow,
                    color: Colors.red,
                    size: 60,
                  ),
                  onTapUp: (detail) =>
                      _onTap(Colors.red, detail.globalPosition),
                ),
                GestureDetector(
                  child: Icon(
                    Icons.play_arrow,
                    color: Colors.amber,
                    size: 60,
                  ),
                  onTapUp: (detail) =>
                      _onTap(Colors.amber, detail.globalPosition),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

class CustomFillAnimation extends AnimatedWidget {
  final Size _size;
  final Color _color;
  final Offset _location;
  final Animation<double> _animation;

  CustomFillAnimation(
    this._size,
    this._animation,
    this._color,
    this._location,
  ) : super(listenable: _animation);

  @override
  Widget build(BuildContext context) {
    return Container(
      height: _animation.value * _size.height,
      decoration: BoxDecoration(color: _color, shape: BoxShape.circle),
    );
  }
}
```

![image](/img/in-post/flutter/animation/3.gif){: width="250" height="600"}

이제 위와 같은 결과물이 나오는데, 누른 위치를 넘겨줘도 `CustomFillAnimation`에서 못쓰고 있다.
`Positioned`를 이용하면 어떻게 될 것 같지만 복잡해질 것 같았다. 그리고 Container를 원으로 만들어서
screen의 높이만큼을 지름으로 하도록 했는데, 아무래도 screen을 넘어서 커질 수는 없는 것 같다.
따라서, 애니메이션에 따라 직접 그리기로 결정하였다. 이번에는 `CustomPainter`말고 `CustomClipper`를 
이용해보기로 하였다. 

이제 `CustomFillAnimation`에서는 `ClipPath`를 리턴한다. `ClipPath`에서는 child로 Clip할
Widget을 받는다. 여기에 스크린 크기의 `Container`를 child로 넘겨주었고, clipper 프로퍼티에 넣어준
`MyCustomClipper`가 이 `Container`를 잘라주게 된다.

```dart
class CustomFillAnimation extends AnimatedWidget {
 ...

  @override
  Widget build(BuildContext context) {
    return ClipPath(
      clipper: MyCustomClipper(
        _location,
        _animation.value,
      ),
      child: Container(
        width: _size.width,
        height: _size.height,
        color: _color,
      ),
    );
  }
}
```

`MyCustomClipper<Path>`에서는 _location(누른 위치)와 _value(애니메이션 값)를 받는다.
getClip 메서드에서 Path를 리턴해야하는데, 먼저 Path를 만들고 원을 추가해준다. 중심은 _location이
되면서, 반지름은 _value에 따라서 최대 size.height / 1.5가 된다. 반지름으로 해주려면 2로 나누면 되지만
1.5로 나눈 이유는 클릭한 위치에 따라서 원이 화면을 다 못 가리는 일이 생겼기 때문이다.

```dart
class MyCustomClipper extends CustomClipper<Path> {
  final Offset _location;
  final double _value;

  MyCustomClipper(this._location, this._value);

  @override
  Path getClip(Size size) {
    final path = Path();
    path.addOval(
      Rect.fromCircle(
        center: _location ?? Offset(0, 0),
        radius: (size.height * _value) / 1.5,
      ),
    );
    return path;
  }

  @override
  bool shouldReclip(covariant CustomClipper<Path> oldClipper) => true;
}
```

![image](/img/in-post/flutter/animation/4.gif){: width="250" height="600"}

이제 위와 같이 나오는데, 버튼을 누를 때마다 원이 다시 0부터 커지는 것이므로 바탕에는 흰색이 보인다.
Stack의 맨 아래에 Container를 하나 더 깔고, 이전에 눌렀던 색상을 저장할 변수 _pastColor를 
`_MyHomePageState`에 추가해 Container의 color로 준 뒤, _onTap메서드에서 
forward 이후 저장하게 해주면 최종 결과물이 된다.

### 최종 코드

```dart
import 'dart:ui';
import 'package:flutter/material.dart';

class MyHomePage extends StatefulWidget {
  @override
  _MyHomePageState createState() => _MyHomePageState();
}

class _MyHomePageState extends State<MyHomePage> with TickerProviderStateMixin {
  AnimationController _controller;
  Animation<double> _animation;
  Color _pickedColor;
  Offset _pickedLocation;
  Color _pastColor;
  @override
  void initState() {
    super.initState();
    _controller = AnimationController(
      duration: Duration(milliseconds: 600),
      vsync: this,
    );
    _animation = Tween<double>(begin: 0, end: 1.0).animate(
      CurvedAnimation(
        parent: _controller,
        curve: Curves.easeInOutQuart,
      ),
    );
  }

  @override
  void dispose() {
    _controller.dispose();
    super.dispose();
  }

  void _onTap(Color color, Offset pickedLocation) async {
    if (_controller.isAnimating) return;
    _controller.reset();
    setState(() {
      _pickedColor = color;
      _pickedLocation = pickedLocation;
    });
    await _controller.forward();
    _pastColor = color;
  }

  @override
  Widget build(BuildContext context) {
    Size _screenSize = MediaQuery.of(context).size;
    return Scaffold(
      body: Stack(
        children: [
          Container(color: _pastColor),
          CustomFillAnimation(
            _screenSize,
            _animation,
            _pickedColor,
            _pickedLocation,
          ),
          Center(
            child: Row(
              children: [
                GestureDetector(
                  child: Icon(
                    Icons.play_arrow,
                    color: Colors.cyan,
                    size: 60,
                  ),
                  onTapUp: (detail) =>
                      _onTap(Colors.cyan, detail.globalPosition),
                ),
                GestureDetector(
                  child: Icon(
                    Icons.play_arrow,
                    color: Colors.red,
                    size: 60,
                  ),
                  onTapUp: (detail) =>
                      _onTap(Colors.red, detail.globalPosition),
                ),
                GestureDetector(
                  child: Icon(
                    Icons.play_arrow,
                    color: Colors.amber,
                    size: 60,
                  ),
                  onTapUp: (detail) =>
                      _onTap(Colors.amber, detail.globalPosition),
                ),
              ],
            ),
          ),
        ],
      ),
    );
  }
}

class CustomFillAnimation extends AnimatedWidget {
  final Size _size;
  final Color _color;
  final Offset _location;
  final Animation<double> _animation;

  CustomFillAnimation(
    this._size,
    this._animation,
    this._color,
    this._location,
  ) : super(listenable: _animation);

  @override
  Widget build(BuildContext context) {
    return ClipPath(
      clipper: MyCustomClipper(
        _location,
        _animation.value,
      ),
      child: Container(
        width: _size.width,
        height: _size.height,
        color: _color,
      ),
    );
  }
}

class MyCustomClipper extends CustomClipper<Path> {
  final Offset _location;
  final double _value;

  MyCustomClipper(this._location, this._value);

  @override
  Path getClip(Size size) {
    final path = Path();
    path.addOval(
      Rect.fromCircle(
        center: _location ?? Offset(0, 0),
        radius: (size.height * _value) / 1.5,
      ),
    );
    return path;
  }

  @override
  bool shouldReclip(covariant CustomClipper<Path> oldClipper) => true;
}
```


