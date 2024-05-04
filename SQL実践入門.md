# 4. 集約とカット -集合の世界
- CASE式とGROUP BYの応用
  GROUP BYを使って集約操作を行った場合、SELECTに書けるのは
  - 定数
  - GROUP BYで指定した集約キー
  - 集約関数
  に限定される
## 4.2 カット
GROUP BY は「集約」の他にも「カット」という機能もある
これは母集合である元のテーブルを小さな部分集合に切り分けること
リスト4−８(人物テーブルから名前の頭文字のアルファベットごとに何人テーブルに存在するかを集計する)
これはPersons集合を部分集合に分けて、それぞれの要素するを調べるということ
name列をSUBSTRINGで先頭一文字だけを抜き出し、それをGROUP BYのキーに指定すれば、カット完了

```sql
SELECT SUBSTRING(name, 1, 1) AS label,
        COUNT(*)
  FROM Persons
GROUP BY SUBSTRING(name, 1, 1);
```
- パーティション
こういうGROUP BY句でカットして作られた一つ一つの部分集合は、数学的には「類」(partition)と呼ばれる
これは互いに重複する要素を持たない部分集合のことで、そのまま「パーティション」と呼ぶことができる

カット基準となるキーを、GROUP BY句とSELECT句の両方に書くのがルール

## PARTITION BY句を使ったカット
「カット」機能しかないGROUP BY

