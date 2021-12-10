---
title: '如何编写您自己的广告过滤器'
taxonomy:
    category:
        - docs
visible: true
---

* [介绍](#introduction)
* [注释](#comments)
* [示例](#examples)
    * [示例：通过域名阻止](#example-blocking-by-domain-name)
    * [示例：阻止准确地址](#example-blocking-exact-address)
    * [示例：基础规则修饰符](#example-basic-rule-modifiers)
    * [示例：放行地址](#example-unblocking-an-address)
    * [示例：放行网页上的所有东西](#example-unblocking-website)
    * [示例：装饰规则](#example-cosmetic-rule)
* [基础规则](#basic-rules)
    * [基础规则语法](#basic-rules-syntax)
    * [特殊字符](#basic-rules-special-characters)
    * [正则表达式支持](#regexp-support)
    * [TLD 的通配符支持](#wildcard-for-tld)
    * [基础规则示例](#basic-rules-examples)
    * [修饰符](#basic-rules-modifiers)
        * [基础修饰符](#basic-rules-common-modifiers)
            * [$domain](#domain-modifier)
            * [$third-party](#third-party-modifier)
            * [$popup](#popup-modifier)
            * [$match-case](#match-case-modifier)
        * [内容类型修饰符](#content-type-modifiers)
            * [Content type modifiers examples](#content-type-modifiers-examples)
            * [$document](#document-modifier)
            * [$image](#image-modifier)
            * [$stylesheet](#stylesheet-modifier)
            * [$script](#script-modifier)
            * [$object](#object-modifier)
            * [$object-subrequest](#object-subrequest-modifier)
            * [$font](#font-modifier)
            * [$media](#media-modifier)
            * [$subdocument](#subdocument-modifier)
            * [$ping](#ping-modifier)
            * [$xmlhttprequest](#xmlhttprequest-modifier)
            * [$websocket](#websocket-modifier)
            * [$webrtc](#webrtc-modifier)
            * [$other](#other-modifier)
        * [Exception rules modifiers](#exceptions-modifiers)
            * [$elemhide](#elemhide-modifier)
            * [$content](#content-modifier)
            * [$jsinject](#jsinject-modifier)
            * [$urlblock](#urlblock-modifier)
            * [$extension](#extension-modifier)
            * [$stealth](#stealth-modifier)
            * [Generic rules](#generic-rules)
                * [$generichide](#generichide-modifier)
                * [$genericblock](#genericblock-modifier)
            * [$specifichide](#specifichide-modifier)
    * [Advanced capabilites](#advanced-modifiers)
        * [$removeparam](#removeparam-modifier)
        * [$important](#important-modifier)
        * [$badfilter](#badfilter-modifier)
        * [$empty](#empty-modifier)
        * [$mp4](#mp4-modifier)
        * [$replace](#replace-modifier)
        * [$csp](#csp-modifier)
        * [$cookie](#cookie-modifier)
        * [$network](#network-modifier)
        * [$app](#app-modifier)
        * [$redirect](#redirect-modifier)
        * [$redirect-rule](#redirect-rule-modifier)
        * [$denyallow](#denyallow-modifier)
        * [noop](#noop-modifier)
        * [$removeheader](#removeheader-modifier)
* [Non-basic rules](#non-basic-rules)
    * [Cosmetic rules](#cosmetic-rules)
        * [Element hiding rules](#cosmetic-elemhide-rules)
        * [CSS规则](#cosmetic-css-rules)
        * [Extended CSS selectors](#extended-css-selectors)
            * [伪类 `:has()`](#extended-css-has)
            * [伪类 `:if-not()`](#extended-css-has)
            * [伪类 `:contains()`](#extended-css-contains)
            * [伪类 `:matches-css()`](#extended-css-matches-css)
            * [伪类 `:matches-attr()`](#extended-css-matches-attr)
            * [伪类 `:matches-property()`](#extended-css-matches-property)
            * [伪类 `:xpath()`](#extended-css-xpath)
            * [伪类 `:nth-ancestor()`](#extended-css-nth-ancestor)
            * [伪类 `:upward()`](#extended-css-upward)
            * [伪类 `:remove()` and pseudo-property `remove`](#remove-pseudos)
        * [Cosmetic rules priority](#cosmetic-rules-priority) 
    * [HTML过滤规则](#html-filtering-rules)
        * [Syntax](#html-filtering-rules-syntax)
        * [Special attributes](#html-filtering-rules-attributes)
           * [tag-content](#tag-content-attribute)
           * [wildcard](#wildcard-attribute)
           * [max-length](#max-length-attribute)
           * [min-length](#min-length-attribute)
        * [Exceptions](#html-filtering-rules-exceptions)
    * [JavaScript规则](#javascript-rules)
    * [Scriptlets](#scriptlets)
    * [Modifiers](#non-basic-rules-modifiers)
        * [Syntax](#non-basic-rules-modifiers-syntax)
        * [$app](#non-basic-rules-modifiers-app)
        * [$domain](#non-basic-rules-modifiers-domain)
        * [$path](#non-basic-rules-modifiers-path)
* [Information for filters maintainers](#for_maintainers)
    * [Pre-processor directives](#pre_processor)
    * [Hints](#hints)
        * [Hints syntax](#hints_syntax)
        * [NOT_OPTIMIZED hint](#not_optimized)
        * [PLATFORM and NOT_PLATFORM hints](#platform_not_platform)
* [如何调试过滤规则](#how-to-debug)
    * [Filtering log](#debug-filtering-log)
    * [Selectors debugging mode](#selectors-debugging-mode)
        * [Testing extended selectors](#testing-extended-selectors)
    * [Debugging scriptlets](#debug-scriptlets)
* [祝您编写过滤器顺利！](#good-luck)

<a id="introduction"></a>
## 介绍

过滤器是应用在特殊内容（banners，popups 等）上的一组过滤规则。AdGuard 拥有我们团队编写的一组标准过滤器。我们团队持续地改善并更新它们。我们希望我们的过滤器可以满足绝大多数用户的需求。

同时，AdGuard 允许您使用与我们的过滤器相同的规则编写自己的过滤器。

我们使用[增强的 BNF 语法说明](https://tools.ietf.org/html/rfc5234)来作为我们的过滤器语法规则。但是我们并不总是严格遵循这份说明。

> AdGuard 语法基于 Adblock Plus 语法，并且为了更好的广告过滤，拓展了一些语法。这篇文章中关于 AdGuard 和 ADB 公共部分的语法可以在[这篇文章](https://adblockplus.org/en/filters)当中找到。

<a id="comments"></a>
## 注释

以感叹号开头的一行就是注释。在规则列表中以灰色显示出来。AdGuard 会忽视这一行，因此您可以随便写您想写的东西。注释通常写在规则的上方来描述规则是用来做什么的。

例如:

```
! 这里是注释。下面一行才是过滤规则。
||example.org^
```

<a id="examples"></a>
## 示例

<a id="example-blocking-by-domain-name"></a>
### 示例：通过域名阻止

<object data="https://cdn.adguard.com/public/Adguard/kb/en/rules_syntax/0_blocking_domain.svg" type="image/svg+xml">
    <img src="https://cdn.adguard.com/public/Adguard/kb/en/rules_syntax/0_blocking_domain.svg" />
</object>

**这条规则阻止：**

* `http://example.org/ad1.gif`
* `http://subdomain.example.org/ad1.gif`
* `https://ads.example.org:8000/`

**这条规则不阻止：**

* `http://ads.example.org.us/ad1.gif`
* `http://example.com/redirect/http://ads.example.org/`

<a id="example-blocking-exact-address"></a>
### 示例：阻止准确地址

<object data="https://cdn.adguard.com/public/Adguard/kb/en/rules_syntax/1_exact_address.svg" type="image/svg+xml">
    <img src="https://cdn.adguard.com/public/Adguard/kb/en/rules_syntax/1_exact_address.svg" />
</object>

**这条规则阻止：**

* `http://example.org/`

**这条规则不阻止：**

* `https://example.org/banner/img`

<a id="example-basic-rule-modifiers"></a>
### 示例：基础规则修饰符

过滤规则支持许多修饰符，这些修饰符使您可以微调规则的行为。 以下是使用了修饰符的规则示例：

<object data="https://cdn.adguard.com/public/Adguard/kb/en/rules_syntax/2_basic_rule_options.svg" type="image/svg+xml">
    <img src="https://cdn.adguard.com/public/Adguard/kb/en/rules_syntax/2_basic_rule_options.svg" />
</object>

**这条规则阻止：**

* `http://example.org/script.js` 如果这条脚本来自 `example.com`。

**这条规则不阻止：**

* `https://example.org/script.js` 如果这条脚本来自 `example.org`。
* `https://example.org/banner.png` 因为它不是一个脚本。

<a id="example-unblocking-an-address"></a>
### 示例：放行地址

<object data="https://cdn.adguard.com/public/Adguard/kb/en/rules_syntax/3_basic_exception.svg" type="image/svg+xml">
    <img src="https://cdn.adguard.com/public/Adguard/kb/en/rules_syntax/3_basic_exception.svg" />
</object>

**这条规则放行：**

* `http://example.org/banner.png` 即使有规则阻止了这个地址。

> 使用修饰符 [`$important`](#important-modifier) 可以覆写其他规则。

<a id="example-unblocking-website"></a>
### 放行网页上的所有东西

<object data="https://cdn.adguard.com/public/Adguard/kb/en/rules_syntax/4_unblock_entire_website.svg" type="image/svg+xml">
    <img src="https://cdn.adguard.com/public/Adguard/kb/en/rules_syntax/4_unblock_entire_website.svg" />
</object>

**这条规则阻止**

* 它阻止所有作用 `example.com` 上的装饰规则。
* 它放行所有来自此网页上的请求，即使有过滤规则匹配到了这些请求。

<a id="example-cosmetic-rule"></a>
### 示例：装饰规则

<object data="https://cdn.adguard.com/public/Adguard/kb/en/rules_syntax/5_cosmetic_rules.svg" type="image/svg+xml">
    <img src="https://cdn.adguard.com/public/Adguard/kb/en/rules_syntax/5_cosmetic_rules.svg" />
</object>

装饰规则基于每个浏览器都能理解的特殊语言 CSS。最基础的是它可以给网页上将要隐藏的特定元素添加一个 CSS 样式。您可以从[这里](https://developer.mozilla.org/zh-CN/docs/Learn/CSS/Building_blocks/Selectors)了解更多 CSS 的知识。

> AdGuard [拓展 CSS](#extended-css-selectors)并且可以使过滤器开发者处理更多复杂的情形。但是想要使用这些规则，您必须熟练使用常用的 CSS。

<a id="example-popular-css-selectors"></a>
#### 常用的 CSS 选择器

| 名字 | CSS 选择器 | 描述 |
| ------| ------ | ----------- |
| ID 选择器 | `#banners`   | 匹配所有 `id` 属性的值等于 `banners` 的元素。<br/>![](https://cdn.adguard.com/public/Adguard/kb/en/rules_syntax/css_id_selector.png) |
| 类选择器 | `.banners`   | 匹配所有 `class` 属性的值包含 `banners` 的元素。<br/>![](https://cdn.adguard.com/public/Adguard/kb/en/rules_syntax/css_class_selector.png) |
| 属性选择器 | `div[class="banners"]`   | 匹配所有 `class` 属性值**完全等于** `banners` 的 `div` 元素。<br/>![](https://cdn.adguard.com/public/Adguard/kb/en/rules_syntax/css_class_attr.png) |
| 属性字串选择器 | `div[class^="advert1"]`   | 匹配所有 `class` 属性值以 `advert1` **开头** 的 `div` 元素。<br/>![](https://cdn.adguard.com/public/Adguard/kb/en/rules_syntax/css_class_attr_start.png) |
| 属性字串选择器 | `div[class$="banners_ads"]`   | 匹配所有 `class` 属性值以 `advert1` **结尾** 的 `div` 元素。<br/>![](https://cdn.adguard.com/public/Adguard/kb/en/rules_syntax/css_class_attr_end.png) |
| 属性字串选择器 | `a[href^="http://example.com/"]`   | 匹配所有加载自 `http://example.com/` 的链接。<br/>![](https://cdn.adguard.com/public/Adguard/kb/en/rules_syntax/css_attr_start.png) |
| 属性选择器 | `a[href="http://example.com/"]`   | 匹配所有链接地址 **等于** `http://example.com/` 。<br/>![](https://cdn.adguard.com/public/Adguard/kb/en/rules_syntax/css_attr_equal.png) |

<a id="basic-rules"></a>
## 基础规则

最简单的规则被叫做 _“基础规则”_。它们通常用来阻止来自特殊 URL 的请求。如果在规则开头有一个特殊的标记“@@”就是取消阻止它。这种类型的规则的基本原理非常简单：您必须指定地址和额外的参数来限制或扩展规则的范围。

> #### 子请求
> 阻止请求的基础规则被作用到 **子请求** 上。这意味着它们不会阻止页面的加载。

> #### 响应状态
> 浏览器检测到被阻止的请求以一个错误完成。

<a id="basic-rules-syntax"></a>
### 基础规则语法

```
      rule = ["@@"] pattern [ "$" modifiers ]
modifiers = [modifier0, modifier1[, ...[, modifierN]]]
```

* **`pattern`** — 地址掩码。每一个请求的 URL 都与这个掩码核对。您也可以使用模板里的特殊字符它们的说明[如下](#basic-rules-special-characters)。请注意 AdGuard 会把 URLs 修剪为 4096 个字符长来加快匹配速度和避免 URLs 过长的问题。
* **`@@`** — 一个例外规则的标记。要想取消阻止您的规则，请使用这个标记作为您规则的开头。
* **`modifiers`** — “明确”基础规则的参数。一些用来限制规则的范围，一些可以完全改变规则的工作方式。

<a id="basic-rules-special-characters"></a>
### 特殊字符

* ```*``` — 通配符。它用来表示“”任何字符集。它也可以表示一个空字符串或任意长度的字符串。
* **`||`** — 匹配地址开头。使用这个符号您就不必指定特定的协议和地址掩码中的子域名。也就是说，`||` 一次性代表了 `http://*.`，`https://*.`，`ws://*.`，`wss://*.`。
* **`^`** — 分隔符。分隔符是除了字母、数字或者`_` `-` `.` `%`以外的任何字符。这个例子中分隔符使用粗体展示：`http:`**`//`**`example.com`**`/?`**`t=1`**`&`**`t2=t3`。地址的结尾同样可以作为一个分隔符。
* **`|`** — 指向地址开头或者结尾的指针。它的值取决于掩码中字符的位置。比方说，一个规则 `swf|` 对应于 `http://example.com/annoyingflash.swf`，而不是 `http://example.com/swf/index.html`。`|http://example.org` 对应于 `http://example.org`，而不是 `http://domain.com?url=http://example.org`。

> **直观表示** 我们同样建议您打开[这篇文章](https://adblockplus.org/filter-cheatsheet#blocking)来更好的理解应该如何编写这样的规则。

<a id="regexp-support"></a>
### 正则表达式支持

如果您想更灵活地编写规则，您可以使用[正则表达式](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Regular_Expressions)来代替带有特殊字符的默认简化掩码。

> **性能** 使用了正则表达式的规则运行的更慢，因此更简易阻止他们或将他们的作用域限制在特殊的域名下。

如果您想让过滤器使用正则表达式，“模板”应当看起来是这样的：
```
pattern = "/" regexp "/"
```

举例来说，`/banner\d+/$third-party` 这条规则将正则表达式 `banner\d+` 作用到所有的第三方请求。排除使用正则表达式的规则应当是这样的：`@@/banner\d+/`。

> **不同版本 AdGuard 的兼容性** AdGuard 的 Safari 浏览器拓展和 iOS 版 AdGuard 由于[内容组织 API 限制](https://webkit.org/blog/3476/content-blockers-first-look/)并不完整支持正则表达式（请查看“The Regular expression format”章节）。

<a id="wildcard-for-tld"></a>
### TLD （顶级域名）的通配符支持

装饰规则、html 规则和 javascript 规则均为域名的 TLD 提供了通配符的支持。
比方说，装饰规则 `example.*##.banner` 将匹配任何 `example.TLD` 请求（`example.ru`, `example.com`, `example.net`, `example.org` 等等）。
对于基础规则，所描述的逻辑仅适用于指定了 `$domain` 修饰符的域名（比如  `||*/banners/*$image,domain=example.*`）。

> **不同版本 AdGuard 的兼容性** 支持：Windows/macOS/Android/Safari/iOS 版本的 AdGuard 和 Chrome/Firefox/Edge 的浏览器拓展。

<a id="basic-rules-examples"></a>
### 基础规则示例

* `||example.com/ads/*` — 一个简单的规则，匹配的地址比如 `http://example.com/ads/banner.jpg` 乃至 `http://subdomain.example.com/ads/otherbanner.jpg`。

* `||example.org^$third-party` —  这条规则阻止请求 `example.org` 和它的子域名的第三方请求。

* `@@||example.com$document` — 一般的例外规则。它完全取消过滤 `example.com` 和所有子域名。有许多可以使用在例外规则上的修饰符。想要了解更多，请点击[下方](#exceptions-modifiers)的链接。

<a id="basic-rules-modifiers"></a>
### 修饰符

> **注意！** 这个小节所描述的特性是给有经验的用户准备的。它们拓展了“基础规则的能力”，但是为了使用它们您需要对您的浏览器的工作方式有基本的了解。

您可以使用额外的修饰符来改变“基础规则”的行为。修饰符应当被放在规则末尾的“$符号后面并且用英文逗号分隔。

示例：
```
||domain.com^$popup,third-party
```

<a id="basic-rules-common-modifiers"></a>
#### 基础修饰符

下面的修饰符是最简单而且最常用的。

<a id="domain-modifier"></a>
##### **`domain`**

`domain` 将规则的应用范围限制在一组域名（和它们的子域名）里。为了在一个规则里增加多个域名，使用 `|` 作为分隔符。

###### `domain` 示例 

* `||baddomain.com^$domain=example.org` — 一个阻止与指定的掩码匹配并且从域名 `example.org` 或它的子域名发送的请求的规则。
* `||baddomain.com^$domain=example.org|example.com` — 同样的规则，但是它同时适用于 `example.org` 和 `example.com`。

如果你不想让规则作用在某个域名上，使用 `~` 作为域名的开头。

###### `domain` 和 `~` 示例

* `||baddomain.com^$domain=~example.org` — 这条规则阻止与模式匹配并且从任何域名除了 `example.org` 和它的子域名发送的请求。
* `||baddomain.com^$domain=example.org|~foo.example.org` — 这个规则阻止了发送自 `example.org` 和它除 `foo.example.org` 之外的子域名的请求。

###### `domain` 修饰符匹配目标域名

在某些情况下 `$domain` 修饰符可以匹配来源域名和目标域名。它在下列条件为真时将发生：

1) 请求具有 `document` 类型
2) 规则模式并不匹配任何特定的域名
3) 规则模式并不包含正则表达式

当所有的条件均满足时，`domain` 修饰符将匹配来源域名**和**目标域名。

当部分条件没有满足但是规则包含了修饰符 `cookie` 或 `csp`，目标域名仍能匹配到。

如果来源页面匹配带有 `domain` 修饰符并明确排除来源域名，这时规则将不会被应用即使目标域名也匹配了规则。这也会影响带有 `cookie` 和 `csp` 修饰符的规则。

**示例：**

* `*$cookie,domain=example.org|example.com` 将阻止所有来自和到 `example.org` 和 `example.com`的请求的 cookies。
* `*$document,domain=example.org|example.com` 将阻止所有来自和到 `example.org` 和 `example.com`的请求。

下面的例子中暗示了从 `http://example.org/page`（来源页面）发送的请求，目标页面为 `http://targetdomain.com/page`。

* `page$domain=example.org` 当来源域名匹配时将会被匹配。
* `page$domain=targetdomain.com` 当它匹配目标域名但满足上述所有要求时会被匹配。
* `||*page$domain=targetdomain.com` 当 `||*page` 匹配到指定的域名时不会被匹配。
* `||*page$domain=targetdomain.com,cookie` 尽管 `||*page` 匹配到指定的域名也会被匹配，因为它包含 `$cookie` 修饰符。
* `/banner\d+/$domain=targetdomain.com` 当其包含了正则表达式时不会被匹配。
* `page$domain=targetdomain.com|~example.org` 不会被匹配因为来源域名被明确排除。

> **重要！** Safari 并不同时支持允许和限制域名。因此像 `||baddomain.com^$domain=example.org|~foo.example.org` 的规则在 AdGuard for Safari 里是不工作的。


<a id="third-party-modifier"></a>
##### **`third-party`**

限制第三方和自己的请求。第三方请求是来自一个不同的域名的请求。比如一个从 `domain.com` 到 `example.org` 的请求就是第三方请求。

> 第三方请求应满足下列条件之一：
> 1) 它的引用不是目标域名的子域名，反之亦然。比方说，一个来自 `example.org` 到 `subdomain.example.org` 的请求就不是第三方请求。
> 2) 它的 `Sec-Fetch-Site` 头被设置为 `cross-site`。

如果有一个 `third-party` 修饰符，这条规则仅被作用于第三方请求。

###### `third-party` 示例

* `||domain.com^$third-party` — 这条规则作用在除了 `domain.com` 和它的子域名之外的所有域名之上。第三方请求示例： `http://example.org/banner.jpg`.

如果有一个 `~third-party` 修饰符，这条规则将仅作用于没有来自第三方的请求。也就是说，它们必须被同一个域名所发送。

###### `~third-party` 示例

* `||domain.com$~third-party` — 这条规则作用于 `domain.com` 而不是其他的域名。比如请求 `http://domain.com/icon.ico` 就不是一个第三方请求。

<a id="popup-modifier"></a>
##### **`popup`**

AdGuard 会尝试关闭任何与此修饰符的阻止规则相匹配的地址的浏览器标签页。请注意不是所有的标签页都会被关闭。

###### `popup` 示例

* `||domain.com^$popup` — 如果你尝试从浏览器的任何页面跳转到 `http://domain.com/` ，此规则将关闭打开指定站点的新标签。

> 如果弹出页面被浏览器缓存的话此规则并不会工作。对于一些棘手的弹出页面此规则也不会工作。在这些情形下，最好使用  [AdGuard Popup Blocker](https://github.com/AdguardTeam/PopupBlocker) 拓展。

> **重要！** 与浏览器拓展不同， `$popup` 修饰符当使用 AdGuard Windows，Mac 和 Android 版应用时非常不可靠。在 AdGuard Safari 和 iOS 版中，`$popup` 规则将立即阻止页面。

<a id="match-case-modifier"></a>
##### **`match-case`**

此修饰符定义了适用于匹配大小写的地址的规则。默认规则不区分大小写。

###### `match-case` 示例

* `*/BannerAd.gif$match-case` — 此规则将阻止 `http://example.com/BannerAd.gif`，但不阻止 `http://example.com/bannerad.gif`。

<a id="content-type-modifiers"></a>
#### 内容类型的限制

这里有一系列用来限制规则的应用范围到确定内容的类型的修饰符。这些修饰符也可以相互结合，例如同时支持图片和脚本。

> **不同版本 AdGuard 的兼容性** 请注意，AdGuard 在不同的平台上确定内容类型上有很大的差异。对于浏览器拓展而言，每个请求的内容类型由浏览器提供。AdGuard Windows，Mac 和 Android 版则遵循以下规则：首先我们尝试使用 `Sec-Fetch-Dest` 请求头或文件名后缀来确定请求的类型。如果这种情形下请求没有被阻止，将会使用在服务器相应的 `Content-Type` 请求头作为将要确定的类型。

<a id="content-type-modifiers-examples"></a>
##### 内容类型修饰符示例

* `||example.org^$image` — 对应所有来自 `example.org` 的图片。
* `||example.org^$script,stylesheet` — 对应所有来自 `example.org` 的脚本和样式。
* `||example.org^$~image,~script,~stylesheet` — 对应所有除了图片、脚本和样式以外来自 `example.org` 的请求。

<a id="document-modifier"></a>
##### **`document`**

此规则对应主框架的文档请求，例如在浏览器标签页内加载的 HTML 文档。它并不匹配内嵌框架（有一个 `$subdocument` 修饰符来处理这种情况）。

默认情况下，AdGuard 并不会阻止在浏览器标签页加载的请求（例如“主框架绕过”）。这个想法并不是为了阻止页面的加载，而是用户明确表示他们想要页面被加载。然而如果 `$document` 修饰符被明确指定，AdGuard 并不会遵从上述逻辑并且阻止页面加载。相对的，它返回一个“阻止页面”。

如果此修饰符使用了排除规则（`@@`），它完全不会阻止匹配的页面。这相当于同时使用`$elemhide`，`$content`，`$urlblock`，`$jsinject` 和 `$extension` 修饰符。

> **不同版本 AdGuard 的兼容性** 阻止请求类型逻辑现在仅支持开发版的 AdGuard。

###### `document` 示例

* `@@||example.com^$document` — 完全禁用在 `example.com` 和所有子域名上所有页面的过滤。
* `@@||example.com^$document,~extension` — 完全不会阻止在 `example.com` 和所有子域名上所有页面，但是仍然会运行用户脚本。

* `||example.com^$document` — 以阻止页面来阻止请求 `example.com` 的 HTML 文档。
* `||example.com^$document,redirect=noopframe` — 重定向请求 `example.com` 的 HTML 文档到一个空 HTML 文档。
* `||example.com^$document,removeparam=test` — 在请求 `example.com` 的 HTML 文档中移除 `test` 查询参数。
* `||example.com^$document,replace=/test1/test2/` — 在请求 `example.com` 的 HTML 文档中替换 `test1` 为 `test2`。

<a id="image-modifier"></a>
##### **`image`**

这条规则对应于图像请求。

<a id="stylesheet-modifier"></a>
##### **`stylesheet`**

这条规则对应于 CSS 文件请求。

<a id="script-modifier"></a>
##### **`script`**

这条规则对应于脚本请求（比如 javascript、vbscript）。

<a id="object-modifier"></a>
##### **`object`**

这条规则对应于浏览器插件资源（比如 Java 或 Flash）。

<a id="object-subrequest-modifier"></a>
##### **`object-subrequest`**

这条规则对应于浏览器插件的请求（通常是 Flash）。

> **不同版本 AdGuard 的兼容性** AdGuard for Windows、macOS 和 Android 通常不能准确侦测这种类型，并且把它定义为 `other`。

<a id="font-modifier"></a>
##### **`font`**

这条规则对应于字体请求。（例如 .woff 文件拓展名）。

<a id="media-modifier"></a>
##### **`media`**

这条规则对应于媒体文件的请求（音乐和视频，比如 .mp4 文件）。

<a id="subdocument-modifier"></a>
##### **`subdocument`**

这条规则对应于内置页面的请求（HTML 标签 `frame` 和 `iframe`）。

<a id="ping-modifier"></a>
##### **`ping`**

这条规则对应于链接属性 `navigator.sendBeacon()` 或 `ping` 所发起的请求。

> **不同版本 AdGuard 的兼容性** AdGuard for Windows、macOS 和 Android 经常不能准确侦测 `navigator.sendBeacon()`。为了更可靠的侦测，请使用 AdGuard 浏览器拓展。

<a id="xmlhttprequest-modifier"></a>
##### **`xmlhttprequest`**

这条规则仅应用于 ajax 请求（请求通过 javascript 对象 `XMLHttpRequest` 发送）。

> **不同版本 AdGuard 的兼容性** AdGuard for Windows, macOS 和 Android 经常不能准确侦测到这种类型，有时被侦测为 `other` 或 `script`。

<a id="websocket-modifier"></a>
##### **`websocket`**

这条规则仅应用于 WebSocket 连接。

> **不同版本 AdGuard 的兼容性** AdGuard for Safari 和 iOS 由于 Safari 的限制不能很好地应用这个修饰符。

<a id="webrtc-modifier"></a>
##### **`webrtc`**

这条规则仅应用于 WebRTC 连接。

> 请注意，阻止 WebRTC 会干扰某些浏览器应用程序的工作，例如信息、聊天、视频或游戏。

###### `webrtc` 示例

* `||example.com^$webrtc,domain=example.org` - 此规则会阻止连接 `example.com` 和 `example.org` 的 webRTC。
* `@@*$webrtc,domain=example.org` - 此规则禁用了 `example.org`的 RTC 包装。

> **弃用通知** 这个修饰符已经被弃用并在将来会被移除。如过你需要阻止 WebRTC，考虑使用 `$nowebrtc` 脚本段。

<a id="other-modifier"></a>
##### **`other`**

这条规则作用于类型未被定义或没有在上面列出的类型的请求。

<!-- TODO -->
<a id="exceptions-modifiers"></a>
#### Exception modifiers

Exception rules disable the other basic rules for the addresses to which they correspond. They begin with a `@@` mark. All the basic modifiers listed above can be applied to them and they also have a few special modifiers.

> **Visual representation.** We recommend to get acquainted with [this article](https://adblockplus.org/filter-cheatsheet#blocking), for better understanding of how exception rules should be made.

<a id="elemhide-modifier"></a>
##### **`elemhide`**

Disables any cosmetic rules on the pages matching the rule. You will find the information about cosmetic rules [further](#cosmetic-rules).

###### `elemhide` example

* `@@||example.com^$elemhide` — disables all cosmetic rules on pages at `example.com` and all subdomains.

<a id="content-modifier"></a>
##### **`content`**

Disables HTML filtering rules and replace rules on the pages that match the rule. You will find the information about HTML filtering rules [here](#html-filtering-rules) and about replace rules [here](#replace-modifier).

###### `content` example

* `@@||example.com^$content` — disables all HTML filtering rules on pages at `example.com` and all subdomains.

<a id="jsinject-modifier"></a>
##### **`jsinject`**

Forbids adding of javascript code to the page. You can read about javascript rules further.

###### `jsinject` example

* `@@||example.com^$jsinject` — disables javascript on pages at `example.com` and all subdomains.

<a id="urlblock-modifier"></a>
##### **`urlblock`**

Disables blocking of all requests sent from the pages matching the rule.

###### `urlblock` example

* `@@||example.com^$urlblock` — any requests sent from the pages at `example.com` and all subdomains are not going to be blocked.

<a id="extension-modifier"></a>
##### **`extension`**

Disables all userscripts on the pages matching this rule. Note, that this modifier only makes sense in AdGuard products that can work as userscript hosts (AdGuard for Windows/macOS/Android).

###### `extension` example

* `@@||example.com^$extension` — userscripts won't work on all pages of the `example.com` website.

<a id="stealth-modifier"></a>
##### **`stealth`**

Disables the Stealth Mode module for all corresponding pages and requests.

###### `stealth` examples

* `@@||example.com^$stealth` — disables `Stealth Mode` for `example.com` (and subdomains) requests, except for blocking cookies and hiding tracking parameters (see below).
* `@@||domain.com^$script,stealth,domain=example.com` — disables `Stealth Mode` only for script requests to `domain.com` (and its subdomains) on `example.com` and all its subdomains.
* Please note that blocking cookies and removing tracking parameters is achieved by using rules with `$cookie` and `$removeparam` modifiers. Exceptions with only `$stealth` modifier won't do those things. If you want to completely disable all Stealth Mode features for a given URL, you need to include all three modifiers: `@@||example.org^$stealth,removeparam,cookie`

> **不同版本 AdGuard 的兼容性** Stealth Mode is currently available in AdGuard for Windows, Mac, Android and AdGuard browser extensions for Chrome, Firefox, Edge. For now, the products that do not support Stealth Mode will ignore the rules with this modifier.

<a id="generic-rules"></a>
##### Generic rules

Before we can proceed to the next modifiers, we have to make a definition of _generic rules_. The rule is generic if it is not limited to specific domains.
Wildcard character `*` is supported as well.

For example, these rules are generic:
```
###banner
*###banner
#@#.adsblock
*#@#.adsblock
~domain.com###banner
||domain.com^
||domain.com^$domain=~example.com
```

And these are not:
```
domain.com###banner
||domain.com^$domain=example.com
```

<a id="generichide-modifier"></a>
###### **`generichide`**

Disables all generic [cosmetic rules](#cosmetic-rules) on pages that correspond to exception rule.

* `@@||example.com^generichide` — disables generic cosmetic rules on any pages at `example.com` and all subdomains.

<a id="genericblock-modifier"></a>
###### **`genericblock`**

Disables generic basic rules on pages that correspond to exception rule.

* `@@||example.com^$genericblock` — disables generic basic rules on any pages at `example.com` and all subdomains.

<a id="specifichide-modifier"></a>
##### **`specifichide`**

Has an opposite effect to [`generichide`](#generichide-modifier). Disables all specific element hiding and CSS rules, but not general ones.

* `@@||example.org^$specifichide` — will disable `example.org##.banner` but not `##.banner`.

> Please note that [`$elemhide` modifier](#elemhide-modifier) can disable all cosmetic rules at once.

> **不同版本 AdGuard 的兼容性** Rules with this modifier are supported by AdGuard for Windows, Mac, Android, and AdGuard browser extensions for Chrome, Firefox, Edge. **Developer builds only at this moment.**

<a id="advanced-modifiers"></a>
### Advanced capabilities

These modifiers are able to completely change the behaviour of basic rules.

<a id="removeparam-modifier"></a>
#### **`removeparam`**

>`$removeparam` and `$queryprune` are completely interchangeable and are aliases to each other.

Rules with `$removeparam` modifier are intended to to strip query parameters from requests' URLs. Please note that such rules are only applied to `GET`, `HEAD`, and `OPTIONS` requests.

> `$removeparam` rules that do not have any content-type modifiers will match only requests where content type is `document`.

##### Syntax

###### Basic syntax

* `$removeparam=param` -- removes query parameter with the name `param` from URLs of any request, e.g. a request to `http://example.com/page?param=1&another=2` will be transformed to `http://example.com/page?another=2`.

> `$removeparam` basic syntax is supported starting with v1.7 of [CoreLibs](https://adguard.com/en/blog/introducing-corelibs.html) and v3.6 of AdGuard Browser Extension.

###### Regular expressions

You can also use regular expressions to match query parameters and/or their values:

* `$removeparam=/regex/[options]` -- removes query parameters matching the regex regular expression from URLs of any request. Unlike basic syntax, it means *"remove query parameters normalized to a `name=value` string which match the regex regular expression"*. `[options]` here is the list of regular expression options. At the moment, the only supported option is `i` which makes matching case-insensitive.

> `$removeparam` syntax for regular expressions will be supported starting with v1.8 of CoreLibs and v4.0 of AdGuard Browser Extension. For now, use the simplified version: `$removeparam=param`.

> **Escaping special characters**: don't forget to escape special characters like `,`, `/` and `$` in the regular expressions. Use `\` character for that purpose. For example, an escaped comma should look like this: `\,`.

> Important: note that regex-type rules target both parameter's name and value. In order to minimize the chance of mistakes, it is safer to start every regex with `/^` unless you specifically target parameter values.

> We will try to detect and ignore unescaped `$` automatically using a simple rule of thumb:
> It is not an options delimiter if all three are true:
> 1. It looks like `$/`,
> 2. There's another slash character (`/`) to the left of it,
> 3. There's another unescaped `$` character to the left of that slash character.

###### Remove all query parameters

Specify naked `$removeparam` to remove all query parameters:

* `||example.org^$removeparam` -- removes all query parameters from URLs matching `||example.org^`.

###### Inversion

Use `~` to apply inversion:

* `$removeparam=~param` -- removes all query parameters with the name different from `param`.
* `$removeparam=~/regex/` -- removes all query parameters that do not match the regex regular expression.

###### Negating `$removeparam`

This sort of rules work pretty much the same way it works with [`$csp`](#csp-modifier) and [`$redirect`](#redirect-modifier) modifiers.

Use `@@` to negate `$removeparam`:

* `@@||example.org^$removeparam` -- negates all `$removeparam` rules for URLs that match `||example.org^`.
* `@@||example.org^$removeparam=param` -- negates the rule with `$removeparam=param` for any request matching `||example.org^`.
* `@@||example.org^$removeparam=/regex/` -- negates the rule with `$removeparam=/regex/` for any request matching `||example.org^`.

>**Multiple rules matching a single request**
>In the case when multiple `$removeparam` rules match a single request, each of them will be applied one by one.

##### Examples

```
$removeparam=utm_source|utm_medium|utm_term
$removeparam=utm_content|utm_campaign|utm_referrer
@@||example.com^$removeparam
```
With these rules some [UTM parameters](https://en.wikipedia.org/wiki/UTM_parameters) will be stripped out from any request, except that requests to `example.com` won't be stripped at all, e.g. `http://google.com/page?utm_source=s&utm_referrer=fb.com&utm_content=img` will be transformed to `http://google.com/page`, but `http://example.com/page?utm_source=s&utm_referrer=fb.com&utm_content=img` won't be affected by the blocking rule.

* `$removeparam=utm_source` -- removes `utm_source` query parameter from all requests.

* `$removeparam=/utm_.*/` -- removes all `utm_* query` parameters from URL queries of any request, e.g. a request to `http://example.com/page?utm_source=test` will be transformed to `http://example.com/page`.

* `$removeparam=/^utm_source=campaign$/` -- removes `utm_source` query parameter with the value equal to `campaign`. It does not touch other `utm_source` parameters.

Negating one `$removeparam` rule and replacing it with a different rule:

```
$removeparam=/^(gclid|yclid|fbclid)=/
@@||example.com^$removeparam=/^(gclid|yclid|fbclid)=/
||example.com^$removeparam=/^(yclid|fbclid)=/
```

With these rules, Google, Yandex, and Facebook Click IDs will be removed from all requests. There's one exception: Google Click ID (gclid) will not be removed from requests to example.com.

Negating `$removeparam` for all parameters:

```
$removeparam=/^(utm_source|utm_medium|utm_term)=/
$removeparam=/^(utm_content|utm_campaign|utm_referrer)=/
@@||example.com^$removeparam
```

With these rules, specified UTM parameters will be removed from any request save for requests to example.org.

> **Compatibility with other modifiers**
> `$removeparam` rules are compatible with [basic modifiers](#basic-rules-common-modifiers), [content-type modifiers](#content-type-modifiers), and with `$important` and `$app` modifiers. The rules which have any other modifiers are considered invalid and will be discarded.

> Please note that `$removeparam` rules can also be disabled by `$document` and `$urlblock` exception rules. But basic exception rules without modifiers don't do that. For example, `@@||example.com^` will not disable `$removeparam=p` for requests to **example.com**, but `@@||example.com^$urlblock` will.

> **不同版本 AdGuard 的兼容性** Rules with this modifier are supported by AdGuard for Windows, Mac, Android, and AdGuard browser extensions for Chrome, Firefox, Edge. **Developer builds only at this moment.**

> **Restrictions.** Please note that this type of rules can be used **only in trusted filters**. This category includes your own **User filter** and all the filters created by AdGuard Team.

<a id="important-modifier"></a>
#### **`important`**

The `$important` modifier applied to a rule increases its priority over any other rule without `$important` modifier. Even over basic exception rules.

##### Example 1:

```
||example.org^$important
@@||example.org^
```

`||example.org^$important` will block all requests despite of the exception rule.

##### Example 2:

```
||example.org^$important
@@||example.org^$important
```

Now the exception rule also has `$important` modifier so it will prevail.

##### Example 3:

The `$important` modifier will be ignored if a document-level exception rule is applied to the document.

```
||example.org^$important
@@||test.org^$document
```

If a request to `example.org` is sent from the `test.org` domain, the rule won't be applied despite it has the `$important` modifier.

<a id="badfilter-modifier"></a>
#### **`badfilter`**

The rules with the `badfilter` modifier disable other basic rules to which they refer. It means that the text of the disabled rule should match the text of the `badfilter` rule (without the `badfilter` modifier).

**Examples:**

* `||example.com$badfilter` disables `||example.com`
* `||example.com$image,badfilter` disables `||example.com,image`
* `@@||example.com$badfilter` disables `@@||example.com`
* `||example.com$domain=domain.com,badfilter` disables `||example.com$domain=domain.com`

Rules with `$badfilter` modifier can disable other basic rules for specific domains if they fulfil the following conditions:

* The rule has a `$domain` modifier
* The rule does not have a negated domain `~` in `$domain` modifier's value.

In that case, the `$badfilter` rule will disable the corresponding rule for domains specified in both the `$badfilter` and basic rules. Please note, that [wildcard-TLD logic](https://kb.adguard.com/en/general/how-to-create-your-own-ad-filters#wildcard-for-tld) works here as well. 

**Examples:**

* `/some$domain=example.com|example.org|example.io` is disabled for `example.com` by `/some$domain=example.com,badfilter`
* `/some$domain=example.com|example.org|example.io` is disabled for `example.com` and `example.org` by `/some$domain=example.com|example.org,badfilter`
* `/some$domain=example.com|example.org` and `/some$domain=example.io` are disabled completely by `/some$domain=example.com|example.org|example.io,badfilter`
* `/some$domain=example.com|example.org|example.io` is disabled completely by `/some$domain=example.*,badfilter`
* `/some$domain=example.*` is disabled for `example.com` and `example.org` by `/some$domain=example.com|example.org,badfilter`
* `/some$domain=example.com|example.org|example.io` is NOT disabled for `example.com` by `/some$domain=example.com|~example.org,badfilter` because the value of `domain` modifier contains a negated domain

<a id="empty-modifier"></a>
#### **`empty`**

Usually, blocked requests look like a server error to browser. If you use `empty` modifier, AdGuard will emulate a blank response from the server with` 200 OK` status.

##### `empty` example

* `||example.org^$empty` — returns an empty response to all requests to `example.org` and all subdomains.

> **Deprecation notice.** Rules with this modifier are deprecated in favor of the `$redirect` modifier. Please note that it will be removed in the future.

> **不同版本 AdGuard 的兼容性** Rules with this modifier are not supported by AdGuard for Safari and iOS.

<a id="mp4-modifier"></a>
#### **`mp4`**

As a response to blocked request AdGuard returns a short video placeholder.

##### `mp4` example

* `||example.com/videos/$mp4` — block a video downloads from `||example.com/videos/*` and changes the response to a video placeholder.

> **Deprecation notice.** Rules with this modifier are deprecated in favor of the `$redirect` modifier. Please note that it will be removed in the future.

> **不同版本 AdGuard 的兼容性** Rules with this modifier are not supported by AdGuard for Safari and iOS.

<a id="replace-modifier"></a>
#### **`replace`**

This modifier completely changes the rule behavior. If it is applied, the rule will not block the request. The response is going to be modified instead. 

> #### Please note
> You will need some knowledge of regular expressions to use this modifier.

##### `$replace` rules features

* `$replace` rules apply to any text response, but will not apply to binary (`media`, `image`, `object`, etc).
* `$replace` rules do not apply if the size of the original response is more than 3MB.
* `$replace` rules have a higher priority than other basic rules (**including** exception rules). So if a request corresponds to two different rules one of which has the `$replace` modifier, this rule will be applied.
* Document-level exception rules with `$content` or `$document` modifiers do disable `$replace` rules for requests matching them.
* Other document-level exception rules (`$generichide`, `$elemhide` or `$jsinject` modifiers) are applied alongside `$replace` rules. It means that you can modify the page's content with a `$replace` rule and disable cosmetic rules there at the same time.

> `$replace` value can be empty in the case of exception rules. See examples section for further information.

> **Multiple rules matching a single request.** In case if multiple `$replace` rules match a single request, we will apply each of them. **The order is defined alphabetically.**

##### **$replace Syntax**

In general, `replace` syntax is similar to replacement with regular expressions [in Perl](http://perldoc.perl.org/perlrequick.html#Search-and-replace).

```
replace = "/" regex "/" replacement "/" modifiers
```

* `regex` — regular expression.
* `replacement` — a string that will be used to replace the string corresponding to `regex`.
* `modifiers` — regular expression flags. For example, `i` - insensitive search, or `s` - single-line mode.

In the `$replace` value, two characters must be escaped: comma (`,`) and (`$`). Use (`\`) for it. For example, an escaped comma looks like this: `\,`.

##### `$replace` example

```
||example.org^$replace=/(<VAST[\s\S]*?>)[\s\S]*<\/VAST>/\$1<\/VAST>/
```

There are three parts in this rule:

* Regular expression: `(<VAST(.|\s)*?>)(.|\s)*<\/VAST>`
* Replacement: `\$1<\/VAST>` (please note that `$` is escaped)
* Regular expression flags: `i` (insensitive search)

You can see how this rule works here:
http://regexr.com/3cesk

##### Multiple `$replace` rules example

1. `||example.org^$replace=/X/Y/`
2. `||example.org^$replace=/Z/Y/`
3. `@@||example.org/page/*$replace=/Z/Y/`

* Both rule 1 and 2 will be applied to all requests sent to `example.org`.
* Rule 2 is disabled for requests matching `||example.org/page/`, **but rule 1 still works!**.

##### Disabling `$replace` rules

* `@@||example.org^$replace` will disable all `$replace` rules matching `||example.org^`.
* `@@||example.org^$document` or `@@||example.org^$content` will disable all `$replace` rules **originated from** pages of `example.org` **including the page itself**.

> **不同版本 AdGuard 的兼容性** These rules are supported by AdGuard for Windows, Mac, Android and by the AdGuard's Firefox add-on. This type of rules don't work in extensions for other browsers because they are unable to modify content on the network level.

> **Restrictions.** Please note that this type of rules can be used **only in trusted filters**. This category includes your own **User filter** and all the filters created by AdGuard Team.

<a id="csp-modifier"></a>
#### **`csp`**

This modifier completely changes the rule behavior. If it is applied to a rule, it will not block the matching request. The response headers are going to be modified instead.

> In order to use this type of rules, it is required to have the basic understanding of the [Content Security Policy](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Content-Security-Policy) security layer.

For the requests matching a `$csp` rule, we will strengthen response's security policy by adding additional content security policy equal to the `$csp` modifier contents. `$csp` rules are applied independently from any other rule type. Other basic rules have no influence on it **save for document-level exceptions** (see the examples section).

> **Multiple rules matching a single request.** In case if multiple `$csp` rules match a single request, we will apply each of them.

##### **`csp` syntax**

`$csp` value syntax is similar to the Content Security Policy header syntax.

`$csp` value can be empty in the case of exception rules. See examples section for further information.

>Limitations

>1. Please note that there're a few characters forbidden in the `$csp` value: (`,`), (`$`)
>2. `csp` rules support limited list of modifiers: `domain`, `important`, `subdocument`
>3. Rules with `report-*` directives are considered invalid.

##### `csp` examples

* `||example.org^$csp=frame-src 'none'` — this rule blocks all frames on example.org and its subdomains.
* `@@||example.org/page/*$csp=frame-src 'none'` — disables all rules with the `$csp` modifier exactly matching `frame-src 'none'` on all the pages matching the rule pattern. For instance, the rule above.
* `@@||example.org/page/*$csp` — disables all the `$csp` rules on all the pages matching the rule pattern.
* `||example.org^$csp=script-src 'self' 'unsafe-eval' http: https:` — disables inline scripts on all the pages matching the rule pattern.
* `@@||example.org^$document` or `@@||example.org^$urlblock` — disables all the `$csp` rules on all the pages matching the rule pattern.

<a id="cookie-modifier"></a>
#### **`cookie`**

The `$cookie` modifier completely changes rule behavior. Instead of blocking a request, this modifier makes us suppress or modify the `Cookie` and `Set-Cookie` headers.

> **Multiple rules matching a single request.** In case if multiple `$cookie` rules match a single request, we will apply each of them one by one.

##### `$cookie` syntax
The rule syntax depends on whether we are going to block all cookies or to remove a single cookie. The rule behavior can be changed with `maxAge` and `sameSite` modifiers.

* `||example.org^$cookie=NAME;maxAge=3600;sameSite=lax` -- every time AdGuard encounters a cookie called `NAME` in a request to `example.org`, it will do the following:
  
  * Set its expiration date to current time plus `3600` seconds
  * Makes the cookie use [Same-Site](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies#SameSite_cookies) "lax" strategy.
* `||example.org^$cookie` -- blocks ALL cookies set by `example.org`. This is an equivalent to setting `maxAge` to zero.
* `||example.org^$cookie=NAME` -- blocks a single cookie named `NAME`
* `||example.org^$cookie=/regular_expression/` -- blocks every cookie that matches a given regular expression

> **Important:** in the case of a regular expression matching, two characters must be escaped: comma (`,`) and (`$`). Use (`\`) for it. For example, escaped comma looks like this: `\,`.

`$cookie` rules are not affected by regular exception rules (`@@`) unless it's a `$document` exception. In order to disable a `$cookie` rule, the exception rule should also have a `$cookie` modifier. Here's how it works:

* `@@||example.org^$cookie` -- unblocks all cookies set by `example.org`
* `@@||example.org^$cookie=NAME` -- unblocks a single cookie named `NAME`
* `@@||example.org^$cookie=/regular_expression/` -- unblocks every cookie matching a given regular expression

> **Limitations:** `$cookie` rules support a limited list of modifiers: `domain`, `~domain`, `important`, `third-party`, `~third-party`.

##### Real-life examples
* `$cookie=__cfduid` -- blocks CloudFlare cookie everywhere
* `$cookie=/__utm[a-z]/` -- blocks Google Analytics cookies everywhere
* `||facebook.com^$third-party,cookie=c_user` -- prevents Facebook from tracking you even if you are logged in

> **不同版本 AdGuard 的兼容性** This type of rules is not supported by AdGuard for iOS and Safari.

<a id="network-modifier"></a>
#### **`network`**

This is basically a Firewall-kind of rules allowing to fully block or unblock access to a specified remote address.

1. `$network` rules match **IP addresses only**! You cannot use it to block or unblock access to a domain.
2. To match an IPv6 address, you have to use the collapsed syntax, e.g. use `[2001:4860:4860::8888]$network` instead of `[2001:4860:4860:0:0:0:0:8888]$network`.
3. A allowlist `$network` rule makes AdGuard bypass data to the matching endpoint, e.g. there will be no further filtering at all.

##### `network` examples

* `174.129.166.49:3478^$network` - blocks access to `174.129.166.49:3478` (but not to `174.129.166.49:34788`).
* `[2001:4860:4860::8888]:443^$network` - blocks access to `[2001:4860:4860::8888]:443`.
* `174.129.166.49$network` - blocks access to `174.129.166.49:*`.
* `@@174.129.166.49$network` - makes AdGuard bypass data to the endpoint. No other rules will be applied.

> **不同版本 AdGuard 的兼容性** Only AdGuard for Windows, Mac, Android are technically capable of using this type of rules.

<a id="app-modifier"></a>
#### **`app`**

This modifier lets you narrow the rule coverage down to a specific application (or a list of applications). This might be not too important on Windows and Mac, but this is very important on Mobile where some of the filtering rules must be application-specific.

* Android - use the apps' package names (i.e. `org.example.app`).
* Windows - use the process name (i.e. `chrome.exe`).
* Mac - use the bundle ID or the process name (i.e. `com.google.Chrome`).

>For Mac, you can find out the bundle ID or the process name of the app by viewing the respective request details in the Filtering log.

##### `app` examples 

* `||baddomain.com^$app=org.example.app` — a rule to block requests that match the specified mask, and are sent from the `org.example.app` Android app.
* `||baddomain.com^$app=org.example.app1|org.example.app2` — the same rule, but it works for both `org.example.app1` and `org.example.app2` apps.

If you want the rule not to be applied to certain apps, start the app name with `~` sign.

##### `app` and `~` examples

* `||baddomain.com^$app=~org.example.app` — a rule to block requests that match the specified mask, and are sent from any app save for the `org.example.app`.
* `||baddomain.com^$app=~org.example.app1|~org.example.app2` — same as above, but now two apps are excluded: `org.example.app1` and `org.example.app2`.

> **不同版本 AdGuard 的兼容性** Only AdGuard for Windows, Mac, Android are technically capable of using this type of rules.

<a id="redirect-modifier"></a>

#### **`redirect`**

AdGuard is able to redirect web requests to a local "resource".

##### `redirect` syntax

AdGuard uses the same filtering rules syntax as uBlock Origin. Also, it is compatible with ABP `$rewrite=abp-resource` modifier.

`$redirect` is a modifier for the [basic filtering rules](https://kb.adguard.com/en/general/how-to-create-your-own-ad-filters#basic-rules) so rules with this modifier support all other basic modifiers like `$domain`, `$third-party`, `$script`, etc.

> The value of the `$redirect` modifier must be the name of the resource that will be used for redirection.

> `$redirect` rules' priority is higher than the regular basic blocking rules' priority. This means that if there's a basic blocking rule (even with `$important` modifier), `$redirect` rule will prevail over it. If there's an allowlist (`@@`) rule matching the same URL, it will disable redirecting as well (unless the `$redirect` rule is also marked as `$important`).

##### Disabling `$redirect` rules

* `@@||example.org^$redirect` will disable all `$redirect` rules for URLs that match `||example.org^`.
* `@@||example.org^$redirect=redirectName` will disable all rules with `$redirect=redirectName` for any request that matches `||example.org^`.

##### `redirect` examples

```
||example.org/script.js$script,redirect=noopjs
```

This rule redirects all requests to `example.org/script.js` to the resource named `noopjs`.

```
||example.org/test.mp4$media,redirect=noopmp4-1s
```

This rule redirects all requests to `example.org/test.mp4` to the resource named `noopmp4-1s`.

> More information on scriptlets, redirects, and their usage is available in [this GitHub section](https://github.com/AdguardTeam/Scriptlets#redirect-resources).

> **不同版本 AdGuard 的兼容性** This type of rules is not supported by AdGuard for iOS and Safari.

<a id="redirect-rule-modifier"></a>
#### **`redirect-rule`**
This is basically an alias to `$redirect` since it has the same "redirection" values and the logic is almost similar. The difference is that `$redirect-rule` is applied only in the case when the target request is blocked by a different basic rule.

> Negating `$redirect-rule` works exactly the same way as for regular `$redirect` rules. Even more than that, `@@||example.org^$redirect` will negate both `$redirect` and `$redirect-rule` rules.

Examples:

```
||example.org/script.js
||example.org^$redirect-rule=noopjs
```

In this case, only requests to `example.org/script.js` will be "redirected". All other requests to `example.org` will be kept intact.

<a id="denyallow-modifier"></a>

#### **`denyallow`**

`denyallow` modifier allows to avoid creating additional rules when it is needed to disable a certain rule for a specific domain(s). `denyallow` matches only target domains and not referrer domains.

Adding this modifier to a rule is equivalent to excluding the domains by the rule's matching pattern or to adding the corresponding exclusion rules. To add multiple domains to one rule, use the `|`  character as a separator.

Please note that rules with the `$denyallow` modifier have the following restrictions:
 
* the rule's matching pattern cannot target any specific domain(s) (e.g., it can't start with `||`)
* domains in the modifier's parameter cannot be negated (e.g. `$denyallow=~x.com`) or have a wildcard TLD (e.g. `$denyallow=x.*`)

The rules which violate these restrictions are considered invalid.

**Example:**

This rule:

```
*$script,domain=a.com|b.com,denyallow=x.com|y.com
```

is equivalent to this one:

```
/^(?!.*(x.com|y.com)).*$/$script,domain=a.com|b.com
```

or to these three:

```
*$script,domain=a.com|b.com
@@||x.com$script,domain=a.com|b.com
@@||y.com$script,domain=a.com|b.com
```

<a id="noop-modifier"></a>
#### **`noop`**

`noop` modifier does nothing and can be used solely to increase rules' readability. It consists of a sequence of underscore characters (`_`) of any length and can appear in a rule as many times as needed.

##### `noop` examples:

```
||example.com$_,removeparam=/^ss\\$/,_,image
||example.com$replace=/bad/good/,___,~third-party
```

> **不同版本 AdGuard 的兼容性** Available in **Developer builds only at this moment.**

<a id="removeheader-modifier"></a>
#### **`$removeheader`**

Rules with `$removeheader` modifier are intended to remove headers from HTTP requests and responses. The initial motivation for this rule type is to be able to get rid of the `Refresh` header which is often used to redirect users to an undesirable location. However, this is not the only case where this modifier can be useful.

Just like `$csp`, `$redirect`, `$removeparam`, and `$cookie`, this modifier exists independently, rules with it do not depend on the regular basic rules, i.e. regular exception or blocking rules will not affect it. By default, it only affects response headers. However, you can also change it to remove headers from HTTP requests as well.

##### Syntax

**Basic syntax**

* `||example.org^$removeheader=header-name` — removes a **response** header called `header-name`
* `||example.org^$removeheader=request:header-name` — removes a **request** header called `header-name`

Please note, that `$removeheader` is case-insensitive, but we suggest always using lower case.

**Negating `$removeheader`**

This type of rules works pretty much the same way it works with `$csp` and `$redirect` modifiers.

Use `@@` to negate `$removeheader`:

* `@@||example.org^$removeheader` — negates **all** `$removeheader` rules for URLs that match `||example.org^`.
* `@@||example.org^$removeheader=header` — negates the rule with `$removeheader=header` for any request matching `||example.org^`.
* `$removeheader` rules can also be disabled by `$document` and `$urlblock` exception rules. But basic exception rules without modifiers don't do that. For example, `@@||example.com^` will not disable `$removeheader=p` for requests to `example.com`, but `@@||example.com^$urlblock` will.

> **Multiple rules matching a single request**
> In case of multiple `$removeheader` rules matching a single request, we will apply each of them one by one.

##### Restrictions

1. Please note that this type of rules can be used **only in trusted filters**. This category includes your own User rules and all the filters created by AdGuard Team.
2. In order to avoid compromising the security `$removeheader` cannot remove headers from the list below:

* `access-control-allow-origin`
* `access-control-allow-credentials`
* `access-control-allow-headers`
* `access-control-allow-methods`
* `access-control-expose-headers`
* `access-control-max-age`
* `access-control-request-headers`
* `access-control-request-method`
* `origin`
* `timing-allow-origin`
* `allow`
* `cross-origin-embedder-policy`
* `cross-origin-opener-policy`
* `cross-origin-resource-policy`
* `content-security-policy`
* `content-security-policy-report-only`
* `expect-ct`
* `feature-policy`
* `origin-isolation`
* `strict-transport-security`
* `upgrade-insecure-requests`
* `x-content-type-options`
* `x-download-options`
* `x-frame-options`
* `x-permitted-cross-domain-policies`
* `x-powered-by`
* `x-xss-protection`
* `public-key-pins`
* `public-key-pins-report-only`
* `sec-websocket-key`
* `sec-websocket-extensions`
* `sec-websocket-accept`
* `sec-websocket-protocol`
* `sec-websocket-version`
* `p3p`
* `sec-fetch-mode`
* `sec-fetch-dest`
* `sec-fetch-site`
* `sec-fetch-user`
* `referrer-policy`
* `content-type`
* `content-length`
* `accept`
* `accept-encoding`
* `host`
* `connection`
* `transfer-encoding`
* `upgrade`

3. `$removeheader` rules are not compatible with any other modifiers except `$domain`, `$third-party`, `$app`, `$important`, `$match-case`, and content type modifiers (e.g. `$script`, `$stylesheet`, etc). The rules which have any other modifiers are considered invalid and will be discarded.

##### Examples

* `||example.org^$removeheader=refresh` — removes `Refresh` header from all HTTP responses returned by `example.org` and its subdomains.
* `||example.org^$removeheader=request:x-client-data` — removes `X-Client-Data` header from all HTTP requests.
* This block of rules removes `Refresh` and `Location` headers from all HTTP responses returned by `example.org` save for requests to `example.org/path/*`, for which no headers will be removed:

    ```
    ||example.org^$removeheader=refresh
    ||example.org^$removeheader=location
    @@||example.org/path/$removeheader
    ```

> **不同版本 AdGuard 的兼容性** Available in **Developer builds only at this moment.**

<a id="non-basic-rules"></a>
# Non-basic rules

However, the capabilities of the basic rules may not be sufficient to block ads. Sometimes you need to hide an element or change part of the HTML code of a web page without breaking anything. The rules described in this section are created specifically for this purpose.

<a id="cosmetic-rules"></a>
## Cosmetic rules

> Work with non-basic rules requires the basic knowledge of HTML and CSS. So, if you want to learn how to make such rules, we recommend to get acquainted with [this documentation](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Getting_started/What_is_CSS).

<a id="cosmetic-elemhide-rules"></a>
### Element hiding rules

Element hiding rules are used to hide the elements of web pages. It is similar to applying `{ display: none; }` style to selected element.

> Note that element hiding rules may operate differently [depending on the platform](https://kb.adguard.com/en/general/how-to-create-your-own-ad-filters#cosmetic-rules-priority).

<a id="elemhide-syntax"></a>
#### Syntax

```
   rule = [domains] "##" selector
domains = [domain0, domain1[, ...[, domainN]]]
```

* **`selector`** — [CSS selector](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Getting_Started/Selectors), defines the elements to be hidden.
* **`domains`** — domain restriction for the rule.

If you want to limit the rule application area to certain domains, just enter them separated with commas. For 示例： `example.org,example.com##selector`.

> Please note that this rule will be also applied to all subdomains of `example.org` and `example.com`.

If you want the rule not to be applied to certain domains, start a domain name with `~` sign. For 示例：
`~example.org##selector`.

You can use both approaches in a single rule. For example, `example.org,~subdomain.example.org##domain` will work for `example.org` and all subdomains, **except `subdomain.example.org`**.

> **Please note,** element hiding rules are not dependent on each other. If there is a rule `example.org##selector` in the filter and you add `~example.org##selector` both rules will be applied independently.

<a id="elemhide-examples"></a>
#### Examples

* `example.com##div.textad` — hides a `div` with a class `textad` at `example.com` and all subdomains.
* `example.com,example.org###adblock` - hides an element with attribute `id` equals `adblock` at `example.com`, `example.org` and all subdomains.
* `~example.com##.textad` - hides an element with a class `textad` at all domains, except `example.com` and its subdomains.

> **Important!** Safari doesn't support both permitted and restricted domains. So the rules like `example.org,~foo.example.org##.textad` are invalid in AdGuard for Safari.

<a id="elemhide-exceptions"></a>
#### Exceptions

Exceptions can disable some rules on particular domains. They are very similar to usual exception rules, but instead of `##` you have to use `#@#`.

For example, there is a rule in filter:
```
##.textad
```

If you want to disable it for `example.com`, you can create an exception rule:
```
example.com#@#.textad
```

Sometimes, it may be necessary to disable all restriction rules. For example, to conduct tests. To do this, use the exclusion rule without specifying a domain. It will completely disable matching CSS elemhide rule on ALL domains:
```
#@#.textad
```

The same can be achieved by adding this rule:
```
*#@#.textad
```

We recommend to use this kind of exceptions only if it is not possible to change the hiding rule itself. In other cases it is better to change the original rule, using domain restrictions.

<a id="cosmetic-css-rules"></a>
### CSS rules

Sometimes, simple hiding of an element is not enough to deal with advertising. For example, blocking an advertising element can just break the page layout. In this case AdGuard can use rules that are much more flexible than hiding rules. With this rules you can basically add any CSS styles to the page.

> **Restrictions.** Styles that lead to loading any resource are forbidden. Basically, it means that you cannot use any `<url>` type of value in the style.

> **不同版本 AdGuard 的兼容性** CSS rules are not supported by AdGuard for iOS.

> Note that CSS rules may operate differently [depending on the platform](https://kb.adguard.com/en/general/how-to-create-your-own-ad-filters#cosmetic-rules-priority).

<a id="cosmetic-css-rules-syntax"></a>
#### Syntax

```
   rule = [domains] "#$#" selector "{" style "}"
domains = [domain0, domain1[, ...[, domainN]]]
```

* **`selector`** — [CSS selector](https://developer.mozilla.org/en-US/docs/Web/Guide/CSS/Getting_Started/Selectors), defines the elements we want to apply the style to.
* **`domains`** — domain restriction for the rule. Same principles as in [element hiding rules](#elemhide-syntax).
* **`style`** — CSS style, that we want to apply to selected elements.

<a id="cosmetic-css-rules-examples"></a>
#### Examples

```
example.com#$#body { background-color: #333!important; }
```

This rule will apply a style `background-color: #333!important;` to the `body` element at `example.com` and all subdomains.

<a id="cosmetic-css-rules-exceptions"></a>
#### Exceptions

Just like with element hiding, there is a type of rules that disable the selected CSS style rule for particular domains.
Exception rules syntax is almost the same, you just have to change `#$#` to `#@$#`.

For example, there is a rule in filter:
```
#$#.textad { visibility: hidden; }
```

If you want to disable it for `example.com`, you can create an exception rule:
```
example.com#@$#.textad { visibility: hidden; }
```

We recommend to use this kind of exceptions only if it is not possible to change the CSS rule itself. In other cases it is better to change the original rule, using domain restrictions.

<a id="extended-css-selectors"></a>
### Extended CSS selectors

CSS 3.0 is not always enough to block ads. To solve this problem AdGuard extends CSS capabilities by adding support for the new pseudo-elements. To use extended CSS selectors we have developed a separate open source [module](https://github.com/AdguardTeam/ExtendedCss).

> **Application area.** Extended selectors can be used in any cosmetic rule, whether they are [element hiding rules](#cosmetic-elemhide-rules) or [CSS rules](#cosmetic-css-rules).

> **不同版本 AdGuard 的兼容性** Note that CSS rules are not supported by AdGuard for iOS.

#### Syntax

Regardless of the CSS pseudo-classes you are using in the rule, you can use special markers to make these rules use the "Extended CSS" engine. It is recommended to use these markers for all "extended CSS" cosmetic rules so that it was easier to find them.
The syntax for extended CSS rules:

* `#?#` — for element hiding (`#@?#` — for exceptions )
* `#$?#` — for CSS injection (`#@$?#` — for exceptions )

We **strongly recommend** using these markers any time when you use an extended CSS selector.

#### Examples

* `example.org#?#div:has(> a[target="_blank"][rel="nofollow"])` — this rule blocks all `div` elements containing a child node that has a link with the attributes `[target="_blank"][rel="nofollow"]`. The rule applies only to `example.org` and its subdomains.
* `example.com#$?#h3:contains(cookies) { display: none!important; }` — this rule sets the style `display: none!important` to all `h3` elements that contain the word `cookies`. The rule applies only to `example.com` and all its subdomains.
* `example.net#?#.banner:matches-css(width: 360px)` — this rule blocks all `.banner` elements with the style property `width: 360px`. The rule applies only to `example.net` and its subdomains.
* `example.net#@?#.banner:matches-css(width: 360px)` — this rule will disable the previous rule.

> Please note that now you can apply simple selectors using the ExtCss engine by using a rule like this:
> `#?#div`

> For more information on how to debug ExtendedCSS selectors, jump to [this section](#selectors-debugging-mode) of the artcile.


<a id="extended-css-has"></a>
##### Pseudo-class `:has()`

Draft CSS 4.0 specification describes [pseudo-class `:has`](https://drafts.csswg.org/selectors/#relational). Unfortunately, it is not yet supported by browsers.

**Syntax**
```
:has(selector)
```

Backward compatible syntax:
```
[-ext-has="selector"]
```

Supported synonyms for better compatibility: `:-abp-has`, `:if`.

Pseudo-class `:has()` selects the elements that includes the elements that fit to `selector`.

**Examples**

Selecting  all `div` elements, which contain an element with the `banner` class:

```html
<!-- HTML code -->
<div>Do not select this div</div>
<div>Select this div<span class="banner"></span></div>
```

Selector:
```
div:has(.banner)
```

Backward compatible syntax:
```
div[-ext-has=".banner"]
```

<a id="extended-css-if-not"></a>
##### Pseudo-class `:if-not()`

This pseudo-class is basically a shortcut for `:not(:has())`. It is supported by ExtendedCss for better compatibility with some filters subscriptions, but it is not recommended to use it in AdGuard filters. The rationale is that one day browsers will add `:has` native support, but it will never happen to this pseudo-class.

<a id="extended-css-contains"></a>
##### Pseudo-class `:contains()`

This pseudo-class principle is very simple: it allows to select the elements that contain specified text or which content matches a specified regular expression. Regex flags are supported. Please note that this pseudo-class uses `textContent` element property for matching (and not the `innerHTML`).

**Syntax**
```
// matching by plain text
:contains(text)

// matching by a regular expression
:contains(/regex/i)
```

Backward compatible syntax:
```
// matching by plain text
[-ext-contains="text"]

// matching by a regular expression
[-ext-contains="/regex/"]
```

> Supported synonyms for better compatibility: `:-abp-contains`, `:has-text`.

**Examples**

Selecting all `div` elements, which contain text `banner`:
```html
<!-- HTML code -->
<div>Do not select this div</div>
<div id="selected">Select this div (banner)</div>
<div>Do not select this div <div class="banner"></div></div>
```

Selector:
```
// matching by plain text
div:contains(banner)

// matching by a regular expression
div:contains(/this .* banner/)

// also with regex flags
div:contains(/this .* banner/gi)
```

Backward compatible syntax:
```
// matching by plain text
div[-ext-contains="banner"]

// matching by a regular expression
div[-ext-contains="/this .* banner/"]
```

> Please note that in this example only a `div` with `id=selected` will be selected, because the next element does not contain any text; `banner` is a part of code, not a text.

<a id="extended-css-matches-css"></a>
##### Pseudo-class `:matches-css()`

These pseudo-classes allow to select an element by its current style property. The work of this pseudo-class is based on using the [`window.getComputedStyle`](https://developer.mozilla.org/en-US/docs/Web/API/Window/getComputedStyle) function.

**Syntax**
```
/* element style matching */
selector:matches-css(property-name ":" pattern)

/* ::before pseudo-element style matching */
selector:matches-css-before(property-name ":" pattern)

/* ::after pseudo-element style matching */
selector:matches-css-after(property-name ":" pattern)
```

Backward compatible syntax:
```
selector[-ext-matches-css="property-name ":" pattern"]
selector[-ext-matches-css-after="property-name ":" pattern"]
selector[-ext-matches-css-before="property-name ":" pattern"]
```

- `property-name` — a name of CSS property to check the element for
- `pattern` —  a value pattern that is using the same simple wildcard matching as in the basic url filtering rules OR a regular expression. For this type of matching, AdGuard always does matching in a case insensitive manner. In the case of a regular expression, the pattern looks like `/regex/`.

> For non-regex patterns, `(`,`)`,`[`,`]` must be unescaped, because we require escaping them in the filtering rules.

> For regex patterns, `"` and `\` should be escaped, because we manually escape those in extended-css-selector.js.

**Examples**

Selecting all `div` elements which contain pseudo-class `::before` with specified content:
```html
<!-- HTML code -->
<style type="text/css">
    #to-be-blocked::before {
        content: "Block me"
    }
</style>
<div id="to-be-blocked" class="banner"></div>
<div id="not-to-be-blocked" class="banner"></div>
```

Selector:
```
// Simple matching
div.banner:matches-css-before(content: block me)

// Regular expressions
div.banner:matches-css-before(content: /block me/)
```

Backward compatible syntax:
```
// Simple matching
div.banner[-ext-matches-css-before="content: block me"]

// Regular expressions
div.banner[-ext-matches-css-before="content: /block me/"]
```

<a id="extended-css-matches-attr"></a>
##### Pseudo-class `:matches-attr()`

This pseudo-class allows to select an element by its attributes, especially if they are randomized.

**Syntax**
```
selector:matches-attr("name"[="value"])
```

- `name` — attribute name OR regular expression for attribute name
- `value` — optional, attribute value OR regular expression for attribute value

> For regex patterns, `"` and `\` should be escaped.

**Examples**

```html
<!-- HTML code -->
<div id="targer1" class="matches-attr" hsd4jkf-link="ssdgsg-banner_240x400"></div>

<div id="targer2" class="has matches-attr">
  <div data-sdfghlhw="adbanner"></div>
</div>

<div id="targer3-host" class="matches-attr has contains">
  <div id="not-targer3" wsdfg-unit012="click">
    <span>socials</span>
  </div>
  <div id="targer3" hrewq-unit094="click">
    <span>ads</span>
  </div>
</div>

<div id="targer4" class="matches-attr upward">
  <div >
    <inner-afhhw class="nyf5tx3" nt4f5be90delay="1000"></inner-afhhw>
  </div>
</div>
```

```
// for div#targer1
div:matches-attr("/-link/")

// for div#targer2
div:has(> div:matches-attr("/data-/"="adbanner"))

// for div#targer3
div:matches-attr("/-unit/"="/click/"):has(> span:contains(ads))

// for div#targer4
*[class]:matches-attr("/.{5,}delay$/"="/^[0-9]*$/"):upward(2)
```

<a id="extended-css-matches-property"></a>
##### Pseudo-class `:matches-property()`

This pseudo-class allows to select an element by its properties.

**Syntax**
```
selector:matches-property("name"[="value"])
```

- `name` — property name OR regular expression for property name
- `value` — optional, property value OR regular expression for property value

> For regex patterns, `"` and `\` should be escaped.

> `name` supports regexp for property in chain, e.g. `prop./^unit[\\d]{4}$/.type`

**Examples**

```javascript
divProperties = {
    id: 1,
    check: {
        track: true,
        unit_2ksdf1: true,
    },
    memoizedProps: {
        key: null,
        tag: 12,
        _owner: {
            effectTag: 1,
            src: 'ad.com',
        },
    },
};
```

```
// element with such properties can be matched by any of such rules:

div:matches-property("check.track")

div:matches-property("check./^unit_.{4,6}$/"))

div:matches-property("memoizedProps.key"="null")

div:matches-property("memoizedProps._owner.src"="/ad/")
```

<details>
  <summary><b>For filters maintainers</b></summary>

  To check properties of specific element, do:
  1. Select the element on the page.
  2. Go to Console tab and run `console.dir($0)`.
  
</details>

<a id="extended-css-xpath"></a>
##### Pseudo-class `:xpath()`

This pseudo-class allows to select an element by evaluating an XPath expression.

> **Can be placed only at the end of a selector, except for [pseudo-class `:remove()`](#remove-pseudos).**

The `:xpath()` pseudo-class is different from other pseudo-classes. Whereas all other operators are used to filter down a resultset of elements, the `:xpath()` operator can be used both to create a new resultset or filter down an existing one. For this reason, subject `selector` is optional. For example, an `:xpath()` operator could be used to create a new resultset consisting of all ancestor elements of a subject element, something not otherwise possible with either plain CSS selectors or other procedural operators.

**Syntax**
```
[selector]:xpath(expression)
```

- `selector`- optional, a plain CSS selector, or a Sizzle compatible selector
- `expression` — a valid XPath expression

**Examples**
```
// Filtering results from selector
div:xpath(//*[@class="test-xpath-class"])
div:has-text(/test-xpath-content/):xpath(../../..)

// Use xpath only to select elements
facebook.com##:xpath(//div[@id="stream_pagelet"]//div[starts-with(@id,"hyperfeed_story_id_")][.//h6//span/text()="People You May Know"])
```

<a id="extended-css-nth-ancestor"></a>
##### Pseudo-class `:nth-ancestor()`

This pseudo-class allows to lookup the nth ancestor relative to the currently selected node.

It is a low-overhead equivalent to `:xpath(..[/..]*)`.

> **Can be placed only at the end of a selector, except for [pseudo-class `:remove()`](#remove-pseudos).**

**Syntax**
```
selector:nth-ancestor(n)
```
- `selector` — a plain CSS selector, or a Sizzle compatible selector.
- `n` — positive number >= 1 and < 256, distance from the currently selected node.

**Examples**
```
div.test:nth-ancestor(4)

div:has-text(/test/):nth-ancestor(2)
```

<a id="extended-css-upward"></a>
##### Pseudo-class `:upward()`

This pseudo-class allows to lookup the ancestor relative to the currently selected node.

> **Can be placed only at the end of a selector, except for [pseudo-class `:remove()`](#remove-pseudos).**

**Syntax**
```
/* selector parameter */
subjectSelector:upward(targetSelector)

/* number parameter */
subjectSelector:upward(n)
```
- `subjectSelector` — a plain CSS selector, or a Sizzle compatible selector
- `targetSelector` — a valid plain CSS selector
- `n` — positive number >= 1 and < 256, distance from the currently selected node

**Examples**
```
div.child:upward(div[id])
div:contains(test):upward(div[class^="parent-wrapper-")

div.test:upward(4)
div:has-text(/test/):upward(2)
```

<a id="remove-pseudos"></a>
##### Pseudo-class `:remove()` and pseudo-property `remove`

Sometimes, it is necessary to remove a matching element instead of hiding it or applying custom styles. In order to do it, you can use pseudo-class `:remove()` as well as pseudo-property `remove`.

> **Pseudo-class `:remove()` can be placed only at the end of a selector.**

**Syntax**
```
! pseudo-class
selector:remove()

! pseudo-property
selector { remove: true; }
```
- `selector` — a plain CSS selector, or a Sizzle compatible selector

**Examples**
```
div.inner:remove()
div:has(> div[ad-attr]):remove()
div:xpath(../..):remove()

div:contains(target text) { remove: true; }
div[class]:has(> a:not([id])) { remove: true; }
```

> Please note that all style properties will be ignored if `:remove()` pseudo-class or `remove` pseudo-property is used.

<a id="cosmetic-rules-priority"></a>
### Cosmetic rules priority

The way **element hiding** and **CSS rules** are applied is platform-specific. 

**In AdGuard for Windows, Mac, and Android**, we use a stylesheet injected into the page. The priority of cosmetic rules is the same as any other websites' CSS stylesheet. But there is a limitation: [element hiding](https://kb.adguard.com/en/general/how-to-create-your-own-ad-filters#elemhide-rules) and [CSS](https://kb.adguard.com/en/general/how-to-create-your-own-ad-filters#cosmetic-css-rules) rules cannot override inline styles. In such cases, it's recommended to use extended selectors or HTML filtering.

**In AdGuard browser extensions**, the so called "user stylesheets" are used. They have higher priority than even the inline styles.

**Extended CSS selectors** use Javascript to work and basically add an inline style themselves, therefore they can override any style.

<a id="html-filtering-rules"></a>
## HTML filtering rules

In most cases, the basis and cosmetic rules are enough to filter ads. But sometimes it is necessary to change the HTML-code of the page itself before it is loaded. This is when you need filtering rules for HTML content. They allow to indicate the HTML elements to be cut out before the browser loads the page.

> **不同版本 AdGuard 的兼容性** Rules are supported by AdGuard for Windows, Mac, Android and by the AdGuard's Firefox add-on. This type of rules don't work in extensions for other browsers because they are unable to modify content on network level.

<a id="html-filtering-rules-syntax"></a>
### Syntax

```
      rule = [domains] "$$" tagName [attributes]
   domains = [domain0, domain1[, ...[, domainN]]]      
attributes = "[" name0 = value0 "]" "[" name1 = value2 "]" ... "[" nameN = valueN "]"
```

* **`tagName`** — name of the element in lower case, for example `div` or `script`.
* **`domains`** — domain restriction for the rule. Same principles as in [element hiding rules syntax](#elemhide-syntax).
* **`attributes`** — a list of attributes, that limit the elements selection. `name` - attribute name, `value` - substring, that is contained in attribute value.

<a id="html-filtering-rules-examples"></a>
### Example

**HTML code**
```html
<script data-src="/banner.js"></script>
```

**Rule**
```
example.org$$script[data-src="banner"]
```

This rule removes all `script` elements with the attribute `data-src` containing the substring `banner`. The rule applies only to `example.org` and all its subdomains.

<a id="html-filtering-rules-attributes"></a>
#### Special attributes

In addition to usual attribures, which value is every element checked for, there is a set of special attributes that change the way a rule works. Below there is a list of these attributes:

<a id="tag-content-attribute"></a>
##### `tag-content`

This is the most frequently used special attribute. It limits selection with those elements whose innerHTML code contains the specified substring.

> You should use `""` to escape `"`, for instance:
> `$$script[tag-content="alert(""this is ad"")"]`

For example, let's take a look at this HTML code:
```html
<script type="text/javascript">
    document.write('<div>banner text</div>" />');
</script>
```

Following rule will delete all `script` elements with a `banner` substring in their code:
```
$$script[tag-content="banner"]
```

> **Nested elements.** If we are dealing with multiple nested elements and they all fall within the same HTML filtering rule, they all are going to be deleted.

<a id="wildcard-attribute"></a>
##### `wildcard`

This special attribute works almost like `tag-content` and allows you to check the innerHTML code of the document. Rule will check if HTML code of the element fits to the [search pattern](https://en.wikipedia.org/wiki/Glob_(programming)).

> You should use `""` to escape `"`, for instance:
> `$$script[wildcard=""banner""]`

For 示例：
`$$script[wildcard="*banner*text*"]`

It will check, if the element's code contains two consecutive substrings `banner` and `text`.

<a id="max-length-attribute"></a>
##### `max-length`

Specifies the maximum length for content of HTML element. If this parameter is set and the content length exceeds the value - a rule does not apply to the element.

> **Default value.** If this parameter is not specified, the `max-length` is considered to be 8192.

For 示例：
```
$$div[tag-content="banner"][max-length="400"]
```
This rule will remove all the `div` elements, whose code contains the substring` banner` and the length of which does not exceed `400` characters.

<a id="min-length-attribute"></a>
##### `min-length`

Specifies the minimum length for content of HTML element. If this parameter is set and the content length is less than preset value - a rule does not apply to the element.

For 示例：
```
$$div[tag-content="banner"][min-length="400"]
```

This rule will remove all the `div` elements, whose code contains the substring` banner` and the length of which exceeds `400` characters.

<a id="html-filtering-rules-exceptions"></a>
#### Exceptions

Similar to hiding rules, there is a special type of rules that disable the selected HTML filtering rule for particular domains.
The syntax is the same, you just have to change `$$` to `$@$`.

For example, there is a rule in filter:
```
$$script[tag-content="banner"]
```

If you want to disable it for `example.com`, you can create an exception rule:
```
example.com$@$script[tag-content="banner"]
```

Sometimes, it may be necessary to disable all restriction rules. For example, to conduct tests. To do this, use the exclusion rule without specifying a domain.
```
$@$script[tag-content="banner"]
```

We recommend to use this kind of exceptions only if it is not possible to change the hiding rule itself. In other cases it is better to change the original rule, using domain restrictions.

<a id="javascript-rules"></a>
## Javascript rules

AdGuard supports a special type of rules that allows you to inject any javascript code to websites pages.

> **Restrictions.** Please note that this type of rules can be used **only in trusted filters**. This category includes your own **User filter** and all the filters created by AdGuard Team.

> **不同版本 AdGuard 的兼容性** Javascript rules aren't supported by AdGuard for iOS.

We **strongly recommend** using scriptlets instead of Javascript rules whenever possible. JS rules are supposed to help with debugging, but as a long-time solution a scriptlet rule should be used.

<a id="javascript-rules-syntax"></a>
### Syntax

```
rule = [domains]  "#%#" script
```

* **`domains`** — domain restriction for the rule. Same principles as in [element hiding rules](#elemhide-syntax).
* **`script`** — arbitrary javascript code **in one string**.

<a id="javascript-rules-examples"></a>
### Examples

* `example.org#%#window.__gaq = undefined;` — executes the code `window.__gaq = undefined;` on all pages at `example.org` and all subdomains.

<a id="javascript-rules-exceptions"></a>
### Exceptions

Similar to hiding rules, there is a special type of rules that disable the selected javascript rule for particular domains.
The syntax is the same, you just have to change `#%#` to `#@%#`.

For example, there is a rule in filter:
```
#%#window.__gaq = undefined;
```

If you want to disable it for `example.com`, you can create an exception rule:
```
example.com#@%#window.__gaq = undefined;
```

Sometimes, it may be necessary to disable all restriction rules. For example, to conduct tests. To do this, use the exclusion rule without specifying a domain.
```
#@%#window.__gaq = undefined;
```

We recommend to use this kind of exceptions only if it is not possible to change the hiding rule itself. In other cases it is better to change the original rule, using domain restrictions.

<a id="scriptlets"></a>
## Scriptlet rules

Scriptlet is a JavaScript function that provides extended capabilities for content blocking. These functions can be used in a declarative manner in AdGuard filtering rules.

> AdGuard supports a lot of different scriptlets. Please note that in order to achieve cross-blocker compatibility, we also support syntax of uBO and ABP.

> **不同版本 AdGuard 的兼容性** Scriptlet rules aren't supported by AdGuard for iOS.

<a id="scriptlets-syntax"></a>
### Syntax
```
rule = [domains]  "#%#//scriptlet(" scriptletName arguments ")"
```

`scriptletName` (mandatory) is a name of the scriptlet from AdGuard's scriptlets library
`arguments` (optional) is a list of String arguments (no other types of arguments are supported)

<a id="scriptlets-examples"></a>
### Examples

```
example.org#%#//scriptlet("abort-on-property-read", "alert")
```
This rule will be applied to example.org pages (and its subdomains) and will execute the "abort-on-property-read" scriptlet with the "alert" parameter.

More information about scriptlets can be found [on GitHub](https://github.com/AdguardTeam/Scriptlets#scriptlets).

> For more information on how to debug scriptlets, jump to [this section](#debug-scriptlets) of the artcile.


<a id="non-basic-rules-modifiers"></a>(#)
## Modifiers
Each rule can be modified using the modifiers described in the following paragraphs.

<a id="non-basic-rules-modifiers-syntax"></a>
### Syntax

```
rule = "[$" modifiers "]" [rule text]
modifiers = modifier0[, modifier1[, ...[, modifierN]]]
```

* `modifier` - set of the modifiers described below.
* `rule text` - a rule to be modified.

For 示例： `[$domain=example.com,app=test_app]##selector`.

In the modifiers values of the following characters must be escaped: `[`, `]`, `,`, and `\` (unless
it's used for the escaping). Use `\` to escape them. For example, an escaped bracket looks like
this: `\]`.

<a id="non-basic-rules-modifiers-app"></a>
### app

`app` lets you narrow the rule coverage down to a specific application (or a list of applications).

The modifier's behavior and syntax perfectly match the corresponding [$app modifier](https://kb.adguard.com/en/general/how-to-create-your-own-ad-filters#app) of basic rules.

`app` examples:
* `[$app=org.example.app]example.com##.textad` - hides a `div` with a class `textad` at `example.com` and all subdomains in requests sent from the `org.example.app` Android app.
* `[$app=~org.example.app1|~org.example.app2]example.com##.textad` - hides a `div` with a class `textad` at `example.com` and all subdomains in requests sent from any app except `org.example.app1` and `org.example.app2`.
* `[$app=com.apple.Safari]example.org#%#//scriptlet('prevent-setInterval', 'check', '!300')`. This rule will apply the corresponding scriptlet only in Safari browser on Mac.
* `[$app=org.example.app]#@#.textad` — disables all `##.textad` rules for all domains while using `org.example.app`.

> **不同版本 AdGuard 的兼容性** This type of rules is supported by AdGuard for Windows, Mac and Android. **Developer builds only at this moment.**

<a id="non-basic-rules-modifiers-domain"></a>
### domain

`domain` limits the rule application area to a list of domains (and their subdomains).
The modifier's behavior and syntax perfectly match the corresponding
[$domain modifier](https://kb.adguard.com/en/general/how-to-create-your-own-ad-filters#domain) of
basic rules.

`domain` examples:
* `[$domain=example.com]##.textad` — hides a `div` with a class `textad` at `example.com` and all subdomains.
* `[$domain=example.com|example.org]###adblock` - hides an element with attribute `id` equals `adblock` at `example.com`, `example.org` and all subdomains.
* `[$domain=~example.com]##.textad` - this rule hides `div` elements of the class `textad` for all domains, except `example.com` and its subdomains.

Please note that there are 2 ways to specify domain restrictions for non-basic rules:
    1) the "classic" way is to specify domains before rule mask and attributes: `example.com##.textad`
    2) the modifier approach is to specify domains via `domain` modifier: `[$domain=example.com]##.textad`

But rules with mixed style domains restriction are considered invalid. So, for example, the rule
`[$domain=example.org]example.com##.textad` will be rejected.

> **不同版本 AdGuard 的兼容性** This type of rules is supported by AdGuard for Windows, Mac and Android. **Developer builds only at this moment.**

<a id="non-basic-rules-modifiers-path"></a>
### path

`path` limits the rule application area to specific locations or pages on websites.

#### Syntax
```
path=pattern
```

`pattern` is a path mask to which the rule is restricted. Its syntax and behavior are pretty much the same as with the [pattern for basic rules](#basic-rules-syntax). You can also use [special characters](#basic-rules-special-characters), except for `||`, which does not make any sense in this case (see examples below).


> Please note that `path` modifier matches the query string as well.

> `path` modifier supports regular expressions in [the same way](#regexp-support) basic rules do.

`path` examples:
* `[$path=page.html]##.textad` - hides a `div` with a class `textad` at `/page.html` or `/page.html?<query>` or `/sub/page.html` or `/another_page.html`
* `[$path=/page.html]##.textad` - hides a `div` with a class `textad` at `/page.html` or `/page.html?<query>` or `/sub/page.html` of any domain but not at `/another_page.html`
* `[$path=|/page.html]##.textad` - hides a `div` with a class `textad` at `/page.html` or `/page.html?<query>` of any domain but not at `/sub/page.html`
* `[$path=/page.html|]##.textad` - hides a `div` with a class `textad` at `/page.html` or `/sub/page.html` of any domain but not at `/page.html?<query>` 
* `[$path=/page*.html]example.com##.textad` - hides a `div` with a class `textad` at `/page1.html` or `/page2.html` or any other path matching `/page<...>.html` of `example.com`
* `[$domain=example.com,path=/page.html]##.textad` - hides a `div` with a class `textad` at `page.html` of `example.com` and all subdomains but not at `another_page.html`
* `[$path=/\\/(sub1|sub2)\\/page\\.html/]##.textad` - hides a `div` with a class `textad` at both `/sub1/page.html` and `/sub2/page.html` of any domain (please, note the [escaped special characters](#non-basic-rules-modifiers-syntax))

> **Compatibility with different versions of AdGuard.** Rules with `path` modifier are supported by AdGuard for Windows, Mac, Android, and AdGuard browser extensions for Chrome, Firefox, Edge. **Developer builds only at this moment.**

<a id="for_maintainers"></a>
## Information for filters maintainers 

If you maintain a third-party filter that is known to AdGuard, you might be interested in the information presented in this section. Please note that hints will be applied to registered filters only. The filter is considered to be registered and known by AdGuard, if it is present in the [known filters index](https://filters.adtidy.org/extension/chromium/filters.json).  If you want your filter to be registered, please file an issue to [AdguardFilters repo](https://github.com/AdguardTeam/AdguardFilters).

<a id="pre_processor"></a>
### Pre-processor directives

We provide pre-processor directives that can be used by filters maintainers to improve compatibility with different ad blockers and provide:
* [including a file](#include-directive)
* [applying rules conditionally by ad blocker type](#conditions-directive)
* [content blocker specifying for rules applying in Safari](#safari-affinity-directive)

> Please note that any mistake in a pre-processor directive will lead to AdGuard failing the filter update in the same way as if the filter URL was unavailable.

> Pre-processor directives can be used in the User Rules or in the custom filters.

<a id="include-directive"></a>
#### Including a file

The `!#include` directive allows to include contents of a specified file into the filter. It supports only files from the same origin to make sure that the filter maintainer is in control of the specified file. The included file can also contain pre-processor directives (even other `!#include` directives). Ad blockers should consider the case of recursive `!#include` and implement a protection mechanism.

**Syntax**
```
!#include file_path
```
- `file_path` — same origin absolute or relative file path to be included

> The files must originate from the same domain but may be located in a different folder.

> If included file is not found or unavailable, the whole filter update should fail.

> Same-origin limitation should be disabled for local custom filters.

**Examples**

Filter URL: `https://example.org/path/filter.txt`
```
! Valid (same origin):
!#include https://example.org/path/includedfile.txt
!
! Valid (relative path):
!#include /includedfile.txt
!#include ../path2/includedfile.txt
!
! Invalid (another origin):
!#include https://domain.com/path/includedfile.txt
```

<a id="conditions-directive"></a>
#### Conditions

Filters maintainers can use conditions to supply different rules depending on the ad blocker type. When an adblocker encounters an `!#if` directive, followed eventually by an `!#endif` directive, it will compile the code inside of the directives block only if the specified condition is true. Condition supports all the basic logical operators.

> A conditional directive beginning with an `!#if` directive must explicitly be terminated with an `!#endif` directive.

> Whitespaces matter. `!#if` is a valid directive, while `!# if` is not.

**Syntax**
```
!#if (conditions)
rules_list
!#endif
```
- `!#if (conditions)` — start of the block
- `conditions` — just like in some popular programming languages, pre-processor conditions are based on constants declared by ad blockers. Ad blocker authors define on their own what exact constants do they declare. Possible values:
  - `adguard` — declared always; shows maintainers that this is one of AdGuard products; should be enough in 95% of cases
  - product-specific constants for cases when you need a rule to work (or not work — then `!` should be used before constant) in a specific product only:
    - `adguard_app_windows` — AdGuard for Windows
    - `adguard_app_mac` — AdGuard for Mac
    - `adguard_app_android` — AdGuard for Android
    - `adguard_app_ios` — AdGuard for iOS
    - `adguard_ext_safari` — AdGuard for Safari
    - `adguard_ext_chromium` — AdGuard Browser extension for Chrome (and chromium-based browsers, e.g. new Microsoft Edge)
    - `adguard_ext_firefox` — AdGuard Browser extension for Firefox
    - `adguard_ext_edge` — AdGuard Browser extension for Edge Legacy
    - `adguard_ext_opera` — AdGuard Browser extension for Opera
    - `adguard_ext_android_cb` — AdGuard Content Blocker for mobile Samsung and Yandex browsers
    - `ext_ublock` — special case; this one is declared when a uBlock version of a filter is compiled by the [FiltersRegistry](https://github.com/AdguardTeam/FiltersRegistry)
- `rules_list` — list of rules
- `!#endif` — end of the block

**Examples**
```
! for all AdGuard propucts except AdGuard for Safari
!#if (adguard && !adguard_ext_safari)
||example.org^$third-party
domain.com##div.ad
!#endif
```

```
! directives even can be combined
!#if (adguard_app_android)
!#include /androidspecific.txt
!#endif
```

<a id="safari-affinity-directive"></a>
#### Safari affinity

Safari is notoriously known for its harsh 50k max limit for filtering rules in content blockers. But in AdGuard for Safari and AdGuard for iOS max rule count is raised to 300k by splitting them into several content blockers. Generally, several filters categories are more or less independent, so there is such content blockers with such categories included:
- AdGuard General — Ad Blocking, Language-specific
- AdGuard Privacy — Privacy
- AdGuard Social — Social Widgets, Annoyances
- AdGuard Security — Security
- AdGuard Other — Other
- AdGuard Custom — Custom

> `User rules` and `Allowlist` are added to every content blocker.

The main issue with using multiple content blockers is that rules inside these content blockers cannot influence each other. This may lead to different unexpected issues. So filters maintainers may use `!#safari_cb_affinity` to define Safari content blockers affinity for the rules inside of the directive block.

**Syntax**
```
!#safari_cb_affinity(content_blockers)
rules_list
!#safari_cb_affinity
```
- `!#safari_cb_affinity(content_blockers)` — start of the block
- `content_blockers` — comma-separated list of content blockers. Possible values:
  - `general` — AdGuard General content blocker
  - `privacy` — AdGuard Privacy content blocker
  - `social` — AdGuard Social content blocker
  - `security` — AdGuard Security content blocker
  - `other` — AdGuard Other content blocker
  - `custom` — AdGuard Custom content blocker
  - `all` — special keyword that means that the rules must be included into **all** content blockers
- `rules_list` — list of rules
- `!#safari_cb_affinity` — end of the block

**Examples**
```
! to unhide specific element which is hidden by AdGuard Base filter:
!#safari_cb_affinity(general)
example.org#@#.adBanner
!#safari_cb_affinity
```
```
! to allowlist basic rule from AdGuard Tracking Protection filter filter:
!#safari_cb_affinity(privacy)
@@||example.org^
!#safari_cb_affinity
```

<a id="hints"></a>
### Hints
"Hint" is a special comment, instruction to the filters compiler used on the server side (see [FiltersRegistry](https://github.com/AdguardTeam/FiltersRegistry)).

<a id="hints_syntax"></a>
#### Syntax 
```
!+ HINT_NAME1(PARAMS) HINT_NAME2(PARAMS)
```
Note, that you can apply multiple hints. 

<a id="not_optimized"></a>
#### NOT_OPTIMIZED hint

For each filter, AdGuard compiles two versions: full and optimized. Optimized version is much more lightweight and does not contain rules which are not used at all or used rarely. 

Rules usage frequency comes from the collected [filter rules statistics](https://kb.adguard.com/en/general/filter-rules-statistics). But filters optimization is based on more than that — some filters have specific configuration. This is how it looks like for Base filter:

```
"filter": AdGuard Base filter,
"percent": 30,
"minPercent": 20,
"maxPercent": 40,
"strict": true
```
Where:

* **filter** — filter identifier
* **percent** — expected optimization percent `~= (rules count in optimized filter) / (rules count in original filter) * 100`
* **minPercent** — lower bound of `percent` value
* **maxPercent** — upper bound of `percent` value
* **strict** — if `percent < minPercent` OR `percent > maxPercent` and strict mode is on then filter compilation should fail, otherwise original rules must be used

>In other words, `percent` is the "compression level". For instance, for the Base filter it is configured to 40%. It means that optimization algorithm should strip 60% of rules.

Eventually, here are the two versions of the Base filter for AdGuard browser extension: 
- full: https://filters.adtidy.org/extension/chromium/filters/2.txt
- optimized: https://filters.adtidy.org/extension/chromium/filters/2_optimized.txt


**Important: If you want to add a rule which shouldn't be removed at optimization use the NOT_OPTIMIZED hint:**

```
!+ NOT_OPTIMIZED
||example.org^
```

**And this rule won't be optimized only for AdGuard for Android:**

```
!+ NOT_OPTIMIZED PLATFORM(android)
||example.org^
```

<a id="platform_not_platform"></a>
#### PLATFORM and NOT_PLATFORM hints

Specify which platforms can apply this rule. List of existing platforms:


* windows - 示例： English filter for Windows - [https://filters.adtidy.org/windows/filters/2.txt](https://filters.adtidy.org/windows/filters/2.txt)

* mac - 示例： English filter for Mac - [https://filters.adtidy.org/mac_v2/filters/2.txt](https://filters.adtidy.org/mac_v2/filters/2.txt)

* android - 示例： English filter for Android - [https://filters.adtidy.org/android/filters/2.txt](https://filters.adtidy.org/android/filters/2.txt)

* ios - 示例： English filter for iOS - [https://filters.adtidy.org/ios/filters/2.txt](https://filters.adtidy.org/ios/filters/2.txt)

* ext_chromium - 示例： AdGuard browser extension for Chrome - [https://filters.adtidy.org/extension/chromium/filters/2.txt](https://filters.adtidy.org/extension/chromium/filters/2.txt)

* ext_ff - 示例： AdGuard browser extension for Firefox - [https://filters.adtidy.org/extension/firefox/filters/2.txt](https://filters.adtidy.org/extension/firefox/filters/2.txt)

* ext_edge - 示例： AdGuard browser extension for Edge - [https://filters.adtidy.org/extension/edge/filters/2.txt](https://filters.adtidy.org/extension/edge/filters/2.txt)

* ext_opera - 示例： AdGuard browser extension for Opera - [https://filters.adtidy.org/extension/opera/filters/2.txt](https://filters.adtidy.org/extension/opera/filters/2.txt)

* ext_ublock - 示例： uBlock Origin - [https://filters.adtidy.org/extension/ublock/filters/2.txt](https://filters.adtidy.org/extension/ublock/filters/2.txt)

* ext_safari - 示例： AdGuard browser extension for Safari - [https://filters.adtidy.org/extension/safari/filters/2.txt](https://filters.adtidy.org/extension/safari/filters/2.txt)

* ext_android_cb - 示例： AdGuard Content Blocker - [https://filters.adtidy.org/extension/android-content-blocker/filters/2.txt](https://filters.adtidy.org/extension/android-content-blocker/filters/2.txt)

Examples:

This rule will be available for Windows, Mac, and Android only:

```
!+ PLATFORM(windows,mac,android)
||example.org^
```

This rule will be available for every platform except Safari extension, iOS, and Android content blocker:

```
!+ NOT_PLATFORM(ext_safari, ext_android_cb, ios)
||example.org^
```


<a id="how-to-debug"></a>
## How to debug filtering rules

It may be possible to create simple filtering rules "in your head", but for anything even slightly more complicated you'd need additional tools to debug and iterate them. There are tools to assist you with that. You can use DevTools in Chrome and its analogs in other browsers, but most AdGuard products provide another one: Filtering log.

<a id="debug-filtering-log"></a>
### Filtering log

Filtering log is an advanced tool that will be helpful mostly to filter developers. It lists all web requests that pass through AdGuard, gives you exhaustive information on each of them, offers multiple sorting options, and has other useful features.

Depending on which AdGuard product you're using, Filtering log can be located in different places. 

* In **AdGuard for Windows** you'll find it inside *Ad Blocker* tab or via the tray menu;
* In **AdGuard for Mac** it's under *Settings > Advanced > Filtering log*;
* In **AdGuard for Android** it's a separate item in the side menu, also filtering log for a specific app or website is accessible from the Assistant. 
* In **AdGuard browser extensions** it's accessible from the *Miscellaneous* settings tab or by right-clicking the extension icon. Only Chromium- and Firefox-based browsers show applied **element hiding rules** (including CSS, ExtCSS) and **JS rules and scriptlets** in their Filtering logs.

> In **AdGuard for iOS** and in **AdGuard for Safari** Filtering log does not exist because of the way content blockers are implemented in Safari. AdGuard doesn't see the web requests and therefore can't display them.

<a id="selectors-debugging-mode"></a>
### Selectors debugging mode

Sometimes, you might need to check the performance of a given selector or a stylesheet. In order to do it without interacting with javascript directly, you can use a special `debug` style property. When `ExtendedCss` meets this property, it enables the debug mode either for a single selector or for all selectors, depending on the `debug` value. Open the browser console while on a web page to see the timing statistics for selector(s) that were applied there. Debugging mode displays the following stats for each of the debugged selectors:

`array`: time that it took to apply the selector on the page, for each of the instances that it's been applied (in milliseconds)
`length`: total number of times that the selector has been applied on the page
`mean`: mean time that it took to apply the selector on the page
`stddev`: standard deviation
`squaredSum`: sum of squared deviations from the mean
`sum`: total time it took to apply the selector on the page across all instances


#### Examples

**Debugging a single selector**

When the value of the `debug` property is `true`, only information about this selector will be shown in the browser console.

```
#$?#.banner { display: none; debug: true; }
```

**Enabling global debug**

When the value of the `debug` property is `global`, the console will display information about all ExtendedCSS selectors that have matches on the current page, for all ExtendedCSS rules from any of the enabled filters.

```
#$?#.banner { display: none; debug: global; }
```

<a id="testing-extended-selectors"></a>
#### Testing extended selectors without AdGuard

If you don't have AdGuard installed, you can still test extended selectors, but you'll have to load ExtendedCSS to the current page first. To do so, copy and execute the following code in the browser console:

```
!function(E,x,t,C,s,s_){C=E.createElement(x),s=E.getElementsByTagName(x)[0],C.src=t,
C.onload=function(){alert('ExtCss loaded successfully')},s.parentNode.insertBefore(C,s)}
(document,'script','https://AdguardTeam.github.io/ExtendedCss/extended-css.min.js')
```

Alternatively, install an "ExtendedCssDebugger" userscript: https://github.com/AdguardTeam/Userscripts/blob/master/extendedCssDebugger/extended-css.debugger.user.js

You can now use the `ExtendedCss` constructor in the global scope, and its method `ExtendedCss.query` as `document.querySelectorAll`.
```
var selectorText = "div.block[-ext-has='.header:matches-css-after(content: Anzeige)']";

ExtendedCss.query(selectorText) // returns an array of Elements matching selectorText
```

<a id="debug-scriptlets"></a>
### Debugging scriptlets

If you're using AdGuard browser extension and want to debug a [scriptlet rule](https://kb.adguard.com/en/general/how-to-create-your-own-ad-filters#scriptlets), you can get additional information by simpy having the Filtering log opened. In that case, scriptlets will switch to debug mode and will write more information to the browser's console.

The following scriptlets may be especially useful for debug purposes:

[`debug-current-inline-script`](https://github.com/AdguardTeam/Scriptlets/blob/master/wiki/about-scriptlets.md#debug-current-inline-script)
[`debug-on-property-read`](https://github.com/AdguardTeam/Scriptlets/blob/master/wiki/about-scriptlets.md#debug-on-property-read)
[`debug-on-property-write`](https://github.com/AdguardTeam/Scriptlets/blob/master/wiki/about-scriptlets.md#abort-on-property-write)
[`log-addEventListener`](https://github.com/AdguardTeam/Scriptlets/blob/master/wiki/about-scriptlets.md#log-addEventListener)
[`log-eval`](https://github.com/AdguardTeam/Scriptlets/blob/master/wiki/about-scriptlets.md#log-eval)
[`log`](https://github.com/AdguardTeam/Scriptlets/blob/master/wiki/about-scriptlets.md#log)

The following scriptlets may be used for debug purposes when applied without any parameters:

[`requestAnimationFrame`](https://github.com/AdguardTeam/Scriptlets/blob/master/wiki/about-scriptlets.md#prevent-requestanimationframe)
[`prevent-setInterval`](https://github.com/AdguardTeam/Scriptlets/blob/master/wiki/about-scriptlets.md#prevent-setinterval)
[`prevent-setTimeout`](https://github.com/AdguardTeam/Scriptlets/blob/master/wiki/about-scriptlets.md#prevent-settimeout)


<a id="good-luck"></a>
## 祝您编写过滤器顺利！

我们希望您编写您自己的过滤器能一切顺利。

如果您需要如何正确编写过滤器的建议，我们的论坛有一个[特殊板块](https://forum.adguard.com/index.php?forums/69/)致力于编写您自己的过滤器。