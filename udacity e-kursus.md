# Arvutivõrkude alused

## I. osa PING ja HTTP

Enne kasutamist käivitan käsud:
`sudo apt-get update && sudo apt-get install netcat-openbsd tcpdump traceroute mtr iputils-ping lsof -y`

Kõigepealt proovimiseks mõned käsklused cloudshellis:
- `ip addr show eth0`- töötas
- `ip route show` - töötas
- `ping -c3 8.8.8.8` - töötas
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
- ühenduse lõpetamiseks EOF `^D` (kirjas nc manualis) – NB! Mul toimis `ctrl+Z` (pausile) ja `ctrl+C` (katkestab)!
- kõrgeim number, millega port kuulata saab, on: 65535 (2. osas seletus)
- madalaim number, millega port kuulata saab, on: 1024 (pordid alla 1024 on küll olemas, aga reserveeritud superkasutajatele)
- NB! Kui luua kaks `nc -l 3456` nimelist porti, siis kolmas (`nc localhost 3456`) ühendub vaid viimase aknaga! Veateadet minul ei andnud (Linuxis peaks andma)!
  Või kui üks ühenduspaar on loodud, siis kolmandas aknas `nc -l 3456` ei saanud tegelikult ühendust localhostiga. Veateadet ka ei andnud.
- `sudo lsof -i` töötas ainult siis, kui enne andsin käsu `sudo apt-get install lsof`! (peale uuendatud alguskäsurida pole vajalik)
- `printf 'HTTP/1.1 302 Moved\r\nLocation: https://www.eff.org/' | nc -l 2345` + `nc -l 2345` teises aknas ei andnud mul tulemusi… 
(st panin Windows aknas oma IP aadressi:2345, aga lehekülge www.eff.org ei lmunud, kuigi näites ilmus)

NC on nagu lihtsustatud mudel programmist, mis kasutab võrku. 
Veebilehed ja serverid on tegelikult palju keerulisemad (sest haldavad mitmeid taotlusi ja üheaegselt).

## 2. osa Nimed ja aadressid

Toon välja selles peatükis kõlama jäänud märksõnad.

- `host` - lihtne käsureautiliit nimepäringute tegemiseks. Tavaliselt kasutatakse seda nimede teisendamisel IP aadressideks ja vastupidi. 
- `ping google.com`: töötas
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
- Miks portide suurim võimalik number on 65535? Sellepärast, et pordi bittide arv on piiratud: 16 bitti kasutatakse TCP pordi jaoks.

## 3. osa Adresseerimine ja võrgud

Selle peatüki märksõnad on IPv4, selle jagunemine võrguosaks ja aadressiosaks, avalikud aadressid, privaatsed kohtvõrgud, NAT ja IPv6.

IPv4 on 32bitine numbritejada, mida arvutid omavahel kasutavad kahendsüsteemis, meie (inimesed) kümnendsüsteemis (punktidega eraldatud neljaks 8bitiseks).  
Kõikidest IPv4 aadressitest veidi üle 1/8 pole kättesaadavad.  
IPv4 aadresse on maailmas kokku veidi üle 4 miljardi, aga seda on vähem, kui inimesi planeedil.  
Samas võrgublokis olevatel IPv4 aadressitel on samasugune IPv4 aadressi algus, ainult lõpp erinev.  
Mida pikem algus ehk prefix (eesliide), seda väiksem ruum jääb lõpuosasse, et eristada täpsemalt kasutajaid ja vastupidi - mida väiksem eesliide, seda rohkem ruumi jääb erinevatele aadressitele, seega see on suurem võrk.  
- nt võrk, mille eesliide on 24 bitti, siis 8 bitti jääb aadressitele:  198.51.100.0/24 võrgublokk  
- kui eesliide 22 bitti, siis 10 bitti jääb aadressitele: 198.51.100.0/22 võrgublokk  
NB! Aadressile peale vaadates ei saa öelda, millisesse võrgublokki kuulub ja see võib kaasa tuua ka segadust.  
10 bitti = 2^10 aadresse = 1024 tk, kasutada ei saa esimest ja viimast (reserveeritud) ja esimene, mida saab kasutada, on tavaliselt ruuter, seega aadressitele vaba 1021  
8 bitti = 2^8 aadresse = 256 tk, kasutada saab 253  

