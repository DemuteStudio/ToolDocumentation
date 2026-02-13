## Introduction
As Dan Reynolds, the father of Metasounds, said: “_Metasounds are not just an implementation environment, they're a design environment._” Yet in game audio, most implementation follows a handful of common designs. This pack provides professionally crafted presets for those designs, along with useful patches and a runtime debugging tool for faster, easier implementation.

## Manual Installation
In the root folder of your project, create a plugins folder if it doesn’t already exist. Drag the `Demute_Metasound_PresetsAndPatches` folder into that new plugins folder. Then activate the new plugin in edit > plugins (along with the base Metasound plugin).

<img width="777" height="169" alt="image" src="https://github.com/user-attachments/assets/2086f04c-ecf0-4915-9acb-967b9c88bfb2" />

## How to Use
#### Source Graphs & Patches
The metasounds in the `source graphs` folder and the `patches` folder contain audio logic to use as nodes. To use them in your own metasound just search for their name (use the versions without prefixes).

<img width="498" height="307" alt="image" src="https://github.com/user-attachments/assets/21ddbdc3-6982-4b4d-88f6-4e4edc051c14" />

#### Containers
The amount of logic in the source graphs may be quite overwhelming so in the `containers` folder you can find simpler looking versions of the logic (that just use the source graph as nodes). Instead of creating new metasounds, you can simply duplicate one of these containers to use as your new baseline.

The containers contain all the same inputs as the corresponding source graphs but some of these inputs are not connected to the node for easier manual editing.

<img width="776" height="387" alt="image" src="https://github.com/user-attachments/assets/ff2cfb0d-a503-4c25-b1c6-cfffbf04b364" />

Take for example the simple loop node. More often than not you will set the looping wave asset and not be changing it at runtime. Therefore the Wave Asset input is disconnected (and moved to the top left of the graph) so that you simply set the asset in the node (this feels more intuitive than changing it in the input list on the side of the screen). 

The same logic applies to the `Ignore Random Start Time` boolean value, this should be something that is “set and forget”. However it is an advanced pin and doesn’t show up on the node by default, to see it you must click on the arrow at the bottom of the node. 

All unconnected inputs are put away at the top left of the graph so you can see at a glance which ones are not controlled via inputs. If you don't like this workflow, nothing is stopping you from reconnecting these inputs to the node's corresponding inputs.

Float inputs are kept connected as they give us access to nice little dials to change values with. If the value you would like to set exceeds (or is less than) the max/min of the wheel, you can increase these values in the input editor (click on the input > Default Editor Options > Range).

## Stopping Loops
The containers containing looping metasound sources (not the sources themselves) do not have the *OneShot* interface, this allows them to virtualize when a player steps out of their attenuation range. This means that they do not have an *OnFinished* node and thus are not automatically grabbed up by garbage collection when they are done.

<img width="716" height="372" alt="image" src="https://github.com/user-attachments/assets/70c9e493-a1dc-49ed-85b0-d48f9e49bcb0" />

To allow them to be collected by garbage collection when needed, these metasounds must be stopped with the Fade Out & Stop blueprint nodes.

The following containers need to be stopped in this way (if you want them to be destroyed) : 
- Simple Loop
- Random Loop
- Flip Flop
- State Loops
- Shuffle Playlist
- Loop by Distance

#### Advanced Loop & Scatterer
The `Advanced Loop` & the `Scatterer` both need to run custom metasound logic when stopping, this means that we cannot use the same blueprint nodes as the others. However they cannot be one-shots either ; they wouldn’t be able to virtualize.

If you want them destroyed when stopped, you must place the actor component `DEM_StopLoop` found in the `ActorComponent` folder on the actor with the sound component.

<img width="775" height="236" alt="image" src="https://github.com/user-attachments/assets/90d3714c-9b1c-44a9-900d-43bd7efe5c26" />

To stop the loops, execute the corresponding trigger parameter for each container (see the container’s triggers), and the metasound logic will activate. When that logic is done, the `StopLoop` actor component will correctly stop the playback, allowing it to be picked up by garbage collection.

<img width="774" height="370" alt="image" src="https://github.com/user-attachments/assets/a544a61b-a2a0-41f8-ae3e-989f0b6fc83c" />

