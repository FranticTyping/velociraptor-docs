---
title: Windows.Forensics.UserAccessLogs
hidden: true
tags: [Client Artifact]
---

UAL is a feature that can help server administrators quantify the
number of unique client requests of roles and services on a local
server.

The UAL only exists on Windows Server edition later than 2012.


```yaml
name: Windows.Forensics.UserAccessLogs
description: |
  UAL is a feature that can help server administrators quantify the
  number of unique client requests of roles and services on a local
  server.

  The UAL only exists on Windows Server edition later than 2012.

reference:
  - https://advisory.kpmg.us/blog/2021/digital-forensics-incident-response.html
  - https://docs.microsoft.com/en-us/windows-server/administration/user-access-logging/manage-user-access-logging
  - https://www.crowdstrike.com/blog/user-access-logging-ual-overview/

parameters:
   - name: SUMGlob
     description: The SUM ESE files to search for.
     default: |
       C:\Windows\System32\LogFiles\SUM\*.mdb

sources:
  - query: |
        -- These come from the SystemIdentity.mdb ROLE_IDS table but
        -- they are not going to change so we hard code them here.
        LET Roles <= SELECT * FROM parse_csv(accessor="data", filename='''
        RoleGuid,RoleName
        "{c50fcc83-bc8d-4df5-8a3d-89d7f80f074b}",Active Directory Certificate Services
        "{b4cdd739-089c-417e-878d-855f90081be7}",Active Directory Rights Management Service
        "{48eed6b2-9cdc-4358-b5a5-8dea3b2f3f6a}",DHCP Server
        "{7cc4b071-292c-4732-97a1-cf9a7301195d}",FAX Server
        "{10a9226f-50ee-49d8-a393-9a501d47ce04}",File Server
        "{bbd85b29-9dcc-4fd9-865d-3846dcba75c7}",Network Policy and Access Services
        "{7fb09bd3-7fe6-435e-8348-7d8aefb6cea3}",Print and Document Services
        "{d6256cf7-98fb-4eb4-aa18-303f1da1f770}",Web Server
        "{4116a14d-3840-4f42-a67f-f2f9ff46eb4c}",Windows Deployment Services
        "{d8dc1c8e-ea13-49ce-9a68-c9dca8db8b33}",Windows Server Update Services
        "{c23f1c6a-30a8-41b6-bbf7-f266563dfcd6}",FTP Server
        "{910cbaf9-b612-4782-a21f-f7c75105434a}",BranchCache
        "{952285d9-edb7-4b6b-9d85-0c09e3da0bbd}",Remote Access
        "{ad495fc3-0eaa-413d-ba7d-8b13fa7ec598}",Active Directory Domain Services
        ''')

        -- Resolve GUID to name
        LET GetRoleName(guid) = SELECT RoleName FROM Roles WHERE RoleGuid = guid

        -- Format the address - it can be IPv4, IPv6 or something else.
        LET FormatAddress(x) = if(condition=len(list=x) = 4,

         -- IPv4 address should be formatted in dot notation
         then=format(format="%d.%d.%d.%d", args=[x[0], x[1], x[2], x[3]]),
         else=if(condition=len(list=x)=16,
           -- IPv6 addresses are usually shortened
           then=regex_replace(source=format(format="%x", args=x),
                              re="(00)+", replace=":"),

           -- We dont know what kind of address it is.
           else=format(format="%x", args=x)))

        LET ParseESE = SELECT FullPath AS _FullPath, RoleGuid,
               GetRoleName(guid=RoleGuid)[0].RoleName AS RoleName,
               TenantId, TotalAccesses, InsertDate, LastAccess,
               FormatAddress(x=Address) AS Address,
               AuthenticatedUserName,
               -- Only show the days that have something in them.
               to_dict(item={
                 SELECT
                   format(format="Day%v", args=_value) AS _key,
                   get(field=format(format="Day%v", args=_value)) AS _value
                 FROM range(start=0, end=366, step=1)
                 WHERE _value
               }) AS Days
        FROM parse_ese(file=FullPath, table="CLIENTS")

        SELECT * FROM foreach(row={
          SELECT FullPath
          FROM glob(globs=SUMGlob)
        }, query=ParseESE)

```
