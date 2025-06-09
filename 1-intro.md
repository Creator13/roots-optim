# Masterclass: Data-oriented memory-optimalisaties voor een procedurele terreingenerator

### Casper van Battum • Graduation HKU Game Development

---

**Intro** • [Frametimes](2-frametimes-profiler.md) • [References, GC](3-references-gc.md) • [Geheugen hergebruiken](4-chunk-pooling.md) • [Geheugen-layout](5-layout.md) • [Afsluiting](9-afsluiting.md) • [→](2-frametimes-profiler.md)

---

Voor mijn afstudeerproject – Roots – heb ik een procedurele terrein-generator geschreven. Een uitdaging die ik hiervoor opgezocht heb was het optimaliseren van de generator: terrein moest tijdens het spelen van de game zonder haperingen geladen worden. 

Unity maakt het makkelijk om snel games te maken, maar daar staat tegenover dat je zelf wat extra werk moet doen om ook _snelle_ games te maken. Met een beetje kennis van hoe programmeertalen en de computer werken valt er een hoop winst te behalen. Het doel van deze masterclass is om een goed begin te leggen voor die kennis.

Door alle optimalisaties heen stond er een ding centraal: hoe ga je om met het geheugen? Feit is dat hier veel winst te behalen valt op moderne hardware. Daarom gaan we voornamelijk kijken naar hoe goed (of slecht) geheugengebruik mij heeft geholpen om een oppervlakte van 24km² verdeeld over 15 chunks in slechts 2ms te genereren. En was dat proces het ook daadwerkelijk waard?
![Fig_Terrain.png](Fig_Terrain.png)
## Wat ga je leren?
Je gaat in deze masterclass leren hoe het bijhouden en bewaren van data de performance van je game beïnvloedt. Dit doen we door eerst naar de basis te kijken van memory-management in C# en Unity, en vervolgens twee concrete voorbeelden uit mijn project te bekijken waar ik deze concepten heb toegepast om performance-bottlenecks op te lossen. Voor deze case studies heb ik een minimaal voorbeeldproject om zelf de stof toe te passen, maar de masterclass is vooral bedoeld als uitgangspunt om zelf verder te leren over de concepten van geheugenmanagement. 

Ik gebruik twee voorbeelden uit mijn project om verschillende onderwerpen uit te lichten:

- Memory hergebruiken
- Code ontwerpen voor memory/cache locality

[//]: # (- Unity native memory &#40;inclusief een kort stukje over GPU buffers&#41;)

Deze masterclass gaat uit van **Unity 6.1**. Hoewel ik Unity gebruik als context voor de code in deze masterclass, zijn de principes toepasbaar op alle engines. Memory werkt uiteindelijk hetzelfde op elke computer, voor elk programma. Zelfs non-game software kan hier winst behalen. De principes komen uit data-oriented design, een tegenhanger van het welbekende object-oriented design, dat steeds meer gebruikt wordt in games.

## Voorkennis

- Je kunt met C# programmeren, of een vergelijkbare programmeertaal.
- Basiskennis van pointers/references zoals in C/C++, Rust, of een vergelijkbare programmeertaal.
- Je weet wat de game loop van een engine is en wat het concept van een frame is.
- Je kan je weg vinden in Unity.
  - Inclusief het gebruik van de profiler (zo niet staan er resources in het volgende hoofdstuk).

## Inhoud

1. **Intro**
2. [Performance-doelen & profiler](2-frametimes-profiler.md)
3. [C# data types & garbage collector](3-references-gc.md)
4. [Geheugen hergebruiken: terrain chunks laden](4-chunk-pooling.md)
5. [Geheugen-layout: vegetatie updaten](5-layout.md)
6. [Afsluiting en verder lezen](9-afsluiting.md)

---

Volgende deel: [Geheugen hergebruiken →](2-frametimes-profiler.md)