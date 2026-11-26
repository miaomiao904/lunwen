# B数据集数据字典 — Schema Freeze 1.0

本字典与 `schema_manifest.json` 共同构成冻结基准。所有表格字段顺序、数据类型和枚举均固定；允许追加记录，不允许原位改列。人工Gold、模型预测、证据支持标签和跨阶段变化各自独立保存。

## 统一语义与异常处理

- `gold_state` / `predicted_state`：`known`（当前文档明确完整值）、`partial`（只提供部分信息）、`unknown`（明示未知/待查）、`missing`（未提及）、`conflict`（同篇无法消解矛盾）、`not_applicable`（确实不适用）。Gold未标注时状态必须为空，不能默认写`missing`。否定用值、证据和限定语表达，不设`negated`状态。
- 所有人数均为非负整数、单位为人；仅原文明示的0记为0。死亡失踪合计不等于死亡人数；一般受伤人数不等于重伤人数；伤亡总数必须由正文直接报告，不能由分项相加。
- 时间只规范原文明确的精度；地点不补全；财产损失保留币种/单位/估算限定，只有无歧义时才折算人民币元。评价需同时考虑精度、单位、统计口径和限定语。
- Gold证据只引当前`doc_id` TXT。偏移是Python Unicode字符串字符偏移、0基左闭右开；证据、起止位置必须互相吻合。证据定位成功不等于语义支持。
- 无TXT的文档仍保留来源和Gold待标行，但不能进入正式评价。错误JSON只记运行错误，不建立伪造字段预测。置信度自报值[0,1]不等同校准概率。
- RQ6下游字段不适用或信息不足时必须保留不适用/无法判定，不强制分类。

## 业务字段注册表

| field_key | 中文名 | 类型 | 定义与规则 | 研究模块 |
|---|---|---|---|---|
| `event_type` | 事件类型 | string | 事故或事件所属的具体类型，限于当前文档明确表达。 按原文提取事故类型；不把原因、后果或事故等级并入事件类型。 未提及为 missing；明确尚未确定为 unknown；仅给出宽泛类别为 partial。 保留原文术语；不依据常识补充更细类别。 引用当前文档中直接命名事件类型的连续原文。 按标准化后的类型标签精确比较；需同义归并时由人工复核。 | RQ1;RQ2;RQ3;RQ4;RQ5 |
| `event_time` | 发生时间 | date/string | 事故实际发生时间，不是通报发布时间或调查结论时间。 只提取正文明确给出的发生日期/时间；不能用标题、发布日期或外部信息补年份、时区或精度。 完全未提及为 missing；明确待核实为 unknown；只给日期或近似时间等不完整信息为 partial。 显式年月日可规范为 YYYY-MM-DD；完整时间可保留原精度并使用 ISO 8601。未明确的组成部分不推定，原时区缺省时不添加时区。 引用含发生时间的连续原文；时间精度限定词须一并保留。 在相同明确精度上比较；不得因时间粒度不同将近似时间判为精确相等。 | RQ1;RQ2;RQ3;RQ4;RQ5;RQ6 |
| `location` | 发生地点 | string | 当前文档明确报告的事故发生地点及其原有空间粒度。 逐级保留原文地点名称；不通过常识、地址库或其他文档补全。 未提及为 missing；地点明确尚待核实为 unknown；只报告到较粗层级为 partial。 保留行政区、道路、场所等原文层级；不擅自规范或补齐地址。 引用能支持地点层级的连续原文。 在同一空间粒度上比较；不同粒度或简称需人工判定。 | RQ1;RQ2;RQ3;RQ4;RQ5;RQ6 |
| `subject` | 涉及主体 | string/list | 文档明确涉及的企业、人员、机构及其明示角色。 提取文档明确指出的主体；只有原文明确角色时才记录该角色，不从关系或责任结论推断。 未提及为 missing；主体身份待查为 unknown；只知道主体类别或不完整名称为 partial。 保留原名和角色边界；多主体可用JSON字符串数组表示。 引用明确点名主体或角色的连续原文。 主体名称与角色分别核对；简称、主体合并或责任推断需人工复核。 | RQ1;RQ2;RQ3;RQ4;RQ5;RQ6 |
| `deaths` | 死亡人数 | integer/null | 当前文档明确报告的死亡人数；统计时点、初步性及限定词均属于值的解释。 仅录入明确死亡人数。不得将失踪人数与死亡人数相加，也不得把“死亡和失踪”合计当作死亡人数；明确的0才记0。 未提及为 missing；明确尚未查明/待核实为 unknown；仅有区间、下限或混合死亡失踪数为 partial；同篇冲突为 conflict。 标准值用非负整数，单位为人；估计、截至时点、范围和是否含失踪写入value_qualifier。不能拆分的混合数不填为死亡人数。 引用明确限定为死亡人数的原文，包含数字、单位和必要限定词。 整数严格比较；统计口径或时点不一致需人工判断。死亡失踪合计不可匹配死亡人数。 | RQ1;RQ2;RQ3;RQ4;RQ5;RQ6 |
| `injuries` | 受伤人数 | integer/null | 当前文档明确报告的受伤人数；伤情类别及统计时点需保留。 仅录入原文报告的受伤人数。普通受伤人数不能改写为重伤人数；不得推定未报告的伤情类别。 未提及为 missing；明确待核实为 unknown；只有部分伤情类别、区间或下限为 partial；同篇冲突为 conflict。 标准值用非负整数，单位为人；轻重伤类别、估计和截至时点放入value_qualifier。不可拆分的类别合计保留原口径。 引用明确报告受伤人数及伤情类别的连续原文。 人数和伤情口径均须匹配；普通受伤人数不等于重伤人数。 | RQ1;RQ2;RQ3;RQ4;RQ5;RQ6 |
| `casualties_total` | 伤亡总人数 | integer/null | 文档直接陈述的伤亡总人数，按原文统计定义和时点解释。 只有文本明确给出伤亡总人数时才填值。绝不由死亡人数与受伤人数相加推导，也不将死亡失踪合计自动当作伤亡总数。 未提及为 missing；明确未统计为 unknown；仅给出组成部分、区间或定义不完整为 partial；同篇冲突为 conflict。 标准值用非负整数，单位为人；组成口径、是否含失踪、统计时点及估计限定放入value_qualifier。 引用直接陈述总人数的原文；仅分别列出死亡和受伤数不足以支持总数。 仅与文档明确报告的总数比较；推导和口径不一致均不计精确匹配。 | RQ1;RQ2;RQ3;RQ4;RQ5;RQ6 |
| `property_loss` | 财产损失 | number/string/null | 当前文档明确报告的财产损失金额及其估算性质、币种和统计范围。 只提取文档明确金额；保留约数、估算、直接经济损失等口径。无金额数值时保留原文表达，不推断金额。 未提及为 missing；明确尚未核定为 unknown；只有范围/定性描述或不完整金额为 partial；同篇冲突为 conflict。 可无歧义换算时统一到人民币元并以十进制数表示，同时在value_qualifier留存原单位和限定词；币种或换算不明则原样保留，不跨币种推算。 引用同时支持金额及单位/损失口径的连续原文。 数额、币种、估算属性及损失口径一起比较；约数、币种换算或范围需人工确认。 | RQ1;RQ2;RQ3;RQ4;RQ5;RQ6 |
| `cause` | 事故原因 | string/null | 当前文档对事故原因作出的明示判断，且不跨越其调查阶段和结论强度。 区分调查结论、初步判断、媒体转述、猜测和正在调查；不得将后期结论写回早期文档。 未提及为 missing；明确仍在调查/尚未查明为 unknown；只披露原因的一部分为 partial；同篇无法消解的原因表述矛盾为 conflict。 保留原文因果层级、责任程度和限定词；不自行增加因果链。 引用支持原因表述及其结论强度的连续原文。 比较时必须核对结论阶段与语气；不同阶段更新不自动构成错误或同篇冲突。 | RQ1;RQ2;RQ3;RQ4;RQ5;RQ6 |
| `response` | 处置措施 | string/list | 当前文档已报告实施中的救援、处置、善后或应急响应措施。 只记录已发生或正在实施的措施；拟议、计划或建议措施需明确标记，不得当作已完成。 未提及为 missing；明确尚未决定为 unknown；只报告部分环节或进展为 partial；同篇矛盾为 conflict。 多项措施可用JSON字符串数组表示；保留措施主体、动作和完成状态。 引用明确说明措施及其时态/进展的连续原文。 措施内容、实施主体和完成状态应匹配；计划不等于执行。 | RQ1;RQ2;RQ3;RQ4;RQ5 |
| `information_source` | 信息来源 | string | 正文明确表述的信息发布者、消息来源或引述来源；不是来源登记表中的元数据。 仅抽取正文可见来源表述；不能仅凭网站域名、sources发布机构栏或文档元数据推断。 正文未提及为 missing；正文明确表示来源尚未确认时为 unknown；来源链条不完整为 partial。 保留原文机构/消息来源名称及直接引述关系。 引用正文中明确出现的来源名称或来源措辞。 只评价输入正文中的可观察信息；登记元数据不能作为证据。 | RQ1;RQ2;RQ3;RQ4;RQ5 |

