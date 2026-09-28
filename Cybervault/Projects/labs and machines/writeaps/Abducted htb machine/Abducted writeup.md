![[Pasted image 20260924162319.png]]![[Pasted image 20260924162507.png]]![[Pasted image 20260924162540.png]]![[Pasted image 20260924162613.png]]
![[Pasted image 20260924171112.png]]
there is a user called scott de rid 0x3e8
![[Pasted image 20260924171357.png]]
![[Pasted image 20260924171426.png]]
the full name of scoot is scott Mercer
Home Drive : \\ABDUCTED\scott
## `bad_password_count`

Très intéressant :

```
bad_password_count : 0x00000000
```

Cela signifie que le compteur de mauvaises authentifications enregistré pour ce compte est actuellement **0**.

Et :

```
logon_count : 0x00000000
```

indique également qu'aucune connexion réussie n'est comptabilisée dans ce champ.

Ces compteurs sont simplement des **informations d'état** exposées par RPC.
![[Pasted image 20260924172149.png]]
![[Pasted image 20260924172208.png]]
![[Pasted image 20260924172224.png]]
![[Pasted image 20260924172306.png enum4linux-ng -A 10.129.244.177          





![[Pasted image 20260924174631.png]]


![[Pasted image 20260924174755.png]]
![[Pasted image 20260924174825.png]]














ENUM4LINUX - next generation (v1.3.10)




 ==========================
|    Target Information    |
 ==========================
[*] Target ........... 10.129.244.177
[*] Username ......... ''
[*] Random Username .. 'euxynpxn'
[*] Password ......... ''
[*] Timeout .......... 10 second(s)

 =======================================
|    Listener Scan on 10.129.244.177    |
 =======================================
[*] Checking LDAP
[-] Could not connect to LDAP on 389/tcp: connection refused
[*] Checking LDAPS
[-] Could not connect to LDAPS on 636/tcp: connection refused
[*] Checking SMB
[+] SMB is accessible on 445/tcp
[*] Checking SMB over NetBIOS
[+] SMB over NetBIOS is accessible on 139/tcp

 =============================================================
|    NetBIOS Names and Workgroup/Domain for 10.129.244.177    |
 =============================================================
[+] Got domain/workgroup name: WORKGROUP
[+] Full NetBIOS names information:
- ABDUCTED        <00> -         B <ACTIVE>  Workstation Service                                                                                                                              
- ABDUCTED        <03> -         B <ACTIVE>  Messenger Service                                                                                                                                
- ABDUCTED        <20> -         B <ACTIVE>  File Server Service                                                                                                                              
- WORKGROUP       <00> - <GROUP> B <ACTIVE>  Domain/Workgroup Name                                                                                                                            
- WORKGROUP       <1e> - <GROUP> B <ACTIVE>  Browser Service Elections                                                                                                                        
- MAC Address = 00-00-00-00-00-00                                                                                                                                                             

 ===========================================
|    SMB Dialect Check on 10.129.244.177    |
 ===========================================
[*] Trying on 445/tcp
[+] Supported dialects and settings:
Supported dialects:                                                                                                                                                                           
  SMB 1.0: false                                                                                                                                                                              
  SMB 2.0.2: true                                                                                                                                                                             
  SMB 2.1: true                                                                                                                                                                               
  SMB 3.0: true                                                                                                                                                                               
  SMB 3.1.1: true                                                                                                                                                                             
Preferred dialect: SMB 3.0                                                                                                                                                                    
SMB1 only: false                                                                                                                                                                              
SMB signing required: false                                                                                                                                                                   

 =============================================================
|    Domain Information via SMB session for 10.129.244.177    |
 =============================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found domain information via SMB
NetBIOS computer name: ABDUCTED                                                                                                                                                               
NetBIOS domain name: ''                                                                                                                                                                       
DNS domain: ''                                                                                                                                                                                
FQDN: abducted                                                                                                                                                                                
Derived membership: workgroup member                                                                                                                                                          
Derived domain: unknown                                                                                                                                                                       

 ===========================================
|    RPC Session Check on 10.129.244.177    |
 ===========================================
[*] Check for anonymous access (null session)
[+] Server allows authentication via username '' and password ''
[*] Check for guest access
[+] Server allows authentication via username 'euxynpxn' and password ''
[H] Rerunning enumeration with user 'euxynpxn' might give more results

 =====================================================
|    Domain Information via RPC for 10.129.244.177    |
 =====================================================
[+] Domain: WORKGROUP
[+] Domain SID: NULL SID
[+] Membership: workgroup member

 =================================================
|    OS Information via RPC for 10.129.244.177    |
 =================================================
[*] Enumerating via unauthenticated SMB session on 445/tcp
[+] Found OS information via SMB
[*] Enumerating via 'srvinfo'
[+] Found OS information via 'srvinfo'
[+] After merging OS information we have the following result:
OS: Windows 7, Windows Server 2008 R2                                                                                                                                                         
OS version: '6.1'                                                                                                                                                                             
OS release: ''                                                                                                                                                                                
OS build: '0'                                                                                                                                                                                 
Native OS: not supported                                                                                                                                                                      
Native LAN manager: not supported                                                                                                                                                             
Platform id: '500'                                                                                                                                                                            
Server type: '0x809a03'                                                                                                                                                                       
Server type string: Wk Sv PrQ Unx NT SNT Hartley Group Document Services                                                                                                                      

 =======================================
|    Users via RPC on 10.129.244.177    |
 =======================================
[*] Enumerating users via 'querydispinfo'
[+] Found 1 user(s) via 'querydispinfo'
[*] Enumerating users via 'enumdomusers'
[+] Found 1 user(s) via 'enumdomusers'
[+] After merging user results we have 1 user(s) total:
'1000':                                                                                                                                                                                       
  username: scott                                                                                                                                                                             
  name: Scott Mercer                                                                                                                                                                          
  acb: '0x00000010'                                                                                                                                                                           
  description: ''                                                                                                                                                                             

 ========================================
|    Groups via RPC on 10.129.244.177    |
 ========================================
[*] Enumerating local groups
[+] Found 0 group(s) via 'enumalsgroups domain'
[*] Enumerating builtin groups
[+] Found 0 group(s) via 'enumalsgroups builtin'
[*] Enumerating domain groups
[+] Found 0 group(s) via 'enumdomgroups'

 ========================================
|    Shares via RPC on 10.129.244.177    |
 ========================================
[*] Enumerating shares
[+] Found 4 share(s):
HP-Reception:                                                                                                                                                                                 
  comment: Reception printer                                                                                                                                                                  
  type: Printer                                                                                                                                                                               
IPC$:                                                                                                                                                                                         
  comment: IPC Service (Hartley Group Document Services)                                                                                                                                      
  type: IPC                                                                                                                                                                                   
projects:                                                                                                                                                                                     
  comment: Hartley Group Project Files                                                                                                                                                        
  type: Disk                                                                                                                                                                                  
transfer:                                                                                                                                                                                     
  comment: Staff file transfer                                                                                                                                                                
  type: Disk                                                                                                                                                                                  
[*] Testing share HP-Reception
[+] Mapping: OK, Listing: NOT SUPPORTED
[*] Testing share IPC$
[+] Mapping: OK, Listing: NOT SUPPORTED
[*] Testing share projects
[+] Mapping: DENIED, Listing: N/A
[*] Testing share transfer
[+] Mapping: DENIED, Listing: N/A

 ===========================================
|    Policies via RPC for 10.129.244.177    |
 ===========================================
[*] Trying port 445/tcp
[+] Found policy:
Domain password information:                                                                                                                                                                  
  Password history length: None                                                                                                                                                               
  Minimum password length: 5                                                                                                                                                                  
  Minimum password age: none                                                                                                                                                                  
  Maximum password age: 49710 days (136 years) 6 hours 21 minutes                                                                                                                             
  Password properties:                                                                                                                                                                        
  - DOMAIN_PASSWORD_COMPLEX: false                                                                                                                                                            
  - DOMAIN_PASSWORD_NO_ANON_CHANGE: false                                                                                                                                                     
  - DOMAIN_PASSWORD_NO_CLEAR_CHANGE: false                                                                                                                                                    
  - DOMAIN_PASSWORD_LOCKOUT_ADMINS: false                                                                                                                                                     
  - DOMAIN_PASSWORD_PASSWORD_STORE_CLEARTEXT: false                                                                                                                                           
  - DOMAIN_PASSWORD_REFUSE_PASSWORD_CHANGE: false                                                                                                                                             
Domain lockout information:                                                                                                                                                                   
  Lockout observation window: 30 minutes                                                                                                                                                      
  Lockout duration: 30 minutes                                                                                                                                                                
  Lockout threshold: None                                                                                                                                                                     
Domain logoff information:                                                                                                                                                                    
  Force logoff time: 49710 days (136 years) 6 hours 21 minutes                                                                                                                                

 ===========================================
|    Printers via RPC for 10.129.244.177    |
 ===========================================
[+] Found 1 printer(s):
\\10.129.244.177\:                                                                                                                                                                            
  description: \\10.129.244.177\,,Reception printer                                                                                                                                           
  comment: Reception printer                                                                                                                                                                  
  flags: '0x800000'                                                                                                                                                                           

Completed after 39.25 se


