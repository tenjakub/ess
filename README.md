Tato uživatelská dokumentace popisuje kompletní postup instalace řešení Elastic SIEM a způsob napojení koncových zařízení pro sběr dat. Součástí je také identifikace možných problémů, které by se mohly v průběhu instalačního skriptu vyskytnout včetně přístupu k potenciálním chybám v průběhu vlastní instalace.

===Instalovaná služba===
Skript v rámci instalace systému nainstaluje a nakonfiguruje tři jednotlivé programy, ze kterých se skládá základní Elastic Stack. Konkrétně se jedná o Elasticsearch, Kibana a Logstash. Všechny tyto aplikace jsou stahovány z oficiálního repositáře Elastic pro verzi 8.x za pomocí správce balíčků dnf, anebo apt-get v závislosti na distribuci OS. Toto řešení zajišťuje nainstalování nejaktuálnější možné verze těchto programů a zároveň se správce balíčků postará o správnou instalaci aplikací. V rámci těchto aplikací je také automaticky stažen balíček java, který je nutný pro jejich fungování.

===Požadavky===
Instalační skript je určen na instalaci řešení Elastic SIEM na zařízení s operačním systémem Linux.  Veškeré součásti řešení SIEM jsou instalovány na jediné zařízení, které musí splňovat následující parametry: 
-	Operační systém Linux distribuce založené na RedHat nebo Debian
-	Splnění minimálních systémových požadavků aplikací Elasticsearch, Kibana a Logstash ve verzi 8
-	Připojení k internetu
-	Plný přístup k administrátorskému účtu root
Do systému jsou zasílána data z koncových zařízení (agentů). Nainstalované řešení přímo obsahuje podporu pro sběr dat ze zařízení s operačními systémy Linux a Windows. Případné jiné možnosti sběru dat musí být doinstalovány manuálně. Pro Windows a Linux agenty platí následné požadavky:
-	Splnění minimálních systémových požadavků Elastic Agent ve verzi 8
-	Internetové připojení k centrálnímu zařízení s nainstalovaným systémem Elastic SIEM
-	Připojení k internetu minimálně po dobu instalace
 
===Průběh instalace===
Tato část dokumentace popisuje postup při používání instalačního skriptu včetně  kroků potřebných k správnému napojení agentů k systému. Dále řeší nejčastější chyby a problémy, které mohou v průběhu práce instalačního skriptu nastat a jak se s nimi potýkat.

===Před spuštěním skriptu===
Instalační skript je dostupný ke stažení z repositáře GitHub, který je dostupný na internetové adrese https://github.com/mitloja20/ess/ ve formě souboru ZIP. Soubor je doporučeno před využitím skriptu stáhnout na místo, kde bude dobře přístupný. Soubor lze stáhnout a následně rozbalit s využitím následujících příkazů:

[root@server ]# curl -L -o ./ess.zip https://github.com/mitloja20/ess/archive/main.zip 
[root@server ]# unzip ./ess.zip

Tímto získáme základní složku skriptu. Pro nás jsou důležité soubory README.md se zkrácenou verzí této dokumentace a poté install.sh, který je samotný instalační skript. Před jeho spuštěním je však nutné zajistit, aby měl soubor potřebná oprávnění pro spuštění. Změnu těchto oprávnění učiníme za využití příkazu:

[root@server ]# chmod 744 ./ess-main/install.sh

Jako poslední před spuštěním skriptu je třeba se ujistit, že aktuálně využíváme terminál pod uživatelem root, jinak skript nelze spustit. Také je doporučeno si zkontrolovat, jakou IP adresu naše zařízení využívá. Skript totiž potřebuje při instalaci zadat IP adresu, na které bude služba spuštěna. Po dokončení těchto kroků již lze spustit skript:

[root@server ]# ./ess-main/install.sh

===Průběh instalace skriptu===
Po spuštění skript vytvoří trvalou složku instalačního skriptu, která se základně nachází v /opt/ess-install/. V této složce se nachází soubor ess-install.log, do kterého se zaznamenává průběh skriptu, včetně chybových hlášení. Dále se zde nachází složka reserved, ve které se nacházejí zálohy souborů, které skript během instalace přepisuje, kdyby nastala chyba a bylo potřeba původní soubor obnovit. Jako poslední je zde vytvořena složka ssl, ve které se nachází certifikační autorita a certifikáty pro sběrače dat.

