## PartⅡ ループとロジックの単純化

- 制御フロー
- 論理式
- 変数

### Chapter7 制御フローを読みやすくする

- 条件やループなどの制御フローはできるだけ、「自然」にする。コードの読み手が立ち止まったり読み返したりしないように書く。

#### 条件式の引数の並び順

```javascript
// x (10がlengthより大きかったら)
if (10 >= length) {}
// o (lengthが10より小さかったら)
if (length <= 10) {}

// o (受け取ったバイトが期待したバイトより小さかったら)
while (bytes_received < bytes_extended) {}
// x (期待したバイトが受け取ったバイトより大きかったら)
while (bytes_extended > bytes_received) {}
```

#### if/elseブロックの並び順

- 条件は否定系よりも肯定系を使う。
- 単純な条件を先に書く。
- 関心を引く条件や目立つ条件を先に書く

```javascript
// x (条件式が否定文、elseの式の方が目立っている)
if (!url.HasQueryParameter('expand_all')) {
  response.Render(items);
} else {
  for ( ...; ) {
    ...;
  }
}

// o
if (url.HasQueryParameter('expand_all')) {
  for ( ...; ) {
    ...;
  }
} else {
  response.Render(items);
}
```

#### 三項演算子

```c
// o
time_str += (hour >= 12) ? "pm" : "am";
// x
if (hour >= 12) {
  time_str += "pm";
} else {
  time_str += "am";
}

// x
return exponent >= 0 ? mantissa * (1 << exponent) : mantissa / (1 << -exponent);
// o
if (exponent >= 0) {
  return mantissa * (1 << exponent);
} else {
  return mantissa / (1 << -exponent);
}
```

#### do/whileループを避ける

```java
// x (do/whileは基本的に読みにくい)
// search the 'node' list for matches against 'name'
// Do not consider nodes that become exceed 'max_length'
public boolean ListHasNode(Node node, String name, int max_length) {
  do {
    if (node.name().equals(name))
      return true;
    node = node.next();
  } while (node != null && --max_length > 0);
  
  return false;
}

// o (読みやすい)
public boolean ListHasNode(Node node, String name, int max_length) {
  while (node != null && max_length-- > 0); {
    if (node.name().equals(name)) return true;
    node = node.next();
  } 
  return false;
}

/* C++ Programming Language (ビャーネ・ストロヴストルップ)
 * 私の経験では、do-statementは、エラーや混乱の原因になることが多い。（中略）私は条件が「前もって」描かれている方が好きだ。そのため、私はdo-statementを避けることが多い。
```

#### 関数から早く返す

```java
public boolean Contains(String str, String substr) {
  if (str == null || substr == null) return false;
  if (substr.equals("")) return false;
}

// 上記のような「ガード節」は次のような機能で改善することができる
/* Language and clean-up-code
 * C++:          destractor
 * Java, Python: try finally
 * Python:       with
 * C#:           using
 */
```

#### 悪名高きgoto

```c
// gotoが唯一許されているコード
if (p == NULL) goto exit;

exit:
  fclose(file1);
  fclose(file2);
  return;
```

#### ネストを浅くする

```c++
// x
if (user_result == SUCCESS) {
  if (permission_result != SUCCESS) {
    reply.WriteErrors("error reading permissions");
    reply.Done();
    return;
  }
  reply.WriteErrors("")
} else {
  reply.WriteErrors(user_result);
}
reply.Done();

// o
if (user_result != SUCCESS) {
  reply.WriteErrors(user_result);
  reply.Done();
  return;
}

if (permission_result != SUCCESS) {
  reply.WriteErrors("error reading permissions");
  reply.Done();
  return;
}

reply.WriteErrors("");
reply.Done();
```

#### 実行の流れを追えるかい？

