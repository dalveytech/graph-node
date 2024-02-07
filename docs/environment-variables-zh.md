# 环境变量

**警告**：某些环境变量的名称在不久的将来可能会更改。

本页面列出了 `graph-node` 使用的环境变量及其效果。一些环境变量可以代替命令行标志使用。这些未在此处列出，请参阅 `graph-node --help` 获取详细信息。

## 用于 EVM 链的 JSON-RPC 配置

- `ETHEREUM_REORG_THRESHOLD`：最大预期的重组大小，如果发生更大的重组，则子图可能会处理不一致的数据。默认为 250。
- `ETHEREUM_POLLING_INTERVAL`：以多久的频率轮询以太坊获取新区块（毫秒，默认为 500ms）。
- `GRAPH_ETHEREUM_TARGET_TRIGGERS_PER_BLOCK_RANGE`：应该在批处理中处理的触发器的理想数量。如果这个值太小，可能会导致向以太坊节点发送太多请求；如果太大，可能会导致对以太坊节点的调用不合理昂贵，并且内存使用过多（默认为 100）。
- `ETHEREUM_TRACE_STREAM_STEP_SIZE`：当子图定义调用处理程序或具有调用过滤器的块处理程序时，`graph-node` 会查询给定区块范围的跟踪。该变量的值控制在单个 RPC 请求中扫描以太坊节点跟踪的区块数量。默认为 50。
- `DISABLE_BLOCK_INGESTOR`：将其设置为 `true` 以禁用块摄取。将其保留未设置或设置为 `false` 以启用块摄取。
- `ETHEREUM_BLOCK_BATCH_SIZE`：请求的以太坊块数量。同时也限制其他并行请求，如 trace_filter。默认为 10。
- `GRAPH_ETHEREUM_MAX_BLOCK_RANGE_SIZE`：每个请求中要扫描触发器的最大块数（默认为 1000）。
- `GRAPH_ETHEREUM_MAX_EVENT_ONLY_RANGE`：`eth.getLogs` 请求的最大范围大小，该请求不会根据合约地址进行过滤，仅根据事件签名进行过滤（默认为 500）。
- `GRAPH_ETHEREUM_JSON_RPC_TIMEOUT`：以太坊 JSON-RPC 请求的超时时间。
- `GRAPH_ETHEREUM_REQUEST_RETRIES`：针对以太坊进行的 JSON-RPC 请求的重试次数。这适用于在达到限制时不会使子图失败但会重新启动同步步骤的请求。该限制用于防止诸如请求已重组的块哈希等场景。默认为 10。
- `GRAPH_ETHEREUM_BLOCK_INGESTOR_MAX_CONCURRENT_JSON_RPC_CALLS_FOR_TXN_RECEIPTS`：在块摄取期间针对以太坊进行请求事务收据的最大并发请求次数。默认为 1,000。
- `GRAPH_ETHEREUM_FETCH_TXN_RECEIPTS_IN_BATCHES`：将其设置为 `true` 以在块摄取期间禁用从以太坊节点并发获取收据。这将使用更少的批量请求。在 MacOS 上始终设置为 `true` 以避免 DNS 问题。
- `GRAPH_ETHEREUM_CLEANUP_BLOCKS`：将其设置为 `true` 以清理数据库中不需要的块。当此值为 `false` 或未设置（默认情况下），块永远不会从块缓存中删除。此设置仅应在开发过程中使用，以减少数据库的大小。在生产环境中，它会导致同一块的多次下载，从而减慢系统速度。如果存储使用多个分片，则无法使用此设置。
- `GRAPH_ETHEREUM_GENESIS_BLOCK_NUMBER`：指定创世块编号。如果未设置标志，则默认值为 `0`。

## 运行映射处理程序

