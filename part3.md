## PartⅢ コードの再構成

- プログラムの主目的と関係のない「無関係の回問題」を抽出する
- コードを再構成して、一度に１つのことをやるようにする
- 最初にコードを言葉で説明する。その説明をもとにしてきれいな解決策を作る

### Chapter10 無関係の下位問題を抽出する

- エンジニアリングとは、大きな問題を小さな問題に分割して、それぞれの解決策を組み立てることに他ならない

1. 関数やコードブロックを見て「このコードの高レベルの目標は何か？」と自問する
2. コードの各行に対して「高レベルの目標に直接的に効果があるのか？あるいは、無関係の下位問題を解決しているのか？」と自問する
3. 無関係の下位問題を解決しているコードが相当量あれば、それらを抽出して別の関数にする

#### 入門的な例：findClosesLocation()

```javascript
// x （コードが長い）
// return the element of 'array' closest to the given latitude and longitude
// it assumes that the earth is a perfect sphere
var findClosesLocation = function(lat, lng, array) {
  var closest;
  var closest_dist = Number.MAX_VALUE;
  for (var i=0; i<array.length; i+=1) {
    // convert two points to radians
    var lat_rad = radians(lat);
    var lng_rad = radians(lng);
    var lat2_rad = radians(array[i].latitude);
    var lng2_rad = radians(array[i].longitude);
    
    // using the formula for the second cosine theorem in spherical trigonometry
    var dist = Math.acos(Math.sin(lat_rad) * Math.sin(lat2_rad) +
                         Math.cos(lat_rad) * Math.cos(lat2_rad) *
                         Math.cos(lng2_rad - lng_rad));
    if (dist < closest_dist) {
      closest = array[i];
      closest_dist = dist;
    }
  }
  return closest;
}

// o
var spherical_distance = function(lat1, lng1, lat2, lng2) {
  var lat1_rad = radians(lat1);
  var lng1_rad = radians(lng1);
  var lat2_rad = radians(lat2);
  var lng2_rad = radians(lng2);

  // using the formula for the second cosine theorem in spherical trigonometry
  return Math.acos(Math.sin(lat_rad) * Math.sin(lat2_rad) +
                   Math.cos(lat_rad) * Math.cos(lat2_rad) *
                   Math.cos(lng2_rad - lng_rad));
}

// return the element of 'array' closest to the given latitude and longitude
// it assumes that the earth is a perfect sphere
var findClosesLocation = function(lat, lng, array) {
  var closest;
  var closest_dist = Number.MAX_VALUE;
  for (var i=0; i<array.length; i+=1) {
    // convert two points to radians
    var dist = spherical_distance(lat, lng, array[i].latitude, array[i].longitude);
    if (dist < closest_dist) {
      closest = array[i];
      closest_dist = dist;
    }
  }
  return closest;
}
```

#### 純粋なユーティリティコード

```c++
ifstream ReadFileToString(file_name);

// calculate the file size and allocated that size to the buffer
ReadFileToString.seekg(0, ios::end);
const int file_size = file.tellg();
char* file_buf = new char [file_size];
// read a file into the buffer
ReadFileToString.seekg(0, ios::beg);
ReadFileToString.read(file_buf, file_size);
ReadFileToString.close();
```

#### その他の汎用コード

```javascript
ajax_post({
  url: 'http://example.com/submit',
  data: data,
  on_success: function(response_data) {
    format_pretty(response_data);
  }
});

var format_pretty = function(obj) {
  var str = "{\n";
  for (var key in obj) {
    str += " " + key + " = " + obj[key] + "\n";
  }
  alert(str + "}");
}

// 上記のformat_pretty(obj)が処理できないケース
// - objが普通の文字列（やundefined）だと例外が発生する
// - objがネストしたオブジェクトだとobject Objectのように表示されてしまう
// 書き直し
var format_pretty = function(obj, indent) {
  // processing the null, undefined, string, and non-object
  if (obj == null) return 'null';
  if (obj == undefined) return 'undefined';
  if (typeof obj === 'string') return '"' + obj + '"';
  if (typeof obj !== 'object') return String(obj);
  if (indent === undefined) indent = '';
  // processing the object (non null)
  var str = "{\n";
  for (var key in obj) {
    str += indent + ' ' + key + " = ";
    str +=format_pretty(obj[key], indent + ' ') + "\n";
  }
  return str + indent + '}';
}
```

#### 汎用コードをたくさん作る

- 汎用コードは簡単に共有できるように特別なディレクトリ（例：util/）を用意すると良い
- またプロジェクトから完全に切り離されているから開発もテストも理解するのも楽になる
- SQLデータベース・JavaScriptライブラリ・HTMLテンプレートシステムも同じようなもの

#### プロジェクトに特化した機能