Instalace vyžaduje zadat několik hlavních údajů pro konfiguraci. Správnost těchto údajů skript kontroluje pouze na velmi základní rovině, je tedy důležité, aby byly zadány správné. Pro opuštění skriptu stačí zvolit u jakéhokoliv okénka možnost “No“, nebo možnost “Cancel“. 

Jako první je třeba do skriptu zadat validní IPv4 adresu, na které budou veškeré služby provozovány. Tuto IP adresu musí mít zařízení přidělenou na funkčním síťovém rozhraní, které má přístup do požadované sítě, jinak nebude možné služby správně spustit. 

Následně zadáme port, na kterém bude po dokončení instalace provozováno webové rozhraní pro aplikaci Kibana. Toto rozhraní je hlavním přístupovým bodem do celé aplikace a je tak vhodné použít snadno zapamatovatelné číslo portu. Na tomto portu je provozováno pouze webové rozhraní na protokolu HTTPS pro uživatele aplikace a v případě potřeby není složité tento port po dokončení instalace změnit.
 
Dále je třeba zadat jméno pro instalovaný server. Toto jméno je využíváno především pro identifikaci zařízení a zobrazovací účely. Dle zadaného jména je pojmenován cluster a stejně tak dle něj lze identifikovat zařízení uvnitř aplikací.
 
Jako poslední je třeba zadat heslo pro hlavní administrátorský účet s názvem elastic, které musí dosahovat délky minimálně 6 znaků. Tento účet má v aplikaci ta nejvyšší oprávnění a je proto doporučeno se ujistit, aby bylo toto heslo používáno bezpečně.
 
Před samotným spuštěním instalace je zobrazeno dialogové okno s informací, že skript začne vykonávat vlastní instalaci. To je poslední možností přerušit instalaci, tudíž v případě nejistoty o správnosti některé ze zadaných hodnot je doporučeno skript přerušit a údaje zkontrolovat. Po zahájení instalace již skript pracuje na instalaci samostatně. Tento proces může nějakou dobu trvat, jelikož se během něj musí nainstalovat všechny aplikace. Pro případy, kdy nastane během instalace nějaký problém slouží součást dokumentace "Chybové stavy".
 
===Po dokončení práce skriptu===
Zobrazením textu "The script has succesfully completed the Elastic SIEM installation process" skript oznamuje úspěšné dokončení instalace hlavních služeb. V případě vypsání upozornění “WARNING: The installation encountered x errors during the installation process“ je třeba postupovat dle postupů v části dokumentace "Chybové stavy". 
Pro fungování aplikace je potřeba ještě povolit používané porty ve firewallu. Konkrétně je potřeba trvale povolit porty TCP 5044, 5055 a port, který jsme si zvolili pro webové rozhraní.
Přístup do webového rozhraní je zpřístupněný skrze internetový prohlížeč na adrese https://zvolena-ip-adresa:zvoleny-port-pro-web-rozhrani. Při prvním přístupu bude prohlížeč varovat před nebezpečím, jelikož skript využívá vlastně vygenerované certifikáty, u kterých prohlížeč nezná jejich certifikační autoritu, kterou skript vygeneroval. Můžeme tedy toto upozornění ignorovat.
Pro přístup do webového rozhraní slouží základní uživatel elastic s heslem, které bylo zvoleno v průběhu instalace. Pro vytváření dalších uživatelů je již možné využít přímo webového rozhraní Kibana. Po prvním přihlášení se také systém může zeptat, jestli má uživatel zájem o to přidat integrace. V tomto případě zvolíme možnost „I will explore on my own“.

