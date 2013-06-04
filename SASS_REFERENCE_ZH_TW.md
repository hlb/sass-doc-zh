# Sass （Syntactically Awesome StyleSheets 語法超讚樣式表）

* 目錄
{:toc}

Sass 是讓 CSS 基礎語法更加強大、優雅的擴充版本。它允許你使用 [變數](#variables_), [巢狀規則(nested rules)](#nested_rules)、
[混搭(mixins)](#mixins)、[直接匯入(inline import)](#import) 等眾多功能，並且完全相容 CSS 既有語法。Sass 有助於保持大型樣式表的結構嚴謹，同時也讓人能夠快速上手小型樣式表，特別是在搭配 [Compass 函式庫](http://compass-style.org)使用的時候。

## 特色

* 完全相容 CSS3
* 變數、巢狀、mixins 等語法擴充
* 許多操控色彩與其他數值的{Sass::Script::Functions 好用函式}
* 進階功能，像是提供函式庫使用的[控制指令](#control_directives)
* 符合語法、客制化的輸出
* [Firebug 整合](https://addons.mozilla.org/en-US/firefox/addon/103988)

## 語法

Sass 有兩種語法。
第一種稱為 SCSS (Sassy CSS)，是一個 CSS3 語法的擴充版本，這整份參考資料都使用本語法。
這代表著所有符合 CSS3 語法的樣式表也都是符合語法並且意義相同的 SCSS 文件。
此外，SCSS 瞭解大多數 CSS hacks 以及瀏覽器專屬語法，像是 [IE 的舊版 `filter` 語法](http://msdn.microsoft.com/en-us/library/ms533754%28VS.85%29.aspx)。
這種語法提供的 Sass 強化特色如下面所述。
使用本語法的文件，副檔名是 `.scss` 結尾。

第二種比較舊的語法稱為縮排語法（indented syntax，或是就稱作 "Sass"），提供一種更簡潔的 CSS 撰寫方式。
它不使用括號，透過縮排來表示選擇符(selectors)的巢狀階層，也捨棄分號，用換行來隔開屬性(properties)。
有些人發現，這樣比 SCSS 更易讀好寫。
縮排語法有著所有相同的特色，雖然有些語法上稍有差異。這些差異在{file:INDENTED_SYNTAX.md 縮排語法參考資料}都有描述。
使用本語法的文件，副檔名是 `.sass` 結尾。

任一種語法都可以[匯入](#import)用另一種語法撰寫的文件。
只要使用 `sass-convert` 命令列工具，就可以將文件自動轉換成另一種語法：

    # 把 Sass 轉換成 SCSS
    $ sass-convert style.sass style.scss

    # 把 SCSS 轉換成 Sass
    $ sass-convert style.scss style.sass

## 使用 Sass

Sass 有三種使用方式：
作為命令列工具、獨立的 Ruby 模組，以及 Ruby on Rails 與 Merb 這些支援 Rack 的程式框架(framework)的外掛套件。
所有這些方式的第一步都是安裝 Sass gem：

    gem install sass

如果你使用 Windoes，你大概需要先 [安裝 Ruby](http://rubyinstaller.org/download.html)。

要在命令列執行 Sass，只要輸入

    sass input.scss output.css

你也可以命令 Sass 監看文件，並且在每次 Sass 文件修改後更新 CSS：

    sass --watch input.scss:output.css

如果你的目錄裡有許多 Sass 文件，你也可以命令 Sass 監看整個目錄：

    sass --watch app/sass:public/stylesheets

使用 `sass --help` 列出完整文件。

在 Ruby 程式碼中使用 Sass 非常容易。
安裝完 Sass gem 之後，你可以執行 `require "sass"`，然後像這樣使用 {Sass::Engine}：

    engine = Sass::Engine.new("#main {background-color: #0000ff}", :syntax => :scss)
    engine.render #=> "#main { background-color: #0000ff; }\n"

### Rack/Rails/Merb 外掛套件

要在 Rails 3 之前的版本啟用 Sass，請把這一行加到 `environment.rb`：

    config.gem "sass"

對於 Rails 3，則是把這一行加到 Gemfile：

    gem "sass"

要在 Merb 裡啟用 Sass，請把這一行加到 `config/dependencies.rb`：

    dependency "merb-haml"

要在一個 Rack 應用程式裡啓用 Sass，在 `config.ru` 加上

    require 'sass/plugin/rack'
    use Sass::Plugin::Rack

Sass 樣式表跟視圖(views)的運作方式不同。
它並沒有動態的內容，所以 CSS 只需要在 Sass 文件更新時產生。

默認情況下，`.sass` 跟 `.scss` 文件是放在 public/stylesheets/sass 目錄（這可以用 [`:template_location`](#template_location-option) 選項客制化）。然後在必要時，他們會被編譯成相應的 CSS 文件放到 public/stylesheets。例如，public/stylesheets/sass/main.scss 會被編譯成 public/stylesheets/main.css。

### 快取(Caching)

默認情況下，Sass 會快取編譯過的樣板與 [partials](#partials)。
這會明顯加快大批 Sass 文件的重新編譯速度，並且在 Sass 樣板被切割成分散文件，再透過 [`@import`](#import) 匯入成一個大文件的時候效果最好。

在沒有使用框架的情況，Sass 會把快取樣板放在 `.sass-cache` 目錄。
在 Rails 與 Merb，它們會放在 `tmp/sass-cache`。
這個目錄可以用 [`:cache_location`](#cache_location-option) 選項客制化。

如果你完全不希望 Sass 使用快取，就把 [`:cache`](#cache-option) 選項設為 `false`.

### 選項

在 Rails 的 `environment.rb` 或 Rack 的 `config.ru` 可以透過使用 {Sass::Plugin::Configuration#options Sass::Plugin#options} 雜湊進行設定

    Sass::Plugin.options[:style] = :compact

… 或者在 Merb 的 `init.rb` 使用 `Merb::Plugin.config[:sass]` 雜湊設定

    Merb::Plugin.config[:sass][:style] = :compact

... 或者傳入一個選項雜湊到 {Sass::Engine#initialize}。

所有對應的選項都可以透過設定旗標在 `sass` 和 `scss` 命令列執行。

可用的選項有：

{#style-option} `:style`
: 設定 CSS 輸出風格.
  請參考 [輸出風格](#output_style).

{#syntax-option} `:syntax`
: 輸入檔案的語法， `:sass` 用於縮排式語法
  以及 `:scss` 用於 CSS-extension 式語法。
  這在建構你自己的 {Sass::Engine} 實例時非常有用；
  當你使用 {Sass::Plugin} 他會自動正確設定。
  預設值是 `:sass`。

{#property_syntax-option} `:property_syntax`
: 強制縮排式語法的文件使用一種描述屬性的語法。
  如果沒有使用正確的語法，將會拋出錯誤。
  `:new` - 強制屬性後面使用冒號。
  例如： `color: #0f3`
  或 `width: $main_width`。
  `:old` - 強制屬性前面使用冒號。
  例如： `:color #0f3`
  或 `:width $main_width`。
  在預設情況，兩者都是正確的。
  但這在 SCSS 文件中並不會生效。

{#cache-option} `:cache`
: 編譯過的 Sass 文件是否被快取，能夠提高重新編譯速度。 預設為 true。

{#read_cache-option} `:read_cache`
: 如果被設定而且 `:cache` 沒有設定，
  那麼只會讀取存在的 Sass 快取，
  如果檔案不存在則不寫入。

{#cache_store-option} `:cache_store`
: 如果這是設定為 {Sass::CacheStores::Base} 子類的實例，
  那麼這個快取儲存將會用於儲存與檢索
  快取編譯結果。
  預設是 {Sass::CacheStores::Filesystem} 透過
  使用 [`:cache_location` 選項](#cache_location-option) 初始化。

{#never_update-option} `:never_update`
: 即使樣板檔案被更改，CSS 檔案也不更新。
  設定這個選項為 true 可能會獲得少量的效能提升。
  這個選項總是預設為 false。
  只有使用 Rack, Ruby 在 Rails 上，或 Merb 才有具有意義。

{#always_update-option} `:always_update`
: 每次 controller 被訪問時 CSS 檔案總是會更新，
  而不是只在樣板檔案被修改的時候。
  預設為 false。
  只有使用 Rack, Ruby 在 Rails 上，或 Merb 才有具有意義。

{#always_check-option} `:always_check`
: 每次 controller 被訪問時總是檢查 Sass 樣板檔案，
  而不是只有在伺服器起動時。
  如果 Sass 樣板檔案被更新，
  它將會重新編譯並且覆蓋對應的 CSS 檔案。
  預設是 false 在 production 模式，在其他情況則為 true。
  只有使用 Rack, Ruby 在 Rails 上，或 Merb 才有具有意義。

{#poll-option} `:poll`
: 當設定為 true 時， 總是使用 {Sass::Plugin::Compiler#watch} 作為後端
  而非使用原生檔案系統作為後端。

{#full_exception-option} `:full_exception`
: 不論是否有錯誤在 Sass 原始碼
  都應該讓 Sass 提供詳細的說明
  在產生的 CSS 檔案中。
  如果設定為 true 那麼錯誤也會同時顯示
  行號與原始碼的註解在 CSS 檔案
  以及頁面的最上方（在支援的瀏覽器中）
  另外，一個例外將會在 Ruby 中被呼叫。
  預設為 false 在 production 模式，在其他情況則為 true。
  只有使用 Rack, Ruby 在 Rails 上，或 Merb 才有具有意義。

{#template_location-option} `:template_location`
: 在你的應用程式中 Sass 樣板檔案的根目錄路徑。
  如果是一個雜湊，那麼 `:css_location` 會被忽略，並且這個選項將會指定輸入與輸出的資料夾。
  也可以給予兩個元素的列表，而非雜湊。
  預設是 `css_location + "/sass"`.
  只有使用 Rack, Ruby 在 Rails 上，或 Merb 才有具有意義。
  註記，如果有多個樣板位置被指定，它們都會被放入匯入路徑，讓你可以在他們之間進行匯入。
  **註記，由於可以採用多種可能的格式，
  這個選項應該直接設定而非訪問或修改。
  使用 {Sass::Plugin::Configuration#template_location_array Sass::Plugin#template_location_array},
  {Sass::Plugin::Configuration#add_template_location Sass::Plugin#add_template_location},
  和 {Sass::Plugin::Configuration#remove_template_location Sass::Plugin#remove_template_location} 方法代替**.

{#css_location-option} `:css_location`
: CSS 檔案寫入的路徑。
  當 `:template_location` 是一個雜湊，那麼這個選項將會被忽略。
  預設為 `"./public/stylesheets"`。
  只有使用 Rack, Ruby 在 Rails 上，或 Merb 才有具有意義。

{#cache_location-option} `:cache_location`
: 快取 `sassc` 檔案寫入的路徑。
  預設為 `"./tmp/sass-cache"` 在 Rails 和 Merb 中，
  或者 `"./.sass-cache"` 其他情況。
  如果 [`:cache_store` 選項](#cache_location-option) 被設定，
  這個選項將會被忽略。

{#unix_newlines-option} `:unix_newlines`
: 如果為 true 則使用 Unix-style 換行方式寫入檔案。
  只有在 Windows 時以及 Sass 寫入檔案時有意義
  (當直接使用 {Sass::Plugin} 在 Rack, Rails 和 Merb
  或使用命令列執行)

{#filename-option} `:filename`
: 正在被渲染的檔案名稱。
  這只用於錯誤報告，
  當使用 Rack, Rails 或 Merb 時會被自動設定。

{#line-option} `:line`
: Sass 樣板的第一行行號。
  用於錯誤回報時的行號。
  當 Sass 樣板是嵌入在 Ruby 檔案時非常有用。

{#load_paths-option} `:load_paths`
: 一個陣列、檔案系統的路徑或匯入器（應要搜尋 Sass 樣板）用於指定 [`@import`](#import) 的目標。
  可能是字串， `Pathname` 物件，或是 {Sass::Importers::Base} 的子類。
  預設是目前工作的資料夾，在 Rack, Rails 或 Merb，
  則是 `:template_location`。
  讀取路徑也會尋找 {Sass.load_paths}
  和 `SASS_PATH` 環境變數設定的路徑。

{#filesystem_importer-option} `:filesystem_importer`
: 一個 {Sass::Importers::Base} 子類，用於處理純文字的讀取路徑。
  這必須從檔案系統匯入檔案。
  它必須是個類別物件繼承自 {Sass::Importers::Base}
  並有一個建構子接受一個字串參數 (讀取路徑).
  預設為 {Sass::Importers::Filesystem}.

{#line_numbers-option} `:line_numbers`
: 當設定為 true 時會將生成該選擇符定義的行號及檔案在 CSS 編譯時輸出在註解。對除錯非常有用，尤其是使用 import 和 mixin 時。
  這個選項也被叫做 `:line_comments`.
  當使用 `:compressed` 輸出風格
  或 `:debug_info`/`:trace_selectors` 選項時會自動停用。

{#trace_selectors-option} `:trace_selectors`
: 當設定為 true 時顯示完整的 import 和 mixin 追蹤在選擇符前。 這可以幫助在瀏覽器中樣式表的 imports 和 mixin 除錯。
  這個選項可以用 `:line_comments` 選項取代而且被
  `:debug_info` 選項取代。 當使用
  `:compressed` 輸出風格時，將會被自動停用。

{#debug_info-option} `:debug_info`
: 當設定為 true 時會將生成該選擇符的行號及檔案在 CSS 編譯為瀏覽器可以理解的格式。
  與 [the FireSass Firebug extension](https://addons.mozilla.org/en-US/firefox/addon/103988) 搭配使用時非常有用於
  呈現 Sass 的檔案及行號。
  當使用 `:compressed` 輸出風格，將會被自動停用。

{#custom-option} `:custom`
: 一個選項可以讓個人應用產生資料給 {Sass::Script::Functions custom Sass functions}.

{#quiet-option} `:quiet`
: 當設定為 true 時，將會導致警告被停用。

### 語法選擇

Sass 命令列工具會根據文件副檔名判斷你使用的語法，但有時未必有檔名這回事。`sass` 命令列程式預設採用縮排語法，但是如果你需要輸入的資料用 SCSS 語法解析，可以傳入 `--scss` 選項。
另外，你可以使用 `scss` 命令列程式，它跟 `sass` 程式完全一模一樣，不過預設採用的語法是 SCSS。

### 編碼

When running on Ruby 1.9 and later, Sass is aware of the character encoding of documents.
By default, Sass assumes that all stylesheets are encoded
using whatever coding system your operating system defaults to.
For many users this will be `UTF-8`, the de facto standard for the web.
For some users, though, it may be a more local encoding.

If you want to use a different encoding for your stylesheet
than your operating system default,
you can use the `@charset` declaration just like in CSS.
Add `@charset "encoding-name";` at the beginning of the stylesheet
(before any whitespace or comments)
and Sass will interpret it as the given encoding.
Note that whatever encoding you use, it must be convertible to Unicode.

Sass will also respect any Unicode BOMs and non-ASCII-compatible Unicode encodings
[as specified by the CSS spec](http://www.w3.org/TR/CSS2/syndata.html#charset),
although this is *not* the recommended way
to specify the character set for a document.
Note that Sass does not support the obscure `UTF-32-2143`,
`UTF-32-3412`, `EBCDIC`, `IBM1026`, and `GSM 03.38` encodings,
since Ruby does not have support for them
and they're highly unlikely to ever be used in practice.

#### Output Encoding

In general, Sass will try to encode the output stylesheet
using the same encoding as the input stylesheet.
In order for it to do this, though, the input stylesheet must have a `@charset` declaration;
otherwise, Sass will default to encoding the output stylesheet as UTF-8.
In addition, it will add a `@charset` declaration to the output
if it's not plain ASCII.

When other stylesheets with `@charset` declarations are `@import`ed,
Sass will convert them to the same encoding as the main stylesheet.

Note that Ruby 1.8 does not have good support for character encodings,
and so Sass behaves somewhat differently when running under it than under Ruby 1.9 and later.
In Ruby 1.8, Sass simply uses the first `@charset` declaration in the stylesheet
or any of the other stylesheets it `@import`s.

## CSS 擴充版本

### 巢狀規則 Nested Rules

Sass 允許 CSS 規則被嵌套在其他規則裡。
內層規則只適用在外層規則的選擇符上。
例如：

    #main p {
      color: #00ff00;
      width: 97%;

      .redbox {
        background-color: #ff0000;
        color: #000000;
      }
    }

被編譯成：

    #main p {
      color: #00ff00;
      width: 97%; }
      #main p .redbox {
        background-color: #ff0000;
        color: #000000; }

這有助於避免重複的父選擇符，
並且讓有許多巢狀選擇符的複雜 CSS 佈局變得更簡單。
例如：

    #main {
      width: 97%;

      p, div {
        font-size: 2em;
        a { font-weight: bold; }
      }

      pre { font-size: 3em; }
    }

被編譯成：

    #main {
      width: 97%; }
      #main p, #main div {
        font-size: 2em; }
        #main p a, #main div a {
          font-weight: bold; }
      #main pre {
        font-size: 3em; }

### 參考父選擇符：`&`

有時候提供其他途徑來使用巢狀語法的父選擇符是非常有幫助的。
例如，你可能會希望在選擇符被滑鼠停留、或是當 body 元件有特定 class 的時候，給予選擇符特定的樣式
在這種情況下，你可以用 `&` 字元來明確指出父選擇符應該放在哪裡。
例如：

    a {
      font-weight: bold;
      text-decoration: none;
      &:hover { text-decoration: underline; }
      body.firefox & { font-weight: normal; }
    }

被編譯成：

    a {
      font-weight: bold;
      text-decoration: none; }
      a:hover {
        text-decoration: underline; }
      body.firefox a {
        font-weight: normal; }

`&` 在編譯成 CSS 時，會被父選擇符置換掉。
這代表說，要是你有一個深層的巢狀規則，父選擇符也會被完整解析，再置換掉 `&`。
例如：

    #main {
      color: black;
      a {
        font-weight: bold;
        &:hover { color: red; }
      }
    }

被編譯成：

    #main {
      color: black; }
      #main a {
        font-weight: bold; }
        #main a:hover {
          color: red; }

### 巢狀屬性

CSS 有相當多屬性都有「命名空間(namespace)」，像是 `font-family`、`font-size`、`font-weight` 都屬於 `font` 命名空間。
在 CSS 裡，如果你要設定屬於相同命名空間的一大堆屬性，你得要每次一個個輸入。
Sass 提供了一個捷徑：只要寫一次命名空間，再用巢狀語法包住每個子屬性。
例如：

    .funky {
      font: {
        family: fantasy;
        size: 30em;
        weight: bold;
      }
    }

被編譯成：

    .funky {
      font-family: fantasy;
      font-size: 30em;
      font-weight: bold; }

命名空間的屬性本身也可以有值。
例如：

    .funky {
      font: 2px/3px {
        family: fantasy;
        size: 30em;
        weight: bold;
      }
    }

被編譯成：

    .funky {
      font: 2px/3px;
        font-family: fantasy;
        font-size: 30em;
        font-weight: bold; }

### 佔位選擇符：`%foo`

Sass 支援一種叫做「佔位選擇符」的特殊選擇符。
它們看起來就像 class 與 id 選擇符，只是 `#` 或 `.` 被 `%` 所取代。
它們必須要配合 [`@extend` 指令](#extend) 使用。
更多資訊請見 [`@extend` 獨享的選擇符](#placeholders).

佔位選擇符如果沒有被 `@extend` 用到，它們裡面的規則將不會輸出成 CSS。

## 註解：`/* */` 與 `//` {#comments}

Sass 支援使用 `/* */` 的標準多行 CSS 註解，以及使用 `//` 的單行註解。
多行註解會在 CSS 輸出裡盡可能地保留，單行注解則會被移除。
例如：

    /* This comment is
     * several lines long.
     * since it uses the CSS comment syntax,
     * it will appear in the CSS output. */
    body { color: black; }

    // These comments are only one line long each.
    // They won't appear in the CSS output,
    // since they use the single-line comment syntax.
    a { color: green; }

被編譯成：

    /* This comment is
     * several lines long.
     * since it uses the CSS comment syntax,
     * it will appear in the CSS output. */
    body {
      color: black; }

    a {
      color: green; }

當註解的第一個字元是 `!` 的時候，即使是在壓縮輸出模式(compressed output mode)，這段註解也會被添加到 CSS 輸出結果裡。
當你需要在生成的 CSS 裡添加版權聲明時，這個功能十分有用。

## SassScript {#sassscript}

除了一般 CSS 屬性語法之外，Sass 也支援一個名為 SassScript 的小型擴充語法集合。
SassScript 允許屬性使用變數、算術和額外的函式。
SassScript 可以用在任何屬性值。

SasScript 也可用於產生選擇符與屬性名稱，
這在撰寫 [mixins](#mixins) 時很有用。
這是靠 [插補(interpolation)](#interpolation_) 機制完成的。

### 互動式界面

你可以輕易地透過互動式界面來嘗試 SassScript。
要啓動此界面，就在執行 Sass 命令列時使用 `-i` 選項。
當提示符號出現時，輸入任何合法的 SassScript 表示式，就會執行並顯示

當提示出現時，輸入任何合法的 SassScript 表示式，就可以執行它，並且把結果印出來給你：

    $ sass -i
    >> "Hello, Sassy World!"
    "Hello, Sassy World!"
    >> 1px + 1px + 1px
    3px
    >> #777 + #777
    #eeeeee
    >> #777 + #888
    white

### 變數：`$` {#variables_}

SassScript 最簡單的使用方式就是操作變數。
變數是用美元符號開頭，並且設定方式就像 CSS 屬性：

    $width: 5em;

然後你就可以在設定屬性時提到它：

    #main {
      width: $width;
    }

變數只會在定義它們的巢狀選擇符內有效。
如果它們是在巢狀選擇符外被定義，那麼他們在任何地方都有效。

變數曾經使用 `!` 前置字元。這種用法仍然有效，但是已經被廢棄(deprecated)了，並且會印出警告訊息。`$` 是推薦的語法。

變數也曾經使用 `=` 而非 `:` 定義。這種用法也仍然有效，但是已經被廢棄，並且會印出警告訊息。`:` 是推薦的語法。

### 資料型別

SassScript 支援六種主要的資料型別：

* 數字（例如 `1.2`、`13`、`10px`）
* 文字字串，無論有沒有引號（例如 `"foo"`、`'bar'`、`baz`）
* 顏色（例如 `blue`、`#04a3f9`、`rgba(255, 0, 0, 0.5)`）
* 布林值（例如 `true`、`false`）
* 空值（例如 `null`）
* 值列表，用空白或逗號隔開（例如 `1.5em 1em 0 2em`、`Helvetica, Arial, sans-serif`）

SassScript 也支援所有其他 CSS 屬性值，
例如 Unicode 範圍和 `!important` 宣告。
然而，它不會對這類型別做任何操作。
它們只會被當成一般字串看待。

#### 字串 {#sass-script-strings}

CSS 提供兩種類型的字串：那些有引號的，像是 `"Lucida Grande"` 或 `'http://sass-lang.com'`，以及那些沒有引號的，像是 `sans-serif` 或 `bold`。
SassScript 兩種都認得，並且如果一種類型的字串被用在 Sass 文件裡，同類型的字串也會被用在 CSS 結果裡。

不過這裡有個例外：
當使用 [`#{}` interpolation](#interpolation_) 時，字串的引號會被拿掉。
這讓它被用在像是 [mixins](#mixins) 內的選擇符名稱，這類用途時容易一些。

例如：

    @mixin firefox-message($selector) {
      body.firefox #{$selector}:before {
        content: "Hi, Firefox users!";
      }
    }

    @include firefox-message(".header");

被編譯成：

    body.firefox .header:before {
      content: "Hi, Firefox users!"; }

也值得注意的是，當使用[廢棄的 `=` 屬性語法](#sassscript)時，
所有字串都會被拿掉引號，無論他們原本撰寫時有沒有引號。

#### 列表

列表是 Sass 表示像 `margin: 10px 15px 0 0` 或 `font-face: Helvetica, Arial, sans-serif` 這類 CSS 宣告的值的方式。
列表就只是一串的數值，用空白或逗號隔開。
事實上，單獨的數值也被視為列表：它們就是只有一個項目的列表。

列表自己做不了什麼事，但是 [Sass 列表函式](Sass/Script/Functions.html#list-functions)讓他們變得有用。

{Sass::Script::Functions#nth nth 函式} 可以取用列表理的項目，
{Sass::Script::Functions#join join 函式} 可以連接多個列表，
{Sass::Script::Functions#append append 函式} 可以把項目添加到列表裡。
[`@each` 規則](#each-directive) 也可以為列表裡的每個項目增添樣式。

除了包含簡單的值，列表也可以包含其他列表。
例如，`1px 2px, 5px 6px` 是一個有兩個項目的列表，它包含了列表 `1px 2px` 和列表 `5px 6px`。
如果內層與外層的列表有相同的分隔符號，你需要使用括號來明確標示內層列表開始與結束之處。
例如，`(1px 2px) (5px 6px)` 也是一個有兩個項目的列表，它包含了列表 `1px 2px` 和列表 `5px 6px`。
差別在於外層列表是用空白隔開，而前一個是用逗號隔開。

當列表被轉換成一般 CSS 的時候，Sass 不會加上任何括號，因為 CSS 並不認得它們。
這代表說，當 `(1px 2px) (5px 6px)` 與 `1px 2px 5px 6px` 被轉換成 CSS 的時候，看起來會是一樣的。
然而，它們在 Sass 中是不同的：
第一個是包含兩個列表的列表，
第二個是包含四個數字的列表。

列表也可以完全不含任何項目。
這種列表是用 `()` 表示。
他們不會直接輸出成 CSS；
如果你試著執行 `font-family: ()`，Sass 會觸發一個錯誤。
如果一個列表含有空列表或空值，
如同 `1px 2px () 3px` 或 `1px 2px null 3px`，
空列表與空值會在該列表被轉換成 CSS 之前被移除掉。

### 運算

所有資料型別都支援等式運算（`==` 和 `!=`）。
此外，每種資料型別有自己特別支援的運算功能。

#### 數字運算

SassScript 支援數字的標準算術運算（`+`、`-`、`*`、`/`、`%`），
而且會盡可能地自動在單位間換算：

    p {
      width: 1in + 8pt;
    }

被編譯成：

    p {
      width: 1.111in; }

數字也支援關係運算子
（`<`、`>`、`<=`、`>=`），
而所有資料型別都支援等式運算子
（`==` 和 `!=`）。

##### 除法和 `/`
{#division-and-slash}

CSS 允許 `/` 出現在屬性值裡，作為分隔數字的一種方法。
既然 SassScript 是 CSS 屬性語法的擴充版本，
它就必須支援這種方法，同時也允許 `/` 可以用在除法上。
這代表說，如果在 SassScript 裡有兩個數字用 `/` 隔開，預設行為應該是在 CSS 結果裡如實呈現。

然而，有三種狀況下 `/` 會被視為除法。
它們包含了絕大多數狀況下，除法真正被使用到的範例，
這些狀況是：

1. 如果數值，或是它的任何部分，是儲存在一個變數裡。
2. 如果數值被括號包住。
3. 如果數值被用在另一個算數式中的一部分。

例如：

    p {
      font: 10px/8px;             // Plain CSS, no division
      $width: 1000px;
      width: $width/2;            // Uses a variable, does division
      height: (500px/2);          // Uses parentheses, does division
      margin-left: 5px + 8px/2px; // Uses +, does division
    }

被編譯成：

    p {
      font: 10px/8px;
      width: 500px;
      height: 250px;
      margin-left: 9px; }

如果你只是想要在一般 CSS 的 `/` 中使用變數，
你可以用 `#{}` 插入它們。
例如：

    p {
      $font-size: 12px;
      $line-height: 30px;
      font: #{$font-size}/#{$line-height};
    }

被編譯成：

    p {
      font: 12px/30px; }

#### 色彩運算

所有算術運算都支援色彩值，並且分段運作。
這表示紅、綠、藍色會輪流運算。
例如：

    p {
      color: #010203 + #040506;
    }

會計算 `01 + 04 = 05`、`02 + 05 = 07` 以及 `03 + 06 = 09`，
並且被編譯成：

    p {
      color: #050709; }

與其嘗試用色彩算數，{Sass::Script::Functions 色彩函式}更有用，並且能達到相同的效果。

算術運算也能在數字與色彩間分段運作。
例如：

    p {
      color: #010203 * 2;
    }

會計算 `01 * 2 = 02`、`02 * 2 = 04` 以及 `03 * 2 = 06`,
並且被編譯成：

    p {
      color: #020406; }

注意那些有 alpha channel 的顏色（像是那些用 {Sass::Script::Functions#rgba rgba} 或 {Sass::Script::Functions#hsla hsla} 函式建立的）必須要有一樣的 alpha 值，才能執行色彩運算。
色彩運算不會影響 alpha 值。
例如：

    p {
      color: rgba(255, 0, 0, 0.75) + rgba(0, 255, 0, 0.75);
    }

被編譯成：

    p {
      color: rgba(255, 255, 0, 0.75); }

一個顏色的 alpha channel 可以透過 {Sass::Script::Functions#opacify opacify} 與 {Sass::Script::Functions#transparentize transparentize} 函式調整。
例如：

    $translucent-red: rgba(255, 0, 0, 0.5);
    p {
      color: opacify($translucent-red, 0.3);
      background-color: transparentize($translucent-red, 0.25);
    }

被編譯成：

    p {
      color: rgba(255, 0, 0, 0.9);
      background-color: rgba(255, 0, 0, 0.25); }

IE 過濾器(filters) 需要包含 alpha 層的所有顏色，並且得用 #AABBCCDD 這樣嚴格的格式。你可以輕鬆地利用 {Sass::Script::Functions#ie_hex_str ie_hex_str} 函式轉換顏色。
例如：

    $translucent-red: rgba(255, 0, 0, 0.5);
    $green: #00ff00;
    div {
      filter: progid:DXImageTransform.Microsoft.gradient(enabled='false', startColorstr='#{ie-hex-str($green)}', endColorstr='#{ie-hex-str($translucent-red)}');
    }

被編譯成：

    div {
      filter: progid:DXImageTransform.Microsoft.gradient(enabled='false', startColorstr=#FF00FF00, endColorstr=#80FF0000);
    }

#### 字串運算

`+` 可以用來連接字串：

    p {
      cursor: e + -resize;
    }

會被編譯成：

    p {
      cursor: e-resize; }

 要注意的是，如果有引號的字串被添加到沒有引號的字串
（這是指說有引號的字串在 `+` 的左邊），
結果會是一個有引號的字串。
同樣的，如果沒有引號的字串被添加到有引號的字串
（沒有引號的字串在 `+` 的左邊），
結果會是一個沒有引號的字串。
例如：

    p:before {
      content: "Foo " + Bar;
      font-family: sans- + "serif";
    }

被編譯成：

    p:before {
      content: "Foo Bar";
      font-family: sans-serif; }

預設狀況下，如我果兩個值彼此相鄰，他們會用空白串接。

    p {
      margin: 3px + 4px auto;
    }

被編譯成：

    p {
      margin: 7px auto; }

在一段字串裡，#{} 形式的插補功能可以用來在字串裡加上動態值：

    p:before {
      content: "I ate #{5 + 10} pies!";
    }

被編譯成：

    p:before {
      content: "I ate 15 pies!"; }

Null 值在字串插補時會被視為空字串：

    $value: null;
    p:before {
      content: "I ate #{$value} pies!";
    }

被編譯成：

    p:before {
      content: "I ate  pies!"; }

#### 布林運算

SassScript 支援 `and`、`or` 和 `not` 等布林運算子。

#### 列表運算

列表不支援任何特殊運算。
但他們可以用[列表函式](Sass/Script/Functions.html#list-functions)操作。

### 括號

括號可以用來影響運算的順序：

    p {
      width: 1em + (2em * 3);
    }

被編譯成：

    p {
      width: 7em; }

### 函式

SassScript 定義了一些有用的函式，
他們可以像一般 CSS 函式語法一樣被呼叫：

    p {
      color: hsl(0, 100%, 50%);
    }

被編譯成：

    p {
      color: #ff0000; }

#### 關鍵字參數(Keyword Arguments)

Sass 函式也可以使用明確的關鍵字參數來呼叫。
上面的範例也可以這樣寫：

    p {
      color: hsl($hue: 0, $saturation: 100%, $lightness: 50%);
    }

雖然不大簡潔，但是它讓樣式表更容易閱讀。
它也允許函式提供更靈活的介面，
在提供許多參數之餘，不會變得難以呼叫。

具名參數(named arguments)可以以任何順序傳入，而且有預設值的參數可以省略。
由於具名參數是變數名稱，底線(underscores)和破折號(dashes)可以交換使用。

完整的 Sass 函式列表和它們的參數名稱，以及在 Ruby 裡定義你自己的函式的步驟，請見 {Sass::Script::Functions}。

### 插補(Interpolation)：`#{}` {#interpolation_}

你也可以在選擇符與屬性名稱中，透過 #{} 插補語法來使用 SassScript 變數：

    $name: foo;
    $attr: border;
    p.#{$name} {
      #{$attr}-color: blue;
    }

被編譯成：

    p.foo {
      border-color: blue; }

也能夠用 `#{}` 來把 SassScript 放到屬性值裡。
在大多數情況下，這沒有比使用變數來得高明，
但是使用 `#{}` 意味著它附近任何的運算符號都會被視為一般 CSS。
例如：

    p {
      $font-size: 12px;
      $line-height: 30px;
      font: #{$font-size}/#{$line-height};
    }

被編譯成：

    p {
      font: 12px/30px; }

### 變數預設值：`!default`

你可以在變數尚未指定時，透過在值的結尾處添加 `!default` 標記來指定變數。
這代表說，如果該變數已經被指定，
就不會被再次指定，
但是如果它沒有指定數值，就會被給定一個。
例如：

    $content: "First content";
    $content: "Second content?" !default;
    $new_content: "First time reference" !default;

    #main {
      content: $content;
      new-content: $new_content;
    }

被編譯成：

    #main {
      content: "First content";
      new-content: "First time reference"; }

變數的值如果是 `null` 的話，會被 !default 當作沒有指定：

    $content: null;
    $content: "Non-null content" !default;

    #main {
      content: $content;
    }

被編譯成：

    #main {
      content: "Non-null content"; }

## `@` 規則與控制指令 {#directives}

Sass 支援所有 CSS3 `@` 規則，以及一些額外的 Sass 專屬的規則，被稱為「指令(directives)」。
這些規則在 Sass 裡有不同的效用，詳述如下。
也可參考[控制指令(control directives)](#control_directives)與 [mixin 指令(mixin directives)](#mixins).

### `@import` {#import}

Sass 擴充了 CSS 的 `@import` 規則，讓它能夠匯入 SCSS 與 Sass 檔案。
所有匯入的 SCSS 與 Sass 檔案會被彙整起來變成單個 CSS 輸出檔案。

此外，任何匯入檔案中定義的變數或 [mixins](#mixins) 都可以被用在主檔案裡。

Sass 會在當下目錄裡尋找其他 Sass 檔案，如果是 Rack、Rails 或 Merb 程式則是在 Sass 檔案目錄。
額外的搜尋目錄可以使用 [`:load_paths`](#load_paths-option) 選項指定，或是使用命令列的 `--load-path` 選項。

`@import` 用檔名進行匯入。
它預設會尋找 Sass 檔案並直接匯入，
但是在少數狀況下，它會被編譯成 CSS 的 `@import` 規則：

* 如果檔案的副檔名是 `.css`。
* 如果檔名用 `http://` 開頭。
* 如果檔名是一個 `url()。`
* 如果 `@import` 包含任何 media queries。

如果沒上述情況都沒有發生，而且副檔名是 `.scss` 或 `.sass`，
該名稱的 Sass 或 SCSS 檔案就會被匯入。
如果沒有註明副檔名，Sass 將會試著找出是該名稱且副檔名為 `.scss` 或 `.sass` 的檔案，並且匯入它。

例如，

    @import "foo.scss";

或

    @import "foo";

兩者都會匯入 `foo.scss` 檔案，
而

    @import "foo.css";
    @import "foo" screen;
    @import "http://foo.com/bar";
    @import url(foo);

會被編譯成

    @import "foo.css";
    @import "foo" screen;
    @import "http://foo.com/bar";
    @import url(foo);

也可以在一個 `@import` 裡匯入多個檔案。例如：

    @import "rounded-corners", "text-shadow";

會匯入 `rounded-corners` 以及 `text-shadow` 兩個檔案。

匯入語法可以包含 `#{}` 插補，但是有著特定限制。
要根據變數來動態匯入一個 Sass 檔案是不可能的；插補只對 CSS 匯入語法有效。
就此來說，它只能用在 `url()` 語法。
例如：

    $family: unquote("Droid+Sans");
    @import url("http://fonts.googleapis.com/css?family=#{$family}");

會被編譯成

    @import url("http://fonts.googleapis.com/css?family=Droid+Sans");

#### 部分(Partials) {#partials}

如果你有一個 SCSS 或 Sass 檔案想要匯入，但是不希望它被編譯成一個 CSS 檔案，你可以在檔名前面加上底線。
這樣會告訴 Sass 不要把它編譯成一般的 CSS 檔案。
接著你可以照樣匯入這些檔案，不需要加上底線。

例如，你也許有個 `_colors.scss`。
這樣並不會建立 `_color.css`，而且你可以這樣做

    @import "colors";

來匯入 `_colors.scss`。

請注意，你不可以在同個目錄放入 partial 與非 partial 的同名檔案。例如，`_colors.scss` 不行與 `colors.scss` 並存。

#### 巢狀 `@import` {#nested-import}

雖然大部份的時間，只要在文件最上層有 `@import` 就非常有用了，
但是你也可以把它們包在 CSS 規則和 `@media` 規則裡。
就像基層的 `@import` 一樣，這樣會包含被 `@import` 匯入的文件內容。
然而，被匯入的規則會被嵌入到原本 `@import` 的位置。

例如，如果 `example.scss` 包含

    .example {
      color: red;
    }

那麼

    #main {
      @import "example";
    }

會被編譯成

    #main .example {
      color: red;
    }

指令只被允許放在文件的基層，
像是 `@mixin` 或 `@charset` 則不被允許出現在被巢狀 `@import` 的文件裡。

巢狀 `@import` mixins 或控制指令是不被允許的。

### `@media` {#media}

`@media` 指令在 Sass 的運作方式跟一般 CSS 裡一模一樣，只是多了一個額外的功能：他們可以被巢狀放置在 CSS 規則內。
如果一個 `@media` 指令出現在一個 CSS 規則內，它會被移動到樣式表的最上層，並且把途中遇到的所有選擇符都放到裡面。
這樣讓添加 media 相關的樣式變得很簡單，不用重複選擇符或是打斷樣式表的連貫性。
例如：

    .sidebar {
      width: 300px;
      @media screen and (orientation: landscape) {
        width: 500px;
      }
    }

被編譯成：

    .sidebar {
      width: 300px; }
      @media screen and (orientation: landscape) {
        .sidebar {
          width: 500px; } }

`@media` queries 也可以被巢狀放置在另一個裡面。
這個 queries 會用 `and` 運算符與另一個合併。
例如：

    @media screen {
      .sidebar {
        @media (orientation: landscape) {
          width: 500px;
        }
      }
    }

被編譯成：

    @media screen and (orientation: landscape) {
      .sidebar {
        width: 500px; } }

最後，`@media` queries 可以用 SassScript 表達式（包含變數、函式和算符）替換掉特徵名稱(feature names)與特徵值(feature values)。例如：

    $media: screen;
    $feature: -webkit-min-device-pixel-ratio;
    $value: 1.5;

    @media #{$media} and ($feature: $value) {
      .sidebar {
        width: 500px;
      }
    }

被編譯成：

    @media screen and (-webkit-min-device-pixel-ratio: 1.5) {
      .sidebar {
        width: 500px; } }

### `@extend` {#extend}

當設計一個頁面時的常見狀況是，一個 class 應該要具備另一個 class 的所有樣式，並且有它自己的特定樣式。
最常驗的處理方式是在 HTML 裡同時使用一般性的 class 以及更特定的 class。
例如，假設我們設計了一個通常錯誤，以及一個嚴重的錯誤。我們也許會像這樣撰寫我們的 HTML markup：

    <div class="error seriousError">
      Oh no! You've been hacked!
    </div>

以及像這樣的樣式：

    .error {
      border: 1px #f00;
      background-color: #fdd;
    }
    .seriousError {
      border-width: 3px;
    }

不幸的是，這意味著我們必須總是記得將 `.seriousError` 與 `.error` 一起使用。
這是一個維護的負擔，會導致棘手的錯誤，並且帶來非語意化(non-semantic)風格的顧慮。

`@extend` 控制指令避免了這些問題，它會告知 Sass 一個選擇符應該繼承另一個選擇符的樣式。
例如：

    .error {
      border: 1px #f00;
      background-color: #fdd;
    }
    .seriousError {
      @extend .error;
      border-width: 3px;
    }

這表示除了 `.seriousError` 特別指定的樣式之外，定義在 `.error` 的所有樣式也應該被套用到 `.seriousError`。
實際上，任何有 class `.seriousError` 的東西都會有 class `.error` 的效用。
例如，如果我們有特殊的樣式來表示駭客造成的錯誤：

    .error.intrusion {
      background-image: url("/image/hacked.png");
    }

如此一來 `<div class="seriousError intrusion">` 也會有 `hacked.png` 背景圖片。

#### 如何運作

`@extend` 的運作方式是在樣式表裡出現被延伸的選擇符（例如 `.error`）的任何位置插入延伸的選擇符（例如 `.seriousError`）。
所以下面這個範例：

    .error {
      border: 1px #f00;
      background-color: #fdd;
    }
    .error.intrusion {
      background-image: url("/image/hacked.png");
    }
    .seriousError {
      @extend .error;
      border-width: 3px;
    }

被編譯成：

    .error, .seriousError {
      border: 1px #f00;
      background-color: #fdd; }

    .error.intrusion, .seriousError.intrusion {
      background-image: url("/image/hacked.png"); }

    .seriousError {
      border-width: 3px; }

在合併選擇符的時候，`@extend` 有足夠的智慧來避免不必要的重複，
所以像是 `.seriousError.seriousError` 會被翻譯成 `.seriousError`。
此外，他不會產生無法批配任何東西的選擇符，像是 `#main#footer`。

#### 延伸複雜的選擇符

類別選擇符不是唯一可以被延伸的東西。
任何只有單獨元素的選擇符都可以被延伸，像是 `.special.cool`、`a:hover` 或 `a.user[href^="http://"]`。
例如：

    .hoverlink {
      @extend a:hover;
    }

如同類別，這表示定義在 `a:hover` 的所有樣式也會套用到 `.hoverlink`。
例如：

    .hoverlink {
      @extend a:hover;
    }
    a:hover {
      text-decoration: underline;
    }

被編譯成：

    a:hover, .hoverlink {
      text-decoration: underline; }

如同上述的 `.error.intrusion`，
所有用到 `a:hover` 的規則也能對 `.hoverlink` 運作，
即使它們也有其他選擇符。
例如：

    .hoverlink {
      @extend a:hover;
    }
    .comment a.user:hover {
      font-weight: bold;
    }

被編譯成：

    .comment a.user:hover, .comment .user.hoverlink {
      font-weight: bold; }

#### 多重延伸

單一選擇符可以延伸多個選擇符。
這樣代表它繼承了所有被延伸選擇符的樣式。
例如：

    .error {
      border: 1px #f00;
      background-color: #fdd;
    }
    .attention {
      font-size: 3em;
      background-color: #ff0;
    }
    .seriousError {
      @extend .error;
      @extend .attention;
      border-width: 3px;
    }

被編譯成：

    .error, .seriousError {
      border: 1px #f00;
      background-color: #fdd; }

    .attention, .seriousError {
      font-size: 3em;
      background-color: #ff0; }

    .seriousError {
      border-width: 3px; }

實際上，任何有 class `.seriousError` 的東西都會有 class `.error` *與* `.attention` 的效用。
因此，定義在文件比較後面的樣式優先權較高：
既然 `.attention` 在 `.error` 之後被定義，`.seriousError` 的背景色會是 `#ff0` 而非 `#fdd`。

多重延伸也可以寫成用逗號隔開的選擇符列表。
例如，`@extend .error, .attention` 跟 `@extend .error; @extend.attention` 是相同的。

#### 連鎖延伸

選擇符是可以一而再、再而三地不斷延伸的。
例如：

    .error {
      border: 1px #f00;
      background-color: #fdd;
    }
    .seriousError {
      @extend .error;
      border-width: 3px;
    }
    .criticalError {
      @extend .seriousError;
      position: fixed;
      top: 10%;
      bottom: 10%;
      left: 10%;
      right: 10%;
    }

現在任何有 `.seriousError` class 的東西也會有 `.error` class，任何有 `.criticalError` class 的東西也會有 `.seriosError` *以及* `.error` class。
它會被編譯成：

    .error, .seriousError, .criticalError {
      border: 1px #f00;
      background-color: #fdd; }

    .seriousError, .criticalError {
      border-width: 3px; }

    .criticalError {
      position: fixed;
      top: 10%;
      bottom: 10%;
      left: 10%;
      right: 10%; }

#### 選擇符序列 Selector Sequences

選擇符序列，像是 `.foo .bar` 或 `.foo + .bar`，目前是不能被延伸的。
然而，巢狀選擇符自己是可以用 `@extend` 的。
例如：

    #fake-links .link {
      @extend a;
    }

    a {
      color: blue;
      &:hover {
        text-decoration: underline;
      }
    }

被編譯成

    a, #fake-links .link {
      color: blue; }
      a:hover, #fake-links .link:hover {
        text-decoration: underline; }

##### 彙整 Merging Selector Sequences

有時候，一個選擇符序列會延伸在另一個序列中的選擇符。
在這種狀況下，兩個序列需要被彙整起來。
例如：

    #admin .tabbar a {
      font-weight: bold;
    }
    #demo .overview .fakelink {
      @extend a;
    }

雖然產生出可以符合任一序列的所有選擇符，在技術面上是可行的，
但這會讓樣式表過於龐大。
例如上面這個簡單的範例，就會需要 10 個選擇符。
因此，Sass 只會產生出可能有用的選擇符。

當兩個被彙整的序列沒有相同的選擇符時，
兩個新的選擇符會被產生：
一個是第一序列在第二序列之前，
另一個是第二序列在第一序列之前。
例如：

    #admin .tabbar a {
      font-weight: bold;
    }
    #demo .overview .fakelink {
      @extend a;
    }

被編譯成：

    #admin .tabbar a,
    #admin .tabbar #demo .overview .fakelink,
    #demo .overview #admin .tabbar .fakelink {
      font-weight: bold; }

如果兩個序列有共享一些選擇符，
這些選擇符就會被彙整起來，
只有差異之處（如果有的話）會輪替。
在這個例子，兩個序列都包含 id `#admin`，
所以產出的選擇符會整合這兩個 id：

    #admin .tabbar a {
      font-weight: bold;
    }
    #admin .overview .fakelink {
      @extend a;
    }

並被編譯成：

    #admin .tabbar a,
    #admin .tabbar .overview .fakelink,
    #admin .overview .tabbar .fakelink {
      font-weight: bold; }

#### 只用來 `@extend` 的選擇符 {#placeholders}

有時候你撰寫 class 樣式的時候，只希望用來 `@extend`，
而從不想將它直接用在你的 HTML 裡面。
這在撰寫 Sass 函式庫時尤其是如此，
因為你可以提供樣式給用戶，如果他們需要就 `@extend`，
不需要時則可以忽略。

如果你用一般的 class 來完成，當樣式表生成時，你最終會創造出許多額外的 CSS，並且得冒著與 HTML 裡正在使用的其他 class 相撞的風險。
這就是為什麼 Sass 支援「佔位選擇符(placeholder selectors)」（例如 `%foo`）。

佔位選擇符看起來像 class 與 id 選擇符，
只是 `#` 或 `.` 被 `%` 取代。
它們可以被用在任何 class 或 id 能用的地方，
而且會避免自己的規則被輸出到 CSS。

例如：

    // This ruleset won't be rendered on its own.
    #context a%extreme {
      color: blue;
      font-weight: bold;
      font-size: 2em;
    }

然而，佔位選擇符可以被 extend，就像 class 與 id 一樣。
被 extend 的選擇符會被生成，但是基礎的佔位選擇符則不會。
例如：

    .notice {
      @extend %extreme;
    }

被編譯成：

    #context a.notice {
      color: blue;
      font-weight: bold;
      font-size: 2em; }

#### `!optional` 旗標

通常，當你繼承一個選擇符，如果 `@extend` 不起作用，這是一個錯誤。
例如，如果你寫了 `a.important {@extend .notice}`，卻沒有任何選擇符包含 `.notice`，就是一個錯誤。

如果唯一包含 `.notice` 的選擇符是 `h1.notice`，既然 `h1` 與 `a` 衝突，這也會是一個錯誤，而且不會有新的選擇符被產生。

不過，有時候你想要允許一個 `@extend` 不產生任何選擇符。要做到這一點，只需要在行末加上 `!optional` 旗標。
例如：

    a.important {
      @extend .notice !optional;
    }

#### 用於指令中的 `@extend`

`@extend` 在 `@media` 等指令內使用時有一些限制。
Sass 無法將 `@media` 區塊之外的 CSS 規則套用到它裡面的選擇符，因為這不可避免地會到處複製樣式，創造出大量膨脹的樣式表。
這意味著，如果你在 `@media`（或是其他 CSS 指令）內使用 `@extend`，你只能繼承出現相同指令區塊的選擇符。

例如，以下範例會正常運作：

    @media print {
      .error {
        border: 1px #f00;
        background-color: #fdd;
      }
      .seriousError {
        @extend .error;
        border-width: 3px;
      }
    }

但是這樣會導致錯誤：

    .error {
      border: 1px #f00;
      background-color: #fdd;
    }

    @media print {
      .seriousError {
        // INVALID EXTEND: .error is used outside of the "@media print" directive
        @extend .error;
        border-width: 3px;
      }
    }

我們希望有朝一日 `@extend` 能獲得瀏覽器原生支援，這將允許它被用在 `@media` 及其它指令中。

### `@debug`

The `@debug` directive prints the value of a SassScript expression
to the standard error output stream.
It's useful for debugging Sass files
that have complicated SassScript going on.
例如：

    @debug 10em + 12em;

輸出：

    Line 1 DEBUG: 22em

### `@warn`

The `@warn` directive prints the value of a SassScript expression
to the standard error output stream.
It's useful for libraries that need to warn users of deprecations
or recovering from minor mixin usage mistakes.
There are two major distinctions between `@warn` and `@debug`:

1. You can turn warnings off with the `--quiet` command-line option
   or the `:quiet` Sass option.
2. A stylesheet trace will be printed out along with the message
   so that the user being warned can see where their styles caused the warning.

Usage Example:

    @mixin adjust-location($x, $y) {
      @if unitless($x) {
        @warn "Assuming #{$x} to be in pixels";
        $x: 1px * $x;
      }
      @if unitless($y) {
        @warn "Assuming #{$y} to be in pixels";
        $y: 1px * $y;
      }
      position: relative; left: $x; top: $y;
    }

## Control Directives

SassScript supports basic control directives
for including styles only under some conditions
or including the same style several times with variations.

**Note that control directives are an advanced feature,
and are not recommended in the course of day-to-day styling**.
They exist mainly for use in [mixins](#mixins),
particularly those that are part of libraries like [Compass](http://compass-style.org),
and so require substantial flexibility.

### `@if`

The `@if` directive takes a SassScript expression
and uses the styles nested beneath it if the expression returns
anything other than `false` or `null`:

    p {
      @if 1 + 1 == 2 { border: 1px solid;  }
      @if 5 < 3      { border: 2px dotted; }
      @if null       { border: 3px double; }
    }

被編譯成：

    p {
      border: 1px solid; }

The `@if` statement can be followed by several `@else if` statements
and one `@else` statement.
If the `@if` statement fails,
the `@else if` statements are tried in order
until one succeeds or the `@else` is reached.
例如：

    $type: monster;
    p {
      @if $type == ocean {
        color: blue;
      } @else if $type == matador {
        color: red;
      } @else if $type == monster {
        color: green;
      } @else {
        color: black;
      }
    }

被編譯成：

    p {
      color: green; }

### `@for`

The `@for` directive repeatedly outputs a set of styles. For each repetition, a
counter variable is used to adjust the output. The directive has two forms:
`@for $var from <start> through <end>` and `@for $var from <start> to <end>`.
Note the difference in the keywords `through` and `to`. `$var` can be any
variable name, like `$i`; `<start>` and `<end>` are SassScript expressions that
should return integers.

The `@for` statement sets `$var` to each successive number in the specified
range and each time outputs the nested styles using that value of `$var`. For
the form `from ... through`, the range *includes* the values of `<start>` and
`<end>`, but the form `from ... to` runs up to *but not including* the value of
`<end>`. Using the `through` syntax,

    @for $i from 1 through 3 {
      .item-#{$i} { width: 2em * $i; }
    }

被編譯成：

    .item-1 {
      width: 2em; }
    .item-2 {
      width: 4em; }
    .item-3 {
      width: 6em; }

### `@each` {#each-directive}

The `@each` rule has the form `@each $var in <list>`.
`$var` can be any variable name, like `$length` or `$name`,
and `<list>` is a SassScript expression that returns a list.

The `@each` rule sets `$var` to each item in the list,
then outputs the styles it contains using that value of `$var`.
例如：

    @each $animal in puma, sea-slug, egret, salamander {
      .#{$animal}-icon {
        background-image: url('/images/#{$animal}.png');
      }
    }

被編譯成：

    .puma-icon {
      background-image: url('/images/puma.png'); }
    .sea-slug-icon {
      background-image: url('/images/sea-slug.png'); }
    .egret-icon {
      background-image: url('/images/egret.png'); }
    .salamander-icon {
      background-image: url('/images/salamander.png'); }

### `@while`

The `@while` directive takes a SassScript expression
and repeatedly outputs the nested styles
until the statement evaluates to `false`.
This can be used to achieve more complex looping
than the `@for` statement is capable of,
although this is rarely necessary.
例如：

    $i: 6;
    @while $i > 0 {
      .item-#{$i} { width: 2em * $i; }
      $i: $i - 2;
    }

被編譯成：

    .item-6 {
      width: 12em; }

    .item-4 {
      width: 8em; }

    .item-2 {
      width: 4em; }

## Mixin Directives {#mixins}

Mixins allow you to define styles
that can be re-used throughout the stylesheet
without needing to resort to non-semantic classes like `.float-left`.
Mixins can also contain full CSS rules,
and anything else allowed elsewhere in a Sass document.
They can even take [arguments](#mixin-arguments)
which allows you to produce a wide variety of styles
with very few mixins.

### Defining a Mixin: `@mixin` {#defining_a_mixin}

Mixins are defined with the `@mixin` directive.
It's followed by the name of the mixin
and optionally the [arguments](#mixin-arguments),
and a block containing the contents of the mixin.
For example, the `large-text` mixin is defined as follows:

    @mixin large-text {
      font: {
        family: Arial;
        size: 20px;
        weight: bold;
      }
      color: #ff0000;
    }

Mixins may also contain selectors,
possibly mixed with properties.
The selectors can even contain [parent references](#referencing_parent_selectors_).
例如：

    @mixin clearfix {
      display: inline-block;
      &:after {
        content: ".";
        display: block;
        height: 0;
        clear: both;
        visibility: hidden;
      }
      * html & { height: 1px }
    }

### Including a Mixin: `@include` {#including_a_mixin}

Mixins are included in the document
with the `@include` directive.
This takes the name of a mixin
and optionally [arguments to pass to it](#mixin-arguments),
and includes the styles defined by that mixin
into the current rule.
例如：

    .page-title {
      @include large-text;
      padding: 4px;
      margin-top: 10px;
    }

被編譯成：

    .page-title {
      font-family: Arial;
      font-size: 20px;
      font-weight: bold;
      color: #ff0000;
      padding: 4px;
      margin-top: 10px; }

Mixins may also be included outside of any rule
(that is, at the root of the document)
as long as they don't directly define any properties
or use any parent references.
例如：

    @mixin silly-links {
      a {
        color: blue;
        background-color: red;
      }
    }

    @include silly-links;

被編譯成：

    a {
      color: blue;
      background-color: red; }

Mixin definitions can also include other mixins.
例如：

    @mixin compound {
      @include highlighted-background;
      @include header-text;
    }

    @mixin highlighted-background { background-color: #fc0; }
    @mixin header-text { font-size: 20px; }

Mixins that only define descendent selectors, can be safely mixed
into the top most level of a document.

### Arguments {#mixin-arguments}

Mixins can take arguments SassScript values as arguments,
which are given when the mixin is included
and made available within the mixin as variables.

When defining a mixin,
the arguments are written as variable names separated by commas,
all in parentheses after the name.
Then when including the mixin,
values can be passed in in the same manner.
例如：

    @mixin sexy-border($color, $width) {
      border: {
        color: $color;
        width: $width;
        style: dashed;
      }
    }

    p { @include sexy-border(blue, 1in); }

被編譯成：

    p {
      border-color: blue;
      border-width: 1in;
      border-style: dashed; }

Mixins can also specify default values for their arguments
using the normal variable-setting syntax.
Then when the mixin is included,
if it doesn't pass in that argument,
the default value will be used instead.
例如：

    @mixin sexy-border($color, $width: 1in) {
      border: {
        color: $color;
        width: $width;
        style: dashed;
      }
    }
    p { @include sexy-border(blue); }
    h1 { @include sexy-border(blue, 2in); }

被編譯成：

    p {
      border-color: blue;
      border-width: 1in;
      border-style: dashed; }

    h1 {
      border-color: blue;
      border-width: 2in;
      border-style: dashed; }

#### Keyword Arguments

Mixins can also be included using explicit keyword arguments.
For instance, we the above example could be written as:

    p { @include sexy-border($color: blue); }
    h1 { @include sexy-border($color: blue, $width: 2in); }

While this is less concise, it can make the stylesheet easier to read.
It also allows functions to present more flexible interfaces,
providing many arguments without becoming difficult to call.

Named arguments can be passed in any order, and arguments with default values can be omitted.
Since the named arguments are variable names, underscores and dashes can be used interchangeably.

#### Variable Arguments

Sometimes it makes sense for a mixin to take an unknown number of arguments. For
example, a mixin for creating box shadows might take any number of shadows as
arguments. For these situations, Sass supports "variable arguments," which are
arguments at the end of a mixin declaration that take all leftover arguments and
package them up as a [list](#lists). These arguments look just like normal
arguments, but are followed by `...`. 例如：

    @mixin box-shadow($shadows...) {
      -moz-box-shadow: $shadows;
      -webkit-box-shadow: $shadows;
      box-shadow: $shadows;
    }

    .shadows {
      @include box-shadow(0px 4px 5px #666, 2px 6px 10px #999);
    }

被編譯成：

    .shadowed {
      -moz-box-shadow: 0px 4px 5px #666, 2px 6px 10px #999;
      -webkit-box-shadow: 0px 4px 5px #666, 2px 6px 10px #999;
      box-shadow: 0px 4px 5px #666, 2px 6px 10px #999;
    }

Variable arguments can also be used when calling a mixin. Using the same syntax,
you can expand a list of values so that each value is passed as a separate
argument. 例如：

    @mixin colors($text, $background, $border) {
      color: $text;
      background-color: $background;
      border-color: $border;
    }

    $values: #ff0000, #00ff00, #0000ff;
    .primary {
      @include colors($values...);
    }

被編譯成：

    .primary {
      color: #ff0000;
      background-color: #00ff00;
      border-color: #0000ff;
    }

You can use variable arguments to wrap a mixin and add additional styles without
changing the argument signature of the mixin. If you do so, even keyword
arguments will get passed through to the wrapped mixin. 例如：

    @mixin wrapped-stylish-mixin($args...) {
      font-weight: bold;
      @include stylish-mixin($args...);
    }

    .stylish {
      // The $width argument will get passed on to "stylish-mixin" as a keyword
      @include wrapped-stylish-mixin(#00ff00, $width: 100px);
    }

### Passing Content Blocks to a Mixin {#mixin-content}

It is possible to pass a block of styles to the mixin for placement within the styles included by
the mixin. The styles will appear at the location of any `@content` directives found within the mixin. This makes is possible to define abstractions relating to the construction of
selectors and directives.

例如：

    @mixin apply-to-ie6-only {
      * html {
        @content;
      }
    }
    @include apply-to-ie6-only {
      #logo {
        background-image: url(/logo.gif);
      }
    }

Generates:

    * html #logo {
      background-image: url(/logo.gif);
    }

The same mixins can be done in the `.sass` shorthand syntax:

    =apply-to-ie6-only
      * html
        @content

    +apply-to-ie6-only
      #logo
        background-image: url(/logo.gif)

**Note:** when the `@content` directive is specified more than once or in a loop, the style block will be duplicated with each invocation.

#### Variable Scope and Content Blocks

The block of content passed to a mixin are evaluated in the scope where the block is defined,
not in the scope of the mixin. This means that variables local to the mixin **cannot** be used
within the passed style block and variables will resolve to the global value:

    $color: white;
    @mixin colors($color: blue) {
      background-color: $color;
      @content;
      border-color: $color;
    }
    .colors {
      @include colors { color: $color; }
    }

Compiles to:

    .colors {
      background-color: blue;
      color: white;
      border-color: blue;
    }

Additionally, this makes it clear that the variables and mixins that are used within the
passed block are related to the other styles around where the block is defined. 例如：

    #sidebar {
      $sidebar-width: 300px;
      width: $sidebar-width;
      @include smartphone {
        width: $sidebar-width / 3;
      }
    }


## Function Directives {#function_directives}

It is possible to define your own functions in sass and use them in any
value or script context. 例如：

    $grid-width: 40px;
    $gutter-width: 10px;

    @function grid-width($n) {
      @return $n * $grid-width + ($n - 1) * $gutter-width;
    }

    #sidebar { width: grid-width(5); }

Becomes:

    #sidebar {
      width: 240px; }

As you can see functions can access any globally defined variables as well as
accept arguments just like a mixin. A function may have several statements
contained within it, and you must call `@return` to set the return value of
the function.

As with mixins, you can call Sass-defined functions using keyword arguments.
In the above example we could have called the function like this:

    #sidebar { width: grid-width($n: 5); }

It is recommended that you prefix your functions to avoid naming conflicts
and so that readers of your stylesheets know they are not part of Sass or CSS. For example, if you work for ACME Corp, you might have named the function above `-acme-grid-width`.

User-defined functions also support [variable arguments](#variable_arguments)
in the same way as mixins.

## 程式碼匯出樣式 Output Style

雖然 Sass 預設匯出的 CSS 已經非常棒，也反應文件的結構，
但眾人需求不同，所以 Sass 也支援其他不同的程式碼樣式。

只要設定 [`:style` 選項](#style-option) 或採用 `--style` 命令列選項，就能讓 Sass 以不同設定匯出。

### 巢狀 `:nested`

Sass 預設使用巢狀方式匯出 CSS，因為這可反應了 CSS 樣式及所套用之 HTML 文件的結構。
每個屬性獨自佔用一行，但縮排幅度則依據其原本巢狀層級的深淺而定，並不一致。
例如：

    #main {
      color: #fff;
      background-color: #000; }
      #main p {
        width: 10em; }

    .huge {
      font-size: 10em;
      font-weight: bold;
      text-decoration: underline; }

巢狀排列在觀看大型 CSS 檔案時相當有用：你不必認真閱讀程式，便可輕鬆理解整個檔案的結構。

### `:expanded`

Expanded is a more typical human-made CSS style,
with each property and rule taking up one line.
Properties are indented within the rules,
but the rules aren't indented in any special way.
例如：

    #main {
      color: #fff;
      background-color: #000;
    }
    #main p {
      width: 10em;
    }

    .huge {
      font-size: 10em;
      font-weight: bold;
      text-decoration: underline;
    }

### `:compact`

Compact style takes up less space than Nested or Expanded.
It also draws the focus more to the selectors than to their properties.
Each CSS rule takes up only one line,
with every property defined on that line.
Nested rules are placed next to each other with no newline,
while separate groups of rules have newlines between them.
例如：

    #main { color: #fff; background-color: #000; }
    #main p { width: 10em; }

    .huge { font-size: 10em; font-weight: bold; text-decoration: underline; }

### `:compressed`

Compressed style takes up the minimum amount of space possible,
having no whitespace except that necessary to separate selectors
and a newline at the end of the file.
It also includes some other minor compressions,
such as choosing the smallest representation for colors.
It's not meant to be human-readable.
例如：

    #main{color:#fff;background-color:#000}#main p{width:10em}.huge{font-size:10em;font-weight:bold;text-decoration:underline}

## Extending Sass

Sass provides a number of advanced customizations for users with unique requirements.
Using these features requires a strong understanding of Ruby.

### Defining Custom Sass Functions

Users can define their own Sass functions using the Ruby API.
For more information, see the [source documentation](Sass/Script/Functions.html#adding_custom_functions).

### Cache Stores

Sass caches parsed documents so that they can be reused without parsing them again
unless they have changed. By default, Sass will write these cache files to a location
on the filesystem indicated by [`:cache_location`](#cache_location-option). If you
cannot write to the filesystem or need to share cache across ruby processes or machines,
then you can define your own cache store and set the[`:cache_store`
option](#cache_store-option). For details on creating your own cache store, please
see the {Sass::CacheStores::Base source documentation}.

### Custom Importers

Sass importers are in charge of taking paths passed to `@import` and finding the
appropriate Sass code for those paths. By default, this code is loaded from
the {Sass::Importers::Filesystem filesystem}, but importers could be added to load
from a database, over HTTP, or use a different file naming scheme than what Sass expects.

Each importer is in charge of a single load path (or whatever the corresponding notion
is for the backend). Importers can be placed in the {file:SASS_REFERENCE.md#load_paths-option
`:load_paths` array} alongside normal filesystem paths.

When resolving an `@import`, Sass will go through the load paths looking for an importer
that successfully imports the path. Once one is found, the imported file is used.

User-created importers must inherit from {Sass::Importers::Base}.
