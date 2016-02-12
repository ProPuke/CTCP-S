CTCP-S
======

> Version: 1.7

CTCP-S adds a few new CTCPs, and changes a few things about CTCP.

First, unless otherwise stated, there can be at most 1 CTCP in a message.

Second, there can be at most 1 normal message in a CTCP message. A normal message is the concatenation of every non-CTCP part of a message.

This specification does not invalidate the original CTCP specification. Its sole purpose is to extend the original.

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

`BATCH` and `BATCHEND`
----------------------

> Since: 1.3  
> See also: http://ircv3.net/specs/extensions/batch-3.2.html

The `BATCH` and `BATCHEND` CTCPs are used to indicate that a series of messages are related.  
Possible use-cases include eliminating the need for pastebin on IRC. Clients are encouraged to collapse a series of `BATCH` messages.

The syntax for the `BATCH` and `BATCHEND` CTCPs is as follows:

    BATCH <ID>
    BATCHEND <ID>

`<ID>` is a simple identifier and may contain spaces.

There can be at most 1 `BATCH` or `BATCHEND` CTCP in one message. `BATCH` and `BATCHEND` CTCPs do not count against the 1 CTCP per message limit.

### Example

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

`r` MAY be specified at the same time as other formatting. In which case, all other formatting MUST be ignored.

`n` MAY be used to make the font color transparent, assuming the default background color is transparent. See below.

Unknown flags MUST be ignored. If a flag appears multiple times, they MUST be combined in-order, from left to right.

There can be any amount of `FORMAT`/`F` CTCPs in a message. `FORMAT`/`F` CTCPs do not count against the 1 CTCP per message limit.

### Color codes

Color codes are a flag in the format FG,BG, where either side can be omitted.

Omitting either side acts as a no-change. When omitting BG, you MAY omit the comma.

There are 2 formats for FG and BG: #RRGGBB and 0-99. The former MUST have all fields present,
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
    \001F #537465#67616e#6f6772#617068#792100\001Steganography!

Null CTCPs
==========

Null CTCPs, `\001\001`, MAY appear at the start and/or end of a message. They MUST NOT appear in the middle of a message.

Compatibility with previous clients
===================================

For compatibility with previous clients, you MAY append a "null CTCP" `\001\001` at the start/end of a message.