===Přidání bezpečnostních pravidel===
Ke kompletnímu zprovoznění aplikace je konečně třeba ještě přidat bezpečnostní pravidla, která je však třeba přidat manuálně dle následujícího postupu. 
V aplikaci Kibana je prvně třeba využít vyhledávání ve vrchní části rozhraní, zadat „rules“ a vybrat možnost „Security / Rules / Detection rules (SIEM)“. Ta míří na stránku bezpečnostních pravidel, kde je třeba kliknout na tlačítko “Add Elastic rules“ a následně tlačítko „Install all“. 
Po doinstalování pravidel zvolíme „Go back to installed Elastic rules“. Pravidla je nutné následně duplikovat, jelikož u těchto oficiálních pravidel není možné provádět změny. To učiníme tak že, na hlavní obrazovce vybereme veškerá nainstalovaná pravidla za pomoci tlačítka „Select all XXX rules“. Po vybrání všech pravidel je třeba je duplikovat z pomocí tlačítka „Bulk actions“ a následně možnosti „Duplicate“.
Tím začne systém nainstalovaná pravidla duplikovat. Po dokončení procesu je v pravém spodním rohu rozhraní vypsána chybová hláška "Error duplicating rule" oznamující, že některá pravidla nebylo možné duplikovat. To však není problém, jelikož pro některá pravidla je nutné disponovat placenou licencí pro užívání možností umělé inteligence. Ve standartním procesu instalace není tato licence zahrnuta, tudíž tato pravidla nemohou být duplikována.
Po duplikování pravidel již není třeba mít v systému původní pravidla, tudíž je vhodné tato pravidla odstranit. Toho lze dosáhnout zvolením základních pravidel tlačítkem „Elastic Rules“, poté výběrem všech pravidel, zvolením "Bulk actions" a následně možností „Delete“.
Po odstranění původních pravidel nastává čas změnit zdroj vyhledávání pro duplikovaná pravidla. K tomu je potřeba vybrat „Custom rules“, rozbalit možnost „Index patterns“ a poté zakliknout „Add index patterns“.
Po tomto kroku v následující nabídce v poli „Add index patterns for selected rules“ zvolíme z výběru možnost „logs-*“, zakliknout možnost „Overwrite all selected rules‘ patterns“ pro přepsání původních hodnot a následně potvrdit akci. Provedení této akce opět chvíli potrvá.
Jako poslední krok již stačí upravená pravidla aktivovat. K tomu stačí mít zvolená veškerá pravidla, zvolit možnost „Bulk actions“ a zvolit možnost „Enable“. Provedení této akce také chvíli potrvá. Po jejím dokončení jsou pravidla úspěšně spuštěna a budou následně automaticky kontrolovat data v systému. Pro přístup k upozorněním, která tato pravidla vytváří, slouží ve webovém rozhraní sekce "Security>Alerts".
Je důležité podotknout, že na duplikovaná pravidla se nebudou nadále automaticky přenášet aktualizace výrobce. Je proto doporučeno tato duplikovaná pravidla průběžně aktualizovat za použití tohoto instalačního procesu. 

===Vytvoření konfigurace pro agenty===
Přidávání agentů je z důvodu kompatibility s různými typy zařízení zpřístupněno za pomoci manuálního postupu. Tento postup je prakticky stejný pro Linux i Windows agenty. První část tohoto procesu je přístupná skrze webové rozhraní Kibana, kde je třeba získat konfiguraci, kterou budou agenti využívat. Jako první je třeba za pomoci vyhledávacího pole ve vrchní části rozhraní vyhledat „agent policies“ a poté vybrat zvolit možnost „Fleet / Agent policies“.

V následují nabídce je možné spravovat politiky pro agenty. Instalační skript automaticky vytvoří dvě. Jednu pro agenty se systémem linux a druhou pro agenty se systémem Windows. Také je možné si vytvořit vlastní politiky s vlastními integracemi pro jiné specifické aplikace, které toto řešení podporuje. Proces instalace je poté shodný. Pro pokračování v přidávání agenta je třeba vybrat politiku pro instalovaného agenta.

Následující rozhraní poskytuje možnost pracovat s danou politikou. Lze do ní takto přidávat nové integrace, modifikovat ty existující, nebo právě možnost přidělení této politiky nějakému agentovi za pomoci tlačítka „Actions“ a možnosti „Add agent“.
 
Skript pracuje s možností samostatně spravovaných agentů, tudíž je nutné v následující nabídce zvolit možnost „Run standalone“. Tak je zobrazena konfigurace pro tuto politiku, kterou je možné zkopírovat do schránky, nebo uložit do souboru. Tuto konfiguraci je bude třeba pro správné fungování mírně modifikovat.

