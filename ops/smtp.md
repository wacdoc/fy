# Bou jo eigen SMTP-postferstjoerserver

## preambule

SMTP kin direkt tsjinsten keapje fan wolkleveransiers, lykas:

* [Amazon SES SMTP](https://docs.aws.amazon.com/ses/latest/dg/send-email-smtp.html)
* [Ali wolk e-post push](https://www.alibabacloud.com/help/directmail/latest/three-mail-sending-methods)

Jo kinne ek jo eigen e-posttsjinner bouwe - ûnbeheind ferstjoeren, lege totale kosten.

Hjirûnder litte wy stap foar stap sjen hoe't jo ús eigen e-posttsjinner bouwe.

## Server seleksje

De sels-hosted SMTP-tsjinner fereasket in iepenbiere IP mei havens 25, 456, en 587 iepen.

Algemien brûkte iepenbiere wolken hawwe blokkearre dizze havens standert, en it kin wêze mooglik om te iepenjen se troch it útjaan fan in wurk oarder, mar it is hiel lestich na.

Ik advisearje te keapjen fan in host dy't dizze havens iepen hat en stipet it ynstellen fan omkearde domeinnammen.

Hjir advisearje ik [Contabo](https://contabo.com) .

Contabo is in hostingprovider basearre yn München, Dútslân, oprjochte yn 2003 mei heul konkurrearjende prizen.

As jo ​​kieze Euro as de faluta fan oankeap, de priis sil wêze goedkeaper (in server mei 8GB ûnthâld en 4 CPUs kostet likernôch 529 yuan per jier, en de earste ynstallaasje fergoeding is fergees foar ien jier).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/UoAQkwY.webp)

By it pleatsen fan in bestelling, opmerking `prefer AMD` , en de tsjinner mei AMD CPU sil hawwe bettere prestaasjes.

Yn it folgjende sil ik Contabo's VPS as foarbyld nimme om te demonstrearjen hoe't jo jo eigen e-posttsjinner bouwe kinne.

## Ubuntu systeem konfiguraasje

It bestjoeringssysteem hjir is Ubuntu 22.04

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/smpIu1F.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/m7Mwjwr.webp)

As de tsjinner op ssh werjûn `Welcome to TinyCore 13!` (lykas werjûn yn 'e ôfbylding hjirûnder), betsjut it dat it systeem noch net ynstalleare is. Verbreken asjebleaft ssh en wachtsje in pear minuten om wer oan te melden.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/-qKACz9.webp)

As `Welcome to Ubuntu 22.04.1 LTS` ferskynt, is de inisjalisaasje foltôge, en jo kinne trochgean mei de folgjende stappen.

### [Opsjoneel] Inisjalisearje de ûntwikkelingsomjouwing

Dizze stap is opsjoneel.

Foar gemak set ik de ynstallaasje en systeemkonfiguraasje fan ubuntu-software yn [github.com/wactax/ops.os/tree/main/ubuntu](https://github.com/wactax/ops.os/tree/main/ubuntu) .

Rinne it folgjende kommando út om te ynstallearjen mei ien klik.

```
bash <(curl -s https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

Sineeske brûkers, brûk dan it folgjende kommando ynstee, en de taal, tiidsône, ensfh. wurdt automatysk ynsteld.

```
CN=1 bash <(curl -s https://ghproxy.com/https://raw.githubusercontent.com/wactax/ops.os/main/ubuntu/boot.sh)
```

### Contabo stelt IPV6 yn

Aktivearje IPV6 sadat SMTP ek e-mails mei IPV6-adressen ferstjoere kin.

`/etc/sysctl.conf`

Feroarje of foegje de folgjende rigels ta

```
net.ipv6.conf.all.disable_ipv6 = 0
net.ipv6.conf.default.disable_ipv6 = 0
net.ipv6.conf.lo.disable_ipv6 = 0
```

Folgje op mei [it kontakttutorial: IPv6-ferbining tafoegje oan jo tsjinner](https://contabo.com/blog/adding-ipv6-connectivity-to-your-server/)

Bewurkje `/etc/netplan/01-netcfg.yaml` , foegje in pear rigels ta lykas werjûn yn 'e figuer hjirûnder (Contabo VPS standert konfiguraasjetriem hat al dizze rigels, gewoan uncomment se).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/5MEi41I.webp)

Dan `netplan apply` om de wizige konfiguraasje effekt te meitsjen.

Nei't de konfiguraasje suksesfol is, kinne jo `curl 6.ipw.cn` brûke om it ipv6-adres fan jo eksterne netwurk te besjen.

## Clone de konfiguraasje repository ops

```
git clone https://github.com/wactax/ops.soft.git
```

## Generearje in fergees SSL-sertifikaat foar jo domeinnamme

It ferstjoeren fan e-post fereasket in SSL-sertifikaat foar fersifering en ûndertekening.

Wy brûke [acme.sh](https://github.com/acmesh-official/acme.sh) om sertifikaten te generearjen.

acme.sh is in iepen boarne automatisearre ark foar sertifikaatûndertekening,

Fier it konfiguraasjepakhûs yn ops.soft, run `./ssl.sh` , en in `conf` -map sil makke wurde yn **'e boppeste map** .

Fyn jo DNS-oanbieder fan [acme.sh dnsapi](https://github.com/acmesh-official/acme.sh/wiki/dnsapi) , bewurkje `conf/conf.sh` .

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Qjq1C1i.webp)

Run dan `./ssl.sh 123.com` om `123.com` en `*.123.com` sertifikaten te generearjen foar jo domeinnamme.

De earste run sil automatysk [acme.sh](https://github.com/acmesh-official/acme.sh) ynstallearje en in plande taak tafoegje foar automatyske fernijing. Jo kinne sjen `crontab -l` , der is sa'n rigel as folget.

```
52 0 * * * "/mnt/www/.acme.sh"/acme.sh --cron --home "/mnt/www/.acme.sh" > /dev/null
```

It paad foar it oanmakke sertifikaat is wat as `/mnt/www/.acme.sh/123.com_ecc。`

Sertifikaatfernijing sil `conf/reload/123.com.sh` skript neame, dit skript bewurkje, jo kinne kommando's tafoegje lykas `nginx -s reload` om de sertifikaatcache fan relatearre applikaasjes te ferfarskjen.

## Bou SMTP-tsjinner mei chasquid

[chasquid](https://github.com/albertito/chasquid) is in iepen boarne SMTP-tsjinner skreaun yn Go-taal.

As ferfanging foar de âlde e-postserverprogramma's lykas Postfix en Sendmail, is chasquid ienfâldiger en makliker te brûken, en it is ek makliker foar sekundêre ûntwikkeling.

Run `./chasquid/init.sh 123.com` sil automatysk ynstalleare wurde mei ien klik (ferfang 123.com mei jo stjoerende domeinnamme).

## E-posthantekening DKIM ynstelle

DKIM wurdt brûkt om e-posthantekeningen te ferstjoeren om foar te kommen dat brieven as ûnpost behannele wurde.

Nei't it kommando mei súkses rint, wurde jo frege om it DKIM-record yn te stellen (lykas hjirûnder werjûn).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/LJWGsmI.webp)

Foegje gewoan in TXT-record ta oan jo DNS (lykas hjirûnder werjûn).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/0szKWqV.webp)

## Besjoch tsjinststatus en logs

 `systemctl status chasquid` Besjoch tsjinststatus.

De tastân fan normale operaasje is lykas werjûn yn 'e figuer hjirûnder

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/CAwyY4E.webp)

 `grep chasquid /var/log/syslog` of `journalctl -xeu chasquid` kin it flaterlog besjen.

## Reverse domeinnamme konfiguraasje

De omkearde domeinnamme is om it IP-adres op te lossen nei de oerienkommende domeinnamme.

It ynstellen fan in omkearde domeinnamme kin foarkomme dat e-mails wurde identifisearre as spam.

As de e-post wurdt ûntfongen, sil de ûntfangende tsjinner omkearde domeinnamme-analyse útfiere op it IP-adres fan 'e stjoerende tsjinner om te befêstigjen oft de stjoerende tsjinner in jildige omkearde domeinnamme hat.

As de stjoerende tsjinner gjin omkearde domeinnamme hat of as de omkearde domeinnamme net oerienkomt mei it IP-adres fan de ferstjoerende tsjinner, kin de ûntfangende tsjinner de e-post as spam werkenne of it ôfwize.

Besykje [https://my.contabo.com/rdns](https://my.contabo.com/rdns) en konfigurearje lykas hjirûnder werjûn

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/IIPdBk_.webp)

Nei it ynstellen fan de omkearde domeinnamme, tink derom om de foarútresolúsje fan 'e domeinnamme ipv4 en ipv6 te konfigurearjen nei de tsjinner.

## Bewurkje de hostnamme fan chasquid.conf

Feroarje `conf/chasquid/chasquid.conf` oan 'e wearde fan 'e omkearde domeinnamme.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/6Fw4SQi.webp)

Run dan `systemctl restart chasquid` om de tsjinst opnij te begjinnen.

## Reservekopy conf nei git repository

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/Fier9uv.webp)

Bygelyks, ik meitsje in reservekopy fan de conf map nei myn eigen github proses as folget

Meitsje earst in privee pakhús

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ZSCT1t1.webp)

Fier de conf-map yn en yntsjinje nei it pakhús

```
git init
git add .
git commit -m "init"
git branch -M main
git remote add origin git@github.com:wac-tax-key/conf.git
git push -u origin main
```

## Foegje stjoerder ta

rinne

```
chasquid-util user-add i@wac.tax
```

Kin taheakje stjoerder

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/khHjLof.webp)

### Kontrolearje dat it wachtwurd goed is ynsteld

```
chasquid-util authenticate i@wac.tax --password=xxxxxxx
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/e92JHXq.webp)

Nei it tafoegjen fan de brûker sil `chasquid/domains/wac.tax/users` wurde bywurke, tink om it yn te tsjinjen oan it pakhús.

## DNS tafoegje SPF-record

SPF (Sender Policy Framework) is in technology foar e-postferifikaasje brûkt om e-postfraude te foarkommen.

It ferifiearret de identiteit fan in e-poststjoerder troch te kontrolearjen dat it IP-adres fan de stjoerder oerienkomt mei de DNS-records fan 'e domeinnamme dy't it beweart te wêzen, foarkomt dat fraudeurs falske e-mails ferstjoere.

It tafoegjen fan SPF-records kin foarkomme dat e-mails safolle mooglik identifisearre wurde as spam.

As jo ​​domeinnammetsjinner gjin SPF-type stipet, foegje dan gewoan TXT-typerecord ta.

Bygelyks, de SPF fan `wac.tax` is as folget

`v=spf1 a mx include:_spf.wac.tax include:_spf.google.com ~all`

SPF foar `_spf.wac.tax`

`v=spf1 a:smtp.wac.tax ~all`

Tink derom dat ik hjir `include:_spf.google.com` haw, dit is om't ik letter `i@wac.tax` sil konfigurearje as it ferstjoeradres yn 'e postfak fan Google.

## DNS konfiguraasje DMARC

DMARC is de ôfkoarting fan (Domain-based Message Authentication, Reporting & Conformance).

It wurdt brûkt om SPF-bounces te fangen (miskien feroarsake troch konfiguraasjeflaters, of immen oars docht as jo om spam te stjoeren).

TXT-record taheakje `_dmarc` ,

De ynhâld is as folget

```
v=DMARC1; p=quarantine; fo=1; ruf=mailto:ruf@wac.tax; rua=mailto:rua@wac.tax
```

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/k44P7O3.webp)

De betsjutting fan elke parameter is as folget

### p (belied)

Jout oan hoe't e-posten wurde behannele dy't SPF (Sender Policy Framework) of DKIM (DomainKeys Identified Mail) ferifikaasje mislearje. De parameter p kin ynsteld wurde op ien fan trije wearden:

* gjin: Gjin aksje wurdt nommen, allinnich it ferifikaasjeresultaat wurdt weromfierd nei de stjoerder fia de e-postmeldingsmeganisme.
* Quarantaine: Set de e-post dy't de ferifikaasje net trochjûn hat yn 'e spam-map, mar sil de e-post net direkt ôfwize.
* reject: Direkt ôfwize e-mails dy't mislearre ferifikaasje.

### fo (failopsjes)

Spesifiseart de hoemannichte ynformaasje werom troch it rapportaazjemeganisme. It kin ynsteld wurde op ien fan 'e folgjende wearden:

* 0: Rapportearje falidaasjeresultaten foar alle berjochten
* 1: Rapportearje allinich berjochten dy't ferifikaasje mislearje
* d: Rapportearje allinich mislearrings foar ferifikaasje fan domeinnamme
* s: allinne rapportearje SPF ferifikaasje mislearrings
* l: Rapportearje allinne DKIM ferifikaasje mislearrings

### rua & ruf

* rua (Rapportearjende URI foar Aggregate rapporten): E-postadres foar it ûntfangen fan aggregearre rapporten
* ruf (Rapportearjende URI foar forensyske rapporten): e-postadres om detaillearre rapporten te ûntfangen

## Foegje MX-records ta om e-mails troch te stjoeren nei Google Mail

Om't ik gjin fergese bedriuwspostfak koe fine dy't universele adressen stipet (Catch-All, kin alle e-postberjochten ûntfange dy't nei dizze domeinnamme stjoerd binne, sûnder beheiningen op foarheaksels), brûkte ik Chasquid om alle e-postberjochten troch te stjoeren nei myn Gmail-postfak.

**As jo ​​​​jo eigen betelle saaklike postfak hawwe, wizigje asjebleaft de MX net en sla dizze stap oer.**

Bewurkje `conf/chasquid/domains/wac.tax/aliases` , set trochstjoerpostfak yn

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/OBDl2gw.webp)

`*` jout alle e-mails oan, `i` is it e-mailadres foarheaksel fan de stjoerende brûker hjirboppe oanmakke. Om e-post troch te stjoeren, moat elke brûker in rigel tafoegje.

Foegje dan it MX-record ta (ik wiis direkt nei it adres fan 'e omkearde domeinnamme hjir, lykas werjûn yn' e earste rigel yn 'e figuer hjirûnder).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/7__KrU8.webp)

Nei't de konfiguraasje foltôge is, kinne jo oare e-adressen brûke om e-post te stjoeren nei `i@wac.tax` en `any123@wac.tax` om te sjen oft jo e-mails kinne ûntfange yn Gmail.

As net, kontrolearje dan it chasquid-log ( `grep chasquid /var/log/syslog` ).

## Stjoer in e-post nei i@wac.tax mei Google Mail

Neidat Google Mail de mail krige, hope ik fansels te antwurdzjen mei `i@wac.tax` ynstee fan i.wac.tax@gmail.com.

Gean nei [https://mail.google.com/mail/u/1/#settings/accounts](https://mail.google.com/mail/u/1/#settings/accounts) en klikje op "Noch in e-postadres tafoegje".

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/PAvyE3C.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/_OgLsPT.webp)

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/XIUf6Dc.webp)

Fier dan de ferifikaasjekoade yn ûntfongen troch de e-post wêrnei't trochstjoerd is.

Uteinlik kin it ynsteld wurde as it standert stjoerderadres (tegearre mei de opsje om te antwurdzjen mei itselde adres).

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/a95dO60.webp)

Op dizze manier hawwe wy de oprjochting fan 'e SMTP-posttsjinner foltôge en tagelyk Google Mail brûke om e-mails te ferstjoeren en te ûntfangen.

## Stjoer in test-e-post om te kontrolearjen oft de konfiguraasje suksesfol is

Fier `ops/chasquid` yn

Run `direnv allow` om ôfhinklikens te ynstallearjen (direnv is ynstalleare yn it foarige inisjalisaasjeproses mei ien kaai en in heak is tafoege oan 'e shell)

dan rinne

```
user=i@wac.tax pass=xxxx to=iuser.link@gmail.com ./sendmail.coffee
```

De betsjutting fan 'e parameters is as folget

* brûker: SMTP brûkersnamme
* pass: SMTP wachtwurd
* oan: ûntfanger

Jo kinne in test-e-post stjoere.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/ae1iWyM.webp)

It is oan te rieden om Gmail te brûken om test-e-mails te ûntfangen om te kontrolearjen oft de konfiguraasjes suksesfol binne.

### TLS standert fersifering

Lykas yn 'e ûndersteande figuer te sjen is, is d'r dit lytse slot, wat betsjut dat it SSL-sertifikaat mei súkses ynskeakele is.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/SrdbAwh.webp)

Klikje dan op "Orizjinele e-post sjen litte"

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/qQQsdxg.webp)

### DKIM

Lykas werjûn yn 'e ûndersteande figuer, toant de orizjinele e-postside fan Gmail DKIM, wat betsjut dat de DKIM-konfiguraasje suksesfol is.

![](https://pub-b8db533c86124200a9d799bf3ba88099.r2.dev/2023/03/an6aXK6.webp)

Kontrolearje de Untfongen yn 'e kop fan' e orizjinele e-post, en jo kinne sjen dat it stjoerderadres IPV6 is, wat betsjut dat IPV6 ek mei súkses konfigureare is.
