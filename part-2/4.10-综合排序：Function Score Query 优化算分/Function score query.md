## Function score query

`function_score`允许你修改一个查询检索到文档的评分。则此功能很有用，例如，如果评分函数在计算成本很高，并且足以在过滤后的文档集上计算评分。

要使用`function_score`，用户必须定义一个查询和一个或多个函数，这些函数为查询返回的每个文档计算一个新评分。

`function_score` 只能与以下一种功能一起使用：

```console
GET /_search
{
    "query": {
        "function_score": {
            "query": { "match_all": {} },
            "boost": "5",
            "random_score": {}, 
            "boost_mode":"multiply"
        }
    }
}
```



此外，可以组合几个评分函数。在这种情况下，只有当文档匹配给定的过滤条件时，才可以选择应用对应的评分函数：

```console
GET /_search
{
    "query": {
        "function_score": {
          "query": { "match_all": {} },
          "boost": "5", 
          "functions": [
              {
                  "filter": { "match": { "test": "bar" } },
                  "random_score": {}, 
                  "weight": 23
              },
              {
                  "filter": { "match": { "test": "cat" } },
                  "weight": 42
              }
          ],
          "max_boost": 42,
          "score_mode": "max",
          "boost_mode": "multiply",
          "min_score" : 42
        }
    }
}
```



每个函数的过滤查询所产生的分数并不重要。

如果没有给评分函数提供过滤器，则等同于指定 `"match_all": {}`

首先，通过定义的评分函数对每个文档评分。 `score_mode`参数指定如何组合计算出的分数：

| `multiply` | 分数相乘（默认）                 |
| ---------- | -------------------------------- |
| `sum`      | 分数相加                         |
| `avg`      | 分数是平均值                     |
| `first`    | 具有匹配过滤器的第一个函数被应用 |
| `max`      | 使用最高分                       |
| `min`      | 使用最低分数                     |

由于分数可以处于不同的范围（例如，衰变函数的分数在0到1之间，而`field_value_factor`可用是任意值），并且由于有时希望不同的评分函数对分数有不同的影响，因此可以使用用户定义的`weight`来调整每个函数的评分 。`weight`可以定义在`functions`列表（上面的例子）的每个function中,并且乘以相应的评分来计算分数。如果在没有声明其他任何函数的情况下给出`weight`，`weight`则充当函数且仅返回的`weight`值。

如果`score_mode`设置为`avg`，则分数将由`weight`平均值合并。例如，如果两个函数返回得分1和2以及它们各自的权重是3和4，那么他们的分数将被组合为 `(1*3+2*4)/(3+4)`而不是 `(1*3+2*4)/2`。

通过设置`max_boost`参数，可以将新分数限制为不超过特定限制。`max_boost`的默认值为FLT_MAX。

新计算的分数与查询分数合并。该参数`boost_mode`定义如何：

| `multiply` | 查询分数和功能分数相乘（默认）   |
| ---------- | -------------------------------- |
| `replace`  | 仅使用功能分数，查询分数将被忽略 |
| `sum`      | 查询分数和功能分数相加           |
| `avg`      | 平均                             |
| `max`      | 查询分数和功能分数的最大值       |
| `min`      | 查询分数和功能分数的最小值       |

默认情况下，修改分数不会更改匹配的文档。要排除不符合特定分数阈值的文档，`min_score`可以将参数设置为所需分数阈值。



为了使`min_score`工作，需要对查询返回的**所有**文档进行打分，然后一一过滤掉。

该`function_score`查询提供了几种类型的得分函数。

