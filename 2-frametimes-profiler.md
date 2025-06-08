# Performance-doelen en de profiler

Om een richtlijn voor jezelf op de stellen voor de performance van je game heb je een maat nodig om te kunnen meten of je je gestelde doel haalt of niet. De meest bekende maat voor het meten van de snelheid van games is FPS, frames per seconde, ook wel framerate genoemd. Een game gebruikt verschillende beelden die erg snel achter elkaar verschijnen om de illusie van beweging te creëren, net als video. Het verschil tussen video en games is dat bij video de beelden al op een eerder moment gemaakt zijn en alleen maar weergegeven hoeven te worden, terwijl bij games de beelden ter plekke door een computer gemaakt worden.

## Frametime
Om te zorgen dat het beeld vloeiend is moet elke nieuwe afbeelding (frame) binnen een bepaalde tijd klaar zijn. Als je 30 frames per seconde wil laten zien (30 FPS), dan heb je hooguit één dertigste van een seconde de tijd om de frame te maken. Dit is (net iets meer dan) 33 milliseconde (`1s / 30 = 0,0333…s ≈ 33ms`). We noemen dit ook de **frametime**. Als het langer duurt om een frame te maken kun je er minder in een seconde maken, en krijg je dus een lagere FPS. Het framebudget is de maximale frametime. 

> **Tip**\
> Voor games wordt framerate vaak uitgedrukt in FPS, maar de standaard-eenheid voor "keer per seconde" (frequentie) is Hz (hertz). Dit zul je vaak zien op monitors of TV's. Een monitor van 120Hz kan 120 beelden per seconde weergeven, ofwel, 120 frames per second.

Game developers rekenen vaak liever met de frametime dan met de framerate van een spel: 
 
- De frametime simpelweg de som van de tijd die het duurt om alle functies uit te voeren en dit is één-op-één te vergelijken met je budget: als je functies samen 5ms duren, en je budget is 33ms, dan past je frame binnen het budget met nog 28ms over. 

- Een framerate kan alleen berekend worden als een gemiddelde over tijd. De framerate is het gemiddelde aantal frames over een tijdsduur van één seconde. Dit zegt weinig tot niets over de tijd die elk individueel frame kost. Om een vloeiende ervaring te maken is het juist belangrijk om te zorgen dat elk individueel frame binnen het budget blijft.

Het kiezen van een goed framebudget hangt van veel vergelijkbare factoren af als het kiezen van hardware. Het soort game is het meest bepalend: een casual- of simulatie-game blijft goed speelbaar met FPS onder de 60 (hierbij hoort een frametime van 16.7ms), terwijl een shooter of een competitief spel vaak pas vanaf 60 FPS speelbaar wordt. 

Opnieuw is de hardware van de speler van belang: monitoren zijn gelimiteerd in het aantal beelden dat ze kunnen weergeven per seconde. De meeste laptop- of kantoormonitoren zullen een framerate aankunnen van 60 of 75 FPS, terwijl hobby-gamemonitoren vaak 120, 144 of zelfs nog hoger aankunnen. 

Je kunt het framebudget ook bijstellen om voor meer of minder krachtige hardware te corrigeren. Stel je hebt een supercomputer waar je op test, maar je verwacht dat je gebruikers gemiddeld slechtere computers gaan hebben, dan is het slim om te zeggen dat het op jouw computer veel sneller moet draaien dan dat je wil op de computer van je gebruiker. Pas hiermee wel op, want hardware schaalt vaak niet lineair. Zo’n correctie is dus geen vervanging voor het af en toe testen op de hardware van gebruikers. 

Hieronder een tabel met de veelgebruikte waarden voor frametimes, en de framerates die erbij horen, evenals de scenario’s waarin die waarden vaak gebruikt worden. 

| Frametime | Framerate  | Scenario                                                                                                         |
|-----------|------------|------------------------------------------------------------------------------------------------------------------|
| 33ms      | 30FPS      | Oudere consoles                                                                                                  |
| 16.7ms    | 60FPS      | Meest voorkomende framerate van monitors; framerate van moderne consoles.                                        |
| 17-10ms   | 60-100FPS  |                                                                                                                  |
| 10-5ms    | 100-200fps | Interessant voor games waar snelheid belangrijk is, bijvoorbeeld shooters die op 144Hz+ monitors gespeeld worden |
| <5ms      | \>200FPS   | Zeldzaam nodig; weinig monitors ondersteunen zulke hoge framerates.                                              |
