+++

+++
Modificarea in ciclu a conturilor e-mail din statutul "Active"  in "Closed". Este utilizat ciclul for si comanda CLI Zimbra "zmprov"

> Am gasit pe internet urmatorea comanda pentru a modifica recursiv toate conturile din statut "Activ" in statut "Closed". Deasemenea poate fi utilizat pentru alte operatiuni cu conturile Zimbra

Folosim urmatorul script in bash

    for i in `zmprov -l gaa bd.xenos`; do  zmprov ma $i zimbraAccountStatus closed; done