Võrgumaskist (<em>ingl</em> subnet mask):  
Seadme lokaalse võrguühenduse konfigureerimisel on vaja peale IP-aadressi teada ka võrgumaski väärtust, mis sõltub võrgu suurusest ja administraatori valikutest.     Võrgumask lisatakse IP-aadressile kaldkriipsu abil. Näiteks IP-aadress 193.40.17.233/24 tähendab, et aadressi võrguossa kuulub 24 bitti, kümnendsüsteemis on säärane võrgumask 255.255.255.0 (selguse annab teisendus kahendsüsteemi – 11111111 11111111 11111111 00000000 –, kus sisaldub 24 ühte).  
Võrgumaski näide /14 võrgubloki kohta:  
11111111111111(14tk - võrguosa)000000000000000000(18tk - aadressite osa)  
255.252.0.0   
aadresse: 2^18 (32-14=18) ehk 262144 tk  

Tegelikult aadressid ei kuulu hostidele, vaid arvuti liidestele(<em>ingl</em> interface)!  (Host on võrgus olev masin, millel on unikaalne IP-aadress ja millega saab suhelda. Host võib olla nii tavaline arvuti, kuid ka server, võrguprinter jne.)  
- nt sülearvutil on mitu liidest (Ethernet liides eth0, wifi liides wlan0, loopback liides (arvutil suhtlemiseks iseendaga) ja igal liidesel võib olla kas null v mitu IP-aadressi.  
`ip addr show` - näitab sinu arvuti liideseid, minu cloudshellis näitas: lo (127.0.0.1/8)  ja eth0 (172.17.0.4/16)  
`ifconfig | less` - MACi peal  
Loopback liidesel on pea alati IP-aadressiks 127.0.0.1 ja localhostil on sama IP-aadress.  
Kodusel ruuteril:  
- local network = LAN – seal asuvad mu arvutid ja seadmed  
- wide area network = WAN – ühendub ülejäänud internetiga  

Arvuti ehk host vajab internetiga suhtlemiseks MyDefault Gateway-d.  
`ip route show default`- cloudshellis tuli vastuseks: default via 172.17.0.1  
`netstat -nr` – näitas ka sisuliselt sama gateway numbrit ja natuke lisa  
`ping 172.17.0.1` - keskmiseks kiiruseks ütles 0.068ms  
ISP telecom Int lubab ainult ühe aadressi luua ühele kodule/kontorile vms.  
Aga ühe IP aadressi taga ühes kodus tänasel päeval peidavad end mitmed arvutid, telekad, telefonid…  

Siin tulebki mängu süsteem NAT (<em>ingl</em> network address translation).  
Koduruuter määrab kodustele masinatele privaatsed aadressid ja peab privaatsed aadressid teisendama avalikeks (public) aadressiteks, et suhelda maailmaga (ruuteri sees on kaart, mis teab, mis privaatne aadress vastab mis avalikule aadressile).  
"Privaatne" ei tähenda, et salajane v isiklik, vaid et neid saab kasutada ainult kohtvõrgus, kuna need pole unikaalsed - samad NAT numbrid võivad kasutusel olla kõikjal maailmas, aga toimivad ainult oma võrgus. Neid ei või kunagi kasutada avalikus internetis.  
NATi numbrid on:   
- 10.0.0.0/8  
- 172.16.0.0/12   
- 192.168.0.0/16
Oma arvuti avalik IP-aadress on lihtsasti leitav google v ping otsingumootoriga, nt mul 195.238.64.94.  
Aga mu arvuti võrguseadetes on kirjas IPv4 aadress: 192.168.1.146, seega see on privaatne aadress.  

Kokkuvõtlikult NAT pole lahendus, vaid ajutine lahendus, IPv6 on lahendus IP aadressite puudusele.  
IPv6 aadressid on palju pikemad : 128 bitti ehk 2^128 aadressivõimalusi.  
Aadress esitatakse kolmekümne kahe 16-ndnumbrina. 16-ndnumbrid on grupeeritud neljakaupa ehk tetraadidena ja eraldatud koolonitega.  
Sarnaselt IPv4 aadressi ülesehitusele ka IPv6 aadress jaguneb eesliiteks (võrgublokiks) ja hosti osaks, selle erinevusega, et aadressite osa on võrratult suurem ja seega NAT pole vajalik, igale seadmele on võimalik eraldada tema isiklik IPv6 aadress.  
IPv6 aadressite osakaal maailmas on aina suurenemas ja on IP-aadressite tulevik.  
<http://test-ipv6.com/> - lehekülg, kust kontrollida, kas su seadmel on IPv6 aadress. Minu telefonil ja arvutil polbud, aga näitab, et koduvõrgul on IPv6 võimekus olemas.

## 4. osa Võrguprotokollide kihid

Protokoll on reeglite kogum, mida arvutid kasutavad andmete võrgu kaudu saatmiseks ja vastuvõtmiseks.  
Protokoll määrab kasutatava veaavastuskoodi, andmetihendusmeetodi ning viisi, kuidas saatev seade annab teada, et sõnumi edastamine on lõpule jõudnud ning kuidas vastuvõttev seade annab teada, et sõnum on vastu võetud.  
Iga protokollikiht sõltub endast allpool olevatest kihtidest ja samuti on alumiste kihtide töö tihti ülemiste kihtide eest varjul. 
- http => tcp => ip => riistvara  

Katse 1: `ping -n3 8.8.8.8` (mul `ping -c3 8.8.8.8`) ühes cloudshelli aknas, `sudo tcpdump -n host 8.8.8.8` teises aknas:
ping aknas 3 vastust, tcpdumpi aknas 3 echopaari, üks echo request, teine echo reply.

Katse 2: `sudo tcpdump -n port 53` ühes aknas, seejärel teises aknas `ping yahoo.com`:
töötas; tcpdump näitab DNS requeste, mida arvuti saadab.

Katse 3: `sudo tcpdump -n port 80` ühes aknas, teises aknas `printf ‘HEAD/http/1.1\r\nHost: example.net\r\n\r\n’ | nc example.net 80`:
lihtsaim selle kirjeldamiseks kopeerida tulemus siia:
-tcpdump: verbose output suppressed, use -v[v]... for full protocol decode
-listening on eth0, link-type EN10MB (Ethernet), snapshot length 262144 bytes
-18:04:12.579436 IP 172.17.0.4.56906 *(minu IP aadress)* > 93.184.216.34.80 *(example.net IP-aadress)*: Flags [S] *(selle tähendusest juttu allpool)*, seq 2589877733, win 64240, options [mss 1460,sackOK,TS val 2028034390 ecr 0,nop,wscale 7], length 0*(paljudel puhkudel length 0! mis on TCP üks oluline fakt)*
-18:04:12.658814 IP 93.184.216.34.80 > 172.17.0.4.56906: Flags [S.], seq 3729182669, ack 2589877734, win 65535, options [mss 1460,sackOK,TS val 546338418 ecr 2028034390,nop,wscale 9], length 0
-18:04:12.658851 IP 172.17.0.4.56906 > 93.184.216.34.80: Flags [.], ack 1, win 502, options [nop,nop,TS val 2028034470 ecr 546338418], length 0
-18:04:12.658943 IP 172.17.0.4.56906 > 93.184.216.34.80: Flags [P.], seq 1:24, ack 1, win 502, options [nop,nop,TS val 2028034470 ecr 546338418], **length 23**: HTTP
-18:04:12.737318 IP 93.184.216.34.80 > 172.17.0.4.56906: Flags [.], ack 24, win 128, options [nop,nop,TS val 546338497 ecr 2028034470], length 0
-18:04:12.737320 IP 93.184.216.34.80 > 172.17.0.4.56906: Flags [P.], seq 1:517, ack 24, win 128, options [nop,nop,TS val 546338497 ecr 2028034470], **length 516**: HTTP: HTTP/1.0 501 Not Implemented *(see tähistab server vastest kliendile! kuna sel on reaalne suurus!)*
-18:04:12.737347 IP 172.17.0.4.56906 > 93.184.216.34.80: Flags [.], ack 517, win 501, options [nop,nop,TS val 2028034548 ecr 546338497], length 0
-18:04:12.737322 IP 93.184.216.34.80 > 172.17.0.4.56906: Flags [F.], seq 517, ack 24, win 128, options [nop,nop,TS val 546338497 ecr 2028034470], length 0
-18:04:12.737934 IP 172.17.0.4.56906 > 93.184.216.34.80: Flags [F.], seq 24, ack 518, win 501, options [nop,nop,TS val 2028034549 ecr 546338497], length 0
-18:04:12.816258 IP 93.184.216.34.80 > 172.17.0.4.56906: Flags [.], ack 25, win 128, options [nop,nop,TS val 546338576 ecr 2028034549], length 0
- kasutas näites käsklust `printf ‘HEAD/http/1.1\r\nHost: example.net\r\n\r\n’ | wc`, et leida konkureetselt suurust

Mis toimub, kui TCP ühendus avatakse?

TCP- ühendus on töökindel, sest toimub kolmepoolne kinnitus ehk three-way handshake:  
- klient saadab serverile ühenduse loomise soovi (järjekorranumbriga 1), server vastab (järjekorranumbriga 2 + ack järjekorranumbriga 1) ning saadab samuti ühenduse loomise soovi, mille klient kadudeta andmevahetuse korral vastusega kinnitab (järjekorranumbriga 1 + ack järjekorranumbriga 2). TCP sobib rakendustele, mis vajavad töökindlat andmeedastust, kus kiirus ei ole kriitiline.  
Teine internetiprotokolli peal käitatav protokoll on kasutajadatagrammi protokoll (UDP), mis saadab pakette kuid ei kontrolli nende kohalejõudmist sihtkohta.  
Seda kasutatakse näiteks multimeedia edastamisel ning mujal, kus tähtis on andmesidekiirus ja vähem tähtis selle kvaliteet (pakettide kadumisel neid uuesti ei saadeta).
6 põhilist TCP lippu:
- [S] – *synchronize* ehk uue ühenduse loomine
- [F] – *finish* ehk ühenduse lõpetamine
- [P] – *push* ehk see pakett on rakenduse andmete, näiteks HTTP-päringu, lõpp
- [R] – *reset* ehk TCP erroriteade
- [.] – *acknowledge* ehk kinnitus, et selle saatja on kätte saanud teise poole info
- [U] – *urgent* ehk see pakett sisaldab infot, mida on vaja järjekorraväliselt edastada, HTTP jpt ei kasuta seda.

Koormusjuhtimine (*congestion control*) tähendab, et TCP jälgib otspunktidevahelise võrgu koormust ning muudab ka selle järgi pakettide parameetreid ehk teisisõnu, 
saadab infot alguses aeglaselt ja etapikaupa ning kui näeb, et teine pool vastab ja on piisavalt kiire, siis lisab kiirust. 

KATSE 4: `sudo tcpdump port 12345`, teises `nc udacity.com 12345`:
Tegu väljamõeldud pordinumbriga. Esimeses aknas saatis järjest aeglasema intervalliga [S] lipuga päringuid, aga vastust ei saanud.
TCP-l on mitmeid sisse-ehitatud ajavõtjaid, mis kontrollivad, et kui soovitud serveriga ühendust mõistliku aja jooksul ei saa, siis saadavad erroriteate (timeout) tagasi vastuseks.

