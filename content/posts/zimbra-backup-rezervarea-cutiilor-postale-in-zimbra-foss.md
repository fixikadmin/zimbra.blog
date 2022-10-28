Title: Zimbra Backup - rezervarea cutiilor postale in Zimbra FOSS
Slug: zimbra-backup-rezervarea-cutiilor-postale-in-zimbra-foss
Date: 2018-06-22 09:19:47
Tags:
Category: Zimbra
Lang: en
Translate: true
Summary:

### Cerinte fata de sistemul de back-up Zimbra FOSS:

1.	Solutie Open Source

2.	Crearea copiilor de rezerva diferentiale.

3.	Restabilirea mesajelor la cerere. (cu revenire la punctele de backup, Anual, Lunar, Saptamanal, Zilnic)

4.	Restabilirea dintr-un cont in altul.

5.	Restabilirea contului  in cazul ca a fost eliminat.

6.	Spatiul minimal pe storage.

### Limitari  ale sistemului de backup:

1.	Nu va fi posibila restabilirea mesajelor care nu sunt acoperite de punctele de backup

2.	In scopul economisirii spatiului fisierele de back-up premergatoare for fi eliminate ( )

### Solutii gasite:

1.	ZMBKPOSE (Zimbra Backup Open Source Edition)

2.	DAR

3.	Cloud Snapshot 

Solutia de backup care corespunde cerintelor este ZMBKPOSE (Branch: rewrite_proposal_v2.0).

Site-ul proiectului: <https://github.com/bggo/Zmbkpose>

Link-ul de download: <http://github.com/bggo/Zmbkpose/archive/rewrite_proposal_v2.0.zip>

### Estimari (rata de compresie, spatiu necesar):

1.	Spatiu estimativ pe Storage care va fi necesar:

Dupa testarea solutiei pentru cateva cutii postale , a fost determinat ca 2.43 GB  dupa  efectuarea copiei de rezerva a fost obtinut 1.3GB , 
respectiv obtinem o rata de 53% . Solutia ZMBKPOSE utilizeaza compresia GZIP+TAR, in documentatia oficiala rata de compresie este in limita 21-26%. 
Rezultatul din exemplu arata ca in copia de rezerva sunt prezente alte arhive care nu vor fi compresate.

2.	Timpul de compresie

Acest parametru depinde de mai multi factori: specificatia hardware, incarcarea sistemului (LA), 
2.43	GB  au fost arhivate in 360s

3.	Mai jos este prezentat graficul de utilizare al CPU la arhivarea unui volum mai mare , cca 18GB
 


Estimativ volumul total al mailbox-urilor de pe serverul meu este de cca. 195GB, care  va fi compresat in 103GB timp de 6 ore si 20 min.

Real volumul de 195 GB  a fost compresat in 158G si a durat 7 ore (de la ora 9 pina la 16, cu intreruperi)


### Continuitatea serviciului de rezervare - CRON

Crontab-ul pentru crearea copiilor de rezerva pentru toate conturile e-mail

```
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root
HOME=/tmp

##FULL BACKUPS every day for 400 accounts(-c 400), or for 6 hours(-C 6h)
#  whichever comes first, only if last full backup has more than 20 days 
#  old(-F 20d). 
# Pause of 5 seconds between accounts backups( -e 5s)
0 0 * * * zimbra LOG_TYPE=full_bk /usr/local/bin/zmbkpose.wrapper -F 20d -C 6h -c 400 -e 5s

##INCREMENTAL BACKUPS every day for 3 hours(-C 3h), only if last incremental backup 
# has more than 30hs old(-I 30h). 
# Perform a full backup is not has previous incremental (-t)
# Pause of 5 seconds betwee accounts backups (-e 5s)
0 6  * * * zimbra LOG_TYPE=inc_bk /usr/local/bin/zmbkpose.wrapper -I 30h -C 3h -t -e 5s
0 9  * * * zimbra LOG_TYPE=inc_bk /usr/local/bin/zmbkpose.wrapper -I 30h -C 3h -t -e 5s
0 18 * * * zimbra LOG_TYPE=inc_bk /usr/local/bin/zmbkpose.wrapper -I 30h -C 3h -t -e 5s
0 21 * * * zimbra LOG_TYPE=inc_bk /usr/local/bin/zmbkpose.wrapper -I 30h -C 3h -t -e 5s

#Remove old backups
#Delete cycles off backups older than 15 days(-d 15d). A cycle is a full backups
#  and its subsequent incremental backups. 
# zmbkpose will not eliminate backups leaving incomplete cycles
0 2 * * *  zimbra LOG_TYPE=del_bk /usr/local/bin/zmbkpose.wrapper -d 15d

#Remove old logs files
0 22 * * * zimbra find /var/log/zmbkpose -type f -mtime +30 -exec rm -f '{}' \;

```

