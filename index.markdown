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
INX       ;Inkrementera (d.v.s. öka) värdet i X-registret
ADC #$c4  ;Addera hexvärde c4 till A-registret
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
3. Motsatsen till `ADC` är `SBC` (subtrahera med carry (d.v.s. lån)). Skriv ett program som
   använder denna instruktion.

<h2 id='branching'>Villkorliga hopp</h2>

Hittills har vi bara kunnat skriva grundläggande program utan villkorliga hopp.
Låt oss ändra på det.

6502 assembler har en massa hoppinstruktioner, som alla
hoppar baserat på om vissa flaggor är satta eller inte. I det här exemplet kommer vi att
titta på `BNE`: "Hoppa om inte lika" (en. "Branch on Not Equal").

{% include start.html %}
  LDX #$08
decrement:
  DEX
  STX $0200
  CPX #$03
  BNE decrement
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

Två av stackinstruktionerna är `PHA` och `PLA`, "push ackumulator" och "pull
ackumulator". Nedan är ett exempel på dessa två i aktion.

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

Jumping is like branching with two main differences. First, jumps are not
conditionally executed, and second, they take a two-byte absolute address. For
small programs, this second detail isn't very important, as you'll mostly be
using labels, and the assembler works out the correct memory location from the
label. For larger programs though, jumping is the only way to move from one
section of the code to another.

Långa hopp är som villkorliga hopp med två huvudsakliga skillnader. För det första, långa hopp utförs inte 
villkorligt, för det andra, de tar en två-byte absolut adress. För 
små program, är denna andra detalj inte särskilt viktig, eftersom du oftast
använder etiketter och assembleraren räknar ut rätt minnesadress med hjälp av 
etiketten. För större program är dock långa hopp det enda sättet att flytta programräknaren från en 
del av koden till en annan.

###JMP###

`JMP` is an unconditional jump. Here's a really simple example to show it in action:

{% include start.html %}
  LDA #$03
  JMP there
  BRK
  BRK
  BRK
there:
  STA $0200
{% include end.html %}

`JMP` är ett ovillkorligt långt hopp. Här är ett mycket enkelt exempel för att visa det i aktion:

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

`JSR` and `RTS` ("jump to subroutine" and "return from subroutine") are a
dynamic duo that you'll usually see used together. `JSR` is used to jump from
the current location to another part of the code. `RTS` returns to the previous
position. This is basically like calling a function and returning.

The processor knows where to return to because `JSR` pushes the address minus
one of the next instruction onto the stack before jumping to the given
location. `RTS` pops this location, adds one to it, and jumps to that location.
An example:

{% include start.html %}
  JSR init
  JSR loop
  JSR end

init:
  LDX #$00
  RTS

loop:
  INX
  CPX #$05
  BNE loop
  RTS

end:
  BRK
{% include end.html %}

The first instruction causes execution to jump to the `init` label. This sets
`X`, then returns to the next instruction, `JSR loop`. This jumps to the `loop`
label, which increments `X` until it is equal to `$05`. After that we return to
the next instruction, `JSR end`, which jumps to the end of the file. This
illustrates how `JSR` and `RTS` can be used together to create modular code.


<h2 id='snake'>Creating a game</h2>

Now, let's put all this knowledge to good use, and make a game! We're going to
be making a really simple version of the classic game 'Snake'.

The simulator widget below contains the entire source code of the game. I'll
explain how it works in the following sections.

