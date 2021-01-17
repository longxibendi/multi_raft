![dragonboat](./docs/dragonboat.jpg)
# Dragonboat - A Multi-Group Raft library in Go / [中文版](README.CHS.md) ##
[![license](http://img.shields.io/badge/license-Apache2-blue.svg)](https://github.com/lni/dragonboat/blob/master/LICENSE)
![Build status](https://github.com/lni/dragonboat/workflows/Test/badge.svg?branch=master)
[![Go Report Card](https://goreportcard.com/badge/github.com/lni/dragonboat)](https://goreportcard.com/report/github.com/lni/dragonboat)
[![codecov](https://codecov.io/gh/lni/dragonboat/branch/master/graph/badge.svg)](https://codecov.io/gh/lni/dragonboat)
[![Godoc](http://img.shields.io/badge/go-documentation-blue.svg)](https://godoc.org/github.com/lni/dragonboat)
[![Join the chat at https://gitter.im/lni/dragonboat](https://badges.gitter.im/Join%20Chat.svg)](https://gitter.im/lni/dragonboat?utm_source=badge&utm_medium=badge&utm_campaign=pr-badge&utm_content=badge)

## News ##
* 2020-03-05 Dragonboat v3.2 has been released, please check [CHANGELOG](CHANGELOG.md) for details.
* 2019-07-04 Dragonboat v3.1 has been released, please read [CHANGELOG](CHANGELOG.md) before you upgrade.
* 2019-06-21 Dragonboat v3.0 has been released with on disk state machine and Go module support ([CHANGELOG](CHANGELOG.md)).

## About ##
Dragonboat is a high performance multi-group [Raft](https://raft.github.io/) [consensus](https://en.wikipedia.org/wiki/Consensus_(computer_science)) library in pure [Go](https://golang.org/).

Consensus algorithms such as Raft provides fault-tolerance by alllowing a system continue to operate as long as the majority member servers are available. For example, a Raft cluster of 5 servers can make progress even if 2 servers fail. It also appears to clients as a single entity with strong data consistency always provided. All Raft replicas can be used to handle read requests for aggregated read throughput.

Dragonboat handles all technical difficulties associated with Raft to allow users to just focus on their application domains. It is also very easy to use, our step-by-step [examples](https://github.com/lni/dragonboat-example) can help new users to master it in half an hour.

## Features ##
* Easy to use pure-Go APIs for building Raft based applications
* Feature complete and scalable multi-group Raft implementation
* Disk based and memory based state machine support
* Fully pipelined and TLS mutual authentication support, ready for high latency open environment
* Custom Raft log storage and transport support, easy to integrate with latest I/O techs
* Prometheus based health metrics support
* Built-in tool to repair Raft clusters that permanently lost the quorum
* [Extensively tested](/docs/test.md) including using [Jepsen](https://aphyr.com/tags/jepsen)'s [Knossos](https://github.com/jepsen-io/knossos) linearizability checker, some results are [here](https://github.com/lni/knossos-data)

Most features covered in Diego Ongaro's [Raft thesis](https://ramcloud.stanford.edu/~ongaro/thesis.pdf) have been supported -
* leader election, log replication, snapshotting and log compaction
* membership change
* ReadIndex protocol for read-only queries
* leadership transfer
* non-voting member
* witness member
* idempotent update transparent to applications
* batching and pipelining
* disk based state machine

## Performance ##
Dragonboat is the __fastest__ open source multi-group Raft implementation on Github. 

For 3-nodes system using mid-range hardware (details [here](docs/test.md)) and in-memory state machine, when RocksDB is used as the storage engine, Dragonboat can sustain at 9 million writes per second when the payload is 16bytes each or 11 million mixed I/O per second at 9:1 read:write ratio. High throughput is maintained in geographically distributed environment. When the RTT between nodes is 30ms, 2 million I/O per second can still be achieved using a much larger number of clients.
![throughput](./docs/throughput.png)

The number of concurrent active Raft groups affects the overall throughput as requests become harder to be batched. On the other hand, having thousands of idle Raft groups has a much smaller impact on throughput.
![nodes](./docs/nodes.png)

Table below shows write latencies in millisecond, Dragonboat has <5ms P99 write latency when handling 8 million writes per second at 16 bytes each. Read latency is lower than writes as the ReadIndex protocol employed for linearizable reads doesn't require fsync-ed disk I/O.

|Ops|Payload Size|99.9% percentile|99% percentile|AVG|
|:-:|:----------:|:--:|:-:|:-:|
|1m|16|2.24|1.19|0.79|
|1m|128|11.11|1.37|0.92|
|1m|1024|71.61|25.91|3.75|
|5m|16|4.64|1.95|1.16|
|5m|128|36.61|6.55|1.96|
|8m|16|12.01|4.65|2.13|

When tested on a single Raft group, Dragonboat can sustain writes at 1.25 million per second when payload is 16 bytes each, average latency is 1.3ms and the P99 latency is 2.6ms. This is achieved when using an average of 3 cores (2.8GHz) on each server.

As visualized below, Stop-the-World pauses caused by Go1.11's GC are sub-millisecond on highly loaded systems. Such very short Stop-the-World pause time is further significantly reduced in Go 1.12. Golang's runtime.ReadMemStats reports that less than 1% of the available CPU time is used by GC on highly loaded system.
![stw](./docs/stw.png)

## Requirements ##
* x86_64/Linux, x86_64/MacOS or ARM64/Linux, Go 1.15 or 1.14

## Getting Started ##
__Master is our unstable branch for development. Please use the latest released versions for any production purposes.__ For Dragonboat v3.2.x, please follow the instructions in v3.2.x's [README.md](https://github.com/lni/dragonboat/blob/release-3.2/README.md). 

Go 1.14 or above with [Go module](https://github.com/golang/go/wiki/Modules) support is required.

To use Dragonboat, make sure to import the package __github.com/lni/dragonboat/v3__. Also add "github.com/lni/dragonboat/v3 v3.2.0" to the __require__ section of your project's go.mod file.

By default, [Pebble](https://github.com/cockroachdb/pebble) is used for storing Raft Logs in Dragonboat. RocksDB and other storage engines are also supported, more info [here](docs/storage.md).

You can also follow our [examples](https://github.com/lni/dragonboat-example) on how to use Dragonboat. 

## Documents ##
[FAQ](https://github.com/lni/dragonboat/wiki/FAQ), [docs](https://godoc.org/github.com/lni/dragonboat), step-by-step [examples](https://github.com/lni/dragonboat-example), [DevOps doc](docs/devops.md), [CHANGELOG](CHANGELOG.md) and [online chat](https://gitter.im/lni/dragonboat) are available.

## Examples ##
Dragonboat examples are [here](https://github.com/lni/dragonboat-example).

## Status ##
Dragonboat is production ready.

## Contributing ##
For reporting bugs, please open an [issue](https://github.com/lni/dragonboat/issues/new). For contributing improvements or new features, please send in the pull request.

## License ##
Dragonboat is licensed under the Apache License Version 2.0. See LICENSE for details.

Third party code used in Dragonboat and their licenses is summarized [here](docs/COPYRIGHT).