```c
// コードを「舞台裏」で実行する構成要素
/* 構成要素と高レベルの流れが不明瞭になる理由
 * スレッド
 *   - どのコードがいつ実行されるのかよくわからない
 * シグナル・割り込みハンドラ
 *   - 他のコードが実行される可能性がある
 * 例外
 *   - いろんな関数呼び出しが終了しようとする
 * 関数ポインタと無名関数
 *   - コンパイル時に判別できないので、どのコードが実行されるのかわからない
 * 仮想メソッド
 *   - object.virtualMethod()は未知のサブクラスのコードを呼び出す可能性がある。
```

#### まとめ

- 比較を書くときは、「変化する値を左」「安定した値を右」に配置する
- if/else文はブロックを適切に並び替えて、肯定系、単純、目立つものを先に処理する
- 三項演算子や、do/whileループ、gotoなどはコードが読みにくくなることが多いため使わない方が良い
- 「ネスト」は浅い方が良い
- 「ガード節」は言語の持っている機能を使うと良い(tryなど)

### Chapter 8 巨大な式を分割する

- コードの規模が大きすぎると、理解が難しくなる

#### 説明変数

```javascript
if line.split(':')[0].strip() == "root"

// 説明変数
username = line.split(':')[0].strip();
if username == "root"
```

#### 要約変数

```javascript
// 考えるのに時間がかかる
if (request.user.id == document.owner_id) {
  // ユーザはこの文書を編集できる
}

if (request.user.id != document.owner_id) {
  // この文書は読み取り専用
}

// 要約変数
var userOwnsDocument = (request.user.id == document.owner_id);

if (userOwnsDocument) {}
if (!userOwnsDocument) {}
```

#### ド・モルガンの法則を使う

```javascript
if (!(file_exists && !is_protected)) Error("Sorry, could not read file")

// 上のコードと下のコードは等価
if (!file_exists || is_protected) Error("Sorry, could not read file")
```

#### 短絡評価の悪用

```c++
// before
assert((!(bucket = FindBucket(key))) || !bucket->IsOccupied());

// after
bucket = FindBucket(key);
if (bucket != NULL) assert(!bucket->IsOccupied());

// 「頭がいい」コードに気をつける。後で他の人がコードを読むときに分かりにくくなる
```

#### 例：　複雑なロジックと格闘する

```c++
struct Range {
  int begin;
  int end;
  
  // for example, [0,5] overlaps with [3,8]
  bool OverlapsWith(Range other);
};

// before
bool Range::OverlapsWith(Range other) {
  // check to see if 'begin' of 'end' is in 'other'
  return (begin >= other.begin && begin <= other.end) ||
         (end >= other.begin && end <= other.end) ||
         (begin <= other.begin && end >= other.end);
}

// after
bool Range::OverlapsWith(Range other) {
  if (other.end <= begin) return false; // One end is before this begin
  if (other.begin >= end) return false; // One begin is before this end
  return true; // What's left is overlapping
}
```

#### 巨大な文を分割する

```javascript
// before
var update_highlight = function(message_num) {
  if ($('vote_value' + message_num).html() === 'Up') {
    $('#thumbs_up' + message_num).addClass('highlighted');
    $('#thumbs_down' + message_num).addClass('highlighted');
  } else if ($('vote_value' + message_num).html() === 'Down') {
    $('#thumbs_up' + message_num).addClass('highlighted');
    $('#thumbs_down' + message_num).addClass('highlighted');
  } else {
    $('#thumbs_up' + message_num).removeClass('highlighted');
    $('#thumbs_down' + message_num).removeClass('highlighted');
  }
};

// after
var update_highlight = function(message_num) {
  let thumbs_up = $('#thumb_up' + message_num);
  let thumbs_down = $('#thumb_down' + message_num);
  let vote_value = $('vote_value' + message_num).html();
  let hi = 'highlighted';

  if (vote_value === 'Up') {
    thumbs_up.addClass(hi);
    thumbs_down.addClass(hi);
  } else if (vote_value === 'Down') {
    thumbs_up.addClass(hi);
    thumbs_down.addClass(hi);
  } else {
    thumbs_up.removeClass(hi);
    thumbs_down.removeClass(hi);
  }
};
```

