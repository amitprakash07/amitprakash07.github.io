---
layout: post
title: "Project coursework: Chasing is fun — basic AI"
date: 2015-09-06
tags:
  - unity
  - navmesh
  - ai
  - game-development
  - csharp
source_wix: "https://amitprakash07.wixsite.com/home/post/2015/09/06/chasing-is-fun-basic-ai"
---

Chasing the player is very common in most of the Games - especially RPG games. The basic idea of the Chasing AI is to chase the particular character when it comes in radius of the character who's reponsiblity is to chase and damage the player. This could be the basic AI for AI system. This was my first task in this semester to implement it.  

  

There are various ways to implement this. One way is to find the position of the player and look for the threshold range, If the player is in the range, enemy/NPC/monster can start walking/running towards the player. The only complicated part is to calculate the shortest path in the environment (which can also be implemented using A* algorithm).  

  

Unity in-built navigation system solves this job very efficiently. All we have to do is to add Navigation Component to the NPCs/Monsters/Enemies.  

![Unity navigation / agent setup]({{ '/assets/img/blog/2015/chasing-ai-basic/01-navigation-setup.png' | relative_url }})

Apart from adding component, we need to tell the Unity system about the scene containing obstacle which needs to be considered while calculating the navigation Mesh. Navigation Mesh in Unity are pre-baked (pre-determined) information about the structure of the scene environment. It mainly helps Unity to calculate various possible path avoiding all obstacle so that any Nav Mesh Agents can go to different location in the scene avoiding the NavMesh Obstacle. Nav Mesh Obstacle are also Unity components which needs to be added a Unity Object if they needs to be avoided in the path tracing.  



![Scene with navigation mesh]({{ '/assets/img/blog/2015/chasing-ai-basic/02-nav-mesh-scene.png' | relative_url }})

After adding these components to the relevant Unity gameobjects, all we need to do is that to bake the Navigation Mesh for the scene. It's a good idea to make game Objects (which won't move at all) as static in Navigation Mesh. This helps Unity to pre-calculate things which makes the game run  faster. 



![Navigation / bake UI]({{ '/assets/img/blog/2015/chasing-ai-basic/03-navigation-ui.png' | relative_url }})

### Output

![Basic chase AI in the scene]({{ '/assets/img/blog/2015/chasing-ai-basic/04-chase-ai-output.png' | relative_url }})

Only thing left is to Bake the scene to create the Navigation System. Another thing required in this implementation is that the Nav Mesh Agents need some sort of command on a particular condition which will intiate the chase towards the player. In this implementation, we are using distance based trigger, which I set in a C# script. 


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

### Video

[Basic Chase AI on YouTube](https://www.youtube.com/watch?v=MdsXjSAFU48)

<div class="video-embed">
  <iframe src="https://www.youtube.com/embed/MdsXjSAFU48" title="Basic Chase AI
 — YouTube video" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</div>