Explicatie:

1.	BACKUP FULL in fiecare zi pentru 400 de conturi (-c 400) sau timp de 6 ore (-6h) care apare mai intai, numai daca ultima copie de rezerva completa are mai mult de 20 de zile (Vechime 20 zile (-F 20d). Pauza de 5 secunde intre copii de rezerva ale conturilor (-e 5s) )
```
0 0 * * * zimbra LOG_TYPE=full_bk /usr/local/bin/zmbkpose.wrapper -F 20d -C 6h -c 400 -e 5s
```
2.	BACKUP INCREMENTAL in fiecare zi timp in fiecare  3 ore (-C 3h), numai in cazul in care ultimul backup incremental are mai mult de 30 de ore vechime (-I 30h).Efectuiaza o copie de rezerva completa in caz ca nu are precedent incremental (-t)
```
# Pauza de 5 secunde intre fie copie de backup (-e 5s)
0 6  * * * zimbra LOG_TYPE=inc_bk /usr/local/bin/zmbkpose.wrapper -I 30h -C 3h -t -e 5s
0 9  * * * zimbra LOG_TYPE=inc_bk /usr/local/bin/zmbkpose.wrapper -I 30h -C 3h -t -e 5s
0 18 * * * zimbra LOG_TYPE=inc_bk /usr/local/bin/zmbkpose.wrapper -I 30h -C 3h -t -e 5s
0 21 * * * zimbra LOG_TYPE=inc_bk /usr/local/bin/zmbkpose.wrapper -I 30h -C 3h -t -e 5s
```

3.	ELIMINARE BACKUP Elimina backup-urile mai vechi de 15 zile (-d 15d). Un ciclu este un backup complet si backup-urile ulterioare incrementale. Zmbkpose nu va elimina backup-urile care lasa cicluri incomplete
```
0 2 * * *  zimbra LOG_TYPE=del_bk /usr/local/bin/zmbkpose.wrapper -d 15d
```
####Termeni: 

1.	Backup Full 
2.	Backup Full Conditionat
3.	Backup Incremental 
4.	Backup Incremental Conditionat
5.	Ciclu de Backup 

Ciclul de backup  reprezinta  un backup FULL(total) si subsecventele de backup INC(incremental). Ciclul complet trebuie sa fie intre data de "prezent" 
argumentul "-d"  de exemplu:
Ziua | Backup

1 FULL
2 INC
3 INC
4 INC
5 FULL
6 INC
7 FULL
8 FULL

In exemplul de mai sus avem 2 cicluri (1 ciclu  ziua 1-5, 2 ciclu  ziua  5-7  )

Atunci cand in ziua a 9-a executam zmbkpose cu cheile:
```
-d 2d => backup-urile pentru zilele 1,2,3,4,5,6 vor fi eliminate deoarece ele sunt create mai devreme de 7 zile (9 - 2 = 7)
-d 4d => backup-urile pentru zilele 1,2,3,4 vor fi eliminate deoarece ele sunt create mai devreme de 5 zile  (9 - 4 = 5)
-d 5d => nu va fi eliminate nici un fisier de backup deoarece  aceasta va duce intreruperea ciclului de  la 1 la 4
```
###Comenzi utile:

1.	Listarea backup-urilor
```
su - zimbra -c '/usr/local/bin/zmbkpose -l -a user@example.md'
```
Rezultat:
```
user@example.md "09/06/2017 17:09:19" FULL 1G
user@example.md "12/06/2017 10:21:07" INC 1M
user@example.md "13/06/2017 09:09:33" INC 650K
user@example.md "03/08/2017 16:18:00" INC 25M
user@example.md "29/08/2017 15:22:15" FULL 1G
user@example.md "29/08/2017 16:50:54" INC 40K
```

2.	Eliminarea backup-ului (vor fi eliminate backup-urile mai vechi de o luna )
```
su - zimbra -c '/usr/local/bin/zmbkpose -d 1m -a user@example.md'
```
Rezultat:
```
2017-08-30 10:49:50 DEBUG: user@example.md - START deleting old backups cycles prior to 30/08/2017 10:48:50 
2017-08-30 10:49:50 DEBUG: user@example.md - Deleting backup from 09/06/2017 17:09:19 FULL 
2017-08-30 10:49:50 DEBUG: user@example.md - Deleting backup from 12/06/2017 10:21:07 INC 
2017-08-30 10:49:50 DEBUG: user@example.md - Deleting backup from 13/06/2017 09:09:33 INC 
2017-08-30 10:49:50 DEBUG: user@example.md - Deleting backup from 03/08/2017 16:18:00 INC 
2017-08-30 10:49:50 DEBUG: user@example.md - END deleting old backups
```

3.	Restabilirea contului 

a.	Restabilirea ultimelor backup-uri  (Va fi restabilit ultimul ciclu FULL + INC(1)+INC(2)+...INC(n+1))

```
su - zimbra -c '/usr/local/bin/zmbkpose -r -a user@example.md'
```
Rezultat:
```
2017-08-30 10:55:01 DEBUG: Testing ldapserver ldap://172.28.92.3:389, using user uid=zimbra,cn=admins,cn=zimbra 
2017-08-30 10:55:01 DEBUG: user@example.md - Using ldap data from backup /opt/zimbra/backup/example.md/user/20170829165054:INC.tar 
2017-08-30 10:55:01 DEBUG: user@example.md - Ldap restore started 
2017-08-30 10:55:01 DEBUG: user@example.md - Mailbox restore started 
2017-08-30 10:55:01 DEBUG: user@example.md - Restauring from 29/08/2017 15:22:15 FULL 
2017-08-30 10:55:01 DEBUG: user@example.md - Restauring from 29/08/2017 16:50:54 INC 
2017-08-30	0:55:01 DEBUG: user@example.md - Restore end
```

b.	Restabilirea conditionata (Vor fi restabilite mesajele din toate backu-urile)

