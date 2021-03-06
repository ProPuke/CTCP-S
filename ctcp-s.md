CTCP-S
======

> Version: 1.11

CTCP-S adds a few new CTCPs, and changes a few things about CTCP.

First, unless otherwise stated, there can be at most 1 CTCP in a message.

Second, there can be at most 1 normal message in a CTCP message. A normal message is the concatenation of every non-CTCP part of a message.

This specification does not invalidate the original CTCP specification. Its sole purpose is to extend the original.

How to read examples in this document
-------------------------------------

Examples in this document use `\ddd` where `d` is a decimal digit to indicate a character with that decimal (**NOT** octal!) value, and `\\` to indicate a literal `\`.

Examples
--------

(Note: `ACTION` doesn't generate an automated reply. This property is NOT exclusive to `ACTION`.)

Simple CTCP message:

    PRIVMSG User :\001ACTION Hello!\001

CTCP message with normal message:

    PRIVMSG User :\001ACTION joins\001Hello!

Special CTCP message:

    :Bridge!Bridge@127.0.0.1 PRIVMSG User :\001FORWARD FROM BridgeUser\001Hi there, User!
    :User!User@127.0.0.1 PRIVMSG Bridge :\001FORWARD TO BridgeUser\001Hi there, BridgeUser!

You can also split the message:

    :Bridge!Bridge@127.0.0.1 PRIVMSG User :Hi there, \001FORWARD FROM BridgeUser\001User!
    :User!User@127.0.0.1 PRIVMSG Bridge :Hi there, \001FORWARD TO BridgeUser\001BridgeUser!

Simple CTCP request (& response):

    :Other!Other@127.0.0.1 PRIVMSG User :\001VERSION\001
    :User!User@127.0.0.1 NOTICE Other :\001VERSION MyClient v1.0.0\001

CTCP request (& response) with normal message:

    :Other!Other@127.0.0.1 PRIVMSG User :\001VERSION\001Sorry, I got curious :>
    :User!User@127.0.0.1 NOTICE Other :\001VERSION MyClient v1.0.0\001

Special CTCP request (& response), to user:

    :Bridge!Bridge@127.0.0.1 PRIVMSG User :\001FORWARD FROM BridgeUser\001\001VERSION\001
    :User!User@127.0.0.1 NOTICE Bridge :\001FORWARD TO BridgeUser\001\001VERSION MyClient v1.0.0\001

Special CTCP request (& response), to bridge:

    :User!User@127.0.0.1 PRIVMSG Bridge :\001FORWARD TO BridgeUser\001\001VERSION\001
    :Bridge!Bridge@127.0.0.1 NOTICE User :\001FORWARD FROM BridgeUser\001\001VERSION MyBridge v1.0.0\001

New CTCPs
=========

`FORWARD`
---------

> Since: 1.0  
> See also: [HTTP X-Forwarded-For](https://en.wikipedia.org/wiki/X-Forwarded-For)

The `FORWARD` CTCP is used to indicate the source and/or destination of a message across a bridge.

The syntax for the `FORWARD` CTCP is as follows:

    FORWARD (FROM|TO) <ID>

`<ID>` SHOULD be a nickname or an username, but MAY be an UUID. Formatting is allowed. The use of UUIDs is strongly discouraged, as UUIDs aren't human-readable.

There can be at most 1 `FORWARD FROM` and 1 `FORWARD TO` CTCP in one message. `FORWARD` CTCPs do not count against the 1 CTCP per message limit, and instead have their own limit.

**Side note:** It is possible to use `FORWARD` in a PM or DCC chat to identify a message as belonging to a channel. In which case, it is up to the receiving end to parse any channel modes and guarantee the sender is allowed to send the given message to the channel. The message should be displayed in the channel.

### New in 1.5

As of CTCP-S 1.5, the syntax for the `FORWARD` CTCP is as follows:

    FORWARD (FROM|TO) <ID>[ <ID>...]

This change was made in order to allow forwarding across multiple bridges. Example:

    User PRIVMSG Bridge1 :\001FORWARD TO Bridge2 Bridge3 Other\001something
    Bridge1 PRIVMSG Bridge2 :\001FORWARD TO Bridge3 Other\001\001FOWARD FROM User\001something
    Bridge2 PRIVMSG Bridge3 :\001FORWARD TO Other\001\001FOWARD FROM Bridge1 User\001something
    Bridge3 PRIVMSG Other :\001FOWARD FROM Bridge2 Bridge1 User\001something

`BATCH` and `BATCHEND` and `BATCH+`
-----------------------------------

> Since: 1.3  
> See also: http://ircv3.net/specs/extensions/batch-3.2.html

The `BATCH` and `BATCHEND` CTCPs are used to indicate that a series of messages are related.  
Possible use-cases include eliminating the need for pastebin on IRC. Clients are encouraged to collapse a series of `BATCH` messages.

The syntax for the `BATCH` and `BATCHEND` CTCPs is as follows:

    BATCH <ID>
    BATCHEND <ID>

> Since: 1.11
> 
>     BATCH+ <ID>

`<ID>` is a simple identifier and may contain spaces.

There can be at most 1 `BATCH` or `BATCHEND` or `BATCH+` CTCP in one message. `BATCH` and `BATCHEND` and `BATCH+` CTCPs do not count against the 1 CTCP per message limit. `BATCH+` concatenates with the next `BATCH` (or `BATCHEND` or `BATCH+`).

### Example

     - - // non-batched messages here // - - 
    PRIVMSG User :\001BATCH 1\001public class HelloWorld {
    PRIVMSG User :\001BATCH 1\001    public static void main(String[] args) {
    PRIVMSG User :\001BATCH+ 1\001        System.out.println(
    PRIVMSG User :\001BATCH 1\001"Hello World!");
    PRIVMSG User :\001BATCH 1\001    }
    PRIVMSG User :\001BATCHEND 1\001}
     - - // more non-batched messages here // - - 

Is equivalent to:

     - - // non-batched messages here // - - 
    PRIVMSG User :\001BATCH 1\001public class HelloWorld {
    PRIVMSG User :\001BATCH 1\001    public static void main(String[] args) {
    PRIVMSG User :\001BATCH 1\001        System.out.println("Hello World!");
    PRIVMSG User :\001BATCH 1\001    }
    PRIVMSG User :\001BATCHEND 1\001}
     - - // more non-batched messages here // - - 

**Note:** Because a CTCP-S can appear anywhere in a message, it is possible to evade most server-side CTCP filtering by appending the `BATCH` CTCPs at the middle or at the end of the message, instead of at the start.

`GZIP`
------

> Since: 1.4

The `GZIP` CTCP is used to indicate a GZIP-compressed IRC message.

The syntax for the `GZIP` CTCP is as follows:

    GZIP 0<base64-encoded gzip data>
    GZIP 1<base128-encoded gzip data>
    GZIP 2<raw gzip data>
    GZIP 3<base252-encoded gzip data>

The first variant (`GZIP 0`) uses the standard base64 alphabet as defined by [RFC 4648](https://tools.ietf.org/html/rfc4648), but without padding.

The second variant (`GZIP 1`) uses only the bytes `0x80-0xFF`, sending only 7 bits per byte.

The third variant (`GZIP 2`) quotes the bytes `NUL`, `CR`, `LF`, the CTCP byte, and the quote byte.

The fourth variant (`GZIP 3`) uses all bytes except for `NUL`, `CR`, `LF`, or the CTCP byte. Parsing and generating this variant requries a BigInteger implementation. The alphabet can be generated in Python 3 with: ` bytes([x for x in range(256) if x not in {0, 1, 10, 13}])`. This variant doesn't follow normal quoting rules and the GZIP stream/`GZIP` CTCP stops at the first CTCP byte.

`IGNORED`
---------

> Since: 1.6
>
> DEPRECATED: Use `QUERY IGNORED` instead (see below). Kept because this section defines its syntax.

The `IGNORED` CTCP is used to find out whether you're currently being ignored by the target user.

The syntax for the `IGNORED` CTCP is as follows:

    For queries: IGNORED
    For replies: IGNORED <true|false>

An `IGNORED` reply MUST NOT contain information about the ignore filter. DO NOT, UNDER ANY CIRCUMSTANCES, SEND THE IGNORE MASK, THE IGNORE MODES, OR ANY OTHER INFORMATION ABOUT THE IGNORE IN AN `IGNORED` REPLY!

`FORMAT` or `F`
---------------

> Since: 1.7

The `FORMAT` or `F` inline CTCP is used to add formatting in a standardized way.

The syntax is as follows:

    F <flags>

Where \<flags\> can be any combination of:

| Flag | Notes                                 |
|------|---------------------------------------|
|  b   | Toggle bold on/off                    |
|  i   | Toggle italic on/off                  |
|  u   | Toggle underline on/off               |
|  s   | Toggle strikethrough on/off           |
|  r   | Reset all formatting                  |
|  n   | Swap foreground and background colors |
| FG,BG| Color codes. See below                |
|  ` ` |(empty space) Ignored. Simple separator|

`r` MAY be specified at the same time as other formatting. In which case, all other formatting MUST be ignored.

`n` MAY be used to make the font color transparent, assuming the default background color is transparent. See below.

Unknown flags MUST be ignored. If a flag appears multiple times, they MUST be combined in-order, from left to right.

There can be any amount of `FORMAT`/`F` CTCPs in a message. `FORMAT`/`F` CTCPs do not count against the 1 CTCP per message limit.

### Color codes

Color codes are a flag in the format `FG,BG`, where either side can be omitted.

Omitting either side acts as a no-change. When omitting `BG`, you MUST omit the comma. Omitting both sides (i.e. leaving just
the comma) is equivalent to `99,99`.

There are 2 formats for `FG` and `BG`: `#RRGGBB` and 0-99. The former MUST have all fields present,
while the latter MUST be represented as a single-digit or two-digit number. 00-09 MUST be
treated as 0-9.

