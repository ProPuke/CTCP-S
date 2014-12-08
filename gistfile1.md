This document describes CTCP vSEx.

CTCP vSEx is basically the CTCP you already know but with a different syntax:  
There can be at most 1 message and 1 CTCP in a single PRIVMSG/NOTICE.

Syntax:

    message[CTCP]SEX[CTCP]message[CTCP]name[CTCP]message[CTCP]arg 1[CTCP]message[CTCP]arg 2[CTCP]message[CTCP]arg n[CTCP]message

where each `[CTCP]` means \001

Basically, the first CTCP in the message has to be "SEX", the next CTCP should be the command, and the following CTCPs should be arguments to the command. With this, both the command and each argument can contain spaces.

message parts are optional and so are CTCP parts. message parts should be concatenated with a space inbetween.

PS: no this isn't a proper RFC yet.