- `GRAPH_MAPPING_HANDLER_TIMEOUT`：映射处理程序允许的时间量（以秒为单位，默认为无限）。
- `GRAPH_ENTITY_CACHE_SIZE`：实体缓存的大小，以千字节为单位。默认为 10,000，即 10MB。
- `GRAPH_MAX_API_VERSION`：支持的最大 `apiVersion`。如果开发人员尝试在其映射中创建比此版本更高的 `apiVersion`，则将收到错误。默认为 `0.0.7`。
- `GRAPH_MAX_SPEC_VERSION`：支持的最大 `specVersion`。如果开发人员尝试创建一个比此版本更高的 `apiVersion` 的子图，他们将收到错误。默认为 `0.0.5`。
- `GRAPH_RUNTIME_MAX_STACK_SIZE`：WASM 运行时的最大堆栈大小，如果超出限制，执行将停止并抛出错误。默认为 512KiB。

## IPFS

- `GRAPH_IPFS_TIMEOUT`：IPFS 的超时时间，包括对清单文件和映射的请求（以秒为单位，默认为 60）。
- `GRAPH_MAX_IPFS_FILE_BYTES`：`ipfs cat` 调用可检索的文件的最大大小。这影响子图定义文件和 `file/ipfs` 数据源。以字节为单位，默认为 25 MiB。
- `GRAPH_MAX_IPFS_MAP_FILE_SIZE`：可以使用 `ipfs.map` 处理的文件的最大大小。当通过 `ipfs.map` 处理文件时，从该文件生成的实体将保留在内存中，直到完成整个文件的处理。因此，此设置限制了 `ipfs.map` 调用可能使用的内存量（以字节为单位，默认为 256MB）。
- `GRAPH_MAX_IPFS_CACHE_SIZE`：缓存的最大文件数（默认为 50）。
- `GRAPH_MAX_IPFS_CACHE_FILE_SIZE`：每个缓存文件的最大大小（以字节为单位，默认为 1MiB）

。
- `GRAPH_IPFS_REQUEST_LIMIT`：限制对 IPFS 的每秒请求次数，用于文件数据源。默认为 100。

## GraphQL

