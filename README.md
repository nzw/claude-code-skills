# claude-code-skills

[Claude Code](https://code.claude.com/docs/en/skills) のスキルを置いています。スキルは、Claude Code に守らせたい手順を 1 つのファイルに書いたものです。`/<スキルの名前>` と打つと、Claude Code がそれを読んで、その手順で作業します。

| スキル | 何をするか |
| --- | --- |
| [ui-check](ui-check/SKILL.md) | 配色・表・一覧・画面の文言を変えたあと、「直しました」と返す前に 5 つの確認を通させる。実データを全件描いて測る、明暗と色を並べて見る、見ていないものも報告する、など |
| [appstore-capture](appstore-capture/SKILL.md) | App Store 提出用のスクリーンショットを、iOS シミュレータから提出する寸法のまま撮る。見本のデータを入れ、透過を外し、どの枠に上げる画像かまで出す |

## 入れ方

使うスキルのフォルダを、個人のスキルの置き場所（`~/.claude/skills/`）に写します。どのリポジトリを開いても使えます。1 つのリポジトリだけで使うなら、そのリポジトリの `.claude/skills/` に写します。

```bash
git clone https://github.com/nzw/claude-code-skills.git
cp -R claude-code-skills/ui-check ~/.claude/skills/
cp -R claude-code-skills/appstore-capture ~/.claude/skills/
```

Claude Code で、たとえば `/ui-check 2 列の表の幅` のように、内容を添えて呼びます。

## 使う前に

- **ui-check** は、Flutter のアプリで使っていたものから、そのアプリだけのパスを外した版です。「実データを全件描く」テストと撮り方の手順は、使うプロジェクトのものに書き換えてください
- **appstore-capture** は macOS・Xcode（iOS シミュレータ）・Flutter・Python の [Pillow](https://pillow.readthedocs.io/en/stable/) が要ります。`capture.sh` はシェルスクリプトで、あなたの権限で動きます。入れる前に中を読んでください
- App Store が受け付ける寸法は変わります。スキルの表は 2026-10-14 に App Store Connect の画面で、2026-11-25 に Apple の [Screenshot specifications](https://developer.apple.com/help/app-store-connect/reference/app-information/screenshot-specifications) で確かめた値です。上げる前に、枠に書かれた対応サイズと突き合わせてください

## ライセンス

[MIT](LICENSE)