Tento konfigurační soubor má v základní podobě nastavený výstup přímo do aplikace Elasticsearch. Toto řešení však využívá pro příjem dat nástroj Logstash, tudíž je třeba konfigurační soubor upravit.
Konkrétně je zde potřeba nahradit část s definováním výstupu (viz. příklad) obsahem souboru složka-skriptu/config-files/elastic-agent-windows-output, nebo složka-skriptu/config-files/elastic-agent-linux-output v závislosti na operačním systému zařízení, na které je třeba agenta nainstalovat a také přepsat údaj YOUR_SERVER_IP_HERE za IP adresu zařízení s centrálním systémem.

tato část je třeba nahradit:

outputs:
  default:
    type: elasticsearch
    hosts:
      - 'http://localhost:9200'
    username: '${ES_USERNAME}'
    password: '${ES_PASSWORD}'
    preset: balanced

Takto upravený konfigurační soubor je poté třeba následně uložit. Je doporučeno jej ukládat v podobě /opt/ess-install/agents/slozka-integrace/elastic-agent.yml. Tento konfigurační soubor je pro všechny agenty využívající danou politiku shodný, tudíž stačí vytvořit pouze jednou.
V případě problému s generováním konfiguračního souboru se v instalační složce skriptu nacházejí ve složce config-files již hotové konfigurace, ve kterých pouze stačí doplnit IP adresu centrálního serveru. Tyto konfigurace však nemusí být zcela aktuální a je tedy doporučeno následovat tento postup. 

===Instalace a napojení agentů===
Po vytvoření konfigurace je možné začít instalovat agenty. Příkazy k jejich instalaci lze nalézt přímo v rozhraní Kibana, přičemž jsou umístěny o něco níž, jak výpis konfigurace v postupu pro získávání konfiguračních souborů. Tyto příkazy jsou dostupné pro instalaci na potřebnou platformu, přičemž pro instalaci na zařízení s operačním systémem Linux je doporučeno využít možnosti „Linux Tar“. Tyto příkazy lze zkopírovat do schránky za použití tlačítka v pravém horním rohu pole, kde jsou příkazy vypsány.
Pro spuštění těchto příkazů na koncovém zařízení, kam je Elastic Agent pro sběr dat do systému instalován, je doporučeno využít terminálové relace s administrátorskými oprávněními. Jejich spuštěním bude automaticky nainstalována aplikace Elastic Agent. Při instalaci třeba zvolit u otázky „Do you want to enroll this Agent into Fleet?“ zvolit možnost “ne“. V případě následování příkazů instalace u zařízení s operačním systémem Windows je možné, že je třeba u cest některých příkazů nahradit syntaxi .příkaz za .\příkaz. 

Pro plnou funkčnost sběru dat je na zařízeních s operačním systémem Windows potřebné před dokončením instalace aplikace Elastic Agent nainstalovat balíček Sysmon. Ten je třeba stáhnout ze stránky https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon, extrahovat stažený ZIP soubor a poté se skrze aplikaci Windows Powershell (spustit jako Administrátor) za pomocí příkazu cd přemístit do složky, kde se extrahované soubory nachází. V případě stažení do Downloads tak příkaz vypadá:
PS C:\\Windows\system32> cd ‘C:\Users\jmenouzivatele\Downloads\Sysmon‘
Po přesunutí do příslušné složky lze balíček nainstalovat za pomoci příkazu:
PS C:\Users\jmenouzivatele\Downloads\Sysmon> .\Sysmon64.exe -i- accepteula
Tímto se automaticky nainstaluje balíček Sysmon, který je také rovnou nastaven na automatické spuštění při startu systému. Zda-li je spuštěn lze zkontrolovat za pomocí Windows aplikace „Services“.

Aplikace Elastic Agent je následně u zařízení s operačním systémem Windows nainstalována v C:\Program Files\Elastic\Agent\ a u zařízení s operačním systémem Linux se za využití instalace skrze Linux tarball nachází v /opt/Elastic/Agent/. V této hlavní složce je třeba vytvořit složku pro soubory certifikátů „certs“ (pro správné fungování musí být jméno zcela shodné). Před kopírováním konfiguračních souborů je také třeba se ujistit, že je služba Elastic Agent vypnutá.