- [`script_score`](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-function-score-query.html#function-script-score)
- [`weight`](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-function-score-query.html#function-weight)
- [`random_score`](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-function-score-query.html#function-random)
- [`field_value_factor`](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-function-score-query.html#function-field-value-factor)
- [衰变函数](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-function-score-query.html#function-decay)：`gauss`，`linear`，`exp`

### 脚本分数

该`script_score`函数允许您包装另一个查询，并可以选择使用脚本表达式从文档中其他数字字段值派生的计算来自定义另一个查询的得分。这是一个简单的示例：

```console
GET /_search
{
    "query": {
        "function_score": {
            "query": {
                "match": { "message": "elasticsearch" }
            },
            "script_score" : {
                "script" : {
                  "source": "Math.log(2 + doc['likes'].value)"
                }
            }
        }
    }
}
```



在Elasticsearch中，所有文档分数均为正的32位浮点数。

如果`script_score`函数产生的分数更高，则将其转换为最接近的32位浮点数。

同样，分数必须为非负数。否则，Elasticsearch返回错误。

除了不同的脚本字段值和表达式之外， `_score`还可以使用script参数基于已包装的查询来检索分数。

脚本编译被缓存以加快执行速度。如果脚本具有需要考虑的参数，则最好重用相同的脚本并为其提供参数：

```console
GET /_search
{
    "query": {
        "function_score": {
            "query": {
                "match": { "message": "elasticsearch" }
            },
            "script_score" : {
                "script" : {
                    "params": {
                        "a": 5,
                        "b": 1.2
                    },
                    "source": "params.a / Math.pow(params.b, doc['likes'].value)"
                }
            }
        }
    }
}
```



请注意，与`custom_score`查询不同，查询的分数将与脚本评分的结果相乘。如果您想禁止这种情况，请设置`"boost_mode": "replace"`

### 权重

该`weight`分数可以让你乘上提供的分数 `weight`。有时可能需要这样做，因为在特定查询上设置的提升值会被标准化，而对于此得分函数则不会。数字值是float类型。

```js
"weight" : number
```

### 随机

的`random_score`产生被均匀地从0分布直到但不包括1.默认情况下，它采用了内部Lucene的文档ID作为随机源，这是非常有效的，但不幸的是不可再现的，因为文档可能通过合并重新编号分数。

如果您希望分数是可复制的，可以提供`seed` 和`field`。然后将基于该种子，所`field`考虑文档的最小值和根据索引名称和分片ID计算的盐来计算最终分数，以便具有相同值但存储在不同索引中的文档得到不同的结果分数。请注意，位于相同分片内且具有相同值的文档`field` 将获得相同的分数，因此通常希望使用对所有文档都具有唯一值的字段。一个好的默认选择是使用该 `_seq_no`字段，其唯一的缺点是，如果文档被更新，则分数会改变，因为更新操作也会更新该`_seq_no`字段的值。



可以在不设置字段的情况下设置种子，但是已弃用该方法，因为这需要在`_id`消耗大量内存的字段上加载字段数据。

```console
GET /_search
{
    "query": {
        "function_score": {
            "random_score": {
                "seed": 10,
                "field": "_seq_no"
            }
        }
    }
}
```



### 字段值因子

该`field_value_factor`功能允许您使用文档中的字段来影响得分。它类似于使用该`script_score`函数，但是它避免了脚本编写的开销。如果用于多值字段，则在计算中仅使用该字段的第一个值。

例如，假设您有一个用数字`likes` 字段索引的文档，并希望通过该字段影响文档的分数，则示例如下所示：

```console
GET /_search
{
    "query": {
        "function_score": {
            "field_value_factor": {
                "field": "likes",
                "factor": 1.2,
                "modifier": "sqrt",
                "missing": 1
            }
        }
    }
}
    
```



它将转化为以下得分公式：

```
sqrt(1.2 * doc['likes'].value)
```

该功能有许多选项`field_value_factor`：

| `field`    | 要从文档中提取的字段。                                       |
| ---------- | ------------------------------------------------------------ |
| `factor`   | 字段值乘以的可选因子，默认为`1`。                            |
| `modifier` | 修改适用于该字段的值，可以是一个：`none`，`log`， `log1p`，`log2p`，`ln`，`ln1p`，`ln2p`，`square`，`sqrt`，或`reciprocal`。默认为`none`。 |

| 修饰符       | 含义                                                         |
| ------------ | ------------------------------------------------------------ |
| `none`       | 不要对字段值应用任何乘数                                     |
| `log`        | 取字段值的[常用对数](https://en.wikipedia.org/wiki/Common_logarithm)。因为此函数将返回负值，并且如果将其用于0到1之间的值，则会导致错误，因此建议改用它`log1p`。 |
| `log1p`      | 将1加到字段值并取对数                                        |
| `log2p`      | 在字段值上加2并取公共对数                                    |
| `ln`         | 取字段值的[自然对数](https://en.wikipedia.org/wiki/Natural_logarithm)。因为此函数将返回负值，并且如果将其用于0到1之间的值，则会导致错误，因此建议改用它`ln1p`。 |
| `ln1p`       | 将1加到栏位值并取自然对数                                    |
| `ln2p`       | 将2加到字段值上并取自然对数                                  |
| `square`     | 对字段值求平方（乘以它本身）                                 |
| `sqrt`       | 取字段值的[平方根](https://en.wikipedia.org/wiki/Square_root) |
| `reciprocal` | [报答](https://en.wikipedia.org/wiki/Multiplicative_inverse)字段值，同`1/x`那里`x`是该字段的值 |

- **`missing`**

  如果文档没有该字段，则使用该值。就像从文档中读取一样，修饰符和因数仍然适用于它。



该`field_value_score`函数产生的分数必须为非负数，否则将引发错误。的`log`和`ln`如果在所使用的值0和1之间一定要限制的字段的值与一系列过滤器，以避免这种情况，或使用改性剂会产生负值`log1p`和 `ln1p`。



请记住，将log（）设为0或负数的平方根是非法操作，并且将引发异常。为避免这种情况，请务必使用范围过滤器限制该字段的值，或使用 `log1p`和`ln1p`。

### 衰减功能

衰减函数对文档进行评分，该函数的衰减取决于文档的数字字段值与用户给定原点的距离。这类似于范围查询，但具有平滑的边缘而不是框。

要在具有数字字段的查询上使用距离计分，用户必须为每个字段定义an `origin`和a `scale`。的`origin` 需要来定义“ 中心点 ”从其计算的距离，并且`scale`以限定衰减速率。衰减函数指定为

```js
"DECAY_FUNCTION": { 
    "FIELD_NAME": { 
          "origin": "11, 12",
          "scale": "2km",
          "offset": "0km",
          "decay": 0.33
    }
}  
```

在上面的示例中，字段为 [`geo_point`](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/geo-point.html)，可以按地理格式提供原点。在这种情况下`scale`，`offset`必须提供一个单位。如果您的字段是日期字段，则可以将`scale`和设置`offset`为天，周等。例：

```console
GET /_search
{
    "query": {
        "function_score": {
            "gauss": {
                "date": {
                      "origin": "2013-09-17", 
                      "scale": "10d",
                      "offset": "5d", 
                      "decay" : 0.5 
                }
            }
        }
    }
}
```



| `origin` | 用于计算距离的原点。对于数字字段，必须指定为数字；对于日期字段，必须指定为日期；对于地理字段，必须指定为地理点。地理位置和数字字段必填。对于日期字段，默认值为`now`。`now-1h`原点支持日期数学（例如）。 |
| -------- | ------------------------------------------------------------ |
| `scale`  | 所有类型均必需。定义到原点的距离+偏移，计算出的分数将等于该距离`decay`。对于地理字段：可以定义为数字+单位（1km，12m，...）。默认单位是米。对于日期字段：可以定义为数字+单位（“ 1h”，“ 10d”，…。）。默认单位是毫秒。对于数字字段：任何数字。 |
| `offset` | 如果`offset`定义了，衰减函数将仅计算距离大于定义的文档的衰减函数 `offset`。默认值为0。 |
| `decay`  | 该`decay`参数定义如何在处给定的距离处对文档进行评分`scale`。如果`decay`未定义，则远处的文档 `scale`得分为0.5。 |

在第一个示例中，您的文档可能代表酒店，并且包含地理位置字段。您要根据酒店距指定位置的距离来计算衰减函数。您可能不会立即看到为高斯功能选择哪种比例，但是您可以说：“在距所需位置2公里的距离处，分数应减少到三分之一。” 然后将自动调整参数“规模”，以确保得分功能为距离期望位置2公里的酒店计算出0.33的得分。

在第二个示例中，字段值在2013-09-12和2013-09-22之间的文档的权重为1.0，从该日期起15天的文档的权重为0.5。

#### 支持的衰减功能

所述`DECAY_FUNCTION`确定衰减的形状：

- **`gauss`**

  正常衰减，计算如下：![高斯型](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/images/Gaussian.png)其中![西格玛](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/images/sigma.png)被计算以确保得分取值`decay`在距离`scale`从`origin`+ -`offset`![sigma calc](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/images/sigma_calc.png)请参阅[正态衰减，关键字`gauss`](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-function-score-query.html#gauss-decay)以显示表示该`gauss`函数生成的曲线的图形。

- **`exp`**

  指数衰减，计算如下：![指数的](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/images/Exponential.png)其中再次参数![拉姆达](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/images/lambda.png)被计算，以确保该得分取值`decay`在距离`scale`从`origin`+ -`offset`![λ计算](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/images/lambda_calc.png)请参阅[指数衰减，关键字`exp`](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/query-dsl-function-score-query.html#exp-decay)以获取说明该`exp`函数生成的曲线的图。

- **`linear`**

  线性衰减，计算如下：![线性的](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/images/Linear.png)。其中再次参数`s`被计算，以确保该得分取值`decay`在距离`scale`从`origin`+ -`offset`![计算](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/images/s_calc.png)与正常和指数衰减相反，如果字段值超过用户给定标度值的两倍，则此功能实际上将分数设置为0。

对于单个函数，三个衰减函数及其参数可以像这样可视化（在此示例中，该字段称为“年龄”）：

![衰减2D](https://www.elastic.co/guide/en/elasticsearch/reference/7.5/images/decay_2d.png)

#### 多值字段

如果用于计算衰减的字段包含多个值，则默认情况下将选择最接近原点的值来确定距离。可以通过设置更改`multi_value_mode`。

| `min` | 距离是最小距离       |
| ----- | -------------------- |
| `max` | 距离是最大距离       |
| `avg` | 距离是平均距离       |
| `sum` | 距离是所有距离的总和 |

例：

```js
    "DECAY_FUNCTION": {
        "FIELD_NAME": {
              "origin": ...,
              "scale": ...
        },
        "multi_value_mode": "avg"
    }
```

### 详细示例

假设您正在寻找某个城镇的酒店。您的预算有限。另外，您希望酒店离市中心很近，因此酒店距离理想位置越远，您入住的可能性就越小。

您希望根据距市中心的距离以及价格来对与您的条件相匹配的查询结果（例如“酒店，南希，不吸烟者”）进行评分。

直观地讲，您想将市中心定义为起点，也许您愿意从酒店步行2公里到市中心。在这种情况下，您的位置字段的**来源**为镇中心，**范围**为〜2km。

如果您的预算低，您可能更喜欢便宜的东西而不是昂贵的东西。对于价格字段，**原点**将为0欧元，**小数**位数取决于您愿意支付的金额，例如20欧元。

在此示例中，字段可能被称为“价格”作为酒店价格，而“位置”则称为该酒店的坐标。

`price`在这种情况下的功能是

```js
"gauss": { 
    "price": {
          "origin": "0",
          "scale": "20"
    }
}
```

|      | 此衰减函数也可以是`linear`或`exp`。 |
| ---- | ----------------------------------- |
|      |                                     |

和为`location`：

```js
"gauss": { 
    "location": {
          "origin": "11, 12",
          "scale": "2km"
    }
}
```

假设您要将这两个函数乘以原始分数，则请求将如下所示：

```console
GET /_search
{
    "query": {
        "function_score": {
          "functions": [
            {
              "gauss": {
                "price": {
                  "origin": "0",
                  "scale": "20"
                }
              }
            },
            {
              "gauss": {
                "location": {
                  "origin": "11, 12",
                  "scale": "2km"
                }
              }
            }
          ],
          "query": {
            "match": {
              "properties": "balcony"
            }
          },
          "score_mode": "multiply"
        }
    }
}
```



接下来，我们展示三个可能衰减函数中的每一个的计算分数如何。

#### 正常衰减，关键字`gauss`

`gauss`在上面的示例中选择作为衰减函数时，乘数的轮廓和曲面图如下所示：

![cd0e18a6 e898 11e2 9b3c f0145078bd6f](https://f.cloud.github.com/assets/4320215/768157/cd0e18a6-e898-11e2-9b3c-f0145078bd6f.png)

![ec43c928 e898 11e2 8e0d f3c4519dbd89](https://f.cloud.github.com/assets/4320215/768160/ec43c928-e898-11e2-8e0d-f3c4519dbd89.png)

假设您的原始搜索结果匹配三家酒店：

- “后退小睡”
- “饮料n驱动器”
- “ BnB Bellevue”。

“ Drink n Drive”距离您定义的位置很近（近2公里），而且价格也不便宜（约13欧元），因此它的系数低至0.56。“ BnB Bellevue”和“ Backback Nap”都非常接近定义的位置，但是“ BnB Bellevue”更便宜，因此它的乘数为0.86，而“ Backpack Nap”的值为0.66。

#### 指数衰减，关键字`exp`

`exp`在上面的示例中选择作为衰减函数时，乘数的轮廓和曲面图如下所示：

![082975c0 e899 11e2 86f7 174c3a729d64](https://f.cloud.github.com/assets/4320215/768161/082975c0-e899-11e2-86f7-174c3a729d64.png)

![0b606884 e899 11e2 907b aefc77eefef6](https://f.cloud.github.com/assets/4320215/768162/0b606884-e899-11e2-907b-aefc77eefef6.png)

#### 线性衰减，关键字`linear`

`linear`在上面的示例中选择作为衰减函数时，乘数的轮廓和曲面图如下所示：

![1775b0ca e899 11e2 9f4a 776b406305c6](https://f.cloud.github.com/assets/4320215/768164/1775b0ca-e899-11e2-9f4a-776b406305c6.png)

![19d8b1aa e899 11e2 91bc 6b0553e8d722](https://f.cloud.github.com/assets/4320215/768165/19d8b1aa-e899-11e2-91bc-6b0553e8d722.png)

### 衰变功能支持的字段

仅支持数字，日期和地理位置字段。

### 如果缺少字段怎么办？

如果文档中缺少数字字段，该函数将返回1