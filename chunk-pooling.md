# Pooling

Een game-wereld bevat zeer veel data. Een stukje terrein van 10 bij 10 meter bevat al gauw 400-1600 vertices. Om een beetje in de verte te kunnen kijken wil je een omgeving in een straal van ongeveer 1km tot 3km om je heen laden. Een stuk wereld van 4km bij 4km kan makkelijk honderden miljoenen vertices bevatten. Voor allerlei redenen is het slim om dit op te breken in kleinere stukjes, die in jargon chunks genoemd worden. Chunks worden in een bepaalde radius om de speler heen geladen, en als de speler zich verplaatst worden oude chunks vergeten en nieuwe gemaakt.

Mijn eerste implementatie van een chunk-systeem was ultra-simpel. Gemaakt voor snel resultaat, niet voor goede performance. De verantwoordelijkheid van dit component is om de geladen chunks bij te houden, en om nieuwe chunks te maken als de speler over een chunk-grens heenloopt.

## De `ChunkLoader`

Een `Chunk` is een MonoBehaviour component waar data over de chunk in wordt opgeslagen. De `ChunkLoader` is verantwoordelijk voor het aanmaken van `Chunks`. Zie hieronder een suboptimale versie van mijn `ChunkLoader`. Chunks worden opgeslagen in een `Dictionary<Vector2Int, Chunk> loadedChunks`, waarbij elke chunk een coördinaat (`Vector2Int`) toegewezen krijgt om ze snel terug te kunnen vinden. In `UpdateVisibleChunks()` wordt er gekeken welke chunks er nodig zijn gebaseerd op de huidige positie van de speler, en worden er nieuwe chunks aangemaakt terwijl de oude verwijderd worden.

Hoewel deze code werkt, wordt er heel inefficiënt gebruik gemaakt van het geheugen. We gaan deze code daarom als uitgangspunt gebruiken om een beter geoptimaliseerde `ChunkLoader` te schrijven. 

```csharp
public class ChunkLoader : MonoBehavior {
    
    private Dictionary<Vector2Int, Chunk> loadedChunks;
    
    // ... //
    
    private void UpdateVisibleChunks()
    {
        Dictionary<Vector2Int, Chunk> newChunks = new();
        
        // Create new chunks
        for (int x = -loadRadius; x < loadRadius + 1; x++)
        {
            for (int z = -loadRadius; z < loadRadius + 1; z++)
            {
                Vector2Int key = new Vector2Int(x + playerChunkX, z + playerChunkZ);
                if (loadedChunks.TryGetValue(key, out var chunk))
                {
                    newChunks.Add(key, chunk);
                }
                else
                {
                    newChunks.Add(key, CreateChunk(x + playerChunkX, z + playerChunkZ));
                }
            }
        }
    
        // Invalidate and remove old chunks
        foreach (var (key, chunk) in loadedChunks)
        {
            if (!newChunks.ContainsKey(key))
            {
                Destroy(chunk.gameObject);
            }
        }
    
        // Update loaded chunks
        loadedChunks = newChunks;
    }
    
    private Chunk CreateChunk(int x, int z)
    {
        Chunk chunk = Instantiate(chunkPrefab, CalculateChunkCenter(x, z), Quaternion.identity, transform);
        chunk.gameObject.name = $"Chunk ({x}, {z})";
        chunk.LoadAt(x, z);
        return chunk;
    }
    
    // ... //
}
```
## Garbage

In deze implementatie gebeuren meer heap allocations dan nodig. Als oefening kun je nagaan of je zelf de drie allocations kunt vinden voor je verder leest. 

1. Bij het aanmaken van een dictionary
    ```csharp
    ...
    private void UpdateVisibleChunks()
        {
            Dictionary<Vector2Int, Chunk> newChunks = new(); // << Heap allocation 1
            
            // Create new chunks 
            ...
    ```
   Dictionaries zijn reference types, dus ze worden opgeslagen op de heap. We maken elke keer dat de functie draait een nieuwe dictionary aan, die op een nieuwe willekeurige plek in het geheugen staat.