```
su - zimbra -c '/usr/local/bin/zmbkpose -R - -a user@example.md'
```
Rezultat:
```
2017-08-30 11:08:32 DEBUG: Testing ldapserver ldap://172.28.92.3:389, using user uid=zimbra,cn=admins,cn=zimbra 
2017-08-30 11:08:32 DEBUG: user@example.md - Using ldap data from backup /opt/zimbra/backup/example.md/user/20170829165054:INC.tar 
2017-08-30 11:08:32 DEBUG: user@example.md - Ldap restore started 
2017-08-30 11:08:32 DEBUG: user@example.md - Mailbox restore started 
2017-08-30 11:08:32 DEBUG: user@example.md - Restauring from 09/06/2017 17:09:19 FULL 
2017-08-30 11:08:32 DEBUG: user@example.md - Restauring from 12/06/2017 10:21:07 INC 
2017-08-30 11:08:32 DEBUG: user@example.md - Restauring from 13/06/2017 09:09:33 INC 
2017-08-30 11:08:32 DEBUG: user@example.md - Restauring from 03/08/2017 16:18:00 INC 
2017-08-30 11:08:32 DEBUG: user@example.md - Restauring from 29/08/2017 15:22:15 FULL 
2017-08-30 11:08:32 DEBUG: user@example.md - Restauring from 29/08/2017 16:50:54 INC 
2017-08-30	1:08:32 DEBUG: user@example.md - Restore end
```
c.	Restabilirea conditionata  (Vor fi restabilite mesajele la data de 03/08/2017 16:18:00)
```
su - zimbra -c '/usr/local/bin/zmbkpose -R 20170803161800 -a user@example.md'
2017-08-30 11:39:50 DEBUG: Testing ldapserver ldap://172.28.92.3:389, using user uid=zimbra,cn=admins,cn=zimbra 
2017-08-30 11:39:50 DEBUG: user@example.md - Using ldap data from backup /opt/zimbra/backup/example.md/user/20170803161800:INC.tar 
2017-08-30 11:39:50 DEBUG: user@example.md - Ldap restore started 
2017-08-30 11:39:50 DEBUG: user@example.md - Mailbox restore started 
2017-08-30 11:39:50 DEBUG: user@example.md - Restauring from 09/06/2017 17:09:19 FULL 
2017-08-30 11:39:50 DEBUG: user@example.md - Restauring from 12/06/2017 10:21:07 INC 
2017-08-30 11:39:50 DEBUG: user@example.md - Restauring from 13/06/2017 09:09:33 INC 
2017-08-30 11:39:50 DEBUG: user@example.md - Restauring from 03/08/2017 16:18:00 INC 
2017-08-30 11:39:50 DEBUG: user@example.md - Restore end
```


###Instalare:

1.	Se download-eaza scriptul de pe: h<ttp://github.com/bggo/Zmbkpose/archive/rewrite_proposal_v2.0.zip>

```
#cd /opt/kit/
#wget http://github.com/bggo/Zmbkpose/archive/rewrite_proposal_v2.0.zip
```
2.	Se de arhiveaza arhiva
```
#unzip rewrite_proposal_v2.0.zip
```
3.	Se trece in directorul de arhivat 
```
#cd Zmbkpose-rewrite_proposal_v2.0/
```
4.	Se instaleaza 
```
#./install.sh
```

Rezultatul executiei :
```
Checking zimbra installation            ...[OK]
  -zmbkpose will use "zimbra" user for execution.
Checking installer integrity            ...[OK]
Checking system for dependencies...     ...
  -Dependencies will be executed by "zimbra" user
  awk                                  ...[OK]
  curl                                 ...[OK]
  date                                 ...[OK]
  du                                   ...[OK]
  egrep                                ...[OK]
  find                                 ...[OK]
  grep                                 ...[OK]
  ldapadd                              ...[OK]
  ldapdelete                           ...[OK]
  ldapsearch                           ...[OK]
  ln                                   ...[OK]
  printf                               ...[OK]
  rm                                   ...[OK]
  sed                                  ...[OK]
  sort                                 ...[OK]
  tar                                  ...[OK]
  uniq                                 ...[OK]
  readlink                             ...[OK]
Installing                              ...[OK]

################################################################################
 I configured /etc/zmbkpose/zmbkpose.conf whith :
    WORKDIR="/opt/zimbra/backup"
 Change it if you decide to place the backup files elsewhere.

################################################################################
 If you want, I can try configure /etc/zmbkpose/zmbkpose.conf
  with the following settings automatically detected:
   * LDAPMASTERSERVER=ldap://192.168.0.1:389
   * LDAPZIMBRADN=uid=zimbra,cn=admins,cn=zimbra
   * LDAPZIMBRAPASS=WyZ3nfKd
Do you like this script make these settings? [y/n]: [YES]

################################################################################
 You will need to configure the follow values manually  
  in /etc/zmbkpose/zmbkpose.conf before to use zmbkpose:
    * ADMINUSER
    * ADMINPASS
The following users were found as administrators, and can be configured as ADMINUSER:
  * admin@example.md
################################################################################

Note: --------------------------------------------------------------------------
 Remember execute zmbkpose using user "zimbra", dependent commands 
  were checked using this user. 
--------------------------------------------------------------------------------
Install completed. Do you want to display the README file? [y/n]: [NO]
```

