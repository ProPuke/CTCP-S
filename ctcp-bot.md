CTCP-BOT
========

> Version: 1.0

CTCP-BOT is a set of CTCPs for bot control.

CTCP-BOT requires CTCP-S.

New CTCPs
=========

`BOT`
-----

> Since: 1.0

The `BOT` CTCP allows you to filter a message to a specific bot.

Syntax:

    BOT <bot nick, as seen by the user>

This allows you to avoid botspam when multiple bots use the same prefix. A comma-separated list of nicks may also be used.

There is no limit to how many `BOT` CTCPs may appear in a message. In case of 2 or more `BOT` CTCPs in the same message, they should all be treated as a single `BOT` CTCP with a comma-separated list of nicks.

`COMMAND` and `CMD`
-------------------

> Since: 1.0

The `COMMAND` CTCP allows you to specify a command name.

Syntax:

    COMMAND <name>

This removes the need for a prefix. Bots SHOULD NOT require a `BOT` CTCP to be present together with a `COMMAND` CTCP.

`CMD` is a short alias for `COMMAND`.

There can be at most 1 `COMMAND` CTCP in a message.

`ARGUMENT` and `ARG`
--------------------

> Since: 1.0

The `ARGUMENT` CTCP allows you to specify a command argument.

Syntax:

    ARGUMENT <data>

A `COMMAND` or `CMD` CTCP MUST appear if an `ARGUMENT` CTCP appears.

`ARG` is a short alias for `ARGUMENT`.

There is no limit to how many `ARGUMENT` CTCPs may appear in a message.
