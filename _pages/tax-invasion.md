---
permalink: /projects/tax-invasion
layout: single
author_profile: true
author: Daniel Munteanu
toc: true
toc_sticky: true
toc_label: "Table of Contents"
toc_icon: "cog"
---

# Tax Invasion

## Short Description

This **pixel-art tower-defense Unity** game is a personal project developed in the span of 4 months.

The premise is that a pirate crew needs to defend their treasure from G.U.M. (**G**ive **U**s **M**oney): a secret agency of tax collectors. After all, stolen treasure counts as "undeclared earnings"...

The player needs to defend their treasure by placing pirates on a hexgrid to battle against enemy waves. Pirates can be upgraded, sold and even killed. For the moment, there is only one type of enenmy and only one playable type of pirate with 2 _mutually exclusive_ upgrades.

However, I intend to continue working on this game for my dissertation project, focusing on adding more enemies, pirates and upgrades to increase replayability value.

Interesting features I implemented include:
- Hexgrid placement system
- Enemy wave spawner
- Infinite scrolling background

This project can also be found on 
<a href="https://github.com/DaniMunteanu/TaxInvasion" target="_blank" rel="noopener noreferrer">Github</a> 
and 
<a href="https://danimunteanu.itch.io/tax-invasion" target="_blank" rel="noopener noreferrer">itch.io</a> 

## Gameplay Video

{% include video id="Izfz_piTBls" provider="youtube" %}

## Playable Web Build

<div class="itchio-widget-wrapper">
    <iframe frameborder="0" src="https://itch.io/embed/4217866?bg_color=141010&amp;fg_color=ffffff&amp;link_color=f21368&amp;border_color=5d5d5d" width="552" height="167"><a href="https://danimunteanu.itch.io/tax-invasion">Tax Invasion by DaniMunteanu</a></iframe>
</div>

## Implemented Features

### Hexgrid Placement System

{% include figure popup=true image_path="/assets/images/tax-invasion/hexgrid-preview.gif" alt="Pirate snaps to a grid cell" caption="Pirate snaps to a grid cell" %}

```csharp
public class PlacePirateButton : MonoBehaviour, IBeginDragHandler, IDragHandler, IEndDragHandler
{
    // The ID of the pirate to be placed when this button is used
    // For now it is redundant since we only have one pirate, but it will be useful when we have multiple pirates to place
    public int pirateID = 0;

    public void OnEndDrag(PointerEventData eventData)
    {
        if(buttonComponent.interactable == false)
            return;
        
        placementSystem.CheckCurrentIndicatorPosition();
        placementSystem.TryPlacing(pirateID);
    }
}
```

{% include figure popup=true image_path="/assets/images/tax-invasion/hexgrid-positions.png" alt="Some of the grid coordinates where a pirate can be placed" caption="Some of the grid coordinates where a pirate can be placed" %}

```csharp
public class PlacementSystem : MonoBehaviour
{
    // The keys are grid positions where pirates can be placed
    // The values are the pirates placed at those positions (if there are any)
    private Dictionary<Vector3Int, Pirate> placedPirates = new Dictionary<Vector3Int, Pirate>();

    private void InitializePlacedPiratesData()
    {
        placedPirates.Add(new Vector3Int(8,-4,0),null);
        placedPirates.Add(new Vector3Int(7,-4,0),null);
        placedPirates.Add(new Vector3Int(6,-4,0),null);
        placedPirates.Add(new Vector3Int(5,-4,0),null);
        // Rest of the grid cells...
    }

    //Called every frame
    private void Update()
    {
        // Get mouse position in world coordinates
        Vector3 worldPos = cam.ScreenToWorldPoint(Mouse.current.position.ReadValue());
        worldPos.z = 0f;
        
        // Convert world position to tile cell position
        cellPos = tilemap.WorldToCell(worldPos);

        // Move indicator to cell center
        // (The indicator is the white hexagon displayed on top of the grid cells)
        cellCenterPos = tilemap.GetCellCenterWorld(cellPos);
        hexIndicator.transform.position = cellCenterPos;
    }

    public void CheckCurrentIndicatorPosition()
    {
        // Check if the current cell position is valid for placing a pirate
        if (placedPirates.ContainsKey(cellPos))
        {
            // Check if the current cell is already occupied
            if (placedPirates[cellPos] == null)
            {
                hexIndicator.SetActive(true);
                return;
            }
        }
        hexIndicator.SetActive(false);
    }

    // Some code lines are omitted in this snippet
    public void TryPlacing(int pirateID)
    {
        // If the indicator is active, it means the current cell position is valid for placing a pirate
        if (hexIndicator.activeSelf)
        {
            Pirate instantiatedPirate = Instantiate(piratesDatabase.piratesData[pirateID].Prefab.GetComponent<Pirate>());
            mainUI.AddPirateProfile(cellPos, instantiatedPirate);

            // Set the position of the instantiated pirate to the center of the cell
            instantiatedPirate.transform.position = cellCenterPos;
            
            // Add the instantiated pirate to the placedPirates dictionary
            placedPirates[cellPos] = instantiatedPirate;
        
            hexIndicator.SetActive(false);
        }
    }
}
```