## 文件与列定义

### `data/raw/sources.xlsx`

- 作用：不可覆盖的来源登记表；保留原始19列和现有值。
- 格式/工作表：xlsx / 资料登记
- 主键：`event_id + doc_id`
- 填写者：人工维护；对应研究模块：RQ1-RQ6（样本与阶段元数据）
- 关联：无外键

| 字段（顺序固定） | 类型 | 可空 | 允许值 | 含义 | 填写者/模块 | 示例 | 异常处理 |
|---|---|---|---|---|---|---|---|
| `event_id` | string | 否 | — | 事件唯一标识；跨表关联键。 | 人工维护 / RQ1-RQ6（样本与阶段元数据） | EVENT_001 | 缺失或拼写不一致视为外键错误。 |
| `doc_id` | string | 否 | — | 来源文档唯一标识。 | 人工维护 / RQ1-RQ6（样本与阶段元数据） | DOC_001_01 | 不得重命名；必须关联来源登记。 |
| `title` | string | 是 | — | 来源登记标题。 | 人工维护 / RQ1-RQ6（样本与阶段元数据） | 江西新余火灾通报 | 保留原始登记内容。 |
| `source_agency` | string | 是 | — | 来源发布机构或转载来源。 | 人工维护 / RQ1-RQ6（样本与阶段元数据） | 应急管理部 | 是来源元数据，不等同正文information_source。 |
| `publish_date` | date/string | 是 | — | 来源公开发布日期。 | 人工维护 / RQ1-RQ6（样本与阶段元数据） | 2024-01-26 | 用于公开时间排序；不代替event_time。 |
| `publish_time` | time/string | 是 | — | 来源公开时间。 | 人工维护 / RQ1-RQ6（样本与阶段元数据） | 18:19 | 缺失时不推断。 |
| `document_date` | date/string | 是 | — | 文档自身标注日期。 | 人工维护 / RQ1-RQ6（样本与阶段元数据） | — | 保留原值，不与publish_date混同。 |
| `stage` | string | 是 | `early`, `update`, `investigation`, `final`, `follow_up` | 来源记录所处的事故信息阶段。 | 人工维护 / RQ1-RQ6（样本与阶段元数据） | early | 只允许五个冻结值；未映射历史值需人工裁决。 |
| `url` | string | 是 | — | 公开来源URL。 | 人工维护 / RQ1-RQ6（样本与阶段元数据） | https://example.org/report | 不得将网页正文自动写入文本。 |
| `local_path` | string | 是 | — | 现有本地来源文件路径。 | 人工维护 / RQ1-RQ6（样本与阶段元数据） | — | 缺失不删除来源记录。 |
| `raw_file_path` | string | 是 | — | 原始PDF/来源文件的相对或绝对路径。 | 人工维护 / RQ1-RQ6（样本与阶段元数据） | — | 程序只检查，不修改文件。 |
| `text_file_path` | string | 是 | — | 对应TXT文件路径。 | 人工维护 / RQ1-RQ6（样本与阶段元数据） | — | 空白时按doc_id查找；多条匹配报错。 |
| `event_date` | date/string | 是 | — | 登记的事故日期元数据。 | 人工维护 / RQ1-RQ6（样本与阶段元数据） | 2024-01-24 | 不替代正文中抽取的event_time。 |
| `retrieval_date` | date/string | 是 | — | 来源获取/登记日期。 | 人工维护 / RQ1-RQ6（样本与阶段元数据） | — | 不从文件时间戳臆造。 |
| `document_type` | string | 是 | — | 原始登记的文档类型。 | 人工维护 / RQ1-RQ6（样本与阶段元数据） | official_notice | 保留既有分类，不在本次重映射。 |
| `included_in_dataset` | string | 是 | — | 来源纳入数据集的登记状态。 | 人工维护 / RQ1-RQ6（样本与阶段元数据） | pending | 本次不据此删除来源行。 |
| `source_type` | string | 是 | — | 来源获取形式。 | 人工维护 / RQ1-RQ6（样本与阶段元数据） | web_pdf | 沿用原值。 |
| `verification_status` | string | 是 | — | 来源核验状态。 | 人工维护 / RQ1-RQ6（样本与阶段元数据） | verified | 沿用原值。 |
| `notes` | string | 是 | — | 标注或处理备注。 | 人工维护 / RQ1-RQ6（样本与阶段元数据） | 需复核时间精度 | 不得用备注替代字段证据。 |

