---
title: Redis CLI outputs CRLF
date: 2019-06-14 16:06
---

I just lost an hour of my life because a command line tool running on Linux (redis-cli) decided it would be a good idea to output CRLF line separators instead of the standard LF that is used for line separators on Linux.

WTF dude. WTF.