Valid values for FG and BG are as follows:

| Value | Description                 |
|-------|-----------------------------|
|#rrggbb| Hexadecimal RGB color code  |
|   0   | White color                 |
|   1   | Black color                 |
|   2   | Blue color                  |
|   3   | Green color                 |
|   4   | Light Red color             |
|   5   | Brown color                 |
|   6   | Purple color                |
|   7   | Orange color                |
|   8   | Yellow color                |
|   9   | Light Green color           |
|  10   | Cyan color                  |
|  11   | Light Cyan color            |
|  12   | Light Blue color            |
|  13   | Pink color                  |
|  14   | Grey color                  |
|  15   | Light Grey color            |
|  99   | Reset Color                 |

Invalid/unknown values should be treated as 99/Reset Color. This includes invalid and incomplete #RRGGBB.

### Examples

    \001F 4\001This line is red.
    \001F bb\001This line is not bold.
    \001F b\001\001F b\001Neither is this one.
    \001F #537465#67616e#6f6772#617068#79210099\001Steganography!

`URI`
-----

> Since: 1.8

The `URI` inline CTCP lets you title URIs.

Syntax:

    URI <URI> <text>

As with other inline CTCPs, there can be any amount of URI CTCPs in a message.

### Special URIs

#### `msg:` URIs

