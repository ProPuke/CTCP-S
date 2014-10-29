This document describes CTCP vSEx.

CTCP vSEx is basically the CTCP you already know but with a different syntax:  
There can be at most 1 message and 1 CTCP in a single PRIVMSG/NOTICE.

Syntax:

message[CTCP]name[CTCP]message[CTCP]arg 1[CTCP]message[CTCP]arg 2[CTCP]message[CTCP]arg n[CTCP]message

message parts are optional and so are CTCP parts. message parts should be concatenated with a space inbetween.

PS: no this isn't a proper RFC yet.