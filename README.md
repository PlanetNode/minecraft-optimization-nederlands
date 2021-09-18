# Minecraft server optimalisatie gids

Opmerking voor gebruikers die op vanilla, Fabric of Spigot zitten (of alles lager dan Paper) - ga naar je server.properties en verander `sync-chunk-writes` naar `false`. Deze optie is geforceerd uitgeschakeld op Paper en zijn afgeleiden, maar op andere server implementaties moet je dit handmatig uitzetten. Dit stelt de server in staat om chunks op te slaan buiten de hoofdthread om, waardoor de belasting op de hoofd tick loop afneemt.

Handleiding voor versie 1.17. Sommige dingen kunnen nog van toepassing zijn op 1.15 - 1.16.

Gebaseerd op [deze gids](https://www.spigotmc.org/threads/guide-server-optimization%E2%9A%A1.283181/) en andere bronnen (ze zijn allemaal gelinkt in de gids wanneer ze relevant zijn).

# Intro
Er zal nooit een gids zijn die je perfecte resultaten geeft. Elke server heeft zijn eigen behoeften en grenzen aan hoeveel je kunt of wilt opofferen. Knutselen aan de opties om ze af te stemmen op de behoeften van je server is waar het allemaal om draait. Deze gids is alleen bedoeld om u te helpen begrijpen welke opties invloed hebben op de prestaties en wat ze precies veranderen.

# Voorbereidingen

## Server JAR
Je keuze van server software kan een groot verschil maken in prestatie en API mogelijkheden. Er zijn op dit moment meerdere levensvatbare populaire server JARs, maar er zijn er ook een paar waar je om verschillende redenen beter weg van kunt blijven.

Aanbevolen top picks:
* [Paper](https://github.com/PaperMC/Paper) - De meest populaire server software die gericht is op het verbeteren van de prestaties terwijl het gameplay en mechanica inconsistenties repareert.
* [Airplane](https://github.com/Technove/Airplane) - Paper vork die tot doel heeft de server prestaties verder te verbeteren.
* [Purpur](https://github.com/pl3xgaming/Purpur) - Airplane vork gericht op functies en de vrijheid van aanpassing.

Je moet wegblijven van:
* Yatopia & Sugarcane - "De gecombineerde kracht van Paper Forks voor maximale instabiliteit en unmaintainablity!" - [KennyTV's lijst van schaamte](https://github.com/KennyTV/list-of-shame). Niets meer over te zeggen. (Bovendien is het project stopgezet).
* Mohist - "Mohist is geprogrammeerd om kwaadaardig te zijn, spelbrekend, en zeer onstabiel!" - [Redenen waarom je het niet zou moeten gebruiken](https://essentialsx.net/do-not-use-mohist.html)
* Elke betaalde server JAR die async claimt - 99.99% kans dat het oplichterij is.
* Bukkit/CraftBukkit/Spigot - Extreem verouderd in termen van performance vergeleken met andere server software waar je toegang tot hebt.
* Elke plugin/software die plugins inschakelt/uitschakelt/laadt op runtime. Zie [deze sectie](#plugins-enablingdisabling-other-plugins) om te begrijpen waarom.
* Veel forks verder stroomafwaarts van Airplane of Purpur zullen instabiliteit en andere problemen tegenkomen. Als u op zoek bent naar meer prestatiewinst, optimaliseer dan uw server of investeer in een persoonlijke private fork.

## Map pregen
Map pregeneratie is een van de belangrijkste stappen in het verbeteren van een low-budget server. Dit helpt servers die gehost worden op een gedeelde CPU/single core node het meest, omdat ze niet volledig gebruik kunnen maken van async chunk loading. Je kunt een plugin zoals [Chunky](https://github.com/pop4959/Chunky) gebruiken om de wereld voor te genereren. Zorg ervoor dat je een wereldgrens instelt zodat je spelers geen nieuwe chunks genereren! Merk op dat het pregennen soms uren kan duren, afhankelijk van de radius die je instelt in de pregen plugin.

Het is belangrijk om te onthouden dat de bovenwereld, de nether en het einde aparte wereldgrenzen hebben die voor elke wereld ingesteld moeten worden. De nether dimensie is 8x kleiner dan de overworld (indien niet aangepast met een datapack), dus als je de grootte verkeerd instelt, kunnen je spelers buiten de wereldgrens terechtkomen!

**Zorg ervoor dat je een vanille wereldgrens instelt (`/worldborder set [radius]`), omdat het bepaalde functionaliteiten beperkt, zoals het opzoekbereik voor schatkaarten dat lag spikes kan veroorzaken.**

# Configuraties

## Networking

### [server.properties]

#### network-compression-threshold

`Goede startwaarde: 256`

Dit staat je toe om een maximum in te stellen voor de grootte van een pakket voordat de server het probeert te comprimeren. Hoger instellen kan wat CPU-bronnen besparen ten koste van bandbreedte, en op -1 zetten schakelt het uit. Hoger instellen kan ook cliënten met tragere netwerkverbindingen schaden. Als je server zich in een netwerk met een proxy bevindt of op dezelfde machine (met minder dan 2 ms ping), zal het uitschakelen (-1) voordelig zijn, aangezien interne netwerksnelheden gewoonlijk het extra ongecomprimeerde verkeer aankunnen.

### [purpur.yml]

#### use-alternate-keepalive

`Goede startwaarde: true`

Je kunt Purpur's alternatieve keepalive systeem aanzetten, zodat spelers met een slechte verbinding niet zo vaak uitgetimed worden. Heeft bekende incompatibiliteit met TCPShield.

> Het inschakelen hiervan stuurt een keer per seconde een keepalive pakket naar een speler, en kopt alleen voor timeout als er niet op gereageerd is binnen 30 seconden. Reageren op elk van deze in elke volgorde zal de speler verbonden houden. AKA, het zal je spelers niet kicken omdat 1 pakket ergens langs de lijnen is gevallen  
~ https://purpur.pl3x.net/docs/Configuration/#use-alternate-keepalive

---

## Chunks

### [spigot.yml]

#### view-distance

`Goede startwaarde: 4`

View-distance is de afstand in chunks rond de speler die de server zal aanvinken. In essentie de afstand van de speler waarop dingen zullen gebeuren. Dit is inclusief ovens die smelten, gewassen en jonge boompjes die groeien, etc. Je moet deze waarde instellen in [spigot.yml], omdat het die van [`server.properties`] overschrijft en per wereld kan worden ingesteld. Dit is een optie die je bewust laag wilt zetten, ergens rond `3` of `4`, vanwege het bestaan van `no-tick-view-distance`. No-tick staat spelers toe om meer stukken te laden zonder ze aan te vinken. Dit stelt spelers in staat om verder te kijken zonder dezelfde impact op de prestaties.

### [paper.yml]

#### no-tick-view-distance

`Goede startwaarde: 7`

Deze optie maakt het mogelijk om de maximale afstand in brokken in te stellen die de spelers zullen zien. Dit maakt het mogelijk om een lagere `view-distance` te hebben en spelers toch verder te laten kijken. Het is belangrijk om te weten dat de chunks die verder weg liggen dan de werkelijke `view-distance` niet zullen worden aangevinkt, maar ze zullen wel worden geladen vanuit je opslag, dus ga niet te ver. `10` is in principe het maximum waar je dit op moet zetten. Vanaf nu worden chunks naar de client gestuurd ongeacht hun kijkafstand instelling, dus hogere waardes voor deze optie kunnen problemen veroorzaken voor spelers met langzamere verbindingen.

#### delay-chunk-unloads-by

`Goede beginwaarde: 10`

Met deze optie kun je instellen hoe lang chunks geladen blijven nadat een speler is vertrokken. Dit helpt om niet constant dezelfde chunks te laden en te lossen als een speler heen en weer beweegt. Te hoge waarden kunnen resulteren in veel te veel chunks die in één keer geladen worden. In gebieden die vaak worden geteleporteerd en geladen, kun je overwegen om het gebied permanent geladen te houden. Dit zal lichter zijn voor je server dan het constant laden en lossen van chunks.

#### max-auto-save-chunks-per-tick

`Goede startwaarde: 8`

Hiermee kun je het incrementeel opslaan van de wereld vertragen door de taak nog meer over de tijd te spreiden voor een betere gemiddelde prestatie. Je zou dit hoger kunnen zetten dan `8` met meer dan 20-30 spelers. Als het incrementele opslaan niet op tijd kan worden voltooid, zal bukkit automatisch de overgebleven brokken in een keer opslaan en het proces opnieuw beginnen.

#### prevent-moving-into-unloaded-chunks

`Goede startwaarde: true`

Wanneer ingeschakeld, voorkomt dit dat spelers in onbeladen chunks lopen en sync loads veroorzaken die de main thread vertragen en lag veroorzaken. De kans dat een speler in een onbelaste chunk stapt is groter naarmate de no-tick-view-afstand kleiner is.

#### entity-per-chunk-save-limit

```
Goede beginwaarden:

      experience_orb: 16
      arrow: 16
      dragon_fireball: 3
      egg: 8
      ender_pearl: 8
      eye_of_ender: 8
      fireball: 8
      small_fireball: 8
      firework_rocket: 8
      potion: 8
      llama_spit: 3
      shulker_bullet: 8
      snowball: 8
      spectral_arrow: 16
      experience_bottle: 3
      trident: 16
      wither_skull: 4
      area_effect_cloud: 8
```

Met behulp van dit item kunt u grenzen instellen voor het aantal entiteiten van een bepaald type dat kan worden opgeslagen. Je zou op zijn minst een limiet moeten instellen voor elk projectiel om problemen te voorkomen met enorme hoeveelheden projectielen die worden opgeslagen en je server die crasht bij het laden. Je kan hier elke entiteit id zetten, zie de minecraft wiki om IDs van entiteiten te vinden. Pas de limiet aan naar je eigen smaak. Aanbevolen waarde voor alle projectielen is rond de `10`. Je kunt ook andere entiteiten toevoegen door hun type naam aan deze lijst toe te voegen. Deze configuratie optie is niet ontworpen om te voorkomen dat spelers grote mob farms maken.

#### seed-based-feature-search-loads-chunks

`Goede startwaarde: true`

Hoewel het instellen van dit op `false` de performance zal verbeteren als je `treasure-maps-return-already-discovered` op `false` zet, kan het resulteren in onverwacht gedrag, zoals structuren die soms niet op de plek staan die op de kaart is aangegeven. Zet het aan als je het niet als een probleem ziet.

---

## Mobs

### [bukkit.yml]

#### spawn-limits

```
Goede start waarden:

    monsters: 20
    animals: 5
    water-animals: 2
    water-ambient: 2
    ambient: 1
```

De wiskunde van het limiteren van mobs is `[playercount] * [limit]`, waar "playercount" het huidige aantal spelers op de server is. Logisch, hoe kleiner de aantallen zijn, hoe minder mobs je zult zien. `per-speler-mob-spawn` past hier een extra limiet op toe, om ervoor te zorgen dat mobs gelijk verdeeld worden tussen spelers. Dit verminderen is een tweesnijdend zwaard; ja, je server heeft minder werk te doen, maar in sommige gamemodes zijn natuurlijk spawnende mobs een groot deel van de gameplay. Je kunt zo laag gaan als 20 of minder als je `mob-spawn-range` goed instelt. Door `mob-spawn-range` lager in te stellen zal het lijken alsof er meer mobs rond elke speler zijn. Als je Paper gebruikt, kun je mob-limieten per wereld instellen in [`paper.yml`].

#### ticks-per

```
Goede beginwaarden:

    monster-spawn: 10
    animal-spawns: 400
    water-spawns: 400
    water-ambient-spawns: 400
    ambient-spawns: 400
```

Deze optie stelt in hoe vaak (in ticks) de server probeert om bepaalde levende wezens te spawnen. Water/ambient mobs hoeven niet elke tick te spawnen omdat ze meestal niet zo snel gedood worden. Wat monsters betreft: De tijd tussen spawns lichtjes verhogen zou geen invloed mogen hebben op de spawn rates, zelfs niet in mob farms. In de meeste gevallen zouden alle waardes onder deze optie hoger moeten zijn dan `1`. Door dit hoger te zetten kan je server ook beter omgaan met gebieden waar mob spawn is uitgeschakeld.

### [spigot.yml]

#### mob-spawn-range

`Goede startwaarde: 2`

Staat je toe om het bereik (in brokken) te verkleinen van waar mobs zullen spawnen rond de speler. Afhankelijk van de gamemode van je server en het aantal spelers zou je deze waarde kunnen verlagen samen met [bukkit.yml]'s `spawn-limits`. Door dit lager in te stellen zal het lijken alsof er meer mobs om je heen zijn. Dit moet lager zijn dan of gelijk aan je zichtafstand, en nooit groter dan je harde despawn range / 16.

#### entity-activation-range

```
Goede beginwaarden:

      animals: 16
      monsters: 24
      raiders: 48
      misc: 8
      water: 8
      villagers: 16
      flying-monsters: 48
```

Je kan instellen hoe ver een entiteit van de speler moet zijn om te tikken (dingen te doen). Het verlagen van deze waarden helpt de prestaties, maar kan resulteren in onresponsieve mobs totdat de speler echt dicht bij ze komt. Dit te ver verlagen kan bepaalde mob farms kapot maken; iron farms zijn het meest voorkomende slachtoffer.

#### entity-tracking-range

```
Goede beginwaarden:

      players: 48
      animals: 48
      monsters: 48
      misc: 32
      other: 64
```

Dit is de afstand in blokken vanaf waar entiteiten zichtbaar zullen zijn. Ze zullen alleen niet naar spelers worden gestuurd. Als je dit te laag instelt kan het lijken of een entiteit uit het niets verschijnt in de buurt van een speler. In de meeste gevallen zou dit hoger moeten zijn dan je `entity-activatie-range`.

#### tick-inactive-villagers

`Goede startwaarde: false`

Hiermee kun je bepalen of dorpelingen moeten worden aangevinkt buiten het activeringsbereik. Dit zal ervoor zorgen dat dorpelingen normaal doorgaan en het activeringsbereik negeren. Het uitschakelen hiervan zal de prestaties verbeteren, maar kan verwarrend zijn voor spelers in bepaalde situaties. Dit kan problemen veroorzaken met ijzeren boerderijen en het herbevoorraden van handel.

#### nerf-spawner-mobs

`Goede startwaarde: true`

Je kunt ervoor zorgen dat mobs die gespawned worden door een monster spawner geen AI hebben. Nerfed mobs zullen niets doen. Je kan ze laten springen in water door `spawner-nerfed-mobs-should-jump` te veranderen in `true` in [paper.yml].

### [paper.yml]

#### despawn-ranges

```
Goede start waarden:

      soft: 30
      hard: 56
```

Hiermee kun je de entiteit despawn bereiken aanpassen (in blokken). Verlaag deze waarden om de mobs die ver weg zijn van de speler sneller te verwijderen. Je zou de soft range rond de 30 moeten houden en de hard range iets meer dan je eigenlijke view-distance, zodat mobs niet meteen despawnen als de speler net voorbij het punt gaat waar een chunk geladen wordt (dit werkt goed door de `delay-chunk-unloads-by` in [paper.yml]). Wanneer een mob buiten het harde bereik is, zal deze direct despawned worden. Wanneer tussen de zachte en harde range, zal het een willekeurige kans hebben om te despawnen. Je harde bereik moet groter zijn dan je zachte bereik. Je moet dit aanpassen aan je zichtafstand door `(zichtafstand * 16) + 8` te gebruiken. Dit houdt gedeeltelijk rekening met chunks die nog niet uitgeladen zijn nadat de speler ze bezocht heeft.

#### per-player-mob-spawns

`Goede beginwaarde: true`

Deze optie bepaalt of mob spawns rekening moeten houden met hoeveel mobs er al rond de speler zijn. Je kunt hiermee veel problemen omzeilen met betrekking tot mob spawns die inconsistent zijn doordat spelers farms maken die de hele mobcap in beslag nemen. Dit zal een meer singleplayer-achtige spawn ervaring mogelijk maken, waardoor je lagere `spawn-limieten` kunt instellen. Het inschakelen hiervan heeft een klein effect op de prestaties, maar het effect wordt overschaduwd door de verbeteringen in `spawn-limits` die het mogelijk maakt.

#### max-entity-collisions

`Goede startwaarde: 2`

Overschrijft optie met dezelfde naam in [spigot.yml]. Het laat je beslissen hoeveel botsingen een entiteit in een keer kan verwerken. Waarde van `0` veroorzaakt het onvermogen om andere entiteiten, inclusief spelers, te duwen. Waarde van `2` zou genoeg moeten zijn in de meeste gevallen. Het is de moeite waard om op te merken dat dit maxEntityCramming gamerule nutteloos maakt als zijn waarde hoger is dan de waarde van deze configuratie optie.

#### update-pathfinding-on-block-update

`Goede startwaarde: false`

Het uitschakelen hiervan zal resulteren in minder pathfinding, wat de performance zal verhogen. In sommige gevallen zal dit ervoor zorgen dat mobs meer laggy lijken; Ze zullen gewoon passief hun pad updaten elke 5 ticks (0.25 sec).

#### fix-climbing-bypassing-cramming-rule

`Goede startwaarde: true`

Door dit in te schakelen worden entiteiten die klimmen niet beïnvloed. Dit zal voorkomen dat absurde hoeveelheden mannetjes worden gestapeld in kleine ruimtes zelfs als ze klimmen (spinnen).

#### armor-stands-tick

`Goede beginwaarde: false`

In de meeste gevallen kun je dit veilig op `false` zetten. Als je armor stands gebruikt of plugins die hun gedrag aanpassen en je ondervindt problemen, schakel dit dan weer in. Dit zal voorkomen dat pantserstatieven worden geduwd door water of worden beïnvloed door de zwaartekracht.

#### armor-stands-do-collision-entity-lookups

`Goede beginwaarde: false`

Hier kun je armor stand collisions uitschakelen. Dit zal helpen als je veel armorstands hebt en ze nergens tegenaan wilt laten botsen.

#### tick-rates

```
Goede beginwaarden:

      sensor:
        villager:
          secondarypoisensor: 80
          nearestbedsensor: 80
          villagerbabiessensor: 40
          playersensor: 40
          nearestlivingentitysensor: 40
      behavior:
        villager:
          validatenearbypoi: 60
          acquirepoi: 120
```

Dit bepaalt hoe vaak gespecificeerde gedragingen en sensoren worden afgevuurd in tikken. `acquirepoi` voor dorpelingen schijnt het zwaarste gedrag te zijn, dus is het sterk verhoogd. Verlaag het in het geval van problemen met dorpelingen die hun weg niet kunnen vinden.

### [airplane.yml]

#### max-loads-per-projectile

`Goede beginwaarde: 8`

Specificeert het maximum aantal chunks dat een projectiel kan laden tijdens zijn levensduur. Verminderen zal de chunkbelasting door entiteitprojectielen verminderen, maar kan problemen veroorzaken met tridents, enderpearls, enz.

#### max-tick-freq

`Goede beginwaarde: 20`

Deze optie bepaalt hoe langzaam entiteiten die het verst van spelers af staan getickt zullen worden. Het verhogen van deze waarde kan de prestaties verbeteren van entiteiten die ver uit het zicht staan, maar kan farms kapot maken of het gedrag van mobs sterk verminderen.

#### activation-dist-mod

`Goede startwaarde: 7`

Regelt de gradiënt waarin mobs worden aangevinkt. DAB werkt op een gradiënt in plaats van een harde cutoff zoals EAR. In plaats van entiteiten dichtbij volledig aan te vinken en entiteiten veraf nauwelijks, zal DAB de hoeveelheid van een entiteit die wordt aangevinkt verminderen gebaseerd op het resultaat van deze berekening. Dit verlagen zal DAB dichter bij de spelers activeren, wat de prestatiewinst van DAB verbetert, maar zal invloed hebben op hoe entiteiten met hun omgeving omgaan en kan mob farms kapot maken.

### [purpur.yml]

#### dont-send-useless-entity-packets

`Goede startwaarde: true`

Door deze optie in te schakelen bespaar je bandbreedte door te voorkomen dat de server lege positiewijzigingspakketten verstuurt (standaard verstuurt de server dit pakket voor elke entiteit, zelfs als de entiteit niet is verplaatst). Dit kan problemen veroorzaken met plugins die client-side entiteiten gebruiken.

#### aggressive-towards-villager-when-lagging

`Goede startwaarde: false`

Als je dit inschakelt zullen zombies stoppen met het aanvallen van dorpelingen als de server onder de tps drempel zit die is ingesteld met `lagging-threshold` in [purpur.yml].

#### entities-can-use-portals

`Goede startwaarde: false`

Deze optie kan het gebruik van portalen uitschakelen voor alle entiteiten behalve de speler. Dit voorkomt dat entiteiten chunks laden door van wereld te veranderen, wat wordt afgehandeld op de hoofddraad. Dit heeft als neveneffect dat entiteiten niet door portalen kunnen gaan.

#### villager.brain-ticks

`Goede startwaarde: 2`

Met deze optie kun je instellen hoe vaak (in tikken) de hersenen van een dorpeling (werk en poi) zullen tikken. Hoger dan `3` is bevestigd dat dorpelingen inconsistent/buggy worden.

#### villager.lobotomize

`Goede startwaarde: true`

Lobotomize dorpelingen worden ontdaan van hun AI en zullen alleen om de zoveel tijd hun aanbod aanvullen. Door dit in te schakelen zullen dorpelingen die niet in staat zijn om hun bestemming te vinden, gelobotomiseerd worden. Hen bevrijden zou hen weer moeten lobotomiseren.

---

## Misc

### [spigot.yml]

#### merge-radius

```
Goede start waarden:

      item: 3.5
      exp: 4.0
```

Dit bepaalt de afstand tussen de items en exp orbs die worden samengevoegd, waardoor er minder items op de grond tikken. Als je dit te hoog instelt zal dit leiden tot de illusie dat items of exp orbs verdwijnen als ze samengevoegd worden. Als je dit te hoog instelt, zullen sommige boerderijen kapot gaan, en zullen items door blokken kunnen teleporteren. Er worden geen controles uitgevoerd om te voorkomen dat items door muren heen samensmelten. Exp wordt alleen samengevoegd bij het aanmaken.

#### hopper-transfer

`Goede startwaarde: 8`

Tijd in tikken dat hoppers zullen wachten om een item te verplaatsen. Het verhogen van deze tijd zal de performance verbeteren als er veel hoppers op je server zijn, maar zal hopper-gebaseerde klokken en mogelijk item sorteer systemen breken als het te hoog is ingesteld.

#### hopper-check

`Goede startwaarde: 8`

Tijd in tikken tussen hoppers die controleren op een item boven hen of in de inventaris boven hen. Het verhogen van deze tijd zal de performance verbeteren als er veel hoppers op je server zijn, maar zal hopper-gebaseerde klokken en item sorteer systemen die vertrouwen op waterstromen breken.

### [paper.yml]

#### alt-item-despawn-rate

```
Goede start waarden:

      enabled: true
      items:
          COBBLESTONE: 300
```

Met deze lijst kun je een alternatieve tijd instellen (in ticks) om bepaalde soorten gedropte items sneller of langzamer te laten verdwijnen dan standaard. Deze optie kan gebruikt worden in plaats van item clearing plugins samen met `merge-radius` om de performance te verbeteren.

#### use-faster-eigencraft-redstone

`Goede beginwaarde: waar`

Wanneer dit is ingeschakeld, wordt het redstone systeem vervangen door een snellere en alternatieve versie die overbodige blokupdates vermindert, waardoor je server minder werk hoeft te doen. Door dit in te schakelen kun je de prestaties aanzienlijk verbeteren zonder inconsistenties in de gameplay te introduceren. Het inschakelen hiervan zal zelfs sommige redstone inconsistenties van craftbukkit oplossen.

#### disable-move-event

`Goede beginwaarde: false`

`InventoryMoveItemEvent` gaat niet af, tenzij er een plugin is die actief luistert naar dat event. Dit betekent dat je dit alleen op true moet zetten als je zulke plugin(s) hebt en het niet erg vindt dat ze niet kunnen reageren op dit event. **Zet dit niet op true als je plugins wilt gebruiken die naar deze gebeurtenis luisteren, b.v. beschermingsplugins!**

#### mob-spawner-tick-rate

`Goede startwaarde: 2`

Deze optie laat je configureren hoe vaak spawners moeten worden aangevinkt. Hogere waarden betekenen minder vertraging als je veel spawners hebt, maar als je deze te hoog instelt (in verhouding tot de vertraging van je spawners) zal de snelheid waarmee mob spawnt afnemen.

#### optimize-explosions

`Goede start waarde: true`

Als je dit op `true` zet wordt het vanilla explosie algoritme vervangen door een sneller algoritme, ten koste van een kleine onnauwkeurigheid bij het berekenen van de explosie schade. Dit is meestal niet merkbaar.

#### enable-treasure-maps

`Goede beginwaarde: false`

Het genereren van schatkaarten is erg duur en kan een server laten hangen als het gebouw dat het probeert te lokaliseren buiten je voorgegenereerde wereld ligt. Het is alleen veilig om dit aan te zetten als je je wereld hebt voorgegenereerd en een vanille wereldgrens hebt ingesteld.

#### treasure-maps-return-already-discovered

`Goede startwaarde: true`

De standaardwaarde van deze optie dwingt de nieuw gegenereerde kaarten om te zoeken naar onontdekte structuren, die meestal buiten je voorgegenereerde terrein liggen. Door dit op true te zetten, kunnen kaarten leiden naar de structuren die eerder ontdekt zijn. Als je dit niet op `true` zet kan het zijn dat de server blijft hangen of crasht bij het genereren van nieuwe schatkaarten.

#### grass-spread-tick-rate

`Goede startwaarde: 4`

Tijd in tikken tussen het moment dat de server gras of mycelium probeert te verspreiden. Dit zorgt ervoor dat grote stukken vuil er iets langer over doen om in gras of mycelium te veranderen. Dit instellen op ongeveer `4` zou goed moeten werken als je het wilt verminderen zonder dat de verminderde verspreiding merkbaar is.

#### container-update-tick-rate

`Goede startwaarde: 1`

Tijd in ticks tussen container updates. Dit verhogen kan helpen als container updates problemen veroorzaken (dit gebeurt zelden), maar maakt het makkelijker voor spelers om desync te ervaren bij interactie met inventories (spook items).

#### non-player-arrow-despawn-rate

`Goede startwaarde: 20`

Tijd in tikken waarna pijlen afgeschoten door mobs moeten verdwijnen nadat ze iets geraakt hebben. Spelers kunnen deze pijlen toch niet oppakken, dus je kunt dit net zo goed instellen op iets als `20` (1 seconde).

#### creative-arrow-despawn-rate

`Goede startwaarde: 20`

Tijd in tikken waarna pijlen die door spelers in de creatieve modus zijn afgeschoten moeten verdwijnen nadat ze iets hebben geraakt. Spelers kunnen deze pijlen toch niet oppakken, dus je kunt dit net zo goed instellen op iets als `20` (1 seconde).

### [purpur.yml]

#### disable-treasure-searching

`Goede startwaarde: true`

Voorkomt dat dolfijnen structuurzoeken uitvoeren die lijken op schatkaarten

#### teleport-if-outside-border

`Goede startwaarde: true`

Maakt het mogelijk om de speler te teleporteren naar de spawn als hij buiten de wereldgrens is. Handig omdat de vanilla wereldgrens te omzeilen is en de schade die het aan de speler toebrengt kan worden beperkt.

---

## Helpers

### [paper.yml]

#### anti-xray

`Goede startwaarde: true`

Schakel dit in om ertsen te verbergen voor x-rayers. Voor gedetailleerde configuratie van deze functie, zie [Stonar96's aanbevolen instellingen](https://gist.github.com/stonar96/ba18568bd91e5afd590e8038d14e245e). Het inschakelen hiervan zal de prestatie verminderen, maar het is veel efficiënter dan een anti-xray plugin. In de meeste gevallen zal de prestatie-impact te verwaarlozen zijn.

#### remove-corrupt-tile-entities

`Goede startwaarde: true`

Verander dit naar `true` als je console wordt overspoeld met fouten over tile-entiteiten. Dit zal alle tile-entiteiten die de fout veroorzaken verwijderen in plaats van negeren. Als u regelmatig waarschuwingen krijgt over tile entiteiten, onderzoek dan waarom ze breken. Dit is geen oplossing voor het hoofdprobleem.

#### nether-ceiling-void-damage-height

`Goede startwaarde: 127`

Als deze optie groter is dan `0`, zullen spelers boven het ingestelde y-niveau beschadigd worden alsof ze in de leegte staan. Dit zal voorkomen dat spelers het nether dak gebruiken. De vanille nether is 128 blocks hoog, dus je moet hem waarschijnlijk op `127` zetten. Als je de hoogte van de nether op een of andere manier aanpast, moet je dit instellen op `[jouw_nether_hoogte] - 1`.

---

# Java opstartvlaggen
[Vanilla Minecraft en Minecraft server software in versie 1.17 vereist Java 16 of hoger](https://papermc.io/forums/t/java-16-mc-1-17-and-paper/5615). Oracle heeft hun licenties veranderd, en er is niet langer een dwingende reden om je java van hen te krijgen. Aanbevolen verkopers zijn [Amazon Corretto](https://aws.amazon.com/corretto/) en [Adoptium](https://adoptium.net/). Alternatieve JVM implementaties zoals OpenJ9 of GraalVM kunnen werken, maar ze worden niet ondersteund door papier en staan erom bekend dat ze problemen veroorzaken, daarom worden ze momenteel niet aanbevolen.

Je vuilnisman kan geconfigureerd worden om vertragingspieken veroorzaakt door grote vuilnisman taken te verminderen. Je kunt opstartvlaggen vinden die geoptimaliseerd zijn voor Minecraft servers [hier](https://mcflags.emc.gs/) [`SOG`]. Houd in gedachten dat deze aanbeveling niet zal werken op alternatieve jvm implementaties.

# Linux CPU schaling 
Sommige hosts kunnen machines leveren die in "PowerSave" mode draaien. Dit kan resulteren in bijna 25% lagere kloksnelheden en dus veel lagere single threaded prestaties. Dit kan leiden tot veel slechtere prestaties dan wanneer de CPU-schaling op prestatiemodus wordt gezet. Merk op dat dit mogelijk niet beschikbaar is voor VPS.

Voor Debian / Ubuntu

`cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor` Toont het prestatieprofiel van de CPU.

`echo "performance" | sudo tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor` Stelt het profiel in op performance.

# "Te mooi om waar te zijn" plugins

## Plugins die gronditems verwijderen
Absoluut onnodig omdat ze vervangen kunnen worden door [merge-radius](#merge-radius) en [alt-item-despawn-rate](#alt-item-despawn-rate) en eerlijk gezegd zijn ze minder configureerbaar dan basis server configs. Ze hebben de neiging om meer middelen te gebruiken voor het scannen en verwijderen van items dan voor het helemaal niet verwijderen van de items.

## Mob stacker plugins
Het is echt moeilijk om het gebruik ervan te rechtvaardigen. Het stapelen van natuurlijk gespawnde entiteiten veroorzaakt meer vertraging dan ze helemaal niet te stapelen doordat de server constant probeert meer mobs te spawnen. De enige "aanvaardbare" use case is voor spawners op servers met een groot aantal spawners.

## Plugins die andere plugins in- of uitschakelen
Alles dat plugins in- of uitschakelt tijdens runtime is extreem gevaarlijk. Het laden van zo'n plugin kan fatale fouten veroorzaken met tracking data en het uitschakelen van een plugin kan leiden tot fouten door het verwijderen van dependency. Het `/reload` commando lijdt aan exact dezelfde problemen en je kunt er meer over lezen in [me4502's blog post](https://madelinemiller.dev/blog/problem-with-reload/)

# Wat is er aan het vertragen? - het meten van prestaties

## mspt
Paper biedt een `/mspt` commando dat je vertelt hoeveel tijd de server nodig had om recente ticks te berekenen. Als de eerste en tweede waarde die je ziet lager zijn dan 50, dan gefeliciteerd! Uw server is niet traag! Als de derde waarde boven de 50 is, betekent dit dat er minstens 1 tick langer over deed. Dat is volkomen normaal en gebeurt van tijd tot tijd, dus geen paniek.

## timings
Een goede manier om te zien wat er aan de hand kan zijn als je server lagging vertoont zijn timings. Timing is een tool waarmee je precies kunt zien welke taken het langst duren. Het is de meest elementaire tool om problemen op te lossen en als je om hulp vraagt met betrekking tot lag zul je waarschijnlijk om je timings worden gevraagd.

Om de timings van je server te krijgen hoef je alleen maar het `/timings paste` commando uit te voeren en op de link te klikken die je krijgt. Je kunt deze link delen met andere mensen zodat zij je kunnen helpen. Het is ook gemakkelijk om verkeerd te lezen als je niet weet wat je doet. Er is een gedetailleerde [video tutorial door Aikar](https://www.youtube.com/watch?v=T4J0A9l7bfQ) over hoe je ze moet lezen.
  
## spark
[Spark](https://github.com/lucko/spark) is een plugin die je toelaat om je servers CPU en geheugengebruik te profileren. Je kunt lezen hoe het te gebruiken [op zijn wiki](https://spark.lucko.me/docs/). Er is ook een gids over hoe je de oorzaak van lag spikes kunt vinden [hier](https://spark.lucko.me/docs/guides/Finding-lag-spikes).


[`SOG`]: https://www.spigotmc.org/threads/guide-server-optimization%E2%9A%A1.283181/
[server.properties]: https://minecraft.fandom.com/Server.properties
[bukkit.yml]: https://bukkit.gamepedia.com/Bukkit.yml
[spigot.yml]: https://www.spigotmc.org/wiki/spigot-configuration/
[paper.yml]:  https://paper.readthedocs.io/en/latest/server/configuration.html
[purpur.yml]: https://purpur.pl3x.net/docs
[airplane.yml]: https://github.com/TECHNOVE/Airplane/wiki
