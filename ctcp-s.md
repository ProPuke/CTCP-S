CTCP-S
======

> Version: 1.3

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

`BATCH` and `BATCHEND`
----------------------

> Since: 1.3  
> See also: http://ircv3.net/specs/extensions/batch-3.2.html

The `BATCH` and `BATCHEND` CTCPs are used to indicate that a series of messages are related.  
Possible use-cases include eliminating the need for pastebin on IRC.

The syntax for the `BATCH` and `BATCHEND` CTCPs is as follows:

    BATCH <ID>
    BATCHEND <ID>

`<ID>` is a simple identifier and may contain spaces.

There can be at most 1 `BATCH` or `BATCHEND` CTCP in one message. `BATCH` and `BATCHEND` CTCPs do not count against the 1 CTCP per message limit.

### Example

     - - // non-batched messages here // - - 
    PRIVMSG User :\BATCH 1\001public class HelloWorld {
    PRIVMSG User :\BATCH 1\001    public static void main(String[] args) {
    PRIVMSG User :\BATCH 1\001        System.out.println("Hello World!");
    PRIVMSG User :\BATCH 1\001    }
    PRIVMSG User :\BATCHEND 1\001}
     - - // more non-batched messages here // - - 

**Note:** Because a CTCP-S can appear anywhere in a message, it is possible to evade most server-side CTCP filtering by appending the `BATCH` CTCPs at the middle or at the end of the message, instead of at the start.

Null CTCPs
==========

Null CTCPs, `\001\001`, MAY appear at the start and/or end of a message. They MUST NOT appear in the middle of a message.

Compatibility with previous clients
===================================

For compatibility with previous clients, you MAY append a "null CTCP" `\001\001` at the start/end of a message.