> Since: 1.9

The `msg:` URI sends a message back to sender. Its purpose is to allow for interactive content.

Syntax:

    msg:<message text>[?notice]

The message MUST be sent to the target, and MUST contain: the sender, followed by a colon, `:`, followed by a space, ` `.

If the message containing a `msg:` URI was sent to a channel, the target of the `msg:` is the channel.

If the message containing a `msg:` URI was sent to an user, the target of the `msg:` is the sender.

A `msg:` URI may contain a query string, which is separated by a `?`. If this query string is "notice", then the message MUST
be sent through a `NOTICE` instead of a `PRIVMSG`.

Note that the message text MUST be percent-encoded.

### Examples

    Hello! So I just heard about \001URI http://t.co/whatever example.com\001 and you should check it out!
    \001URI msg:prev%20some_long_id <- Previous Messages\001 \001URI msg:next%20some_other_long_id Next Messages ->\001

`IMAGE`
-------

> Since: 1.8

The `IMAGE` inline CTCP lets you add inline images. Useful for emoticons and emoji.

Syntax:

    IMAGE <URI> <width> <height> <fallback>

`<width>` and `<height>` can be set to `-1` for automatic width and height.

As with other inline CTCPs, there can be any amount of IMAGE CTCPs in a message.

### Examples

    Hey I'm an \001IMAGE http://example.com/aceofhearts.png -1 -1 🂱\001!