### `data/configs/schema.xlsx`

- 作用：字段定义、FSU旧模板、填写说明和原Gold模板；既有工作表及内容保留。
- 格式/工作表：xlsx / 工作簿（含四个既有工作表）
- 主键：`field_key`
- 填写者：研究者维护；对应研究模块：RQ1-RQ6
- 关联：无外键

| 字段（顺序固定） | 类型 | 可空 | 允许值 | 含义 | 填写者/模块 | 示例 | 异常处理 |
|---|---|---|---|---|---|---|---|
| `field_key` | string | 否 | — | Schema 1.0固定业务字段键。 | 研究者维护 / RQ1-RQ6 | deaths | 必须属于11个固定字段。 |
| `字段中文名` | string | 否 | — | 业务字段的中文名称。 | 研究者维护 / RQ1-RQ6 | 死亡人数 | 优先沿用既有Schema名称。 |
| `字段组` | string | 否 | — | 业务字段所属的概念分组。 | 研究者维护 / RQ1-RQ6 | 数值结果 | 保留原字段分组，不据此改变字段语义。 |
| `数据类型` | string | 否 | — | 业务字段标准值的逻辑类型。 | 研究者维护 / RQ1-RQ6 | non-negative integer | Gold/Predict CSV仍按字符串保存，逻辑类型由Schema约束。 |
| `抽取口径（待核实/可修改）` | string | 是 | — | 原Schema已有的抽取口径说明。 | 研究者维护 / RQ1-RQ6 | 只抽取原文明确报告值 | 保留原文；具体规则见本工作表新增规范列。 |
| `常见状态/难点` | string | 是 | — | 既有Schema记录的常见状态或标注难点。 | 研究者维护 / RQ1-RQ6 | unknown / conflict | 作为提示，不替代冻结状态枚举。 |
| `示例值（虚构）` | string | 是 | — | 格式示例，不是数据集中的真实标签。 | 研究者维护 / RQ1-RQ6 | 2 | 不得导入Gold或模型结果。 |
| `标注备注` | string | 是 | — | 既有字段的人工标注提示。 | 研究者维护 / RQ1-RQ6 | 0不等于未知 | 保留既有含义。 |
| `字段定义` | string | 否 | — | 冻结的业务字段定义。 | 研究者维护 / RQ1-RQ6 | 文档明确报告的死亡人数 | 不允许通过语义近似扩张字段范围。 |
| `提取规则` | string | 否 | — | 字段级抽取规则。 | 研究者维护 / RQ1-RQ6 | 不将失踪人数并入死亡 | 不得从其它文档或外部知识补齐。 |
| `缺失与未知判断` | string | 否 | — | 字段的unknown、missing、partial区分规则。 | 研究者维护 / RQ1-RQ6 | 未提及为missing | 空白标注状态不等于missing。 |
| `数值单位和标准化规则` | string | 否 | — | 字段数值/时间单位及规范化方法。 | 研究者维护 / RQ1-RQ6 | 人数按人计整数 | 不可丢失币种、统计时点或估算限定。 |
| `证据要求` | string | 否 | — | 原文证据的范围和形式要求。 | 研究者维护 / RQ1-RQ6 | 连续原文片段 | Gold证据只能来自当前doc_id文本。 |
| `评价规则` | string | 否 | — | 字段值与状态的比较规则。 | 研究者维护 / RQ1-RQ6 | 数值严格比较 | 不可跨口径或跨阶段倒判。 |
#### 工作表 `11字段定义`

- 作用：11个业务字段的冻结定义及规范。
- 主键：`field_key`

| 字段（顺序固定） | 类型 | 可空 | 允许值 | 含义 | 填写者/模块 | 示例 | 异常处理 |
|---|---|---|---|---|---|---|---|
| `field_key` | string | 否 | — | Schema 1.0固定业务字段键。 | 研究者维护 / RQ1-RQ6 | deaths | 必须属于11个固定字段。 |
| `字段中文名` | string | 是 | — | 业务字段的中文名称。 | 研究者维护 / RQ1-RQ6 | 死亡人数 | 优先沿用既有Schema名称。 |
| `字段组` | string | 是 | — | 业务字段所属的概念分组。 | 研究者维护 / RQ1-RQ6 | 数值结果 | 保留原字段分组，不据此改变字段语义。 |
| `数据类型` | string | 是 | — | 业务字段标准值的逻辑类型。 | 研究者维护 / RQ1-RQ6 | non-negative integer | Gold/Predict CSV仍按字符串保存，逻辑类型由Schema约束。 |
| `抽取口径（待核实/可修改）` | string | 是 | — | 原Schema已有的抽取口径说明。 | 研究者维护 / RQ1-RQ6 | 只抽取原文明确报告值 | 保留原文；具体规则见本工作表新增规范列。 |
| `常见状态/难点` | string | 是 | — | 既有Schema记录的常见状态或标注难点。 | 研究者维护 / RQ1-RQ6 | unknown / conflict | 作为提示，不替代冻结状态枚举。 |
| `示例值（虚构）` | string | 是 | — | 格式示例，不是数据集中的真实标签。 | 研究者维护 / RQ1-RQ6 | 2 | 不得导入Gold或模型结果。 |
| `标注备注` | string | 是 | — | 既有字段的人工标注提示。 | 研究者维护 / RQ1-RQ6 | 0不等于未知 | 保留既有含义。 |
| `字段定义` | string | 是 | — | 冻结的业务字段定义。 | 研究者维护 / RQ1-RQ6 | 文档明确报告的死亡人数 | 不允许通过语义近似扩张字段范围。 |
| `提取规则` | string | 是 | — | 字段级抽取规则。 | 研究者维护 / RQ1-RQ6 | 不将失踪人数并入死亡 | 不得从其它文档或外部知识补齐。 |
| `缺失与未知判断` | string | 是 | — | 字段的unknown、missing、partial区分规则。 | 研究者维护 / RQ1-RQ6 | 未提及为missing | 空白标注状态不等于missing。 |
| `数值单位和标准化规则` | string | 是 | — | 字段数值/时间单位及规范化方法。 | 研究者维护 / RQ1-RQ6 | 人数按人计整数 | 不可丢失币种、统计时点或估算限定。 |
| `证据要求` | string | 是 | — | 原文证据的范围和形式要求。 | 研究者维护 / RQ1-RQ6 | 连续原文片段 | Gold证据只能来自当前doc_id文本。 |
| `评价规则` | string | 是 | — | 字段值与状态的比较规则。 | 研究者维护 / RQ1-RQ6 | 数值严格比较 | 不可跨口径或跨阶段倒判。 |

