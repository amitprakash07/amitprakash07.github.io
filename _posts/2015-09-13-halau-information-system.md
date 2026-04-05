---
layout: post
title: "Halau - Information System"
date: 2015-09-13
tags:
  - unity
  - ui
  - game-development
  - tools
  - halau
  - editor-workflow
source_wix: "https://amitprakash07.wixsite.com/home/post/2015/09/13/halau-information-system"
---

**Halau** is a small in-game information system for presenting details about the various **gods** the player can discover after a specific event in the game.

Halau, is a small system giving information about the various Gods which can be seen by the players/user upon some event (yet to be defined). This was little unconventional for me to work on it as I never worked with UI system of Unity. Although, Unity UI system is very easy to use, I had to spend some hours to learn especially about placement of the anchors and pivots for UI objets.

Every UI element in Unity have two important settings, Anchors and Pivots. Achors actually defines the positioning of the UI Object with respect to the Unity UI Canvas. If these anchors are not placed correctly, the UI elements will may not stay at exact placed position upon change of the game resolution or resizing of game window.

This system was very simple and contains four buttons for four Gods and upon clicking it gives it's information. I designed the system in such a way that these descriptions can be set from the editor itself so that the level designer may not have to worry about background coding (which is very small).

![Halau information UI]({{ '/assets/img/blog/2015/halau-information-system/halau-ui.png' | relative_url }})

![Halau editor setup]({{ '/assets/img/blog/2015/halau-information-system/halau-editor.png' | relative_url }})


Interesting part of the system was not coding, it was good flow design - giving more abstract design to the level editors. I created an empty game (Info Manager) object which handles all of the underlying things. This also gives an opportunity to extend the number of Gods (buttons) which would be easy to do from the editor itself. Just we have to set the button function to the Info Manager and rest will be taken care of. 

```c#  

   public Text description;

   public Image godImage;


   public void onClick(GameObject button)

    {

        description.gameObject.SetActive(true);

        godImage.gameObject.SetActive(true);

        description.text = button.GetComponent<Info_Image>().description;

        godImage.sprite = button.GetComponent<Info_Image>().image;

    }
``` 

Below is the very basic demo of the system with arbitrary message. 

### Video

[Halau Information System on YouTube](https://www.youtube.com/watch?v=k4y85b2VjcU)

<div class="video-embed">
  <iframe src="https://www.youtube.com/watch?v=k4y85b2VjcU" title="Halau-Information System
 — YouTube video" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</div>
