Title: Add calendar to all Zimbra users
Slug: add-calendar-to-all-zimbra-users
Date: 2018-05-22 13:34:05
Category: Zimbra
Tags: zimbra, bash
Authors: JekA
Summary: In acest post voi descrie o sacina ce tine de calendarul Zimbra. Celendarul Zimbra permite multe operatii insa unele functii pot fi efectuate doar din consola. Deasemenea utilizarea unui script usureaza automatizarea procesului.  
###Sarcina pusa este:
1. Crarea unui calendar comun pentru toti utilizatorii cu urmatoarele proprietati:
	* Calendarul trebuie sa fie inclus in toate conturile participantilor
	* Va fi completat de fiecare participant in conformitate cu programul zilei propriu.
	* Participantii pot vedea statutul celulilalt paricipant (Ocupar/Disponibil) in calendarul sau.
	* Participantii vor fi adaugati automat la crearea unui nou utilizator
	* Un cont specializat va integra toate calendarele pentru a crea reguli VOIP de readresare a apelurilor
2. Crearea unui calendar comun cu zilelel de odihna si sarbatori.
	* Calendarul trebuie sa fie public.
	* Va fi administrat doar de o persoana

### 1. Crarea unui calendar comun pentru toti utilizatorii

Se creaza un Operator de calendar, de ex. `robocall@domen.md`
Se creaza un un grup de distibutie cu toti paricipantii, de ex. `call.center@domen.md`
In acest grup se adauga toti participantii care vor partaja calendarele proprii
 
Se executa comanda `zmprov`  pentru a aadauga calendele din grupul `call.center@domen.md`:
```
zmmailbox -z -m robocall@domen.md modifyFolderGrant /Calendar group  call.center@domen.md r
```
Se adauga calendarele partajate ale participantilor grupului
```
zmmailbox -z -m robocall@domen.md createMountpoint --view appointment "/OP1" operator1@domen.md /Calendar
zmmailbox -z -m robocall@domen.md createMountpoint --view appointment "/OP2" operator2@domen.md /Calendar
zmmailbox -z -m robocall@domen.md createMountpoint --view appointment "/OP3" operator3@domen.md /Calendar
```

Adaugarea calendarului in contul de integrare prin Export/Import

Salvam calendarul necesar
```
wget --user operator1@domen.md --password password --no-check-certificate https://hostname/home/operator1@domen.md/Calendar.ics
```
Importam in calendarul contului specilaizat
```
curl --insecure  -u robcall@domen.md:password --upload-file Calendar.ics https://hostname/service/home/robocall@domen.md/Calendar?fmt=ics
```
###2. Adaugarea mai multor calendare ale utilizatorilor intr-un calendar specializat.

Creem calendarul
```
zmprov sm operator2@domen.md createFolder --view appointment --color blue --flags "#" /RoboCal
```
Setam permisiuni de citire pentru calendareele create pentru utilizatorii; operatpr1, operator2, operator3
```
zmmailbox -z -m operator1@domen.md modifyFolderGrant /RoboCal group call.center@domen.md r
zmmailbox -z -m operator2@domen.md modifyFolderGrant /RoboCal group call.center@domen.md r
zmmailbox -z -m operator3@domen.md modifyFolderGrant /RoboCal group call.center@domen.md r
```
Se include fiecare calendar in contul specializat
```
zmmailbox -z -m robocall@domen.md createMountpoint --view appointment -color blue --flags "#" "/Operator1" operator1@domen.md /RoboCal
zmmailbox -z -m robocall@domen.md createMountpoint --view appointment -color blue --flags "#" "/Operator2" operator2@domen.md /RoboCal
zmmailbox -z -m robocall@domen.md createMountpoint --view appointment -color blue --flags "#" "/Operator3" operator3@domen.md /RoboCal

```
`--flags "#"`  semnifica ca calendarul este bifat

Pentru a automatiza procesul vom folosi un script bash:  <https://wiki.zimbra.com/wiki/Automating_Reciprocal_Calendar_Sharing>
```
#!/bin/bash

if [ "$#" == "1" ]
then
users=$(zmsoap --type admin --zadmin GetDistributionListRequest/dl="$1" @by="name" | grep dlm | sed -e 's/.*<dlm>\(.*\)<\/dlm>/\1/g')

echo $users

for u in $users
do
    echo zmmailbox -z -m $u modifyFolderGrant /RoboCal group call.center@domen.md r
    uname=$(zmmailbox -z -m $u gid -v | grep zimbraPrefFromDisplay | sed -e 's/.*: "\(.*\)",/\1/')
    for v in $users
    do
        if [ $u != $v ]
        then
            echo zmmailbox -z -m $v createMountpoint --view appointment \"/$uname\'s Calendar\" $u /RoboCal
        fi
    done
done
else
    echo "usage: $0 distlist@zimbradomain.com"
fi
```
Salvam scriptul ca *zimbraRecipCalShare.sh*.  Dam drepturi de executie
```
chmod +x zimbraRecipCalShare.sh
```
Rulam scriptul de sub utilizatorul zimbra (parametrul liniei este grupul in care sunt adaugati toti participantii)
```
zimbraRecipCalShare.sh call.center@domen.md > check_me_first.sh
```
Verifica daca continutul scriptului contine comenzi `zmmailbox` corecte si executam rezultatul
```
bash check_me_first.sh
```
In rezultat fiecare din participanti va avea calendarele celorlalti cu drepturi doar de citire.

