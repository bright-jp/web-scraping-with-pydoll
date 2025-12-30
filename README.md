# PydollでのWebスクレイピング

[![Bright Data Promo](https://github.com/bright-jp/LinkedIn-Scraper/raw/main/Proxies%20and%20scrapers%20GitHub%20bonus%20banner.png)](https://brightdata.jp/)

このガイドでは、Pydollを使用してJavaScriptが多用されたWebサイトをスクレイピングし、Cloudflareをバイパスし、Bright Dataのようなローテーティングプロキシでスケールさせる方法を解説します。

- [Pydollの紹介](#an-introduction-to-pydoll)
- [WebスクレイピングでPydollを使う：完全チュートリアル](#using-pydoll-for-web-scraping-complete-tutorial)
- [PydollでCloudflareをバイパスする](#bypassing-cloudflare-with-pydoll)
- [このWebスクレイピング手法の制限](#limitations-of-this-approach-to-web-scraping)
- [PydollとBright Dataのローテーティングプロキシの統合](#integrating-pydoll-with-bright-datas-rotating-proxies)
- [WebスクレイピングにおけるPydollの代替手段](#alternatives-to-pydoll-for-web-scraping)

## Pydollの紹介

このガイドではPydollの基本のみを説明します。機能の詳細や、PythonのWebスクレイピングライブラリとして際立っている点については、[こちらのブログ記事](https://brightdata.jp/blog/web-data/python-web-scraping-libraries)で詳しく学べます。

### これは何ですか

[Pydoll](https://autoscrape-labs.github.io/pydoll/)は、Webスクレイピング、テスト、反復的なタスクの自動化のために構築されたPythonのブラウザ自動化ライブラリです。特徴的なのは、従来のWeb driverが不要である点です。具体的には、DevTools Protocolを介してブラウザに直接接続するため、外部依存関係が不要です。

### 機能

- **Web driver不要**: ブラウザドライバーへの依存がなく、セットアップが容易で、バージョン問題も少なくなります。
- **Async-first**: `asyncio`上に構築された完全非同期で、高い同時接続と効率を実現します。
- **人間らしい操作**: ボット検知を回避するために、現実的なタイピング、マウス移動、クリックを行います。
- **イベント駆動**: ブラウザ、DOM、ネットワーク、ライフサイクルイベントにリアルタイムで反応します。
- **マルチブラウザ対応**: 統一されたAPIでChrome、Edge、その他のChromiumブラウザで動作します。
- **スクリーンショット＆PDF出力**: ページや要素をキャプチャし、高品質なPDFを生成します。
- **Cloudflareのネイティブバイパス**: サードパーティツールなしでCloudflareをバイパスします（良好なIPレピュテーションが前提です）。
- **同時スクレイピング**: 複数ページ/サイトを並列にスクレイピングし、タスクを高速化します。
- **高度なキーボード制御**: キーやタイミングを制御して実際のタイピングをシミュレートします。
- **強力なイベントシステム**: ネットワークとページのイベントを動的に監視・処理します。
- **ファイルアップロード対応**: inputやファイル選択ダイアログ経由でアップロードを自動化します。
- **プロキシ統合**: プロキシを使用してIPをローテーションし、ジオターゲティングします。
- **リクエストのインターセプト**: HTTPリクエスト/レスポンスをインターセプトし、変更またはブロックします。

詳細は[公式ドキュメント](https://autoscrape-labs.github.io/pydoll/features/)をご覧ください。


## WebスクレイピングでPydollを使う：完全チュートリアル

このセクションでは、非同期でJavaScript駆動の「[Quotes to Scrape](https://quotes.toscrape.com/js-delayed/?delay=2000)」からデータを抽出するためにPydollを活用する方法をご紹介します。

![The target site loading the data after 2 seconds](https://github.com/bright-jp/web-scraping-with-pydoll/blob/main/images/The-target-site-loading-the-data-after-2-seconds.gif)

このWebページは、短い遅延の後にJavaScriptを使ってquote要素を動的にレンダリングします。そのため、従来のスクレイピングツールでは正しく動作しません。このページからコンテンツを抽出するには、Pydollのようなブラウザ自動化ソリューションが必要です。

### Step #1: プロジェクトのセットアップ

開始する前に、システムにPython 3+がインストールされていることを確認してください。未インストールの場合は、[ダウンロード](https://www.python.org/downloads/)してインストールガイドに従ってください。

次に、スクレイピングプロジェクト用のディレクトリを作成するために、このコマンドを実行します。

```sh
mkdir pydoll-scraper
```

`pydoll-scraper`フォルダがプロジェクトディレクトリになります。

ターミナルでそのフォルダに移動し、Pythonの[virtual environment](https://docs.python.org/3/library/venv.html)を初期化します。

```sh
cd pydoll-scraper
python -m venv venv
```

任意のPython IDEでプロジェクトフォルダを開きます。[Python拡張機能付きのVisual Studio Code](https://code.visualstudio.com/docs/languages/python)または[PyCharm Community Edition](https://www.jetbrains.com/pycharm/download/#section=windows)がおすすめです。

プロジェクトフォルダに`scraper.py`ファイルを作成します。フォルダ構成は次のようになります。

![The project file structure for web scraping with Pydoll](https://github.com/bright-jp/web-scraping-with-pydoll/blob/main/images/The-project-file-structure-for-web-scraping-with-Pydoll.png)

この時点で`scraper.py`は空のPythonスクリプトですが、まもなく[データパースのロジック](https://brightdata.jp/blog/web-data/what-is-data-parsing)を含むようになります。

次に、IDEのターミナルでvirtual environmentを有効化します。LinuxまたはmacOSでは次を実行します。

```sh
source venv/bin/activate
```

同様にWindowsでは次を実行します。

```sh
venv/Scripts/activate
```

これで、PydollでWebスクレイピングを行うためのPython環境が設定できました。

### Step #2: Pydollのセットアップ

有効化したvirtual environmentで、[`pydoll-python`](https://pypi.org/project/pydoll-python/)パッケージを通じてPydollをインストールします。

```sh
pip install pydoll-python
```

次に、Pydollの利用を開始するために、以下のコードを`scraper.py`ファイルに追加します。

```python
import asyncio
from pydoll.browser.chrome import Chrome

async def main():
    async with Chrome() as browser:
        # Launch the Chrome browser and open a new page
        await browser.start()
        page = await browser.get_page()

        # scraping logic...

# Execute the async scraping function
asyncio.run(main())
```

PydollはWebスクレイピング向けに非同期APIを提供しており、Python標準ライブラリの[`asyncio`](https://docs.python.org/3/library/asyncio.html)を使用する必要がある点に注意してください。

### Step #3: 対象サイトに接続する

`page`オブジェクトから利用できる[`go_to()`](https://autoscrape-labs.github.io/pydoll/deep-dive/page-domain/#navigation-system)メソッドを呼び出して、対象Webサイトにアクセスします。

```python
await page.go_to("https://quotes.toscrape.com/js-delayed/?delay=2000")
```

`?delay=2000`クエリパラメータは、2秒の遅延後に目的のデータを動的に読み込むようページに指示します。これは対象のサンドボックスサイトの機能で、動的スクレイピングの挙動をテストできるよう設計されています。

ここで上記スクリプトを実行してみてください。すべてが正しく動作していれば、Pydollは次を行います。

1.  Chromeインスタンスを起動する
2.  対象サイトに移動する
3.  まだ追加ロジックがないため、ブラウザウィンドウをすぐに閉じる

閉じる直前に、次のような画面が一瞬表示されるはずです。

![The target page being loaded by the Chrome instance controlled by Pydoll](https://github.com/bright-jp/web-scraping-with-pydoll/blob/main/images/The-target-page-being-loaded-by-the-Chrome-instance-controlled-by-Pydoll.png)

### Step #4: HTML要素が表示されるのを待つ

前のステップの最後の画像を確認してください。Pydollが制御しているChromeインスタンス内のページ内容が表示されています。データがまだ読み込まれていないため、完全に空であることに気づくはずです。

これは対象サイトが2秒の遅延後に動的にデータをレンダリングするために発生します。この遅延は例のサイトに固有ですが、ページ要素がレンダリングされるのを待つことは、[SPA（single-page applications）のスクレイピング](https://hackernoon.com/how-to-scrape-modern-spas-pwas-and-ai-driven-dynamic-sites)や、AJAXに依存する他の動的Webサイトをスクレイピングする際によく必要になります。

詳細は、[Pythonで動的Webサイトをスクレイピングする](https://brightdata.jp/blog/how-tos/scrape-dynamic-websites-python)記事をご覧ください。

この一般的なシナリオに対応するため、Pydollは次のメソッドを介して[組み込みの待機メカニズム](https://autoscrape-labs.github.io/pydoll/deep-dive/find-elements-mixin/#waiting-mechanisms)を提供します。

- `wait_element()`: 単一要素の出現を待ちます（タイムアウト対応）

このメソッドはCSSセレクタ、XPath式などをサポートしており、[Seleniumの`By`オブジェクトの動作](https://brightdata.jp/blog/how-tos/using-selenium-for-web-scraping)に似ています。

対象ページのHTML構造を確認しましょう。ブラウザで開いてquotesが読み込まれるのを待ち、いずれかのquoteを右クリックして「Inspect」を選択します。

![The HTML of the quote elements](https://github.com/bright-jp/web-scraping-with-pydoll/blob/main/images/The-HTML-of-the-quote-elements.png)

DevToolsパネルでは、各quoteが`quote`クラスを持つ`<div>`にラップされていることが分かります。つまり、次のCSSセレクタで対象にできます。

```python
.quote
```

次に、処理を進める前にこれらの要素が表示されるまで待つようにPydollを設定します。

```python
await page.wait_element(By.CSS_SELECTOR, ".quote", timeout=3)
```

`By`のimportも忘れないでください。

```python
from pydoll.constants import By
```

スクリプトを再度実行すると、今回はPydollがquote要素の読み込みを待ってからブラウザを閉じることが分かります。

### Step #5: Webスクレイピングの準備

対象ページには複数のquoteが含まれていることを思い出してください。すべてを抽出したいので、この情報を保存するデータ構造が必要です。シンプルな配列で十分なので、初期化します。

```python
quotes = []
```

Pydollは[ページ上の要素を特定する](https://autoscrape-labs.github.io/pydoll/deep-dive/find-elements-mixin/#findelementsmixin-architecture)ために、便利な2つのメソッドを提供します。

- `find_element()`: 最初に一致する要素を特定します
- `find_elements()`: 一致するすべての要素を特定します

`wait_element()`と同様に、これらのメソッドは`By`オブジェクトを使ってセレクタを受け取ります。

したがって、ページ上のすべてのquote要素を次で選択します。

```python
quote_elements = await page.find_elements(By.CSS_SELECTOR, ".quote")
```

次に、要素を反復し、スクレイピングロジックを適用する準備をします。

```python
for quote_element in quote_elements:
  # Scraping logic...
```

### Step #6: データパースのロジックを実装する

まず、単一のquote要素を確認します。

![The HTML of a quote element](https://github.com/bright-jp/web-scraping-with-pydoll/blob/main/images/The-HTML-of-a-quote-element.png)

上のHTMLから分かるとおり、各quote要素には次が含まれます。

- `.text`ノード内のquoteテキスト
- `.author`要素内の著者
- `.tag`要素内のタグ一覧

これらの要素を選択し、関連データを抽出するスクレイピングロジックを実装します。

```python
# Extract the quote text (and remove curly quotes)
text_element = await quote_element.find_element(By.CSS_SELECTOR, ".text")
text = (await text_element.get_element_text()).replace(""", "").replace(""", "")

# Extract the author name
author_element = await quote_element.find_element(By.CSS_SELECTOR, ".author")
author = await author_element.get_element_text()

# Extract all associated tags
tag_elements = await quote_element.find_elements(By.CSS_SELECTOR, ".tag")
tags = [await tag_element.get_element_text() for tag_element in tag_elements]
```

**Note**: [`replace()`](https://docs.python.org/3/library/stdtypes.html#str.replace)メソッドは、抽出したquoteテキストから不要な波括弧付きの二重引用符を削除します。

次に、スクレイピングしたデータを使って新しい辞書オブジェクトを作成し、`quotes`配列に追加します。

```python
# Populate a new quote with the scraped data
quote = {
    "text": text,
    "author": author,
    "tags": tags
}
# Append the extracted quote to the list
quotes.append(quote)
```

### Step #7: CSVにエクスポートする

現在、スクレイピングしたデータはPythonのリストに格納されています。CSVのような人が読みやすい形式にエクスポートして、共有と分析を容易にします。

Pythonを使って`quotes.csv`という新しいファイルを生成し、抽出したデータを埋め込みます。

```python
with open("quotes.csv", "w", newline="", encoding="utf-8") as csvfile:
            # Add the header
            fieldnames = ["text", "author", "tags"]
            writer = csv.DictWriter(csvfile, fieldnames=fieldnames)

            # Populate the output file with the scraped data
            writer.writeheader()
            for quote in quotes:
                writer.writerow(quote)
```

Python標準ライブラリから[`csv`](https://docs.python.org/3/library/csv.html)をimportすることを忘れないでください。

```python
import csv
```

### Step #8: まとめて完成させる

完全な`scraper.py`ファイルは次のようになります。

```python
import asyncio
from pydoll.browser.chrome import Chrome
from pydoll.constants import By
import csv

async def main():
    async with Chrome() as browser:
        # Launch the Chrome browser and open a new page
        await browser.start()
        page = await browser.get_page()

        # Navigate to the target page
        await page.go_to("https://quotes.toscrape.com/js-delayed/?delay=2000")

        # Wait up to 3 seconds for the quote elements to appear
        await page.wait_element(By.CSS_SELECTOR, ".quote", timeout=3)

        # Where to store the scraped data
        quotes = []

        # Select all quote elements
        quote_elements = await page.find_elements(By.CSS_SELECTOR, ".quote")

        # Iterate over them and scrape data from them
        for quote_element in quote_elements:
            # Extract the quote text (and remove curly quotes)
            text_element = await quote_element.find_element(By.CSS_SELECTOR, ".text")
            text = (await text_element.get_element_text()).replace(""", "").replace(""", "")

            # Extract the author
            author_element = await quote_element.find_element(By.CSS_SELECTOR, ".author")
            author = await author_element.get_element_text()

            # Extract all tags
            tag_elements = await quote_element.find_elements(By.CSS_SELECTOR, ".tag")
            tags = [await tag_element.get_element_text() for tag_element in tag_elements]

            # Populate a new quote with the scraped data
            quote = {
                "text": text,
                "author": author,
                "tags": tags
            }
            # Append the extracted quote to the list
            quotes.append(quote)

    # Export the scraped data to CSV
    with open("quotes.csv", "w", newline="", encoding="utf-8") as csvfile:
                # Add the header
                fieldnames = ["text", "author", "tags"]
                writer = csv.DictWriter(csvfile, fieldnames=fieldnames)

                # Populate the output file with the scraped data
                writer.writeheader()
                for quote in quotes:
                    writer.writerow(quote)

# Execute the async scraping function
asyncio.run(main())
```

次を実行してスクリプトをテストします。

```sh
python scraper.py
```

完了すると、プロジェクトフォルダ内に`quotes.csv`ファイルが生成されます。

![The output data in quotes.csv](https://github.com/bright-jp/web-scraping-with-pydoll/blob/main/images/The-output-data-in-the-quotes-csv.png)

## PydollでCloudflareをバイパスする

ブラウザ自動化ツールを通じてWebサイトとやり取りする際に直面する[主要な課題](https://brightdata.jp/blog/web-data/web-scraping-challenges)の1つは、Cloudflareのような[Web application firewalls（WAFs）](https://www.cloudflare.com/learning/ddos/glossary/web-application-firewall-waf/)です。

リクエストが自動化ブラウザからのものだと識別されると、これらのシステムはしばしばCAPTCHAを表示します。場合によっては、サイトへの初回アクセス時にすべての訪問者へ提示します。

[PythonでCAPTCHAをバイパスする](https://brightdata.jp/blog/web-data/bypass-captchas-with-python)のは難しいです。しかし、Cloudflareに正当なユーザーだと納得させ、最初からCAPTCHAチャレンジが表示されないようにする手法は存在します。Pydollはこの目的のために専用APIを提供することで対応しています。

この機能をデモするために、ScrapingCourseサイトの「[Antibot Challenge](https://www.scrapingcourse.com/antibot-challenge)」テストページを使用します。

![Automatic Cloudflare verification on the target page](https://github.com/bright-jp/web-scraping-with-pydoll/blob/main/images/Automatic-Cloudflare-verification-on-the-target-page.gif)

表示されているとおり、このページは一貫して[Cloudflare JavaScript Challenge](https://hackernoon.com/bypassing-javascript-challenges-for-effective-web-scraping)を実行します。バイパス後、アンチボット保護が突破されたことを確認できるサンプルコンテンツが表示されます。

PydollはCloudflareに対応するために2つのアプローチを提供します。

1. [コンテキストマネージャアプローチ](https://autoscrape-labs.github.io/pydoll/features/#context-manager-approach-synchronous): アンチボットチャレンジを同期的に処理し、解決するまでスクリプト実行を一時停止します。
2. [バックグラウンド処理アプローチ](https://autoscrape-labs.github.io/pydoll/features/#background-processing-approach): アンチボットをバックグラウンドで非同期に処理します。

両方の方法を取り上げます。ただし公式ドキュメントで述べられているとおり、Cloudflareのバイパスは保証されません。IPレピュテーションや閲覧履歴などの要因が成功に影響します。

より高度な手法については、[Cloudflare保護サイトのスクレイピング](https://brightdata.jp/blog/web-data/bypass-cloudflare-for-web-scraping)に関する包括的チュートリアルをご覧ください。

### コンテキストマネージャアプローチ

PydollにCloudflareのアンチボットチャレンジを自動的に処理させるには、`expect_and_bypass_cloudflare_captcha()`メソッドを次のように使用します。

```python
import asyncio
from pydoll.browser.chrome import Chrome
from pydoll.constants import By

async def main():
    async with Chrome() as browser:
        # Launch the Chrome browser and open a new page
        await browser.start()
        page = await browser.get_page()

        # Wait for the Cloudflare challenge to be executed
        async with page.expect_and_bypass_cloudflare_captcha():
            # Connect to the Cloudflare-protected page:
            await page.go_to("https://www.scrapingcourse.com/antibot-challenge")
            print("Waiting for Cloudflare anti-bot to be handled...")

        # This code runs only after the anti-bot is successfully bypassed
        print("Cloudflare anti-bot bypassed! Continuing with automation...")

        # Print the text message on the success page
        await page.wait_element(By.CSS_SELECTOR, "#challenge-title", timeout=3)
        success_element = await page.find_element(By.CSS_SELECTOR, "#challenge-title")
        success_text = await success_element.get_element_text()
        print(success_text)

asyncio.run(main())
```

このスクリプトを実行すると、Chromeウィンドウが自動的にチャレンジを突破して対象ページを読み込みます。

出力は次のとおりです。

```
Waiting for Cloudflare anti-bot to be handled...
Cloudflare anti-bot bypassed! Continuing with automation...
You bypassed the Antibot challenge! :D
```

### バックグラウンド処理アプローチ

PydollがCloudflareチャレンジに対処している間にスクリプト実行を停止したくない場合は、`enable_auto_solve_cloudflare_captcha()`および`disable_auto_solve_cloudflare_captcha()`メソッドを次のように使用できます。

```python
import asyncio
from pydoll.browser import Chrome
from pydoll.constants import By

async def main():
    async with Chrome() as browser:
        # Launch the Chrome browser and open a new page
        await browser.start()
        page = await browser.get_page()

        # Enable automatic captcha solving before navigating
        await page.enable_auto_solve_cloudflare_captcha()

        # Connect to the Cloudflare-protected page:
        await page.go_to("https://www.scrapingcourse.com/antibot-challenge")
        print("Page loaded, Cloudflare anti-bot will be handled in the background...")

        # Disable anti-bot auto-solving when no longer needed
        await page.disable_auto_solve_cloudflare_captcha()

        # Print the text message on the success page
        await page.wait_element(By.CSS_SELECTOR, "#challenge-title", timeout=3)
        success_element = await page.find_element(By.CSS_SELECTOR, "#challenge-title")
        success_text = await success_element.get_element_text()
        print(success_text)

asyncio.run(main())
```

このアプローチにより、PydollがバックグラウンドでCloudflareアンチボットチャレンジを解決している間、スクレイパーは他の操作を実行できます。

この場合の出力は次のとおりです。

```
Page loaded, Cloudflare anti-bot will be handled in the background...
You bypassed the Antibot challenge! :D
```

## このWebスクレイピング手法の制限

Pydoll（または任意の[スクレイピングツール](https://brightdata.jp/blog/web-data/best-web-scraping-tools)）でリクエストを送りすぎると、対象サーバーからブロックされる可能性が高いです。これは、多くのWebサイトが、過剰なリクエストによってサーバーが圧迫されるのを防ぐためにレート制限を実装しているためです（あなたのスクレイピングスクリプトのようなボットを防ぐ目的です）。

これは[標準的なアンチスクレイピング](https://brightdata.jp/blog/web-data/anti-scraping-techniques)およびanti-DDoS戦略です。Webサイト運営者が自動化トラフィックの洪水からサイトを守りたいのは当然です。

[`robots.txt`を尊重する](https://brightdata.jp/blog/how-tos/robots-txt-for-web-scraping-guide)などのベストプラクティスに従っていても、単一のIPアドレスから多数のリクエストを送ると疑わしい挙動として検知されることがあります。その結果、[`403 Forbidden`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/403)や[`429 Too Many Requests`](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Status/429)エラーに遭遇する可能性があります。

最も効果的な解決策は、Webプロキシを使ってIPアドレスをローテーションすることです。

この概念に馴染みがない方のために説明すると、[Webプロキシ](https://brightdata.jp/blog/proxy-101/what-is-proxy-server)は、スクレイパーと対象Webサイトの間に入る仲介役として機能します。リクエストを転送してレスポンスを返すため、対象サイトからはトラフィックがあなたの実デバイスではなくプロキシから発生しているように見えます。

この手法は実IPを隠すだけでなく、ジオ制限の回避や[その他多くの用途](https://brightdata.jp/use-cases)にも役立ちます。

[プロキシの種類](https://brightdata.jp/blog/proxy-101/ultimate-guide-to-proxy-types)にはさまざまなものがあります。ブロックを避けるには、Bright Dataのように本物のローテーティングプロキシを提供するプレミアムプロバイダが必要です。

次のセクションでは、より効果的なWebスクレイピング（特にスケール時）を実現するために、Pydollと[Bright Dataのローテーティングプロキシ](https://brightdata.jp/solutions/rotating-proxies)を組み合わせる方法を学びます。

## PydollとBright Dataのローテーティングプロキシの統合

PydollでBright Dataのレジデンシャルプロキシを実装しましょう。

まだアカウントをお持ちでない場合は、[Bright Dataに登録](https://brightdata.jp/cp/start)してください。すでにある場合は、サインインしてダッシュボードにアクセスします。

![The Bright Data dashboard](https://github.com/bright-jp/web-scraping-with-pydoll/blob/main/images/The-Bright-Data-dashboard-1.png)

ダッシュボードから「Get proxy products」ボタンを選択します。

![Get proxy products](https://github.com/bright-jp/web-scraping-with-pydoll/blob/main/images/Clicking-the-Get-proxy-products-button.png)

「Proxies & Scraping Infrastructure」ページに移動します。

![The "Proxies & Scraping Infrastructure" page](https://github.com/bright-jp/web-scraping-with-pydoll/blob/main/images/The-Proxies-Scraping-Infrastructure-page-1.png)

表の中で「Residential」行を見つけてクリックします。

![Clicking the "residential" row](https://github.com/bright-jp/web-scraping-with-pydoll/blob/main/images/Clicking-the-residential-row.png)

レジデンシャルプロキシの設定ページに移動します。

![The "residential" page](https://github.com/bright-jp/web-scraping-with-pydoll/blob/main/images/The-residential-page.png)

初回利用者は、セットアップウィザードに従って要件に応じてプロキシを設定してください。

「Overview」タブに移動し、プロキシのhost、port、username、passwordを確認します。

![The proxy credentials](https://github.com/bright-jp/web-scraping-with-pydoll/blob/main/images/The-proxy-credentials.png)

それらの情報を使ってプロキシURLを構築します。

```
proxy_url = "<brightdata_proxy_username>:<brightdata_proxy_password>@<brightdata_proxy_host>:<brightdata_proxy_port>";
```

プレースホルダー（`<brightdata_proxy_username>`, `<brightdata_proxy_password>`, `<brightdata_proxy_host>`, `<brightdata_proxy_port>`）は実際のプロキシ認証情報に置き換えてください。

トグルを「Off」から「On」に切り替えて、プロキシプロダクトを有効化してください。

![Clicking the activation toggle](https://github.com/bright-jp/web-scraping-with-pydoll/blob/main/images/Clicking-the-activation-toggle.png)

プロキシの設定ができたら、Pydollの[組み込みプロキシ設定機能](https://autoscrape-labs.github.io/pydoll/features/#proxy-integration)を使って統合する方法は次のとおりです。

```python
import asyncio
from pydoll.browser.chrome import Chrome
from pydoll.browser.options import Options
from pydoll.constants import By
import traceback

async def main():
    # Create browser options
    options = Options()

    # The URL of your Bright Data proxy
    proxy_url = "<brightdata_proxy_username>:<brightdata_proxy_password>@<brightdata_proxy_host>:<brightdata_proxy_port>" # Replace it with your proxy URL

    # Configure the proxy integration option
    options.add_argument(f"--proxy-server={proxy_url}")

    # To avoid potential SSL errors
    options.add_argument("--ignore-certificate-errors")

    # Start browser with proxy configuration
    async with Chrome(options=options) as browser:
        await browser.start()
        page = await browser.get_page()

        # Visit a special page that returns the IP of the caller
        await page.go_to("https://httpbin.io/ip")

        # Extract the page content containing only the IP of the incoming
        # request and print it
        body_element = await page.find_element(By.CSS_SELECTOR, "body")
        body_text = await body_element.get_element_text()
        print(f"Current IP address: {body_text}")

# Execute the async scraping function
asyncio.run(main())
```

このスクリプトを実行するたびに、Bright Dataのプロキシローテーションにより、異なる出口IPアドレスが表示されるのを確認できます。

> **Note**:
> 
> 通常、Chromeの`--proxy-server`フラグは認証付きプロキシを直接サポートしません。しかし、[Pydollの高度なプロキシマネージャ](https://autoscrape-labs.github.io/pydoll/deep-dive/browser-domain/#proxy-manager)によりこの制限が解消され、パスワード保護されたプロキシサーバーを使用できます。

## WebスクレイピングにおけるPydollの代替手段

Pydollは確かに強力なWebスクレイピングライブラリであり、特にアンチボット回避機能が組み込まれたブラウザ自動化において有用ですが、他にも有益なツールは存在します。

検討する価値があるPydollの代替手段をいくつかご紹介します。

- [**SeleniumBase**](https://brightdata.jp/blog/web-data/web-scraping-with-seleniumbase): Selenium/WebDriver API上に構築されたPythonフレームワークで、Web自動化のためのプロフェッショナル向けツールキットを提供します。E2Eテストから高度なスクレイピングワークフローまで対応します。
- [**Undetected ChromeDriver**](https://brightdata.jp/blog/web-data/web-scraping-with-undetected-chromedriver): Imperva、DataDome、Distil Networksなどの代表的なアンチボットサービスによる検知を回避するよう設計された、ChromeDriverの改変版です。Seleniumを使用する際に、ステルス性の高いスクレイピングに最適です。

## 結論

IPローテーションの仕組みなしでPydollを使用すると、結果が安定しない可能性があります。Webスクレイピングを信頼性が高くスケーラブルにするために、データセンタープロキシ、レジデンシャルプロキシ、ISPプロキシ、モバイルプロキシを含むBright Dataの[プロキシネットワーク](https://brightdata.jp/proxy-types)をお試しください。

アカウントを作成して、今すぐ無料でプロキシのテストを開始しましょう！