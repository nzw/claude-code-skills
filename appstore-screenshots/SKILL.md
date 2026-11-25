---
name: appstore-screenshots
description: >-
  スクリーンショットを App Store 提出用のサイズ（iPhone 6.3"（必須）/6.9" + iPad 13"）に一括変換する。引数にソースディレクトリを指定（例: /appstore-screenshots ~/Desktop/MyApp）。省略時はカレントディレクトリを対象にする。撮り直せるなら /appstore-capture でネイティブ解像度から撮るほうが上。
---

<!-- 配布元: https://github.com/nzw/claude-code-skills
     寸法の表は 2026-10-14 に App Store Connect の画面で、2026-11-25 に Apple の仕様の表で確かめた値です。提出の前に、枠に表示される対応サイズと突き合わせてください。 -->

# /appstore-screenshots — App Store スクリーンショット変換スキル

引数で渡されたディレクトリ（省略時はカレントディレクトリ）内の PNG ファイルを
App Store 提出用のサイズに一括変換する。

> **撮り直せるなら `/appstore-capture` を使う。** 変換は情報が減るだけで増えない。
> このスキルは、撮り直せない画像（実機で撮ったもの・古い版のもの）を各枠へ合わせるときに使う。

---

## 変換対象サイズ

**2026-10-14 に App Store Connect の画面で確かめ、2026-11-25 に Apple の
[Screenshot specifications](https://developer.apple.com/help/app-store-connect/reference/app-information/screenshot-specifications)
と突き合わせた。** 必須の iPhone の枠は **「Dynamic Island 搭載 iPhone（中型ディスプレイ）」**。
以前このスキルが「必須」としていた 6.9" 1290×2796 は、大型の枠では使えるが、
**中型の枠では「サイズが無効です」で弾かれる**（1242×2688 も同じ。6 枚とも弾かれた）。

| フォルダ名 | 解像度 | 縦横比 | 枠 |
|-----------|--------|--------|-----|
| `iPhone_6.3in_1206x2622/` | 1206 × 2622 | 0.460 | **Dynamic Island 搭載 iPhone（中型）。必須** |
| `iPhone_6.9in_1320x2868/` | 1320 × 2868 | 0.460 | Dynamic Island 搭載 iPhone（大型）。任意 |
| `iPad_13in_2064x2752/` | 2064 × 2752 | 0.750 | **iPad 13"。iPad で動くアプリなら必須** |

**フォルダ名に寸法を入れる。** 枠ごとに受け付ける寸法が違い、別の枠のフォルダを
ドラッグしたときに「サイズが無効です」の原因がすぐ分かるようにするため。

各枠が受け付ける寸法（縦。横向きは幅と高さを入れ替えたものも可）:

| 枠 | 受け付ける寸法 |
| --- | --- |
| iPhone 中型（6.3"・6.1"） | 1206×2622 / 1179×2556 |
| iPhone 大型（6.9"・6.7"） | 1320×2868 / 1290×2796 / 1260×2736 |
| iPhone Face ID 大型（6.5"） | 1284×2778 / 1242×2688 |
| iPhone Face ID 中型（6.1"・5.8"） | 1170×2532 / 1125×2436 / 1080×2340 |
| iPad 13" | 2064×2752 / 2048×2732 |

> - **ドロップ欄の下に書いてある対応サイズが正。** Apple は機種を出すたびに枠と寸法を変える。
>   上の表と画面が食い違ったら、画面に合わせてこの表を直す
> - 大きい枠に上げた画像は、小さい枠へ縮小して流用される（流用された画像はその枠では
>   薄く表示され、消せない。消すときは持ち主の枠で消す）
> - 6.5" や 12.9" の枠へ別の絵を出したいときだけ、上の表の寸法で個別に作る

---

## 引き伸ばしと余白の使い分け（重要）

**元の画像と出し先の、縦横の比（横 ÷ 縦）で処理を変える。**

| 元と出し先の比 | 処理 | 例 |
|---------|------|------|
| ほぼ同じ（差が 0.01 以内） | そのまま拡大・縮小する | Face ID・Dynamic Island のある iPhone（0.46 前後）→ iPhone の枠 |
| 違う | **比を保って枠に収め、余白を地の色で埋める** | iPhone（0.46）→ iPad（0.75）、ホームボタンのある iPhone（0.56）→ iPhone の枠 |

**比の違う画像を引き伸ばしてはいけない。** iPhone の画像を iPad の枠へ引き伸ばすと横に 1.6 倍になり、
**円が楕円に、正方形が長方形に潰れる**。DayGrid v2.1.2 で実際にそうなり、
円グラフが目視で分かるほど歪んで提出に耐えなかった。
ホームボタンのある iPhone（SE など）の画像を今の iPhone の枠へ引き伸ばしても、横に 2 割ほど潰れる。

余白の色はアプリの背景色に合わせること（既定 `FFFFFF`）。地と違う色にすると
スクリーンショットの外に額縁が付いたように見える。

> iPad スロットが必要かどうかは `TARGETED_DEVICE_FAMILY` で決まる。
> `"1"` なら iPhone のみで iPad の提出は不要、`"1,2"` なら iPad も必要。
> ```bash
> grep -m1 TARGETED_DEVICE_FAMILY ios/Runner.xcodeproj/project.pbxproj
> ```

---

## 手順

要るもの: macOS（`sips`）と Python 3 の画像ライブラリ Pillow（`pip3 install Pillow`）。
縦向きの PNG だけを対象にする（横向きは飛ばす）。

### ステップ 0 — ソースの確認

```bash
SRC_DIR="${ARG:-$(pwd)}"
find "$SRC_DIR" -maxdepth 1 -name "*.png" | sort
sips -g pixelWidth -g pixelHeight -g hasAlpha "$SRC_DIR"/*.png
```

**アプリの地の色を確認する。** アプリのテーマに書いた背景色、
無ければスクリーンショットの上端の色を見る。以降 `PAD` に 6 桁の 16 進数（`#` なし）で入れる。

### ステップ 1 — 変換と透過の除去

比を見て拡大・縮小か余白かを選び、同時に透過（アルファ）も外す。
**App Store Connect は透過のある画像を受け付けない。** シミュレータの
`simctl io screenshot` の PNG は RGBA で出る。見た目は変わらないので、
寸法が合っていても気づけない（Certly で 11 枚とも弾かれた）。

```bash
PAD=F5F7FA   # アプリの地の色（# なし）。不明なら FFFFFF

python3 - "$SRC_DIR" "$PAD" <<'PY'
import sys, pathlib
from PIL import Image

src, pad = pathlib.Path(sys.argv[1]), sys.argv[2]
bg = tuple(int(pad[i:i + 2], 16) for i in (0, 2, 4))
out = src / "AppStore"
SLOTS = [  # フォルダ名・幅・高さ
    ("iPhone_6.3in_1206x2622", 1206, 2622),
    ("iPhone_6.9in_1320x2868", 1320, 2868),
    ("iPad_13in_2064x2752", 2064, 2752),
]
for f in sorted(src.glob("*.png")):
    im = Image.open(f).convert("RGBA")
    w, h = im.size
    if w >= h:
        print(f"  飛ばした（横向き）: {f.name}")
        continue
    for name, W, H in SLOTS:
        canvas = Image.new("RGB", (W, H), bg)
        if abs(w / h - W / H) <= 0.01:          # 比がほぼ同じ: そのまま拡大・縮小
            fitted, how = im.resize((W, H), Image.LANCZOS), "拡大縮小"
        else:                                    # 比が違う: 比を保って収め、余白で埋める
            k = min(W / w, H / h)
            fitted, how = im.resize((round(w * k), round(h * k)), Image.LANCZOS), "余白"
        canvas.paste(fitted, ((W - fitted.width) // 2, (H - fitted.height) // 2), fitted)
        (out / name).mkdir(parents=True, exist_ok=True)
        canvas.save(out / name / f.name, "PNG", optimize=True)
        print(f"  {name}/{f.name}  {how}")
PY
OUT="$SRC_DIR/AppStore"
```

### ステップ 2 — サイズ・透過・枚数の確認

> **`${n}枚` と波括弧を付ける。** zsh はマルチバイト文字も識別子に使えるので、
> `$n枚` と書くと変数 `n枚` を参照してしまい、枚数が空で出る。

```bash
for dir in "$OUT"/*/; do
  n=$(find "$dir" -name '*.png' | wc -l | tr -d ' ')
  echo "=== $(basename "$dir") (${n}枚) ==="
  sips -g pixelWidth -g pixelHeight -g hasAlpha "$dir"*.png 2>/dev/null \
    | grep -E 'pixel|hasAlpha' | sort | uniq -c
done
```

フォルダ名の寸法と一致し、`hasAlpha: no` だけになっていること。

### ステップ 3 — 目視で確認する（必須）

**サイズが合っていることと、見た目が壊れていないことは別。**
余白になった画像（iPad など）を 1 枚ずつ Read ツールで開き、次を確かめる:

- 円が円のままか（楕円に潰れていないか）
- 文字が横に間延びしていないか
- 左右の余白の色が画面の地と揃っているか

崩れていたら `PAD` の色かステップ 1 の処理を見直す。

---

## 完了レポート

| フォルダ | サイズ | 枚数 | 処理 |
|---------|--------|------|------|
| `AppStore/iPhone_6.3in_1206x2622/` | 1206 × 2622 | N枚 | 拡大縮小か余白（**必須の枠**） |
| `AppStore/iPhone_6.9in_1320x2868/` | 1320 × 2868 | N枚 | 拡大縮小か余白（任意） |
| `AppStore/iPad_13in_2064x2752/` | 2064 × 2752 | N枚 | 余白（**iPad 対応なら必須**） |

App Store Connect の各枠へ、**フォルダ名の寸法とドロップ欄の対応サイズが一致する**
フォルダの画像を上げること。