- `GRAPH_GRAPHQL_QUERY_TIMEOUT`：GraphQL 查询的最大执行时间（以秒为单位，默认为无限）。
- `GRAPH_GRAPHQL_MAX_COMPLEXITY`：GraphQL 查询的最大复杂度。默认为无限。有关详细信息，请参见[此处](https://developer.github.com/v4/guides/resource-limitations)。典型的内省查询的复杂度略高于 1 百万，因此将值设置为低于此值可能会干扰由 GraphQL 客户端执行的内省。
- `GRAPH_GRAPHQL_MAX_DEPTH`：GraphQL 查询的最大深度。默认（也是最大）为 255。
- `GRAPH_GRAPHQL_MAX_FIRST`：GraphQL 查询中可以用于 `first` 参数的最大值。如果未提供，则 `first` 默认为 100。`GRAPH_GRAPHQL_MAX_FIRST` 的默认值为 1000。
- `GRAPH_GRAPHQL_MAX_SKIP`：GraphQL 查询中可以用于 `skip` 参数的最大值。`GRAPH_GRAPHQL_MAX_SKIP` 的默认值为无限。
- `GRAPH_GRAPHQL_WARN_RESULT_SIZE` 和 `GRAPH_GRAPHQL_ERROR_RESULT_SIZE`：如果 GraphQL 结果的大小超过这些大小（以字节为单位），则分别记录警告或中止查询执行并返回错误。在构建响应时检查结果的大小，以便执行不会使用比配置更多的内存。两者的默认值都是无限。
- `GRAPH_GRAPHQL_MAX_OPERATIONS_PER_CONNECTION`：每个 WebSocket 连接的最大 GraphQL 操作数。超出限制后，任何创建的操作都将向客户端返回错误。默认为 1000。
- `GRAPH_GRAPHQL_HTTP_PORT`：GraphQL HTTP 服务器的端口。
- `GRAPH_GRAPHQL_WS_PORT`：GraphQL WebSocket 服务器的端口。
- `GRAPH_SQL_STATEMENT_TIMEOUT`：GraphQL 执行期间允许的单个 SQL 查询的最长时间（默认为无限）。
- `GRAPH_DISABLE_SUBSCRIPTION_NOTIFICATIONS`：禁用用于触发 GraphQL 订阅更新的内部机制。当此变量设置为任何值时，`graph-node` 仍将接受 GraphQL 订阅，但它们将不会收到任何更新。
- `ENABLE_GRAPHQL_VALIDATIONS`：启用基于 GraphQL 规范的 GraphQL 验证。这将验证并确保每个查询都遵循执行规则。默认为 `false`。
- `SILENT_GRAPHQL_VALIDATIONS`：如果启用了 `ENABLE_GRAPHQL_VALIDATIONS`，还可以仅静默打印 GraphQL 验证错误，而不会导致实际查询失败。注意：查询可能仍会因 GraphQL 引擎执行期间的后续阶段验证失败而失败。默认为 `true`。
- `GRAPH_GRAPHQL_DISABLE_BOOL_FILTERS`：禁用使用 AND/OR 过滤器的功能。如果出于性能原因要禁用过滤器，则此设置很有用。
- `GRAPH_GRAPHQL_DISABLE_CHILD_SORTING`：禁用使用基于子项的排序功能。如果出于性能原因要禁用基于子项的排序，则此设置很有用。
- `GRAPH_GRAPHQL_TRACE_TOKEN`：用于启用 GraphQL 请求的查询跟踪的令牌。如果设置了此项，则具有头 `X-GraphTraceQuery` 设置为该值的请求将包含对运行的 SQL 查询的跟踪。默认为空字符串，表示禁用跟踪。

### GraphQL 缓存

- `GRAPH_CACHED_SUBGRAPH_IDS`：当设置为 `*` 时，缓存所有子图（默认行为）。否则，逗号分隔的子图列表，用于缓存查询。
- `GRAPH_QUERY_CACHE_BLOCKS`：应在查询缓存中保留的每个网络的最近区块数。这应该保持较小，因为查找时间和缓存内存使用量与此值成比例。设置为 0 以禁用缓存。默认为 1。
- `GRAPH_QUERY_CACHE_MAX_MEM`：查询缓存可使用的最大总内存量，以 MB 为单位。用于缓存的总内存量将是这个值的两倍——一次用于最近的块，均匀分配在 `GRAPH_QUERY_CACHE_BLOCKS` 中，一次用于对旧块进行频繁查询。对于大多数负载来说，默认值足够了，尤其是如果 `GRAPH_QUERY_CACHE_BLOCKS` 保持较小。默认为 1000，对应 1GB。
- `GRAPH_QUERY_CACHE_STALE_PERIOD`：缓存条目被视为过时之前的查询次数。默认为 100。

## 其他

- `GRAPH_NODE_ID`：设置节点 ID，允许并行运行多个 Graph 节点并部署到特定节点；每个 ID 必须在节点集合中唯一。单个节点应在连续重启之间具有相同的值。子图将分配到节点 ID，并且不会自动重新分配到其他节点。
- `GRAPH_NODE_ID_USE_LITERAL_VALUE`：（仅限 Docker）使用提供给 docker 启动脚本的字面值 `node_id`，而不是将名称中的连字符（-）替换为下划线（\_）。更改现有 `graph-node` 安装的此设置还需要在数据库中的 `subgraphs.subgraph_deployment_assignment` 表中更改分配的节点 ID。可以使用 GraphMan 或通过 PostgreSQL 命令行完成。
- `GRAPH_LOG`：控制日志级别，方式与 `RUST_LOG` [这里](https://docs.rs/env_logger/0.6.0/env_logger/) 中描述的方式相同。
- `THEGRAPH_STORE_POSTGRES_DIESEL_URL`：运行测试时使用的 postgres 实例。设置为 `postgresql://<DBUSER>:<DBPASSWORD>@<DBHOST>:<DBPORT>/<DBNAME>`。
- `GRAPH_KILL_IF_UNRESPONSIVE`：如果设置，则进程将在无响应时被终止。
- `GRAPH_KILL_IF_UNRESPONSIVE_TIMEOUT_SECS`：如果 `GRAPH_KILL_IF_UNRESPONSIVE` 为 true，则在终止节点之前的超时时间（以秒为单位）。默认值为 10s。
- `GRAPH_LOG_QUERY_TIMING`：控制进程是否记录处理 GraphQL 和 SQL 查询的详细信息。值是逗号分隔的列表，包括 `sql`、`gql` 和 `cache`。如果列表中包含 `gql`，则每个针对节点执行的 GraphQL 查询都以 `info` 级别记录。日志消息包含查询的子图、查询、变量、查询所花费的时间以及唯一的 `query_id`。如果包含 `sql`，则记录 GraphQL 查询导致的 SQL 查询。日志消息包含子图、查询、绑定变量、执行查询所花费的时间、查询找到的实体数量以及导致 SQL 查询的 GraphQL 查询的 `query_id`。这些 SQL 查询标有 `component: GraphQlRunner`。当给出 `sql` 时，还会记录其他 SQL 查询。这些是在处理子图的块和处理订阅时导致的查询。如果 `cache` 除了 `gql` 外还包含，则还会为每个顶层 GraphQL 查询字段记录是否可以从缓存中检索到的信息。默认为不记录。
- `GRAPH_LOG_TIME_FORMAT`：自定义日志时间格式。默认值为 `%b %d %H:%M:%S%.3f`。更多信息 [此处](https://docs.rs/chrono/latest/chrono/#formatting-and-parsing)。
- `STORE_CONNECTION_POOL_SIZE`：允许与存储建立的同时连接数。由于实现细节，可能不严格遵守此值。默认为 10。
- `GRAPH_LOG_POI_EVENTS`：以确定性方式记录 Proof of Indexing 事件。这对于调试可能很有用。
- `GRAPH_LOAD_WINDOW_SIZE`、`GRAPH_LOAD_BIN_SIZE`：如果在 `GRAPH_LOAD_WINDOW_SIZE` 秒的时间段内的负载测量超过阈值，则负载可以自动限制。每个窗口内的测量被分组到 `GRAPH_LOAD_BIN_SIZE` 秒的时间段中。变量默认为 300s 和 1s。
- `GRAPH_LOAD_THRESHOLD`：如果获取数据库连接的等待时间超过此阈值，则在等待时间下降到阈值以下之前，会限制查询。值以毫秒为单位，默认为 0，表示关闭限制和任何相关的统计信息收集。
- `GRAPH_LOAD_JAIL_THRESHOLD`：当系统过载时，任何导致工作量超过此分数的查询都将被拒绝（即使在解决过载情况后也是如此）。如果未设置此变量，则永远不会将查询拘禁，但在系统过载时仍然会受到正常的负载管理限制。
- `GRAPH_LOAD_SIMULATE`：在给定其他负载管理配置设置的情况下执行负载管理将进行所有步骤，但实际上不会拒绝运行查询，而是记录有关负载管理决策的日志。设置为 `true` 以启用模拟，默认为 `false`。
- `GRAPH_STORE_CONNECTION_TIMEOUT`：连接到数据库前等待多长时间（以毫秒为单位）。默认为 5000ms。
- `EXPERIMENTAL_SUBGRAPH_VERSION_SWITCHING_MODE`：默认为 `instant`，设置为 `synced` 以仅在同步后将命名子图切换到新部署，使新部署成为“Pending”版本。
- `GRAPH_REMOVE_UNUSED_INTERVAL`：在删除未使用的部署之前等待多长时间。系统会定期检查并标记不再由任何子图使用的部署。一旦识别出部署未使用，`graph-node` 将等待至少这么长时间才会实际删除数据（值以分钟为单位，默认为 360，即 6 小时）。
- `GRAPH_ALLOW_NON_DETERMINISTIC_IPFS`：启用使用 `ipfs.cat` 作为子图映射的一部分的子图索引。**这是一个实验性功能，不确定，并且将在将来被删除**。
- `GRAPH_STORE_BATCH_TARGET_DURATION`：复制或嫁接期间批处理操作应该持续多长时间。这限制了长时间运行操作的事务持续时间。默认为 5s。
- `GRAPH_STORE_BATCH_SIZE`：复制或嫁接期间应执行的操作数量。默认为 100。
- `GRAPH_MIGRATION_DRY_RUN`：运行迁移时不修改数据库。

## 开发/测试

- `GRAPH_DISABLE_INDEXING`：在启动时，设置此变量以禁用索引创建和更新。
- `GRAPH_STORE_DEBUG`：用于调试存储后端的环境变量。设置为 `true` 以启用 `diesel` SQL 日志记录。默认为 `false`。
- `GRAPH_STORE_DEBUG_DISABLE_STORAGE_LIMITS`：在 `GRAPH_STORE_DEBUG` 启用时，设置此变量以禁用对存储的大小进行限制的功能（例如，每个实体的字段数量和长度）。这可以用于测试目的。
- `GRAPH_TREAT_H213_AS_H212`：用于测试，如果已经是 H213 了，则它将始终被视为 H212。默认为 `false`。
- `GRAPH_PRE_MIGRATION_DUMP_DIR`：在迁移之前导出数据库中的内容的目录。导出将在任何迁移步骤之前发生。默认为无。
- `GRAPH_DISABLE_STATE_CLEANUP`：在启动时，设置此变量以禁用缓存和索引的状态清理。
- `GRAPH_DISABLE_STARTUP_CLEANUP`：设置为 `true` 以禁用启动时的状态清理。
- `GRAPH_CONTRACT_RUNTIME_TRACE`：记录 Solidity 函数调用及其参数。这会导致磁盘 I/O 骤增。
- `GRAPH_DISABLE_BLOCK_INGESTOR_INDEXING`：设置为 `true` 以禁用块摄取期间的索引。这对于速度很慢的以太坊主网同步非常有用，但是在此期间，子图查询结果可能会有所减少。
- `GRAPH_GENESIS_BLOCK`：启用 GENESIS 模式。默认为 `false`。
- `GRAPH_ALLOW_NON_DETERMINISTIC_MIGRATIONS`：允许不确定性迁移。如果此变量未设置，`graph-node` 将拒绝运行不确定的迁移。
- `GRAPH_NODE_EXPORT_BLOCK`：在块处于不良状态时将存储节点导出到本地的路径。默认为 `""`。
- `GRAPH_CHECKPOINT_BLOCK_TRIGGER`：在此块号之后创建快照。这只适用于块源使用区块摄取器的情况。默认为 `0`。
- `GRAPH_DISABLE_CREATE_ANCESTOR`：禁用创建祖先索引。这会使运行时间更快，但是当索引是为空时，内部函数调用可能会失败。
- `GRAPH_GRAPHQL_SKIP_HEAP_CHECK`：禁用 GraphQL 查询中的堆栈使用情况检查。
- `GRAPH_GRAPHQL_SKIP_HEAP_CHECK_AMOUNT`：在禁用堆栈检查时，只有在使用的堆栈超过此值时才会收集查询的性能度量。默认为 0。
- `GRAPH_ENV`：应用程序当前运行的环境。默认为 `development`。
- `GRAPH_NAME`：用于追踪指标的应用程序名称。默认为 `graph-node`。
- `GRAPH_NAMESPACE`：用于追踪指标的命名空间。默认为空。

## 与搜索相关的环境变量

- `GRAPH_ENABLE_SEARCH`：设置为 `false` 以禁用搜索索引。这在测试或不需要搜索的环境中很有用。默认为 `true`。
- `GRAPH_INDEXER_IMPORT_BUCKET_SIZE`：每次导入的索引器数据的大小。默认为 1000。
- `GRAPH_INDEXER_TRIGGER_BUCKET_SIZE`：每次导入的触发器的大小。默认为 1000。
- `GRAPH_INDEXER_IMPORT_THRESHOLD`：触发器数量超过此值时，将触发导入程序。默认为 10,000。
- `GRAPH_INDEXER_EXPORT_BUCKET_SIZE`：导出时每个索引器请求的触发器的数量。默认为 1000。
- `GRAPH_INDEXER_EXPORT_INTERVAL`：导出的间隔时间（毫秒）。默认为 500ms。

## 其他

- `GRAPH_RUNTIME_HOST_NAME`：运行时的主机名。默认为 `host.docker.internal`。
- `GRAPH_RUNTIME_HOST_IP`：用于通信的主机 IP。默认为 `0.0.0.0`。
- `GRAPH_ENABLE_RELAY_HUB_MISSED_CALL_MAILBOX`：启用继电器接口的未接电话邮箱。如果为 `true`，则继电器节点将为丢失的调用创建查询，该查询可能已被注册的子图处理，直到调用重新发送或过期。默认为 `false`。
- `GRAPH_ENABLE_RELAY_HUB_PUSH_NOTIFICATIONS`：启用继电器节点发送电子邮件和 Webhooks 的功能。默认为 `false`。
- `GRAPH_SUBGRAPH_COMMISSIONING_MAX_FAULTY_POTENTIAL_SLOTS`：当计算子图容错时，该变量用于确定在占用的插槽中，多少可以包含故障的子图。默认为 2。

## 子图运行时设置

这些环境变量是在子图映射文件中使用的，它们可以在映射中以 `<env>` 占位符的形式进行替换。

- `GRAPH_NODE`：GraphQL 节点的 URL。
- `GRAPH_ETHEREUM_RPC`：以太坊节点的 URL。
- `GRAPH_IPFS`：IPFS 节点的 URL。
- `GRAPH_NAME`：将在追踪指标中使用的子图名称。
- `GRAPH_NAMESPACE`：将在追踪指标中使用的子图命名空间。
- `GRAPH_DEPLOYMENT_ID`：部署 ID。
- `GRAPH_PEER`：用于 IPC 通信的 Peer ID。
- `GRAPH_START_BLOCK`：开始块。
- `GRAPH_END_BLOCK`：结束块。

## 实验性环境变量

这些环境变量控制实验性功能。它们可能随时更改或删除。

- `GRAPH_RUNTIME_ALLOW_PARALLEL_QUERIES`：允

许对图运行时实例执行多个并行查询。默认为 `false`。
- `GRAPH_RUNTIME_ALLOW_EVICTION`：允许从图运行时删除实例。默认为 `false`。
- `GRAPH_SUBGRAPH_COMMISSIONING`：启用/禁用在映射文件中声明的子图的插槽管理功能。默认为 `false`。
- `GRAPH_ALLOW_NON_DETERMINISTIC_IPFS`：如果 `true`，则在构建索引时允许从 IPFS 获取子图数据。默认为 `false`。
- `GRAPH_EXPERIMENTAL_IPFS_MULTIBASE`：使用实验性 Multibase 格式解析 IPFS 哈希。默认为 `false`。
- `GRAPH_ENABLE_META_GQC`：启用支持查询连接的元数据字段的实验性 GraphQL 功能。默认为 `false`。
- `GRAPH_ALLOW_NON_DETERMINISTIC_INDEXING`：允许以非确定性方式对索引的字段进行排序。默认为 `false`。
- `GRAPH_ENABLE_NON_DETERMINISTIC_DDL`：允许在具有相同名称的字段上运行的DDL更改。默认为 `false`。

## 相关链接

- [GitHub 存储库](https://github.com/graphprotocol/graph-node)
- [编写子图](https://thegraph.com/docs/define-subgraph)
- [运行 Graph Node](https://thegraph.com/docs/quick-start)
- [Subgraph 编译](https://thegraph.com/docs/define-subgraph#compile)

如果需要更多详细信息，请参阅[这里](https://thegraph.com/docs/).

如果需要支持，请在[Graph 论坛](https://forum.thegraph.com/)上发布问题或建议。