---
2. Bij het toevoegen van een item
    ```csharp
    ...
    if (loadedChunks.TryGetValue(key, out var chunk))
    {
        newChunks.Add(key, chunk); // << Heap allocation 2a
    }                
    else
    {
        newChunks.Add(key, CreateChunk(x + playerChunkX, z + playerChunkZ)); // << Heap allocation 2b
    }
    // NB. De if-else zorgt ervoor dat ofwel 2a, ofwel 2b altijd aangeroepen wordt, 
    // maar nooit allebei tegelijk. Het is dus feitelijk dezelfde allocation.
    ...
    ```
   Deze is een beetje gemeen, er is namelijk niet *altijd* een allocation als je `.Add(key, value)` op een dictionary aanroept. Als de capaciteit van de dictionary groot genoeg is, kan het element direct toegevoegd worden en vindt er geen allocation plaats. Maar als er geen ruimte is, moet de dictionary groeien. Dit kan alleen maar door de hele dictionary opnieuw te allocaten, met een grotere capaciteit (de capaciteit wordt in C# verdubbelt). Dan pas wordt het element toegevoegd.
---
3. Bij het aanmaken van een GameObject (`Instantiate()`)
    ```csharp
    private Chunk CreateChunk(int x, int z)
    {
        Chunk chunk = Instantiate(chunkPrefab, CalculateChunkCenter(x, z), Quaternion.identity, transform); 
                   // ^^ Heap allocation 3
        chunk.gameObject.name = $"Chunk ({x}, {z})";
        chunk.InitAt(x, z);
        return chunk;
    }
    ```
    Als je `Instantiate()` aanroept wordt er altijd een nieuw geheugen voor een GameObject, inclusief components, geallocate. Dit gebeurt in deze functie één keer voor elke nieuwe `Chunk` die gemaakt wordt.  
    > **Note**\
    In het bovenstaand stukje code zit zelfs nog een vierde allocation verstopt: `name = $"Chunk ({x}, {z})"` maakt ook gebruik van de heap. Dit is grotendeels onvermijdelijk als je werkt met strings, maar ook hier kun je optimaliseren. \
    Meer info: https://medium.com/@johnklaumann/c-10-the-dark-and-evil-true-about-string-interpolation-29e10acc001b
---
We kunnen ook duidelijk zien waar oude data als afval achtergelaten wordt

1. De oude dictionary wordt vernietigd door hem te overschrijven:
    ```csharp
    loadedChunks = newChunks; 
    ```
   De oude loadedChunks wordt overschreven met de nieuwe, waardoor de oude out-of-scope gaat en opgeruimd moet worden door de garbage collector. Je kunt je afvragen waarom de nieuwe niet gewoon de plek van de oude inneemt. Ten eerste is het nieuwe object al gemaakt, op een andere plek. Ten tweede overschrijf je alleen de referentie, omdat `Dictionary` een reference type is. 
2. Het aanroepen van `Destroy(gameObject)` genereert ook afval dat opgeruimd moet worden
    ```csharp
    // Invalidate and remove old chunks
    foreach (var (key, chunk) in loadedChunks)
    {
        if (!newChunks.ContainsKey(key))
        {
            Destroy(chunk.gameObject); // << chunk.gameObject wordt hier afval
        }
    }
    ```
   
Kortom, de memory allocator heeft het maar druk in het frame waarin deze code gerund wordt, en de garbage collector wordt flink belast als dit frame voorbij is.

## Objecten hergebruiken (*pooling*)

We kunnen ervoor zorgen dat we dezelfde objecten hergebruiken, zodat we telkens maar één keer in de hele game een object hoeven aan te maken. Er zijn een aantal manieren om dat voor elkaar te krijgen, maar het komt ook met een paar eigenaardigheden.

1. ### `loadedChunks` dictionary
Achter de schermen bestaat een verzameling (o.a. `Dictionary`, `List` en `Array`) uit een pointer naar het memory-adres van het eerste element, en een getal om bij te houden hoeveel elementen er in de lijst zitten. Iedere verzameling heeft twee eigenschappen voor de grootte: `Length` of `Count`, en `Capacity`. De lengte geeft aan hoeveel elementen er op dit moment in de lijst zitten. De capaciteit geeft aan hoeveel elementen er in de lijst *kunnen* zitten. In feite is dit hoeveel vrijgemaakt geheugen de verzameling geclaimd heeft. In het geheugen ziet dit er ongeveer zo uit:

```csharp
List<int> myList = new List { 8, 15, 88, 47 }; // Capacity en Length zijn allebei 4

---------|geclaimed geheugen |--------------
 ## | ## | 08 | 15 | 88 | 47 | ## | ## | ##
---------|----- myList ------|--------------

myList.Capacity = 6; // maak de capaciteit 6

---------|     geclaimed geheugen      |----
 ## | ## | 08 | 15 | 88 | 47 | ## | ## | ##
---------|---------- myList -----------|----
```

Een verzameling kan leeggemaakt worden met de functie `Clear()`. `Clear()` zorgt ervoor dat alle data in de verzameling gewist wordt, terwijl de capaciteit gelijk blijft. We kunnen dan het oude geheugen gebruiken om nieuwe data op te slaan. Omdat we tijdens 

```csharp
public class ChunkLoader : MonoBehavior {
    
    private Dictionary<Vector2Int, Chunk> loadedChunks;
    
    private void Start() 
    {
        loadedChunks = new Dictionary<Vector2Int, Chunk>(ChunkCount); // Maak een dictionary één keer aan met een capaciteit van het aantal chunks.
    }
    
    private void UpdateVisibleChunks()
    {
        Dictionary<Vector2Int, Chunk> newChunks = new();
        
        // Create new chunks
        for (int x = -loadRadius; x < loadRadius + 1; x++)
        {
            for (int z = -loadRadius; z < loadRadius + 1; z++)
            {
                Vector2Int key = new Vector2Int(x + playerChunkX, z + playerChunkZ);
                if (loadedChunks.TryGetValue(key, out var chunk))
                {
                    newChunks.Add(key, chunk);
                }
                else
                {
                    newChunks.Add(key, CreateChunk(x + playerChunkX, z + playerChunkZ));
                }
            }
        }
    
        // Invalidate and remove old chunks
        foreach (var (key, chunk) in loadedChunks)
        {
            if (!newChunks.ContainsKey(key))
            {
                Destroy(chunk.gameObject);
            }
        }
    
        // Update loaded chunks
        loadedChunks = newChunks;
    }
}
```

> **Tip**\
> Je kunt een verzameling een start-capaciteit meegeven. Als je van tevoren al weet hoeveel elementen je moet gaan opslaan (wat vaak het geval is), zorg je er zo voor dat je maar één keer geheugen hoeft vrij te maken. Je doet dit in de meeste gevallen door de capaciteit in de constructor mee te geven: `new Dictionary<int, string>(capacity)`.

2. ### `Chunk` GameObjects

## Verdere optimalisatie