`QUERY`
-------

> Since: 1.10

The `QUERY` CTCP deprecates `VERSION`, `TIME`, `USERINFO`, `CLIENTINFO`, `AVATAR` (KVIrc), `IGNORED` (see this document), `DCC`, `FINGER`, and other query CTCPs.

Note: CTCP `PING` is special.

Syntax:

    QUERY
    QUERY <command> <data>
    For replies:
    QUERY QUERY <command list>
    QUERY <command> <data>

With no arguments, `QUERY` generates a reply of the form `QUERY QUERY <command list>`, where `<command list>` is a space-separated list of commands. This reply
should also be generated by `QUERY QUERY`.

There can be only 1 `QUERY` CTCP in a message.

For backwards compatibility, it is strongly recommended to keep replying to the deprecated CTCPs.

### Examples

    :User PRIVMSG Target :\001QUERY\001
    :Target NOTICE User :\001QUERY QUERY QUERY VERSION TIME USERINFO CLIENTINFO AVATAR IGNORED DCC FINGER\001
    :User PRIVMSG Target :\001QUERY QUERY\001
    :Target NOTICE User :\001QUERY QUERY QUERY VERSION TIME USERINFO CLIENTINFO AVATAR IGNORED DCC FINGER\001
    :User PRIVMSG Target :\001QUERY VERSION\001
    :Target NOTICE User :\001QUERY VERSION SomeIRC 1.0 - Linux x86_64\001

`SUB`
-----

> Since: 1.11

The `SUB` CTCP allows you to create (temporary) replacement/substitution "functions".

Syntax:

    SUB <ID> <replacement function>

Where `ID` is a CTCP ID and `replacement function` is a string.

The special syntax for `replacement function` is described below:

- `$$` = literal `$`.
- `${number}` = `number`th argument, where `0` is the CTCP name.
- `${number+}` = `number`th argument onwards.
- `${number1+number2}` = `number1`th argument onwards, up until `number2`th argument, inclusive, where `number1` < `number2`.
- Everything else: literal.

A `replacement function` that doesn't conform to the above syntax should be ignored and silently discarded.

There can be any amount of `SUB` CTCPs in a message.

Notes:

- A `SUB` CTCP can override a previous `SUB` CTCP and/or any user- or client-defined CTCP, except for `SUB` itself.
- A `SUB` CTCP with only an ID (and no space after the ID) clears any previously-defined `SUB` with the same ID.
- A `SUB` CTCP, when used together with `BATCH`, propagates through the same `BATCH` until overriden by another `SUB` CTCP, or until the end of the batch.
- A `SUB` can define another `SUB`.
- A `SUB` may or may not halt. It is currently unknown whether you can determine if a given `SUB` halts. It is also currently unknown whether `SUB` is turing-complete.

### Examples

    :User PRIVMSG Target :\001SUB HUG \\\001ACTION hugs ${1}\\\001\001\001HUG Someone\001\001HUG SomeoneElse\001\001HUG YetAnother\001
    :User PRIVMSG Target :\001SUB A test\001\001A\001
    :User PRIVMSG Target :\001SUB A \\\001SUB ${1} ${2+}\\\001\001\001A TEST Hi!\001\001TEST\001
    :User PRIVMSG Target :\001SUB A \\\001A\\\001\001\001A\001

Are equivalent to:

    :User PRIVMSG Target :\001ACTION hugs Someone\001\001ACTION hugs SomeoneElse\001\001ACTION hugs YetAnother\001
    :User PRIVMSG Target :test
    :User PRIVMSG Target :Hi!
    [never terminates]

Null CTCPs
==========

Null CTCPs, `\001\001`, MAY appear at the start and/or end of a message. They MUST NOT appear in the middle of a message.

Compatibility with previous clients
===================================

For compatibility with previous clients, you MAY append a "null CTCP" `\001\001` at the start/end of a message.
