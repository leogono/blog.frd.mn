---
title: "Backscatter due to loopback MX"
date: 2015-03-23T17:05:00.000Z
disqus_id: 20
slug: backscatter-due-to-loopback-mx
---

At work we have a medium sized shared web-hosting platform which consists out of two main parts: one database/web (MySQL and Apache) server and one mail (Postfix and Dovecot) server.

Recently the web server ended up fairly often on (DNSBL/RBL) blacklists, so I tried to  figure out what's the actual reason for this. Turns out the root of the problem was [backscatter](http://en.wikipedia.org/wiki/Backscatter_%28email%29). And since it was quite thrilling to troubleshoot, I thought I should write a little blog post about it.

# Investigation

The first thing I did was to look through Postfix's `mail.log` and search for anomalies. I couldn't find anything promising while skimming through it so I grep'ed for the timestamp shown in the blacklist listing to search for any violations in the logs at that given time. And who'd have thought? We found the culprit:

```shell
$ grep "8170A4D23E6F" /var/log/mail.log
postfix/smtpd[18858]: 8170A4D23E6F: client=localhost[127.0.0.1]
postfix/cleanup[18861]: 8170A4D23E6F: message-id=<rW9zoq-dylj8W-rZ@correspondencect.web.company.de>
postfix/qmgr[10478]: 8170A4D23E6F: from=<antonettabaune@godetia.com>, size=1144, nrcpt=1 (queue active)
postfix/smtp[18862]: 8170A4D23E6F: to=<jfhfdq@hot.de>, relay=none, delay=0.06, delays=0.03/0/0.03/0, dsn=5.4.6, status=bounced (mail for hot.de loops back to myself)
postfix/bounce[18863]: 8170A4D23E6F: sender non-delivery notification: 955C04D2300E
postfix/qmgr[10478]: 8170A4D23E6F: removed
```

The bounced message matches the timestamp of the DNSBL exactly, probably the mail that caused the listing.

```shell
$ cat /var/log/mail.log* | grep "loops back to myself"
```

You maybe also want to search through compressed (log rotated) log files as well:

```shell
$ zcat /var/log/mail.log*.gz | grep "loops back to myself"
postfix/smtp[15520]: 56F464D343AF: to=<treehnh@iphone.com>, relay=mail.iphone.com[127.0.0.6]:25, delay=0.36, delays=0.03/0/0.33/0, dsn=5.4.6, status=bounced (mail for iphone.com loops back to myself)
postfix/smtp[15520]: 84BA23D801C7: to=<lulu21@hotmaol.de>, relay=none, delay=0.06, delays=0.03/0/0.04/0, dsn=5.4.6, status=bounced (mail for hotmaol.de loops back to myself)
postfix/smtp[15520]: B41E04D82877: to=<gdhfgkj@hotmaol.de>, relay=none, delay=0.02, delays=0.02/0/0/0, dsn=5.4.6, status=bounced (mail for hotmaol.de loops back to myself)
postfix/smtp[15520]: 620384D830C0: to=<roberlo@hahoo.es>, relay=none, delay=0.39, delays=0.02/0/0.37/0, dsn=5.4.6, status=bounced (mail for hahoo.es loops back to myself)
postfix/smtp[17135]: 506EB4D830F6: to=<kl@beer.com>, relay=none, delay=0.22, delays=0.03/0/0.19/0, dsn=5.4.6, status=bounced (mail for beer.com loops back to myself)
postfix/smtp[25622]: 6404C4D830DE: to=<stan@man.net>, relay=none, delay=0.18, delays=0.03/0/0.15/0, dsn=5.4.6, status=bounced (mail for man.net loops back to myself)
postfix/smtp[25622]: 6B8664D830E3: to=<jame@live.ong>, relay=your-dns-needs-immediate-attention.ong[127.0.53.53]:25, delay=0.05, delays=0.03/0/0.03/0, dsn=5.4.6, status=bounced (mail for live.ong loops back to myself)
postfix/smtp[26295]: 47B584D830F6: to=<cesarelkappo@adulto.com>, relay=none, delay=0.07, delays=0.03/0/0.04/0, dsn=5.4.6, status=bounced (mail for adulto.com loops back to myself)
postfix/smtp[26295]: B1FE94D836AF: to=<anita@hot.de>, relay=none, delay=0.04, delays=0.03/0/0.01/0, dsn=5.4.6, status=bounced (mail for hot.de loops back to myself)
postfix/smtp[26295]: EF7B44D836B0: to=<vacapinta@gamiel.com>, relay=none, delay=0.41, delays=0.03/0/0.37/0, dsn=5.4.6, status=bounced (mail for gamiel.com loops back to myself)
postfix/smtp[27487]: 604824D83851: to=<0x0fgc@mhx.com>, relay=mail.mhx.com[0.0.0.0]:25, delay=0.17, delays=0.03/0/0.13/0, dsn=5.4.6, status=bounced (mail for mhx.com loops back to myself)
postfix/smtp[30751]: 0CF244D8316C: to=<paco-ds-@hotmeil.es>, relay=none, delay=0.44, delays=0.03/0/0.41/0, dsn=5.4.6, status=bounced (mail for hotmeil.es loops back to myself)
postfix/smtp[30521]: 6FA364D83CB6: to=<hgf@ghn.com>, relay=none, delay=0.15, delays=0.02/0/0.14/0, dsn=5.4.6, status=bounced (mail for ghn.com loops back to myself)
postfix/smtp[30521]: 524074D83D44: to=<bho3397@beer.com>, relay=none, delay=0.4, delays=0.02/0/0.37/0, dsn=5.4.6, status=bounced (mail for beer.com loops back to myself)
postfix/smtp[31226]: AC9A84D83D52: to=<ggs6@webv.de>, relay=none, delay=0.04, delays=0.03/0/0.01/0, dsn=5.4.6, status=bounced (mail for webv.de loops back to myself)
postfix/smtp[31226]: 4B3B24D83D5B: to=<luis_bahamonde@hotmeil.es>, relay=none, delay=0.26, delays=0.03/0/0.22/0, dsn=5.4.6, status=bounced (mail for hotmeil.es loops back to myself)
postfix/smtp[38806]: 1D1954D8177A: to=<www.suneth.a90@gaile.com>, relay=none, delay=0.11, delays=0.02/0/0.08/0, dsn=5.4.6, status=bounced (mail for gaile.com loops back to myself)
postfix/smtp[39895]: 339A74D81843: to=<harley811@version.net>, relay=none, delay=0.28, delays=0.02/0/0.26/0, dsn=5.4.6, status=bounced (mail for version.net loops back to myself)
```