```python
business = Business()
business.name = request.POST['name']

# x
url_path_name = business.name.lower()
url_path_name = re.sub(r"[\.]+", "", url_path_name)
url_path_name = re.sub(r"[^a-z0-9]+", "-", url_path_name)
url_path_name = url_path_name.strip('-')
business.url = "/biz/" + url_path_name

# o
CHARS_TO_REMOVE = re.compile(r"[\.]+")
CHARS_TO_DASH = re.compile(r"[^a-z0-9]+")

def make_url_friendly(text):
  text = text.lower()
  text = CHARS_TO_REMOVE.sub('', text)
  text = CHARS_TO_DASH.sub('-', text)
  return text.strip('-')

business.date_created = datetime.datetime.urcnow()
business.save_to_database()
```

#### 既存のインタフェースを簡潔にする

```javascript
// x コードが汚い
var max_results;
var cookies = document.cookie.split(';');
for (var i=0; i<cookies.length; i++) {
  var c = cookies[i];
  c = c.replace(/^[ ]+/, ''); // delete leading white space
  if (c.indexOf("max_results=") === 0) {
    max_results = Number(c.substring(12, c.length));
  }
}

// o
var max_results = Number(get_cookie("max_results"));

function get_cookie(value) {
  var cookies = document.cookie.split(';');
  for (var i=0; i<cookies.length; i++) {
    var c = cookies[i];
    c = c.replace(/^[ ]+/, ''); // delete leading white space
    if (c.indexOf(value) === 0)
      return c.substring(value.length+1, c.length);
  }
}
```

#### 必要に応じてインターフェースを整える

```python
user_info = { 'username': '...', 'password': '...'}

# x
user_str = json.dumps(user_info)
cipher = Cipher('aes_128_cbc', key=PRIVATE_KEY, init_vector=INIT_VECTOR, op=ENCODE)
encrypted_bytes = cipher.update(user_str)
encrypted_bytes += cipher.final() # flash the current 128-bit block
url = "http://example.com/?user_info=" + base64.urlsafe_b64encode(encrypted_bytes)

# o
def url_safe_encrypt(obj):
  obj_str = json.dumps(obj)
  cipher = Cipher('aes_128_cbc', key=PRIVATE_KEY, init_vector=INIT_VECTOR, op=ENCODE)
  encrypted_bytes = cipher.update(obj_str)
  encrypted_bytes += cipher.final() # flash the current 128-bit block
  return base64.urlsafe_b64encode(encrypted_bytes)

url = "http://example.com/?user_info=" + url_safe_encrypt(user_info)
```

#### やりすぎ

```python
user_info = { 'username': '...', 'password': '...'}
url = "http://example.com/?user_info=" + url_safe_encrypt(user_info)

def url_safe_encrypt_obj(obj):
  obj_str = json.dumps(obj)
  return url_safe_encrypt_str(obj_str)

def url_safe_encrypt_str(data):
  encrypted_bytes = encrypt(data)
  return base64.urlsafe_b64encode(encrypted_bytes)

def encrypt(data):
  cipher = make_cipher()
  encrypted_bytes = cipher.update(data)
  encrypted_bytes += cipher.final() # flash the current 128-bit block
  return encrypted_bytes

def make_cipher():
  return Cipher('aes_128_cbc', key=PRIVATE_KEY, init_vector=INIT_VECTOR, op=ENCODE)
```

#### まとめ

- 本章を簡単にまとめるとプロジェクト固有のコードから汎用コードを分離するということ

### Chapter11 一度に一つのことを

- コードは1つずつタスクを行うようにしたければいけない

1. コードが行なっている「タスク」を全て列挙する。
2. タスクをできるだけ異なる関数に分割する。少なくとも異なる領域に分割する

#### タスクは小さくできる

```javascript
/* ブログ用の投票ウィジェット
 * アップ（賛成）、ダウン（反対）を投票できる
 * scoreは全ての投票を合計したもの。アップは1、ダウンは-1
 */

vote_changed(old_vote, new_vote);

// x
var vote_changed = function(old_vote, new_vote) {
  var score = get_score();
  
  if (new_vote !== old_vote) {
    if (new_vote === 'UP') {
      score += (old_vote === 'Down' ? 2 : 1);
    } else if (new_vote === 'Down') {
      score -= (old_vote === 'Up' ? 2 : 1);
    } else if (new_vote === '') {
      score += (old_vote === 'Up' ? -1 : 1);
    }
  }
  
  set_score(score);
};

// o
var vote_value = function(vote) {
  switch(value) {
    case 'Up':
      return 1;
    case 'Down':
      return -1;
    default:
      return 0;
    }
};

var vote_changed = function(old_vote, new_vote) {
  var score = get_score();
	
  score -= vote_value(old_vote); // delete old value
  score += vote_value(new_vote); // add new value

	set_score(score);
}
```

#### オブジェクトから値を抽出する

