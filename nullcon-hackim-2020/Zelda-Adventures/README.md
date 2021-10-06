### 0x01 Challenge:

**Description:**

<img src="https://i.imgur.com/o47A2Dd.png" width=300 height=400>

### 0x02 Write-up:

For this challenge and the next ones I'm going to explain they provide us with a game in Unity. For the whole process of modifying it I will make use of dnSpy.

First of all, for this challenge we need to kill any NPC, so let's open the file Assembly-CSharp.dll from the folder Data > Managed in our dnSpy.

Within the available methods, we are going to modify Enemy > TakeDamage so that it receives the damage * 1000000 to be able to kill them in a single blow as we see in the image.

<img src="https://i.imgur.com/F8d89L9.png" width=800 height=600>

If we save the changes, log in to the game and try to attack any zombie...

<img src="https://i.imgur.com/TLRZINO.png" width=800 height=100>

<br> `Flag: REVOLUTIONSTARTSWITHME`

* * *

### 1x01 Challenge:

**Description:**

<img src="https://i.imgur.com/Gy5IN8X.png" width=300 height=400>

### 1x02 Write-up:

In this challenge we need to get to FlagTown to locate the string that validates us as a flag. To make this move easier we modify the movement speed of our character * 20.

<img src="https://i.imgur.com/3ZMqSdU.png" width=800 height=450>

We walk in search of the pond and that's it.

<img src="https://i.imgur.com/ZcsnJS2.png" width=800 height=600>

<br> [Link to the correct way](https://i.imgur.com/dJaST6x.mp4)

`Flag: BENDTHERULES42PIRATE`

* * *

### 2x01 Challenge:

**Description:**

<img src="https://i.imgur.com/x7QTATP.png" width=300 height=400>

### 2x02 Write-up:

The third challenge (the last one with this game file) sends us looking for other survivors. If we advance on the map to the end (northwest direction) we will reach the flag.

<img src="https://i.imgur.com/DhfEo9b.png" width=800 height=600>

<br> `Flag: EXPLORERFORLIFE`
