# DDBOT 模板介绍

DDBOT的模板大致与GO标准库`text/template`与相同，想深入了解请参考[官方文档](https://pkg.go.dev/text/template) 。

**注：DDBOT模板从`v1.0.5`版本开始支持**

## 配置

DDBOT默认不启用自定义模板，当需要使用时，需要修改配置文件`application.yaml`，在其中增加一段配置，并且重启bot：

```yaml
template:
  enable: true
```

如果配置成功，启动时将显示日志信息：`已启用模板`。

配置成功后，DDBOT将会自动创建一个`template`文件夹，用于存放所有的模板文件。

DDBOT会**监控**该文件夹目录，这意味着对模板的创建/修改/删除等操作，无需重启BOT即可生效，DDBOT会自动使用最新的模板。

## 认识模板

模板是`template`文件夹下的一个文件，后缀名为`.tmpl`。

它是一个文本文件，可以使用文本编辑器进行编辑，它的文件名即是模板的名字，它的内容即是模板的内容。

### 文字模板

*创建一个模板，编辑其内容为：*

```text
这是一段文字，也是一段模板
```

*该模板将发送为：*

<img src="https://user-images.githubusercontent.com/11474360/161202913-63f2064d-de49-4d54-b1d8-748b62b91294.jpg" width="300">

### 模板的高级功能

在模板中，所有的`{{ ..... }}`都有特殊意义，例如：

- 使用`{{ pic "uri" }}`发送图片：

*创建一个模板，编辑其内容为：*

```text
发送一张图片 {{ pic "https://i2.hdslb.com/bfs/face/0bd7082c8c9a14ef460e64d5f74ee439c16c0e88.jpg" }}
```

*该模板将发送为：*

<img src="https://user-images.githubusercontent.com/11474360/161203028-e53a4fe0-c8ca-4e3e-a098-45c1f7ea78c9.jpg" width="300">

在这个例子中，使用了一个模板函数`pic`，它可以用来发送一张图片。

- 使用`{{ cut }}`发送分段消息：

*创建一个模板，编辑其内容为：*

```text
一、啦啦啦
{{- cut -}}
二、啦啦啦
```

*该模板将发送为：*

<img src="https://user-images.githubusercontent.com/11474360/161203068-0d26024f-37e6-4a04-b88e-cfedd0235924.jpg" width="300">

在这个例子中，使用了一个模板函数`cut`，它可以用来切分消息。

*请注意括号内的短横线`-`符号，它的作用是控制前后换行符*。

- 使用模板变量：

`.`符号表示引用一个模板变量，**根据模板应用的场合，能够使用的模板变量也不尽相同**。

例如：当自定义`/签到`命令模板时，能够使用`.score`变量与`.success`变量。

*创建一个模板，编辑其内容为：*

```text
{{- reply .msg -}}
{{ if .success -}}
签到大大大大大成功！获得1只萝莉，当前拥有{{.score}}只萝莉
{{- else -}}
明天再来吧，当前拥有{{.score}}只萝莉
{{- end }}
```

*该模板将发送为：*

<img src="https://user-images.githubusercontent.com/11474360/161203356-84f28ac5-a883-4213-92ed-3c03ad2e524e.jpg" width="300">

这个例子还展示了**回复消息语法**：`{{- reply .msg -}}`以及模板的**条件控制语法**：`{{if}} {{else}} {{end}}`

## 通过模板创建自定义命令回复

得益于模板的高度定制化能力，DDBOT现在支持通过模板发送消息的自定义命令：

例如可以创建一个`/群主女装`命令，并让这个命令发送定义在模板内的群主女装照。

首先需要在配置文件`application.yaml`中定义命令：

```yaml
autoreply:
  group:
    command: [ "群命令1", "群命令2" ]
  private:
    command: [ "私聊命令1", "私聊命令2" ]
```

在上面这段配置中，自定义了两个群命令`/群命令1`和`/群命令2`，两个私聊命令`/私聊命令1`和`/私聊命令2`。

完成后需要创建对应的模板文件：

- `custom.command.group.群命令1.tmpl`
- `custom.command.group.群命令2.tmpl`
- `custom.command.private.私聊命令1.tmpl`
- `custom.command.private.私聊命令2.tmpl`

当触发`/群命令1`的时候，则会自动发送模板消息`custom.command.group.群命令1.tmpl`。

当触发`/私聊命令1`的时候，则会自动发送模板消息`custom.command.private.私聊命令1.tmpl`。

其他命令也遵守这个规则。

## DDBOT新增的模板函数

- {{- cut -}}

用于发送分段消息，上面已介绍过

- {{ reply .msg }}

用于回复消息，上面已介绍过

- {{ prefix }}

引用配置中的command prefix，默认为`/`

- {{ pic "图片地址" }}

用于发送图片，支持`http/https`链接，以及本地路径。

图片格式支持 jpg / png / gif。

*如果路径是一个文件夹，则会在文件夹内随机选择一张图片。*

- {{ roll a b }}

在a ~ b范围内中随机一个数字，a b 返回值均为int64类型。

- {{ choose "a" "b" "c" }}

从传入的参数中随机返回一个，参数类型为string，支持变长参数。

- {{ at 123456 }}

发送@指定的qq号

- {{ icon 123456 }}

发送指定qq号的头像

## 当前支持的命令模板

命令通用模板变量：

| 模板变量        | 类型     | 含义                  |
|-------------|--------|---------------------|
| group_code  | int    | 本次命令触发的QQ群号码（私聊则为空） |
| group_name  | string | 本次命令触发的QQ群名称（私聊则为空） |
| member_code | int    | 本次命令触发的成员QQ号        |
| member_name | string | 本次命令触发的成员QQ名称       |

- /签到

模板名：`command.group.checkin.tmpl`

| 模板变量    | 类型   | 含义                             |
|---------|------|--------------------------------|
| success | bool | 表示本次签到是否成功，一天内只有第一次签到成功，后续签到失败 |
| score   | int  | 表示目前拥有的签到分数                    |

<details>
  <summary>默认模板</summary>

```text
{{ reply .msg }}{{if .success}}签到成功！获得1积分，当前积分为{{.score}}{{else}}明天再来吧，当前积分为{{.score}}{{end}}
```

</details>

- /help （私聊版）

模板名：`command.private.help.tmpl`

| 模板变量 | 类型  | 含义  |
|------|-----|-----|
| 无    |     ||

<details>
  <summary>默认模板</summary>

```text
常见订阅用法：
以作者UID:97505为例
首先订阅直播信息：{{ prefix }}watch 97505
然后订阅动态信息：{{ prefix }}watch -t news 97505
由于通常动态内容较多，可以选择不推送转发的动态
{{ prefix }}config filter not_type 97505 转发
还可以选择开启直播推送时@全体成员：
{{ prefix }}config at_all 97505 on
以及开启下播推送：
{{ prefix }}config offline_notify 97505 on
BOT还支持更多功能，详细命令介绍请查看命令文档：
https://gitee.com/sora233/DDBOT/blob/master/EXAMPLE.md
使用时请把作者UID换成你需要的UID
当您完成所有配置后，可以使用{{ prefix }}silence命令，让bot专注于推送，在群内发言更少
{{- cut -}}
B站专栏介绍：https://www.bilibili.com/read/cv10602230
如果您有任何疑问或者建议，请反馈到唯一指定交流群：755612788
```

</details>

- /help （群聊版）

模板名：`command.group.help.tmpl`

| 模板变量 | 类型  | 含义  |
|------|-----|-----|
| 无    |     ||

<details>
  <summary>默认模板</summary>

```text
DDBOT是一个多功能单推专用推送机器人，支持b站、斗鱼、油管、虎牙推送
```

</details>

- /lsp

模板名：`command.group.lsp.tmpl`

| 模板变量 | 类型  | 含义  |
|------|-----|-----|
| 无    |     ||

<details>
  <summary>默认模板</summary>

```text
{{ reply .msg -}}
LSP竟然是你
```

</details>

## 当前支持的推送模板

- b站直播推送

模板名：`notify.group.bilibili.live.tmpl`

| 模板变量   | 类型     | 含义          |
|--------|--------|-------------|
| living | bool   | 是否正在直播      |
| name   | string | 主播昵称        |
| title  | string | 直播标题        |
| url    | string | 直播间链接       |
| cover  | string | 直播间封面或者主播头像 |

<details>
  <summary>默认模板</summary>

```text
{{ if .living -}}
{{ .name }}正在直播【{{ .title }}】
{{ .url -}}
{{ pic .cover "[封面]" }}
{{- else -}}
{{ .name }}直播结束了
{{ pic .cover "[封面]" }}
{{- end -}}
```

</details>

- ACFUN站直播推送

模板名：`notify.group.acfun.live.tmpl`

| 模板变量   | 类型     | 含义          |
|--------|--------|-------------|
| living | bool   | 是否正在直播      |
| name   | string | 主播昵称        |
| title  | string | 直播标题        |
| url    | string | 直播间链接       |
| cover  | string | 直播间封面或者主播头像 |

<details>
  <summary>默认模板</summary>

```text
{{ if .living -}}
ACFUN-{{ .name }}正在直播【{{ .title }}】
{{ .url -}}
{{ pic .cover "[封面]" }}
{{- else -}}
ACFUN-{{ .name }}直播结束了
{{ pic .cover "[封面]" }}
{{- end -}}
```

</details>

- 斗鱼直播推送

模板名：`notify.group.douyu.live.tmpl`

| 模板变量   | 类型     | 含义          |
|--------|--------|-------------|
| living | bool   | 是否正在直播      |
| name   | string | 主播昵称        |
| title  | string | 直播标题        |
| url    | string | 直播间链接       |
| cover  | string | 直播间封面或者主播头像 |

<details>
  <summary>默认模板</summary>

```text
{{ if .living -}}
斗鱼-{{ .name }}正在直播【{{ .title }}】
{{ .url -}}
{{ pic .cover "[封面]" }}
{{- else -}}
斗鱼-{{ .name }}直播结束了
{{ pic .cover "[封面]" }}
{{- end -}}
```

</details>

- 虎牙直播推送

模板名：`notify.group.huya.live.tmpl`

| 模板变量   | 类型     | 含义          |
|--------|--------|-------------|
| living | bool   | 是否正在直播      |
| name   | string | 主播昵称        |
| title  | string | 直播标题        |
| url    | string | 直播间链接       |
| cover  | string | 直播间封面或者主播头像 |

<details>
  <summary>默认模板</summary>

```text
{{ if .living -}}
虎牙-{{ .name }}正在直播【{{ .title }}】
{{ .url -}}
{{ pic .cover "[封面]" }}
{{- else -}}
虎牙-{{ .name }}直播结束了
{{ pic .cover "[封面]" }}
{{- end -}}
```

</details>

## 当前支持的事件模板

- 有新成员加入群

模板名：`trigger.group.member_in.tmpl`

| 模板变量        | 类型     | 含义        |
|-------------|--------|-----------|
| group_code  | int64  | 群号码       |
| group_name  | string | 群名称       |
| member_code | int64  | 新加入的成员QQ号 |
| member_name | string | 新加入的成员昵称  |

<details>
  <summary>默认模板</summary>

*该模板默认为空，即不发送消息*

```text
```

</details>

- 有成员退出群

模板名：`trigger.group.member_out.tmpl`

| 模板变量        | 类型     | 含义       |
|-------------|--------|----------|
| group_code  | int64  | 群号码      |
| group_name  | string | 群名称      |
| member_code | int64  | 退出的成员QQ号 |
| member_name | string | 退出的成员昵称  |

<details>
  <summary>默认模板</summary>

*该模板默认为空，即不发送消息*

```text
```

</details>