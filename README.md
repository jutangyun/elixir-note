
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

#### 创建Context
我理解这个context就是生成某个表的crud
```elixir
mix phx.gen.context Schemas.RecordTagRel record_tag_rel openid:string record_id:integer tag_id:integer
```

该命令会生成schema,migrate,crud文件.



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