Po vytvoření složky pro certifikáty je již možné začít kopírovat potřebné soubory z centrálního zařízení se systémem na koncová zařízení s agenty. Ti potřebují následující soubory:
Soubor	            Cesta na centrálním zařízení	           Cesta na agentovi
===================================================================================
ca.crt	             /opt/ess-install/ssl/ca/	                Podložka certs
es-agent.crt	       /opt/ess-install/ssl/agents/es-agent/	   Podložka certs
es-agent.key	       /opt/ess-install/ssl/agents/es-agent/	   Podložka certs
elastic-agent.yml  	/opt/ess-install/agents/jmenointegrace/	 Hlavní složka aplikace

Pro přenos souborů mezi zařízeními lze zvolit jakoukoliv vyhovující metodu přenosu. Jedno z možných řešení pro přesunutí těchto souborů je využití protokolu SCP, který využívá SSH připojení. Pokud je mezi oběma potřebnými zařízeními zřízeno SSH spojení, je možné využít následujících příkazů. 
 
Při přenosu na agenty s operačním systémem windows:
[root@server ]# scp /opt/ess-install/ssl/agents/es-agent/* jmenouzivatele@IPadresa:'"/c:/program files/elastic/agent/certs"'
[root@server ]# scp /opt/ess-install/agents/windows/elastic-agent.yml jmenouzivatele@IPadresa:'"/c:/program files/elastic/agent/"'

Při přenosu na agenty s operačním systémem linux:
[root@server ]# scp /opt/ess-install/ssl/agents/es-agent/* jmenouzivatele@IPadresa:/opt/Elastic/Agent/certs/
[root@server ]# scp /opt/ess-install/agents/linux/elastic-agent.yml jmenouzivatele@IPadresa:/opt/Elastic/Agent/

V příkazech je třeba nahradit jmenouzivatele za jméno uživatele s dostatečnými oprávněními a IPadresa za IP adresu požadovaného koncového zařízení, na které jsou soubory přenášeny. 
Po přenesení veškerých souborů lze službu Elastic Agent spustit. U zařízení s operačním systémem Windows toho učiníme za použití služby „Services“, u zařízení s operačním systémem linux za použití příkazu „systemctl“.
Tímto by měl agent zasílat data do systému. Funkčnost spojení lze ověřit za pomoci log souboru služby elastic-agent-datum.ndjson, ve kterém fungující spojení značí zpráva: "Connection to backoff(async(tcp://adresacentralnihozarizeni:5055)) established", případně zkontrolováním dat nástrojem discovery v aplikaci Kibana, kde by se měla zobrazit data z tohoto agenta.

===Chybové stavy===
V průběhu práce instalačního skriptu může dojít k určitým problémům, které zabraňují správnému průběhu instalace řešení Elastic SIEM. Tyto chyby nemusí však být pro průběh instalačního skriptu kritické a ten tak může samotnou instalaci dokončit a na konci procesu instalace vydat uživateli upozornění. V takovém případě lze buď dle kapitoly "Odinstalování aplikace" kompletně vymazat nainstalovaný systém a skript spustit znovu, nebo vzniklé chyby opravit za pomoci této části dokumentace.

Po výskytu chyby “ERROR: jmenouzivatele password has not been created.“ je průběh instalace automaticky ukončen. Tato chyba znamená, že nemohlo být vygenerováno heslo pro jednoho z interních uživatelů. V takovém případě je nutné se ujistit, že plně funguje balíček openssl, který je využíván ke generování hesel, a poté skript spustit znovu.

Další možné chyby již průběh instalačního skriptu nezastaví a skript pouze po dokončení svojí činnosti vypíše, že v průběhu práce skriptu se naskytla chyba. Pro zjištění konkrétní chyby je třeba následně využít souboru /opt/ess-install/ess-install.log nebo upozornění vypsaných v terminálu k nalezení jejich přesné podoby.

Chybová hláška "ERROR: jmenosluzby certificates have NOT been generated" se naskytne při problému s generováním certifikátu pro určité služby. V takovém případě je třeba chybějící certifikáty doplnit ručně ve formátu popsaném v kapitole "Šifrování komunikace za pomoci SSL" dle postupu v kapitole "Generování nových SSL certifikátů".

