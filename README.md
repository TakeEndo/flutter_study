# flutter_study
Flutterで知っておきたい小ネタ（archive）

// 2020/03/27
# おそらくもう更新することはありません。

小ネタというかメモ。
おすすめ機能やら、検索でヒットしにくかったものなどを備忘録としてまとめることにしました。
※随時更新予定です
// 2018/09/23
ベジェ曲線について、追記しました。

// 2018/09/26
Transformについて、追記しました。

# 親要素(Widget)から子要素(Widget)にアクセスしたい
例えば、親画面側でボタン操作などがあり、そのタイミングで子側の関数を呼びたいとか。
そんな時は、GlobalKeyを使うと良いみたいです。

まずは、子クラス。

```Child.dart
class Child extends StatefulWidget{
  const Child({Key key}) : super(key: key);
  ChildState createState() => ChildState();
}

```

子のステートクラス。
必要な関数などを定義しておきます。

```ChildState.dart
class ChildState extends State<Child> {
  void callChild(String message){
    debugPrint(message);
  }

  @override
  Widget build(BuildContext context) {
    return new Container();
  });
}
```

呼び出し元（親）

```Parent.dart
//key
final childKey = new GlobalKey<ChildState>();

//親
class Parent extends State<MyHomePage> {
  // 例えば、onTapButtonみたなものを用意する
  void onTapButton(){
    // 子側で定義している関数を呼び出せます。
    childKey.currentState.callChild('call from Parent');
  }
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      body: new Child(key: childKey);
      floatingActionButton: FloatingActionButton(onPressed: onTapButton,),
    );
  });
}
```

関数だけでなく、変数へのアクセスも可能。
でも、間に色々Widgetが入ってくるケースがあってソースが大変なことになるので、できれば要素内で完結させたい。
GlobalKeyを使わなくても、普通にアクセスは可能。


# 子要素から親要素にアクセスしたい
逆のパターン。
これはあんまりベストプラクティスみたいなのが見つからず。
関数渡しみたいなことができるので、それをsetter/getterっぽく使えば良いかなと思いました。
それは良くないとか、もっと良い方法があるなど、ご意見いただきたく。

個人的な意見としては、VoidCallbackを渡すのはセーフですが、Functionを渡す実装にすると、引数も戻り値も見えないのず、バグの温床になりそうな気がして怖いです。
引数と戻り値の個数、型が異なっていると**実行時エラー**になります。

## 子側

```Child.dart
import 'package:flutter/material.dart';

class Child extends StatelessWidget{

  const Child({Key key, this.hoge, this.fuga}) : super(key: key);

  final VoidCallback hoge; // VoidCallback型の関数。
  final Function fuga; // 抽象的なFunctionも定義可能。

  // 一旦、void Functionで受けて、親の指定した関数を呼び出している。
  void _callFugaFuga(){
    debugPrint(fuga('child'));
  }

  @override
  Widget build(BuildContext context) {
    return new Container(
      child: new ListView(
          children: <Widget>[
            new RaisedButton(
              onPressed: hoge, // onPressedがVoidCallbackなので、hogeをそのまま渡す。
              child: new Text('hoge',),
            ),
            new RaisedButton(
              onPressed: _callFugaFuga, // 直接fugaを呼べない（実行時エラーになる）ので、void Functionを呼んでいる。
              child: new Text('fuga',),
            )]),
      margin: new EdgeInsets.all(10.0),
      height: 45.0,
    );
  }

}
```

## 親側

```Parent.dart
class _MyHomePageState extends State<MyHomePage> with TickerProviderStateMixin {
  void hogehoge(){
    debugPrint('hogehoge');
  }

  String message = "fugafuga";
  // Functionなので、引数も戻り値もなんでもOKです。
  String fugafuga(String fuga){
    return message + fuga;
  }

  @override
  Widget build(BuildContext context) {
    Child c = new Child(
      hoge: hogehoge,
      fuga: fugafuga,);
    return c;
  }
}
```

なんでもできる反面、結構怖い実装になりそう。

# ジェスチャーを取得したい
Flutterで開発していて、あまりにも簡単にできるから驚いた機能の一つ。

