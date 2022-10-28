Title: Export grants and views of a delegated admin
Slug: exportarea-drepturilor-administrator-delegat-zimbra
Date: 2018-10-18 09:42:19
Tags: zimbra, security
Category: Zimbra
Lang: en
Translate: true
Summary: Export grants & views of a delegated admin and create a new delegated admin with the same grants & views

###Problem

Export grants & views of a delegated admin and create a new delegated admin with the same grants & views
###Solution

Note: This is only applicable for legacy admin module and does not apply for Zimbra NG module.

Here we have an existing delegated admin "myadmin@DOMAIN.COM" and we will create a new delegated admin '"newadmin@DOMAIN.COM" with the same views and grants.

1). Check enabled views of existing delegated admin:-
```
zmprov -l ga myadmin@DOMAIN.COM | egrep -i 'zimbraAdminConsoleUIComponents|zimbraIsDelegatedAdminAccount:' 
```
Output:-
```
zimbraAdminConsoleUIComponents: accountListView
zimbraAdminConsoleUIComponents: downloadsView
zimbraAdminConsoleUIComponents: DLListView
zimbraAdminConsoleUIComponents: aliasListView
zimbraAdminConsoleUIComponents: resourceListView
zimbraAdminConsoleUIComponents: saveSearch
zimbraIsDelegatedAdminAccount: TRUE 
```

2). Check or export assigned rights of the delegated admin :-
```
zmprov gg -g usr myadmin@DOMAIN.COM 
```
Output:-
```
    target type  target id                            target name        grantee type grantee id                           grantee name       right
    ------------ ------------------------------------ ------------------ ------------ ------------------------------------ ------------------ --------------------
    global                                            globalacltarget    usr          87609353-a8fb-4ed5-b750-6b538cd52f35 myadmin@DOMAIN.COM adminLoginCalendarResourceAs
    global                                            globalacltarget    usr          87609353-a8fb-4ed5-b750-6b538cd52f35 myadmin@DOMAIN.COM domainAdminZimletRights
    domain       1ccb92be-56cc-4962-b964-b07af84dc118 DOMAIN.COM         usr          87609353-a8fb-4ed5-b750-6b538cd52f35 myadmin@DOMAIN.COM domainAdminConsoleRights
```

3). Now we have to fine tune above output for a new admin.
```
global usr myadmin@DOMAIN.COM adminLoginCalendarResourceAs
global usr myadmin@DOMAIN.COM domainAdminZimletRights
domain DOMAIN.COM usr myadmin@DOMAIN.COM domainAdminConsoleRights 
```

4). Create a file "/tmp/grants.txt" with the exported grants and replace old admin name with new delegated admin.
Prepare exported grant file for new delegated admin (newadmin@DOMAIN.COM). The file must have grants in the following format:-
```
grr global usr newadmin@DOMAIN.COM adminLoginCalendarResourceAs
grr global usr newadmin@DOMAIN.COM domainAdminZimletRights 
grr domain DOMAIN.COM usr newadmin@DOMAIN.COM domainAdminConsoleRights 
```

5). Now we will create a new delegated admin with the same views as existing admin has: -
```
zmprov ca newadmin@DOMAIN.COM <PASSWORD> zimbraIsDelegatedAdminAccount TRUE zimbraAdminConsoleUIComponents accountListView zimbraAdminConsoleUIComponents downloadsView zimbraAdminConsoleUIComponents DLListView zimbraAdminConsoleUIComponents aliasListView zimbraAdminConsoleUIComponents resourceListView zimbraAdminConsoleUIComponents saveSearch
```

6). Here we will assign grants from the prepared file in Step4:-
```
zmprov < /tmp/grants.txt 
```

7). Now check grants of newly created delegated admin, the output of below command must be similar to the output of Step2:-
```
zmprov gg -g usr newadmin@DOMAIN.COM 
```



###Extra Notes:

Some additional tips for those admins who loves to play with sed and awk:-

Here we are exporting and redirecting grants to a file, and preparing grants for new delegated admin.
NOTE: These steps are only for Domain and Global level grants. If there are other level grants assigned to delegated admin then use "awk" carefully to extract correct column.
```
zmprov gg -g usr myadmin@DOMAIN.COM | grep ^global | awk '{print $1,$3,$5,$6}'  >> /tmp/grants.txt
zmprov gg -g usr myadmin@DOMAIN.COM | grep ^domain | awk '{print $1,$3,$4,$6,$7}'  >> /tmp/grants.txt 
```
Check the content of the file "/tmp/grants.txt" and the output will look like the following:-
```
cat /tmp/grants.txt 
global usr newadmin@DOMAIN.COM adminLoginCalendarResourceAs
global usr newadmin@DOMAIN.COM domainAdminZimletRights
domain DOMAIN.COM usr newadmin@DOMAIN.COM domainAdminConsoleRights 
```
Add "grr" at the beginning of each line:-
```
sed -i 's/^/grr /' /tmp/grants.txt 
```
Now file will show content in following format:-
```
cat /tmp/grants.txt 
grr global usr newadmin@DOMAIN.COM adminLoginCalendarResourceAs
grr global usr newadmin@DOMAIN.COM domainAdminZimletRights
grr domain DOMAIN.COM usr newadmin@DOMAIN.COM domainAdminConsoleRights 
```
Replace old delegated admin email-id with new delegated admin:-
```
sed -i 's/myadmin@DOMAIN.COM/newadmin@DOMAIN.COM/' /tmp/grants.txt 
```
Now we will assign grants to new delegated admin with prepared file /tmp/grants.txt
```
zmprov < /tmp/grants.txt 
```

