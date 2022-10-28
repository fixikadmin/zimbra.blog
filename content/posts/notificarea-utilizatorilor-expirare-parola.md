Title: Notificarea utilizatorilor despre expirarea parolei
Date: 2018-10-20 11:20
Modified: 2018-10-20 11:31
Category: Zimbra
Tags: zimbra,bash
Slug: notificarea-utilizatorilo-despre-expirarea-parolei
Authors: JekA
Summary: ![Zimbra](http://drive.google.com/uc?export=view&id=1DwIusgQeMAAOb33Z_SsPp6lwF-gSezva) Script bash care notifica utilizatorii despre expirarea parolei

###INTRO
In instalatia default Zimbra notificarea catre utilizatori care informeaza utilizatorul despre expirarea parolei sale nu este transmis. Astfel utilizatorul este nevoit sa apeleze
la administratorul serviciului de posta electronica cu rugamintea de ai reseta parola (In cazul ca nu o cunoaste). Utilizatorii MS Outlook  primesc eroare la conectare.

In acest caz folosim un script Bash care in vom pune in cron, pentru a rula in fiecare zi.

###SCRIPT
```
#!/bin/bash

FROM="postmaster@zimbra.md"
DOMAIN="zimbra.md"
SENDMAIL=$(ionice -c3 find /opt/zimbra/ -type f -iname sendmail)
USERS=`/opt/zimbra/bin/zmprov -l gaa | grep -v "galsync" | grep -v "zimbra.md"`;
DATE=$(date +%s)

for USER in $USERS
do
OBJECT="(&(objectClass=zimbraAccount)(mail=$USER))"
ZIMBRA_LDAP_PASSWORD=`su - zimbra -c "zmlocalconfig -s zimbra_ldap_password | cut -d ' ' -f3"`
LDAP_MASTER_URL=`su - zimbra -c "zmlocalconfig -s ldap_master_url | cut -d ' ' -f3"`
LDAPSEARCH=$(ionice -c3 find /opt/zimbra/ -type f -iname ldapsearch)
PASS_SET_DATE=`$LDAPSEARCH -H $LDAP_MASTER_URL -w $ZIMBRA_LDAP_PASSWORD -D uid=zimbra,cn=admins,cn=zimbra \
-x $OBJECT | grep zimbraPasswordModifiedTime: | cut -d " " -f 2 | cut -c 1-8`

EXPIRES=$(date -d  "$PASS_SET_DATE 60 days" +%s)
DEADLINE=$(( (($DATE - $EXPIRES)) / -86400 ))

SUBJECT="$USER - Parola contului DVS. e-mail expira in $DEADLINE zile"
BODY="
Stimate client, detinator al adresei e-mail $USER,

Va anuntam ca parola dvs. de e-mail va expira in $DEADLINE zile. Va rugam sa modificati parola e-mail \
prin Web Mail:

 - Adresa : https://$DOMAIN

1. Conectati-va la Web Mail prin adresa de mai sus
2. Selectati fila Preferinte
3. Faceti clic pe butonul Modifica parola
4. Completati parola veche, noua parola si confirmati noua parola
5. Faceti clic pe Modificare parola pentru a o schimba

Parola contului e-mail trebuie sa contina de cel putin 8 caractere, cu o combinatie de caractere alfanumerice \
(majuscule, litere mici, numere) si simboluri (! @ # $, Etc.).

Daca aveti intrebari cu privire la modul de a schimba parola de e-mail, va rugam sa contactati echipa de asistenta \
la nr. de tel. x(xx)xxxxxx

Multumim, echipa Zimbra.md
"

if [[ "$DEADLINE" -eq "5" ]]
then
        echo "Subject: $SUBJECT" "$BODY" | $SENDMAIL -f "$FROM" "$USER"
        echo "Reminder email sent to: $USER - $DEADLINE days left"

elif [[ "$DEADLINE" -eq "3" ]]
then
        echo "Subject: $SUBJECT" "$BODY" | $SENDMAIL -f "$FROM" "$USER"
        echo "Reminder email sent to: $USER - $DEADLINE days left"

elif [[ "$DEADLINE" -eq "1" ]]
then
        echo "Subject: $SUBJECT" "$BODY" | $SENDMAIL -f "$FROM" "$USER"
        echo "Last chance for: $USER - $DEADLINE days left"

else
        echo "Account: $USER reports; $DEADLINE days on Password policy"
fi

done
```
###REFERINTE:

[1] <https://imanudin.net/2017/02/04/script-notify-expired-password-on-zimbra>

[2] <https://github.com/wuxmedia/Zimbra_passpoll/blob/master/passpoll.sh>
