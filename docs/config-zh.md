# 高级 Graph Node 配置

TOML 配置文件可用于设置比 CLI 中暴露的更复杂的配置。文件的位置通过 `--config` 命令行开关传递。当使用配置文件时，不可能使用选项 `--postgres-url`、`--postgres-secondary-hosts` 和 `--postgres-host-weights`。

TOML 文件由四个部分组成：

- `[chains]` 设置区块链客户端的端点。
- `[store]` 描述可用的数据库。
- `[ingestor]` 设置负责块摄取的节点的名称。
- `[deployment]` 描述如何放置新部署的子图。

这些部分中的一些支持环境变量扩展，最值得注意的是 Postgres 连接字符串。官方的 `graph-node` Docker 镜像包括 [`envsubst`](https://github.com/a8m/envsubst) 用于更复杂的用例。

## 配置多个数据库

对于大多数用例，单个 Postgres 数据库足以支持一个 `graph-node` 实例。当 `graph-node` 实例增长到一个单个 Postgres 数据库不够时，可以将 `graph-node` 数据的存储拆分到多个 Postgres 数据库中。所有数据库一起形成 `graph-node` 实例的存储。每个单独的数据库称为一个 _分片_。

`[store]` 部分必须始终配置一个主分片，必须称为 `primary`。每个分片可以有额外的读副本，用于响应查询。只有查询由读副本处理。索引和块摄取将始终使用主数据库。

还可以配置任意数量的其他分片，每个分片都有自己的读副本。当使用读副本时，查询流量将根据其权重在主数据库和副本之间分割。在下面的示例中，对于主要分片，不会将任何查询发送到主数据库，而副本将分别接收 50% 的流量。在 `vip` 分片中，50% 的流量发送到主数据库，另外 50% 发送到副本。

```toml
[store]
[store.primary]
connection = "postgresql://graph:${PGPASSWORD}@primary/graph"
weight = 0
pool_size = 10
[store.primary.replicas.repl1]
connection = "postgresql://graph:${PGPASSWORD}@primary-repl1/graph"
weight = 1
[store.primary.replicas.repl2]
connection = "postgresql://graph:${PGPASSWORD}@primary-repl2/graph"
weight = 1

[store.vip]
connection = "postgresql://graph:${PGPASSWORD}@${VIP_MAIN}/graph"
weight = 1
pool_size = 10
[store.vip.replicas.repl1]
connection = "postgresql://graph:${PGPASSWORD}@${VIP_REPL1}/graph"
weight = 1
```

`connection` 字符串必须是有效的 [libpq 连接字符串](https://www.postgresql.org/docs/current/libpq-connect.html#LIBPQ-CONNSTRING)。在将连接字符串传递给 Postgres 之前，嵌入在字符串中的环境变量会被扩展。

### 设置 `pool_size`

每个分片必须指示每个 `graph-node` 实例在该数据库的连接池中保留多少数据库连接。对于副本，池大小默认为主数据库的池大小，但也可以显式设置。这样的设置会替换主数据库的设置。

`pool_size` 可以是一个数字，就像上面的示例一样，这样任何 `graph-node` 实例都将使用该大小的连接池；或者是一组规则，针对不同的 `graph-node` 实例使用不同的大小，规则基于命令行上设置的 `node_id`。使用规则时，`pool_size` 设置如下：

```toml
pool_size = [
  { node = "index_node_general_.*", size = 20 },
  { node = "index_node_special_.*", size = 30 },
  { node = "query_node_.*", size = 80 }
]
```

每个规则由正则表达式 `node` 和如果当前实例的 `node_id` 匹配该正则表达式，则应使用的大小组成。您可以使用命令 `graphman config pools` 检查每个 `graph-node` 实例将使用多少连接以及所有 `graph-node` 实例将打开多少数据库连接。规则按照编写的顺序检查，使用第一个匹配的规则。如果没有规则匹配，则出错。

强烈建议每次更改配置时运行 `graphman config pools $all_nodes`，以确保连接池符合预期。这里，`$all_nodes` 应该是将使用此配置文件的所有节点名称的列表。

## 配置以太坊提供者

`[chains]` 部分控制 `graph-node` 连接到的以太坊提供者，以及每个链的块和其他元数据存储在哪里。该部分由进行块摄取的节点的名称（目前未使用）和一系列链组成。链的配置 `name` 在 `[chains.<name>]` 部分指定，由存储链数据的 `shard` 和该链的提供者列表组成。对于每个提供者，必须给出以下信息：

- `label`：在记录有关该提供者的信息时使用的标签（尚未实现）
- `transport`：`rpc`、`ws` 和 `ipc` 中的一个。默认为 `rpc`。
- `url`：提供者的 URL
- `features`：提供者支持的特性数组，可以为空，也可以是 `traces` 和 `archive` 的任意组合
- `headers`：要添加到每个请求的 HTTP 标头。默认为无。
- `limit`：可以使用此提供者的子图的最大数量。默认为无限制。至少应该有一个提供者是无限制的，否则 `graph-node` 可能无法处理所有

子图。对于此跟踪，该值是近似的，预计会有少量偏差。偏差将小于 10。

以下示例配置了两个链，`mainnet` 和 `kovan`，其中 `mainnet` 的块存储在 `vip` 分片中，而 `kovan` 的块存储在主分片中。`mainnet` 链可以使用两个不同的提供者，而 `kovan` 只有一个提供者。

```toml
[chains]
ingestor = "block_ingestor_node"
[chains.mainnet]
shard = "vip"
provider = [
  { label = "mainnet1", url = "http://..", features = [], headers = { Authorization = "Bearer foo" } },
  { label = "mainnet2", url = "http://..", features = [ "archive", "traces" ] }
]
[chains.kovan]
shard = "primary"
provider = [ { label = "kovan", url = "http://..", features = [] } ]
```

### 控制使用提供者的子图数量

**此功能是实验性的，可能会在将来的版本中删除**

每个提供者可以设置使用该提供者的子图的限制。对使用提供者的子图数量的测量是近似的，可能会与真实数量略有不同（通常少于 10）。

限制是通过匹配节点名称的规则设置的。如果节点的名称不匹配任何规则，则该节点的相应提供者将被禁用。

如果省略了匹配属性，则提供者将在每个节点上无限制。

建议至少有一个提供者通常是无限制的。
限制的设置方式如下：

```toml
[chains.mainnet]
shard = "vip"
provider = [
  { label = "mainnet-0", url = "http://..", features = [] },
  { label = "mainnet-1", url = "http://..", features = [],
    match = [
      { name = "some_node_.*", limit = 10 },
      { name = "other_node_.*", limit = 0 } ] } ]
```

命名为 `some_node_.*` 的节点将最多使用 `mainnet-1` 的 10 个子图，而其他一切则使用 `mainnet-0`，命名为 `other_node_.*` 的节点永远不会使用 `mainnet-1`，总是使用 `mainnet-0`。任何名称不匹配这些模式之一的节点都将无法使用 `mainnet-1`。

## 控制部署

当 `graph-node` 接收到部署新子图部署的请求时，它需要决定将部署的数据存储在哪个分片，并决定连接到存储的任何数量的节点中的哪些节点索引部署。该决定基于在 `[deployment]` 部分中定义的一系列规则。部署规则可以匹配子图名称和部署索引的网络。

规则按顺序评估，第一个匹配的规则确定部署的位置。规则的 `match` 元素可以有一个 `name`，用于针对部署的子图名称进行匹配的 [正则表达式](https://docs.rs/regex/1.4.2/regex/#syntax)，以及与新部署的索引的网络进行比较的 `network` 名称。`network` 名称可以是字符串，也可以是字符串列表。

最后一个规则不得具有 `match` 语句，以确保始终有一些分片和一些索引器将处理部署。

规则指示应将部署的数据存储在哪个 `shard` 中，默认为 `primary`，以及一系列 `indexers`。对于匹配规则，从 `indexers` 列表中选择一个索引器，以便部署均匀分布在所有在 `indexers` 中提到的节点上。索引器的名称必须与启动这些索引节点时传递的 `--node-id` 相同。

可以使用固定的 `shard`，也可以使用 `shards` 列表；在这种情况下，系统将使用给定列表中活动部署最少的分片。

```toml
[deployment]
[[deployment.rule]]
match = { name = "(vip|important)/.*" }
shard = "vip"
indexers = [ "index_node_vip_0", "index_node_vip_1" ]
[[deployment.rule]]
match = { network = "kovan" }
# No shard, so we use the default shard called 'primary'
indexers = [ "index_node_kovan_0" ]
[[deployment.rule]]
match = { network = [ "xdai", "poa-core" ] }
indexers = [ "index_node_other_0" ]
[[deployment.rule]]
# There's no 'match', so any subgraph matches
shards = [ "sharda", "shardb" ]
indexers = [
    "index_node_community_0",
    "index_node_community_1",
    "index_node_community_2",
    "index_node_community_3",
    "index_node_community_4",
    "index_node_community_5"
  ]

```

## 查询节点

节点可以通过在配置文件中包含以下内容来显式地配置为查询节点：

```toml
[general]
query = "<regular expression>"
```

任何 `--node-id` 匹配正则表达式的节点将被设置为仅响应查询。目前，这只意味着该节点不会尝试连接到任何配置的以太坊提供者。

## 基本设置

以下文件相当于使用 `--postgres-url` 命令行选项：

```toml
[store]
[store.primary]
connection="<.. postgres-url argument ..>"
[deployment]
[[deployment.rule]]
indexers = [ "<.. list of all indexing nodes ..>" ]
```

## 验证配置文件

可以使用 `config check` 命令检查配置文件的有效性。运行

```shell
graph-node --config $CONFIG_FILE config check
```

将读取配置文件并打印有关语法错误和一些内部不一致性的信息

，例如，在部署规则中使用未声明为存储的分片时。

## 模拟部署位置

给定一个配置文件，可以模拟新部署的子图部署的位置：

```shell
graphman --config $CONFIG_FILE config place some/subgraph mainnet
```

该命令不会进行任何更改，而只是打印该子图将被放置在哪里的信息。输出将指示将保存子图数据的数据库分片，以及可以用于对该子图进行索引的索引节点列表。在部署过程中，`graph-node` 从该列表中选择当前分配的子图最少的索引节点。