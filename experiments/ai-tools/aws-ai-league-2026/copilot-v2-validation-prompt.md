# Copilot実行プロンプト：設計仮説 v1 を3〜4時間で検証

以下の枠内だけを社内GitHub Copilot Agentへ転記する。設計仮説の版はv1、新規構成の名前はv2。関連：[従来の高ROI改善プロンプト](copilot-roi-prompt.md)。

```text
AWS AI League 2026 Agentic AI Challengeの「設計仮説 v1」を3〜4時間でMVP検証してください。目的は、Treasureへの到達余力を常に確保しながら、時間・Life・Tokenの予算内で期待総得点を最大化すること。原則は Simple Architecture, Rich Model。完全再設計を目指さず、現物コード・設定・Tool schema・競技ルール・ログで仮説を確かめ、許可済み範囲で実装・実験まで進めてください。未確認仕様は推測で埋めない。

【保持と切替】
既存Supervisor / Pathfinder / Lambda / Prompt / 設定はすべて保持し、削除・上書きしない。最初に現行をv1 Baselineとして保存する。commit、Agent・Lambdaのversion/ID、Prompt、Tool接続、設定、実行条件・結果、復元手順を社内に記録し、旧構成が再実行できる状態にする。Git外の設定も対象。
新構成は既存命名に沿うv2として横に新規作成する。既存Lambdaは変更せず再利用し、変更が必要ならv2側のコピーまたは独立versionに限定する。最後に接続先だけを付け替えられるよう、feature flag / alias / configのうち現環境で最も簡単な方法を1つ選び、切替前後の値と戻す操作を保存する。
MemoryとGuardrailの構造は変更しない。Guardrailは必要ならパラメータ調整のみとし、旧構成への影響を避けて変更前後を記録する。

【試す最小構成】
- Mission Planner（Supervisor後継候補）：Game Stateから次の目的を決め、必要能力を選ぶことに限定する。
- Route Planner（Pathfinder後継候補）：指定目的地までの移動方針を決める。Route Engineは既存Lambdaを最大限再利用し、shortest / coin-aware / safe / safe-coin等をStrategyとして扱う。新規Lambdaを乱造しない。
- Game Model：Game State（位置・残時間・Life・Token・鍵・扉・Challenge/Boss状態等）、Strategy Graph（鍵→扉等の依存関係と候補の価値・コスト・リスク）、Treasure Reserveを小さなデータ構造と関数で試す。
- Treasure Reserve = Remaining Time - Estimated Time To Treasure - Safety Margin。推定時間には移動・必須処理・呼出し遅延を含め、寄り道後も余力とLife/Token予算を保てるか判断する。推定不能を0扱いしない。余力低下時は寄り道を打ち切る。
- Missionは固定、Planは目的地の順序、Routeは具体座標に分離する。Planの再計画は毎ターンではなく、Challenge成功/失敗、Key取得、Door開放、Boss完了、Life低下、残時間低下、経路逸脱等のイベントで行う。実APIが経路事前固定なら対応可能な境界に限定し、制約を報告する。
- Haiku前提でSystem PromptとContextを小さくする。巨大な全マップ・全ルールを毎回渡さず、必要状態と候補行動の期待得点・コスト・リスク・前提条件を圧縮して渡す。

【優先順位と時間配分：合計180〜240分】
① baseline保存・基準run（30〜40分）：現行を記録し、同条件で比較できる基準結果を採る。各処理を KEEP / DIRECT LLM / MOVE TO CODE / TOOL REQUIRED / DELETE で短く棚卸しする。DELETEはv2からの不採用であり、既存資産の削除ではない。配置は Context + LLM出力 + Tool call + Tool結果再投入 + Retry + Latency + Credit の総コストと正答率で判断する。
② v2骨格（50〜70分）：上記の責務分離と最小Game Modelを接続する。抽象化や改名そのものに時間を使わない。
③ Boss実行（30〜40分）：専用Agent新設の前に、Mission Planner + 既存Tool + Route Planner + Game Stateの統合テストとしてv2で1回通す。到達・問題文・Tool入出力・正誤・時間・失敗原因を記録する。未到達や失敗を成功扱いしない。
④ 高ROI改善1〜2個（30〜40分）：実ログから期待効果/実装時間/リスクで選び、1個ずつ変更する。遅れたら2個目を省く。
⑤ 旧構成との比較・切替判断（最後の40〜50分を確保）：可能なら同一map/seed・モデル・設定で score / Treasure到達 / tool calls / context・token / credit / latency / retry・error とBoss成否を比較する。未計測・推定を区別し、tokenをcredit実測値に代用しない。1回だけなら暫定評価とする。

改善と復元手順を確認できた場合に、許可済み範囲で最後に接続先をv2へ切り替える。不明・悪化・時間不足なら旧接続を維持し、試験切替済みなら戻す。社内コード・設定・ログを公開リポジトリへ送らない。実行不能なら原因を記録し、アクセスできない操作だけユーザーへ依頼する。
終了時は Current State / v1 Baseline / v2 Changes / Result / Rollback方法 / Next Best Action / User Action Needed を各1〜3行で報告する。実際の接続先、比較値・未検証事項、再現可能な戻し方を含め、必要操作がなければ「なし」と書く。
```
