# Layer 2/Layer 3 Switch Configuration





```
  
!
banner login ^CCC

+-------------------------------------------------+
|     ___ _           _ __ __     _       _       |
|    |  _| |___ _ _ _| |  \  \_ _| |  ___| |_     |
|    | <_| / . | | / . |     | | | |_<_> | . \    |
|    `___|_\___`___\___|_|_|_`_. |___<___|___/    |
|                            <___'                |
|                                                 |
+-------------------------------------------------+

          Device: Mock Core Layer 3 Switch
^C
!



interface Vlan1
 ip address 10.1.10.102 255.255.255.0
 no ip route-cache
 no ip mroute-cache
end


hostname mock-core01

ip domain-name dc.local

crypto key gen rsa


line vty 0 4

line console 0
logging synchronous
login local

enable secret 1234QWer
aaa new-model
username admin password 1234QWer 
username admin priv 16



```