#### 式を簡潔にするもう一つの創造的な方法

```c++
// before
void AddStats(const Stats& add_from, Stats* add_to) {
  add_to->set_total_memory(add_from.total_memory() + add_to->total_memory());
  add_to->set_free_memory(add_from.free_memory() + add_to->free_memory());
  add_to->set_swap_memory(add_from.swap_memory() + add_to->swap_memory());
  add_to->set_status_string(add_from.status_string() + add_to->status_string());
  add_to->set_num_processes(add_from.num_processes() + add_to->num_processes());
}

// after (define macro)
void AddStats(const Stats& add_from, Stats* add_to) {
  #define ADD_FIELD(field) add_to->set_##field(add_from.field() + add_to->field())
  ADD_FIELD(total_memory);
  ADD_FIELD(free_memory);
  ADD_FIELD(swap_memory);
  ADD_FIELD(status_string);
  ADD_FIELD(num_processes);  
  #undef ADD_FIELD
}
```

#### まとめ

- 説明変数を導入する
  - 巨大な式を分割できる
  - 簡潔な名前で式を説明することで、コードを文書化できる
  - 「概念」を認識しやすくなる
- ド・モルガンの法則を使うことによって論理式をきれいに書き直すことができる
- コードの改善方法がわからなくなったら、反対のことを考えてみるのも良い

### Chapter9 変数と読みやすさ

- 変数が多いと変数を追跡するのが難しくなる。
- 変数のスコープが大きいとスコープを把握する時間が長くなる
- 変数が頻繁に変更されると現在の値を把握するのが難しくなる

#### 変数を削除する

```python
# x
now = datetime.datetime.now()
root_message.last_view_time = now

# o
root_message.last_view_time = datetime.datetime.now()
```

```javascript
// x
var remove_one = function(array, value_to_remove) {
  var remove_one = null
  for (var i=0; i<array.length; i++) {
    if (array[i] == value_to_remove) {
      index_to_remove = i;
      break;
    }
  }
  if (index_to_remove !== null) {
    array.splice(index_to_remove, 1);
  }
};

// o
var remove_one = function(array, value_to_remove) {
  for (var i=0; i<array.length; i++) {
    if (array[i] != null) {
      array.splice(i, 1);
      return;
    }
  }
}
```

#### 変数のスコープを縮める

```c++
// x
PaymentInfo* info = database.ReadPaymentInfo();
if (info) {
  cout << "User paid: " << info->amount() << endl;
}

// o (スコープを縮めることに成功)
if (PaymentInfo* info = database.ReadPaymentInfo()) {
  cout << "User paid: " << info->amount() << endl;
}
```

#### 変数は一度だけ書き込む

- 変数を操作する場所が増えると、現在地の判断が難しくなる

#### 最後の例

```javascript
// x (変数が三つ定義されている、バラバラに見える)
var setFirstEmptyInput = function(new_value) {
  var found = false;
  var i = 1;
  var elem = document.getElementById('input' + i);
  
  while (elem != NULL) {
    if (elem.value === '') {
      found = true;
      break;
    }
    i++;
    elem = document.getElementById('input' + i);
  }
  if (found) elem.value = new_value;
  return elem;
}

// o (変数が１つしか定義されていない、まとまって見える)
var setFirstEmptyInput = function(new_value) {
  for (var i=1; true; i++) {
    var elem = document.getElementById('input' + i);

    if (elem === null)
      return null; // search failed, not found empty input

    if (elem.value === '') {
      elem.value = new_value;
      return elem;
    }
  }
};
```

#### まとめ

- 邪魔な変数を削除する
- 変数のスコープをできるだけ小さくする
- 一度だけ書き込む変数を使う