```GestureSample.dart
class GestureSample extends State<MyHomePage> {
  // Dragをハンドリングする
  void _onDrag(DragUpdateDetails drag){
    // 縦スクロールと、横スクロールは別々に扱われます。
    drag.globalPosition; // 画面のどの位置にいるか
    drag.delta; // 前回の呼び出しから、どれくらい移動したか
  }
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      body: GestureDetector(
        onVerticalDragUpdate: _onDrag, //　縦スクロール 
        onHorizontalDragUpdate: _onDrag, // 横スクロール
        child: Container();
      )
    );
  });
}

```

取得できるジェスチャー一覧。
最低限必要なものは揃っている気がします。
タップしている指の本数などを取得できないので、2本指タップなどは検知できません。

```GestureDetector.dart
    this.onTapDown,
    this.onTapUp,
    this.onTap,
    this.onTapCancel,
    this.onDoubleTap,
    this.onLongPress,
    this.onVerticalDragDown,
    this.onVerticalDragStart,
    this.onVerticalDragUpdate,
    this.onVerticalDragEnd,
    this.onVerticalDragCancel,
    this.onHorizontalDragDown,
    this.onHorizontalDragStart,
    this.onHorizontalDragUpdate,
    this.onHorizontalDragEnd,
    this.onHorizontalDragCancel,
    this.onPanDown,
    this.onPanStart,
    this.onPanUpdate,
    this.onPanEnd,
    this.onPanCancel,
    this.onScaleStart,
    this.onScaleUpdate,
    this.onScaleEnd,
```

# 画面サイズを取得する
これも意外と便利。
ググればすぐ出て来ますけど。

```hoge.dart
  @override
  Widget build(BuildContext context) {
    double width = MediaQuery.of(context).size.width;
    double height = MediaQuery.of(context).size.height;
```

ちなみに、buildのタイミングでは、contextのサイズが取れない。


# Textを上下左右真ん中にしたい
非常に地味ですが。
縦横が固定されたエリアの中央にテキストを配置したい場合。
```textAlign: TextAlign.center```だと、左右は真ん中になりますが、上下が真ん中に来ません。
一旦、Centerでラッピングすると、良い感じに真ん中に来ます。
```verticalTextAlign```みたいなものは無いらしい。

```CenterText.dart
new Container(
  width:200.0,
  height:200.0,
  child: Center(
    child: Text('message',
      textAlign: TextAlign.center,
    );
  );
);
```

# Containerの背景色にグラデーションをかけたい

## 1. 直線的に3段階の色に変化させるサンプル。

```LinearGradientSample.dart
  decoration: BoxDecoration(
    gradient: new LinearGradient(
      begin: Alignment.topCenter,
      end: Alignment.bottomCenter,
      stops: [0.2, 0.3, 1.0],
      colors: [
        Color.fromARGB(255, 0, 0, 0),
        Color.fromARGB(255, 128, 128, 128),
        Color.fromARGB(255, 255, 255, 255),
      ],
    ),
  ),
```
<img width="191" alt="スクリーンショット 2018-09-07 13.35.01.png" src="images/a75ed51e-57b4-ca02-0bb0-a616f1b89823.png">
```begin```と```end```には、それぞれ右上、右下などの```enum```を設定すれば角度を変更できる。


## 2. 円形のグラデーション

```RadialGradientSample.dart
  decoration: BoxDecoration(
    gradient:
      new RadialGradient(
        center: Alignment(-0.3, 0.3), // 円の中心座標
          radius: 0.2, // 円の大きさ
          tileMode: TileMode.clamp, //塗りつぶしモード
          colors: [
            Color.fromARGB(255, 0, 0, 0),
            Color.fromARGB(255, 128, 128, 128),
            Color.fromARGB(255, 255, 255, 255),]
      )

```

### TileModeでグラデーションの繰り返し方法を指定できる
#### .clamp
<img width="195" alt="スクリーンショット 2018-09-07 13.39.04.png" src="images/1642ccc8-9668-8caa-71be-b87cdef3131b.png">

#### .mirror
<img width="191" alt="スクリーンショット 2018-09-07 13.39.15.png" src="images/a25b8025-3b92-9b4f-0a98-07ac36550a27.png">

