# Copilot実行プロンプト：Generic One-shot Planner v2

最新版（2026-09-10）。枠内だけを社内GitHub Copilot Agentへ一括転記する。予選の静的Mapはユーザー観察に基づく検証前提。旧案は[設計仮説 v1](copilot-v2-validation-prompt.md)として保持。

```text
AWS AI League 2026 Agentic AI Challengeの現行リポジトリ・設定・Tool schema・ログを確認し、3〜4時間でGeneric One-shot Plannerの最小v2を既存構成の横に実装・比較してください。使命は「Treasureへの到達余力を確保しつつ、時間・Life・Tokenの制約内で期待総得点を最大化する」。今回の予選は敵や地形が実行中にほぼ変わらないという前提を現物で確認し、Dynamic Plannerを作り込まない。未確認仕様は推測で埋めず、許可済み範囲で実験まで進める。

【保持・切替】
既存Supervisor / Pathfinder / Lambda / Prompt / 設定・旧プロンプトは削除・上書きせず、v1 Baselineとして保持する。commit、Agent/Lambdaのversion・ID、Prompt、Tool接続、Git外の設定、実行条件・結果、復元手順を社内に保存する。変更が必要な資産はv2コピーまたは独立versionに限定。最後にconfig / alias等の最も簡単な方法1つで接続先だけ切替可能にし、旧値・新値・戻す操作を記録する。
MemoryとGuardrailの構造は変更しない。Guardrailは必要ならパラメータ調整のみ。baselineに影響しない形で変更前後を記録する。

【最小設計】
- Missionは上記の固定目的、Planは開始時に生成する訪問順序、Routeは具体的な座標列として分離する。
- 開始時にMapからChallenge / Coin / Key / Door / Boss / Treasure / Risk / Cost / Rewardと依存関係を小さなGame Model / Strategy Graphに表現する。鍵→扉などの前提、期待報酬、移動・処理コストを評価し、Treasure到達とLife/Token予算を守る高得点Mission PlanとRouteを一度生成する。Graphは訪問順序・依存関係・価値最大化に使い、動的制御基盤にしない。
- 固定座標をハードコードせず、未知Mapから毎回計画を生成する汎用アルゴリズムにする。準決勝/決勝への持ち越しを意識し、ルール差は設定に分離する。巨大な探索基盤は不要。
- Supervisor後継は薄いMission PlannerまたはNavigator。初期計画と例外対応に絞り、Haikuに毎ターン「次どこ？」を判断させない。全Map・全ルールの毎回再投入を避ける。
- Pathfinder後継のRoute Plannerは開始時の訪問順序と経路計画を担う。Route Engineは既存Lambdaを最大限再利用し、shortest / coin-aware / safe / safe-coin等をStrategyとして扱う。新規Lambdaを乱造しない。
- 通常はRouteを決定論的に実行する。再計画はBoss/Challenge失敗、想定外Life減少、想定以上の時間消費、Tool失敗など計画前提が崩れた場合だけ。予定どおりの成功・Key取得・Door開放では再計画しない。実APIの実行・再計画可能な境界に従う。
- Treasure Reserve = Remaining Time - Estimated Time To Treasure - Safety Marginを評価指標にしてよい。推定時間には移動・必須処理・Tool遅延を含め、推定不能を0にしない。余力不足は前提崩れとして寄り道を打ち切る。過剰な状態管理は作らない。

【優先順：合計180〜240分】
① baseline保存・棚卸し（20〜30分）：既存Supervisor Prompt / Tool / Lambdaの各処理をKEEP / DIRECT LLM / MOVE TO CODE / TOOL REQUIRED / DELETEに短く分類。DELETEはv2での不採用であり旧資産の削除ではない。「コード化できるからコード」ではなく、Context + LLM出力 + Tool call + Tool結果再投入 + Retry + Latency + Creditの総コストと正答率で配置を判断する。
② Boss実行（25〜35分）：Boss未経験なのでv2実装前に既存構成で1回試し、問題・Tool入出力・回答・正誤・時間・失敗原因と基準scoreを採る。未到達・実行不能を成功扱いせず記録する。
③ Generic One-shot Planner最小v2（55〜70分）：上記の初期計画と決定論的実行を最小実装。抽象化・改名に時間を使わない。
④ 既存Route Engine接続（20〜30分）：v2を接続して試験実行する。未知Mapでも同じアルゴリズムで依存関係とTreasure到達経路を生成できるか確認する。
⑤ 高ROI改善1〜2個（20〜25分）：ログから期待効果/実装時間/リスクで選ぶ。遅れたら省き、比較時間を守る。
⑥ 比較・Rollback確認（最後の40〜50分）：可能なら同一map/seed・モデル・設定でscore / Treasure到達 / tool calls / context・token / credit / latency / retry・errorとBoss成否を比較する。未計測・推定を区別し、tokenをcredit実測に代用しない。1回だけなら暫定評価とする。

改善と復元手順を確認できれば最後に接続先をv2へ切り替える。不明・悪化・時間不足なら旧接続を維持し、試験切替済みなら戻す。社内コード・設定・ログは公開リポジトリへ送らない。アクセス不能な操作だけユーザーへ依頼する。
終了時はCurrent State / v1 Baseline / v2 Changes / Result / Rollback / Next Best Action / User Action Neededを各1〜2行で報告。実接続先、比較値・未検証事項、再現可能な戻し方を含め、必要操作がなければ「なし」と書く。
```
