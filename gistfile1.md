CTCP vSEx
=========

CTCP vSEx adds a few new CTCPs, and changes a few things about CTCP.

First, unless otherwise stated, there can be at most 1 CTCP in a message.

Second, you may have at most 1 normal message in a CTCP message.

Examples
--------

(Note: ACTION doesn't generate an automated reply. This property is NOT exclusive to ACTION.)

Simple CTCP message:

    PRIVMSG User :\001ACTION Hello!\001

CTCP message with normal message:

    PRIVMSG User :\001ACTION joins\001Hello!

Special CTCP message:

    :Bridge!Bridge@127.0.0.1 PRIVMSG User :\001FORWARD FROM BridgeUser\001Hi there, User!
    :User!User@127.0.0.1 PRIVMSG Bridge :\001FORWARD TO BridgeUser\001Hi there, BridgeUser!

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

FORWARD
-------

The syntax for the FORWARD CTCP is as follows:

    FORWARD (FROM|TO) <ID>

<ID> SHOULD be a nickname or an username.

There can be at most 2 FORWARD CTCPs in one message. FORWARD CTCPs do not count against the 1 CTCP per message limit.