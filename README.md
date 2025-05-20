# nhk-gogaku-xml
NHK gogaku streaming xml files

このリポジトリは、NHK語学講座のストリーミング配信の `listdataflv.xml` ファイル更新内容を記録するためのものです。  
毎週自動的に更新を行い、過去に配信されたストリーミングのファイル名を記録しています。

## 自動更新について

### ワークフローの概要
現在、このリポジトリでは **GitHub Actions** を使用して、毎週月曜日に自動更新を行っています。  
以下の手順で更新処理が実行されます：

1. リポジトリをチェックアウト
2. `xml-wget.sh` スクリプトを実行（存在する場合）
3. 変更されたファイルをコミット
   - コミットメッセージには更新日の月曜日の日付を含めます
4. リモートリポジトリ（`master` ブランチ）へプッシュ

### 実行スケジュール
- 自動実行: 毎週月曜日 日本時間 10:30 (JST)
  - スケジュール設定: `cron: '30 1 * * 1'` (UTC 1:30 = JST 10:30)
- 手動実行: 必要に応じて **GitHub Actions** の「Run workflow」ボタンから実行可能

### シェルスクリプトについて
リポジトリ内の `xml-wget.sh` スクリプトが実行されることで、`listdataflv.xml` ファイルが更新されます。  
もしスクリプトが存在しない場合、ログに「`xml-wget.sh does not exist`」と出力され、更新はスキップされます。

## 注意点
- XMLファイルに変更がない場合、コミットやプッシュは行われません。
- コミットメッセージの日付は、その週の月曜日として設定されます。

## 開発者向け情報
### 手動実行の手順
1. GitHubリポジトリの「Actions」タブを開きます。
2. `Weekly XML Git Update` ワークフローを選択します。
3. 「Run workflow」ボタンをクリックして手動で実行します。

### ワークフローのファイル
自動更新のワークフローの定義は `.github/workflows/auto-update.yml` に記載されています。

---

以前は外部のサーバにシェルスクリプトを設置し、cron による手動設定で更新を行っていましたが、現在は GitHub Actions による完全自動化を実現しています。




# nhk-gogaku-xml
NHK gogaku streaming xml files

NHK 語学講座のストリーミング配信の listdataflv.xml ファイルです。毎週実行してログを残すようにしているので，過去に配信されたストリーミングのファイル名が分かるようになっています。

## このリポジトリの自動更新
listdataflv.xml の取得とリポジトリへのプッシュについては，もともとは手動で行っていたのですが，忘れることもあるかと思い自動化しました。
外部のサーバに以下のシェルスクリプトを置き，cron で定期的に実行を行っています。
README.md は GitHub で編集するので，最初に pull をしてから push しています。
```
% cat nhk-gogaku-xml-gitupdate.sh
#!/bin/sh

cd /path_to/nhk-gogaku-xml || exit;

git pull origin master

test -f xml-wget.sh && sh xml-wget.sh || echo "File doesn't exists"

git add -A

# Set the date of Monday of this week
# FreeBSD
git commit -am "`date -v-sun -v+1d +'%Y-%m-%d'` Updated files"
# Linux GNU coreutils
# git commit -am "`date -d 'last sunday + 1 day' +'%Y-%m-%d'` Updated files"

git push -u origin master
```
NHK のサーバでは毎週月曜日10:00に更新が行われるので，その後に更新用スクリプトの起動を行います。ここでは1時間後の11:00に起動するようにcrontabに書いています。
上のシェルスクリプトでは，月曜日以降であればいつ動作させても月曜日の日付になるように，日付の取得を行っています。
```
% crontab -l
0 11 * * 1 sh /path_to/nhk-gogaku-xml-gitupdate.sh
```