#### .repeated
<img width="194" alt="スクリーンショット 2018-09-07 13.38.47.png" src="images/19ef1077-1fca-dfcb-78df-9ed0d7899af1.png">
きもい。

## 3. Sweepグラデーション
こういうやつ
<img width="189" alt="スクリーンショット 2018-09-07 14.25.12.png" src="images/31826b29-c569-7f38-0797-5c39fbea0d65.png">

```SweepGradientSample.dart
  decoration: BoxDecoration(
    gradient:
      new SweepGradient(
        center: Alignment(-0.3, 0.3),
          startAngle: math.pi * 1,
          endAngle: math.pi * 1.2,
          tileMode: TileMode.repeated,
          colors: [
            Color.fromARGB(255, 0, 0, 0),
            Color.fromARGB(255, 128, 128, 128),
            Color.fromARGB(255, 255, 255, 255),]
      ),
```
### これもTileModeあります。
#### .mirror

<img width="192" alt="スクリーンショット 2018-09-07 14.30.45.png" src="images/bf77b4ab-8e96-444a-c56a-69c26aed63cc.png">
目がっ！目がぁっ！

#### .repeated
<img width="191" alt="スクリーンショット 2018-09-07 14.29.57.png" src="images/9505054b-3a7e-3552-f08e-43ea9bdf8ef8.png">


# ディレイをかけてアニメーションをスタートさせたい

