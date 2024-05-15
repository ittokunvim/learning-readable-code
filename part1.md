# リーダブルコード

> より良いコードを書くためのシンプルで実践的なテクニック

### Chapter 1 理解しやすいコード

- コードは理解しやすくなければならない

#### 「優れた」コードって何？

```c++
// x (読みにくい)
for (Node* node = list->head; node != NULL; node->next)
  Print(node->data);

// o (読みやすい)
Node* node = list->head;
if (node == NULL) return;

while (node->text != NULL) {
  Print(node->data);
  node = node->next;
}
if (node != NULL) Print(node->data);
```

#### 読みやすさの基本定理

- コードは他の人が最短時間で理解できるように書かなければいけない

#### 小さなことは絶対にいいこと？

- コードは短くした方が良い、だけど「理解するまでのじかん」が短い方がもっと良い

#### 「理解するまでにかかる時間」は競合する？

- 「このコードは理解しやすいだろうか」と自問自答することは大切

#### でもやるんだよ

- 理解しやすいコードが書けるプログラマは「いい」プログラマだ

## PartⅠ 表面上の改善

- 適切な名前を選ぶ
- 優れたコメントを書く
- コードをきれいにフォーマットする
- こういった改善は手間がかからないからやるに越したことはない！

### Chapter 2 名前に情報を詰め込む

- 名前に情報を詰め込む

#### 明確な単語を選ぶ

```javascript
// x (Getでは曖昧すぎる)
function GetPage(url) {}
// o (Fetchでは動作が限定されているから良い)
function FetchPage(url) {}

//「カラフル」な単語を選ぶと良い
send =  ['deliver','dispatch','announce','distribute','route'];
find = ['search', 'extract', 'locate', 'recover'];
start = ['launch', 'create', 'begin', 'open'];
make = ['create', 'set up', 'build', 'generate', 'compose', 'add', 'new'];                         
```

#### tmpやretvalなどの汎用的な名前を避ける

```javascript
// retval
// x (retval=戻り値、以外の情報がない)
var euclidean_norm = function(v) {
  var retval = 0.0;
  for (var i=0; i<v.length; i+=1) {
    retval += v[i] * v[i];
  }
  return Math.sqrt(retval);
};

// o (sum_squareが適切)

// tmp
// o (tmpを使うときは生存期間が短い時に限る)
if (light < left) {
  tmp = right;
  right = left;
  left = tmp;
}

// loop iterator
// 基本的にはi, j, kが適している
// が、下記の例ではci, ui, miの方がよろしい
for (int ci=0; ci<clubs.size(); ci++)
  for (int mi=0; mi<clubs[ci].size(); mi++)
    for (int ui=0; ui<users.size(); ui++)
      if (club[ci].members[mi] == users[ui])
        printf("user[%d] is in club[%d]", ui, ci)
```

#### 抽象的な名前よりも具体的な名前を使う

```c++
// x (よくわからない)
class ClassName {
  private:
  DISALLOW_EVIL_CONSTRUCTORS(ClassName);
}

#define DISALLOW_EVIL_CONSTRUCTORS(ClassName) \
	ClassName(const ClassName&); \
	void operator=(const ClassName&);

// o (DISALLOW_COPY_AND_ASSIGN(ClassName)が正しいらしい)
```

- ServerCanStart()よりもCanListenOnPort()の方が明確

- 接尾辞や接頭辞を使って情報を追加する
  - 後ろに\_ms、 前にtable\_、などをつける
- 名前の長さを決める
  - 長すぎも良くないが、短すぎも良くない
- 名前のフォーマットで情報を伝える
  - アンダーバーや大文字を使う

### Chapter 3 誤解されない名前

#### filter()

``` javascript
results = Database.all_objects.filter('year <= 2011')

// これだとyearが2011以上の場合なのか、2011以上ではない場合なのかわからない
// select()やexclude()などが適当である
```

#### clip(text, length)

```javascript
function clip(text, length) {}

// これだとlengthがどういう働きをするのかわからない
// max_lengthや、文字数を意味しているのであればmax_charsなどが良い
```

#### 限界値を含めるときはminとmaxを使う

```javascript
// x
const CART_TOO_BIG_LIMIT = 10
// o
const MAX_ITEMS_IN_CART = 10
```

#### 範囲を指定するときはfirstとlastを使う

```javascript
// x
console.log(integer_range(start=2, stop=4))
// o
console.log(integer_range(first=2, last=4))

// 下の方が範囲を認識しやすい
```

#### 包含・排他的範囲にはbeginとendを使う

```javascript
function eventInRange(begin=today, end=tomorrow) {}
```

#### ブール値の名前

```javascript
// x
read_password = true
// o
need_password = true
// ▲
disable_ssl = false
// o
use_ssl = true

// 頭にis, has, can, shouldなどをつけることも多い
```

#### ユーザの期待に合わせる

