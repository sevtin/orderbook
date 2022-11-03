# High Performance Order Book for matching engines

[![Go Report Card](https://goreportcard.com/badge/github.com/geseq/orderbook)](https://goreportcard.com/report/github.com/geseq/orderbook) [![GoDoc](https://godoc.org/github.com/geseq/orderbook?status.svg)](https://godoc.org/github.com/geseq/orderbook)  [![Go](https://github.com/geseq/orderbook/actions/workflows/go.yml/badge.svg)](https://github.com/geseq/orderbook/actions/workflows/go.yml)


An experimental low latency, high throughput order book in Go.

## Why?

Becuase it's fun!

## Why Go? What about GC?

This is an experiment to see how far I can take the performance of a full fledged matching engine in Go, and that includes trying to see how much GC will have an affect on latency guarantees, and find ways around them.

## Features

- [x] Simple API
- [x] Standard price-time priority
- [x] Market and limit orders
- [x] Order cancellation. (No in-book updates. Updates will have to be handled with Cancel+Create, and all that entails)
- [x] Stop loss / take profit orders (limit and market)
- [x] AoN, IoC, FoK, etc. Probably not trailing stops. They're probably better handled outside the order book.
- [ ] Snapshot the ordebook state for recovery
- [ ] Handle any GC latency shenanigans
- [ ] Extensive tests and benchmarks
- [ ] Add metrics counters
- [ ] Improve consistent latency
- [x] Extremely high throughput (see below)


## Latency

On 2.1 Ghz Base Freq 12th Gen i& with 5200 MHz LPDD5, Hyperthreading off, Turbo off, AND `GOMAXPROCS=2`

Results are generally similar with test run without OS thread locking or run in a goroutine locked to an OS thread which in turn is run on an isolated CPU core on a standard `-generic` ubuntu kernel. Results are also similar with `GOMAXPROCS=1`

With CPU isolation on only the locked thread, there is very little effect of background processes on the latencies. Without CPU isolation latency measurements are highly sensitive to other running processes. Any background processes will create significant jitter.

Other threads run by the Go scheduler don't seem to make a notable difference whether run on isolated cores or not, as long as they're not all pinned to the same core.

The `nohz_full` column represents tests run with `GOMAXPROCS=1` on a `nohz_full` isolated core on an ubuntu kernel compiled with `nohz_full` and all possible IRQs moved out of that core. The primary thread was pinned to this isolated core and all other threads were left on non-isolated cores.


| AddOrder     | Latency (approx.)     | nohz_full          |
|--------------|-----------------------|--------------------|
|  p50         |  170ns                |  164ns             |
|  p99         |  229ns                |  256ns             |
|  p99.99      |  2.4us                |  2.3us             |
|  p99.9999    |  12us                 |  6.5us             |
|  Max         |  36us                 |  15us              |

| CancelOrder  | Latency (approx.)     | nohz_full          |
|--------------|-----------------------|--------------------|
|  p50         |  35ns                 |  33ns              |
|  p99         |  50ns                 |  50ns              |
|  p99.99      |  86ns                 |  60ns              |
|  p99.9999    |  3.9us                |  3.1us             |
|  Max         |  25us                 |  6.8us             |

## Throughput
 - [x] 12.5 million Order Add/Cancel per second:
   - 2.1 Ghz Base Frequency 12th Gen i7 with 5200 Mhz LPDDR5
   - Turbo Boost disbabled
   - Hyperthreading disabled
  
 - [x] 21 million Order Add/Cancel per second:
   -   2.1 Ghz Base Frequency 12th Gen i7 with 5200 Mhz LPDDR5
   -   Turbo Boost enabled 4.7 Ghz
   -   Hyperthreading disabled

## Limitations

- 8 decimal places due to decimal library used. This should be fine for most use cases.
- Consider [LMAX Disruptor](https://lmax-exchange.github.io/disruptor/) to maintain the throughput while post-processing with a matching engine, although this level of thoughput is probably not necessary for most use cases.

## How do I use this?

You probably shouldn't use this as-is. This is an *experimental* project with the sole goal of optimizing latency and throughput at the expense of everything else.

That said, it should be extremely simple to use this. Create an `OrderBook` object as a starting point.

## There's a bug

Please create an issue. Also PRs are welcome!


