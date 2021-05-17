---
machine_translated: true
machine_translated_rev: 5decc73b5dc60054f19087d3690c4eb99446a6c3
toc_priority: 52
toc_title: "\u7CFB\u7EDF\u8868"
---

# 系统表 {#system-tables}

## 引言 {#system-tables-introduction}

系统表能够提供以下信息:

-   服务器的状态、进程以及环境。
-   服务器的内部进程。

系统表:

-   存储于 `system` 数据库中。
-   仅提供数据读取功能。
-   无法被删除或更改，但可以对其进行detach操作。

大多数系统表将表中数据存储在RAM中。 任何一个ClickHouse服务在刚开始启动时都会创建此类系统表。

与其他系统表不同，包括 [metric_log](../../operations/system-tables/metric_log.md#system_tables-metric_log), [query_log](../../operations/system-tables/query_log.md#system_tables-query_log), [query_thread_log](../../operations/system-tables/query_thread_log.md#system_tables-query_thread_log), [trace_log](../../operations/system-tables/trace_log.md#system_tables-trace_log), [part_log](../../operations/system-tables/part_log.md#system.part_log), crash_log and text_log 在内记录系统日志的表默认采用[MergeTree](../../engines/table-engines/mergetree-family/mergetree.md) 引擎并在默认情况下将其数据存储在文件系统中。 如果存储于文件系统的表被人为删除，那么ClickHouse服务器会在下一次进行数据写入时重新创建一张空表。 如果系统表结构在新版本中发生更改，那么ClickHouse会重命名当前表并创建一个新表。

用户可以通过在`/etc/clickhouse-server/config.d/`下创建与系统表同名的配置文件, 或者在`/etc/clickhouse-server/config.xml`中设置相应配置项，来自定义系统日志表的结构。可以自定义的配置项如下:

-   `database`: 系统日志表所在的数据库。这个选项现在已经被抛弃了。所有的系统日表存储在`system`库中。
-   `table`: 存储系统日志数据的表。
-   `partition_by`: 指定[PARTITION BY](../../engines/table-engines/mergetree-family/custom-partitioning-key.md)表达式。
-   `ttl`: 指定系统日志表TTL选项。
-   `flush_interval_milliseconds`: 指定将系统日志表数据刷新到磁盘的时间间隔。
-   `engine`: 指定完整的表引擎定义。(以`ENGINE = `开头)。 这个选项与`partition_by`以及`ttl`冲突。如果engine和其两者一起设置，服务启动时会抛出异常并且退出。

配置定义的示例如下：

```
<yandex>
    <query_log>
        <database>system</database>
        <table>query_log</table>
        <partition_by>toYYYYMM(event_date)</partition_by>
        <ttl>event_date + INTERVAL 30 DAY DELETE</ttl>
        <!--
        <engine>ENGINE = MergeTree PARTITION BY toYYYYMM(event_date) ORDER BY (event_date, event_time) SETTINGS index_granularity = 1024</engine>
        -->
        <flush_interval_milliseconds>7500</flush_interval_milliseconds>
    </query_log>
</yandex>
```

默认情况下，表增长是无限的。 要控制表的大小，可以使用 TTL 删除过期日志记录的设置。 你也可以利用`MergeTree`系列引擎表的分区特征。

## 系统指标的来源 {#system-tables-sources-of-system-metrics}

用于收集ClickHouse服务器使用的系统指标:

-   `CAP_NET_ADMIN` 能力。
-   [procfs](https://en.wikipedia.org/wiki/Procfs) （仅在Linux中）。

**procfs**

如果ClickHouse服务器没有 `CAP_NET_ADMIN` 功能，它试图退回到 `ProcfsMetricsProvider`. `ProcfsMetricsProvider` 允许收集每个查询的系统指标（用于CPU和I/O）。

如果系统上支持并启用procfs，ClickHouse server将收集如下指标:

-   `OSCPUVirtualTimeMicroseconds`
-   `OSCPUWaitMicroseconds`
-   `OSIOWaitMicroseconds`
-   `OSReadChars`
-   `OSWriteChars`
-   `OSReadBytes`
-   `OSWriteBytes`

[原始文章](https://clickhouse.tech/docs/en/operations/system-tables/) <!--hide-->
