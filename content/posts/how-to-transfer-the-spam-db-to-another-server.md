Title: How to transfer the spam db to another server
Date: 2018-05-24 13:52:22
Modified: 2018-05-24 13:52:22
Category: Zimbra
Tags: zimbra, bash
Slug: how-to-transfer-the-spam-db-to-another-server
Lang: en
Translate: true
Summary:How to transfer the anti-spam database to another Zimbra server

### How to transfer the anti-spam database to another Zimbra server

In this post we will see how the Spamasssain base can be transferred from the old server to a new one.

1. Run this command on the old server
```
su - zimbra
/opt/zimbra/libexec/sa-learn -p /opt/zimbra/conf/salocal.cf.in --dbpath /opt/zimbra/data/amavisd/.spamassassin/ --siteconfigpath /opt/zimbra/conf/spamassassin --backup > /tmp/zimbra_q.backup
```
2. Copy the file `/tmp/zimbra_q.backup` to the new server

3. Run this command on the new server
```
su - zimbra
/opt/zimbra/libexec/sa-learn -p /opt/zimbra/conf/salocal.cf.in --dbpath /opt/zimbra/data/amavisd/.spamassassin/ --siteconfigpath /opt/zimbra/conf/spamassassin --restore /tmp/zimbra_q.backup
```