### Enemy Wave Spawner

{% include figure popup=true image_path="/assets/images/tax-invasion/lanes.png" alt="Lanes where the enemies can come from" caption="Lanes where the enemies can come from" %}

{% include figure popup=true image_path="/assets/images/tax-invasion/round-so.png" alt="Rounds consist of enemy waves coming from multiple lanes" caption="Rounds are composed of enemy waves coming from multiple lanes"%}

{% include figure popup=true image_path="/assets/images/tax-invasion/lane-so.png" alt="Each of these waves are composed of an enemy queue" caption="Each of these waves are composed of an enemy queue" %}

I didn't want all enemies to spawn at once, which is why I added the **Max Agents Spawned** property. In the image above, the wave can only have 1 enemy alive at a time. The next one will be spawned only after the previous one dies.

```csharp
public class WaveSpawner : MonoBehaviour
{
    private Round currentRound;

    public void StartNewRound()
    {
        roundCount++;
        newRoundStarted.Invoke();

        // Load the round data from the Resources folder
        currentRound = Resources.Load<Round>("Round" + roundCount + "/Round" + roundCount);

        // Count how many enemies are in the current round so we can know when the round is over
        CountCurrentRoundEnemies();

        // Array used to keep track of how many agents are alive in each lane, so we can know when to spawn more agents
        agentsSpawned = new int[NUMBER_OF_LANES];
        
        SpawnAgents();
    }

    void SpawnAgents()
    {
        bool keepSpawning = false;

        // Check if there are any lanes that can still spawn agents
        foreach(var lane in currentRound.lanes)
            keepSpawning = keepSpawning || TryToSpawn(lane);
        
        //If so, try to spawn them again after a delay
        if(keepSpawning)
            Invoke("SpawnAgents",2);
    }

    bool TryToSpawn(LaneWave lane)
    {
        // Check if the lane can spawn more agents
        if(agentsSpawned[lane.laneIndex] < lane.maxSpawnedAgents)
        {
            // When an agent is spawned, we dequeue it from the lane's queue of agents to spawn, and instantiate it in the world
            Agent newAgent = Instantiate(lane.agents.Dequeue());
            newAgent.transform.position = lanePositions[lane.laneIndex];
            newAgent.treasurePile = treasureTarget;
            newAgent.lane = lane;

            // This method is called when the agent dies
            newAgent.agentDead.AddListener(OnAgentDead);

            agentsSpawned[lane.laneIndex]++;
            return true;
        }
        return false;
    }

    void OnAgentDead(LaneWave lane)
    {
        agentsSpawned[lane.laneIndex]--;
        currentRoundTotalEnemies--;

        if (currentRoundTotalEnemies == 0)
        {
            // Send an event if the round is over
            roundOver.Invoke();
            return;
        }

        // If there are still agents left to spawn in this lane, try to spawn them
        if (lane.agents.Count > 0)
            TryToSpawn(lane);
    }
}
```

### Infinite Scrolling Background