#### 工作表 `FSU填写模板`

- 作用：为兼容性保留的旧模板；正式预测写入outputs/predictions/predictions.csv。
- 主键：`doc_id + run_id + field_key`

| 字段（顺序固定） | 类型 | 可空 | 允许值 | 含义 | 填写者/模块 | 示例 | 异常处理 |
|---|---|---|---|---|---|---|---|
| `doc_id` | string | 否 | — | 来源文档唯一标识。 | 研究者维护 / RQ1-RQ6 | DOC_001_01 | 不得重命名；必须关联来源登记。 |
| `event_id` | string | 否 | — | 事件唯一标识；跨表关联键。 | 研究者维护 / RQ1-RQ6 | EVENT_001 | 缺失或拼写不一致视为外键错误。 |
| `run_id` | string | 是 | — | 一次模型调用的唯一运行标识。 | 研究者维护 / RQ1-RQ6 | run-20261124-001 | 不得复用或覆盖。 |
| `model_id` | string | 是 | — | 旧FSU模板中的模型标识。 | 研究者维护 / RQ1-RQ6 | model-a | 正式运行记录以experiment_runs.csv为准。 |
| `prompt_version` | string | 是 | — | 提示词规范版本。 | 研究者维护 / RQ1-RQ6 | prompt-1.0 | 与annotation_version分离。 |
| `field_key` | string | 否 | — | Schema 1.0固定业务字段键。 | 研究者维护 / RQ1-RQ6 | deaths | 必须属于11个固定字段。 |
| `value_pred` | string | 是 | — | 旧FSU模板中的模型预测值列。 | 研究者维护 / RQ1-RQ6 | 39 | 正式预测只写predictions.csv。 |
| `state_pred` | string | 是 | — | 旧FSU模板中的模型预测状态列。 | 研究者维护 / RQ1-RQ6 | known | 正式预测只写predictions.csv。 |
| `evidence_pred` | string | 是 | — | 旧FSU模板中的模型证据列。 | 研究者维护 / RQ1-RQ6 | 造成39人死亡 | 正式预测只写predictions.csv。 |
| `support_pred` | string | 是 | — | 旧FSU模板中的证据支持评价列。 | 研究者维护 / RQ1-RQ6 | supported | 正式人工支持判断写evidence_labels.xlsx。 |
| `confidence_pred` | number | 是 | — | 旧FSU模板中的模型置信度。 | 研究者维护 / RQ1-RQ6 | 0.8 | 自报值不等同校准概率；正式值写predictions.csv。 |
| `stability_score` | number | 是 | — | 旧FSU模板中的重复抽取稳定性指标。 | 研究者维护 / RQ1-RQ6 | 0.8 | 仅经定义和计算后填写。 |
| `error_type` | string | 是 | — | 旧FSU模板中的错误归因标签。 | 研究者维护 / RQ1-RQ6 | unsupported_value | 仅基于已审核Gold归因。 |
| `risk` | string | 是 | — | 旧FSU模板中的下游风险标签。 | 研究者维护 / RQ1-RQ6 | not_applicable | 不适用/无法判定不得强制分类。 |
| `notes` | string | 是 | — | 标注或处理备注。 | 研究者维护 / RQ1-RQ6 | 需复核时间精度 | 不得用备注替代字段证据。 |

#### 工作表 `填写说明`

- 作用：保留并补充数据填写说明。
- 主键：`无`

| 字段（顺序固定） | 类型 | 可空 | 允许值 | 含义 | 填写者/模块 | 示例 | 异常处理 |
|---|---|---|---|---|---|---|---|
| `项目` | string | 是 | — | 填写说明项目名。 | 研究者维护 / RQ1-RQ6 | 状态定义 | 仅作为工作簿说明。 |
| `填写指导` | string | 是 | — | 项目对应的填写解释。 | 研究者维护 / RQ1-RQ6 | Gold必须人工标注 | 遵守Schema Freeze 1.0，不用模型填Gold。 |

#### 工作表 `人工标注Gold`

- 作用：保留的原模板；Schema 1.0人工Gold正式表为data/annotations/gold_values.xlsx。
- 主键：`event_id + doc_id + field_key`

| 字段（顺序固定） | 类型 | 可空 | 允许值 | 含义 | 填写者/模块 | 示例 | 异常处理 |
|---|---|---|---|---|---|---|---|
| `doc_id` | string | 否 | — | 来源文档唯一标识。 | 研究者维护 / RQ1-RQ6 | DOC_001_01 | 不得重命名；必须关联来源登记。 |
| `event_id` | string | 否 | — | 事件唯一标识；跨表关联键。 | 研究者维护 / RQ1-RQ6 | EVENT_001 | 缺失或拼写不一致视为外键错误。 |
| `field_key` | string | 否 | — | Schema 1.0固定业务字段键。 | 研究者维护 / RQ1-RQ6 | deaths | 必须属于11个固定字段。 |
| `gold_value` | string | 是 | — | 人工审核后的字段标准答案；待标注为空。 | 研究者维护 / RQ1-RQ6 | 39 | 不得由模型自动填入；值与状态需相符。 |
| `gold_state` | string | 是 | — | Gold六状态；未标注留空。 | 研究者维护 / RQ1-RQ6 | known | 仅允许六状态或空白；不得由空值推成missing。 |
| `gold_evidence` | string | 是 | — | 当前doc_id原文中的连续证据片段。 | 研究者维护 / RQ1-RQ6 | 造成39人死亡 | 必须是当前TXT的原文子串；不得引用其他文档。 |
| `value_qualifier` | string | 是 | — | 数值/文本的时点、估计、范围、口径或不完整限定。 | 研究者维护 / RQ1-RQ6 | 截至1月25日；初步统计 | 不得丢弃会改变语义的限定。 |
| `annotator` | string | 是 | — | 人工标注者标识。 | 研究者维护 / RQ1-RQ6 | annotator_01 | 无人工标注时为空。 |
| `review_status` | string | 是 | — | 人工审核状态。 | 研究者维护 / RQ1-RQ6 | pending | 仅允许pending/reviewed/adjudicated。 |
| `notes` | string | 是 | — | 标注或处理备注。 | 研究者维护 / RQ1-RQ6 | 需复核时间精度 | 不得用备注替代字段证据。 |


### `data/configs/source_field_mapping.csv`

- 作用：实际来源列与冻结字段映射、未自动覆盖的阶段值说明。
- 格式/工作表：csv / CSV
- 主键：`target_field + actual_source_header + mapping_status`
- 填写者：程序生成/研究者确认；对应研究模块：RQ1-RQ6
- 关联：无外键

