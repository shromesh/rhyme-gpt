# rhyme-gpt
GPT for Rhyming in Japanese

## Environment
```
conda create -n {env_name} python=3.10
conda activate {env_name}
conda install conda-forge::openai
conda install conda-forge::pykakasi
conda install conda-forge::python-levenshtein
conda install conda-forge::unidecode
conda install conda-forge::python-dotenv
```

- make .env file
```
OPENAI_API_KEY={YOUR_API_KEY}
```

## 実験
- 仮説：日本語で高度（単語や短文でほぼ全文字の母音が共通するほど）に韻を踏む性能は、few-shot promptingで向上するのではないか。
- 方法：GPT-4oに2種類のプロンプトを与える（以下normal_prompt、better_promptと呼ぶ）。normal_promptはzero-shot prompt、better_promptはR指定さんの韻のうち私が好きなものにより構成されたfew-shot promptに対応する。ここで、input_wordとして事前にChatGPTにより生成された20個の単語に対して、上の2種類のプロンプトによって出力を生成し、その結果を評価した。
- 評価方法：各入力と出力について、それぞれローマ字に直し母音（aiueo）を抽出し、それらのレーベンシュタイン距離を計算し、10-距離をその入出力ペアに対するスコアとする。20個のペアに対してこれを行い、その合計をプロンプトに対するスコアとする。

## 結果
- normal promptのスコアは67、better promptのスコアは94となった。
- 合計点ではbetter promptの圧勝に見えるが、normal promptでは1度outputの形式がおかしくなっているせいで大きく点を失っていたり、better promptより高い点を獲得したものもあったりと、完璧ではなかった。

## 改善案
- outputが""や「」で囲まれる場合があるので、それをpythonで除去
- 文脈を考慮して漢字からひらがなに変換する作業はpykakashiよりGPTの方が得意そう。
    - 一方で、ひらがなをローマ字に変換するのはGPTは苦手。
- 評価を、レーベンシュタイン距離だけでなく次の2つを反映する。
    - 意味が通っているか
    - 元と同じ言葉を含んでいないか

## Appendix

- prompts

```
def get_normal_prompt(input_word):
    return f"""
    あなたはラッパーです。与えられた単語や短文に対してほとんどの母音が韻を踏んでいるような単語や短文を出力してください。ただし、出力は意味の通る単語か短文である必要があり、かつ入力と同じ単語を含んではいけません。
    input: {input_word}
    output:
    """

def get_better_prompt(input_word):
    return f'''
    あなたはラッパーです。与えられた単語や短文に対してほとんどの母音が韻を踏んでいるような単語や短文を出力してください。ただし、出力は意味の通る単語か短文である必要があり、かつ入力と同じ単語を含んではいけません。
    input: オートチューン
    output: 音の宇宙
    input: 皆のユートピア
    output: ミラノ風ドリア
    input: 幼稚園児の作文
    output: 広辞苑十冊分
    input: 押し倒して乗ります
    output: お慕いしております
    input: ノリとテンション
    output: 飲みニケーション
    input: 明日は危なくない
    output: 客はサクラくさい
    input: 小野小町
    output: 男泣き
    input: {input_word}
    output:
    '''
```

- outputs for all inputs

```
input: ペンギンが踊る夢
normal response: センチュリーの虹の夜
better response: 天気が戻る梅雨
score for normal prompt: 4
score for better prompt: 9

input: 空飛ぶたこ焼き
normal response: 高鳴る春の日
better response: ほら吹く神話劇
score for normal prompt: 5
score for better prompt: 5

input: 深夜のカラオケバトル
normal response: ミステリアスなソロライブ
better response: フィンランドのカラメルタルト
score for normal prompt: 4
score for better prompt: 5

input: 猫が会議を仕切る恋のライバルはAI
normal response: 人工天才が舞う愛の戦場で魅せる未来
better response: 明日、会いたいな、恋のカリスマは舞台
score for normal prompt: 0
score for better prompt: 1

input: うさぎがDJする夜
normal response: クラブで音楽鳴る光景
better response: サラミとチーズで乾杯
score for normal prompt: 4
score for better prompt: 2

input: 月面でサッカー観戦
normal response: 月面でサッカー観戦  
こんな奇跡に乾杯の宣言
better response: 律令で合唱団全員
score for normal prompt: 0
score for better prompt: 4

input: サボテンと朝ごはん
normal response: カーテンの長さ指南
better response: 箱根でパラソルさん
score for normal prompt: 7
score for better prompt: 8

input: ドラゴンと契約社員
normal response: ドラマで上役も驚愕达人
better response: 誰かの声援
score for normal prompt: 2
score for better prompt: 3

input: 自転車で宇宙旅行砂漠でカフェ開業
normal response: 森の中で映画撮影嵐の下でコーヒー焙煎
better response: 美術館で遊牧民がパフェ提供
score for normal prompt: 0
score for better prompt: 2

input: カエルがピアノ演奏
normal response: スリルなカードゲームメント  
better response: タヌキが大根満載
score for normal prompt: 4
score for better prompt: 3

input: ゴリラが書いた詩集
normal response: トカゲがささやく技術
better response: 怒りは大河の支流
score for normal prompt: 6
score for better prompt: 7

input: 雨の日の宇宙人
normal response: 風の中の旅人
better response: 亀の甲羅数人前
score for normal prompt: 4
score for better prompt: 5

input: ひよこが占い師
normal response: input: ひよこが占い師  
output: 命も受けない日
better response: 気持ちがゆらゆらし
score for normal prompt: 0
score for better prompt: 7

input: トーストが恋をした
normal response: オーバーな声をした
better response: モーツァルトが恋を知った
score for normal prompt: 7
score for better prompt: 7

input: 海底でヨガレッスン
normal response: タイトルのメッセージスキル
better response: 最低で酔わすセッション
score for normal prompt: 3
score for better prompt: 8

input: 鳥が語る哲学論
normal response: 空を舞う深い約束
better response: 森は騒ぐミステリーロン
score for normal prompt: 4
score for better prompt: 6

input: ネコ耳社長の午後
normal response: テトリス魔女の衝動
better response: 「猫見たくなったのも」
score for normal prompt: 6
score for better prompt: 6

input: 火星で花見大会
normal response: カフェで華麗な舞会
better response: 助成で叶う見栄え
score for normal prompt: 7
score for better prompt: 6
```