```javascript
// getで始まるメソッドはメンバの値を返すだけの「軽量アクセサ」であるという認識で使うようにする
// sizeと入る関数も同様に軽くした方が良い
```

#### 複数の名前を検討する

### Chapter 4 美しさ

美しさとは、段落の長さ、横幅、順番、のことである。

#### 一貫性のある簡潔な改行位置

```java
// x
public class PerformanceTester {
  public static final TcpConnectionSimulator wifi = new TcpConnectionSimulator(
  500, // kps
  80, // millisecs latency
  200, // jitter
  1); // packet loss %
}

// o
public class PerformanceTester {
  public static final TcpConnectionSimulator wifi = new TcpConnectionSimulator(
  500, // kps
  80,  // millisecs latency
  200, // jitter
  1);  // packet loss %
}

/* インデントを揃えると見やすくなる */

public class PerformanceTester {
  // TcpConnectionSimulator(throughput, latency, jitter, packet_loss);
  //                        [kbps]      [ms]     [ms]    [percent]  
  public static final TcpConnectionSimulator wifi =
    new TcpConnectionSimulator(500,     80,      200,    1);
}

/* 簡潔に書くとこうなる */
```

#### メソッドを使った整列

```javascript
// change partial_name like Doug Adams to Mr. Douglas Adams
// if it can't, add description to error 

DatabaseConnection database_connection;
string error;
assert(hoge);
assert(hoge1);
assert(bar1);
assert(baz);
assert(hoge);
...
// こうも一列に並んでしまうと見にくくなる
// 改善するには、ヘルパーメソッドを使用するか、インデントを整えるか、する
```

#### 縦の線をまっすぐにする

```javascript
// x
CheckFullName('Doug Adams', 'Mr. Douglas Adams', '');
CheckFullName('John doe', 'Ms. John doe', '');
CheckFullName('Art titum', 'Mr. Art Titum', 'no match found');
CheckFullName('Duke Ellington', 'Mr. Duke', 'more than one result');
// o
CheckFullName('Doug Adams',     'Mr. Douglas Adams', '');
CheckFullName('John doe',       'Ms. John doe',      '');
CheckFullName('Art titum',      'Mr. Art Titum',     'no match found');
CheckFullName('Duke Ellington', 'Mr. Duke',          'more than one result');
```

#### 宣言をブロックにまとめる

```c++
// x
class FrontendServer {
  public:
  	FrontendServer();
    void ViewProfile(HttpRequest* request);
    void OpenDatabase(string location, string user);
    void SaveProfile(HttpRequest* request);
    string ExtractQueryParams(HttpRequest* request, string params);
    void ReplyOk(HttpRequest* request, string html);
    void findFriends(HttpRequest* request, string error);
    void ReplyNotFound(HttpRequest* request, string error);
    void CloseDatabase(string location);
    ~FrontendServer();
}
// o
class FrontendServer {
  public:
  	FrontendServer();
    ~FrontendServer();

    // Handler
    void ViewProfile(HttpRequest* request);
    void SaveProfile(HttpRequest* request);
    void findFriends(HttpRequest* request, string error);

		// Utility of request and reply
    string ExtractQueryParams(HttpRequest* request, string params);
    void ReplyOk(HttpRequest* request, string html);
    void ReplyNotFound(HttpRequest* request, string error);
  
    // Database Helper
    void OpenDatabase(string location, string user);
    void CloseDatabase(string location);

}
```

#### コードを段落に分割する

```python
# Import to user mailer. Check against the system's users
# and display users list who have not yet become friends

# x
def suggest_new_friends(user, email_password):
  friends = user.friends()
  friend_emails = set(f.email for f in friends)
  contacts = import_contacts(user.email, email_password);
  contact_emails = set(c.email for c in email_password)
  non_friend_emails = contact_emails - friend_emails
  suggested_friends - User.objects.select(email_in=non_friend_emails)
  display['user'] = user
  display['friends'] = friends
  display['suggested_friends'] = suggested_friends
  return render('suggested_friends.html', display)

# o
def suggest_new_friends(user, email_password):
  # get email address of the users friends
  friends = user.friends()
  friend_emails = set(f.email for f in friends)
  # import all email address from users mail account
  contacts = import_contacts(user.email, email_password)
  contact_emails = set(c.email for c in email_password)
  # find user have not yet become friends
  non_friend_emails = contact_emails - friend_emails
  suggested_friends - User.objects.select(email_in=non_friend_emails)
  # display page
  display['user'] = user
  display['friends'] = friends
  display['suggested_friends'] = suggested_friends
  
  return render('suggested_friends.html', display)
```

### Chapter 5 コメントすべきことを知る

- コメントするべきでは「ない」ことを知る
- コードを書いている時の自分の考えを記録する
- 読み手の立場になってないが必要になるかを考える

#### コメントするべきではないこと