```javascript
/* ユーザの所在地を読みやすい文字列（都市, 国）に整形する。（例：Santa Monica, USA | Osaka, Japan）
 * - 都市を選ぶときにLocalityName（市や町）、SubAdministrativeAreaName（都市や群）、AdministrativeAreaName（州や地域）
 * の順で使用可能なものを使う
 * - デフォルト値にMiddle-of-Nowwhereを設定する
 * - 国にCountryNameが使えない場合は、Planet Earth（地球）を設定する
 */

//x
var place = location_info['LocalityName']; // example: 'Santa Monica'
if (!place) {
  place = location_info['SubAdministrativeAreaName']; // example: 'Los Angeles'
}
if (!place) {
  place = location_info['AdministrativeAreaName'];
}
if (!place) {
  place = 'Middle-of-Nowwhere';
}
if (location_info['CountryName']) {
  place += ', ' + location_info['CountryName']; // example: 'USA'
} else {
  place += ', Planet Earth';
}

return place;

// o
var town = location_info['LocalityName']; // example: 'Santa Monica'
var city = location_info['SubAdministrativeAreaName']; // example: 'Los Angeles'
var state = location_info['AdministrativeAreaName']; // 'CA'
var country = location_info['CountryName']; // example: 'USA'

// first, set default value. and rewrite if value is found
var second_half = 'Planet Earth';
if (country) {
  second_half = country;
}
if (state && country === 'USA') {
  second_half = state;
}

var first_half = 'Middle-of-Nowhere';
if (state && country !== 'USA') {
  first_half = state;
}
if (city) {
  first_half = city;
}
if (town) {
  first_half = town;
}

return first_half + ', ' + second_half;

// その他の手法
var first_half, second_half;

if (country === 'USA') {
  first_half = town || ciry || 'Middle-of-Nowhere';
  second_half = state || 'USA';
} else {
  first_half = town || city || state || 'Middle-of-where';
  second_half = country || 'Planet Earth';
}

return first_half + ', ' + second_half;
```

#### もっとも大きな例

```c++
/* UpdateCounts()
 * さまざまな統計値を更新する関数。Webページをダウンロードした後に毎回呼ばれる
 * 
 * 
 */
// 理想
void UpdateCounts(HttpDownload hd) {
  counts["Exit State"   ][hd.exit_state()]++;    // example: 'SUCCESS'or 'FAILURE'
  counts["Http Response"][hd.http_response()]++; // example: "404 NOT FOUND"
  counts["Content-Type" ][hd.content_type()]++;  // example: "text/html"
}

// x
void UpdateCount(HttpDownload hd) {
  // find Exit state if possible
  if (!hd.has_event_log() || !hd.has_event_log().has_exit_state()) {
    counts["Exit State"]["unknown"]++;
  } else {
    string state_str = ExitStateTypeName(hd.event_log().exit_state());
    counts["Exit State"][state_str]++;
  }

  // if there is no HTTP header, set "unknown" to the reset of the elements
  if (!hd.has_http_headers()) {
    counts["Http Response"]["unknown"]++;
    counts["Content-type"]["unknown"]++;
    return;
  }

  HttpHeaders headers = hd.http_headers();

  // log the HTTP response. if not, set "unknown"
  if (!headers.has_response_code()) {
    counts["HTTP Response"]["unknown"]++;
  } else {
    string code = StringPrintf("%d", headers.response_code());
    counts["HTTP Response"][code]++;
  }

  // log the Content-Type. if not, set"unknown"
  if (!headers.has_content_type()) {
    counts["Content-Type"]["unknown"]++;
  } else {
    string coutent_type = ContentTypeMime(headers.content_type());
    counts["Content-Type"][content_type]++;
  }
}

// o
void UpdateCount(HttpDownload hd) {
  // TASK: set default value for the values you want to extract
  string exit_state = "unknown";
  string http_response = "unknown";
  string content_type = "unknown";

  // TASK: extract value from HttpDownload one by one
  if (!hd.has_event_log() || !hd.has_event_log().has_exit_state()) {
    exit_state = ExitStateTypeName(hd.event_log().exit_state());
  }
  if (hd.has_http_headers() && hd.http_headers().has_response_code()) {
    http_response = StringPrintf("%d", hd.http_headers().response_code());
  }
  if (hd.has_http_headers() && hd.http_headers().has_content_type()) {
    content_type = ContentTypeMime(hd.http_headers().content_type());
  }

  // TASK: update counts[]
  counts["Exit State"][exit_state]++;
  counts["HTTP Response"][http_response]++;
  counts["Content-Type"][content_type]++;
}

// 更なる改善
void UpdateCount(HttpDownload hd) {
  counts["Exit State"][ExitState(hd)]++;
  counts["HTTP Response"][HttpResponse(hd)]++;
  counts["Content-Type"][ContentType(hd)]++;
}

string ExitState(HttpDownload hd) {
  if (!hd.has_event_log() || !hd.has_event_log().has_exit_state()) {
    return ExitStateTypeName(hd.event_log().exit_state());
  } else {
    return "unknown";
  }
}

string HttpResponse(HttpDownload hd) {
  if (hd.has_http_headers() && hd.http_headers().has_response_code()) {
    http_response = StringPrintf("%d", hd.http_headers().response_code());
  } else {
    return "unknown";
  }
}

string ContentType(HttpDownload hd) {
 if (hd.has_http_headers() && hd.http_headers().has_content_type()) {
    content_type = ContentTypeMime(hd.http_headers().content_type());
  } else {
    return "unknown";
  }
}
```

#### まとめ

- 読みにくいコードがあれば、そこで行われているタスクを全て列挙して、分割する
