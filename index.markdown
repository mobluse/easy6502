---
layout: default
---

<h2 id="intro">Introduktion</h2>

I denna lilla bok jag ska visa dig hur du kommer igång att skriva 6502-assembler. 6502-processorn var massivt populär på sjuttio- och åttiotalet, och drev kända datorer som [BBC Micro](http://sv.wikipedia.org/wiki/BBC_Micro), [Atari 2600](http://sv.wikipedia.org/wiki/Atari_2600), [Commodore 64](http://sv.wikipedia.org/wiki/Commodore_64), [Apple II](http://sv.wikipedia.org/wiki/Apple_II), och [Nintendo Entertainment System (NES)](http://sv.wikipedia.org/wiki/Nintendo_Entertainment_System). Bender i Futurama [har en 6502-processor som hjärna](http://www.transbyte.org/SID/SID-files/Bender_6502.jpg). [Även Terminator programmerades i 6502](http://www.pagetable.com/docs/terminator/00-37-23.jpg).

Så, varför skulle du vilja lära dig 6502? Det är ett dött språk, eller hur? Tja, det är latin också. Och de lär fortfarande ut det. [Q.E.D.](http://sv.wikipedia.org/wiki/Quod_erat_demonstrandum)

(Faktiskt, jag har fått tillförlitlig information om att 6502-processorer fortfarande produceras av [Western Design Center (WDC)](http://65xx.com/Developer_Products/65xx-chips/), så uppenbarligen är 6502 *inte* ett dött språk! Vem kunde tro det?)

Allvarligt talat, jag tycker det är värdefullt att ha en förståelse för assembler. Assembler är den lägsta nivån av abstraktion i datorer - den punkt där koden är fortfarande läsbar. Assembler översätter direkt till byte som sedan utförs av datorns processor. Om du förstår hur det fungerar, har du i princip blivit en [datortrollkarl](http://skilldrick.co.uk/2011/04/magic-in-software-development/).

Varför 6502? Varför inte ett *användbart* assemblerspråk, som [x86](http://sv.wikipedia.org/wiki/X86)? Tja, jag tror inte att lära x86 är användbart. Jag tror inte att du någonsin kommer att behöva skriva assembler i ditt vanliga jobb - det är enbart en akademisk övning, något att utöka ditt sinne och ditt tänkande med. 6502 skrevs ursprungligen i en annan tid, en tid då de flesta av utvecklarna skrev assembler direkt, i stället för i dessa nymodiga högnivåprogrammeringsspråk. Så de var konstruerade för att skrivas av människor. Mer moderna assemblerspråk är tänkta att skrivas av kompilatorer, så låt oss lämna det till dem. Dessutom är 6502 *kul*. Ingen har någonsin kallat x86 *kul*.


<h2 id="first-program">Vårt första program</h2>

Låt oss dyka i! Den saken här nedan är en liten [JavaScript 6502-assemblator och 
-simulator/emulator](https://github.com/skilldrick/6502js) som jag anpassade för denna bok. 
Klicka **Assemble** och sedan **Run** att assemblera och köra snutten med assemblerkod. 

{% include start.html %}
LDA #$01
STA $0200
LDA #$05
STA $0201
LDA #$08
STA $0202
{% include end.html %}

Förhoppningsvis har det svarta området till höger nu tre färgade "pixlar" 
uppe till vänster. (Om detta inte fungerar, behöver du förmodligen uppgradera din webbläsare till
en mer modern, som Chrome eller Firefox.) 

Alltså, vad gör det här programmet egentligen? Låt oss gå igenom den med 
debuggern. Klicka **Reset**, kryssa sedan i **Debugger** kryssrutan för att starta 
debuggern. Klicka **Step** en gång. Om du tittade noga, bör du ha 
märkt att `A=` ändras från `$00` till `$01` och `PC=` ändras från `$0600` till 
`$0602`.

Alla tal med prefixet `$` i 6502-assembler (och, i förlängningen, i
denna bok) är i hexadecimalt (hex) format. Om du inte är bekant med hex-tal, rekommenderar jag att du läser [Wikipedia
artikeln](http://sv.wikipedia.org/wiki/Hexadecimala_talsystemet). Allt som har `#` som prefix
är verkliga, konkreta tal (`#` är i detta sammanhang den amerikanska mostvarigheten till förkortningen, *no*, d.v.s. numero). Alla andra tal (d.v.s. utan `#`) hänvisar till en minnesplats (d.v.s. en adress).

Utrustad med den kunskapen, bör du kunna inse att instruktionen
`LDA #$01` laddar hextalet `$01` i register `A` (LoaD A med talet `$01`). Jag ska gå in mer
i detalj på register i nästa avsnitt.

Klicka **Step** igen för att exekvera (d.v.s. utföra) den andra instruktionen. Den övre vänstra pixeln i
simulatorns display bör nu vara vit. Denna simulator använder minnesadresserna `$0200` till `$05ff` för att rita pixlar på sin skärm. Värdena `$00` till
`$0f` representerar 16 olika färger (`$00` är svart och `$01` är vit). Att lagra värdet `$01` på adress `$0200` ritar en vit pixel i
övre vänstra hörnet. Detta är enklare än hur en verklig dator skulle lagra data i grafikminnet, men det får duga tills vidare.

Alltså, instruktionen `STA $0200` lagrar värdet för `A`-registret i minnesaddressen
`$0200` (STore A i adressen `$0200`). Klicka **Step** ytterligare fyra gånger för att utföra resten av
instruktionerna, och håll ett öga på `A`-registret, ty det ändrar sig.

###Övningar###

1. Försök att ändra färgen på de tre pixlarna. 
2. Ändra en av bildpunkterna (d.v.s. pixlarna) så att den ritas i det nedre högra hörnet (adress `$05ff`).
3. Lägg till fler instruktioner för att rita in extra bildpunkter på olika positioner med olika färger.

<h2 id='registers'>Register och flaggor</h2>

Vi har redan tittat lite på processorstatusavdelningen (avdelningen med
`A`, `PC` m.fl.), men vad betyder allt detta?

Den första raden visar `A`-, `X`- och `Y`-registren (`A` kallas ofta
"ackumulatorn"). Varje register innehåller en enda byte. De flesta operationerna arbetar på
innehållet i dessa register.

'SP' är stackpekaren. Jag kommer inte gå igenom stacken ännu, men i grund och botten minskas detta
registrer varje gång en byte läggs på stacken, och
ökas när en byte plockas upp från stacken.

`PC` är programräknaren - det är så processorn vet vid vilken punkt i
programmet den är just nu. Det är som den aktuella raden i ett körande
skript. I JavaScript-simulatorn assembleras koden med början på minnesadress `$0600`, så `PC` börjar alltid där.

Den sista delen visar processorflaggor. Varje flagga är en bit, så alla sju
flaggor lever i en enda byte. Flaggorna ställs in av processorn för att ge
information om den föregående instruktionen. Mer om det senare. [Läs mer
om register och flaggor här](http://www.obelisk.demon.co.uk/6502/registers.html).

<h2 id='instructions'>Instruktioner</h2>

Instruktionerna i assembler är som en liten uppsättning fördefinierade funktioner.
Alla instruktioner tar noll eller ett argument. Här är några kommenterade
källkoder för att introducera några olika instruktioner:

{% include start.html %}
LDA #$c0  ;Ladda hex värde c0 in i A-registret
TAX       ;Transferera (d.v.s. överför) värdet i A till X
INX       ;INkrementera (d.v.s. öka) värdet i X-registret
ADC #$c4  ;ADdera med Carry hexvärde c4 till A-registret
BRK       ;Break - vi är klara
{% include end.html %}

Assemblera koden, slå sedan på debuggern och stega igenom koden, titta
då på `A`- och `X`-registret. Något lite underligt händer på raden `ADC #$c4`.
Du förväntade dig kanske att lägga till `$c4` till `$c0` skulle ge `$184`, men denna
processor ger resultatet som `$84`. Vad pågår här?

Problemet är, `$184` är för stort för att passa i en enda byte (max är `$FF`),
och registren kan endast lagra en enda byte. Det är dock OK; processorn
är faktiskt inte dum. Om du tittade tillräckligt noga, märkte du
att carry-flaggan (d.v.s. minnessiffran) sattes till `1` efter denna operation. Så det är så du
vet.

I simulatorn nedan **skriv in** (inte klistra in) följande kod:

    LDA #$80
    STA $01
    ADC $01

{% include widget.html %}

En viktig sak att notera här är skillnaden mellan `ADC #$01` och
`ADC $01`. Den första adderar värdet `$01` till `A`-registret, men den
andra adderar värdet lagrat på minnesplats `$01` till `A`-registret.

Assemblera, kryssa i **Monitor**-kryssrutan, stega sedan igenom dessa tre
instruktioner. Monitorn visar en del av minnet, och kan vara till hjälp för att
visualisera exekveringen av programmen. `STA $01` lagrar värdet på `A`-registret
på minnesplats `$01` och `ADC $01` adderar värdet lagrat i
minnesplats `$01` till `A`-registret. `$80 + $80` skall motsvara `$100`, men
eftersom det är större än en byte, sätts `A`-registret till `$00` och
carry-flaggan sätts. Förutom detta sätts dock zero-flaggan (d.v.s. nollflaggan). Zero-flaggan
sätts av alla instruktioner där resultatet är noll.

En fullständig lista över 6502 instruktionsuppsättning är [tillgängliga
här](http://www.6502.org/tutorials/6502opcodes.html) och
[här](http://www.obelisk.demon.co.uk/6502/reference.html) (jag brukar hänvisa till
bägge sidorna enär de har sina styrkor och svagheter). Dessa sidor anger
argument till varje instruktion, vilka register de använder, och vilka flaggor som de
sätter. De är din bibel.

###Övningar###

1. Du har sett `TAX`  Du kan nog gissa vad `TAY`, `TXA` och `TYA` gör,
   men skriv lite kod för att testa dina antaganden.
2. Skriv om det första exemplet i detta avsnitt för att använda `Y`-registret i stället för
   `X`-registeret.
3. Motsatsen till `ADC` är `SBC` (SuBtrahera med Carry (d.v.s. lån)). Skriv ett program som
   använder denna instruktion.

<h2 id='branching'>Villkorliga hopp</h2>

Hittills har vi bara kunnat skriva grundläggande program utan villkorliga hopp.
Låt oss ändra på det.

6502 assembler har en massa hoppinstruktioner, som alla
hoppar baserat på om vissa flaggor är satta eller inte. I det här exemplet kommer vi att
titta på `BNE`: "Hoppa om inte lika" (en. "Branch on Not Equal").

{% include start.html %}
  LDX #$08
minska:
  DEX        ;DEkrementera (d.v.s. minska) värdet i X-registret
  STX $0200
  CPX #$03
  BNE minska
  STX $0201
  BRK
{% include end.html %}

Först laddar vi värdet `$08` i `X`-registret. Nästa rad är en etikett.
Etiketter markerar bara vissa punkter i ett program så att vi kan gå tillbaka till dem senare.
Efter etiketten minskar (dekrementerar) vi `X` med 1, lagrar det i `$0200` (övre, vänstra pixeln), och
jämför det sedan med värdet `$03`.
[`CPX`](http://www.obelisk.demon.co.uk/6502/reference.html#CPX) jämför
värde i `X`-registret med ett annat värde. Om de två värdena är lika,
sätts `Z`-flaggan till `1`, annars sätts den till `0`.

Nästa rad, `BNE decrement`, flyttar exekveringen till decrement-etiketten om
den så kallade `Z`-flaggan är satt till `0` (vilket betyder att de två värdena i `CPX`-jämförelsen
inte var lika), annars gör den ingenting och vi lagrar `X` i `$0201`, och sen
avslutar vi programmet.

I assembler brukar man använda etiketter tillsammans med hoppinstruktioner. När
en sådan assemblerats så omvandlas etiketten till en en-byte relativ förflyttning (ett
antal byte att hoppa bakåt eller framåt från nästa instruktion) så
hoppinstruktioner kan bara gå framåt och tillbaka runt 256 byte. Detta betyder att
de endast kan användas för att förflytta sig omkring i lokal kod. För att förflytta längre måste du
använda långhoppinstruktionerna.

###Övningar###

1. Motsatsen till `BNE` är `BEQ`. Försök att skriva ett program som använder `BEQ`. 
2. `BCC` och `BCS` ("hoppa om minnessiffran är nollställd" (en. "branch on carry clear") och "hoppa om minnessiffran är satt" (en. "branch on carry set")) används 
   för att hoppa baserat på carry-flaggan. Skriv ett program som använder en av dessa två.

<h2 id='addressing'>Adresseringssätt</h2>

6502 använder en 16-bitars adressbuss, vilket innebär att det finns 65536 bytes 
minne tillgängligt för processorn. Kom ihåg att en byte representeras av två 
hex-tecken, så minnesadresserna anges i allmänhet som `$0000 - $ffff`. Det finns 
olika sätt att hänvisa till dessa minnesadresser, vilket beskrivs i detalj nedan. 

Tillsammans med alla dessa exempel kan du tycka att det underlättar att använda minnesmonitorn för
att se när minnet ändras. Monitorn tar en utgångsadress och antalet byte som 
visas från den adressen. Båda dessa är hex-värden. 
Till exempel, för att visa 16 byte minne från `$c000`, ange `c000` och `10` 
i **Start** respektive **Length**.

###Absolut: `$c000`###

Med absolut adressering, används den fullständiga minnesadressen som argument till instruktionen. Till exempel:

    STA $c000 ;Lagra värdet i ackumulatorn på minnesadress $c000

###Noll-sida: `$c0`###

Alla instruktioner som stödjer absolut adressering (med undantag för långhoppinstruktionerna) 
har också möjlighet att ta en en-byte adress. Denna typ av
adressering kallas "noll-sida" - bara den första sidan (de första 256 byte) av
minnet är tillgängligt. Detta är snabbare, eftersom endast en byte behöver slås upp,
och tar även upp mindre plats i den assemblerade koden (d.v.s. maskinkoden).

###Noll-sida,X: `$c0,X`###

Det är här som adressering blir intressant. I detta läge är en nollsidesadress given, och sedan läggs värdet för `X`-registret till. Här är ett exempel:  

    LDX #$01   ;X är $01
    LDA #$aa   ;A är $aa
    STA $a0,X  ;Lagra värdet av A i minnesadress $a1
    INX        ;Öka X med 1
    STA $a0,X  ;Lagra värdet av A i minnesadress $a2

Om resultatet av additionen är större än en enda byte, så går adressen runt. Till exempel:  

    LDX #$05
    STA $ff,X  ;Lagra värdet av A i minnesadress $04

###Noll-sida,Y: `$c0,Y`###

Detta motsvarar noll-sida,X, men kan bara användas med `LDX` och `STX`.

###Absolut,X och absolut,Y: `$c000,X` och `$c000,Y`###

Dessa är absolutadresseringsversionerna av noll-sida,X och noll-sida,Y. Till exempel: 

    LDX #$01 
    STA $0200,X ;Lagra värdet av A på minnesadress $0201

###Omedelbar: `#$c0`###

Omedelbar adressering handlar inte direkt om minnesadresser - detta är det 
adresseringssätt där faktiska värden används. Till exempel, `LDX #$01` laddar värdet 
`$01` i `X`-registret. Det är väldigt annorlunda jämfört med noll-sideinstruktionen 
`LDX $01` som läser in värdet från minnesadress `$01` i `X`-registret.

###Relativ: `$c0` (eller etikett)###

Relativ adressering används för hoppinstruktioner. Dessa instruktioner tar
en enda byte, som används som en förskjutning från den följande instruktionen.

Assemblera följande kod, klicka sedan på **Hexdump**-knappen för att se den assemblerade koden.

{% include start.html %}
  LDA #$01
  CMP #$02
  BNE ejlika
  STA $22
ejlika:
  BRK
{% include end.html %}

Hexkoden bör se ut ungefär så här:  

    a9 01 c9 02 d0 02 85 22 00  

`a9` och `c9` är processor-opkoder för omedelbart adresserade `LDA` respektive `CMP`. 
`01` och `02` är argumenten till dessa instruktioner. `d0` är
opkoden för `BNE`, och dess argument är `02`. Det innebär att "hoppa över nästa
två byte" (`85 22`, den assemblerade versionen av `STA $22`). Försök att redigera koden
så att `STA` tar en två-byte absolut adress i stället för en enda byte noll-sideadress 
(t.ex. ändra `STA $22` till `STA $2222`). Assemblera om koden och titta på
hexdumpen igen (klicka på **Hexdump**) - argumentet till `BNE` ska nu vara `03`, eftersom
instruktionen som processorn skall hoppa över nu är tre byte lång.

###Implicit###

Vissa instruktioner hanterar inte minnesadresser (t.ex. `INX` - öka (d.v.s. inkrementera)
`X`-registret). Dessa sägs ha implicit (eller underförstådd) adressering - argumentet 
är inbyggt i instruktionen.

###Indirekt: `($c000)`###

Indirekt adressering använder en absolut adress för att slå upp en annan adress. Den 
första adressen ger den minst signifikanta byten i adressen, och 
den följande byten ger den mest signifikanta byten. Detta kan vara svårt att förstå, 
så här är ett exempel: 

{% include start.html %}
LDA #$01
STA $f0
LDA #$cc
STA $f1
JMP ($00f0) ;avrefereras till $cc01
{% include end.html %}

I detta exempel innehåller `$f0` värdet `$01` och `$f1` innehåller värdet
`$cc`. Instruktionen `JMP ($00f0)` får processorn att slå upp de två
byten på `$f0` och `$f1` (`$01` och `$cc`) och sätta ihop dem för att bilda
adressen `$cc01`, som blir den nya programräknaren. Assemblera och stega
igenom programmet ovan för att se vad som händer. Jag ska prata mer om `JMP` i
avsnittet om [Långa hopp](#jumping).

###Indexerad indirekt: `($c0,X)`###

Den här är ganska konstig. Det är som en korsning mellan noll-sida,X och indirekt. 
I grund och botten tar du noll-sidans adress, lägger till värdet av `X`-registret, 
använder sedan summan för att slå upp en två-byte adress. Till exempel:

{% include start.html %}
LDX #$01
LDA #$05
STA $01
LDA #$06
STA $02
LDY #$0a
STY $0605
LDA ($00,X)
{% include end.html %} 

Minnesadresserna `$01` och `$02` innehåller värdena `$05` respektive `$06`. 
Tänk på `($00,X)` som `($00+X)`. I detta fall är `X` lika med `$01`, så 
detta förenklas till `($01)`. Härifrån fortgår det som vid normal indirekt 
adressering - de två byten vid `$01` och `$02` (`$05` och `$06`) slås upp 
för att bilda adressen `$0605`. Detta är den adress som `Y`-registret lagrades
i i den tidigare instruktionen, så `A`-registret får samma 
värde som `Y`, om än genom en omfattande omväg. Du kommer inte att se denna 
ofta.

###Indirekt indexerad: `($c0),Y`###

Indirekt indexerad är som indexerad indirekt men mindre galen. I stället för att addera
`X`-registret till adressen *innan* avreferering, så avrefereras noll-sidans adress, 
och `Y`-registret adderas till den resulterande adressen.

{% include start.html %}
LDY #$01
LDA #$03
STA $01
LDA #$07 
STA $02
LDX #$0a 
STX $0704
LDA ($01),Y
{% include end.html %}

I detta fall, `($01)` slår upp de två byten i `$01` och `$02`: `$03` och 
`$07`. Dessa utgör adressen `$0703`. Värdet på `Y`-registret läggs 
till den här adressen för att ge den slutliga adressen `$0704`.

###Övning###

1. Försök att skriva kodsnuttar som använder sig av var och en av 6502-adresseringssätt. 
   Kom ihåg att du kan använda monitorn för att titta på en del av minnet.

<h2 id='stack'>Stacken</h2>

Stacken i en 6502-processor är precis som alla andra stackar (d.v.s. travar/högar) - värden läggs
("pushas") på den och lyfts ("poppas", eller "pullas" i 6502-språkbruk) av den. Det aktuella djupet för
stacken mäts av stackpekaren, ett särskilt register. Stacken bor i
minnet mellan `$0100` och `$01ff`. Stackpekaren är initialt `$ff`, vilket
pekar på minnesadress `$01ff`. När en byte skjuts på stacken, så blir
stackpekaren `$fe`, eller minnesadress `$01fe`, och så vidare.

Två av stackinstruktionerna är `PHA` och `PLA`, "PusH Ackumulator" och "PulL
Ackumulator". Nedan är ett exempel på dessa två i aktion.

{% include start.html %}
  LDX #$00
  LDY #$00
foerstaloopen:
  TXA
  STA $0200,Y
  PHA
  INX
  INY
  CPY #$10
  BNE foerstaloopen ;loopa tills Y är $10
andraloopen:
  PLA
  STA $0200,Y
  INY
  CPY #$20 ;loopa tills Y är $20
  BNE andraloopen
{% include end.html %}

`X` lagrar pixelfärgen, och `Y` lagrar positionen för den aktuella pixeln.
Den första loopen ritar den aktuella färgen som en pixel (via `A`-registret),
pushar färgen till stacken, ökar sedan färgen och positionen med ett. Den
andra loopen poppar stacken, ritar den poppade färgen som en bildpunkt, och ökar
sedan positionen. Som man kan förvänta skapar detta ett speglat mönster.

<h2 id='jumping'>Långa hopp och subrutiner</h2>

Långa hopp är som villkorliga hopp men med två huvudsakliga skillnader. För det första, långa hopp utförs inte 
villkorligt, för det andra, de tar en två-byte absolut adress. För 
små program, är denna andra detalj inte särskilt viktig, eftersom du oftast
använder etiketter och assembleraren räknar ut rätt minnesadress med hjälp av 
etiketten. För större program är dock långa hopp det enda sättet att flytta körningen från en 
del av koden till en annan.

###JMP###

`JMP` är ett ovillkorligt långt hopp (en. JuMP). Här är ett mycket enkelt exempel för att visa det i aktion:

{% include start.html %} 
   LDA #$03 
   JMP dit
   BRK 
   BRK 
   BRK 
dit: 
   STA $0200 
{% include end.html %}

###JSR/RTS###

`JSR` och `RTS` ("hopp (en. Jump) till SubRutin" och "ReTur från Subrutin/underprogram") är en
dynamisk duo som man brukar se tillsammans. `JSR` används för att hoppa från
den aktuella adressen till en annan del av koden. `RTS` återgår till föregående
position. Detta är i princip som att anropa en funktion och återvända/returnera.

Processorn vet vart den skall återvända eftersom `JSR` lägger/pushar adressen minus
ett för nästa instruktion på stacken innan den hoppar till den givna
adressen. `RTS` lyfter/poppar av denna adress, adderar ett till den, och hoppar till den adressen.
Ett exempel:

{% include start.html %}
  JSR init
  JSR loopa
  JSR avsluta

init:
  LDX #$00
  RTS

loopa:
  INX
  CPX #$05
  BNE loopa
  RTS

avsluta:
  BRK
{% include end.html %}

Den första instruktionen gör så att körningen hoppar till `init`-etiketten. Detta sätter
`X`, och sedan återvänder vi till nästa instruktion, `JSR loopa`. Denna hoppar till `loopa`-etiketten, 
som ökar `X` med ett tills det är lika med `$05`. Efter detta återvänder vi till
nästa instruktion, `JSR avsluta`, som hoppar till slutet av filen. Detta
illustrerar hur `JSR` och `RTS` kan användas tillsammans för att skapa modulär kod.


<h2 id='snake'>Att skapa ett spel</h2>

Låt oss nu se till att all denna kunskap kommer till nytta, och göra ett spel! Vi ska 
göra en riktigt enkel version av det klassiska spelet "Masken" (en. "Snake").

Simulator-fönstret nedan innehåller hela källkoden till spelet. Jag ska 
förklara hur det fungerar i de följande avsnitten. 

[Willem van der Jagt](https://twitter.com/wkjagt) har gjort en [fullständigt kommenterad gist
av denna källkod](https://gist.github.com/wkjagt/9043907), så följ med 
i den för fler detaljförklaringar.

{% include snake.html %}

###Övergripande struktur###

Efter det första blocket med kommentarer (rader som börjar med semikolon), så är de första
två raderna:

    jsr init
    jsr loop

`init` och `loop` är bägge subrutiner. `init` initierar speltillståndet, och
`loop` är den viktigaste spelslingan.

Själva `loop`-subrutinen anropar bara ett antal subrutiner sekventiellt,
innan den loopar tillbaka till sig själv:

    loop:
      jsr readkeys
      jsr checkCollision
      jsr updateSnake
      jsr drawApple
      jsr drawSnake
      jsr spinwheels
      jmp loop

Först kontrollerar `readkeys` om en av riktningsknapparna (W, A, S, D)
har tryckts ner, och om så är fallet, så ställs riktningen på masken/ormen in i enlighet därmed. Sedan
kontrollerar `checkCollision` om ormen kolliderade med sig själv eller med äpplet.
`updateSnake` uppdaterar den interna representationen av ormen, baserat på dess
riktning. Därefter ritas äpple och orm. Slutligen får `spinWheels`
processorn att göra en del fördröjningsarbete, för att hindra spelet från att köra för fort. Tänk
på det som ett sov-kommando. Spelet fortsätter att köra tills ormen kolliderar
med väggen eller med sig själv.

###Hur nollsidan används###

Minnets nollsida används för att lagra ett antal speltillståndsvariabler, som
noteras i kommentarssektionen överst i spelet. Allt i `$00`, `$01`
och `$10` och uppåt är par av bytes som representerar en två-byte minnesadress
som kommer att slås upp med hjälp av indirekt adressering. Dessa minnesadresser kommer
alla att vara mellan `$0200` och `$05ff` - den del av minnet som motsvarar
simulatorns skärm. Till exempel, om `$00` och `$01` innehöll värdena `$01`
och `$02`, skulle de hänvisa till den andra bildpunkten på skärmen (`$0201`
- kom ihåg, den minst signifikanta byten kommer först vid indirekt adressering).

De första två byten lagrar placeringen av äpplet. Denna uppdateras varje gång
ormen äter äpplet. Byte `$02` innehåller den aktuella riktningen. `1` betyder
upp, `2` höger, `4` ner, och `8` vänster. Resonemanget bakom dessa siffror kommer
att framgå senare.

Slutligen, byte `$03` innehåller den aktuella längden på ormen, i form av antal byte
i minnet vid adressen `$10` (så att en längd på 4 betyder 2 bildpunkter).

###Initialisera###

`init`-subrutinen anropar två subrutiner: `initSnake` och
`generateApplePosition`. `initSnake` ställer in ormens riktning, längd och
laddar därefter de ursprungliga minnesadresserna för ormens huvud och kropp. Byte-paret vid
`$10` innehåller huvudets skärmposition, och paret på `$12` innehåller
positionen av det enda kroppssegmentet, och `$14` innehåller positionen av
svansen (svansen är det sista segmentet av kroppen och ritas i svart för att hålla
ormen i rörelse). Detta händer i följande kod:

    lda #$11
    sta $10
    lda #$10
    sta $12
    lda #$0f
    sta $14
    lda #$04
    sta $11
    sta $13
    sta $15

Detta laddar värdet `$11` i minnesadress `$10`, värdet `$10` i
`$12` och `$0f` i `$14`. Den laddar sedan värdet `$04` in `$11`, `$13`
och `$15`. Detta leder till detta i minnet:

    0010: 11 04 10 04 0f 04

som representerar de indirekt-adresserade minnesadresserna `$0411`, `$0410` och
`$040f` (tre pixlar i mitten av displayen). Jag betonar kanske denna sak överdrivet,
men det är viktigt att till fullo greppa hur indirekt adressering fungerar.

Nästa subrutin, `generateApplePosition` ställer in äpplets läge till en
slumpmässig position på skärmen. Först laddar det en slumpmässig byte till
ackumulatorn (`$fe` är en slumpgenerator i denna simulator). Detta
lagras i `$00`. Därefter laddas en annan slumpmässig byte in i
ackumulatorn, som sedan `AND`-as (d.v.s. bitvis och) med värdet `$03`. Detta avsnitt behöver nu en
liten avstickare.

Hexvärdet `$07` representeras i binär form som `00000111`. `AND` opkoden
genomför en bitvis *och* (en. *and*) av argumentet med ackumulatorn. Till exempel, om
ackumulatorn innehåller det binära talet `01010101`, då blir resultatet av `AND`
med `00000111` blir `00000101`.

Resultatet av `AND #$03` är att maska de två minst signifikanta bitararna i
ackumulatorn, och sätta de andra till noll. Detta konverterar ett tal i intervallet
0&ndash;255 till ett tal i intervallet 0&ndash;3.

Efter detta adderas värdet `2` till ackumulatorn, för att skapa ett slutligt, slumpvist
tal i området 2&ndash;5.

Resultatet av denna subrutin är att ladda en slumpmässig byte i '$00', och ett slumpmässigt
tal mellan 2 och 5 i `$01`. Eftersom den minst signifikanta byten kommer
först med indirekt adressering, leder detta till en minnesadress mellan
`$0200` och `$05ff`: det exakta intervallet som används för att rita på skärmen.

###Spel-loopen###

Nearly all games have at their heart a game loop. All game loops have the same
basic form: accept user input, update the game state, and render the game
state. This loop is no different.

Nästan alla spel har i sitt hjärta en spel-loop. Alla spel-loopar har samma 
grundläggande form: acceptera användarinmatning, uppdatera speltillståndet, och rita 
speltillståndet. Denna loop är inte annorlunda.

####Läs inmatningen####

The first subroutine, `readKeys`, takes the job of accepting user input. The
memory location `$ff` holds the ascii code of the most recent key press in this
simulator. The value is loaded into the accumulator, then compared to `$77`
(the hex code for W), `$64` (D), `$73` (S) and `$61`. If any of these
comparisons are successful, the program branches to the appropriate section.
Each section (`upKey`, `rightKey`, etc.) first checks to see if the current
direction is the opposite of the new direction. This requires another little detour.

As stated before, the four directions are represented internally by the numbers
1, 2, 4 and 8. Each of these numbers is a power of 2, thus they are represented
by a binary number with a single `1`:

    1 => 0001 (up)
    2 => 0010 (right)
    4 => 0100 (down)
    8 => 1000 (left)

The `BIT` opcode is similar to `AND`, but the calculation is only used to set
the zero flag - the actual result is discarded. The zero flag is set only if the
result of AND-ing the accumulator with argument is zero. When we're looking at
powers of two, the zero flag will only be set if the two numbers are not the
same. For example, `0001 AND 0001` is not zero, but `0001 AND 0010` is zero.

So, looking at `upKey`, if the current direction is down (4), the bit test will
be zero. `BNE` means "branch if the zero flag is clear", so in this case we'll
branch to `illegalMove`, which just returns from the subroutine. Otherwise, the
new direction (1 in this case) is stored in the appropriate memory location.

Den första subrutinen, `readKeys`, tar på sig jobbet att acceptera användarinmatningen. 
Minnesadressen `$ff` lagrar ASCII-koden för den senaste knapptryckningen i denna
simulator. Värdet laddas in i ackumulatorn, jämförs sedan med `$77`
(hex-koden för W), `$64` (D), `$73` (S) och `$61`. Om någon av dessa
jämförelser är framgångsrika, hoppar programmet till det lämpliga avsnittet.
Varje avsnitt (`upKey`, `rightKey` o.s.v.) kontrollerar först för att se om den aktuella
riktningen är motsatsen till den nya riktningen. Detta kräver en annan liten avstickare.

Som nämnts tidigare, representeras de fyra riktningarna internt av talen
1, 2, 4 och 8. Vart och ett av dessa tal är en potens av 2, och därmed är de representerade
av ett binärt tal med en enda `1`:

    1 => 0001 (upp)
    2 => 0010 (höger)
    4 => 0100 (ner)
    8 => 1000 (vänster)

`BIT`-opkoden liknar `AND`, men beräkningen används endast för att sätta
nollflaggan - det faktiska resultatet kastas. Nollflaggan sätts endast om
resultatet av att AND-a ackumulatorn med argumentet är noll. När vi tittar på
potenser av två, kommer nollflaggan endast ställas in om de två talen inte är
desamma. Till exempel är `0001 AND 0001` inte noll, men `0001 AND 0010` är noll.

Vi undersöker `upKey`. Om den aktuella riktningen är nedåt (4), kommer bittestet
vara noll. `BNE` betyder "hoppa om nollflaggan är nollställd", så i detta fall vi kommer
hoppa till `illegalMove`, som bara återvänder från subrutinen. Annars
lagras den nya riktningen (1 i detta fall) på den avsedda minnesadressen.

####Uppdatera speltillståndet####

Nästa subrutin, `checkCollision`,  anropar `checkAppleCollision` och 
`checkSnakeCollision`. `checkAppleCollision` bara kontrollerar om de två 
byte som lagrar positionen av äpplet matchar de två byte som lagrar 
positionen av huvudet. Om de gör det, ökas längden och en ny äppleposition genereras. 

`checkSnakeCollision` loopar igenom ormens kroppssegment och kontrollerar varje 
byte-par mot huvudparet. Om det finns en matchning, då är spelet över. 

Efter kollisionsdetektering, uppdaterar vi ormens position. Detta görs på en 
hög nivå som så här: För det första, flytta varje byte-par av kroppen upp en position i 
minnet. För det andra, uppdatera huvuded enligt den aktuella riktningen. Slutligen, om 
huvudet är utanför ramen, hantera det som en kollision. Jag ska illustrera detta med 
lite ASCII-konst. Varje par av hakparenteser innehåller en x,y-koordinat snarare än ett 
byte-par för enkelhets skull.

      0    1    2    3    4
    Huvud               Svans
    
    [1,5][1,4][1,3][1,2][2,2]    Utgångsläge
    
    [1,5][1,4][1,3][1,2][1,2]    Värdet av (3) kopieras in i (4)
    
    [1,5][1,4][1,3][1,3][1,2]    Värdet av (2) kopieras in i (3)
    
    [1,5][1,4][1,4][1,3][1,2]    Värdet av (1) kopieras in i (2) 
    
    [1,5][1,5][1,4][1,3][1,2]    Värdet av (0) kopieras in i (1) 
    
    [0,5][1,5][1,4][1,3][1,2]    Värdet av (0) uppdateras utifrån riktning

På en låg nivå, är denna subrutin något mer komplex. Först laddas längden 
in i `X`-registret, som sedan minskas. Strängen nedan 
visar utgångsminnet för ormen.

    Minnesadress: $10 $11 $12 $13 $14 $15
    
    Värde:        $11 $04 $10 $04 $0f $04

The length is initialized to `4`, so `X` starts off as `3`. `LDA $10,x` loads the
value of `$13` into `A`, then `STA $12,x` stores this value into `$15`. `X` is
decremented, and we loop. Now `X` is `2`, so we load `$12` and store it into
`$14`. This loops while `X` is positive (`BPL` means "branch if positive").

Once the values have been shifted down the snake, we have to work out what to
do with the head. The direction is first loaded into `A`. `LSR` means "logical
shift right", or "shift all the bits one position to the right". The least
significant bit is shifted into the carry flag, so if the accumulator is `1`,
after `LSR` it is `0`, with the carry flag set.

To test whether the direction is `1`, `2`, `4` or `8`, the code continually
shifts right until the carry is set. One `LSR` means "up", two means "right",
and so on.

The next bit updates the head of the snake depending on the direction. This is
probably the most complicated part of the code, and it's all reliant on how
memory locations map to the screen, so let's look at that in more detail.

You can think of the screen as four horizontal strips of 32 &times; 8 pixels.
These strips map to `$0200-$02ff`, `$0300-$03ff`, `$0400-$04ff` and `$0500-$05ff`.
The first rows of pixels are `$0200-$021f`, `$0220-$023f`, `$0240-$025f`, etc.

As long as you're moving within one of these horizontal strips, things are
simple. For example, to move right, just incrememnt the least significant byte
(e.g. `$0200` becomes `$0201`). To go down, add `$20` (e.g. `$0200` becomes
`$0220`). Left and up are the reverse.

Längden initieras till `4`, så `X` börjar som `3`. `LDA $10,x` laddar
värdet i `$13` in i `A`, sedan `STA $12,x` lagrar detta värde i `$15`. `X`
minskas, och vi loopar. Nu är `X` lika med `2` , så vi laddar `$12` och sparar det i
`$14`. Detta loopar så länge `X` är positivt (`BPL` betyder "hoppa om positivt").

När värdena väl har skiftats ormen ned, måste vi räkna ut vad som ska
göras med huvudet. Riktningen är laddas först in i `A`. `LSR` betyder "logisk
skift höger", eller "flytta alla bitar ett steg åt höger". Den minst
signifikanta biten skiftas in i carry-flaggan. Så om ackumulatorn är `1`,
efter `LSR` är den `0`, med carry-flaggan satt (d.v.s. ettställd).

För att testa om riktningen är `1`, `2`, `4` eller `8`, skiftar koden ständigt
höger tills carry är satt. En `LSR` betyder "upp", två betyder "höger" o.s.v.

Nästa bit uppdaterar ormens huvud beroende på riktning. Detta är
förmodligen den mest komplicerade delen av koden, och det beror helt på hur
minnesaddresser motsvarar skärmkoordinater. Så låt oss titta på det mer i detalj.

Du kan tänka på skärmen som fyra horisontella band av 32 &times; 8 pixlar.
Dessa remsor motsvarar `$0200-$02ff`, `$0300-$03ff`, `$0400-$04ff` och `$0500-$05ff`.
De första raderna av bildpunkter är `$0200-$021f`, `$0220-$023f`, `$0240-$025f` o.s.v.

Så länge man rör sig inom ett av dessa horisontella band, är saker
enkla. Till exempel, för att flytta höger, öka bara den minst signifikanta byten
(t.ex. `$0200` blir `$0201`). För att gå ner, lägg till `$20` (t.ex. `$0200` blir
`$0220`). Vänster och uppåt är det omvända.

Att gå mellan remsorna är mer komplicerat, eftersom vi även måste ta hänsyn till den
mest signifikanta byten. Till exempel, att gå ner från `$02e1` bör leda
till `$0301`. Lyckligtvis är det ganska lätt att åstadkomma. Att addera `$20` till `$e1`
resulterar i `$01` och sätter carry-biten. Om carry-biten blev satt, vet vi att vi
också måste öka den mest signifikanta byten.

Efter ett steg i varje riktning, måste vi också kontrollera om huvudet
skulle hamna utanför ramen. Detta hanteras på olika sätt för varje riktning. För
vänster och höger, kan vi kontrollera om huvudet i praktiken har "gått
runt". Att gå höger från `$021f` genom att öka den minst signifikanta byten
skulle leda till `$0220`, men detta innebär faktiskt att hoppa från den sista pixeln i den
första raden till den första pixeln i den andra raden. Så varje gång vi går åt höger,
måste vi kontrollera om den nya minst signifikanta byten är en multipel av `$20`. Detta
görs med hjälp av en bitkontroll mot masken `$1f`. Förhoppningsvis ska illustrationen
nedan visa hur maskering av de lägsta 5 bitarna avslöja om ett tal
är en multipel av `$20` eller inte.

    $20: 0010 0000
    $40: 0100 0000
    $60: 0110 0000
    
    $1f: 0001 1111

Jag kommer inte att förklara ingående hur varje riktning fungerar, men
förklaringen ovan bör ge dig tillräckligt för att kunna reda ut det med lite iakttagelser.

####Rita upp spelet####

Eftersom speltillståndet lagras i termer av pixeladresser, är uppritningen av
spelet mycket enkel. Den första subrutinen, `drawApple`, är extremt
enkel. Den sätter `Y` till noll, laddar en slumpmässig färg i ackumulatorn, sen
lagrar den detta värde i `($00),y`. `$00` är där äpplets adress
lagras, så `($00),y` avrefereras till denna minnesadress. Läs avsnittet "Indirekt
indexerad" i [Adresseringssätt](#addressing) för mer information.

Härnäst kommer `drawSnake`. Denna är också ganska enkel. `X` sätts till noll och `A`
till ett. Vi lagrar sedan `A` i `($10,x)`. `$10` lagrar huvudets två byte-adress,
så det ritar en vit pixel på den aktuella huvudpositionen. Därefter laddar vi
`$03` i `X`. `$03` lagrar längden på ormen, så `($10,x)` kommer i detta
fall att vara läget för svansen. Eftersom `A` är noll nu, ritar detta en
svart pixel över svansen. Eftersom bara huvudet och svansen på ormen flyttas,
är detta tillräckligt för att hålla ormen i rörelse.

Den sista subrutinen, `spinWheels`, är bara där för att spelet skulle gå för
snabbt annars. Allt `spinWheels` gör är att räkna ner `X` från noll tills den blir
noll igen. Den första `dex` slår runt, vilket gör `X` lika med `#$ff`.