| 字段（顺序固定） | 类型 | 可空 | 允许值 | 含义 | 填写者/模块 | 示例 | 异常处理 |
|---|---|---|---|---|---|---|---|
| `target_field` | string | 否 | — | 待映射的Schema目标字段或阶段分类。 | 程序生成/研究者确认 / RQ1-RQ6 | stage | 不得通过相似名称推断后覆盖原始列。 |
| `actual_source_header` | string | 否 | — | 来源工作簿实际表头。 | 程序生成/研究者确认 / RQ1-RQ6 | stage | 必须与实际表头一致。 |
| `mapping_status` | string | 否 | `identity`, `unmapped_existing_value` | 映射确认状态。 | 程序生成/研究者确认 / RQ1-RQ6 | identity | 未确认枚举映射时使用unmapped_existing_value。 |
| `action` | string | 否 | `preserve`, `preserve; do not overwrite` | 对原值采取的兼容处理。 | 程序生成/研究者确认 / RQ1-RQ6 | preserve; do not overwrite | 不允许自动修改既有元数据。 |
| `notes` | string | 是 | — | 标注或处理备注。 | 程序生成/研究者确认 / RQ1-RQ6 | 需复核时间精度 | 不得用备注替代字段证据。 |

### `data/annotations/gold_values.xlsx`

- 作用：人工审核的文档级字段Gold；模型不得写入。
- 格式/工作表：xlsx / gold_values
- 主键：`event_id + doc_id + field_key`
- 填写者：人工标注/复核；对应研究模块：RQ1-RQ2-RQ4-RQ5-RQ6
- 关联：['event_id', 'doc_id'] → data/raw/sources.xlsx(event_id,doc_id)；['field_key'] → data/configs/schema.xlsx(11字段定义.field_key)

| 字段（顺序固定） | 类型 | 可空 | 允许值 | 含义 | 填写者/模块 | 示例 | 异常处理 |
|---|---|---|---|---|---|---|---|
| `event_id` | string | 否 | — | 事件唯一标识；跨表关联键。 | 人工标注/复核 / RQ1-RQ2-RQ4-RQ5-RQ6 | EVENT_001 | 缺失或拼写不一致视为外键错误。 |
| `doc_id` | string | 否 | — | 来源文档唯一标识。 | 人工标注/复核 / RQ1-RQ2-RQ4-RQ5-RQ6 | DOC_001_01 | 不得重命名；必须关联来源登记。 |
| `field_key` | string | 否 | — | Schema 1.0固定业务字段键。 | 人工标注/复核 / RQ1-RQ2-RQ4-RQ5-RQ6 | deaths | 必须属于11个固定字段。 |
| `gold_value` | string | 是 | — | 人工审核后的字段标准答案；待标注为空。 | 人工标注/复核 / RQ1-RQ2-RQ4-RQ5-RQ6 | 39 | 不得由模型自动填入；值与状态需相符。 |
| `gold_state` | string | 是 | `known`, `partial`, `unknown`, `missing`, `conflict`, `not_applicable` | Gold六状态；未标注留空。 | 人工标注/复核 / RQ1-RQ2-RQ4-RQ5-RQ6 | known | 仅允许六状态或空白；不得由空值推成missing。 |
| `gold_evidence` | string | 是 | — | 当前doc_id原文中的连续证据片段。 | 人工标注/复核 / RQ1-RQ2-RQ4-RQ5-RQ6 | 造成39人死亡 | 必须是当前TXT的原文子串；不得引用其他文档。 |
| `value_qualifier` | string | 是 | — | 数值/文本的时点、估计、范围、口径或不完整限定。 | 人工标注/复核 / RQ1-RQ2-RQ4-RQ5-RQ6 | 截至1月25日；初步统计 | 不得丢弃会改变语义的限定。 |
| `annotator` | string | 是 | — | 人工标注者标识。 | 人工标注/复核 / RQ1-RQ2-RQ4-RQ5-RQ6 | annotator_01 | 无人工标注时为空。 |
| `review_status` | string | 否 | `pending`, `reviewed`, `adjudicated` | 人工审核状态。 | 人工标注/复核 / RQ1-RQ2-RQ4-RQ5-RQ6 | pending | 仅允许pending/reviewed/adjudicated。 |
| `notes` | string | 是 | — | 标注或处理备注。 | 人工标注/复核 / RQ1-RQ2-RQ4-RQ5-RQ6 | 需复核时间精度 | 不得用备注替代字段证据。 |
| `evidence_start` | integer (Python Unicode character offset, 0-based inclusive) | 是 | — | 证据在当前TXT中的Python Unicode字符串起始偏移（0基、含）。 | 人工标注/复核 / RQ1-RQ2-RQ4-RQ5-RQ6 | 120 | 必须与evidence_end配对，且切片等于证据文本。 |
| `evidence_end` | integer (Python Unicode character offset, exclusive) | 是 | — | 证据在当前TXT中的Python Unicode字符串结束偏移（不含）。 | 人工标注/复核 / RQ1-RQ2-RQ4-RQ5-RQ6 | 127 | 必须大于evidence_start，且为左闭右开。 |
| `annotation_version` | string | 是 | — | 人工标注规范版本。 | 人工标注/复核 / RQ1-RQ2-RQ4-RQ5-RQ6 | 1.0 | 不是模型或运行版本。 |
| `reviewed_by` | string | 是 | — | 复核或裁决者标识。 | 人工标注/复核 / RQ1-RQ2-RQ4-RQ5-RQ6 | reviewer_01 | 未复核时为空。 |

### `data/annotations/evidence_labels.xlsx`

- 作用：人工审核具体预测的证据支持关系；与Gold证据分离。
- 格式/工作表：xlsx / evidence_labels
- 主键：`prediction_id + annotator`
- 填写者：人工标注/复核；对应研究模块：RQ2-RQ5
- 关联：['prediction_id'] → outputs/predictions/predictions.csv(prediction_id)；['run_id'] → outputs/runs/experiment_runs.csv(run_id)

