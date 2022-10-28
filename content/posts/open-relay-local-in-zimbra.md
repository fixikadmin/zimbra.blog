Title: Open Relay local in Zimbra
Slug: open-relay-local-in-zimbra
Date: 2018-10-18 09:42:19
Tags: zimbra, security
Category: Zimbra
Lang: en
Translate: true
Summary: Implicit in Zimbra este deschis Open Relay local , adica mesajele pot fi transmise cu modificarea identitatii in cadrul aceluiasi domeniu.

###INTRO

De sub userul  zimbra se face urmatoarea comanda

```
zmprov mcf zimbraMtaSmtpdSenderLoginMaps proxy:ldap:/opt/zimbra/conf/ldap-slm.cf +zimbraMtaSmtpdSenderRestrictions reject_authenticated_sender_login_mismatch
```
Editam fisierul `/opt/zimbra/conf/zmconfigd/smtpd_sender_restrictions.cf`

Adaugam
```
permit_mynetworks, reject_sender_login_mismatch
```
Aplicam modificarile
```
zmmtactl restart && zmconfigdctl restart
```
Entru a adauga exceptii trebuie sa cream un fisier

```
zmprov mcf zimbraMtaSmtpdSenderLoginMaps 'lmdb:/opt/zimbra/conf/slm-exceptions-db, proxy:ldap:/opt/zimbra/conf/ldap-slm.cf' +zimbraMtaSmtpdSenderRestrictions reject_authenticated_sender_login_mismatch
```
Cream fisierul de exceptii `/opt/zimbra/conf/slm-exceptions-db` [allow to printer@zimbra.md send from alen.delon@zimbra.md]
alen.delon@zimbra.md alen.delon@zimbra.md printer@zimbra.md
Aplicam 
```
postmap /opt/zimbra/conf/slm-exceptions-db
zmmtactl restart && zmconfigdctl restart
```

