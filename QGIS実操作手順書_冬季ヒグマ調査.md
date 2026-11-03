# QGIS実操作手順書
## 冬季ヒグマ調査ルート・痕跡GISデータベースの構築

### 完成目標

最終的にQGIS上で以下の構造を作る。

```text
QGISプロジェクト
│
├── 地理院地図_淡色
│
└── higuma_winter.gpkg
     │
     ├── source_map      紙地図情報
     │       ↓ 1:N
     ├── survey_route    調査ルート
     │       ↓ 1:N
     └── bear_trace      ヒグマ痕跡
```

入力時には、

```text
紙地図を1枚選ぶ
      ↓
source_map登録
      ↓
調査ルートを描く
      ↓
痕跡2〜3点をクリック
      ↓
痕跡種類をプルダウン選択
```

という作業だけでデータを蓄積できる状態を目指す。

---

## STEP 0　作業フォルダを作る

Windowsのエクスプローラー等で研究用フォルダを作る。

例えば、

```text
higuma_winter_project
```

その中に、

```text
higuma_winter_project
│
├── qgis
├── source_maps
├── backup
├── export
└── analysis
```

を作る。

QGIS関係は、

```text
qgis/
```

に保存する。

---

## STEP 1　QGISを起動する

① QGISを起動

② 上部メニューから

**プロジェクト → 新規**

③

**プロジェクト → 名前を付けて保存**

④ `qgis`フォルダを選択

⑤ ファイル名を

```text
higuma_winter.qgz
```

にする。

---

## STEP 2　地理院地図を追加する

左側に「ブラウザ」パネルがあることを確認する。

表示されていなければ、

**表示 → パネル → ブラウザ**

にチェック。

---

## 2-1 XYZ Tilesを探す

ブラウザ内の

```text
XYZ Tiles
```

を探す。

右クリック。

↓

**新規接続**

---

## 2-2 淡色地図を登録

名前：

```text
地理院地図_淡色
```

URL：

```text
https://cyberjapandata.gsi.go.jp/xyz/pale/{z}/{x}/{y}.png
```

入力。

↓

**OK**