| 字段（顺序固定） | 类型 | 可空 | 允许值 | 含义 | 填写者/模块 | 示例 | 异常处理 |
|---|---|---|---|---|---|---|---|
| `prediction_id` | string | 否 | — | 一次预测字段记录的唯一标识。 | 人工标注/复核 / RQ2-RQ5 | pred-001 | 必须唯一并关联run_id。 |
| `run_id` | string | 否 | — | 一次模型调用的唯一运行标识。 | 人工标注/复核 / RQ2-RQ5 | run-20261124-001 | 不得复用或覆盖。 |
| `event_id` | string | 否 | — | 事件唯一标识；跨表关联键。 | 人工标注/复核 / RQ2-RQ5 | EVENT_001 | 缺失或拼写不一致视为外键错误。 |
| `doc_id` | string | 否 | — | 来源文档唯一标识。 | 人工标注/复核 / RQ2-RQ5 | DOC_001_01 | 不得重命名；必须关联来源登记。 |
| `field_key` | string | 否 | — | Schema 1.0固定业务字段键。 | 人工标注/复核 / RQ2-RQ5 | deaths | 必须属于11个固定字段。 |
| `predicted_value` | string | 是 | — | 模型输出的字段值。 | 人工标注/复核 / RQ2-RQ5 | 39 | 不可写入Gold；无合法JSON字段时不建预测行。 |
| `predicted_evidence` | string | 是 | — | 模型给出的当前输入文档证据；多段证据以JSON数组字符串保存。 | 人工标注/复核 / RQ2-RQ5 | 造成39人死亡 | Gold与模型证据独立。单段证据可定位；多段或未定位时偏移留空，语义支持另由evidence_labels人工审核。 |
| `support_label` | string | 是 | `supported`, `partially_supported`, `unsupported`, `contradicted`, `not_enough_information` | 人工对模型证据支持关系的判断。 | 人工标注/复核 / RQ2-RQ5 | supported | 审核前为空；只能使用5类标签。 |
| `annotator` | string | 是 | — | 人工标注者标识。 | 人工标注/复核 / RQ2-RQ5 | annotator_01 | 无人工标注时为空。 |
| `review_status` | string | 否 | `pending`, `reviewed`, `adjudicated` | 人工审核状态。 | 人工标注/复核 / RQ2-RQ5 | pending | 仅允许pending/reviewed/adjudicated。 |
| `reviewed_by` | string | 是 | — | 复核或裁决者标识。 | 人工标注/复核 / RQ2-RQ5 | reviewer_01 | 未复核时为空。 |
| `notes` | string | 是 | — | 标注或处理备注。 | 人工标注/复核 / RQ2-RQ5 | 需复核时间精度 | 不得用备注替代字段证据。 |

### `data/annotations/temporal_changes.xlsx`

- 作用：同一事件跨文档的Gold信息变化；变化不是错误。
- 格式/工作表：xlsx / temporal_changes
- 主键：`event_id + field_key + earlier_doc_id + later_doc_id`
- 填写者：程序候选生成/人工审核；对应研究模块：RQ1-RQ5
- 关联：['event_id', 'earlier_doc_id'] → data/raw/sources.xlsx(event_id,doc_id)；['event_id', 'later_doc_id'] → data/raw/sources.xlsx(event_id,doc_id)；['event_id', 'field_key', 'earlier_doc_id', 'later_doc_id'] → data/annotations/gold_values.xlsx

| 字段（顺序固定） | 类型 | 可空 | 允许值 | 含义 | 填写者/模块 | 示例 | 异常处理 |
|---|---|---|---|---|---|---|---|
| `event_id` | string | 否 | — | 事件唯一标识；跨表关联键。 | 程序候选生成/人工审核 / RQ1-RQ5 | EVENT_001 | 缺失或拼写不一致视为外键错误。 |
| `field_key` | string | 否 | — | Schema 1.0固定业务字段键。 | 程序候选生成/人工审核 / RQ1-RQ5 | deaths | 必须属于11个固定字段。 |
| `earlier_doc_id` | string | 是 | — | 同一事件中较早公开/信息时间的文档。 | 程序候选生成/人工审核 / RQ1-RQ5 | DOC_001_01 | 不得依据doc_id排序。 |
| `later_doc_id` | string | 是 | — | 同一事件中较晚公开/信息时间的文档。 | 程序候选生成/人工审核 / RQ1-RQ5 | DOC_001_02 | 必须与earlier_doc_id属于同一事件。 |
| `earlier_value` | string | 是 | — | 较早文档已审核Gold值。 | 程序候选生成/人工审核 / RQ1-RQ5 | 25 | 不得用较晚调查结论回填。 |
| `later_value` | string | 是 | — | 较晚文档已审核Gold值。 | 程序候选生成/人工审核 / RQ1-RQ5 | 39 | 必须来自later_doc_id的审核Gold。 |
| `earlier_state` | string | 是 | — | 较早文档Gold状态。 | 程序候选生成/人工审核 / RQ1-RQ5 | known | 必须来自对应审核Gold。 |
| `later_state` | string | 是 | — | 较晚文档Gold状态。 | 程序候选生成/人工审核 / RQ1-RQ5 | known | 必须来自对应审核Gold。 |
| `change_type` | string | 是 | `unchanged`, `value_updated`, `unknown_to_known`, `known_to_unknown`, `partial_to_known`, `conflict_emerged`, `conflict_resolved`, `newly_reported`, `no_longer_reported`, `other` | 跨阶段信息变化类别。 | 程序候选生成/人工审核 / RQ1-RQ5 | value_updated | 仅允许指定10类；不等于模型错误。 |
| `annotator` | string | 是 | — | 人工标注者标识。 | 程序候选生成/人工审核 / RQ1-RQ5 | annotator_01 | 无人工标注时为空。 |
| `review_status` | string | 否 | `pending`, `reviewed`, `adjudicated` | 人工审核状态。 | 程序候选生成/人工审核 / RQ1-RQ5 | pending | 仅允许pending/reviewed/adjudicated。 |
| `notes` | string | 是 | — | 标注或处理备注。 | 程序候选生成/人工审核 / RQ1-RQ5 | 需复核时间精度 | 不得用备注替代字段证据。 |

### `data/splits/splits.csv`

- 作用：事件级开发集、校准集、测试集归属；冻结前保持空表。
- 格式/工作表：csv / CSV
- 主键：`event_id`
- 填写者：程序辅助/研究者冻结；对应研究模块：RQ1-RQ6
- 关联：['event_id'] → data/raw/sources.xlsx(event_id)

| 字段（顺序固定） | 类型 | 可空 | 允许值 | 含义 | 填写者/模块 | 示例 | 异常处理 |
|---|---|---|---|---|---|---|---|
| `event_id` | string | 否 | — | 事件唯一标识；跨表关联键。 | 程序辅助/研究者冻结 / RQ1-RQ6 | EVENT_001 | 缺失或拼写不一致视为外键错误。 |
| `split` | string | 否 | `development`, `calibration`, `test` | 事件级数据集分区。 | 程序辅助/研究者冻结 / RQ1-RQ6 | development | 仅允许development/calibration/test。 |

### `data/splits/split_manifest.json`

- 作用：事件级划分策略、版本及随机种子记录；正式划分前不填配置。
- 格式/工作表：json / JSON字段
- 主键：`schema_version`
- 填写者：研究者/程序记录；对应研究模块：RQ1-RQ6
- 关联：无外键

