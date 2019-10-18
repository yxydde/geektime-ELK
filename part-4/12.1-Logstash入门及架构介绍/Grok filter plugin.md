### Description

解析任意文本并将其结构化。

Grok是将非结构化日志数据解析为结构化和可查询内容的好方法。

该工具非常适合syslog日志，apache和其他Web服务器日志，mysql日志，以及通常用于人类而非计算机使用的任何日志格式。

Logstash默认附带大约120种模式。您可以在这里找到它们: https://github.com/logstash-plugins/logstash-patterns-core/tree/master/patterns. 您也可以添加自己的内容。 (See the `patterns_dir` setting)

如果您需要构建模式以匹配日志的帮助，您会发现 [http://grokdebug.herokuapp.com](http://grokdebug.herokuapp.com/) 和  http://grokconstructor.appspot.com/  应用程序非常有用！

#### Grok or Dissect? Or both?

The [`dissect`](http://www.elastic.co/guide/en/logstash/7.2/plugins-filters-dissect.html) 过滤器插件是另一种方式，使用定界符提取非结构化数据到对应字段。

Dissect与Grok的不同之处在于，它不使用正则表达式，并且速度更快。当可靠地重复数据时，解析效果很好。当文本的结构逐行变化时，Grok是更好的选择。

如果可靠地重复了某行的一部分，但没有重复整个行，则可以在混合用例中同时使用Dissect和Grok。Dissect过滤器可以解构重复的字段。Grok过滤器可以更正则表达式可预测性处理其余字段。

### Grok Basics

Grok 通过将文本模式组合 与 日志内容进行匹配。

Grok模式的语法是 `%{SYNTAX:SEMANTIC}`

 `SYNTAX` 是要匹配文本模式的名称，例如, `3.44` 匹配 `NUMBER` 模式 、 `55.3.244.1` 匹配 `IP` 模式. `SYNTAX` 就是匹配模式。

`SEMANTIC`是你给一段文字的标识相匹配。例如，`3.44`可能是事件的持续时间，因此您可以简单地将其称为`duration`。此外，字符串`55.3.244.1`可能会标识`client` 发出请求。

对于上面的示例，grok过滤器将如下所示： 

```ruby
%{NUMBER:duration} %{IP:client}
```

另外可以将数据类型转换添加到grok模式。默认情况下，所有语义都保存为字符串。如果要转换语义的数据类型，例如将字符串更改为整数，然后在目标数据类型后缀后缀。例如`%{NUMBER:num:int}`，将`num`语义从字符串转换为整数。目前，仅支持的转换是`int`和`float`。

例如：有了语法和语义的概念，我们可以从示例日志中提取有用的字段，例如虚构的http请求日志：

```ruby
    55.3.244.1 GET /index.html 15824 0.043
```

此模式可以是：

```ruby
    %{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}
```

一个真实的示例，让我们从文件中读取这些日志：

```ruby
    input {
      file {
        path => "/var/log/http.log"
      }
    }
    filter {
      grok {
        match => { "message" => "%{IP:client} %{WORD:method} %{URIPATHPARAM:request} %{NUMBER:bytes} %{NUMBER:duration}" }
      }
    }
```

在grok过滤器之后，该事件将在其中包含一些其他字段：

- `client: 55.3.244.1`
- `method: GET`
- `request: /index.html`
- `bytes: 15824`
- `duration: 0.043`

### Regular Expressions

Grok位于正则表达式之上，因此任何正则表达式在grok中也有效。正则表达式库是Oniguruma，您可以[在Oniguruma网站上](https://github.com/kkos/oniguruma/blob/master/doc/RE)看到完整的受支持的regexp语法。

### Custom Patterns

有时logstash没有您需要的模式。为此，你还有一其它选择。

首先，您可以对命名捕获使用Oniguruma语法，这将使您匹配一段文本并将其保存为字段：

```ruby
    (?<field_name>the pattern here)
```

例如，后缀日志的 `queue_id`是10或11个字符的十六进制值。我可以像这样轻松地捕获它：

```ruby
    (?<queue_id>[0-9A-F]{10,11})
```

或者，您可以创建一个自定义模式文件：

- 创建一个目录，`patterns`其中包含一个名为`extra` 的文件（文件名无关紧要，但为您自己有意义的命名）
- 在该文件中，将所需的模式写为模式名称，一个空格，然后输入该模式的regexp。

例如，执行上面的后缀队列ID示例：

```ruby
    # contents of ./patterns/postfix:
    POSTFIX_QUEUEID [0-9A-F]{10,11}
```

然后使用`patterns_dir`此插件中的设置告诉Logstash自定义模式目录在哪里。这是带有示例日志的完整示例：

```ruby
    Jan  1 06:25:43 mailserver14 postfix/cleanup[21403]: BEF25A72965: message-id=<20130101142543.5828399CCAF@mailserver14.example.com>
    filter {
      grok {
        patterns_dir => ["./patterns"]
        match => { "message" => "%{SYSLOGBASE} %{POSTFIX_QUEUEID:queue_id}: %{GREEDYDATA:syslog_message}" }
      }
    }
```

上面的内容将匹配并导致以下字段：

- `timestamp: Jan  1 06:25:43`
- `logsource: mailserver14`
- `program: postfix/cleanup`
- `pid: 21403`
- `queue_id: BEF25A72965`
- `syslog_message: message-id=<20130101142543.5828399CCAF@mailserver14.example.com>`

The `timestamp`, `logsource`, `program`, and `pid` fields come from the `SYSLOGBASE` pattern which itself is defined by other patterns.

Another option is to define patterns *inline* in the filter using `pattern_definitions`. This is mostly for convenience and allows user to define a pattern which can be used just in that filter. This newly defined patterns in `pattern_definitions` will not be available outside of that particular `grok` filter.

另一个选择是使用来在过滤器中*内联*定义模式`pattern_definitions`。这主要是为了方便起见，并允许用户定义仅可在该过滤器中使用的模式。这种新定义的模式`pattern_definitions 在`grok 过滤器外部将无法使用。