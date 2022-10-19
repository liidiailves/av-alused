## I. osa PING ja HTTP

Kõigepealt proovimiseks mõned käsklused cloudshellis:
- `ip addr show eth0`- töötas
- `ip route show` - töötas
- `ping -c3 8.8.8.8` - ei töötanud; cloudshellis vastus: "command not found" ja arvuti päris cmd-s töötas käsklus `ping -n 3 8.8.8.8`
- `host -t aaaa google.com` - töötas (näitab IPv6 aadressi/aadresse)
- `host -t mx udacity.com` - töötas
- `tcpdump -n -c5 -i eth0 port 22` - vastus: "You don't have permission to capture on that device"
- `traceroute udacity.com` - töötas
- `mtr www.udacity.com` - töötas
- `printf 'HEAD / HTTP/1.1\r\nHost: www.udacity.com\r\n\r\n' | nc www.udacity.com 80` – töötas

Ping ja HTTP võrdlus:
- ping saadab kajataotluse süsteemi op. süsteemile (eeldab saaja süsteemis interneti olemasolu) (võib olla nii linux, windows, mac); ping on lihtsam kui http 
- HTTP võtab ühendust serveriga

Käskluse `printf 'HEAD / HTTP/1.1\r\nHost: en.wikipedia.org\r\n\r\n' | nc en.wikipedia.org 80` analüüs:
- `printf` on nagu `echo`, aga veidi targem
- `nc` = netcat - kirjutab ja loeb informatsiooni üle võrgu protokollide
- `|` - toru (pipe) siin: võtab printf väljundi ja viib selle nc sisendiks

See, et `printf` ja `nc`– ga saame HTTP vastuseid, on suur asi, kuna printf ei tea midagi võrgust (sisendi käsklus lihtsalt juhtub olema HTTP taotlus) 
ja netcat ei tea midagi HTTP taotlustest (nc saadab teksti üle võrgu).

Interneti 4 olulist kihti:
kõige peal rakendused (http, ssh), siis transportijad (tcp, udp), 
siis internet (mis koosneb ainult IP-st ja seetõttu nimetatakse ka pudelikaelaks) ning kõige all töötab riistvara (Wifi, Ethernet, DSL)

NC-ga portide kuulamine:
- näide: ühes aknas `nc -l 3456`, teises `nc localhost 3456`
- kui kirjutad ühes aknas, ilmub ka teises ja vastupidi, AGA ei kasuta HTTP-d! (toimetab kiht madalamal: tcp server)
- ühenduse lõpetamiseks EOF `^D` (kirjas nc manualis) – NB! Mul toimis ainult `ctrl+Z`!
- kõrgeim number, millega port kuulata saab, on: 65535 (2. osas seletus)
- madalaim number, millega port kuulata saab, on: 1024 (pordid alla 1024 on küll olemas, aga reserveeritud superkasutajatele)
- NB! Kui luua kaks `nc -l 3456` nimelist porti, siis kolmas (`nc localhost 3456`) ühendub vaid viimase aknaga! Veateadet minul ei andnud (Linuxis peaks andma)!
- `sudo lsof -i` töötas ainult siis, kui enne andsin käsu `sudo apt-get install lsof`!
- `printf 'HTTP/1.1 302 Moved\r\nLocation: https://www.eff.org/' | nc -l 2345` + `nc -l 2345` teises aknas ei andnud mul tulemusi… 
(st panin Windows aknas oma IP aadressi:2345, aga lehekülge www.eff.org ei lmunud, kuigi näites ilmus)

NC on nagu lihtsustatud mudel programmist, mis kasutab võrku. 
Veebilehed ja serverid on tegelikult palju keerulisemad (sest haldavad mitmeid taotlusi ja üheaegselt).

## 2. osa Nimed ja aadressid

Toon välja selles peatükis kõlama jäänud märksõnad.

- `host` - lihtne käsureautiliit nimepäringute tegemiseks. Tavaliselt kasutatakse seda nimede teisendamisel IP aadressideks ja vastupidi. 
- `ping google.com`: cloudshellis ei töötanud ("command not found"), arvuti cmd-s töötas ja IP-aadressiks näitas 172.217.16.206.
- DNS tõlgib veebiaadressi (www.example.com) IP-aadressiks. 
- DNSi teekond: domeeninimi – domeeni registreerimine/äratundmine – IP aadress
- DNS on väga oluline veebilehtedele – kui see ei tööta, siis leht ei toimi
- DNSi kasutatakse ka turvalisuse loomisel HTTP-dele (k.a. SSL ja küpsised)
- `host -t a google.com` – töötas
- `host` käsk vastab rohkem inimkeeli, `dig` käsk rohkem arvutikeeles
- `a` näitab IPv4 aadressi
- `aaaa` näitab IPv6 aadressi (4 x a nagu IPv6 on 4 x pikem kui IPv4)
- `CNAME` – domeeninime alias 
- `NS` - domeeninimi nimeserveritele selle domeeni alamdomeenide pärimiseks
- DNS serveritel on vahemälu, millel on oma kindel eluaeg (TTL ehk "time to live")
- On kahte tüüpi üladomeene: ccTLD - country code Top Level Domain ja gTLD - generic Top Level Domain
- DNS on nagu puu, mille oksad on erinevad serverid, mis vastutavad igaüks erinevate tippdomeenide eest
- www.example.com on subdomeen example.com-ile ja test.www.example.com on subdomeen www.example.com-ile
- et DNS domeeni luua,  tuleb minna lehele: domains.google.com ja domeen osta
- kui domeen ostetud, vajutan configure DNS, seejärel custom resourch records
- IPv4 koosneb neljast 8-bitisest osast eraldatud punktidega (8 bitti on kümnendsüsteemis numbrid 0-255)
- Miks portide suurim võimalik number on 65535? Sellepärast, et pordi bittide arv on piiratud: 16biti kasutatakse TCP pordi jaoks.

<em>Tulemas ka 3., 4. ja 5. osa.</em>
