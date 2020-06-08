# syslog Parser Plugin

The `syslog` parser plugin parses syslog generated logs. This plugin
supports two RFC formats, rfc3164 and rfc5424.


## Parameters

See [Parse section configurations](/configuration/parse-section.md)


### time\_format

| type   | default         | version |
|:-------|:----------------|:--------|
| string | %b %d %H:%M:%S  | 0.14.10 |

Specify time format for event time. Default is "%b %d %H:%M:%S" for
rfc3164 protocol. If your log uses sub-second timestamp, change this parameter to like "%b %d %H:%M:%S.%N".


### rfc5424\_time\_format

| type   | default                 | version |
|:-------|:------------------------|:--------|
| string | %Y-%m-%dT%H:%M:%S.%L%z  | 0.14.14 |

Specify time format for event time for rfc5424 protocol.


### message\_format

| type | default | available values     | version |
|:-----|:--------|:---------------------|:--------|
| enum | rfc3164 | rfc3164/rfc5424/auto | 0.14.14 |

Specify protocol format. Supported values are `rfc3164`, `rfc5424` and
`auto`. Default is `rfc3164`. If your syslog uses `rfc5424`, use
`rfc5424` instead.

`auto` is useful when this parser receives both `rfc3164` and `rfc5424`
message. `syslog` parser detects message format by using message prefix.


### with\_priority

| type | default | version |
|:-----|:--------|:--------|
| bool | false   | 0.14.0  |

If the incoming logs have priority prefix, e.g. `<9>`, set `true`.
Default is `false`.

This parameter is used inside `in_syslog` plugin because the file logs
via syslog don't have `<9>` like priority prefix.


### parser\_type

| type | default | available values | version |
|:-----|:--------|:-----------------|:--------|
| enum | regexp  | regexp/string    | 1.7.1(for rfc3164)/1.11.0(for rfc5424) |

Specify internal parser type for `rfc3164`/`rfc5424` format. Supported values are `regexp` and `string`.
Both parsers generate same record for standard format.

If `regexp` doesn't fit your logs, consider `string` type instead.

We recommend to use `string` parser because it is 2x faster than `regexp`. The default is `regexp` for existing users.
Fluentd v2 will change the default to `string` parser.


### support\_colonless\_ident

| type | default | version |
|:-----|:--------|:--------|
| bool | true    | 1.7.1   |

This parameter is used when `parser_type` is `string`. If your message doesn't contain ident field, set `false` to avoid ident mismatch.

```
# No ident field log
Feb  5 17:32:18 10.0.0.99 Use the BFG!

# generated record with true is wrong
{"host":"10.0.0.99","ident":"Use","message":"the BFG!"}

# generated record with false is correct
{"host":"10.0.0.99","message":"Use the BFG!"}
```


## Regexp patterns

Show regexp patterns for parsing logs.

### rfc3164 pattern

```
expression /^\<(?<pri>[0-9]+)\>(?<time>[^ ]* {1,2}[^ ]* [^ ]*) (?<host>[^ ]*) (?<ident>[^ :\[]*)(?:\[(?<pid>[0-9]+)\])?(?:[^\:]*\:)? *(?<message>.*)$/
time_format "%b %d %H:%M:%S"
```

`pri`, `host`, `ident`, `pid` and `message` are included in the event
record. `time` is used for the event time.

`pri` value is converted into integer type.

If `with_priority` is `false`, `^\<(?<pri>[0-9]+)\>` is removed from the pattern.


### rfc5424 pattern

```
expression /\A\<(?<pri>[0-9]{1,3})\>[1-9]\d{0,2} (?<time>[^ ]+) (?<host>[!-~]{1,255}) (?<ident>[!-~]{1,48}) (?<pid>[!-~]{1,128}) (?<msgid>[!-~]{1,32}) (?<extradata>(?:\-|(?:\[.*?(?<!\\)\])+))(?: (?<message>.+))?\z/
time_format "%Y-%m-%dT%H:%M:%S.%L%z"
```

`pri`, `host`, `ident`, `pid`, `msgid`, `extradata` and `message` are
included in the event record. `time` is used for the event time.

`pri` value is converted into integer type.

If `with_priority` is `false`, `\<(?<pri>[0-9]{1,3})\>[1-9]\d{0,2}` is removed from the pattern.


## Example


### rfc3164 log

```
<6>Feb 28 12:00:00 192.168.0.1 fluentd[11111]: [error] Syslog test
```

This incoming event is parsed as:

```
time:
1362020400 (Feb 28 12:00:00)

record:
{
  "pri"    : 6,
  "host"   : "192.168.0.1",
  "ident"  : "fluentd",
  "pid"    : "11111",
  "message": "[error] Syslog test"
}
```


### rfc5424 log

```
<16>1 2013-02-28T12:00:00.003Z 192.168.0.1 fluentd 11111 ID24224 [exampleSDID@20224 iut="3" eventSource="Application" eventID="11211"] Hi, from Fluentd!
```

This incoming event is parsed as:

```
time:
1362052800 (2013-02-28T12:00:00.003Z)

record:
{
  "pri"      : 16,
  "host"     : "192.168.0.1",
  "ident"    : "fluentd",
  "pid"      : "11111",
  "msgid"    : "ID24224",
  "extradata": "[exampleSDID@20224 iut=\"3\" eventSource=\"Application\" eventID=\"11211\"]",
  "message"  : "Hi, from Fluentd!"
}
```


------------------------------------------------------------------------

If this article is incorrect or outdated, or omits critical information, please [let us know](https://github.com/fluent/fluentd-docs-gitbook/issues?state=open).
[Fluentd](http://www.fluentd.org/) is a open source project under [Cloud Native Computing Foundation (CNCF)](https://cncf.io/). All components are available under the Apache 2 License.
