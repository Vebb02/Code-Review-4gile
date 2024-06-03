# CodeReview av 4-gile sin PlaneMane

## Bugs
- Det er ingenting som stopper spilleren fra å fly ut av skjermen på venstre siden.
- Spawning av enemies følger ikke samme mønster som koden egentlig antyder.

## Kontroller

### GameController
I keyDown kunne de forskjellige casene i switchen vært egne metoder, for å korte ned på metoden og gjøre det mer oversiktlig.
Direction kunne veldig gjerne vært sin egen klasse slik at man blant annet slipper å skrive lange boolske uttrykk flere ganger og abstraherer det til metoder og inkapsulerer logikken et annet sted enn i kontrolleren.

## Modell

### Mappestruktur entities
entities-mappen kunne gjerne hatt undermapper for hovedgruppene som utvider entities. Feks: spiller-relatert, enemies og pickupables.

### Entity
World behøver ikke være med i både EntityGameModel og som parameter i konstruktøren til Entity, i tillegg er den kun brukt i en metode i klassen som kun er brukt i konstruktøren, som vil si at det er heller ingen grunn til å beholde den som en variabel og kunne heller vært et parameter i metoden som bruker den.
Entity har også generelt sett for mange parametre i konstruktøren. Ved å bruke Rectangle-klassen, for x, y, width og height, og få tak i world fra EntityGameModel kan man enkelt korte det ned til (Rectangle dimensions, EntityGameModel model, String texRegionString)
entitySprite og getEntitySprite er/vil alltid returnere null. Dette virker til å være kode som har vært del av en tidligere implementasjon eller aldri blitt implementert. Dette kan fjernes.
Generelt har entity-ene sin implementasjon av update litt vel mange magiske tall. Altså feks det gjengående temaet av "(model.getGameTime() / 1000000)". Tall bør ha navn sånn at noen kan lese koden i etterkant og forstå hva tallene representerer. Det kunne også kanskje vært en tanke om å inkludere tid/deltaTid som et parameter i update.

### Factories
Begge fabrikkene er hardkodet, som de eksplisitt i oppgaveteksten ikke skulle være. For å endre på det som produserer må de fabrikkene endres. Litt i samme gate som dette så er også begge fabrikkene sin kode veldig like (basically copy/paste) og inneholder derfor repetisjon av kode. Det er greit å ha forskjellige fabrikk-klasser for forskjellige formål, men her må de lages på en mer utvidbar måte og de kunne hatt en generisk abstrakt klasse som de arvet fra og endret på forskjellene i implementasjon.
spawnInterval kunne vært en del av konstruktøren i stedet for å få en verdi satt i konstruktøren og ha en setter-metode.
Fabrikkene burde ikke ha GameModel, men heller grensesnittet EntityGameModel for å unngå at de har tilgang til for mye.

### Configurations
WALLTHICKNESS -> WALL_THICKNESS.
FACTORY_HEIGHT -> ENTITY_HEIGHT_OPTIONS (eller lignende).

### GameModel
For mange magiske tall i konstruktøren som burde vært definert i f.eks. Configurations.
updateEntities tar inn et delta parameter som ikke brukes. Som nevnt tidligere burde dette kanskje vært med videre til hver enkelt entity i stedet for at alle gjør den samme uforklarlige utregningen. 
restartGame har enda flere magiske tall.
SpawnEnemies burde vært refaktorert ved å ekstrakte en metode for å spawne en ting i stedet for å skrive den samme koden to ganger med forskjellige timere. Dessuten så spawner de to delene av metoden bare en tilfeldig enemy uansett, fordi begge kaller createObject som gir en tilfeldig enemy og ikke en spesifikk enemy, som resten av koden tilsier at det skal være.
spawnObjects går også i samme kategori som tilbakemeldingen over og kunne brukt samme hjelpemetode, i stedet for å skrive nøyaktig samme kode for tredje gang.
I stedet for å ha en egen liste entities som skal fjernes, der de legger seg selv til, kunne det kanskje vært en bedre implementasjon å ha en metode isAlive i entities der hver entity sjekkes om de er levende, og hvis de er dø fjernes de.
getGameTime er redundant hvis man endrer implementasjonen i entities til å bruke delta.

### PlaneContactListener
Bruk av instanceof er en generell "no no".
I beginContact skrives omtrent det samme tre ganger på rad. Her hadde det kanskje vært mer fornuftig å først sjekke om minst en av de er et fly. Altså "if (!isPlane(f1, f2)) return;". Deretter ha et mer generelt grensesnitt for oppførselen når man treffer spilleren. Hvert kall av isPlane er redundant etter første kallet, det vil ikke endre verdi etter første kall.

### TaskManager
I run brukes det magiske char som også brukes i fabrikkene. Disse kunne vært definert i Configurations.

## View

### GameView
Render-metoden er alt for lang. Her bør det gjøres refaktorering ved å gjøre de forskjellige delene om til egne metoder med beskrivende navn.

## Main

### PlaneMane
Det er en if i render-metoden som kunne vært i create-metoden uten if, den kun skal gjøres en gang, som er første kallet av render.

## Ressurser
Bildene i graphics-mappen kunne gjerne hvert gruppert i undermapper.

## Tester
Som tidligere nevnt om kontrolleren burde Direction være en egen klasse og dette vises også i testene der det er mange unødvendige linjer som ikke hadde vært der hvis Direction var en egen klasse som ikke var avhengig av input fra controller-klassen.
I EntityTest ville man nok lagt merke til at GetEntitySprite alltid er null hvis det hadde vært sjekket nøyere i stedet for å sjekke at getter gir tilbake forventet variabel som den er knyttet til.

Ellers ser testene generelt sett greie ut. Ser at det er mye sjekking direkte på variabler i klasser, som kanskje egentlig burde vært private for å sikre at ingen utenfra kan endre på variablene utenfra. Det er også mye sjekking at getter og variabelen er de samme, disse testene skal det mye til å at de skal feile og tester ikke at det funker som det skal.


# Oppsummering
- Ikke bruk magiske verdier. Gi navn til verdier som ikke ikke er selvforklarende.
- Ikke skriv for lange metoder. GameModel:update er et eksempel på hvordan metoder skal være og GameView:render er et eksempel på hvordan det ikke skal være.
- Bruk generaliserende hjelpemetoder eller klasser i stedet for å skrive det samme flere ganger.
- Det er litt mange tendenser til at felt-variabler er tilgjenlig i pakken og ikke private, uten at det er nødvendigvis har en veldig tydelig hensikt. En tommelfinger regel er at felt-variabler er private med mindre de eksplisitt skal være tilgjenglig og ikke minst muterbare utenfra.