### Configurare:

1.	In  /etc/zmbkpose/zmbkpose.conf  se afla configurarile:

```
WORKDIR="/opt/zimbra/backup"  -  directorul unde se va face backup  (este creat automat la instalare)
ADMINUSER=zmbkpose@example.md.md  - cont de administrator
ADMINPASS=password  - parola contului de administrator
LDAPMASTERSERVER=ldap://192.168.0.1:389 - URL-ul serviciului Open LDAP (este setat automat la instalare)
LDAPZIMBRADN="uid=zimbra,cn=admins,cn=zimbra"  - schema LDAP dupa care sunt extrase datele  (este setat automat la instalare)
LDAPZIMBRAPASS="Wy**nfKd"  - parola de access la Open Ldap  (este setat automat la instalare)
PARALLEL_SUPPORT=0	-  suport pentru backup paralel
MAX_PARALLEL_PROCESS=3 -  numarul de procese paralele
```
In caz ca nu vrem sa indicam username-ul si parola de administrator:
```
# su zimbra
# zmprov ca zmbkpose@example.md "password"
# zmprov ma zmbkpose@example.md zimbraIsAdminAccount TRUE
```

2.	Scriptul executabil se afla  in /usr/local/bin/zmbkpose

```
Aditional este propus un script care ofera posibilitatea de a automatiza unele procese. Scriptul de afla in directorul  unde a fost dezarhivata arhiva cu instalatia example/usr/local/bin/zmbkpose.wrapper
Acest script este copiat in /usr/local/bin/zmbkpose
#cd  /opt/kit/Zmbkpose-rewrite_proposal_v2.0/
#cp example/usr/local/bin/zmbkpose.wrapper /usr/local/bin/
# chown zimbra:root /usr/local/bin/zmbkpose.wrapper 
#chmod 700 /usr/local/bin/zmbkpose.wrapper
```

3.	Se instaleaza crontab-ul:

```
# crontab -e   
```
Se adauga urmatoarele: 

```
##FULL BACKUPS every day for 400 accounts(-c 400), or for 6 hours(-C 6h)
#  whichever comes first, only if last full backup has more than 20 days
#  old(-F 20d).
# Pause of 5 seconds between accounts backups( -e 5s)

0 0 * * * su - zimbra -c 'LOG_TYPE=full_bk /usr/local/bin/zmbkpose.wrapper -F 20d -C 6h -c 400 -e 5s'

##INCREMENTAL BACKUPS every day for 3 hours(-C 3h), only if last incremental backup 
# has more than 30hs old(-I 30h). 
# Perform a full backup is not has previous incremental (-t)
# Pause of 5 seconds betwee accounts backups (-e 5s)

0 6  * * * su - zimbra -c 'LOG_TYPE=inc_bk /usr/local/bin/zmbkpose.wrapper -I 30h -C 3h -t -e 5s'
0 9  * * * su - zimbra -c 'LOG_TYPE=inc_bk /usr/local/bin/zmbkpose.wrapper -I 30h -C 3h -t -e 5s'
0 18 * * * su - zimbra -c 'LOG_TYPE=inc_bk /usr/local/bin/zmbkpose.wrapper -I 30h -C 3h -t -e 5s'
0 21 * * * su - zimbra -c 'LOG_TYPE=inc_bk /usr/local/bin/zmbkpose.wrapper -I 30h -C 3h -t -e 5s'

#Remove old backups
#Delete cycles off backups older than 15 days(-d 15d). A cycle is a full backups
#  and its subsequent incremental backups. 
# zmbkpose will not eliminate backups leaving incomplete cycles
0 2 * * *  su - zimbra -c 'LOG_TYPE=del_bk /usr/local/bin/zmbkpose.wrapper -d 15d'

#Remove old logs files
0 22 * * * su - zimbra -c 'find /var/log/zmbkpose -type f -mtime +30 -exec rm -f '{}' \;'
```		