Now if we try to resolve the MX record (to find out the responsible mail server) for the given recipients we notice something odd:

```shell
$ dig mx hot.de +short
10 null.hot.de.
$ dig null.hot.de. +short
127.0.0.1
```

```shell
$ dig mx iphone.com +short
10 mail.iphone.com.
$ dig mail.iphone.com +short
127.0.0.6
```

```shell
$ dig mx hotmaol.de +short
10 mail.hotmaol.de.
$ mail.hotmaol.de. +short
127.0.0.1
```

```shell
$ dig mx beer.com +short
10 mailnull.aftermarket.com.
$ mailnull.aftermarket.com +short
127.0.0.1
```

Almost any resolved MX of the offending log entries returns a loopback address like "`localhost`", "`127.0.0.1`" or even "`0.0.0.0`". Ergo, spammer selectively uses recipient addresses whose MX return such a bogus IP to create non delivery messages which then gets send to the forged (but existing) sender mail account.

Now a bit more in detail: The postfix resolves the MX → receives "`localhost`" → tries to relay the mail via the given "`localhost`" server → gets rejected → generated mailer daemon / NDM, which gets sent to the existing user "`antonettabaune@godetia.com`" → reports the sending server (we) for backscatter.

Now since we figured out what's going on, we also have to fix the issue somehow. Maintaining a transport rule list that contains domains with bogus MX addresses manually? Nah, not a future proof option to stick with. We have to fix the actual root cause of the problem not only it's symptoms.

# The solution

After quite a lot of trial and error in the dev environment, googling and spending some hours in the [#postfix](http://webchat.freenode.net/?channels=%23postfix&uio=d4) IRC channel, I came up with the following solution:

I added the following line in the `smtpd_recipient_restrictions` in Postfix's `main.cf`:

```
smtpd_recipient_restrictions =
    check_recipient_mx_access cidr:/etc/postfix/recipient_mx_access.cidr
    [...]
```

And created the file `/etc/postfix/recipient_mx_access.cidr` that only contains CIDR network notations of bogus MX'es with loopback:

```
0.0.0.0/8       REJECT Domain MX in broadcast network
10.0.0.0/8      REJECT Domain MX in RFC 1918 private network
127.0.0.0/8     REJECT Domain MX in loopback network
169.254.0.0/16  REJECT Domain MX in link local network
172.16.0.0/12   REJECT Domain MX in RFC 1918 private network
192.0.2.0/24    REJECT Domain MX in TEST-NET-1 network
192.168.0.0/16  REJECT Domain MX in RFC 1918 private network
198.51.100.0/24 REJECT Domain MX in TEST-NET-2 network
203.0.113.0/24  REJECT Domain MX in TEST-NET-3 network
224.0.0.0/4     REJECT Domain MX in class D multicast network
240.0.0.0/5     REJECT Domain MX in class E reserved network
248.0.0.0/5     REJECT Domain MX in reserved network

::1/128         REJECT Domain MX is Loopback address
::/128          REJECT Domain MX is Unspecified address
::/96           REJECT Domain MX in IPv4-Compatible IPv6
ff00::/8        REJECT Domain MX in Multicast network
fe80::/10       REJECT Domain MX in Link-local unicast network
fec0::/10       REJECT Domain MX in Site-local unicast network
```

Last but not least, I needed to restart Postfix:

```shell
$ service postfix restart
```

That's it! To make sure it's fixed, I tried to reproduce the issue via Telnet:

```shell
$ telnet localhost 25
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
220 web.company.de ESMTP Postfix (Ubuntu)
ehlo me
250-web.company.de
250-PIPELINING
250-SIZE 10240000
250-VRFY
250-ETRN
250-STARTTLS
250-ENHANCEDSTATUSCODES
250-8BITMIME
250 DSN
mail from: j@frd.mn
250 2.1.0 Ok
rcpt to: jfhfdq@hot.de
554 5.7.1 <jfhfdq@hot.de>: Recipient address rejected: Domain MX in loopback network
```

As you can see, I got rejected directly in the SMTP transaction, so this problem won't be an issue anymore.

TL;DR: To fix backscatter due to loopback MX records, add a `check_recipient_mx_access` table in your Postfix's `smtpd_recipient_restrictions`.
