---
title: Location headers shouldn't 404, Jenkins
---

What's wrong with this picture?

```bash
@host ~ $ curl -v -k "https://jenkins.stupid.internal.domain/jenkins/job/some-bullshit/buildWithParameters?foo=bar&token=REDACTED" 2>&1|grep Location
< Location: https://jenkins.stupid.internal.domain/queue/item/231369/
@host ~ $ curl -k https://jenkins.stupid.internal.domain/queue/item/231369/ 2>&1 |grep "< HTTP"
< HTTP/1.1 404 Not Found
@host ~ $
```

Seems like the URL you send back in a `Location:` header should exist, right? Whoops, you have to know the Jenkins Secret Handshake to actually get what you want:

```bash
@host ~ $ curl -v -k "https://jenkins.stupid.internal.domain/queue/item/231408/api/json" 2>&1 | grep "< HTTP"
< HTTP/1.1 200 OK
```

Ok, filing that one away in my bin of tribal knowledge that is taking up so much space in by brain I can't remember where my damn watch charger is.