Chybová hláška "ERROR: jmenopolitiky agent policy has NOT been created" nebo “ERROR: jmenointegrace integration has NOT been added to agent policy“ značí, že při průběhu instalačního skriptu nastala chyba při vytváření politik pro agenty, případně při přidávání integrací do těchto politik. 
Prvním možným řešením je využít předem vytvořené konfigurace agentů obsažené v instalační složce skriptu ve složce config-files/jmenointegrace-agents a následně pozměnit IP adresu dle postupu pro vytváření konfiguračních souborů pro agenty. Druhou možností je tyto politiky s integracemi vytvořit manuálně. To lze učinit skrze rozhraní pro správu politik agentů pomocí tlačítka “Create agent policy“ a vytvořenou politiku následně rozkliknout, čímž se dostaneme do rozhraní pro správu politiky. Zde je již možné přidat integraci Windows, nebo Auditd Logs (pro Linux agenty) tlačítkem “Add integration“. Tím je politika pro agenty vytvořena.

Chybová hláška "ERROR: Logstash internal user has NOT been created" nebo "ERROR: Logstash writer role has NOT been created" značí problém s vytvářením interního logstash uživatele a jeho role. V takovém případě je třeba jej vytvořit manuálně skrze webové rozhraní Kibana za pomoci postupu v sekci Configuring Logstash to use basic authentication v následující části oficiální dokumentace: https://www.elastic.co/guide/en/logstash/current/ls-security.html.

===Odinstalování aplikace===
V případě potřeby kompletního odinstalování systému je třeba pro plné vymazání veškerých dat také kromě odinstalování samotných aplikací Elasticsearch, Kibana a Logstash ještě vymazat obsah určitých složek, které mohou stále obsahovat data. Tento postup slouží především pro případy, kdy je třeba aplikace vymazat a přeinstalovat od začátku. Pro standartní vymazání aplikací toto není přímo nutné provádět.
Tyto složky stále po odinstalování obsahují konfigurační soubory:
/etc/elasticsearch/
/etc/kibana/
/etc/logstash/

Tyto složky obsahují samotná data aplikací, včetně veškerých uložených dat:
/var/lib/elasticsearch/
/var/lib/kibana

Pro odinstalování agenta z koncového zařízení slouží následující část oficiální dokumentace Elastic: https://www.elastic.co/guide/en/fleet/current/uninstall-elastic-agent.html

===Šifrování komunikace za pomoci SSL===
Další součástí nastavení zabezpečení, kterou skript vykonává, je konfigurace šifrování síťového provozu mezi částmi systému za použití self-signed SSL certifikátů. Pro tyto účely disponuje Elasticsearch nástrojem elasticsearch certutil pro generování vlastních self-signed SSL náležitostí, jehož funkcí skript využívá. Skript tak vygeneruje vlastní certifikační autoritu, která poté podepisuje veškeré potřebné certifikáty. Tyto certifikáty jsou vytvářeny na dobu 5 let a některé musí mít definované IP adresy, které figurují v komunikaci, jako SAN hodnotu certifikátu. Skript generuje tyto certifikáty a certifikační autority:

Účel	IP adresy	Název souborů	                                                            Umístění
certifikační autorita	-	ca.crtca.key	                                                    /opt/ess-install/ssl/ca/
komunikace na Elasticsearch API	127.0.0.1, IP adresa centrálního serveru	es-http.crt
                                                                         es-http.key	    /etc/elasticsearch/certs/
komunikace mezi internetovým prohlížečem a webovou aplikací Kibana	IP adresa centrálního serveru	kibana-server.crt
kibana-server.key	/etc/kibana/certs/
komunikace s nástrojem Logstash (strana serveru)	IP adresa centrálního serveru	logstash-input.crt
logstash-input.key	/etc/logstash/certs/
komunikace s nástrojem Logstash (strana zdroje dat)	-	es-agent.crt
es-agent.key	/opt/ess-install/ssl/agents/es agent/


