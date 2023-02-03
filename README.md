# FirestoreWhere
where文でStreambuilderを使う

# where_search_sample
Listとwhereを使ったプログラム
ソースコードが古かったので、書き直して動かしてみた!

## splitメソッドについて
### 公式を翻訳
patternにマッチした文字列を分割し、その部分文字列のリストを返す。
Pattern.allMatchesを使用して、この文字列内のpatternのすべてのマッチを見つけ、
マッチの間、最初のマッチの前、および最後のマッチの後の部分文字列のリストを返します。

https://api.dart.dev/stable/2.19.1/dart-core/String/split.html<br>
https://zenn.dev/tris/articles/bf623e5e65fac3<br>

```dart
import 'dart:math';

import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:firebase_core/firebase_core.dart';
import 'package:flutter/material.dart';

import 'firebase_options.dart';

Future<void> main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp(options: DefaultFirebaseOptions.currentPlatform);
  runApp(MyApp());
}

class MyApp extends StatefulWidget {
  @override
  _MyAppState createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  String name = "";
  final TextEditingController _addNameController = TextEditingController();

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
          appBar: AppBar(
            title: TextField(
              onChanged: (val) => initiateSearch(val),
            ),
          ),
          body: Center(
            child: Column(
              children: [
                TextField(
                  controller: _addNameController,
                ),
                ElevatedButton(
                    onPressed: () async {
                      _addToDatabase(_addNameController.text);
                    },
                    child: Text('追加')),
                Expanded(
                  child: StreamBuilder<QuerySnapshot>(
                    stream: name != "" && name != null
                        ? FirebaseFirestore.instance
                            .collection('brands')
                            .where("nameSearch", arrayContains: name)
                            .snapshots()
                        : FirebaseFirestore.instance
                            .collection("brands")
                            .snapshots(),
                    builder: (BuildContext context,
                        AsyncSnapshot<QuerySnapshot> snapshot) {
                      if (snapshot.hasError)
                        return Text('Error: ${snapshot.error}');
                      switch (snapshot.connectionState) {
                        case ConnectionState.waiting:
                          return const Text('Loading...');
                        default:
                          return ListView(
                            children: snapshot.data!.docs
                                .map((DocumentSnapshot document) {
                              Map<String, dynamic> data =
                                  document.data()! as Map<String, dynamic>;
                              return ListTile(
                                title: Text(data['name']),
                              );
                            }).toList(),
                          );
                      }
                    },
                  ),
                ),
              ],
            ),
          )),
    );
  }

  void initiateSearch(String val) {
    setState(() {
      name = val.toLowerCase().trim();
    });
  }

  void _addToDatabase(String name) async {
    List<String> splitList = name.split(" ");

    List<String> indexList = [];

    for (int i = 0; i < splitList.length; i++) {
      for (int y = i; y < splitList[i].length + 1; y++) {
        indexList.add(splitList[i].substring(0, y).toLowerCase());
      }
    }

    print(indexList);

    await FirebaseFirestore.instance
        .collection('brands')
        .doc()
        .set({'name': name, 'nameSearch': indexList});
  }
}

```