| 字段（顺序固定） | 类型 | 可空 | 允许值 | 含义 | 填写者/模块 | 示例 | 异常处理 |
|---|---|---|---|---|---|---|---|
| `schema_version` | string | 否 | — | 运行使用的Schema版本。 | 研究者/程序记录 / RQ1-RQ6 | 1.0 | 本冻结基准为1.0。 |
| `strategy` | string | 是 | — | 事件级分层划分方法。 | 研究者/程序记录 / RQ1-RQ6 | — | 划分完成前为空。 |
| `random_seed` | integer | 是 | — | 划分使用的随机种子。 | 研究者/程序记录 / RQ1-RQ6 | — | 未执行划分时为空。 |
| `data_version` | string | 是 | — | 用于划分的数据版本。 | 研究者/程序记录 / RQ1-RQ6 | — | 未冻结划分时为空。 |
| `split_time` | UTC datetime ISO 8601 | 是 | — | 划分生成时间。 | 研究者/程序记录 / RQ1-RQ6 | — | 未执行划分时为空。 |
| `splits_file` | string | 是 | — | 事件级划分表路径。 | 研究者/程序记录 / RQ1-RQ6 | data/splits/splits.csv | 固定路径。 |
| `notes` | string | 是 | — | 标注或处理备注。 | 研究者/程序记录 / RQ1-RQ6 | 需复核时间精度 | 不得用备注替代字段证据。 |

### `outputs/predictions/predictions.csv`

- 作用：逐运行、逐文档、逐字段的模型预测历史；不得覆盖历史记录。
- 格式/工作表：csv / CSV
- 主键：`prediction_id`
- 填写者：模型输出/程序写入；对应研究模块：RQ1-RQ2-RQ3-RQ4-RQ5-RQ6
- 关联：['run_id'] → outputs/runs/experiment_runs.csv(run_id)；['event_id', 'doc_id'] → data/raw/sources.xlsx(event_id,doc_id)；['field_key'] → data/configs/schema.xlsx(11字段定义.field_key)

| 字段（顺序固定） | 类型 | 可空 | 允许值 | 含义 | 填写者/模块 | 示例 | 异常处理 |
|---|---|---|---|---|---|---|---|
| `prediction_id` | string | 否 | — | 一次预测字段记录的唯一标识。 | 模型输出/程序写入 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | pred-001 | 必须唯一并关联run_id。 |
| `run_id` | string | 否 | — | 一次模型调用的唯一运行标识。 | 模型输出/程序写入 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | run-20261124-001 | 不得复用或覆盖。 |
| `event_id` | string | 否 | — | 事件唯一标识；跨表关联键。 | 模型输出/程序写入 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | EVENT_001 | 缺失或拼写不一致视为外键错误。 |
| `doc_id` | string | 否 | — | 来源文档唯一标识。 | 模型输出/程序写入 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | DOC_001_01 | 不得重命名；必须关联来源登记。 |
| `field_key` | string | 否 | — | Schema 1.0固定业务字段键。 | 模型输出/程序写入 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | deaths | 必须属于11个固定字段。 |
| `predicted_value` | string | 是 | — | 模型输出的字段值。 | 模型输出/程序写入 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | 39 | 不可写入Gold；无合法JSON字段时不建预测行。 |
| `predicted_state` | string | 是 | `known`, `partial`, `unknown`, `missing`, `conflict`, `not_applicable` | 模型输出的六状态。 | 模型输出/程序写入 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | known | 仅允许六状态。 |
| `predicted_evidence` | string | 是 | — | 模型给出的当前输入文档证据；多段证据以JSON数组字符串保存。 | 模型输出/程序写入 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | 造成39人死亡 | Gold与模型证据独立。单段证据可定位；多段或未定位时偏移留空，语义支持另由evidence_labels人工审核。 |
| `self_reported_confidence` | decimal number in [0,1] | 是 | — | 模型自报置信度，范围0-1。 | 模型输出/程序写入 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | 0.75 | 可空；不能直接视为校准概率。 |
| `evidence_start` | integer (Python Unicode character offset, 0-based inclusive) | 是 | — | 证据在当前TXT中的Python Unicode字符串起始偏移（0基、含）。 | 模型输出/程序写入 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | 120 | 必须与evidence_end配对，且切片等于证据文本。 |
| `evidence_end` | integer (Python Unicode character offset, exclusive) | 是 | — | 证据在当前TXT中的Python Unicode字符串结束偏移（不含）。 | 模型输出/程序写入 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | 127 | 必须大于evidence_start，且为左闭右开。 |
| `json_valid` | boolean | 是 | — | 模型响应是否为满足输出结构的合法JSON。 | 模型输出/程序写入 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | true | 非法JSON只登记run错误，不生成伪字段行。 |
| `parse_error` | string | 是 | — | JSON/字段结构解析错误。 | 模型输出/程序写入 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | — | 可定位和语义支持是不同概念。 |
| `raw_response_path` | string | 是 | — | 原始响应归档路径。 | 模型输出/程序写入 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | outputs/raw/run-1.json | 避免把密钥写入原始记录。 |

### `outputs/runs/experiment_runs.csv`

- 作用：每次模型调用的可追溯配置、时间、token和状态。
- 格式/工作表：csv / CSV
- 主键：`run_id`
- 填写者：程序记录；对应研究模块：RQ1-RQ3-RQ4-RQ5-RQ6
- 关联：无外键

