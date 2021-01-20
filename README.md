
# HTTPpoison
POST请求，第二个参数body需要如下来传
```elixir
HTTPoison.post "url",  {:form, [page: 1,iplist: "MjMuMjI3LjM4LjMy"]}
```

# Elixir/Phoenix笔记
主要记录我司开发人员在日常开发中的一些开发笔记，毕竟elixir/phoenix太小众了，国内资料难找啊..

 

## Phoenix Fromework [https://www.phoenixframework.org/]
####创建phoenix项目
```elixir
mix phx.new PATH [--module MODULE] [--app APP]
Options
  • --live - include Phoenix.LiveView to make it easier than ever to build
    interactive, real-time applications
  • --umbrella - generate an umbrella project, with one application for
    your domain, and a second application for the web interface.
  • --app - the name of the OTP application
  • --module - the name of the base module in the generated skeleton
  • --database - specify the database adapter for Ecto. One of:
    • postgres - via https://github.com/elixir-ecto/postgrex
    • mysql - via https://github.com/elixir-ecto/myxql
    • mssql - via https://github.com/livehelpnow/tds

    Please check the driver docs for more information and requirements.
    Defaults to "postgres".

  • --no-webpack - do not generate webpack files for static asset building.
    When choosing this option, you will need to manually handle JavaScript
    dependencies if building HTML apps
  • --no-ecto - do not generate Ecto files.
  • --no-html - do not generate HTML views.
  • --no-gettext - do not generate gettext files.
  • --no-dashboard - do not include Phoenix.LiveDashboard
  • --binary-id - use binary_id as primary key type in Ecto schemas
  • --verbose - use verbose output
```
如果创建一个api项目，不需要html,assets，可以用:
```elixir
mix phx.new demo --no-webpack --no-html
```

安装依赖包
```elixir
mix deps.get
```
网络不好可以用镜像，shell(linux/osx)执行
```
HEX_MIRROR=https://repo.hex.pm mix deps.get
```
#### 创建Schema
```elixir
mix phx.gen.schema Schemas.RecordTagRel record_tag_rel openid:string record_id:integer tag_id:integer
```

该命令会生成一个schema和一个migrate文件，schema更像是 java中的entity吧，用来定义属性，以及关系等。
    * Schemas 是模块名称，会生成一个schemas目录
    * RecordTagRel为模块名称，同时会生成record_tag_rel.ex这样的文件
    * record_tag_rel为schema名称/表名，如：
  ```elixir
    schema "record_tag_rel" do
          field :openid, :string
          field :record_id, :integer
          field :tag, :string
          timestamps()
    end
  ```
最后执行
```elixir
mix ecto.migrate
```

#### 创建Context
我理解这个context就是生成某个表的crud
```elixir
mix phx.gen.context Schemas.RecordTagRel record_tag_rel openid:string record_id:integer tag_id:integer
```

该命令会生成schema,migrate,crud文件.

## 解决cors跨域问题
mix.exs中添加
```elixir
{:corsica, "~> 1.0"}
```
然后在endpoint文件底部添加
```elixir
plug Corsica, origins: "*", allow_headers: :all  <---这一行
plug ApiGatewayWeb.Router
```


## liveview 和live_component
liveview 一个实时视图，扩展名用leex，l就是live意思。
live_component顾名思义是一个组件，可以随便引用的
有个基于liveview/component的开源库surface
https://github.com/msaraiva/surface

在liveview中引用一个component：
```elixir
<%= live_component @socket, CardengineWeb.Card, id: "1", cardname: "bbbb"%>
```
组件要显示的内容可以放到render方法中或指定一个文件来渲染:
```elixir
 def render(assigns) do
    Phoenix.View.render(AppWeb.BoxesView, "boxes_list_component.html", assigns)
  end
  ```
id是组件编号，设置了id那么为状态组件，无状态组件不用传id。
如果组件内部有handle_event/3方法，那么id是必须传的，否则要报错，有了这个处理自身事件的方法
自然是有状态的组件了。

另外如果是要调用自己，需要传入phx-target="<%= @myself %>"，如在组件内部的render:
```elixir
  def render(assigns) do
    ~L"""
    <div class="card">
      <div>myname:<%=@cardname %></div>
      <button phx-click="card-click" phx-target="<%= @myself %>">card click</button>
    </div>
    """
  end
  ```
  渲染组件会生成一个按钮，点击按钮会触发card-click事件，如果没有phx-target="<%= @myself %>"，调用的是父view的handle_event/3 ,要调用组件内部的handle_event/3就需要加上这个。

## ECTO

1. 如果记录不存在就创建，否则counter 做update
```elixir
defp upsert!(path, counter) do
  import Ecto.Query
  date = Date.utc_today()
  query = from(m in Dashbit.Metrics.Metric, update: [inc: [counter: ^counter]])

  Dashbit.Repo.insert!(
    %Dashbit.Metrics.Metric{date: date, path: path, counter: counter},
    on_conflict: query,
    conflict_target: [:date, :path]
  )
end
```

2. 执行更新后返回一些值？
```elixir
result = from(v in MyModel, where: v.id == ^instance_id, select: {v.id, v.counter})
         |> MyRepo.update_all([inc: [counter: 1]])

```

3. 修改ECTO默认的inserted_at,updated_at字段
```elixir
use Ecto.Schema
下面添加
  @timestamps_opts inserted_at: :create_time
  @timestamps_opts updated_at: :update_time
  ```


## VSCode
1. 保存时候自动格式化
```vscode
"editor.formatOnSave": true,
```

2. debug
在项目根目录建立 launch.json文件:
```js
{
  // 使用 IntelliSense 了解相关属性。 
  // 悬停以查看现有属性的描述。
  // 欲了解更多信息，请访问: https://go.microsoft.com/fwlink/?linkid=830387
  "version": "0.2.0",
  "configurations": [
    {
      "type": "mix_task",
      "name": "mix (Default task)",
      "request": "launch",
      "projectDir": "${workspaceRoot}"
    },
    {
      "type": "mix_task",
      "name": "phx",
      "request": "launch",
      "task": "phx.server",
      "projectDir": "${workspaceRoot}"
     
    }
  ]
}
```
在debug中选择phx run 启动起来的项目可直接调试