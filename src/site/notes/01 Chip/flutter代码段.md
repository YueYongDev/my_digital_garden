---
{"dg-publish":true,"tags":["flutter"],"permalink":"/01 Chip/flutter代码段/","dgPassFrontmatter":true}
---



### container设置最大宽高
```dart
Container(
  constraints: BoxConstraints(maxWidth: 160),
```


### container设置圆角

```dart
Container(
     margin: EdgeInsets.all(15.0),
     decoration: BoxDecoration(
          borderRadius: BorderRadius.circular(5),
          color: Colors.white,
     ),                      
     child: Column(
        //...
     )
)
```

### 圆角图片

```dart
ClipRRect(
  borderRadius: BorderRadius.circular(8),
  child: Image.network(
    'https://pic2.zhimg.com/v2-639b49f2f6578eabddc458b84eb3c6a1.jpg',
    width: 120,
    height: 120
  )
)

```


### 基础按钮

```dart
MaterialButton buildMaterialButton() {  
  return MaterialButton(  
    //边框样式  
    shape: const RoundedRectangleBorder(  
      //边框颜色  
      side: BorderSide(  
        color: Colors.blueGrey,  
        width: 1,  
      ),  
      //边框圆角  
      borderRadius: BorderRadius.all(  
        Radius.circular(8),  
      ),  
    ),  
    //按钮高度  
    height: 40,  
    //按钮最小宽度  
    minWidth: Get.width / 1.6,  
    //点击事件  
    onPressed: () {  
      Get.back();  
    },  
    child: const Text(  
      "取消",  
      style: TextStyle(  
          fontSize: 16, color: Colors.white70, fontWeight: FontWeight.w300),  
    ),  
  );  
}
```


### 强制横/竖屏

```dart
void main(){
  WidgetsFlutterBinding.ensureInitialized();  		//不加这个强制横/竖屏会报错
  SystemChrome.setPreferredOrientations([     // 强制竖屏
    DeviceOrientation.portraitUp,
    DeviceOrientation.portraitDown
  ]);
  SystemChrome.setPreferredOrientations([ 	 //强制横屏
    DeviceOrientation.landscapeLeft,
    DeviceOrientation.landscapeRight
  ]);
  runApp(new MyApp());
}

```