[公式のanimation tutorial](https://flutter.io/tutorials/animation/)をカスタマイズする。
Tweenの```.animate()```の引数に、```Interval```を設定した```CurvedAnimation```を渡してあげる。

```AnimationSample.dart
import 'package:flutter/animation.dart';
import 'package:flutter/material.dart';

class LogoApp extends StatefulWidget {
  _LogoAppState createState() => _LogoAppState();
}

class _LogoAppState extends State<LogoApp> with SingleTickerProviderStateMixin {
  Animation<double> animation;
  AnimationController controller;

  initState() {
    super.initState();
    controller = AnimationController(
        duration: const Duration(milliseconds: 2000), vsync: this);
    // 以下が、元々のソースコード
    // animation = Tween(begin: 0.0, end: 300.0).animate(controller)
    // controllerを渡していたところを、CurvedAnimationにして、Intervalを設定する
    animation = Tween(begin: 0.0, end: 300.0).animate(
        CurvedAnimation(parent: controller, curve: Interval(0.9, 1.0)))
      ..addListener(() {
        setState(() {
          // the state that has changed here is the animation object’s value
        });
      });
    controller.forward();
  }

  Widget build(BuildContext context) {
    return Center(
      child: Container(
        margin: EdgeInsets.symmetric(vertical: 10.0),
        height: animation.value,
        width: animation.value,
        child: FlutterLogo(),
      ),
    );
  }

  dispose() {
    controller.dispose();
    super.dispose();
  }
}

void main() {
  runApp(LogoApp());
}
```

ググったら[Staggerd Animation](https://flutter.io/animations/staggered-animations/)を使うのが良いって出てきたけど
delayだけなら、```AnimatedBuilder```を使わなくてもできます。

# テキストフィールドにフォーカスを当てたい（外したい）

```textFocus.dart
// 当てる
FocusScope.of(context).requestFocus(_textField.focusNode);

// 外す
FocusScope.of(context).requestFocus(new FocusNode());

// おまけ
// フォーカスが当たったイベント
TextEditingController _textEditingController = new TextEditingController();
FocusNode _focus = new FocusNode();

// フォーカス当たった時の関数
void _onFocusChange(){
  if (_focus.hasFocus) {
     // なんか処理書く
  }
}
// リスナーに登録
_focus.addListener(_onFocusChange);

_textField = new TextField(
  focusNode: _focus,
  controller: _textEditingController,
  decoration: InputDecoration(
      border: InputBorder.none,
      hintText: 'プレースホルダ'
  ),
);

// テキストの取得
String text = _textEditingController.text;

//テキストのクリア
_textEditingController.clear();

```

# Databaseを使いたい
今の所、SQLiteが一般的なのでしょうか？Realmの記事が見当たらず。
(sqflite)[https://pub.dartlang.org/packages/sqflite]というパッケージがあるので、これを導入してみました。
性能は計測していないので、遅いとかありそう。

[使い方はこちらの方を参考にしました。](https://proandroiddev.com/flutter-bookshelf-app-part-2-personal-notes-and-database-integration-a3b47a84c57)
ライセンス的にセーフかな？？

## 流れ
### 1 ```pubspec.yaml```に```sqflite```と```path_provider```を追加する。

```pubspec.yaml
dependencies:
  flutter:
    sdk: flutter
  sqflite: any
  path_provider: "^0.3.1"    

```

### 2. Modelクラスを作る。

```model.dart
import 'package:meta/meta.dart';

class Model {
  static final db_title = "title";
  static final db_id = "id";
  static final db_star = "star";

  String title;
  int id;
  bool starred;
  Model({
    @required this.title,
    @required this.id,
    this.starred = false,
  });

  Model.fromMap(Map<String, dynamic> map): this(
    title: map[db_title],
    id: map[db_id],
    starred: map[db_star] == 1,
  );

  // Currently not used
  Map<String, dynamic> toMap() {
    return {
      db_title: title,
      db_id: id,
      db_star: starred? 1:0,
    };
  }
}
```
[パクリ元](https://github.com/Norbert515/BookSearch/blob/master/lib/model/Book.dart)



### 3. databaseクラスを作る。


```database.dart
import 'dart:async';
import 'dart:io';
import 'package:path/path.dart';
import 'package:sqflite/sqflite.dart';
import 'package:path_provider/path_provider.dart';
import 'package:test_app/model/Model.dart';


class ModelDatabase {
  static final ModelDatabase _modelDatabase = new ModelDatabase._internal();

  final String tableName = "Models";

  Database db;

  bool didInit = false;

  static ModelDatabase get() {
    return _modelDatabase;
  }

  ModelDatabase._internal();

  /// Use this method to access the database, because initialization of the database (it has to go through the method channel)
  Future<Database> _getDb() async{
    if(!didInit) await _init();
    return db;
  }

  Future init() async {
    return await _init();
  }

  Future _init() async {
    // Get a location using path_provider
    Directory documentsDirectory = await getApplicationDocumentsDirectory();
    String path = join(documentsDirectory.path, "demo.db");
    db = await openDatabase(path, version: 1,
        onCreate: (Database db, int version) async {
          // When creating the db, create the table
          await db.execute(
              "CREATE TABLE $tableName ("
                  "${Model.db_id} INTEGER PRIMARY KEY,"
                  "${Model.db_title} TEXT,"
                  "${Model.db_star} BIT,"
                  ")");
        });
    didInit = true;
  }

  /// Get a book by its id, if there is not entry for that ID, returns null.
  Future<Model> getModel(String id) async{
    var db = await _getDb();
    var result = await db.rawQuery('SELECT * FROM $tableName WHERE ${Model.db_id} = "$id"');
    if(result.length == 0)return null;
    return new Model.fromMap(result[0]);
  }

  /// Get all books with ids, will return a list with all the books found
  Future<List<Model>> getModels(List<String> ids) async{
    var db = await _getDb();
    // Building SELECT * FROM TABLE WHERE ID IN (id1, id2, ..., idn)
    var idsString = ids.map((it) => '"$it"').join(',');
    var result = await db.rawQuery('SELECT * FROM $tableName WHERE ${Model.db_id} IN ($idsString)');
    List<Model> models = [];
    for(Map<String, dynamic> item in result) {
      models.add(new Model.fromMap(item));
    }
    return books;
  }


  Future<List<Book>> getFavoriteBooks() async{
    var db = await _getDb();
    var result = await db.rawQuery('SELECT * FROM $tableName WHERE ${Book.db_star} = "1"');
    if(result.length == 0)return [];
    List<Book> books = [];
    for(Map<String,dynamic> map in result) {
      books.add(new Book.fromMap(map));
    }
    return books;
  }

  //TODO escape not allowed characters eg. ' " '
  /// Inserts or replaces the book.
  Future updateModel(Model model) async {
    var db = await _getDb();
    await db.rawInsert(
        'INSERT OR REPLACE INTO '
            '$tableName(${Model.db_id}, ${Model.db_title}, ${Model.db_star})'
            ' VALUES(?, ?, ?)',
        [model.id, model.title, model.starred? 1:0]);

  }

  Future close() async {
    var db = await _getDb();
    return db.close();
  }

}
```

[こちらのパクり元。]
(https://github.com/Norbert515/BookSearch/blob/master/lib/data/database.dart) 

適宜必要なメソッドを追加してください。

### 4. 呼び出す

```main.dart
// データのインサート
Model model = new Model(title: 'title', id: 1);
ModelDatabase.get().updateModel(home);

// データの読み込み
Model model = await ModelDatabase.get().getModel(1);
```


なぜか、```STRING PRIMARY KEY```の部分がどうしてもintになってしまい、Map関数が動かなかったので、```INTEGER PRIMARY KEY```に変更して、idもintにしています。
idに"1"とか"2"じゃなくて、文字列"one"とか"two"って入れたらちゃんと動くのでしょうか。

Realm使った後だと、自分でSQLを組み上げて、Stringくっつけて、、、というのが非常に面倒ですね。

# ベジェ曲線を引く
// 2018/09/23 追記

iOSだと、DrawRect、AndroidだとCanvasになりますかね。

## CustomPainter
直接ベジェ曲線を描画する部分は、CustomPainterを拡張したクラスの中で実装します。

```CustomPainter.dart
class Painter extends CustomPainter{
  @override
  void paint(Canvas canvas, Size size) {
    // TODO: implement paint
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) {
    return true;
  }
  
}
```

オーバーライドした```paint```の中に色々書きます。

## CustomPaint
その前に、CustomPainterクラスをどうやって使うかと言うと、```CustomPaint```Widgetのプロパティに指定してあげます。

```CustomPaint.dart
      body: new Center(
        child: new CustomPaint(painter: Painter(),)
      ),
```

こんなイメージですね。

## Paint
本題のベジェ曲線の描画処理はこんな感じ。

```CustomPainter.dart
class Painter extends CustomPainter{
  @override
  void paint(Canvas canvas, Size size) {
    Paint p = new Paint()
      ..color = Colors.black
      ..strokeCap = StrokeCap.round
      ..style = PaintingStyle.stroke
      ..strokeWidth = 18.0;

    canvas.save();
    Path path = Path();

    // 線の開始位置
    path.moveTo(0.0, 200.0);

    // ベジェ曲線の終了位置と、制御位置
    // 制御点を指定して、ゴールに向かっています。
    path.conicTo(100.0,
        50.0,
        200.0,
        200.0,
        1.3);

    // 描画する
    canvas.drawPath(path, p);
    p.style = PaintingStyle.fill;
    canvas.restore();
  }

  @override
  bool shouldRepaint(CustomPainter oldDelegate) {
    return true;
  }
}
```

 これを実行すると、こうなります。
<img width="370" alt="スクリーンショット 2018-09-23 23.21.02.png" src="images/59697da5-2b32-026f-29b6-2af2c6a56c8d.png">

最後に指定している```1.3```は重みらしく、1を指定すると、パラボラって書いてある。なにそれ。
わかりやすいように、0.3を指定すると、こうなります。

<img width="210" alt="スクリーンショット 2018-09-23 23.24.44.png" src="images/6ade3537-6c0a-efd5-72ff-2574559ba5ae.png">

カーブがちょっと緩くなりますね。

```bezie.dart
// ベジェ曲線の終了位置と、制御位置
    path.conicTo(100.0,
        50.0,
        200.0,
        200.0,
        1.0);
    path.conicTo(300.0,
        350.0,
        400.0,
        200.0,
        1.0);
```

繋げるとこうなります。

<img width="366" alt="スクリーンショット 2018-09-23 23.27.12.png" src="images/7cdc8997-7398-5239-1710-7e9581f05b1a.png">


あとは色々できますね。

## ```Paint```や```path```についてもう少し

さて、```path```にはcloseという関数があります。
これは、最後のポイントと、最初のポイントをつないでくれます。

<img width="312" alt="スクリーンショット 2018-09-23 23.30.06.png" src="images/b51a457c-b03d-d9d9-ea6b-1ea5e8e0c289.png">

Paintのstyleを```PaintingStyle.fill```にすると、塗りつぶしてくれます。
<img width="297" alt="スクリーンショット 2018-09-23 23.34.04.png" src="images/f66dbf48-391a-4e80-b98f-ba8c3de5dd22.png">

というか、デフォルトが塗りつぶしなので、線を引きたければ```stroke```にしてください。
繋がっていないpathは、fillの場合描画されないので、ちょっと焦ります。

## Maskフィルター
個人的に使えそうと思ったのは、```Paint```のmaskFilter。
例えば、

### BlurStyle.outer
```
p.maskFilter = MaskFilter.blur(BlurStyle.outer, 10.0);
```
こんな指定をする。

<img width="301" alt="スクリーンショット 2018-09-23 23.38.39.png" src="images/e31a896f-2470-556e-1d87-6db1d247ed51.png">

外側にブラーをかけてくれます。

### BlurStyle.inner

内側にブラー。

```
p.maskFilter = MaskFilter.blur(BlurStyle.inner, 20.0);
```

<img width="305" alt="スクリーンショット 2018-09-23 23.39.41.png" src="images/ca1d4b8d-a6eb-2f76-5368-200728e85cec.png">


### BlurStyle.normal

標準のブラー（先に書けって話ですが）。
いわゆる普通のぼかし。

```
p.maskFilter = MaskFilter.blur(BlurStyle.normal, 20.0);
```

<img width="320" alt="スクリーンショット 2018-09-23 23.41.30.png" src="images/eff58222-9ef8-2d12-786c-e2b020327239.png">


### BlurStyle.solid

いわゆる影、ですかね。。

```
p.maskFilter = MaskFilter.blur(BlurStyle.solid, 20.0);
```

<img width="302" alt="スクリーンショット 2018-09-23 23.42.13.png" src="images/b1aba6ab-fa8f-437d-ba8f-434cdbd214f9.png">



# Transform
// 2018/09/26 追記

オブジェクト(Widget)の回転やサイズ、座標を司るWidget。それがTransform。
FloatingActionButtonの位置を変えたかったので、使ってみました。

使い方は以下のような感じで。

```Transform.dart
new Transform(
  transform: new Matrix4.identity()..translate(0.0,-80.0), // 一つ目はx座標、2つ目以降はオプションで、y座標とz座標を指定可能。
  alignment: Alignment.center,
  child:FloatingActionButton(
    onPressed: null,
    child: Icon(Icons.favorite,color: Colors.pinkAccent,),
    backgroundColor: Colors.white,
  ),
),

```

私は、FloatingActionButtonがAdmobに隠されてしまうため、80ポイントほど上にずらすために使いました。

他にも、scaleで拡大/縮小ができたり

```
Matrix4.identity()..scale(0.0,-80.0),
```

lotateX,lotateYなどで、回転できたりします。

```
Matrix4.identity()..lotateX(0.0),
```

些細な問題としては、見た目の位置だけが変わるので、タップ判定の領域などは変わらないという点ですね。（**大問題**）

なので、FloatingActionButtonの位置を変えたい場合は、以下のようにContainerに入れてmarginを付ける方が正しいと思われます。

```fab.dart
floatingActionButton: new Container(
  margin: EdgeInsets.only(bottom: 80.0),
  child:FloatingActionButton(
    onPressed: null,
    child: Icon(Icons.favorite,color: Colors.pinkAccent,),
    backgroundColor: Colors.white,
  ),
```

ついでに言うと、```Container```にもtransformのプロパティがあるので、あえて```Transform```を使う理由が見当たらない。
と思いましたが、回転座標の中心位置を指定したりするのは、```transform```ではできないかも。

```rotate.dart
Container(
    width: width,
    height: width,
    child: Transform.rotate(
      origin: Offset(0.0, 0.0),
        angle: 100,
        child: new Image.asset('assets/hoge.png'),
    ),
),
```