[Willem van der Jagt](https://twitter.com/wkjagt) made a [fully annotated gist
of this source code](https://gist.github.com/wkjagt/9043907), so follow along
with that for more details.

{% include snake.html %}


###Overall structure###

After the initial block of comments (lines starting with semicolons), the first
two lines are:

    jsr init
    jsr loop

`init` and `loop` are both subroutines. `init` initializes the game state, and
`loop` is the main game loop.

The `loop` subroutine itself just calls a number of subroutines sequentially,
before looping back on itself:

    loop:
      jsr readkeys
      jsr checkCollision
      jsr updateSnake
      jsr drawApple
      jsr drawSnake
      jsr spinwheels
      jmp loop

First, `readkeys` checks to see if one of the direction keys (W, A, S, D) was
pressed, and if so, sets the direction of the snake accordingly. Then,
`checkCollision` checks to see if the snake collided with itself or the apple.
`updateSnake` updates the internal representation of the snake, based on its
direction. Next, the apple and snake are drawn. Finally, `spinWheels` makes the
processor do some busy work, to stop the game from running too quickly. Think
of it like a sleep command. The game keeps running until the snake collides
with the wall or itself.


###Zero page usage###

The zero page of memory is used to store a number of game state variables, as
noted in the comment block at the top of the game. Everything in `$00`, `$01`
and `$10` upwards is a pair of bytes representing a two-byte memory location
that will be looked up using indirect addressing.  These memory locations will
all be between `$0200` and `$05ff` - the section of memory corresponding to the
simulator display. For example, if `$00` and `$01` contained the values `$01`
and `$02`, they would be referring to the second pixel of the display (`$0201`
- remember, the least significant byte comes first in indirect addressing).

The first two bytes hold the location of the apple. This is updated every time
the snake eats the apple. Byte `$02` contains the current direction. `1` means
up, `2` right, `4` down, and `8` left.  The reasoning behind these numbers will
become clear later.

Finally, byte `$03` contains the current length of the snake, in terms of bytes
in memory (so a length of 4 means 2 pixels).


###Initialization###

The `init` subroutine defers to two subroutines, `initSnake` and
`generateApplePosition`. `initSnake` sets the snake direction, length, and then
loads the initial memory locations of the snake head and body. The byte pair at
`$10` contains the screen location of the head, the pair at `$12` contains the
location of the single body segment, and `$14` contains the location of the
tail (the tail is the last segment of the body and is drawn in black to keep
the snake moving). This happens in the following code:

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

This loads the value `$11` into the memory location `$10`, the value `$10` into
`$12`, and `$0f` into `$14`. It then loads the value `$04` into `$11`, `$13`
and `$15`. This leads to memory like this:

    0010: 11 04 10 04 0f 04

which represents the indirectly-addressed memory locations `$0411`, `$0410` and
`$04ff` (three pixels in the middle of the display). I'm labouring this point,
but it's important to fully grok how indirect addressing works.

The next subroutine, `generateApplePosition`, sets the apple location to a
random position on the display. First, it loads a random byte into the
accumulator (`$fe` is a random number generator in this simulator). This is
stored into `$00`. Next, a different random byte is loaded into the
accumulator, which is then `AND`-ed with the value `$03`. This part requires a
bit of a detour.

The hex value `$03` is represented in binary as `00000111`. The `AND` opcode
performs a bitwise AND of the argument with the accumulator. For example, if
the accumulator contains the binary value `01010101`, then the result of `AND`
with `00000111` will be `00000101`.

The effect of this is to mask out the least significant three bytes of the
accumulator, setting the others to zero. This converts a number in the range of
0&ndash;255 to a number in the range of 0&ndash;3.

After this, the value `2` is added to the accumulator, to create a final random
number in the range 2&ndash;5.

The result of this subroutine is to load a random byte into `$00`, and a random
number between 2 and 5 into `$01`. Because the least significant byte comes
first with indirect addressing, this translates into a memory address between
`$0200` and `$05ff`: the exact range used to draw the display.


###The game loop###

Nearly all games have at their heart a game loop. All game loops have the same
basic form: accept user input, update the game state, and render the game
state. This loop is no different.


####Reading the input####

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


####Updating the game state####

The next subroutine, `checkCollision`, defers to `checkAppleCollision` and
`checkSnakeCollision`. `checkAppleCollision` just checks to see if the two
bytes holding the location of the apple match the two bytes holding the
location of the head. If they do, the length is increased and a new apple
position is generated.

`checkSnakeCollision` loops through the snake's body segments, checking each
byte pair against the head pair. If there is a match, then game over.

After collision detection, we update the snake's location. This is done at a
high level like so: First, move each byte pair of the body up one position in
memory. Second, update the head according to the current direction. Finally, if
the head is out of bounds, handle it as a collision. I'll illustrate this with
some ascii art. Each pair of brackets contains an x,y coordinate rather than a
pair of bytes for simplicity.

      0    1    2    3    4
    Head                 Tail

    [1,5][1,4][1,3][1,2][2,2]    Starting position

    [1,5][1,4][1,3][1,2][1,2]    Value of (3) is copied into (4)

    [1,5][1,4][1,3][1,2][1,2]    Value of (2) is copied into (3)

    [1,5][1,4][1,3][1,2][1,2]    Value of (1) is copied into (2)

    [1,5][1,4][1,3][1,2][1,2]    Value of (0) is copied into (1)

    [0,4][1,4][1,3][1,2][1,2]    Value of (0) is updated based on direction

At a low level, this subroutine is slightly more complex. First, the length is
loaded into the `X` register, which is then decremented. The snippet below
shows the starting memory for the snake.

    Memory location: $10 $11 $12 $13 $14 $15

    Value:           $11 $04 $10 $04 $0f $04

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

Going between sections is more complicated, as we have to take into account the
most significant byte as well. For example, going down from `$02e1` should lead
to `$0301`. Luckily, this is fairly easy to accomplish. Adding `$20` to `$e1`
results in `$01` and sets the carry bit. If the carry bit was set, we know we
also need to increment the most significant byte.

After a move in each direction, we also need to check to see if the head
would become out of bounds. This is handled differently for each direction. For
left and right, we can check to see if the head has effectively "wrapped
around". Going right from `$021f` by incrementing the least significant byte
would lead to `$0220`, but this is actually jumping from the last pixel of the
first row to the first pixel of the second row. So, every time we move right,
we need to check if the new least significant byte is a multiple of `$20`. This
is done using a bit check against the mask `$1f`. Hopefully the illustration
below will show you how masking out the lowest 5 bits reveals whether a number
is a multiple of `$20` or not.

    $20: 0010 0000
    $40: 0100 0000
    $60: 0110 0000

    $1f: 0001 1111

I won't explain in depth how each of the directions work, but the above
explanation should give you enough to work it out with a bit of study.


####Rendering the game####

Because the game state is stored in terms of pixel locations, rendering the
game is very straightforward. The first subroutine, `drawApple`, is extremely
simple. It sets `Y` to zero, loads a random colour into the accumulator, then
stores this value into `($00),y`. `$00` is where the location of the apple is
stored, so `($00),y` dereferences to this memory location. Read the "Indirect
indexed" section in [Addressing modes](#addressing) for more details.

Next comes `drawSnake`. This is pretty simple too. `X` is set to zero and `A`
to one. We then store `A` at `($10,x)`. `$10` stores the two-byte location of
the head, so this draws a white pixel at the current head position. Next we
load `$03` into `X`. `$03` holds the length of the snake, so `($10,x)` in this
case will be the location of the tail. Because `A` is zero now, this draws a
black pixel over the tail. As only the head and the tail of the snake move,
this is enough to keep the snake moving.

The last subroutine, `spinWheels`, is just there because the game would run too
fast otherwise. All `spinWheels` does is count `X` down from zero until it hits
zero again. The first `dex` wraps, making `X` `#$ff`.