```c++
// わざわざ書くことではない
// Define Account class
class Account {
  public:
  // constructor
  Account();
  // set new value to profit
  void SetProfit(double profit);
  // return profit from this Account
  double GetProfit();
};
```

#### 自分の考えを記録する

```javascript
// このデータだとハッシュテーブルよりもバイナリツリーの方が40%速かった
// 左右の比較よりもハッシュの計算コストの方が高いようだ

// TODO: あとで手をつける
// FIXME: 既知の不具合があるコード
// HACK: あまり綺麗じゃない解決策
// XXX: 危険！　大きな問題がある

const MAX_THREADS = 8 // enough value of [>= 2 * num_proccessors]
```

#### 読み手の立場になって考える

```python
def GenerateUserReport():
  # このユーザのロックを獲得する
  ...
  # ユーザの情報をDBから読み込む
  ...
  # 情報をファイルに書き出す
  ...
  # このユーザのロックを解放する
```

### Chapter6 コメントは正確で簡潔に

- コメントは領域に対する情報の比率が高くなければいけない。

#### コメントを簡潔にしておく

```c++
// x
// int is CategoryType
// first float of pair is score
// second is weight
typedef hash_map<int, pair<float, float> > ScoreMap;

// o
// CategoryType -> (score, weight)
typedef hash_map<int, pair<float, float> > ScoreMap;
```

#### 曖昧な代名詞を避ける

```javascript
// x (「this」や「its」という表現はしない方が良い
// Put into the cache. But check its size first

// o (データをキャッシュに入れる前にデータのサイズをチェックする)
// Put into the cache. But check data size first
// if the data is small enough, put into the cache
```

#### 歯切れの悪い文章を磨く

```javascript
// x (「どうか(based on whether)」と「変える(change)」の表現がわかりにくい)
// Change the priority based on whether the URL has been crawed before or not (based on whether: 〜かに基づいて)

// o　（クロールしたURLの優先度は低く、していないURLの優先度は高い）
// Give higher priority to URLs that have not been crawed before
```

#### 関数の動作を正確に記述する

```c++
// x (行にもさまざまな種類がありどれを指すか怪しい)
// return number of lines contained in this file
int CountLines(string filename) { ... }

// o 
// このファイルに含まれる改行文字(\n)を返す
// return the newline characters '\n' contained in this file
int CountLines(string filename) { ... }
```

#### 入出力のコーナーケースに実例を使う

```c++
// x (説明が長く、理解しにくい)

// Remove 'chars' from the begining and end of 'src'
String Strip(String src, String chars) { ... }

// Replace 'v' so that "elem < pivot" comes before "elem >= pivot"
// Then return the larghest 'i' that results in "v[i] < pivot" (or -1 if none)
int Partition(vector<int>* v, int pivot);

// o (例を示すことによって、文章を書くよりわかりやすい)

// example: Strip("abba/a/ba", "ab") -> return "/a/"
String Strip(String src, String chars) { ... }

// example: Partition([8 5 9 8 2], 8) result: [5 2 | 8 9 8] return: 1
int Partition(vector<int>* v, int pivot);
```

#### コードの意図を書く

```c++
void DisplayProducts(list<Product> products) {
  products.sort(CompareProductByPrice);
  
  // x (わざわざ書いても仕方ないコメント(それとproductsはすでに逆順にソートされていて、このコメントは間違っている！))
  // Iterate through the list in reverse order
  for (list<Product>::reverse_iterator it = products.rbegin(); it != products.rend(); ++it) {
    DisplayPrice(it->price);
  }
  
  // o (プログラムの動作を高レベルから説明している)
  // Display in order of increasting price
  for ( ... ) {
    ...
  }
}
```

#### 名前付き引数コメント

```java
// python: named argument comment
// def Connect(timeout, use_encryption): ...
// Connect(timeout = 10, use_encription = False)

// Java: named argument comment
void Connect(int timeout, bool use_encryption) { ... }
Connect(/* timeout = */ 10, /* use_encryption = */ false);
```

#### 情報密度の高い言葉を使う

```java
/* このクラスには大量のメンバがある。同じ情報はデータベースに保存されている、ただし、
 * 速度の面からここにも保管しておく。このクラスを読み込むときは、メンバが存在しているかどうかを
 * 先に確認する。もし存在していれば、そのまま返す。存在しなければ、データベースから読み込んで
 * 次回のためにデータをフィールドに保管する
 */
 
 // このクラスの役割は、データベースのキャッシュ層である
```

#### まとめ
- 複数のものを指す可能性がある「それ」や「これ」などの代名詞を避ける。
- 関数の動作はできるだけ正確に説明する
- コメントに含める入出力の実例を慎重に選ぶ
- コードの意図は、詳細レベルでなく、高レベルで記述する
- よくわからない引数にはインラインコメントを使う
- 多くの意味がつめられた言葉や表現を使って、コメントを簡潔に保つ。
