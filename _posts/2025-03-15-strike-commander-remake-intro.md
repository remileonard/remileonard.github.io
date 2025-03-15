---
layout: post
title:  "Strike Commander Remake, We have an Intro"
date:   2025-03-15 12:18:59 +0100
categories: LibRealSpace
use_mermaid: true
---

The videos, or midgames, in Strike Commander remain a true mystery to me. I have searched and re-searched the data, uncovering the music, images, and texts, but nothing that explains how to piece them together. Here is how I solved the problem to recreate the game's introduction.

![prideau dans l'intro](/assets/img/Intro.png)
<!--more-->

# The Data

The game's assets are all stored in TRE-format archives. The format is well-documented, and with a bit of effort, every image and animation can be identified. However, the introduction involves more than just sequences of images stitched together. There are scrolling effects, text, sounds, animations, and music.

I searched for a long time for a file that, like the rest of the game, would contain the definition of the sequences to display, possibly with some opcodes to create the effects. But nothing—nada—no such information can be found in the game's assets.


# However, Some Clues

If I were skilled in reverse engineering with tools like IDA or GHIDRA, I might have more information. While going through the executable file *OPTEST.EXE*, I came across some interesting findings.

At offset `0x49CFC`, you can find the string "air," and a bit further along, "Back off, almost got tone."


![executable affichage chaine air](/assets/img/exe.png)

It turns out that the value "air" corresponds to the internal identifier the game uses to display the player's face in conversations. If you replace "air" with the identifier of another character in the game (while keeping it to three letters), you can modify the introduction to display that character's face instead.


![prideau dans l'intro](/assets/img/pri_intro.png)

It seems that the data I'm interested in is located within the executable, and a thorough decompilation should allow me to retrieve it and use it more easily.

But here's the thing—even if I manage to find my string in Ghidra:  

![ghidra affichage chaine air](/assets/img/back_off_ghidra_1.png)

Unfortunally Ghidra doesn't find any reference to the address :(

![ghidra affichage chaine air](/assets/img/back_off_ghidra.png)

# La solution.

For now, while waiting to gain more knowledge in disassembly and decompilation, I turned to a good old technique: re-coding the data to display the introduction.

I implemented the introduction sequence using the following data schema

<div class="diagram-container" id="diagram-container">
<pre class="mermaid">

flowchart TD
    MIDGAME --> SEQUENCE
    SEQUENCE --> SHOTS
    SHOTS --> BACKGROUND
    SHOTS --> SPRITES
    SHOTS --> FOREGROUND
    SHOTS --> MUSIC
    SHOTS --> SOUNDS

</pre>
</div>

I didn’t start from scratch. I drew inspiration from the data model used to represent the game in *GAMEFLOW.IFF*.  

# The Result

The result is this first video of the game's introduction. I'm thrilled to finally be able to display it, though there’s still much to do to implement the data for the other videos. Perhaps, in the meantime, I’ll discover how to decompile the executable properly, allowing me to extract the data without having to comb through the game's assets to find them.

<div style="text-align:center">
<iframe width="560" height="315" src="https://www.youtube.com/embed/qbw6tIBHlrE?si=n94k5w5d9hTqH00b" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
</div>

See you soon, and happy flying!  
