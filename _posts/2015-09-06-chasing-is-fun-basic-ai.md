---
layout: post
title: "Chasing is fun — basic AI"
date: 2015-09-06
tags:
  - unity
  - navmesh
  - ai
  - game-development
  - csharp
source_wix: "https://amitprakash07.wixsite.com/home/post/2015/09/06/chasing-is-fun-basic-ai"
---

**Chasing the player** shows up everywhere, especially in **RPGs**. The core idea: when the player enters a **radius**, an enemy/NPC/monster **moves toward** them (and often attacks). This was my **first task** of the semester.

There are many ways to build it. One approach: read the **player position**, test against a **threshold**, and if the player is in range, start **walking/running** toward them. The trickier piece is **pathfinding** in a cluttered level—often handled with something like **A-star** pathfinding.

**Unity’s built-in navigation** handles a lot of that well: add a **Nav Mesh Agent** to your NPCs/enemies.

![Unity navigation / agent setup]({{ '/assets/img/blog/2015/chasing-ai-basic/01-navigation-setup.png' | relative_url }})

You also teach Unity what counts as **walkable** vs **blocked**: mark static geometry, place **Nav Mesh Obstacles** on objects that should affect paths, and **bake** a **NavMesh** so the engine knows the walkable surface.

![Scene with navigation mesh]({{ '/assets/img/blog/2015/chasing-ai-basic/02-nav-mesh-scene.png' | relative_url }})

After wiring components on the right **GameObjects**, **bake** the NavMesh for the scene. Mark non-moving objects as **Navigation Static** where appropriate so Unity can precompute and keep runtime work down.

![Navigation / bake UI]({{ '/assets/img/blog/2015/chasing-ai-basic/03-navigation-ui.png' | relative_url }})

Agents still need **gameplay logic** to decide *when* to chase. Here I used a simple **distance check** in **C#**: if the player is within range, set the agent’s **destination** to the player; if they get inside **stopping distance**, **look at** the player and trigger **combat** animation; otherwise send the agent back to a **home** position.

```csharp
void Update()
{
    float distance = (transform.position - player.transform.position).magnitude;
    if (distance <= distanceToPlayer)
    {
        GetComponent<NavMeshAgent>().SetDestination(player.transform.position);
        if (distance < GetComponent<NavMeshAgent>().stoppingDistance)
        {
            transform.LookAt(player.transform);
            combat.Animate();
        }
    }
    else
    {
        GetComponent<NavMeshAgent>().SetDestination(startPosition);
    }
}
```

### Output

![Basic chase AI in the scene]({{ '/assets/img/blog/2015/chasing-ai-basic/04-chase-ai-output.png' | relative_url }})
