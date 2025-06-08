# References en de garbage collector

---

[←](2-frametimes-profiler.md) • [Intro](1-intro.md) • [Frametimes](2-frametimes-profiler.md) • **References, GC** • [→]()

---

We beginnen bij een herhaling van de basis: bij het programmeren moeten we kunnen bijhouden waar we onze data opslaan. Dit doen we door variabelen mooie namen te geven waarmee we ze altijd kunnen terugvinden. Maar voor de computer werkt dit heel anders. Deze houdt een verwijzing bij naar waar het stukje data opgeslagen staat in het geheugen. Dit heet een memory address, en wordt vaak opgeslagen in een pointer of een reference.

Elk programma op de computer krijgt een stukje geheugen toegewezen als die daar om vraagt. De programmeur kan zeggen “ik heb 10 megabyte aan geheugen nodig” en dan gaat de computer kijken waar hij 10 megabyte heeft, en geeft jou een adres terug van een plekje in het geheugen dat jij mag hebben. Maar je krijgt daarmee ook de verantwoordelijkheid om dit geheugen weer vrij te maken. Een programmeertaal als C# is ontworpen zodat jij hier niet over na hoeft te denken. Het enige wat jij als programmeur moet doen is zeggen wat je gaat opslaan, en de rest bepaalt het systeem: hoe groot jouw data is, het geheugen dat moet worden opgevraagd, en zelfs het opruimen daarvan. Dit scheelt de programmeur veel werk, maar net als bij de meeste dingen die de programmeur werk schelen zitten er ook nadelen aan, en in dit geval lever je een klein stukje performance in. In veel gevallen niet noemenswaardig, maar in bepaalde situaties die je tegenkomt bij het maken van games kan dit enorm optellen.

## Reference types en value types
Het verschil tussen een reference en een value type is een van de belangrijkste dingen die je moet weten over werken met geheugen in C#. Een value type bewaart de volledige inhoud van de instance in de variabele. Een reference type bewaart slechts een memory address naar de plek waar de inhoud van de variabele in het geheugen staat. Een value type is dus vergelijkbaar met een pointer in C/C++.

Om te weten of iets een reference of een value type is, moet je vooral kijken naar of het een class of een struct is. We kunnen bijvoorbeeld een data type maken voor een vertex met een class of een struct:
```csharp
public class VertexRefType
{
    float x, y, z;
}

public struct VertexValueType
{
    float x, y, z;
}
```
Als je een lokale variabele opslaat, komt deze terecht op de zogenaamde ***stack***. Je kunt dit zien als het lokale geheugen van een functie of een statement. De stack staat altijd op een vaste plek, en alles wat op de stack staat, staat ook naast elkaar in het fysieke geheugen. Als je een variabele van een value type aanmaakt, wordt alle inhoud ervan op de stack geplaatst. Maar als je een reference type aanmaakt, wordt de inhoud op een willekeurige andere plek in het geheugen opgeslagen, op de zogenaamde ***heap***, en wordt slechts de pointer naar die plek op de stack opgeslagen.

```csharp
VertexRefType myVert = new VertexRefType(2, 3, 4);


------------- stack -------------        ------------- heap ------------
 ## | ## | ## | 0x003b | ## | ##          ## | 2 | 3 | 4 | ## | ## | ## 
---------------- ^ --------------        ----- ^ -----------------------
                 myVert                        0x003b
```

```csharp
VertexValueType myVert = new VertexValueType(2, 3, 4);

------------- stack -----------
 ## | ## | ## | 2 | 3 | 4 | ## 
---------------- ^ ------------
                 myVert
```

Een ander belangrijk verschil tussen reference en value types is hoe de data gekopieerd wordt. Bij een reference type maak je alleen een kopie van het memory adres. Bij een value type kopieer je de volledige inhoud. Dit maakt uit als je de variabelen daarna gaat aanpassen. Dit is wat er gebeurt bij een reference type:

```csharp
VertexRefType myVert = new VertexRefType(2, 3, 4);

var copyVert = myVert;

--------------------------------        ------------------------------
 ## | ## | 0x003b | 0x003b | ##         ## | 2 | 3 | 4 | ## | ## | ##
---------- ^ ----------- ^ -----        ---- ^ -----------------------
      myVert      copyVert                   0x003b
```
```csharp
// Pas een waarde van copyVert aan
copyVert.y = 5;

// De waarde veranderd voor zowel myVert als copyVert
--------------------------------        ------------------------------
 ## | ## | 0x003b | 0x003b | ##         ## | 2 | 5 | 4 | ## | ## | ##
---------- ^ ----------- ^ -----        ---- ^ -----------------------
      myVert      copyVert                   0x003b 

```
En hier dezelfde code met een value type:
```csharp
VertexRefType myVert = new VertexRefType(2, 3, 4);

var copyVert = myVert;

--------------------------------------
 ## | ## | 2 | 3 | 4 | 2 | 3 | 4 | ## 
---------- ^ --------- ^ -------------
      myVert           copyVert
```
```csharp
// Pas een waarde van copyVert aan
copyVert.y = 5;

// De waarde veranderd alleen voor copyVert
--------------------------------------
 ## | ## | 2 | 3 | 4 | 2 | 5 | 4 | ## 
---------- ^ --------- ^ -------------
      myVert           copyVert

```

De volgende onderdelen gaan over waarom het verschil tussen value en reference types uit kan maken voor performance.

### Meer informatie
De C# language reference gaat een stuk dieper in op de werking van value en reference types, en alle bijzondere gevallen en uitzonderingen.

