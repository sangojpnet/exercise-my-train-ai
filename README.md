# exercise-my-train-ai


## このアプリについて
- **これは何か**：Streamlitアプリ「マイ乗換案内AI」。駅名から路線・方面を選び、ODPT APIまたはSQLiteの手動時刻表で直近の発車を表示し、Geminiで駅員風の案内文を出す。
- **状態**：不明
- **関連リポジトリ**：なし
- **主な技術**：Python, Streamlit, SQLite, ODPT API, Gemini API
- **実行環境**：

## 実装されていること

リポジトリ内の実行ファイルは `app.py` のみ。Jupyterノートブックや機械学習の学習コードは無い。

画面タイトルは「マイ乗換案内 AI」。流れは次の3ステップ。

1. 駅名を入力する。SQLite（`transport_data_v3.db`）の `stations` と `manual_timetables` から路線一覧を出す。
2. 路線を選ぶ。API由来か手動登録かを表示する。JRと判定した路線がある場合、サイドバーでJR遅延を確認できる旨を出す。
3. 方面を選ぶ。現在時刻（JST）以降の発車を最大3件表示する。表示は電光掲示板風のHTMLで、種別はコード上「各駅停車」固定。続けて Gemini（`gemini-2.5-flash`）が駅員風の案内文を生成する。

サイドバー:

- 「JR運行情報を確認」: ODPT `odpt:TrainInformation`（`odpt.Operator:JR-East`）を取得し、本文に「平常」を含まない件を Gemini に渡して要約する。
- 「マイ時刻表 手動登録」: 駅名・路線・方面・発車時刻・行き先・平日/土休日を `manual_timetables` へ INSERT する。

時刻表の取得元:

- `source == "api"`: ODPT `odpt:StationTimetable`。方面名は SQLite の `directions` を参照する。
- 手動登録: `manual_timetables` を曜日区分（Weekday / SaturdayHoliday）で読む。

`stations` と `directions` の作成処理は `app.py` に無い。既存の `transport_data_v3.db` を前提にしている。`manual_timetables` だけ `CREATE TABLE IF NOT EXISTS` する。

## 起動

環境変数:

- `ODPT_API_KEY`
- `GEMINI_API_KEY`

```bash
pip install -r requirements.txt
streamlit run app.py
```

`app.py` が import している第三者ライブラリは `streamlit`、`requests`、`google-generativeai`。`sqlite3` は標準ライブラリ。`requirements.txt` に `pandas` があるが、`app.py` は import していない。
