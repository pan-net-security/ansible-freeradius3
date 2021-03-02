

To debug FreeRADIUS, stop the service, then start it in DEBUG mode:
```
sudo freeradius -X
```

To validate FreeRADIUS configuration files, run:  
```
radiusd -XC
```

Test the FreeRADIUS server by querying it directly with requests.
```
radtest {username} {password+otp} {hostname} 10 {radius_<type>_secret}
example:
    radtest a000000 heslo123987654 localhost:1812 0 radius1234

Sending Access-Request of id 221 to 127.0.0.1 port 1812
	User-Name = "a000000"
	User-Password = "heslo123987654"
	NAS-IP-Address = X.X.X.X
	NAS-Port = 0
	Message-Authenticator = 0x00000000000000000000000000000000
rad_recv: Access-Accept packet from host 127.0.0.1 port 1812, id=221, length=48
	Reply-Message = "privacyIDEA access granted"

```

Test the FreeRADIUS using the Radclient radius client program:
```
echo "User-Name = test" | radclient localhost:1812 auth {{radius_<type>_secret}}

echo "User-Name=a000000@example.com,User-Password=heslo123987654" | radclient localhost:1812 auth radius1234
```

To test ldap bind with credentials, use:  
```
ldapwhoami -vvv -h <hostname> -p <port> -D <binddn> -x -W
```

To test user search with ldap authentification:  
```
ldapsearch -x -D "cn=admin,cn=Administrators,cn=config" -h server.example.com -w "password..." -b "ou=users,ou=accounts,dc=example,dc=com" uid=someuser
```