国土地理院の地理院タイル一覧で公開されている淡色地図のXYZタイルである。 ([maps.gsi.go.jp](https://maps.gsi.go.jp/development/ichiran.html))

---

## 2-3 地図を表示

ブラウザに、

```text
地理院地図_淡色
```

が出てくる。

ダブルクリック。

日本地図が表示されれば成功。

---

## STEP 3　対象地域まで移動

QGIS上部の検索機能や地図操作を使って対象地域まで移動する。

最初は、

```text
地理院地図
```

だけがレイヤパネルにある状態でよい。

---

## STEP 4　解析用CRSを決める

画面右下に現在のCRSが表示されている。

例えばXYZタイル表示中は、

```text
EPSG:3857
```

などになっている場合がある。

これはWeb地図表示には使えるが、今回の距離・バッファ等の解析用データには別途、対象地域に適した平面直角座標系を使用する。

北海道では研究対象地域によってXI・XII・XIII系が異なるため、対象地域を確認して適切な系を選択する。

国土地理院の適用区域一覧を参照する。 ([vldb.gsi.go.jp](https://vldb.gsi.go.jp/sokuchi/surveycalc/surveycalc/jpc/jpc.htm))

**ここでは仮に「研究対象地域に適した平面直角座標系」を選択するものとして先へ進む。**

---

## STEP 5　GeoPackageを作る

ここからデータ本体を作る。

上部メニュー：

**レイヤ**

↓

**レイヤを作成**

↓

**新規GeoPackageレイヤ**

をクリック。

QGIS公式でもこの手順でGeoPackageレイヤを作成できる。 ([docs.qgis.org](https://docs.qgis.org/3.44/ja/docs/user_manual/managing_data_source/create_layers.html))

---

## STEP 6　`source_map`を作る

まず紙地図管理テーブルを作る。

「データベース」の右側にある

**…**

をクリック。

保存場所：

```text
higuma_winter_project/qgis/
```

ファイル名：

```text
higuma_winter.gpkg
```

保存。

---

## 6-1 テーブル名

テーブル名：

```text
source_map
```

---

## 6-2 ジオメトリ

ジオメトリタイプ：

**ジオメトリなし**

を選択。

つまり`source_map`は地図上に表示されない普通のデータテーブルである。

---

## STEP 7　source_mapのフィールドを作る

「新規フィールド」のところから以下を順番に追加する。

---

### ① map_uuid

名前：

```text
map_uuid
```

型：

**テキスト**

長さ：

```text
40
```

↓

**フィールドリストに追加**

---

### ② map_code

```text
map_code
```

型：

テキスト

長さ：

```text
10
```

追加。

---

### ③ year

```text
year
```

型：

**整数**

追加。

---

### ④ survey_date

```text
survey_date
```

型：

**日付**

追加。

---

### ⑤ observer

```text
observer
```

型：

テキスト

長さ：

```text
100
```

---

### ⑥ map_scale

```text
map_scale
```

型：

整数

---

### ⑦ source_no

```text
source_no
```

型：

テキスト

---

### ⑧ note

```text
note
```

型：

テキスト

長さ：

```text
500
```

---

### ⑨ created_at

```text
created_at
```

型：

**日時**

---

全部入ったら、

**OK**

をクリック。

---

## STEP 8　source_mapのUUID自動生成を設定

レイヤパネルに、

```text
source_map
```

が出てくる。

右クリック。

↓

**プロパティ**

↓

左側：

**属性フォーム**

を開く。

---

## 8-1 map_uuidを選択

フィールド一覧から、

```text
map_uuid
```

をクリック。

「デフォルト値」の欄を探す。

そこに、

```text
uuid('WithoutBraces')
```

を入力。

QGISの`uuid()`関数は一意なUUID文字列を生成できる。 ([docs.qgis.org](https://docs.qgis.org/3.44/ja/docs/user_manual/expressions/functions_list.html))

---

## 8-2 制約

同じ画面で、

**Not null**

にチェック。

**Unique**

にチェック。

---

## STEP 9　map_codeの制約

`map_code`

を選択。

Not null：

**ON**

Unique：

**ON**

「式による制約」に、

```text
regexp_match("map_code", '^M[0-9]{4}$')
```

を入力。

制約強度：

**Hard**

とする。

これで、

```text
M0001
M0002
```

は保存できるが、

```text
M01
map1
```

などは保存できなくなる。

---

## STEP 10　yearを必須にする

`year`

を選択。

**Not null**

ON。

必要なら制約式：

```text
"year" >= 1900 AND "year" <= 2100
```

Hard。

---

## STEP 11　created_atを自動入力

`created_at`

を選択。

デフォルト値：

```text
now()
```

を入力。

OK。

---

## STEP 12　survey_routeを作る

再び、

**レイヤ → レイヤを作成 → 新規GeoPackageレイヤ**

を選択。

今回は既存のGeoPackageを指定する。

データベース：

```text
higuma_winter.gpkg
```

テーブル名：

```text
survey_route
```

ジオメトリ：

**LineString / ライン**

CRS：

先ほど決めた、

**研究対象地域に適した平面直角座標系**

を選択。

---

## STEP 13　survey_routeフィールド

以下を追加する。

| 名前 | 型 |
|---|---|
| `route_uuid` | Text 40 |
| `route_code` | Text 10 |
| `map_fk` | Text 40 |
| `route_conf` | Text 1 |
| `route_err_m` | Integer |
| `route_type` | Text 30 |
| `note` | Text 500 |
| `created_at` | DateTime |

全部入れたら、

**OK**

---

## STEP 14　survey_routeのUUID設定

`survey_route`

右クリック

↓

**プロパティ**

↓

**属性フォーム**

↓

`route_uuid`

を選択。

デフォルト：

```text
uuid('WithoutBraces')
```

Not null：

ON

Unique：

ON

---

## STEP 15　route_code

`route_code`

を選択。

Not null：

ON

Unique：

ON

制約式：

```text
regexp_match("route_code", '^R[0-9]{4}$')
```

Hard。

---

## STEP 16　route_confをプルダウンにする

`route_conf`

を選択。

「ウィジェットタイプ」を、

**値マップ / Value Map**

に変更。

値を登録する。

```text
表示                 保存値

A：高精度             A
B：中程度             B
C：低精度             C
```

Not null：

ON。

制約式：

```text
"route_conf" IN ('A','B','C')
```

Hard。

---

## STEP 17　route_typeをプルダウン化

`route_type`

↓

Value Map

として、

```text
林道        forest_road
道路        road
登山道      trail
沢          stream
その他      other
不明        unknown
```

を登録する。

---

## STEP 18　bear_traceを作る

**レイヤ → レイヤを作成 → 新規GeoPackageレイヤ**

データベース：

```text
higuma_winter.gpkg
```

テーブル名：

```text
bear_trace
```

ジオメトリ：

**Point / ポイント**

CRS：

`survey_route`と同じ平面直角座標系。

---

## STEP 19　bear_traceフィールド

以下を作成。

| 名前 | 型 |
|---|---|
| `trace_uuid` | Text 40 |
| `trace_code` | Text 10 |
| `route_fk` | Text 40 |
| `trace_type` | Text 20 |
| `loc_conf` | Text 1 |
| `loc_error_m` | Integer |
| `event_id` | Text 20 |
| `trace_age` | Text 20 |
| `note` | Text 500 |
| `created_at` | DateTime |

OK。

---

## STEP 20　trace_uuid

`bear_trace`

右クリック

↓

プロパティ

↓

属性フォーム

↓

`trace_uuid`

デフォルト：

```text
uuid('WithoutBraces')
```

Not null：

ON

Unique：

ON。

---

## STEP 21　trace_code

`trace_code`

Not null：

ON

Unique：

ON

制約：

```text
regexp_match("trace_code", '^T[0-9]{4}$')
```

Hard。

---

## STEP 22　trace_typeをプルダウン化

`trace_type`

↓

Value Map

以下を登録。

```text
表示        保存値

足跡        footprint
糞          feces
背擦り      rub
爪痕        claw
その他      other
不明        unknown
```

Not null：

ON。

制約：

```text
"trace_type" IN (
'footprint',
'feces',
'rub',
'claw',
'other',
'unknown'
)
```

Hard。

---

## STEP 23　loc_confをプルダウン化

`loc_conf`

↓

Value Map

```text
A：ほぼ特定可能       A
B：概ね特定可能       B
C：位置が曖昧         C
```

Not null：

ON。

制約：

```text
"loc_conf" IN ('A','B','C')
```

Hard。

---

## STEP 24　trace_age

`trace_age`

↓

Value Map

```text
新鮮        fresh
古い        old
不明        unknown
```

これは必須にはしない。

---

## STEP 25　created_at

`created_at`

デフォルト：

```text
now()
```

---

## STEP 26　source_map → survey_routeのRelationを作る

ここから3つのデータをつなぐ。

上部メニュー：

**プロジェクト**

↓

**プロパティ**

↓

**リレーション / Relations**

を開く。

QGISではこの画面から1対多relationを設定できる。 ([docs.qgis.org](https://docs.qgis.org/3.44/en/docs/user_manual/working_with_vector/joins_relations.html))

「＋」または

**リレーションを追加**

をクリック。

---

## Relation 1

名前：

```text
map_route
```

参照される親レイヤ：

```text
source_map
```

親フィールド：

```text
map_uuid
```

子レイヤ：

```text
survey_route
```

子フィールド：

```text
map_fk
```

つまり、

```text
source_map.map_uuid
        ↓
survey_route.map_fk
```

を結ぶ。

---

## STEP 27　survey_route → bear_traceのRelation

もう1つ追加。

名前：

```text
route_trace
```

親：

```text
survey_route
```

親フィールド：

```text
route_uuid
```

子：

```text
bear_trace
```

子フィールド：

```text
route_fk
```

つまり、

```text
survey_route.route_uuid
        ↓
bear_trace.route_fk
```

となる。

適用。

OK。

---

## STEP 28　source_mapの表示名を設定

このままだとRelation選択時にUUIDが表示される可能性がある。

そこで、

`source_map`

右クリック

↓

**プロパティ**

↓

**表示 / Display**

またはDisplay Expressionの設定箇所を開く。

Display Expression：

```text
"map_code" || ' : ' || to_string("year")
```

とする。

すると、

```text
M0001 : 1998
M0002 : 1999
```

のように表示できる。

---

## STEP 29　survey_routeの表示名

同様に、

`survey_route`

のDisplay Expressionを、

```text
"route_code"
```

にする。

これで、

```text
R0001
R0002
```

として選択できる。

---

## STEP 30　map_fkをRelation Referenceにする

`survey_route`

↓

プロパティ

↓

属性フォーム

↓

`map_fk`

を選択。

Widget Type：

**Relation Reference**

を選択。

対象Relation：

```text
map_route
```

を選ぶ。

これでUUIDを手入力する必要がなくなる。

入力画面では、

```text
Source Map

[M0023 : 1998 ▼]
```

のように選択できる。

---

## STEP 31　route_fkもRelation Referenceにする

`bear_trace`

↓

プロパティ

↓

属性フォーム

↓

`route_fk`

Widget：

**Relation Reference**

Relation：

```text
route_trace
```

を選択。

入力画面で、

```text
Survey Route

[R0023 ▼]
```

と選べるようになる。

---

## STEP 32　痕跡の色分け

`bear_trace`

右クリック

↓

**プロパティ**

↓

**シンボロジ**

上部を、

**分類された / Categorized**

に変更。

値：

```text
trace_type
```

↓

**分類**

これで、

```text
footprint
feces
rub
claw
```

ごとに異なるシンボルを設定できる。

作業時に判別しやすい形・色を選択する。

---

## STEP 33　痕跡IDを表示

`bear_trace`

↓

プロパティ

↓

**ラベル**

↓

**単一ラベル**

値：

```text
trace_code
```

適用。

これでポイント横に、

```text
● T0001
```

と表示される。

---

## STEP 34　ルートIDを表示

`survey_route`

↓

プロパティ

↓

ラベル

↓

単一ラベル

↓

```text
route_code
```

これでルート上に、

```text
R0001
```

が表示される。

---

## STEP 35　試しに紙地図1枚を登録する

ここから実データ入力。

まず、

`source_map`

右クリック。

↓

**属性テーブルを開く**

↓

**編集モード**

↓

**地物追加**

または新規レコード追加。

例えば、

```text
map_code       M0001
year           1998
survey_date    NULL
observer       A
map_scale      25000
source_no      001
note
```

を入力。

`map_uuid`は自動生成。

`created_at`も自動生成。

保存。

---

## STEP 36　調査ルートを1本描く

`survey_route`

をクリック。

↓

**編集モード**

↓

**ライン地物を追加**

紙地図を横に置く。

地理院地図を見ながら、

```text
クリック
 ↓
クリック
 ↓
クリック
 ↓
クリック
```

してルートをトレースする。

最後に右クリック。

属性フォームが出る。

```text
route_code
R0001

source map
M0001 : 1998

route_conf
A

route_err_m
30

route_type
forest_road
```

保存。

---

## STEP 37　痕跡を1地点入力

`bear_trace`

を選択。

↓

編集モード。

↓

**ポイント地物を追加**

紙地図上の痕跡位置を地理院地図上でクリック。

属性フォーム：

```text
trace_code
T0001

survey route
R0001

trace type
足跡

location confidence
A

location error
30
```

保存。

---

## STEP 38　2個目の痕跡

次の地点をクリック。

```text
trace_code
T0002

survey route
R0001

trace type
糞

location confidence
B

location error
100
```

保存。

---

## STEP 39　紙地図と照合

紙地図：

```text
調査ルート     1
足跡           1
糞             1
```

QGIS：

```text
R0001

T0001 footprint
T0002 feces
```

一致を確認。

---

## STEP 40　編集を保存する

各レイヤで、

**レイヤ編集内容を保存**

をクリック。

その後、

**プロジェクト → 保存**

も実行する。

重要なのは、

```text
higuma_winter.gpkg
```

と

```text
higuma_winter.qgz
```

は別物ということ。

GeoPackageにはデータ、QGZにはプロジェクト設定等が保存される。

両方保存する。

---

## STEP 41　5枚だけ入力する

ここで100枚全部に進まない。

まず、

```text
M0001
M0002
M0003
M0004
M0005
```

だけ入力する。

例えば、

```text
source_map
│
├ M0001
├ M0002
├ M0003
├ M0004
└ M0005

survey_route
│
├ R0001
├ R0002
├ R0003
├ R0004
└ R0005

bear_trace
│
├ T0001
├ T0002
├ T0003
├ ...
└ T0013
```

程度になる。

---

## STEP 42　Pilot QC

5枚入力後に確認する。

### ① ID

```text
M0001
R0001
T0001
```

が重複していない。

### ② Relation

痕跡を選択すると対応する、

```text
R0001
```

が分かる。

さらにR0001から、

```text
M0001
```

まで辿れる。

### ③ 痕跡タイプ

```text
footprint
feces
rub
claw
```

だけになっている。

### ④ 位置精度

すべて、

```text
A
B
C
```

のいずれかが付いている。

### ⑤ 紙地図との個数

紙地図の痕跡数とGISポイント数が一致する。

---

## STEP 43　問題がなければ100枚へ

Pilotで問題がなければ、

```text
紙地図
 ↓
source_map
 ↓
survey_route
 ↓
bear_trace
 ↓
紙地図と照合
 ↓
保存
 ↓
次の紙地図
```

を繰り返す。

---

## STEP 44　バックアップ

作業終了時に、

```text
higuma_winter.gpkg
```

をコピー。

例えば、

```text
backup/

higuma_winter_2026-11-03.gpkg
```

として保存する。

翌日は、

```text
higuma_winter_2026-11-04.gpkg
```

とする。

---

## STEP 45　100枚入力終了時の完成形

例えば、

```text
source_map

約100 records
```

```text
survey_route

約100 lines
```

```text
bear_trace

約200〜300 points
```

となる。

構造として、

```text
M0042
紙地図
 │
 └── R0042
      調査ルート
       │
       ├── T0117 footprint
       ├── T0118 feces
       └── T0119 claw
```

まで追跡可能なら、GISデータベース構築は成功である。

---

## 次段階

ここまで完了したら、原データ入力フェーズを終了する。

次に別工程として、

```text
国土地理院DEM
     ↓
標高
傾斜
斜面方位
northness
eastness
     ↓

道路データ
     ↓
道路距離

植生データ
     ↓
森林タイプ

積雪データ
     ↓
積雪条件
```

を追加する。

さらに、

```text
survey_route
      ↓
50 / 100 / 200 m buffer
      ↓
available points

bear_trace
      ↓
presence points
```

を作れば、IPP等による冬季ヒグマの環境選択性解析へ進める。

**原データ入力段階では、ここまでの解析用変数を手入力しない。**