## Debugging
In the *ActorComponent* folder there is an actor component that can help you debug your metasounds called `DEM_MetasoundDebuggingTool`. When you place this on an actor with a sound component, it will display the name of the wave file playing in the metasound, as well as the playtime and the full duration of the wave file.

<img width="772" height="83" alt="image" src="https://github.com/user-attachments/assets/5572a748-b3fb-43c9-b662-b4803ff4aa29" />

Additionally, important actions within the metasound will also be displayed ; such as changing states, starting a crossfade, etc…

The included preset part of the documentation lists the event update logs that can be displayed.

The debugging tool has the following options :

<img width="770" height="333" alt="image" src="https://github.com/user-attachments/assets/81bbc7e8-7ef4-44a7-af40-3ad9fbbf0297" />

- **Show File Path?** - Toggle to show the complete wave asset path on top of just the file name for the playing wave asset.
- **Text Color** - Changes the display color of the debug text.
- **Multiple Component Index** - If the debug tool is placed on an actor with multiple audio components, choose which component to debug display (0 being the first one). To have multiple audio components on the same actor be debuggable at the same time, you will need to change the way the tool manages display keys.

#### Using with your own Metasounds
To make your own metasounds compatible with this debugging tool, you need to make sure they have five outputs named according to this list : 

- `Event Update` (string) - in which you log the important actions
- `Playing Wave File` (string) - for the playing file’s name
- `Playing Wave Path` (string) - for the playing file’s path (optional)
- `Playback Time` (time) - the playback time
- `Wave File Duration` (time) - the wave file’s duration

Here is how to hook up the outputs for simpler metasounds.
Playback time is just hooked up to the wave player output of the same name.

<img width="774" height="430" alt="image" src="https://github.com/user-attachments/assets/991d9e33-a905-42a2-9cd9-6b5dcc4f2ead" />

Things become more complicated when routing information from multiple wave players and randomly selected wave assets. Look into the `Debug Patches` folder for tools to help you route the information.

## Included "Presets"
_Every preset and patch made to be used as nodes all have tooltips for their inputs & outputs. Hold your mouse over them for more information._

#### Simple One Shot
_Comes in both Mono & Stereo._

<img width="334" height="257" alt="image" src="https://github.com/user-attachments/assets/97cb2b71-b5ac-448b-92ae-c42d64cf6344" />

Plays a simple one shot with pitch shifting and debugging outputs.

List of Event Update Messages :
- “Playback Started”
- “Playback Finished”

#### Random One Shot
_Comes in both Mono & Stereo._

<img width="328" height="286" alt="image" src="https://github.com/user-attachments/assets/2a2f4c2b-4968-465d-9621-6b18155e5ea9" />

Plays a one shot randomly selected with optional pitch shifting and debugging outputs.

List of Event Update Messages :
- “Playback Started”
- “Playback Finished”

#### Simple Loop
_Comes in both Mono & Stereo._

<img width="337" height="381" alt="image" src="https://github.com/user-attachments/assets/691b2bdd-16d1-49a3-995c-e95ca6f830f7" />

Loops a sound with a random start time, fades in/out and debugging outputs.

List of Event Update Messages :
- “Playback Started”
- “Fade In Completed”

#### Random Loop
_Comes in both Mono & Stereo._

<img width="331" height="414" alt="image" src="https://github.com/user-attachments/assets/1ee978a0-297a-4e37-95ca-a996765e4bf0" />

Loops a single asset randomly chosen from a list of assets with a random start time, fades in/out and debugging outputs.

List of Event Update Messages :
- “Playback Started”
- “Fade In Completed”

#### Switch
_Comes in both Mono & Stereo, with 8, 16, or 32 switch values._

<img width="319" height="463" alt="image" src="https://github.com/user-attachments/assets/eee0135c-a25f-42ab-b26c-470cd723324a" />

Play a random one-shot from one of 8/16/32 arrays driven by the Switch Value input, each individual switch acts like a Random One Shot.

List of Event Update Messages :
- “Playback Started”
- “Playback Finished”
- “Switch Index Out Of Bounds”

#### Flip Flop
_Comes in both Mono & Stereo._

<img width="333" height="523" alt="image" src="https://github.com/user-attachments/assets/417a37b2-0290-4747-a202-cc4917cf1464" />

Crossfades between two different loops on command.

List of Event Update Messages :
- “Playback Started”
- “Fade In Completed”
- "Crossfade Started"
- "Crossfade Completed"

