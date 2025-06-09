# Afsluiting

---

[←](5-layout.md) • [Intro](1-intro.md) • [Frametimes](2-frametimes-profiler.md) • [References, GC](3-references-gc.md) • [Geheugen hergebruiken](4-chunk-pooling.md) • [Geheugen-layout](5-layout.md) • **Afsluiting**

---

We zijn bij het einde van deze masterclass aangekomen. Je hebt nu hopelijk een beter idee over hoe je projecten moet ontwerpen die slimmer en efficiënter gebruik maken van het geheugen, de onzichtbare bottleneck in veel games. Nog een laatste tip:

> **Tip**\
> Hoewel veel programmeurs zullen zeggen dat het niet handig is om voortijdig te optimaliseren, is er wel degelijk iets voor te zeggen om code te schrijven die rekening houdt met slim geheugenmanagement. De technieken die we gezien hebben maken je code vaak niet complexer of moeilijker te onderhouden, en kosten weinig tot geen extra tijd om te implementeren vanaf het begin. Zelfs voor prototypes kan het interessant zijn.\
> De grootste factor waar de benodigdheid van geheugenoptimalisatie van afhangt, is vooral de frequentie waarmee er beroep op het geheugen gedaan wordt. Ofwel: bij veel objecten loont het meer om te optimaliseren dan bij weinig.

## Samenvatting

Je hebt geleerd wat frametimes zijn, hoe je die moet gebruiken om een performance-doel op te stellen voor je game, en hoe je kunt testen of je dit doel bereikt hebt met de profiler.

We hebben gekeken naar de verschillende data-typen in C# en hoe deze zich gedragen in het geheugen. Je hebt hier geleerd dat tijdelijke heap allocations opgeruimd moeten worden door de garbage collector, en hoe deze per ongeluk de performance van je game kan beïnvloeden. 

We hebben een aantal technieken bekeken om het aantal tijdelijke heap allocations te verminderen of zelfs te elimineren, door lijsten en dictionaries te hergebruiken, of door pools te gebruiken.

Tot slot hebben we gekeken hoe we reference types kunnen vervangen door value types, en de value types in een array kunnen zetten om de data sneller en efficiënter toegankelijk te maken voor de processor.

## Verdere resources

- ### *What Every Programmer Should Know About Memory* - door Ulrich Drepper (2007)
    Deze paper van 114 pagina's gaat zo ver de diepte in als maar kan. De titel is vind ik persoonlijk een beetje overdreven, maar de informatie is erg waardevol voor game-programmeurs die werken aan optimalisatie. Gaat vooral in op de technische details de werking van RAM, CPU cache gedrag, NUMA (Non Uniform Memory Access) systemen, en de consequenties die dit heeft op het ontwerp van snelle code. Dateert uit 2007 maar de technologie werkt grotendeels hetzelfde.\
    Mirror link: https://people.freebsd.org/~lstewart/articles/cpumemory.pdf
- ### *Practical Optimizations* - Jason Booth (2023)
    Gaat over dezelfde onderwerpen als deze masterclass, en geeft een paar praktische voorbeelden van toepassingen die Jason Booth tegen is gekomen in zijn 20-jarige carrière als graphics engineer.\
    YouTube link: https://www.youtube.com/watch?v=NAVbI1HIzCE \
    Bijbehorende Medium-post: https://medium.com/@jasonbooth_86226/intro-to-jobs-burst-dod-66c6b81c017f
- ### Catlike Coding (Jasper Flick)
    Uitgebreide tutorials door Jasper Flick die ingaan op allerlei technische aspecten van Unity (en tegenwoordig ook Godot). In zijn Basics tutorial implementeert hij Jobs en compute shaders, en zijn Object Management tutorial gaat in op het bouwen van een geavanceerd pooling-systeem.\
    Links: [catlikecoding.com](https://catlikecoding.com/) • [Unity Basics](https://catlikecoding.com/unity/tutorials/basics/) • [Unity Object Management](https://catlikecoding.com/unity/tutorials/object-management/)