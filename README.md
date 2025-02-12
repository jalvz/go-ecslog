# ecslog

A CLI for pretty-printing (and filtering) of [ecs-logging](https://www.elastic.co/guide/en/ecs-logging/overview/master/intro.html)
formatted log files.


# Install

For homebrew users:

    brew install trentm/tap/ecslog

Or download a pre-built binary package from [the releases page](https://github.com/trentm/go-ecslog/releases)
and copy to somewhere on your PATH.

Or you can build from source via:

    git clone git@github.com:trentm/go-ecslog.git
    cd go-ecslog
    make
    ./ecslog --version


# Goals

- Easy to install and use.
- Fast.
- Reliably handles any ECS input and doesn't crash.
- Colors are decent on dark *and* light backgrounds. Many hacker-y tools
  messy this up.

Nice to haves:

- Configurable/pluggable output formatting would be nice.
- Filtering support: levels, other fields.
- `less` integration a la `bunyan`
- Stats collection and reporting, if there are meaningful common cases
  here. Otherwise this could get out of hand.

Non-goals:

- An ambitious CLI that handles multiple formats (e.g. also bunyan format, pino
  format, etc). Leave that for a separate science project.
- Full less-like curses-based TUI for browsing around in a log file, though
  that would be fun.


# Output formats

`ecslog` has multiple output formats for rendering ECS logs that may be selected
via the `-f, --format NAME` option. Note that some formats as *lossy*, i.e.
elide some fields, typically for compactness.

- "default": A lossless default format that renders each log record with a
  title line to convey core and common fields, followed by all remaining
  extra fields. Roughly:

  ```
  [@timestamp] LOG.LEVEL (log.logger/service.name on host.hostname): message
      extraKey1: extraValue1-as-multiline-jsonish
      extraKey2: extraValue2-as-multiline-jsonish
  ```

  where "multiline jsonish" means 4-space-indented JSON with the one special
  case that multiline string values are printed indented and with newlines.
  For example, "error.stack\_trace" in the following:

  ```
  [2021-02-11T06:24:53.251Z]  WARN (myapi on purple.local): something went wrong
      process: {
          "pid": 82240
      }
      error: {
          "type": "Error"
          "message": "boom"
          "stack_trace":
              Error: boom
                  at .../pino/examples/express-simple.js:67:15
                  ...
  ```

  The format of the title line may change in future versions.

- "ecs": The native/raw ECS format, ndjson.

- "simple": A *lossy* (i.e. elides some fields for compactness) format that
  simply renders `LOG.LEVEL: message`. If extra fields (other than the core
  "@timestamp" and "ecs.version" fields) are being elided, a ellipsis is
  appended to the line.

- "compact": A lossless format similar to "default", but attempts are made
  to make the "extraKey" info more compact by balancing multiline JSON with
  80-column output.

- "http": A lossless format similar to "default", but attempts to render
  HTTP-related ECS fields in HTTP request and response text representation.
  TODO: not yet implemented.


# Config file

Some `ecslog` options can be set via a "~/.ecslog.toml" file. For example:

```
# Set the output format (equivalent of `-f, --format` option).
# Valid values are: "default" (the default), "compact", "ecs", "simple"
format="compact"

# Whether output should be colorized.
# Valid values are: "auto" (the default), "yes", "no".
color="auto"

# Set the maximum number of bytes long for a single line that will be
# considered for processing. Longer lines will be treated as if they are
# not ecs-logging records.
# Valid values are: -1 (to use the default 16384), or a value between 1
# and 1048576 (inclusive).
maxLineLen=32768
```

See https://toml.io/ for TOML syntax information.
The `--no-config` option can be used to ignore a possible "~/.ecslog.toml"
config file.


# Troubleshooting

The `ECSLOG_DEBUG` environment variable can be set to get some internal
debugging information on stderr. For example:

    ECSLOG_DEBUG=1 ecslog ...

Internal debug logging is disabled if `ECSLOG_DEBUG` is unset, or is set
to one of: the empty string, "0", or "false".