| 字段（顺序固定） | 类型 | 可空 | 允许值 | 含义 | 填写者/模块 | 示例 | 异常处理 |
|---|---|---|---|---|---|---|---|
| `run_id` | string | 否 | — | 一次模型调用的唯一运行标识。 | 程序记录 / RQ1-RQ3-RQ4-RQ5-RQ6 | run-20261124-001 | 不得复用或覆盖。 |
| `model_provider` | string | 是 | — | 模型服务提供方。 | 程序记录 / RQ1-RQ3-RQ4-RQ5-RQ6 | provider | API不提供时为空。 |
| `model_name` | string | 是 | — | 请求使用的模型名称。 | 程序记录 / RQ1-RQ3-RQ4-RQ5-RQ6 | model-name | 必须可追溯。 |
| `model_version` | string | 是 | — | 服务商提供的模型版本/快照。 | 程序记录 / RQ1-RQ3-RQ4-RQ5-RQ6 | — | 未知时为空，不得猜测。 |
| `prompt_version` | string | 是 | — | 提示词规范版本。 | 程序记录 / RQ1-RQ3-RQ4-RQ5-RQ6 | prompt-1.0 | 与annotation_version分离。 |
| `schema_version` | string | 否 | — | 运行使用的Schema版本。 | 程序记录 / RQ1-RQ3-RQ4-RQ5-RQ6 | 1.0 | 本冻结基准为1.0。 |
| `dataset_version` | string | 是 | — | 运行使用的数据集版本。 | 程序记录 / RQ1-RQ3-RQ4-RQ5-RQ6 | dataset-1.0 | 需随数据版本记录。 |
| `split` | string | 否 | `development`, `calibration`, `test` | 事件级数据集分区。 | 程序记录 / RQ1-RQ3-RQ4-RQ5-RQ6 | development | 仅允许development/calibration/test。 |
| `temperature` | decimal number | 是 | — | API采样温度。 | 程序记录 / RQ1-RQ3-RQ4-RQ5-RQ6 | 0 | API不支持/未记录时为空。 |
| `top_p` | decimal number in [0,1] | 是 | — | API nucleus sampling参数。 | 程序记录 / RQ1-RQ3-RQ4-RQ5-RQ6 | — | 不支持时为空。 |
| `seed` | integer | 是 | — | API随机种子。 | 程序记录 / RQ1-RQ3-RQ4-RQ5-RQ6 | — | 不支持时为空。 |
| `repeat_index` | integer | 否 | — | 同配置重复抽取序号。 | 程序记录 / RQ1-RQ3-RQ4-RQ5-RQ6 | 1 | 用于RQ3；从1开始。 |
| `request_timestamp` | UTC datetime ISO 8601 | 是 | — | 请求发出UTC时间。 | 程序记录 / RQ1-RQ3-RQ4-RQ5-RQ6 | 2026-11-24T07:35:12Z | 调用失败也应记录。 |
| `response_timestamp` | UTC datetime ISO 8601 | 是 | — | 响应收到UTC时间。 | 程序记录 / RQ1-RQ3-RQ4-RQ5-RQ6 | — | 无响应时为空。 |
| `input_tokens` | integer | 是 | — | API报告的输入token数。 | 程序记录 / RQ1-RQ3-RQ4-RQ5-RQ6 | — | 服务商未返回时为空。 |
| `output_tokens` | integer | 是 | — | API报告的输出token数。 | 程序记录 / RQ1-RQ3-RQ4-RQ5-RQ6 | — | 服务商未返回时为空。 |
| `status` | string | 是 | `success`, `failed`, `invalid_json`, `invalid_output`, `not_run` | 模型调用最终状态。 | 程序记录 / RQ1-RQ3-RQ4-RQ5-RQ6 | success | 失败应记录error_message，不造成功预测。 |
| `error_message` | string | 是 | — | 调用或解析失败原因。 | 程序记录 / RQ1-RQ3-RQ4-RQ5-RQ6 | — | 成功时为空；不得保存密钥。 |

### `data/processed/data_quality.csv`

- 作用：来源、原始文件、TXT及Gold覆盖的自动质量报告。
- 格式/工作表：csv / CSV
- 主键：`event_id + doc_id`
- 填写者：程序生成；对应研究模块：RQ1-RQ2-RQ3-RQ4-RQ5-RQ6
- 关联：无外键

| 字段（顺序固定） | 类型 | 可空 | 允许值 | 含义 | 填写者/模块 | 示例 | 异常处理 |
|---|---|---|---|---|---|---|---|
| `event_id` | string | 否 | — | 事件唯一标识；跨表关联键。 | 程序生成 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | EVENT_001 | 缺失或拼写不一致视为外键错误。 |
| `doc_id` | string | 否 | — | 来源文档唯一标识。 | 程序生成 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | DOC_001_01 | 不得重命名；必须关联来源登记。 |
| `source_registered` | boolean | 是 | — | 文档是否登记在sources.xlsx。 | 程序生成 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | true | 程序生成。 |
| `raw_file_exists` | boolean | 是 | — | 登记或可唯一定位的原始PDF/文件是否存在。 | 程序生成 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | false | 空路径不代表来源记录应删除。 |
| `text_file_exists` | boolean | 是 | — | 对应TXT是否存在。 | 程序生成 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | true | 缺少TXT的来源文档仍保留登记。 |
| `text_readable` | boolean | 是 | — | TXT是否可按UTF-8读取。 | 程序生成 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | true | 不存在或解码失败为false。 |
| `text_char_count` | integer | 是 | — | UTF-8解码后Python字符串字符数。 | 程序生成 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | 2345 | 空白正文仍计数并报告质量问题。 |
| `text_sha256` | string | 是 | — | TXT字节内容SHA-256。 | 程序生成 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | a1b2... | 无文件时为空。 |
| `gold_expected_count` | integer | 是 | — | 该文档预期的11条Gold行数。 | 程序生成 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | 11 | 程序按固定11字段计算。 |
| `gold_reviewed_count` | integer | 是 | — | 状态为reviewed或adjudicated的Gold行数。 | 程序生成 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | 0 | pending行不计入。 |
| `quality_status` | string | 是 | `ok`, `warning`, `error` | 该文档数据质量汇总等级。 | 程序生成 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | error | error/warning/ok由程序生成。 |
| `issues` | string | 是 | — | 质量问题代码，以分号分隔。 | 程序生成 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | TXT_MISSING;GOLD_PENDING | 机器生成，不手动编辑。 |
| `checked_at` | UTC datetime ISO 8601 | 是 | — | 质量检查UTC时间。 | 程序生成 / RQ1-RQ2-RQ3-RQ4-RQ5-RQ6 | 2026-11-24T07:35:12Z | 每次程序检查更新。 |

## 本次兼容性迁移记录

- 来源表13个目标元数据名均与当前19列中的实际列名完全匹配；不新增重复语义字段，全部原列与值保留。
- `DOC_003_05` 的 `rectification` 候选映射为 `follow_up`；`DOC_008_04`、`DOC_009_03` 的 `final_news` 候选映射为 `final`。这是待确认建议，来源表值未被覆盖，质量表和映射报告均标记未映射。
- 旧版 `outputs/gold_template_sources_v1.2.xlsx` 的638行全部为 `approved`，其中66行为 `assistant_draft`；且仅5篇TXT可用。为避免将未核实草稿带入正式Gold，旧文件原样保留但不导入。Schema 1.0 Gold新建638条空值/pending记录。

## 关联键与冻结规则

来源→Gold按`event_id + doc_id`；Schema→Gold按`field_key`；运行→预测按`run_id`；预测→证据标签按`prediction_id`；时间变化按同一`event_id + field_key + earlier_doc_id + later_doc_id`连接Gold；划分按`event_id`。预测也必须关联一次运行、来源文档和固定字段。

Schema 1.0之后只允许追加行或创建兼容的新版本。修改列名、类型、枚举、字段含义或文件结构前必须先提交迁移方案，记录兼容影响并保留旧数据；不得重命名ID、删除Gold审核记录或将跨阶段更新倒灌为早期真值。
