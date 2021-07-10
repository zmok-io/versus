# Versus

Versus takes a stream of requests and runs them against multiple endpoints
simultaneously, comparing the output and timing.

## Setup

[Grab a binary release](https://github.com/INFURA/versus/releases) or build from source.


## Usage

```
Usage:
  versus [OPTIONS] [endpoint...]

Application Options:
      --timeout=     Abort request after duration (default: 30s)
      --stop-after=  Stop after N requests per endpoint, N can be a number or duration.
      --concurrency= Concurrent requests per endpoint (default: 1)
  -v, --verbose      Show verbose logging.
      --version      Print version and exit.

Help Options:
  -h, --help         Show this help message

Arguments:
  endpoint:          API endpoint to load test, such as "http://localhost:8080/"
```

By default, HTTP endpoints will POST their requests. Versus is designed to be
used with a stream of JSONRPC requests. For example,
[ethspam](https://github.com/shazow/ethspam) can be used to generate realistic
Ethereum JSONRPC requests.

For example:

```
$ export ZMOK_APP_ID="..."
$ ethspam | versus --stop-after=200 --concurrency=20 "https://api.zmok.io/mainnet/${ZMOK_APP_ID}"
Endpoints:

0. "https://api.zmok.io/mainnet/XXX"

   Requests:   573.17 per second
   Timing:     0.0349s avg, 0.0094s min, 0.1576s max
               0.0273s standard deviation

   Percentiles:
     25% in 0.0204s
     50% in 0.0273s
     75% in 0.0331s
     90% in 0.0799s
     95% in 0.1104s
     99% in 0.1236s

   Errors: 0.00%

** Summary for 1 endpoints:
   Completed:  200 results with 200 total requests
   Timing:     34.893493ms request avg, 516.87595ms total run time
   Errors:     0 (0.00%)
   Mismatched: 0
```

Similarly, we can run versus against multiple endpoints and each response body will be compared to match.

```
$ ethspam | versus --stop-after=200 --concurrency=20 "https://api.zmok.io/mainnet/${ZMOK_APP_ID}" "https://cloudflare-eth.com"
Endpoints:

0. "https://api.zmok.io/mainnet/XXX"

   Requests:   468.64 per second
   Timing:     0.0427s avg, 0.0084s min, 0.1740s max
               0.0350s standard deviation

   Percentiles:
     25% in 0.0242s
     50% in 0.0318s
     75% in 0.0423s
     90% in 0.1014s
     95% in 0.1311s
     99% in 0.1712s

   Errors: 0.00%

1. "https://cloudflare-eth.com"

   Requests:   160.35 per second, 157.61 per second for errors
   Timing:     0.1247s avg, 0.0048s min, 0.4648s max
               0.0851s standard deviation

   Percentiles:
     25% in 0.0940s
     50% in 0.1101s
     75% in 0.1402s
     90% in 0.2966s
     95% in 0.3215s
     99% in 0.3913s

   Errors: 9.50%
     19 × "bad status code: 429"

** Summary for 2 endpoints:
   Completed:  200 results with 400 total requests
   Timing:     83.702623ms request avg, 1.611950277s total run time
   Errors:     19 (4.75%)
   Mismatched: 57
```

Note that there was one response mismatched out of the 500 iterations. If we
run versus with verbose flags (`-v` or `-vv`), then mismatched bodies will be
printed.

### Caveats

Things to keep in mind while using versus and reading the reports:

- Mismatched results are not always bad, often it's just a matter of JSON
  key ordering or formatting or some extra attributes. Future versions of
  versus could do a better job about parsing and comparing JSON subsets.
- Your latency (ping) to the endpoint you're benchmarking is included in the
  timing. When comparing multiple endpoints, be mindful that the latency to
  each endpoint could vary.
- Pay attention to the standard deviation in timing, that's a good hint about
  the variance between the easiest and the hardest request during the
  benchmark, regardless of fixed latency.
- While HTTP connections are reused, the extra time to spin up a fresh
  connection at the beginning is also included. With more concurrency, make
  sure to use a higher iteration count so that the effect is not as pronounced.
  For example, 50 iterations at 50 concurrency, practically every iteration
  will end up creating a fresh socket and no connection reuse will occur.

There may be ways to improve the benchmark process to account for some of these
caveats, please open an issue with ideas for pull requests!

## Goals

Features:

- [x] Run against a single endpoint or many.
- [x] Run against local or remote endpoints.
- [x] Real-time parallel test execution.
- [ ] Compare results across separately-run tests
- [ ] Use live-streaming tcpdump data as test payloads

Compare between endpoints:

- [x] Response integrity: Body, status
- [x] Duration
- [x] Throughput


## License

MIT