#### State Loops
_Comes in both Mono & Stereo._

<img width="331" height="426" alt="image" src="https://github.com/user-attachments/assets/26dd86a5-68d5-484a-b0c7-2038de5f56af" />

Plays a loop driven by a state value, on value change will crossfade into the corresponding version of the loop.

List of Event Update Messages :
- “Playback Started”
- “Fade In Completed”
- "Crossfade Started"
- "Crossfade Completed"
- "State Change"

#### Loop by Distance
_Comes in both Mono & Stereo._

<img width="305" height="643" alt="image" src="https://github.com/user-attachments/assets/3b9bfa54-85ff-40ca-ac0b-f666a578f0b2" />

Fades between three loops depending on listener distance to emitter.

List of Event Update Messages :
- “Playback Started”
- “Fade In Completed”
-  “Entered Far Range”
- “Entered Far/Medium Range”
- “Entered Medium Range”
- “Entered Medium/Close Range”
- “Entered Close Range”

#### Shuffle Playlist
_Comes in both Mono & Stereo._

<img width="347" height="445" alt="image" src="https://github.com/user-attachments/assets/0f686d7d-8ee9-4542-89a1-cae4ed0d9a8a" />

Plays through a playlist of audio assets on shuffle with crossfades.

List of Event Update Messages :
- “Playback Started”
- “Fade In Completed”
- "Crossfade Started"
- "Crossfade Completed"
- "Starting Next Track"

#### Advanced Loop
_Comes in both Mono & Stereo._

<img width="372" height="638" alt="image" src="https://github.com/user-attachments/assets/e038af3b-d34d-4de3-8eb7-f2511bc51c4b" />

Loops a wave asset with a startup one-shot and a shutdown one-shot.

List of Event Update Messages :
- “Playback Started”
- “Playback Finished”
- "Startup Completed"
- "Shutdown Started"

#### Scatterer
_Scatters Mono To Stereo_

<img width="367" height="554" alt="image" src="https://github.com/user-attachments/assets/337b374f-de23-42da-be8b-0c804f88ff6c" />

Scatters up to 8 one-shots simultaneously (random distance, pan & pitch).

List of Event Update Messages :
- “Playback Started”
- “Playback Finished”
- "Scatterer Turned Off"

## Included Patches
_Every preset and patch made to be used as nodes all have tooltips for their inputs & outputs. Hold your mouse over them for more information._

#### Get Pitch Shift

<img width="323" height="95" alt="image" src="https://github.com/user-attachments/assets/a527d6c6-246d-4e08-8b5a-394b31c93a18" />

Generates a random float value between -Pitch Variation and +Pitch Variation to be used as a pitch shifting input for a wave player.

#### Random Start Time

<img width="308" height="128" alt="image" src="https://github.com/user-attachments/assets/aa69cd4c-4c91-4e44-a6e4-541e770be0be" />

Selects a random start time for a wav file using the file's length as the upper bounds of the selection.

#### Modular Start Time

<img width="301" height="175" alt="image" src="https://github.com/user-attachments/assets/539bb065-d5ec-480d-a21f-3f961c0a52ad" />

Generates a random start time or returns start of the file if set to ignore. A modular building block for the looping presets.

#### Timed Crossfade

<img width="221" height="202" alt="image" src="https://github.com/user-attachments/assets/5de3a586-1dd7-44a7-8d1d-7106386f0fc2" />

Allows for linear crossfading between two audio streams depending on an input crossfade length (time) in seconds.

#### Trigger Select Overflow

<img width="249" height="356" alt="image" src="https://github.com/user-attachments/assets/f4dee96a-1afd-48cc-866b-9644e279d0bb" />

A `Trigger Select` node that returns overflow values and triggers instead of ignoring them. The returned **Overflow** value is `Index - 8` to be plugged into the next `Trigger Select` in the chain.

#### Scatter Mono Asset

<img width="333" height="342" alt="image" src="https://github.com/user-attachments/assets/76800e20-9c30-46d4-9afc-c6a29b4338db" />

Scatters a single mono asset (random distance, pan and pitch). Used in the `Scatterer` source graph.

#### Debug Patches
This small collection of patches are used to route string and wave values for debugging purposes.

## Credits & Licensing
- Created by **Demute**.
- Free for personal and commercial use in your projects.
- Please do not resell the pack.
- Attribution is appreciated but not required.