Je důležité poznamenat, že tyto certifikáty neobsahují SAN hodnotu pro DNS názvy zařízení. V případě potřeby využívat identifikace zařízení za pomoci DNS pojmenování je třeba změnit nastavení řešení a potřebné certifikáty vygenerovat znova.
Tyto certifikáty jsou také využívány pro zašifrovanou komunikaci mezi sběrači dat z koncových zařízení (agenty) a nástrojem Logstash. Zde je používána oboustranná autentizace, kde se za pomocí certifikátu musí prokázat jak cíl pro zasílání dat (Logstash), tak jejich zdroj (agenti). Toto řešení zvyšuje míru zabezpečení, kdy pro zasílání dat je třeba disponovat správnými soubory certifikátů pro ověření. Je však třeba zmínit, že za účelem zjednodušení procesu nasazování agentů na koncová zařízení a jejich správy je u veškerých agentů využívána pouze jediná shodná sada certifikátu a klíče. Toto platí jak pro vstupy dat ze sběračů Elastic Agent, tak ze sběračů Beats.

===Generování nových SSL certifikátů===
Základním způsobem získání SSL certifikátů potřebných pro fungování systému je terminálový nástroj elasticsearch-certutil, který je součástí instalace aplikace Elasticsearch. Tento nástroj lze nalézt na: /usr/share/elasticsearch/bin/elasticsearch-certutil.
Tento skript může generovat různé typy SSL souborů, pro případ generování nových certifikátů slouží možnost cert. Při využití v této instalaci budou mít veškeré příkazy pro generování certifikátů následnou strukturu:

[root@server]# /usr/share/elasticsearch/bin/elasticsearch-certutil cert --ca-cert /opt/ess-install/ssl/ca/ca.crt --ca-key /opt/ess-install/ssl/ca/ca.key --days 1826 --name JMENOCERTIFIKATU --out /opt/ess-install/ssl/JMENOCERTIFIKATU.zip --pem

Do tohoto příkazu je třeba přidat parametr --ip IPADRESA, pokud je třeba aby generovaný certifikát měl přidělenou IP adresu nebo parametr --dns DNSJMENO pro přidělení DNS jména k certifikátu. Certifikáty, které mají tento údaj obsahovat jsou popsány v kapitole "Šifrování komunikace za pomoci SSL". Pro použití je třeba nahradit hodnoty JMENOCERTIFIKÁTU vlastní hodnotou, nejlépe shodnou s původními názvy dle popisu v kapitole "Šifrování komunikace za pomoci SSL". Tato hodnota slouží především k organizaci, tudíž tolik nezáleží na její přesné podobě. Je důležité využít možnosti --pem umístěné na konci příkazu pro zachování správného formátu certifikátů.
Tento certifikát je následně vygenerován ve formě ZIP souboru, který je uložený ve složce /opt/ess-install/ssl/. Tento soubor je třeba extrahovat a soubory, které obsahuje přesunout na příslušné místo dle popisu v kapitole "Šifrování komunikace za pomoci SSL". Následně je třeba restartovat veškeré služby, pro které byl certifikát změněn.

===Změna hesel pro interní uživatele===
Pro interní komunikaci mezi jednotlivými aplikacemi jsou využíváni interní uživatelé, pro které je během instalace automaticky vygenerováno náhodné heslo. Tito uživatelé jsou u konfigurace instalované za pomoci instalačního skriptu kibana_system pro komunikaci aplikace Kibana s databází Elasticsearch a logstash_internal pro komunikaci mezi nástrojem Logstash s databází Elasticsearch. Ke změně některého z těchto hesel je třeba využít následující příkaz:

[root@server]# /usr/share/elasticsearch/bin/elasticsearch-reset-password -i -b -u JMENOUZIVATELE

Po samotné změně hesel pro tyto uživatele je dále potřeba tato hesla změnit v cílových aplikacích, které tyto uživatele využívají. Tato data jsou uložena na zabezpečeném místě, které se nazývá keystore. Pro změnu hesel využijte následující příkazy a následně nechte přepsat původní hodnotu.

Pro přepsání hesla uživatele kibana_system v kibana keystore:
[root@server ]# /usr/share/logstash/bin/logstash-keystore --path.settings /etc/logstash add ES_PWD

Pro přepsání hesla uživatele logstash_internal v logstash keystore:
[root@server ]# /usr/share/kibana/bin/kibana-keystore add elasticsearch.password
