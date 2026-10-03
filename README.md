# japanese-llm-security

[![DOI](https://zenodo.org/badge/DOI/10.5281/zenodo.23122807.svg)](https://doi.org/10.5281/zenodo.23122807)

[English](#english) | [日本語](#日本語)

## English

A map of **Japanese-language resources for LLM / agent security** and the tools to use them together. The aim is one place
to start: which dataset or benchmark answers which question, how they relate to their English originals, and how to run them.

### Indirect prompt injection and agent security (this project's focus)

| Resource | What it is | Size | Origin / license | Kind |
|---|---|---|---|---|
| [agentdojo-ja](https://github.com/masahiroid/agentdojo-ja) | Japanese localization of the AgentDojo benchmark: dynamic, state-based agent evaluation (banking, slack, travel, workspace), 25 Japanese attacks, defenses | 97 user / 31 injection tasks | [AgentDojo](https://github.com/ethz-spylab/agentdojo), MIT | localization + extras |
| [InjecAgent-ja](https://huggingface.co/datasets/masahiroid/injecagent-ja) | Tool-using agents hit by injections in tool responses (direct harm / data stealing), incl. Japanese tool definitions | 1,054 cases x 2 settings | [InjecAgent](https://github.com/uiuc-kang-lab/InjecAgent), MIT | translation |
| [BIPIA-attacks-ja](https://huggingface.co/datasets/masahiroid/bipia-attacks-ja) | Attack instructions of BIPIA (text and code attacks); contexts are not redistributed | 250 attacks + 50 Japanese-native (v0.2) | [BIPIA](https://github.com/microsoft/BIPIA), MIT | translation + original extras |
| [Nemotron-RL-Agentic-IPI-ja](https://huggingface.co/datasets/masahiroid/nemotron-agentic-ipi-ja) | RL / eval records with deterministic trace verification across 9 enterprise domains | 1,272 records | [NVIDIA](https://huggingface.co/datasets/nvidia/Nemotron-RL-Agentic-Indirect-Prompt-Injection-v1), CC-BY-4.0 | translation |
| [Japanese indirect prompt-injection probes](https://huggingface.co/datasets/masahiroid/japanese-indirect-prompt-injection-probes) | Small probe set written in Japanese: keigo-style injection, full-width/hiragana/romaji obfuscation, fake 【システム】 markers, position axis | 60 probes (10 categories) | original, CC-BY-4.0 | original |

### Tools

| Tool | Use |
|---|---|
| [model-audit-lite](https://github.com/masahiroid/model-audit-lite) | File audit (pickle / custom code / checksums), conversion-integrity compare, probe runner (`--probe-set ja-injection`), CycloneDX ML-BOM with conversion lineage (`bom`) |

### Which one for which question

- *How does my agent behave end to end with tools and state?* -> **agentdojo-ja** (dynamic; utility and attack success).
- *How many tool-response injections does the agent follow, at scale?* -> **InjecAgent-ja** (static cases, English tool names).
- *Does a model follow instructions hidden in a document I give it?* -> **BIPIA-attacks-ja** (attacks; bring your own contexts) or the **probes** (self-contained, minutes).
- *Training or RL data with a deterministic verifier?* -> **Nemotron-RL-Agentic-IPI-ja**.
- *Is this converted model repo safe to ship, and did conversion change its behavior?* -> **model-audit-lite**.

### Quick start

```python
from huggingface_hub import hf_hub_download
import json
cases = json.load(open(hf_hub_download("masahiroid/injecagent-ja", "test_cases_dh_base_ja.json", repo_type="dataset")))
tools = json.load(open(hf_hub_download("masahiroid/injecagent-ja", "tools_ja.json", repo_type="dataset")))

# environment schemas differ per domain, so read the JSONL directly rather than through Arrow
nemotron = [json.loads(l) for l in open(hf_hub_download("masahiroid/nemotron-agentic-ipi-ja", "train.jsonl", repo_type="dataset"))]

probes = hf_hub_download("masahiroid/japanese-indirect-prompt-injection-probes", "probes.jsonl", repo_type="dataset")
```

```bash
pip install model-audit-lite
model-audit-lite probe your/model --probe-set ja-injection --max-tokens 300

pip install git+https://github.com/masahiroid/agentdojo-ja
python -m agentdojo_ja.run --model-id <id> --suite all --language ja    # needs an OpenAI-compatible endpoint
```

### How these relate to the originals

Everything translated here is **translation-first (v0.1)**: faithful, machine-translated and automatically validated (structure,
identifiers, argument literals), not yet reviewed by a native-speaker panel, and not localized (names, currencies and services stay as in
the original). The next versions add Japan-specific variants (keigo and indirect phrasing, full-width / kana / kanji mixtures, yen and
Japanese services). agentdojo-ja and the probes already include such Japan-specific material. Verifiers of the originals depend on
English tool names and argument values, which is why those are kept verbatim.

### Related: in-browser demo and on-device pieces

[Ruri Atlas](https://huggingface.co/spaces/masahiroid/ruri-atlas) is a WebGPU demo (3-D map of Japanese semantic search) built on our ONNX rerankers ([xsmall-v2](https://huggingface.co/masahiroid/japanese-reranker-xsmall-v2-onnx-web), [small-v2](https://huggingface.co/masahiroid/japanese-reranker-small-v2-onnx-web)); the Core ML / TFLite / MLX conversions and the Swift / Kotlin RAG libraries are listed on the author's [Hugging Face profile](https://huggingface.co/masahiroid).

### Related Japanese safety datasets (not ours)

These cover *harmful requests / refusal behavior*, a different question from agent injection: [AnswerCarefully](https://huggingface.co/datasets/llm-jp/AnswerCarefully) (llm-jp),
[japanese-multiturn-safety](https://huggingface.co/datasets/sbintuitions/japanese-multiturn-safety) (SB Intuitions),
[llm-safety-japanese-multiturn-dataset](https://huggingface.co/datasets/APTO-001/llm-safety-japanese-multiturn-dataset) (APTO).

### Licenses and citation

Each resource keeps its original license (MIT / CC-BY-4.0) with attribution; see the individual cards. Please cite the originals:
AgentDojo (Debenedetti et al., NeurIPS D&B 2024), InjecAgent (Zhan et al., ACL Findings 2024), BIPIA (Yi et al., 2023), and NVIDIA's Nemotron dataset.
This hub is an index; it contains no data. The hub itself (this index and its documentation) is licensed under Apache-2.0; see [LICENSE](LICENSE).

### Contributing

Issues for awkward translations, wrong labels or missing resources are welcome. Planned: Japan-specific variants of each set, a reviewer pass by
native speakers, and a combined runner.

---

## 日本語

**LLM／エージェントのセキュリティに関する日本語リソース**の地図と、それらを組み合わせて使うためのツールの案内です。
「どのデータセット／ベンチマークが何の問いに答えるか」「英語の原本との関係」「実行方法」を1か所で分かるようにすることが目的です。

### 間接プロンプトインジェクション／エージェントのセキュリティ（本プロジェクトの焦点）

| リソース | 内容 | 規模 | 出典・ライセンス | 種別 |
|---|---|---|---|---|
| [agentdojo-ja](https://github.com/masahiroid/agentdojo-ja) | AgentDojoの日本語ローカライズ。動的・状態ベースのエージェント評価（banking / slack / travel / workspace）、日本語攻撃25種、防御 | user 97 / injection 31 | [AgentDojo](https://github.com/ethz-spylab/agentdojo)、MIT | ローカライズ＋独自拡張 |
| [InjecAgent-ja](https://huggingface.co/datasets/masahiroid/injecagent-ja) | ツール応答に仕込まれた注入（直接的な害／データ窃取）。日本語のツール定義つき | 1,054件×2設定 | [InjecAgent](https://github.com/uiuc-kang-lab/InjecAgent)、MIT | 翻訳 |
| [BIPIA-attacks-ja](https://huggingface.co/datasets/masahiroid/bipia-attacks-ja) | BIPIAの攻撃文（テキスト／コード）。文脈データは再配布しない | 250件 + 日本語固有50件（v0.2） | [BIPIA](https://github.com/microsoft/BIPIA)、MIT | 翻訳＋独自拡張 |
| [Nemotron-RL-Agentic-IPI-ja](https://huggingface.co/datasets/masahiroid/nemotron-agentic-ipi-ja) | 9つの企業ドメインの、決定的なトレース検証つき RL／評価データ | 1,272件 | [NVIDIA](https://huggingface.co/datasets/nvidia/Nemotron-RL-Agentic-Indirect-Prompt-Injection-v1)、CC-BY-4.0 | 翻訳 |
| [日本語 間接プロンプトインジェクション・プローブ](https://huggingface.co/datasets/masahiroid/japanese-indirect-prompt-injection-probes) | 日本語で書いた小規模プローブ。敬語、全角・ひらがな・ローマ字、偽【システム】、位置の軸 | 60件（10カテゴリ） | 独自、CC-BY-4.0 | 独自 |

### ツール

| ツール | 用途 |
|---|---|
| [model-audit-lite](https://github.com/masahiroid/model-audit-lite) | ファイル監査（pickle／カスタムコード／チェックサム）、変換の完全性比較、プローブ実行（`--probe-set ja-injection`）、変換系譜つきCycloneDX ML-BOM（`bom`） |

### どの問いにどれを使うか

- *ツールと状態を持つエージェントの挙動を、エンドツーエンドで見たい* → **agentdojo-ja**（動的。有用性と攻撃成功率）。
- *ツール応答の注入に、エージェントがどれだけ従うかを大量に測りたい* → **InjecAgent-ja**（静的ケース。ツール名は英語）。
- *与えた文書に隠された指示にモデルが従うか* → **BIPIA-attacks-ja**（攻撃文。文脈は各自）または **プローブ**（単体で数分）。
- *決定的な検証器つきの学習／RLデータ* → **Nemotron-RL-Agentic-IPI-ja**。
- *変換したモデルを配布して安全か、変換で挙動が変わっていないか* → **model-audit-lite**。

### クイックスタート

上の English セクションのコードをそのまま使えます。

### 原本との関係

ここで翻訳したものは全て**翻訳優先（v0.1）**です。忠実な機械翻訳と自動検証（構造・識別子・引数リテラル）のみで、ネイティブによる査読は未実施、
ローカライズもしていません（人名・通貨・サービスは原文のまま）。今後の版で、日本固有の変種（敬語・婉曲、全角・かな・漢字の混在、円と日本のサービス）を
追加します。agentdojo-ja とプローブには、すでに日本固有の素材が含まれます。原本の検証器は英語のツール名と引数の値に依存するため、それらは原文のまま保持しています。

### 関連: ブラウザで動くデモとオンデバイスの部品

[Ruri Atlas](https://huggingface.co/spaces/masahiroid/ruri-atlas) は、自作のONNXリランカー（[xsmall-v2](https://huggingface.co/masahiroid/japanese-reranker-xsmall-v2-onnx-web)、[small-v2](https://huggingface.co/masahiroid/japanese-reranker-small-v2-onnx-web)）で動く、WebGPUのデモ（日本語の意味検索を3Dの地図で見る）です。Core ML / TFLite / MLX の変換と、Swift / Kotlin のRAGライブラリは、作者の[Hugging Faceのプロフィール](https://huggingface.co/masahiroid)にまとまっています。

### 関連する日本語の安全性データセット（本プロジェクトのものではありません）

これらは*有害依頼・拒否の挙動*を扱い、エージェントへの注入とは別の問いです: [AnswerCarefully](https://huggingface.co/datasets/llm-jp/AnswerCarefully)（llm-jp）、
[japanese-multiturn-safety](https://huggingface.co/datasets/sbintuitions/japanese-multiturn-safety)（SB Intuitions）、
[llm-safety-japanese-multiturn-dataset](https://huggingface.co/datasets/APTO-001/llm-safety-japanese-multiturn-dataset)（APTO）。

### ライセンスと引用

各リソースは原本のライセンス（MIT／CC-BY-4.0）と帰属を維持します。個別のカードを参照してください。原本を引用してください:
AgentDojo（Debenedetti et al., NeurIPS D&B 2024）、InjecAgent（Zhan et al., ACL Findings 2024）、BIPIA（Yi et al., 2023）、NVIDIAのNemotronデータセット。
このハブは索引であり、データは含みません。ハブ自体（この索引と文書）のライセンスは Apache-2.0 です（[LICENSE](LICENSE) を参照）。

### 貢献

不自然な訳、誤ったラベル、足りないリソースの指摘を歓迎します。予定: 各セットの日本固有の変種、ネイティブによる査読、統合ランナー。