- [C# language reference - Value types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/builtin-types/value-types) 
- [C# language reference - Reference types](https://learn.microsoft.com/en-us/dotnet/csharp/language-reference/keywords/reference-types)

## De garbage collector
De garbage collector ruimt jouw ongebruikte variabelen op de heap op, reference types dus. De garbage collector is heel fijn: Je hoeft je niet druk te maken over het vrijmaken van geheugen dat je opgevraagd had, geen gedoe met pointers en memory leaks en ownership. Dit is een kernfeature van C# (of specifieker van *.NET Core* of Unity's *Mono*), dus het is niet iets wat je kunt uitzetten.

Elke keer dat je een nieuwe *reference type* aanmaakt, wordt er geheugen op de *heap* vrijgemaakt om de data op te slaan. De garbage collector moet dit geheugen later weer opruimen en vrijgeven. Dit is waarom garbage collection tijd kost, en uiteindelijk toch een compromis is.

De garbage collector doet dit in de achtergrond. Je hebt er zo geen last van tijdens gebruik van het programma. C# en .NET waren oorspronkelijk vooral ontwikkeld als een framework voor desktop-applicaties op Windows. Deze programma's zijn vaak een stuk minder zwaar, en hebben meer dan genoeg ruimte om in de achtergrond af en toe wat geheugen op te ruimen.

### Games
Games daarentegen zijn een heel ander verhaal. Als de garbage collector in een game aan de slag moet, neemt dit hoe dan ook een hapje uit je frametime. Meestal is dit een vrij klein hapje, maar als je niet uitkijkt met heap allocations, kan het ook een vrij groot hapje worden. Dit zul je merken als periodieke uitschieters in frametime, ofwel: **frame spikes**. (Frame spikes als gevolg van de garbage collector worden ook vaak **GC spikes** genoemd.)

Om deze reden heeft Unity haar eigen garbage collector die een stuk beter geoptimaliseerd is voor games dan de oorspronkelijke. Het lost een deel van de problemen op, maar nog lang niet alles. Een garbage collection zal vaak niet meer dan 5ms kosten, maar kan in erge gevallen tot tientallen milliseconden duren, waardoor het dus makkelijk een framebudget van 16.7ms (60FPS) overschrijdt. De snelheid is direct afhankelijk van de hoeveelheid op te ruimen geheugen en de snelheid van de computer. 

Tegenwoordig (sinds Unity 2019.4) is er zelfs een optie om een iets complexere garbage collector te gebruiken die het werk verspreidt over meerdere frames (zie afbeelding). Maar toch moet het werk hoe dan ook gedaan worden, en dit gaat altijd ten koste van jouw frame time. Het is dus een goede zaak om ervoor zorgen dat de garbage collector *altijd* zo min mogelijk werk hoeft te doen. Dat betekent: we willen het aantal allocations minimaliseren!

![](Fig_UnityIncrementalGC.png)

### In de praktijk

Een aantal voorbeelden van allocations die de garbage collector later moet opruimen:

```csharp
// Het aanmaken van een nieuwe instantie van een eigen class (maar niet van een struct!)
CustomClass newCustomObject = new CustomClass();
StateMachine myStateMachine = new StateMachine();
myStateMachine.SwitchState(new GamePlayingState());

// Alle verzamelingen zijn vrijwel per definitie reference types die op de heap aangemaakt worden, ook als de waarden in de verzameling zelf géén reference type zijn.
List<int> newList = new List<int> { 4, 5 };
GameObject[] newGameObjectArray = new GameObject[3];
Dictionary<int, string> newDictionary = new Dictionary<int, string>();

// In Unity: GameObjects aanmaken via 'new' of 'Instantiate()' gebruikt ruimte op de heap, evenals het toevoegen van components
GameObject newGameObject = new GameObject();
PlayerController playerController = Instatiate(PlayerControllerPrefab);
ParticleSystem particles = newGameObject.AddComponent<ParticleSystem>();

// Strings maken met dynamische waarden (waarden die onbekend zijn voor de game start)
string newString = "Time: " + Time.time;

// In Unity: Coroutines starten
StartCoroutine(ExampleCoroutine());

// Boxing: een fenomeen waarin een value type geforceerd wordt om een reference type te worden, omdat de functie of een reference type verwacht
private void RefTypeFunc(object obj) { /* ... */ } // << object is per definitie een reference type
RefTypeFunc(18) // << 18 is een int (een value type) en kan door het ontwerp van de taal alleen doorgegeven worden via de heap.
object unknownObj = 18 // << 18 wordt ook hier "geboxed", omdat "object" een reference type moet worden.
```

Let op dat `new` in C# niet per se een heap allocation doet, in tegenstelling tot C++. Het hangt puur af van het soort object wat aangemaakt wordt:

```csharp
struct Vertex { float x, y, z; }
class Player { float age; }

Vertex vertex = new Vertex { x = 5, y = 5, z = 8 }; // << Vertex is een struct, dus ondanks 'new' wordt er geen heap garbage gemaakt.
Player player = new Player { age = 22 }; // << Player is een class, dus er wordt wel heap garbage gemaakt.
```

### Meer informatie
- [Garbage collector overview - Unity Manual](https://docs.unity3d.com/Manual/performance-garbage-collector.html)

## Samenvatting

- In C# wordt er onderscheid gemaakt tussen twee soorten datatypen: **value types** die op **stack** bestaan, en **reference types** die op de **heap** bestaan.  
  - Value types slaan al hun data op in zichzelf
  - Reference types slaan slechts een referentie op naar een plek in het geheugen waar de daadwerkelijke data staat (de heap).
- Geheugen dat gebruikt wordt moet ook altijd opgeruimd worden. 
  - Stack-geheugen gebruikt door value types kan automatisch opgeruimd worden door programma-logica.
  - Heap-geheugen gebruikt door reference types kan alleen worden opgeruimd door de **garbage collector** (**GC**).
- Als de garbage collector aan de slag moet, kost dit altijd tijd van je frame. In ernstige gevallen kan dit voor merkbare haperingen in de game zorgen.
- We willen allocations vermijden om de garbage collector te ontlasten.