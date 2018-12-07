The Southbird 2011 Super Mario Bros. 3 Disassembly
==================================================

Hello, Southbird here. If you're just looking to just PLAY Super Mario Bros. 3Mix, this page is unimportant to you. If you're looking for technical information about the disassembly that made this possible, read on.

An important goal of this disassembly is that it must reassemble byte-for-byte back into its source. That is, I worked from the Super Mario Bros. 3 USA Release Revision 2 (i.e. "Super Mario Bros. 3 (U) (PRG1)") and, after assembly is complete, the assembled image should be byte-for-byte identical with it. This was the same goal as the [Sonic the Hedgehog](http://info.sonicretro.org/Disassemblies) disassemblies and I felt this was an important goal. All that really means to you is I also had to maintain all data -- used or unused -- and particularly with the levels that can be ugly. There is, in fact, over 50 completely empty level headers in the bank that holds "Water" levels. I don't really know why this is the case, but it is, and every single one is maintained for integrity. The greatest likelihood is that you're going to want to obliterate all of them to make room for your own levels.

Table of contents kind of sucks here but I already spent 7 hours on a Saturday getting this far, so you'll just have to accept it for now :)

This document contains a ton of information. If you just want to build and run the disassembly, all that really needs to be done is to run "nesasm" against "smb3.asm" in the bin/SMB3 folder. Note that running NoDice and saving a level also invokes "nesasm." Have fun out there guys.

-   [Some Questions to Get Out of the Way](#Some)
    -   [A brief overview of the MMC3 and the PRGxyz madness](#A)
    -   [Fixed point and 16-bit coordinates](#Fixed)
    -   [PRGxyz_abcd labels?](#PRGxyz)
    -   [Consistency](#Consistency)
    -   [Jump (technically always)' to ...](#Jump)
    -   [Lost Bonus Games](#Lost)
    -   [Rolling Die](#Rolling)
    -   [Alternate Hosts / Koopa Troopa's Prize Game](#Alternate)
    -   [Workflow for Bonus Games](#Workflow)
    -   [Other Points of Interest](#Other)
    -   [Tiles, Tile Quadrants, Tilesets](#Tiles)
    -   [DynJump (Finite State Machine, indexed jump table)](#DynJump)
    -   [Objects, Special Objects, and Cannon Fires](#Objects)
    -   [Level Format](#Level)
    -   [Music Format (defined in PRG028 and PRG029)](#Music)
    -   [SEGMENT HEADER](#SEGMENTHDR)
    -   [SEGMENT START/END/LOOP](#SEGMENTSEL)
    -   [TRACK DATA](#TRACK)
    -   [MUSCONV TOOL OVERVIEW](#MUSCONV)
    -   [MUSCONV TOOL MIDI STRUCTURE](#MUSCONVSTR)
    -   [SEGEND SysEx (53 45 47 45 4e 44 f7)](#SEGEND)
    -   [STOP SysEx (53 54 4f 50 f7)](#STOP)
    -   [LOOP SysEx (4c 4f 4f 50 f7)](#LOOP)
    -   [PRG bank overview](#PRG)
    -   [PRG000) Bank 0 is mainly a bank that contains shared functionality/data for the main gameplay objects.](#PRG000)
    -   [PRG000.1) Slope tile definition, slope shape definition](#PRG000.1)
    -   [PRG000.2) Misc. data](#PRG000.2)
    -   [PRG000.3) Shared object functions (see source for full description) that are commonly called by object:](#PRG000.3)
    -   [PRG000.4) Glue routines (the game engine calls these, but you might want to know about their existence)](#PRG000.4)
    -   [PRG000.5) Additional data they put in here:](#PRG000.5)
    -   [PRG000.POI) Points of Interest](#PRG000.POI)
    -   [PRG001) First bank for object code. Contains code for object IDs $00-$23. Of all the object banks, this one has the most gaps in IDs, and a few objects that are never used in-game with strange behaviors.](#PRG001)
    -   [PRG001.1) As in all object banks, it starts off with the regular, fixed-position common lookup tables as required; ObjectGroup00_InitJumpTable, ObjectGroup00_NormalJumpTable, ObjectGroup00_CollideJumpTable, ObjectGroup00_Attributes/2/3, ObjectGroup00_PatTableSel, ObjectGroup00_KillAction, ObjectGroup00_PatternStarts. Let's go through these just once, but the idea is the same for banks 1-5. The engine just assumes these lookup tables appear at particular addresses (which, as long as their position and order is left intact, they absolutely will) and this is enforced by use of ".org" tags. Just note the ".org" tags are not strictly necessary (and as long as the data is good, effectively do nothing), but are pretty much there as reminders.](#PRG001.1)
    -   [PRG001.POI) Points of Interest](#PRG001.POI)
    -   [PRG002) Second bank for object code. Contains code for object IDs $24-$47.](#PRG002)
    -   [PRG002.1) As in all object banks, it starts off with the regular, fixed-position common lookup tables as required; see PRG001.1 for the breakdown.](#PRG002.1)
    -   [PRG003) Third bank for object code. Contains code for object IDs $48-$6B.](#PRG003)
    -   [PRG003.1) As in all object banks, it starts off with the regular, fixed-position common lookup tables as required; see PRG001.1 for the breakdown.](#PRG003.1)
    -   [PRG003.POI) Points of Interest](#PRG003.POI)
    -   [PRG004) Third bank for object code. Contains code for object IDs $6C-$8F.](#PRG004)
    -   [PRG004.1) As in all object banks, it starts off with the regular, fixed-position common lookup tables as required; see PRG001.1 for the breakdown.](#PRG004.1)
    -   [PRG004.POI) Points of Interest](#PRG004.POI)
    -   [PRG005) Fourth and final bank for object code. Contains code for object IDs $90-$B3, and also the special extended object IDs $B4+ that define "Cannon Fire" and "Event"](#PRG005)
    -   [PRG005.1) As in all object banks, it starts off with the regular, fixed-position common lookup tables as required; see PRG001.1 for the breakdown.](#PRG005.1)
    -   [PRG005.2) The "special" objects, ID $B4+, are handled with the check at PRG005_B8DB. These objects do NOT conform to the typical definitions and are basically more for triggering events and creating the special "Cannon Fire" objects.](#PRG005.2)
    -   [PRG005.POI) Points of Interest](#PRG005.POI)
    -   [PRG006) This bank actually contains the object layouts for ALL levels. Given 8192 bytes, 3 bytes per object definition, the unused start byte and $FF termination byte, and the implied limit of 48 objects per level, the maximum possible "fully filled" levels would be 8192 / (1 + 3 * 48 + 1) = 8192 / 146 = only 56 levels! But rest assured, most levels don't come anywhere near full capacity, and quite a few "alternate" levels have no object definitions at all, so it generally is not an issue.](#PRG006)
    -   [PRG006.POI) Point of Interest](#PRG006.POI)
    -   [PRG007) This bank is a peculiar one. It has a lot of miscellaneous gameplay functions that, I suppose, didn't fit anywhere else. But more solidly it is the defintion of all "special" and "cannon fire" objects. The former is a simplified object type -- without all the states and a lot of the overhead of the main enemy-type objects -- and the latter is a special object type that's extremely limited and only really designed to support a cannon firing.](#PRG007)
    -   [PRG007.1) The Miscellaneia -- all called by Gameplay_UpdateAndDrawMisc](#PRG007.1)
    -   [PRG007.2) Note that with Special Objects there is no implicit states or a lot of the other prepackaged stuff as seen with the mainstream objects. Of note, Special Objects have no "X Hi" (i.e. no 16-bit X coordinate) and will be destroyed if they fall horizontally off-screen. They're meant to be mostly transitory, and thus typically are projectiles or occasional, short-life objects like the coins that pop out of bricks. SpecialObjs_UpdateAndDraw is the entry point for updating and drawing special objects.](#PRG007.2)
    -   [PRG007.3) Cannon Fires -- See "Objects, Special Objects, and Cannon Fires" for more information about what they are. CannonFire_UpdateAndDraw is the entry point for updating and sometimes drawing cannon fires. Most cannon fires are invisible until they actually fire.](#PRG007.3)
    -   [PRG007.POI) Points of Interest](#PRG007.POI)
    -   [PRG008) This bank contains a lot of the Player specific code and animation frame lookup tables. Actual drawing of the Player occurs in bank 29.](#PRG008)
    -   [PRG008.1) The Animation Frame LUTs -- too numerous to list, follow comments in PRG008's source. Examples of what you'll find here is "Player_WalkFramesByPUp" (walking frames by power-up), "Player_TailAttackFrames" (frames used during tail attack), etc.](#PRG008.1)
    -   [PRG008.2) Player_DoGameplay -- this is a major macro subroutine which pretty much makes all the calls to make a playable character.](#PRG008.2)
    -   [PRG008.3) Player_XAccel* lookup tables: this controls the Player's acceleration in different states. If you're interested in tweaking the horizontal movement rate of the Player.](#PRG008.3)
    -   [PRG008.4) Level Action Tiles: Some tiles are tiles that the Player can hit the right way to get a bounce out of them, e.g. [?] blocks, note blocks, etc.](#PRG008.4)
    -   [PRG008.5) Special Tiles: Certain tiles in certain tilesets cause special things to happen, e.g. conveyor belts, spikes, slippery ice, etc. The macro subroutine is "Player_DoSpecialTiles." It has a lot of tileset-specific coding, so you'll definitely want to go through it if you want something special to happen in a particular tileset!](#PRG008.5)
    -   [PRG008.POI) Points of Interest](#PRG008.POI)
    -   [PRG009) This bank is used for the 2P Vs challenge mode (when a Player challenges the other Player in a 2P game.) While this is mostly pretty straightforward, there's still a few hidden gems in here! Of note, every time a challenge is initiated, an internal counter is incremented which sets the "game type." This most commonly sends you to the "4 pipes and POW block" battle arena, but sometimes you get one of the other special ones like the "fountain" or the "ladder" area. This also includes the actual level layout data for all of the arenas, which incidentally are the same format as all other levels. This bank also includes code for auto-scroll levels, probably just a convenient bit of free space rather than any sort of organization.](#PRG009)
    -   [PRG009.1) Vs_EnemySetByGameType: This points to the set of enemies that will be used in this 2P Vs game type. This only applies to the "battle arena" levels; otherwise this value is unused.](#PRG009.1)
    -   [PRG009.2) Vs_2PVsInit: The macro subroutine which performs initializations for this 2P Vs game type. For all regular arenas this doesn't do much, but this does place coins in the special "statically placed coins" arena variant and the hidden coin blocks in the ladder variant.](#PRG009.2)
    -   [PRG009.3) Vs_2PVsRun: The macro subroutine which actually runs 2P Vs game logic! Again, of course, it's mostly the same for the common arena, and there's specific logic for the special variants. However, see PRG009.POI for some tidbits... Essentially the "features" of a game type are determined by different subroutines you must call to utilize a feature.](#PRG009.3)
    -   [PRG009.4) 2P Vs Object related](#PRG009.4)
    -   [PRG009.5) AutoScroll_Do: Macro subroutine used for the various sorts of automatic scrolls that occur in levels that are not the normal "camera on Player" type following. "Level_AScrlSelect" chooses the type of auto-scroll that is active. Auto-scroll is activated in a level by using a Object ID $D3 (OBJ_AUTOSCROLL) which configures automatic scrolling based on its Y coordinate (used as a parameter rather than a literal coordinate location.) Auto-scroll is responsible most obviously for levels which move horizontally and sometimes vertically on their own. Other somewhat subtler effects are, for example, the spiked ceiling in the World 1 Mini-Fortress, which employs a "32 pixel partition" floor and scrolls the rest of the area for the spiked ceiling. While most of the subtler effects are hardcoded and pretty straightforward, the horizontal-driven auto-scroll is worth looking in to a little bit...](#PRG009.5)
    -   [PRG009.POI) Point of Interest](#PRG009.POI)
    -   [PRG010) The first bank controlling the World Map. Contains code for the pop-up "message boxes", post-Fortress FX, and handling Player movement.](#PRG010)
    -   [PRG010.1) The first part of the bank contains data in SMB3's standard simple video data instruction format. These are all the "message box" type displays of the World Map. Of important note, the map scroll position only ever sits idle at $00 or $80 (i.e. "aligned" or "aligned half"), and while this may appear outwardly to be stylistic, it also simplifies (and, in this case, hardcodes) how the "message boxes" are displayed. For each box, there is a "00" version ("aligned") and an "80" version ("aligned half") so something to keep in mind if you wish to alter these.](#PRG010.1)
    -   [PRG010.2) With that out of the way, we can move on to the proper map stuff! Of note there's the simple lookup table](#PRG010.2)
    -   [PRG010.3) This bank draws to a close with DMC samples 8, 3, and 7. These samples thus only play correctly on the world map.](#PRG010.3)
    -   [PRG010.POI) Points of Interest](#PRG010.POI)
    -   [PRG011) This bank continues map functionality from PRG010. Here we see the implementation of map objects (Hammer Bros, the bonus objects, HELP bubble, Airship, etc.) "Twirling to start" (game over), warp whistle, hand trap action code...](#PRG011)
    -   [PRG011.1) This bank begins with several arrays that define the set of map objects. Each world is granted a fixed size set of exactly 9 map objects. If any slots are to be unused, just use the constant value of MAPOBJ_EMPTY as the ID. Note that slot 0 is expected to be the HELP bubble (or absense thereof after the Airship has begun) and slot 1 is expected to be the Airship (thus starts as MAPOBJ_EMPTY until called upon.) Other than that, the slots are free, but again, remember that there should be room for a bonus object or two (N-Spade, White Toad House, etc.)](#PRG011.1)
    -   [PRG011.2) In to the map code...](#PRG011.2)
    -   [PRG011.3) Airship Travel Data. The Airship can travel to any of 3 sets each containing 6 destinations. The set is random (chosen when the world starts), the destinations are visited in incremental order.](#PRG011.3)
    -   [PRG011.POI) Points of Interest](#PRG011.POI)
    -   [PRG012) This bank is officially the "tileset" bank for the World Maps (Tileset 0), containing the data which makes up the 16x16 tiles from their 8x8 pattern components and provides the list of tiles which are enterable (Tile_Attributes_TS0.) Also contains some of the bootstrap logic for the World map. Like all tileset banks, this is also where the World Map construction data is stored, although this is the only instance where a tile grid is not in the standard level format. World Maps are stored as a raw grid of tiles.](#PRG012)
    -   [PRG012.1) Palettes](#PRG012.1)
    -   [PRG012.2) World Map structure data: World map structure data is broken up in the following way:](#PRG012.2)
    -   [PRG012.POI) Points of Interest](#PRG012.POI)
    -   [PRG013) Tileset 14: Underground. This bank is a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information. Note that this bank can actually fully support a "Hills" level, and sometimes does.](#PRG013)
    -   [PRG013.POI) Points of Interest](#PRG013.POI)
    -   [PRG014) Tileset 18, 2P Vs. This bank is a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information. It also contains shared Tileset generators.](#PRG014)
    -   [PRG014.POI) Points of Interest](#PRG014.POI)
    -   [PRG015) Tileset 1: Plains. This bank is a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information. Compared to some tileset banks, this one is straightforward, very packed (with levels), and does not contain any "empty" levels.](#PRG015)
    -   [PRG015.POI) Point of Interest](#PRG015.POI)
    -   [PRG016) Tileset 3: Hills. This bank is a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information. Note that this bank can actually fully support an "Underground" level, and sometimes does.](#PRG016)
    -   [PRG016.POI) Points of Interest](#PRG016.POI)
    -   [PRG017) Tileset 4 and 12: High-Up and Frozen Tundra. This bank is a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information. These tilesets are ultimately different styles, so some of the shared generators only make sense in one or the other. The merge idea is that they had enough similar between them that the majority of generators would work for both of them.](#PRG017)
    -   [PRG017.POI) Points of Interest](#PRG017.POI)
    -   [PRG018) Tileset 6, 7, and 8: Underwater levels, Toad houses, and Vertical levels. This bank is a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information. These tilesets are ultimately different styles, so some of the shared generators only make sense in one or the other. The merge idea is that they had enough similar between them that the majority of generators would work for all of them. Of particular note, Tileset 8 is the one to actually include support for "pipe maze" type areas, but in fact, any SMB3 level can operate in "Vertical" mode with more or less success, depending on the generator used and other factors. The pipe maze generators however do not work in a non-vertical level as-is. ONLY Tileset 8 levels are officially stored vertical, and conversely, Tileset 8 levels are officially never non-vertical.](#PRG018)
    -   [PRG018.POI) Points of Interest](#PRG018.POI)
    -   [PRG019) Tileset 5, 11, and 13: Plant infested levels, Giant World, and Sky levels. This bank is a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information. These tilesets are ultimately different styles, so some of the shared generators only make sense in one or the other. The merge idea is that they had enough similar between them that the majority of generators would work for all of them.](#PRG019)
    -   [PRG019.POI) Points of Interest](#PRG019.POI)
    -   [PRG020) Tileset 9: Desert. This bank is a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information. Of note, this tileset has the most unused tiles and yet relatively very few levels. Seems like there were greater plans for this tileset that were never reached.](#PRG020)
    -   [PRG020.POI) Points of Interest](#PRG020.POI)
    -   [PRG021) Tileset 2: Fortress. This bank is a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information.](#PRG021)
    -   [PRG021.POI) Points of Interest](#PRG021.POI)
    -   [PRG022) Tileset 15, 16, and 17: The Bonus Game bank! This bank alone has the most lost gameplay content in all of SMB3. Bonus games in SMB3 pretty much boil down to the Roulette (Spade Panel) game and the Card Matching (N-Spade) game. But originally there was a much more complex and varied system planned. This bank is also a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information.](#PRG022)
    -   [PRG022.1) First off, this is a tileset bank. Why? Well, the bonus introduction scene with the Mario and (typically) Toad talking about how to win at the Roulette or Card Matching is a "level" in the standard format. The background is generated, the border is generated, the table is generated, and even the Player graphic is generated from tiles! (The host, however, is a sprite construction.)](#PRG022.1)
    -   [PRG022.2) The actual gameplay logic is so intertwined with the lost content that there's not much point in describing it here as it would be redundant. See the section "Lost Bonus Games."](#PRG022.2)
    -   [PRG023) Tileset 10: Airships / Coin Ships. This bank is a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information.](#PRG023)
    -   [PRG023.POI) Point of Interest](#PRG023.POI)
    -   [PRG024) This bank handles the title screen and ending sequences along with the "Toad and King" end-of-world scenarios (BEFORE Airship; after airship is PRG027)](#PRG024)
    -   [PRG024.1) "Toad and King" Cinematic (BEFORE Airship; after airship is PRG027); so "King" here refers to the TRANSFORMED king](#PRG024.1)
    -   [PRG024.2) Title Screen. Semi-autonomous activity sequence for the enjoyment of viewers leaving the game idle. Really should have had music/sound I think, but maybe that would get annoying for store displays?](#PRG024.2)
    -   [PRG024.3) Ending Sequence.](#PRG024.3)
    -   [PRG024.POI) Points of Interest](#PRG024.POI)
    -   [PRG025) Ending Sequence continued; contains the rest of the sprite lists and all of the full-screen ending images. Also contains the graphics to load for the title screen logo (pieced apart due to limited buffer lengths)](#PRG025)
    -   [PRG025.1) Ending continued](#PRG025.1)
    -   [PRG025.2) Title screen palette/logo graphics; see Video_Upd_Table2 in source. Standard format as described in PRG030's "Video_Upd_Table."](#PRG025.2)
    -   [PRG026) Player inventory management, pointers to Big [?] and Generic Exit areas and the greater Level Junctioning code, palette fade in/out routines, status bar stuff](#PRG026)
    -   [PRG026.1) Inventory](#PRG026.1)
    -   [PRG026.2) Level Junctioning (and Big [?] areas)](#PRG026.2)
    -   [PRG026.3) Status Bar stuff and miscellaneous video updates](#PRG026.3)
    -   [PRG026.POI) Points of Interest](#PRG026.POI)
    -   [PRG027) Toad and King cinematic AFTER the wand is recovered (So "King" is the king restored), palette data](#PRG027)
    -   [PRG027.1) T&K Cinematic, post-airship](#PRG027.1)
    -   [PRG027.2) Palette data; this defines 8 BG palettes and 4 sprite palettes for every tileset, indexed by table Palette_By_Tileset. Use this to trace back to the set of palettes you're targeting by tileset.](#PRG027.2)
    -   [PRG028) The Sound Engine. See "Music Format" for how the music format is composed.](#PRG028)
    -   [PRG028.1) Sound Effects. Sound effects are coded wave programmer routines; they generally do not use data (except for the "map" sounds, which does have its own very primitive format); if sound effects do use data, it's data specific to the routine in question. Basically, editing sound effects is more akin to general hacking. There is no way to make a "sound effect converter" like the MusConv tool is for music.](#PRG028.1)
    -   [PRG028.2) Music. This does have a defined format used for all songs (as defined in "Music Format".) Each segment of music has a header, one or more segments, plays from start to end segment (except Set 1, which only plays a single segment.) Tables to look for when editing music:](#PRG028.2)
    -   [PRG028.3) Music Segment Data. This defines the actual notation/command data (again, as described in "Music Format"), and is found in the tables labeled "M12ASegData[xx]" for Set 1 / 2A and "M2BSegData[xx]" for Set2B. Note that due to a single byte offsets specified in the header, total segment data is limited to 256 bytes. Some segment data blocks flow into PRG029.](#PRG028.3)
    -   [PRG028.POI) Points of Interest](#PRG028.POI)
    -   [PRG029) Remainder of music segments and Player animation code. The last few segment data blocks flow into this bank. See PRG028 for more information about those. After that, we get into Player animation and action response. There is also some stuff related to the Toad House and graphics updates for gameplay tile changes.](#PRG029)
    -   [PRG029.1) Player sprite/animation data. "Player_Frame" is the variable which selects the frame to display for the Player.](#PRG029.1)
    -   [PRG029.2) Toad House lottery data.](#PRG029.2)
    -   [PRG029.3) Blockchange: When a block is broken or otherwise changed in gameplay, the update is not automatic. The hreftable must be informed of the change.](#PRG029.3)
    -   [PRG029.POI) Points of Interest](#PRG029.POI)
    -   [PRG030) Primary bank! This bank always sits at $8000-$9FFF and provides a lot of skeleton support for SMB3 and runs all of the gameplay loops.](#PRG030)
    -   [PRG030.1) Tile memory offsets. You should probably NEVER modify these! They exist as critical lookups that define how to get to a particular tile row and column within the gameplay tile grid. It's good to know what they are and how they're important.](#PRG030.1)
    -   [PRG030.2) Video Update Data table (Video_Upd_Table). This table provides pointers to blocks using a standard format which describes how to apply mostly fixed-location, fixed-content updates to the hreftable VRAM. This is used mainly for things like the HUD or messages displayed at the goal. It also has a special entry at index 0 which is in RAM (under the href of "Graphics_Buffer") which allows for dynamic-location, dynamic-content updates. Dynamic content is pretty much defined on-the-fly throughout SMB3 as needed, using the same standard format as these fixed updates. Note that some of these updates are in some other bank and thus require proper context before they'll fire correctly!](#PRG030.2)
    -   [PRG030.3) Tileset data; data that is defined per-tileset. Useful especially if creating a new tileset or at least modifying an existing one.](#PRG030.3)
    -   [PRG030.4) Miscellaneous data / subroutines](#PRG030.4)
    -   [PRG030.4) Map data (not contained in the other map banks)](#PRG030.4)
    -   [PRG030.5) Game loops; depending on state...](#PRG030.5)
    -   [PRG030.POI) Points of Interest](#PRG030.POI)
    -   [PRG031) Primary bank, interrupt handler! This bank always sits at $E000-$FFFF and mainly provides the interrupt vector/handlers for SMB3. The bottom of the bank contains the 6502 vector table for NMI (V-Blank) "IntNMI" / Reset "IntReset" / IRQ (MMC3 Scanline) "IntIRQ". The music engine is technically interrupt driven for timing's sake, so it is mostly contained here.](#PRG031)
    -   [PRG031.1) Some DMC sounds and DMC Playback. The bank starts with some of the DMC sounds used in the game. It also contains the bulk of the music engine logic.](#PRG031.1)
    -   [PRG031.2) Music playback. A lot of the common music engine exists here.](#PRG031.2)
    -   [PRG031.3) Miscellaneous Data / Subroutines](#PRG031.3)
    -   [PRG031.POI) Points of Interest](#PRG031.POI)
    -   [NoDice (What a drag)](#NoDice)
    -   [NoDice Level (Not World Map) Editing](#NoDice2)
    -   [Level Properties (Edit -> Level Properties)](#Level)
    -   ["Gens" Generator Editing Mode](#GensEdit)
    -   ["Objs" Sprite-based Object Editing Mode](#ObjsEdit)
    -   ["Starts" Level Junction Start Spot / Style Editing Mode](#StartsEdit)
    -   [NoDice World Map (Not Level) Editing](#NoDice)
    -   ["Tiles" Tile Grid Editing Mode](#TilesMap)
    -   ["Objs" Sprite-based Object Editing Mode](#ObjsMap)
    -   ["Links" Tile -> Level Linkage Editing Mode](#LinksMap)
    -   [NoDice Configuration](#NoDiceC)
    -   [Boot Configuration: config.xml](#Boot)
    -   [Game Configuration: game.xml](#Game)
    -   [Property Box Option Blocks](#Property)
    -   [<levelheader> Level Header](#levelheader)
    -   [<jctheader> Junction parameter header](#jctheader)
    -   [<objects> Sprite-based Objects, except World Map](#objects)
    -   [<maptiles> Special Map tiles](#maptiles)

Some Questions to Get Out of the Way
------------------------------------

* * * * *

-   Will you be doing any other disassemblies?Doubt it. This took a very long time to do and was mostly done with pure heart because SMB3 had such a dramatic effect on my childhood and future educational interest. I will PROBABLY never do a full disassembly again. But I may be able to provide assistance for others trying to perform a disassembly in a similar manner to what I did for SMB3.

-   How do I even get started with this??I had to do the hard part; I started from the software entry point in ROM and worked onward from there. But you guys have a sort of "index" -- if you're looking for something, try finding a related variable in SMB3.ASM and start searching through files! Undoubtedly you will come across at least a trace of what you're looking for. For example, want to change the World 3 map bridge toggling? Try searching for "World3_Bridge" and you should be able to find where the toggle occurs...

-   A comment like "Something removed here" followed by typically two or more NOPs?This probably has something to do with the assemblers and/or programming style used by Nintendo "back in the day." Whether the assemblers were unable to easily deal with code/data being relocated or Nintendo programmers often used the bad practice of hard-coded addresses, I'm not sure. What you have to remember is, if you want to remove (or add) some amount of code, it's going to cause a cascade through the bank that shifts all of the following addresses. This could cause a problem if you remove code from "Bank A" that shifts a lookup table that "Bank B" is relying on by a hard-coded address. Now, this is mostly not an issue in my disassembly because every address is mapped to a label, so the code is easily relocated if you add/remove something. But Nintendo programmers may not have had this blessing; quite possibly their assembler only assembled a single bank at a time and to reference between them meant manually coding in a fixed label or address themselves.

If anyone has any insight into Nintendo's tools they could likely answer this question better... in any case, rest assured, if you see a series of NOPs inline with regular code, it most likely was a removal of a handful of instructions for whatever reason, possibly to fix a bug or change behavior. I would assume that a major change would require retooling of the source, but if it's just a couple instructions, it's a lot easier to "NOP out" the instructions and leave the code aligned the way it was before.

-   What do these markers mean: "Rest of ROM bank was empty" / "BEGIN/END UNUSED SPACE"?You'll find the "Rest of ROM bank was empty" label at the bottom of some of the PRG banks which indicates that the remainder of the bank was filled with repeating $FF values. This is a typical flash ROM "unwritten" value and is also what the assembler assumes occupies the rest of the bank in the absence of any code/data following the last line of code/data. Rather than leave the filler in there, I delete it, and leave this marking. Some banks actually did use every last byte, but that's not too common. Removing the filler also enables easier expansion.

However, "BEGIN UNUSED SPACE" is unused space left at a fragmented location that I can't simply remove (as it would shift the code/data following it), but it is most likely safe to remove in your own projects. (EXCEPT FOR DMC SAMPLES PRESENT IN PRG010 AND PRG031!) I left it in as a matter of reproduction integrity (this assembly should form a byte-for-byte image of Super Mario Bros. 3 USA.)

NOTE: PRG010 and PRG031 contain DMC data; the DMC address register is very limited in precision and requires specific alignment within the banks. In PRG010, some DMC samples are present and are aligned to start at $D800. The alignment must appear at an address divisible by $40. It doesn't have to be at $D800 (it could be as early as $D580 in that bank actually), but, again, that's for the integrity I left it there.

A brief overview of the MMC3 and the PRGxyz madness
---------------------------------------------------

* * * * *

The original NES was only designed to address 32KB of program ROM, 8KB of 8x8 character graphics tiles, and 2KB of RAM. The original Super Mario Bros. ran entirely within these limits. To support a larger, more complex game like Super Mario Bros. 3, a creative solution was needed. Obviously the original NES could never change after release, so instead the trick needed to lie within the cartridge. The "magic" came out in chips called "mappers" that provided access to extended memory. The mapper used in SMB3 is known as MMC3. There are some variants to the MMC3 and configuration options available, but since SMB3 uses one single consistent configuration, we will talk about it only as it applies to SMB3.

Under SMB3, the MMC3 provides the following:

-   Additional RAM at $6000-$7FFF, well employed by the game

-   Scanline interrupts

-   The 32KB of ROM space at $8000-$FFFF is now broken up into quarters, two of which are switchable at any time. The switchable chunks are "banks" and in SMB3 range from PRG000-PRG031 (32 banks of 8KB each.)

    $8000-$9FFF remains constant containing PRG030

    $A000-$BFFF is switchable to any of the 32 banks:\
    Set PAGE_A000 and call PRGROM_Change_A000 (just this bank) or PRGROM_Change_Both (to also set the bank at $C000-$DFFF)

    $C000-$DFFF is switchable to any of the 32 banks\
    Set PAGE_C000 and call PRGROM_Change_C000 (just this bank) or PRGROM_Change_Both (to also set the bank at $A000-$BFFF)

    $E000-$FFFF remains constant containing PRG031

-   The 8KB of character graphics is now switchable (typically) among 128 1KB banks:

    The "background" graphics are switchable in halves (first 128 tiles / 2KB and last 128 tiles / 2KB)
    Set the first and second bytes of the array "PatTable_BankSel"\
    NOTE: Loads first specific bank and following bank to make up the 2KB

    The "sprite" graphics are switchable in quarters (64 tiles / 1KB each)
    Set third through sixth bytes of the array "PatTable_BankSel"

No direct programming of the MMC3 is ever intended from SMB3 code except in the regularly called interrupt handlers. Only use the above variables and the magic will be performed behind the scenes. For a full list of all variables, consult the root file "smb3.asm"

Fixed point and 16-bit coordinates
----------------------------------

* * * * *

I won't go into a full discussion of fixed point here, but basically the concept is that given an integer-only CPU like the 6502, it is extremely costly and inefficient for a game engine to implement full floating point math. The solution is "fixed point" math, which works on the following principle shown in base 10:

	1.6 + 2.5 = 4.1		<-- simple floating point op
	 16 +  25 =  41		<-- using a shifted decimal point, we get the same spirit of the values, but remain as integers.\
As long as there's an understanding of what the value "means" this works very well. Obviously you run into a problem of accuracy, but it's better than not having a decimal type value at all.

In SMB3, the common fixed point is "4.4", i.e. the upper 4 bits of the value consist of the "whole" part, and the lower 4 bits of the value are the "fractional" part. The most common use of this are the horizontal "X" velocity and the vertical "Y" velocity of the Player, game objects, etc. The value of $48 essentially equates to 4.5 (the upper 4 bits form the "whole" part of 4, the lower 4 bits form the "fractional" part of 8, where in the range of values being $0-$F, 8 is halfway, thus 0.5.) Essentially this means that the object will move at 4.5 pixels per frame. (Although visually there is no such thing as a "half pixel", but the maintained value knows where it's at.)

PRGxyz_abcd labels?
-------------------

* * * * *

Since assembler either progresses linearly or jumps, every jump needs a label, and that means we end up with a lot of arbitrary labels that just exist for essentially IF-THEN structure. I could have made descriptive names for each and every one, but I chose to label important entry points only and leave general flow control labels as generic. The formation of the name is program bank followed by the original address that the label pointed to.

Note that you don't need to worry about the code continuing to actually remain at that address; the label name is arbitrary. Therefore, although "PRG000_C43C" originally was at address $C43C in program bank 000, that doesn't mean that inserting or removing code before it, and thus disturbing the original address, will break anything. It will continue to assemble and the new effective address of the label will be calculated by the assembler. The assembler doesn't actually care if the label is named PRG000_C43C or MyCustomNameHere, it's just a relic from my pass through.

Consistency
-----------

* * * * *

While it's not to say there's a great lack of consistency (really, there is a lot), there's instances of inconsistent programming.

For example, how do you check for an object being horizontally flipped? Do you shift it to the left twice and check the carry flag? Do you mask the attribute bits looking for $40? Do you use the BIT instruction and check for the overflow flag? I've seen it all! Sometimes whether or not you're trying to spare the accumulator comes into play, but a lot of times I think it was just a different programmer on a different day.

Use zero page special instructions or absolute addresses? In various spots, I found LDA/STA $00xx (i.e. a zero page address using absolute notation) type instructions, which amount to a full 3-byte absolute address load/store. This is slower and consumes more space than using an optimized LDA/STA <$xx instruction for zero page locations. There doesn't seem to be any reason to use the less efficient instruction in these cases, I can only guess that it was a mistake in syntax with whatever assembler the original programmers used all those years ago. Even in the assembler I use here, you must denote the address with the '<' symbol to indicate you want the zero page version rather than a full absolute address.

There's also quite a bit of variable misuse, such as in PRG002 an instance of using what should be the space for an object's detection bits as an internal state variable! I tried to name variables as specific as possible where able, but sometimes these old programmers just get the better of me...

But what about my own part here? This was an on again, off again project over [X] years, so as I began to figure out the engine, a lot of my comments are likely to be clearer as time progressed. Also my style may have altered here and there, depending on my depth of understanding and what area of code I was working in. For example, I don't really care much at all about how the random number generator algorithm works, so I really didn't bother to figure it out, and the lack of comment reflects that. But I always strived to make sure that no absolute labels remained so that modifications to the code could be carried out with relative ease.

Jump (technically always)' to ...
---------------------------------

* * * * *

Whenever someone clearly deliberately set up a situation that would perform effectively "relative branch always" (which does not exist in 6502), for example:

	LDA #$05
	BNE Label	<--- Branch is always taken because the constant "$05" is always "not zero"

The main reason you'd want to do this is to save a byte; a "JMP xxxx" instruction requires 3 bytes (opcode and 16-bit address) while a "Byy xx" instruction only requires 2 bytes, sparing a byte which might add up if there are several instances in a bank. Note this only makes sense if the condition was set up beforehand; if the logic flowing to the jump-always doesn't naturally provide a fixed condition, the JMP instruction makes more sense. Also, the branch instructions are limited to a -128 to +127 range of jumping, so a JMP might be necessary anyway.

Lost Bonus Games
----------------

* * * * *

SMB3 never had a lot for bonus minigames; there were only two, the Roulette ("Spade") game, found occasionally on the world map, and the Card ("N-Spade") game, awarded every 80,000 points. For the most part, SMB3 does not contain a lot of "unused" content; there are 16 known underdeveloped levels and even a couple unused enemies. But the bonus games are one area that seemed like they never got a lot of attention; and, interestingly, it appears there was to be more diversity, some of which remains. It is also known that there were additional bonus hosts and bonus games, but other than face value, it's unclear what they really were supposed to be. Now, with this disassembly, SOME of that should be cleared up.

There are several unused text strings in the PRG022 bonus game bank. These are mostly untranslated Japanese, and some aren't even complete instructions, which shows that a lot of the bonus game development was simply halted at some point, probably due to lack of time. (After all, the main gameplay was much more important than bonus minigames.) These text strings are some clues as to what was in mind, and leftover code provides further clues.

Through clues in the text, there was possibly different workflows to get in to bonus games at all. Quite possibly there was going to be a "rolling die" which decided everything. What we currently know as "Spade" and "N-Spade" were Roulette and Card, sub-games of the die, not necessarily meant to be standalone. A lot of the theory of the lost game function is based on the respective sections found at [The Cutting Room Floor](http://tcrf.net/Super_Mario_Bros._3#Unfinished_bonus_games) and [The Mushroom Kingdom](http://themushroomkingdom.net/smb3_lost.shtml#games), along with what was found by doing the disassembly. The following simple names for the bonus games are what I use. Let's talk about the Rolling Die first...

### Rolling Die

![Taken from TMK](https://sonicepoch.com/sm3mix/smb3_bonus2.gif)

The Rolling Die is the most significant leftover piece of the puzzle. It appeared there would be varities of Rolling Die games to play, with different awards depending on your luck. An Internet user named BMF54123 created a patch for SMB3-J which "fixes" the die to a simple six-sided die display, because otherwise the graphics have been lost. Using the existing translations of the unused text, we gather:

**The Key/Coin Game**

	"If 1 appears, 1 (?)
	If 2 appears, I'll give you a key
	Otherwise, I'll give you coins."\
**The Odd Game**

	"If an odd number appears, I'll let you play the Roulette Game."\
**The Even Game**

	"If an even number appears, I'll let you play the Card Game."

So there were three different types of Rolling Die games, and since the game type is specified by the property set on the bonus panel on the world map, we can infer that these would be selected there, possibly even other panels besides the "Spade" existed (or it would have been a more generic symbol and you just found out the bonus type on arrival.) In the case of Odd/Even, we can see that by the roll of the die that we would transition into "Roulette" or "Card"; these are better known as the "Spade" and "N-Spade" games, and will prove that later by examining code in PRG022.

In the case of "Key/Coin", this game is definitely interesting for each of the prize possibilities:

1.  The unspecified "1"; the sentence is just incomplete, there is no hint as to what "1" was ever going to be, or even planned to be. As it stands, it gives you an extra life, but not in a "safe" way (no 99 check) and no 1-up chime. It could POSSIBLY have meant to be "you'll get a 1-up", but not for certain.

3.  The case of getting a "key" is interesting; this suggests a lost inventory item, although there is no hint or a "key" anywhere ... or is there? One thought is that the "locks" on the map currently busted by defeating Boom Boom were once things that the Player unlocked with one of these "keys." However, this may have been seen as redundant (or even replaced by) breaking boulders on the world map with the hammer. Of note, the Mario Adventure hack for SMB3 employed the idea of key -> lock, although this was just a graphics hack of hammer -> boulder.

### Alternate Hosts / Koopa Troopa's Prize Game

![Credit to TMK](https://sonicepoch.com/sm3mix/smb3_lost_hosts.gif)\
(Credit to [TMK](http://themushroomkingdom.net) for this image)

Another thing that is now well known is that originally there were to be two additional "hosts" instead of just Toad, probably meant for their own respective games of some sort. These hosts were a Koopa Troopa and a Hammer Bro. The reason for villains playing a game with you seems contrary, and unfortunately we'll never have a sure reason as to why. The Koopa Troopa and Hammer Bro were probably unmaintained, made clear when attempting to use them results in their display being distorted; however this is a simple, one-byte fix in PRG022 in the routines "HostTroopa_DrawSprites" and "HostHammerBro_DrawSprites". See those routines for a comment describing the fix.

For the most part, the chosen host has no effect; you can have the Roulette hosted by Toad, Koopa Troopa, or Hammer Bro just the same. However, there is an "alternate" value that gets you Koopa Troopa plus a large [?] box. From that box rising "something"; BMF54123's patch suggested a Mushroom, Flower, Star, or Judgem's Cloud as a "prize", hence my naming.

The main problem is with "Koopa Troopa's Prize Game" is that it is uncertain whether there is any text to describe what the game rules would have been. Since the host setting selects whether we have the prize box, rather than the game type, I cannot definitively link a particular text string of instruction to the game. Also, the "prize" emerges immediately from the box at start; MOST LIKELY that was not the intention.

### Workflow for Bonus Games

A lot of clues emerge when we actually look at the code for bonus games to see how they execute, so let's start at the beginning. The "Spade" panel on the map configures the host and game type. Only a couple types are actually "valid" (Spade / N-Spade); the rest go unused and may lead to leftovers. We will focus on the type and ignore the host, since the host does not actually change the game played.\
The game type (Bonus_GameType) specifies the initial instruction which gets displayed. There are 8 game types defined, and their associated instructions (US release version, in English) are as follows (pointed to by BonusText_HostGreetPtrL/H in PRG022.) "[J]" indicates this is a translated, unused Japanese string.

	Game Type 0 (BONUS_UNUSED_KEYCOIN): [J] If "1" appears, 1 (?) / If "2" appears, I'll give you a key / Otherwise, I'll give you coins.
	Game Type 1 (BONUS_SPADE): "Line up the pictures and" / "get a prize!" / "You only get one try."
	Game Type 2 (BONUS_NSPADE): "Flip over any two cards" / "and see if they match." / "You can only miss twice!"
	Game Type 3 (BONUS_UNUSED_CCCC): "CCCCCCC" / "CCCCCCC"
	Game Type 4 (BONUS_UNUSED_DDDD): "DDDDDDD"
	Game Type 5 (BONUS_UNUSED_ODDROULETTE): [J] "If an odd number appears, I'll let you play the Roulette Game."
	Game Type 6 (BONUS_UNUSED_EVENCARD): [J] "If an even number appears, I'll let you play the Card Game."
	Game Type 7 (BONUS_UNUSED_2RETURN): [J] "2, return (?)"\
So after receiving instruction, the game logic will, in general, run the rolling die sequence. However, in game types 1 (Spade) and 2 (N-Spade), it deliberately hides the die. We'll revisit this in a bit.

There's a second piece to the bonus game system that is almost completely unused, except in a hacked fashion; this is what I call "Round 2", which was a transition for the Rolling Die game to go from a successful roll to a result message (determined by Bonus_Round2). Let's look at the "Round 2" messages (pointed to by BonusText_Round2PtrL/H in PRG022):

	Round 2 Type 0: [J] "Give something?"  (Google translates as "I'll give you something?")
	Round 2 Type 1: [J] "Play three times!"
	Round 2 Type 2: [J] "Chance to twice" / "Set aside two identical cards" (NOTE!  This is apparently the Card/N-Spade instructions!)
	Round 2 Type 3: "CCCC CCCC"
	Round 2 Type 4: "DDDD"
	Round 2 Type 5: [J] "Play three times!" (NOTE! Repeated from type 1)
	Round 2 Type 6: [J] "Chance to twice" / "Set aside two identical cards" (NOTE! Repeated from type 2)
	Round 2 Type 7: [J] "2, return (?)"\
Although the different rounds are controlled by two different variables, there's seems to be a connection between the Game Type text and the Round 2 text on the same index. For example, consider Game Type 0, which awards different prizes based on the die roll. The "Round 2" text is "Give something?", which could be an incomplete sentence or a placeholder, but nonetheless, shows a flow of action; you rolled the die, and you've been "given something."

Now look at Game Type 6 text; supposedly, if the Player were to roll an even number, they get to play the "Card" game; if you look at the Round 2 Type 6 text, we see mangled instructions that are essentially N-Spade; you get two chances (at failing) to find two identical cards.

So if we assume there's a flow as such, look at the repeat of Round 2 Type 2 and Round 2 Type 6. Round 2 Type 2 text is the mangled N-Spade instruction, and Game Type 2 is the actual English N-Spade instruction. This suggest that game types 5 and 6 were moved down to types 1 and 2 at some point, and further that 1 and 2 were hacked to bypass the rolling die portion of the game.

The bonus games are run by a state variable (Bonus_GameState) and the jump table with comments is shown below (found at PRG022_C8A0):

	.word Bonus_Init	; 0: Draw the dialog box, initialize the greeting text,
	.word Bonus_DoHostText	; 1: Giving instructions for ALL UNUSED GAMES (then initializing the "prize"!)
	.word Bonus_DieRotate	; 2: Rotating die logic; press 'A' and you may get a prize by the game type (or so it was intended)
	.word Bonus_GetDiePrize	; 3: Get your die prize!  (If you "won") Includes the "coin confetti"
	.word Bonus_Wait80	; 4: Waits $80 ticks
	.word Bonus_DieFlyAway	; 5: Die "flies away"; go to state 6 if you won the Odd/Even game and to state 8 otherwise
	.word Bonus_InitRound2	; 6: Initialize for Round 2
	.word Bonus_DoHostText	; 7: Giving instructions for Spade/N-Spade (LEGACY: For "Round 2" game)
	.word Bonus_WaitA0	; 8: Wait $A0 ticks
	.word Bonus_KTPrizeGame	; 9: Appears to be all that was implemented toward "winning" the Koopa Troopa "Prize" Game\
In final SMB3, entering a Spade or N-Spade game executes this bit of hack code at "PRG022_C8E6"; the Die is deliberately set to an off-screen Y coordinate and the bonus game state jumps to 7:

	; BonusDie_Y = $F8 (hide the die)
	LDA #$f8
	STA <BonusDie_Y

	; Bonus_GameState = 7 (give instructions to Spade/N-Spade game)
	LDA #$07
	STA Bonus_GameState\
... and so the die is never shown (even though it is still being drawn) and all of the logic for the die is bypassed.

To complete the workflow discussion, let's look at what happens if we're playing the "Even" game (where an "even" roll would net you a chance at playing the "Card"/N-Spade game.)

Proving that Game Type 1 and 2 were intended to be the even/odd rolling die game, examine "Bonus_DieRotate", the function which spins the die 1 thru 6 on loop until the Player presses 'A'. After having pressed 'A', there is check logic that is unreachable by final SMB3 (because this ENTIRE STATE [1] IS ALWAYS SKIPPED [to 7]):

	LDY #$00	 ; Y = 0 (Odd die face value will "win")

	LDA Bonus_GameType
	CMP #BONUS_SPADE	; <-- should be replaced by BONUS_UNUSED_ODDROULETTE
	BEQ PRG022_CC34	 ; If Bonus_GameType = BONUS_SPADE, jump to PRG022_CC34

	INY		 ; Y = 1 (Even die face value will "win")

	CMP #BONUS_NSPADE	; <-- should be replaced by BONUS_UNUSED_EVENCARD
	BNE PRG022_CC4C	 ; If Bonus_GameType <> BONUS_NSPADE, jump to PRG022_CC4C\
Obviously if Spade and N-Spade were always meant to be directly accessed as they are now, there would be no reason to be checking for die results! But using the same Game Type values as Spade and N-Spade, even/odd winning checks are made! (Not shown here, but certainly in source for PRG022.) The point is, what is now known as Game Type 1 Spade used to be Game Type 1 Odd, and Game Type 2 N-Spade used to be Game Type 2 Even.

As when any die game ends (including if you lose the odd/even roll), the game pauses for a moment and then die will fly away (bonus game state 4 and 5.) In Game Type 0, you get your prize as appropriate, and the die flies away. In the Odd/Even case, if you won the roll, we will continue into bonus game state 6... which initializes Round 2. What's most interesting is a variable named Bonus_DieCnt, which after the die portion ends becomes a tracking value, is set to 5 or 6 if won the Odd or Even game, respectively.

And what are Round 2 Types 5 and 6? Duplicates of Round 2 Types 1 and 2, of course! This further suggests that the overall Game Types 1 and 2 were once something else, or at the very least placeholders. Likely as the bonus game system was being trimmed, the moved types 5 and 6 down to 1 and 2, but since it was ultimately decided that the entire rolling die would get dropped, this code fell into disuse and was never updated, hence why it still uses 5 and 6.

See this code snippet from the end of the die rotation (after Player has pushed 'A'):

	LDA Bonus_DieCnt
	BEQ PRG022_CD4B		; If Bonus_DieCnt = 0 (not a winner of the Odd/Even game), jump to PRG022_CD4B

	; Winners of Odd/Even Game only!
	INC Bonus_GameState	; Bonus_GameState = 6

	; NOTE: Bonus_Round2 (i.e. what game we're going to next) was set by the intro dialog text!

	RTS		 ; Return

PRG022_CD4B:

	; Bonus_GameState = 8
	LDA #$08
	STA Bonus_GameState

PRG022_CD50:
	RTS		 ; Return\
The main purpose of this code is, in all cases of the die EXCEPT where Bonus_DieCnt is non-zero, jump to bonus game state 8, which waits a bit and exits to the map. If Bonus_DieCnt is non-zero, go to state 6 instead, which proceeds into Round 2. How do we know what Round 2 game to go into? Interestingly, Round 2 is set by the end of the instruction text string!

Assuming you won the Odd/Even game, we enter into Round 2. The first thing the game does (in "Bonus_InitRound2") is clear out the previous instructional text and load up the Round 2 text that will be displayed. Then the next bonus game state is, again, to display instructional text; in this case, it would be the Round 2 text! (This is where having Bonus_Round2 not been set properly would be apparent.) After the text ends, the bonus loop is bailed, and PRG030 takes over.

So, in summary, here are the workflows as can be determined by looking at code.

Game Type 0:
	Round 1: Rolling die, hit 'A'; get a prize based on roll
	Round 2: N/A

Game Type 1 (Apparent former Type 5):
	Release SMB3: Round 1 is bypassed; jumps directly to Round 2
	Round 1: Rolling die, hit 'A'; if you roll an odd number, go on to round 2
	Round 2: You get to play the Roulette (Spade) game (originally 3 times perhaps?  See "Other Points of Interest" below.)

Game Type 2 (Apparent former Type 6):
	Release SMB3: Round 1 is bypassed; jumps directly to Round 2
	Round 1: Rolling die, hit 'A'; if you roll an even number, go on to round 2
	Round 2: You get to play the Card (N-Spade) game

Game Type 3:
	No clear behavior, will mimic Game Type 0 due to lack of specific code

Game Type 4:
	No clear behavior, will mimic Game Type 0 due to lack of specific code

Game Type 5:
	Same as Game Type 1, although handling code has been rewritten for Game Type 1

Game Type 6:
	Same as Game Type 2, although handling code has been rewritten for Game Type 2

Game Type 7:
	No clear behavior, will mimic Game Type 0 due to lack of specific code

### Other Points of Interest

-   In case you missed it, the text strings include a byte after the terminating byte ($FF); a value which is pushed into Bonus_Round2 to control what game you will transition into (if applicable.) This piece of functionality remains in SMB3 although a code hack for the two remaining bonus games (Roulette/Spade and Card/N-Spade) prevents the workflow from occurring.

-   In Game Type 0, we know you were supposed to get one of three prizes; an unknown prize simply referred to as "1", a key, or coins. The actual results of the game are as follows:

1.  "1" currently blindly increments the Player's first "card" slot; so the first card would go Nothing -> Mushroom -> Flower -> Star -> (glitch.) This is probably placeholder code and not really the intention. Although it is possible that cards were going to be something you gambled for in this manner instead of (or in addition to) being an item at the end of a level.

3.  The "key" currently just gives you an extra life, but does not cap at 99 like it should nor does it play the 1-up sound. Again, probably placeholder code. So whatever the "key" would have been is lost.

5.  Otherwise, coins -- whatever you rolled on the die, 3 to 6, causes that number of "coins" to erupt in a confetti like manner. But what you actually get are that x10 points on your score; so you are awarded with 30 to 60 points. Once more, I'm sure this is not the intention, especially since it doesn't make much sense nor is it even all that valuable a "prize", compounded by the fact that "coins" sprout out. Conversely, of course, a meager 3 to 6 coins isn't much of an award either. Finally, as described before, coins on the world map as SMB3 stands are pretty much worthless. So it is likely that either this game was accessed through gameplay somehow or coins were once accumulated on the map for some reason, e.g. gambling or buying items (instead of them being given away.)\
-   In Round 2 Type 5 text, we find the message "Play three times!"; what's interesting are the code artifacts which suggest that this WAS the Roulette entry point (and this Round 2 being after you successfully rolled an odd number.) But "play three times" doesn't make sense for the Roulette... unless you realize there is actually a variable Roulette_Turns which, when greater than zero, gives you additional spins if you should lose the Roulette game! So the Roulette game was actually probably supposed to be played in a manner where you had up to three chances to make a match.\
-   In "Bonus_DoHostText", the game state routine used to draw out and blip the dialog text, after it has completed the text, it initializes the X and Y position of the "prize" for Koopa Troopa's prize game. Without the Koopa Troopa host, however, this assignment is meaningless since only that host actually draws/handles the "prize."\
-   When exiting the bonus loop back to PRG030, a couple of the very undefined bonus games actually have special handling code that seems to indicate at least a little bit of development toward what they would be.

1.  Game Type 4 (BONUS_UNUSED_DDDD): Sets an otherwise unused variable Bonus_DDDD to 1.

3.  Game Type 7 (BONUS_UNUSED_2RETURN): Marks Player as having DIED (calls Bonus_Return2_SetMapPos in PRG022) and will cause the Player to return to one of four possible hardcoded spots on the map based on the value used in the Koopa Troopa Prize game. This doesn't necessarily mean they were related, but I can't really say one way or the other since there's no text that can be verified as part of the Koopa Troopa game. But still... four "prizes", four locations on the map? Maybe he was in charge of sending you to some spot on the map... but the reason why is lost to time.

-   An unused block of code appears to be the only thing coded towards Koopa Troopa's Prize game; this is found at the label "Bonus_KTPrizeGame." The reason for this relationship is twofold:

1.  It actually cares about which host is running the game! If not the Koopa Troopa w/ Box host, it bypasses the second check

3.  The second check cares about what height the "powerup prize" is at! It must be fully raised.\
If both of the above pass, a subroutine is called which I aptly named Bonus_DoNothing because... unfortunately... it only contains an RTS, thus does nothing! Perhaps this would lead to getting the "prize" or whatever the result would have been from that game.

Tiles, Tile Quadrants, Tilesets
-------------------------------

* * * * *

As many know or at least have guessed, the map and gameplay levels are broken up into a grid of reusable 16x16 tiles. These tiles are internally numbered so they can be referenced for different things (or not at all if they're purely scenery.)

Instead of having a large 256-byte lookup table or a convoluted set of logic to determine which tiles are what, Nintendo programmers instead opted to break the tiles into "quadrants" (my term), i.e. tiles $00-$3F are quadrant 0, $40-$7F are quadrant 1, $80-$BF are quadrant 2, and $C0-$FF are quadrant 3. The quadrant also determines what palette is used to display the tile. Quadrant 2 tiles always display in BG palette 2, for example.

The Tile_AttrTable array contains 8 values, which are two sets of 4 quadrant information bytes. On the world map, the first and second set are redundant, and they specify which tile index, per quadrant, begins the range of tiles that are potentially "enterable" (as in, you press the A button when standing over this tile to enter a level area.) Some additional code is used to specifically filter it, but this eliminates some unnecessary checking. In gameplay levels, both sets determine some type of "solidity" beginning at the tile specified for the given quadrant. The first four values specify tiles that are solid only on the top. The second four specify tiles that are solid on the sides and bottom. An overlap makes a tile solid all around. No tiles exist that are only solid on the sides and bottom, and I'm not sure where that would be useful. Setting up the ranges that way would also prohibit the possibility of "solid on top only" tiles which are fairly common.

A "tileset" is a set of 16x16 tiles that work with a particular theme. Like the overworld levels that are generally all flat with bushes in the background I call "Plains style". The tiles in this "tileset" support generating the flat ground, the rectangle blocks, the bushes, etc. Another example of a "tileset" is "Underground" style; obviously if you've ever played the game you know this style. The "underground" tileset has some unique tiles to it, including some that exist for the "above ground" and "below ground" sections. The most important feature of tilesets is that they define what a "tile" index number translates to in terms of 8x8 patterns. Tilesets generally have their own program bank except those that share similiar properties. The active tileset may also be used to control certain game logic (e.g. to enable slope handling or not.) Each tileset also dictates level structural routines that are used for it, and further all levels of a paritcular tileset are together in the same bank as the rest of the tileset data. The full list of tileset styles is as follows:

	Tileset  0: Used for World Map
	Tileset  1: "Plains" style (like 1-1, 1-3, etc.)
	Tileset  2: Fortress / Bowser castle style
	Tileset  3: "Hills" style (like 1-2, etc.) (Shares a lot of functionality with Tileset 14)
	Tileset  4: High-Up style (like 1-4, 1-6, etc.) (bank shared with Tileset 12)
	Tileset  5: Only used for the two "Plant infestation" levels of World 7 (the "hammer bro battle" replacement levels) (bank shared with Tilesets 11 and 13)
	Tileset  6: Underwater levels (bank shared with Tilesets 7 and 8)
	Tileset  7: Only used for Toad houses (bank shared with Tilesets 6 and 8)
	Tileset  8: Vertical levels (like 7-1 etc.) with support for the "pipe maze" mechanics (bank shared with Tilesets 6 and 7)
	Tileset  9: Desert areas
	Tileset 10: Airships / Coin Ships
	Tileset 11: Giant World levels (bank shared with Tilesets 5 and 13)
	Tileset 12: Frozen Tundra levels (bank shared with Tileset 4)
	Tileset 13: Sky areas (like Coin Heaven, 5-4, etc.)  (bank shared with Tilesets 5 and 11)
	Tileset 14: Underground style (Shares a lot of functionality with Tileset 3)
	Tileset 15: Bonus Game Host Introduction and Directions (bank shared with Tilesets 16 and 17)
	Tileset 16: Spade Bonus game (bank shared with Tilesets 15 and 17)
	Tileset 17: N-Spade Bonus game (bank shared with Tilesets 15 and 16)
	Tileset 18: 2P "Mario Bros." Battle

All banks that represent tilesets have a few common features:

1.  Tile Layout: A lookup table, always named in the form of Tile_Layout_TS[xx], where [xx] is at least one tileset. Banks which host multiple tilesets will generally list them all in line, e.g. Tile_Layout_TS4_TS12. (See above list for tilesets that share banks.) The Tile Layout LUT always defines all 256 possible 16x16 tiles (even if a particular tile index is not used at all) by pattern quarter, i.e. the first 256 bytes are the "upper left" pattern, then the "lower left", then the "upper right", then the "lower right." This LUT is linked by PRG030 in the "TileLayout_ByTileset" table.

3.  Tile Attribute: Not referring to the name table attribute (that's fixed by a rule as described above), this defines a range by "quadrant" where, within that quadrant, tiles are to start being recognized as solid in some fashion. (See above for details.) This table is always named in the form Tile_Attributes_TS[xx], same rules as Tile Layout. This LUT is linked by PRG030 in the "TileAttribute_ByTileset" table.

5.  Level Load entry point: In every tileset bank EXCEPT World Map (which does not use the standard "generator" format), there is an entry point that kicks off level loading, doing whatever a tileset needs to do first (typically some kind of background clear) named LevelLoad_TS[xx]. The "[xx]" MAY follow the rules of Tile Layout, but sometimes there is a unique start point. This LUT is linked by PRG030 in the "LevelLoad_ByTileset" table.

7.  Level Load Variable Sized Generators: In every tileset bank EXCEPT World Map (which does not use the standard "generator" format), variable size "generators" are launched via a lookup table with the standard name of LoadLevel_Generator_TS[xx], same rules as Tile Layout. These are the tile generators which can accept parameters to vary the size of what they construct. PRG014 has common/shared generators, tileset-specific generators will follow this LUT. This LUT is linked by PRG030 in the "LeveLoad_Generators" table.

9.  Level Load Fixed Sized Generators: In every tileset bank EXCEPT World Map (which does not use the standard "generator" format), fixed size "generators" are launched via a lookup table with the standard name of LeveLoad_FixedSizeGen_TS[xx], same rules as Tile Layout. These are the tile generators which do NOT accept parameters to vary the size of what they construct; they do whatever they do, often with a fixed output size (e.g. one tile, the entire background, etc.) PRG014 has common/shared generators, tileset-specific generators will follow this LUT. This LUT is linked by PRG030 in the "LeveLoad_FixedSizeGens" table.

11. Whatever levels this tileset provides. In every tileset bank EXCEPT World Map (which does not use the standard "generator" format), any grid of tiles is stored in "generator" format and generically referred to as a "level", even if there is no gameplay involved. This includes the bonus game intro room and 2P Vs, both of which are constructed by generators. The label names of the levels will vary, although I generally named them after their world location, e.g. W101L is the generator tile layout data for World 1 Level 1. W101_BonusL would be the "Giant 3" made of coins bonus room.

DynJump (Finite State Machine, indexed jump table)
--------------------------------------------------

* * * * *

"DynJump" is a special, commonly-used subroutine in SMB3 that enables jumping to a predefined set of addresses by an index. This is very useful when there's functionality based on an internal state counter of some kind. It is similar to a SWITCH/SELECT-CASE type construct in high level programming languages. The usage is as follows:

	LDA (some kind of index value starting at zero)
	JSR DynJump

	.word Label1	; 0
	.word Label2	; 1
	.word Label3	; 2
	... etc...\
The index value in the accumulator is used to select which jump address will be called with the "JSR DynJump." The way "DynJump" is written, the end result is just as if you had called one of those labels directly, e.g. with the above example, if A = 1, the "JSR DynJump" is equivalent to calling "JMP Label2."

Objects, Special Objects, and Cannon Fires
------------------------------------------

* * * * *

"Objects" typically refer to in-game objects like power-ups, enemies, or some miscellaneous things. Basically, anything not glued to the background that's not a "Special Object." What I call "Special Objects" are a classification of simpler objects like (mostly) projectiles that don't have nearly the overhead of regular Objects. There can also be more of them on-screen simultaneously.

Objects are the bigger, more functional of the bunch, and have the most interactability. You'll learn a lot about them by examining PRG000-PRG005. In-game, there are only up to five "typical" objects in action at any time, an absolute max of eight, but the last three slots are not useful for all objects. Some Object variables are only available to the first five, usually noted in SMB3.ASM by "OBJECT SLOT 0 - 4 ONLY!" Basically anything you see commonly in gameplay sits in the first five slots, and special objects like the "card" at the end of levels is something that will take one of the three upper slots. After all, something like the card "must" show up, so it should have a reserved slot.

Special objects are much simpler and far less interactable. Their most prominent use is projectiles (fireballs, spike balls, cannonballs, etc.) but a few others exist (e.g. the wand recovered from a Koopaling.) Special objects have their existence inside of PRG007, though they occassionaly use some support routines from PRG000. They employ far fewer states and much less overhead, but of course are not nearly as featured as regular objects and can not do as much.

Also in PRG007 is the "Cannon Fire." Cannon fires are another dimunitive subset of an object, even simpler than a "Special Object", mostly used as activators for cannons that fire regularly (i.e. they are the timing and explosion for the cannon, but not the cannonball itself), like on Airship/Tanks, or bullet bill cannons. The Bowser Castle laser is also a Cannon Fire. Cannon Fire objects have a full 16-bit X and Y, and are never destroyed by moving off-screen. However, they are overwritten in circular order, which with their typical use amounts to much the same thing. (That is, as the screen scrolls horizontally, given that all objects are stored in left-to-right order, the current oldest cannon fire object likely having exited left will be overwritten by the next one coming from the right.)

Level Format
------------

* * * * *

The SMB3 "level" format is used for any grid of 16x16 tiles except the World Maps (which just use raw storage.) This includes the "Bonus Intro" of Spade/N-Spade games and the 2P Vs arenas. The level format does not store data for each individual tile like the World Map does; instead, it stores index values into what I call "generators", little internal subroutines that construct level geometry in a predetermined way. The simplest example of a generator is the [?] block containing a Fire Flower; this generator just places this one tile. But then there is a generator for a downward vertical pipe; this one generates the two "lip" tiles at the top of the pipe, followed by a user-specified length of pipe body tiles.

Note that this description does NOT include in-level sprite-based objects; those are a separate, raw list format found in PRG006, which is pretty straightforward.

The former generator that only places a single [?] block is what I call a "fixed size" generator. This does not imply it only places a single block or even necessarily a specific formation of tiles (although that is the typical case), but more that there is no user input tht changes the output. It just generates in whatever manner it is programmed to do.

Conversely, there are dynamic generators, which always have a 4-bit (nibble) value as a "parameter." Some will also take an additional byte for another full 8-bit parameter. Theoretically a dynamic generator could take as many additional bytes as it wants, but stock SMB3 never took more than one if any were taken at all.

The last type of data is "level junction" data, which defines, per screen of the level, where the Player will land on the opposite map. Each "screen" is made up of 16 tiles horizontally in non-vertical levels, or 15 tiles vertically in vertical levels. Levels in SMB3 are "vertical" or "non-vertical", which relates to the general shape of the tile grid and how the level is able to scroll. Non-vertical levels are typical horizontal and usually vertically-capable scrolling playfield levels. They are 15 "screens" wide, each "screen" being 16 tiles horizontally. Vertical levels only scroll vertically and are are 16 "screens" tall, each "screen" being 15 tiles vertically. The latter is used sparingly for levels such as the pipe maze of 7-1 or the first underground portion of 5-2 where you free fall. Note that "tileset 8" (see "Tiles, Tile Quadrants, Tilesets") used for pipe mazes like 7-1 is technically the only tileset in SMB3 that is ever used for vertical levels, but interestingly any "tileset" can be set to vertical. (However, the generators may fail to operate properly in vertical, and in particular, tileset 8 specific generators are not designed to operate non-vertically at all. Of course, that can be fixed with a little elbow grease, but other gameplay fixes will likely be required before we can have a non-vertical pipe maze level.

The level data always appears in the following manner (where xB = 'x' bytes):

[2B Alt Layout]
[2B Alt Objects]
[1B Size / Y-Start]
[1B Palettes / X-Start]
[1B Alt Tileset / Is Vertical / V-Scroll style / Pipe Not Exit]
[1B BG bank / Initial Action]
[1B BGM / time]
[Generators / Junctions...]
[$FF Terminator]

-   [2B Alt Layout] / [2B Alt Objects]: Pointers to the "Alternate" Level's level and object set.\
-   [1B Size / Y-Start]: Selects the level size (in "screens") and sets the Player's Y start position. See LEVEL1_* definitions in SMB3.ASM. Note that in "vertical" levels, the Y starts < $100 will start at the absolute top screen and otherwise start at the absolute bottom screen.\
-   [1B Palettes / X-Start]: Selects the palettes for the tiles and the objects. Sets the Player's X start position. See LEVEL2_* definitions in SMB3.ASM.\
-   [1B Alt Tileset / Is Vertical / V-Scroll style / Pipe Not Exit]: Selects the "Alternate" Level's tileset. And for the current level, whether this level is vertical, how the vertical scroll works (assuming non-vertical), and whether entering a pipe exits the area. The "Pipe Not Exit" (LEVEL3_PIPENOTEXIT) bit is USUALLY set except in "pipe junctions." Note that NOT having this bit set causes pipes to exit the level and makes doorways malfunction. See LEVEL3_* definitions in SMB3.ASM.\
-   [1B BG bank / Initial Action]: Selects the BG bank set (what actually determines what CHRROM graphics to use; see Level_BG_Pages1/2 in PRG030) and what the "initial action" upon entering the level will be. Note that "initial actions" are generally not used except in special levels, e.g. the "pipe" entrance initial actions are intended for pipe junctions only. The only initial action used in an otherwise "normal" level is the "start sliding" action, used only in W1-5. See LEVEL4_* definitions in SMB3.ASM.\
-   [1B BGM / time]: Selects the song to play and starting time on the clock for this level. See LEVEL5_* definitions in SMB3.ASM.\
-   [Generators / Junctions...]\
Generators and junctions follow. The level load loop always retrieves 3 bytes on its own, which go into Temp_Var15, Temp_Var16, and LL_ShapeDef, respectively. The following are listed in logic order as LevelLoad in PRG030 will check for them.

1.  Level Junction

12. Generators (Fixed / Dynamic)

28. Fixed Size Generator

35. Dynamic Generator

[$FF Terminator]: A single $FF byte in the "[Generators / Junctions...]" flow signifies End-of-Level; a completely empty level thus has no generators / junctions, just a $FF byte (but the header is always present.)

Music Format (defined in PRG028 and PRG029)
-------------------------------------------

* * * * *

Music in SMB3 is not stored in a single unit making up the entire score, like a MIDI file would be. It is broken up into "segments", a section of the music that may be reusable. The music is also internally divided into what I referred to as "Set 1", "Set 2A", and "Set 2B", referring to how the music is queued in the game engine.

The NES provides sound hardware of:

-   Square Wave 1
-   Square Wave 2
-   Triangle Wave (as it is pre-programmed, makes the "soft" sounds)
-   Noise Generator (used as one form of percussion)
-   Digital sound; DMC (used as the other, semi-realistic form of percussion)\
The variable Sound_QMusic1 queues Set 1 music, made up of "event" songs (death, time warning, etc.) It is queued by a bit weight ($01, $02, $04, $08, etc.) which allows a simple "priority" system to be used to queue the "most important" "Set 1" song from the set.

The variable Sound_QMusic2 defines Set 2A and Set 2B. Set 2A contains almost all the music used outside of levels and is specified in the lower nibble ($01-$0F.) Set 2B contains all the music used inside of levels and is specified in the upper nibble ($10, $20, $30 ... $C0.)

### SEGMENT HEADER

The first part of understanding the music format is the "segment header", which defines what this particular segment of music is going to play and at what the base set of rest values will be. The header defines the base rest value (always divisible by $10; see PRG031 Music_RestH_LUT for values), the pointer to the segment data, and starting offsets within that data for all tracks except Square Wave 2, which is always assumed to exist and start at offset zero. For the non-Square Wave tracks, an offset of zero specifies that the track is not used in the segment. Both Square Wave tracks cannot be disabled.

The group of headers is defined under the labels Music_Set1_Set2A_Headers (Set 1 and 2A) and Music_Set2B_Headers (Set 2B.) The related Music_Set1_Set2A_IndexOffs and Music_Set2B_IndexOffs define offsets into those labels to get at a particular header. These define headers for all segments of all songs.

### SEGMENT START/END/LOOP

There are additional arrays "Music_Set2A_Starts/Ends/Loops" and "Music_Set2B_Starts/Ends/Loops" which specify which segment index to start on, end on (inclusive), and return to after reaching the end (i.e. would be the same as start for a full loop, or some value ahead to loop somewhere ahead of a song with an "intro.") Note that Set 1 uses implied index values of 0 to 7 based on the trigger value (8 bit weights, thus 8 unique songs) and none of those songs loop. ($80 in particular triggers a special STOP MUSIC request.) Using a loop value of zero is coded to make a song not loop, but I don't think this is used (Set 1 songs are the only ones that don't loop and they don't use the start/end/loop system.)

### TRACK DATA

Each track has basically the same pattern of data, though some unique values. Remember that those other tracks could just be playing "rests" ($7E for non-Noise or $01 for Noise Track only) until they catch up to track 2's expiration. Note that there are no "end track" event bytes, and Square Track 2 officially marks the end of a segment. That means that all enabled tracks are expected to continue playing until Square Track 2 reaches a $00 value. A tip here is that when Square Track 2 dictates that the segment is over, all other tracks are done as well. This means that if a track finishes playing before Square Track 2 finishes, you must pad it with rest events that at least cover it until Square Track 2 is done. But it can continue passed Square Track 2, it will just be cut off. I take advantage of this in the MusConv tool where I can use the longest rest possible repeatedly to cover the minimum required time and overflow is okay. Basically just remember that Square Track 2 is your timing track; it dictates the length of the segment. When it's done, everyone's done. If others finish before it does, pad them out. If others are to continue after Square Track 2, Square Track 2 must be padded to the length required. But in that case, use precise rests because, again, Square Track 2 determines the length of the entire segment!!

| $00 | Square 1: Activates a ramp effect\
Square 2: END OF SEGMENT (Set 2A/B) OR SONG (Set 1) ALL TRACKS\
Triangle: Stop triangle output (otherwise constant)\
Noise: Restart offset (in-segment looping, typically percussive)\
DMC: Same as noise, provides in-segment looping |
| $01-$7D | "Note On" events, usually a note on the typical musical scale MULTIPLIED BY 2 because it was convenient for their frequency lookup table. (Took me a while to figure that out.) Dividing the note value by 2 and adding 36 seems to get it into the MIDI musical scale pretty well!\
NOISE TRACK ONLY: Notes are shifted right (divided by 2) and used in a three parallel lookup tables (Music_NoiseLUTA/B/C) |\
(The shift to the right makes the notes stay as double bytes although in this case it was unnecessary.) In any case, there are only four values and this shift, so that means the only "Note On" values which are valid are $01-$07, with repeats happening every even value (i.e. $02 same as $03, $04 same as $05, etc.) Note: $01 works as a "rest" instead of $7E In short, the noise track is percussive, not musical.\
DMC TRACK ONLY: Similar to noise, this track is intended to be percussive, and the "note" value is fed into lookup tables DMC_MODxxx_LUT; SMB3 defines 16 valid samples, so $01-$10 are the only valid values. |
| $7E | Rest (plays no sound, Note Off) NOTE: Does not work on Noise Track, use $01 instead |
| $80-$FE | Bits 4-7 ON SQUARE WAVE TRACKS ONLY: Store a "patch" (instrument sound, sort of) value into Music_Sq1/2Patch which modifies the output sound of channel\
Bits 0-3 (on all tracks) index a new "rest" value (in ticks) that is added to the current value of: (Music_RestH_Base + Music_RestH_Off.)\
Note that the byte that immediately follows this one is EXPECTED to be $00-$7F; (essentially the timing/patch byte is an atomic value with the following Note On, there will not be a lost cycle for changing patch/timing.)\
 |
| $FF | Square Wave Tracks: If used after a Note On ($01 - $7D), this activates a bend effect. In this case, it is considered atomic with the Note On event, i.e. does not require an additional cycle to be queued.\
Other Tracks: Not recognized as anything special since only the lower 4 bits of $8x-$Fx values are even used |\
Vague description of the Square Patches:

Patch 0: Long square\
Patch 1: Short "piano" (?) World 1 sound\
Patch 2: Short square (more "organ-like")\
Patch 3: Square with quick attack, long decay ("bell-like" perhaps)\
Patch 4: Wavey sound (string-like, water sound)\
Patch 5: Short "piano" with an "echo"\
Patch 6: Same as Patch 1\
Patch 7: Pizzicato string sound ("plucking")

### MUSCONV TOOL OVERVIEW

MusConv exists as a tool to convert MIDI data to SMB3 format and it is included with the disassembly source. BUT PLEASE NOTE!! MUSCONV SHOULD BE CONSIDERED ALPHA SOFTWARE. IT IS A BEAR TO USE AND IT IS VERY TOUCHY ABOUT ITS INPUT AND IS EASILY MESSED UP. If anyone wants to make a version with automatic segment splitting and/or a nice GUI app with real time previews, please feel free. MusConv does exactly what I need it to do so I'm not going to put any more effort into it, and don't ask either. I'll help you use it, but I won't be fixing its deficiencies at this point.

Some folks may not agree, but I find the MIDI file format to be a great container for developing synthesized music, even targeting a limited console like the NES. However, it must be acknowledged that the sound hardware of the NES is extremely limited, and further this document assumes you're trying to use the SMB3 music engine as-is. (If you want to replace it with something else, you may get even better sound, but that's an exercise left to you.)

Some important things to remember when using MIDI to represent a song from SMB3:

1.  We only have the tracks as described above; Square Track 1 and 2, Triangle, Noise, and DMC.
2.  All of those can only play one waveform at a time; no "mixing" is possible
3.  Only Square Track 1 and 2 have "instrument" patches; Triangle always sounds the same, Noise and DMC have fixed predefined sounds
4.  The output of MusConv will play, but not necessarily be the most efficient possibility

Point #1 means that even though a MIDI file can have a great deal of tracks, I actually use a subset of that capability in MusConv (out of simplicity) where the track position maps directly to the sound hardware in use. We will cover that further in the structure described below.

Point #2 means that although a MIDI track can have several chords, that just won't render correctly on the NES. In fact, the MusConv tools will try to compensate for chords and such overlap by dropping notes, but usually just ends up wrecking the data :(. Most likely you will want to tool the MIDI yourself. If you are converting an existing file and it has a "complex" track like that, I recommend trying to remove the lower notes of the chord first, just as a rule of thumb. In any case, for any given moment on a track, you only want ONE NOTE actively playing!

Point #3 means that although each MIDI track can have a channel/instrument assignment that can affect its sound, there's just some hard limits you have to deal with on the NES and SMB3. You can't set a String Ensemble patch and reasonably expect that's the sound that will come from the square wave generator. Instrument patches are configurable for the Square Tracks, but that's it.

Point #4 mostly means, space is limited, keep it short and sweet as possible. Also, changing lengths of notes or bumping your tempo +/- 1 may cause a much smaller (or larger) output, so tweaking may result in saved space. Further, given Nintendo's idea of very limited note/resting lengths, it is tricky to pad out a track efficiently when needed. The music in SMB3 was likely handcrafted, and thus the limited resting values were idealized for the original music creators.

### MUSCONV TOOL MIDI STRUCTURE\
MusConv processes a regular format 1 MIDI file with a very particular organization. To properly employ MusConv you will need a MIDI editor that tries to be minimially invasive. For example, a good MIDI editor I've enjoyed for macro editing is one called Magix Music Studio, but when it saves a MIDI it adds SysEx events and an extra track after zero. But you can always use one editor for getting the music in order and then another for post-processing. In that case, I have never found a better, simple, uninvasive MIDI editor than "WinJammer Shareware." As simple or complex as your MIDI file is, it retains it perfectly.

Whatever you use, you MUST have a MIDI that conforms to these specifications:

1.  Track 0 is used to set tempo and use a couple custom SysEx events; all other events are ignored in this track

3.  Track 1 is Square Track *2*; you have limited instrument patch support based on configuration. Only notes and Program Changes (for the instrument patches) are supported. Everything else is ignored.

5.  Track 2 is Square Track *1* (2 comes first because it is the "primary" track in the SMB3 music engine); pretty much same rules as Track 1 otherwise

7.  Track 3 is Triangle Track; only notes are recognized, everything else is ignored

9.  Track 4 is Noise Track; only notes are recognized, and limitedly at that, based on configuration; this should use MIDI Channel 10 because it is percussive, but this is not enforced.

11. Track 5 is DMC Track; similar to Track 4, just somewhat better sound. Same rules apply, MIDI-wise.\
Custom SysEx events that can be used on Track 0:

### SEGEND SysEx (53 45 47 45 4e 44 f7)\
The SEGEND SysEx specifies "End of Segment"; this is used so you can have a single MIDI file that outputs into multiple segments. The MIDI file will be virtually split by MusConv where the SEGEND appears. Do not add SEGEND in a place where it will straddle a note; MusConv just won't handle it well, and will cause problems on segment cleanup. Make sure all notes come to an end before SEGEND and new notes start at and after SEGEND. If MusConv errors out reporting that you must "partition" your MIDI file, take note of the segment it reports and further subdivide it. Note that the beginning of the MIDI up to the first SEGEND is implicitly "Segment 0." Do not place a SEGEND at the beginning of a MIDI, it is unnecessary and will cause a useless split. Each segment after a SEGEND is the previous +1, so Segment 0 is beginning of MIDI up to first SEGEND, then it is Segment 1 until the next SEGEND, etc...

### STOP SysEx (53 54 4f 50 f7)\
The STOP SysEx is mostly provided for work-in-progress conversion of a MIDI file. MusConv will stop processing the MIDI file when it hits a STOP. It is intended that you will remove it later when you are ready for the conversion to be taken farther, but whatever...

Custom SysEx events that can be used on other Track 4 (Noise) and Track 5 (DMC):

### LOOP SysEx (4c 4f 4f 50 f7)\
The LOOP SysEx enables employment of a unique feature of the Noise and DMC tracks, the ability to repeat everything up to that point for the rest of the segment. This is used most notably in the Underground tune, which actually just has a single repeating noise on the Noise track. As far as MusConv goes, this will actually terminate it processing any MIDI events on the given track until the next SEGEND occurs. That way you can have the actual repetition occuring in your MIDI for previewing sake, but MusConv will be able to generate a more efficient output. Literally you could have any notation follow the LOOP until the next SEGEND, but unless it's the actual repetition, you'll likely just confuse yourself...

PRG bank overview
-----------------

* * * * *

### PRG000) Bank 0 is mainly a bank that contains shared functionality/data for the main gameplay objects.

### PRG000.1) Slope tile definition, slope shape definition

"Slopes" are of course the ground surfaces which are not perfectly parallel with the horizon, i.e. hills and, to a lesser extent, the sloped ceilings seen in underground levels. SLOPES ARE ONLY IN USED IN TILESET 3 (Hills) and TILESET 14 (Underground); the logic is not active in any other tileset**. Meaning for example, without modification, a "Plains" level like 1-1 could never have sloped tiles, Player sliding, etc. It's also worth noting that there is no real proper slope detection system on the ceiling. There are crude calculations for the Player or object to "bump head" off of a sloped ceiling, but it's very deficient and cannot support "walking on the ceiling." Even though there are upside-down enemies, their walk code is more of a hack, and they cannot trace the sloped part of ceilings.

** NOTE: While slopes are only available in tileset 3 and 14 in normal SMB3, there is an unused feature where it can be in 5. See bank section PRG008.POI for more info.

The four Level_SlopeQuadx0 sets define what slope "shape index" represents each "solid" tile for the quadrant (see "Tiles, Tile Quadrants, Tilesets".) The values map into the Slope_LUT table which follows. Slope_LUT defines, per horizontal pixel, what the slope's "height" is across the tile, both as a "floor" and as a "ceiling." The "floor" height is comprised in the lower 4 bits, the "ceiling" height is the upper 4 bits. The reference is where the tile appears in detection, that is, if the Player/object is checking out the tile at their "head", this is a "ceiling." If at their "feet", it's a floor. All slope tiles in SMB3 are designed with a slope only on the "floor" or "ceiling" with the other side completely "flat."

+---------------+ 0
|             / | 1
|            /  | 2
|           /   | 3
|          /    | 4
|         /     | 5
|        /      | 6
|       /       | 7
|      /        | 8
|     /         | 9
|    /          | A
|   /           | B
|  /            | C
| /             | D
|/              | E
+---------------+ F

45 degree slope

Finally, Slope_PlayerVel_Effect and Slope_ObjectVel_Effect define values which effect the Player/object's horizontal speed as they walk along the slope. This factor is reversed if they turn around. The values in Slope_PlayerVel_Effect also determine if and how fast a Player can slide (by pressing DOWN on a sloped floor.) Note that the cooresponding object table is incomplete; however, the missing tiles are all "ceiling" tiles which the objects are generally incapable of handling anyway. Further there's a calculation that overflows if the object detects a slope above "shape index" $0F, so they can't access the table anyway. This "lazy" calculation bug was probably intentional as they figured no object would generally hit sloped ceilings anyway.

### PRG000.2) Misc. data

-   Level_MinTileUWByQuad: Defines when a tile is considered "underwater" (can be swam in)

-   ToadItem_PalPerItem: Palette entries for when a Player gets an item in a Toad House

-   Object_TileDetectOffsets: Defines X/Y pixel offsets for objects in different movements; see inline comments for breakdown

-   Object_BoundBox: Defines a "bounding box" (a rectangle that "bounds" the object for detection against other objects or the Player); chosen by a property assigned to the object

-   Object_AttrFlags: Set of attributes regarding object that must be known to all other objects. Mainly defines bounding box and how an enemy responds to Player interactions.

* As seen in banks 1-5, objects have a lot of properties that are defined in their grouping bank, meaning those banks must be "swapped in" before those properties can be read. Thus Object_AttrFlags contains a particular set that any object ID to any other object ID might want to cross-reference, and thus is shared.

-   SpikesEnable: Per tileset, marks which tile (and one before it) are the spike/drill/pointy-ouchy tiles; $FF indicates there is no "spike" tile in this tileset.

-   ConveyorEnable: Per tileset, marks which tile (and one before it) are the conveyor tiles; $FF indicates there is no "conveyor" tile in this tileset.

-   MuncherJelectroSet: Per tileset, defines the other "hazard" tile, typically a Muncher, occasionally a Jelectro; no $FF available here! Note this does not change the behavior of the Kuribo's Shoe, which specifically checks for Munchers only.

-   PowerUp_Ability: Per power up, marks if it is able to fly or slide on slopes.

### PRG000.3) Shared object functions (see source for full description) that are commonly called by object:

-   DoTimeBonus: Converts your time remaining to score bonus

-   SpecialObj_FindEmptyAbort/Y: Locates an empty object slot or "aborts" (i.e. does not return to caller); the 'Y' variant is for searching a sub/superset of available slots

-   Score_Get100PlusPts/Y: Gives at least 100 points based on input

-   Score_PopUp: General call to "pop up" a reward score

-   Object_HandleConveyorCarry: Object calls this if it wants to respond to conveyor tiles

-   Object_HitGround: Used after object has been determined as "hitting floor" (consult Objects_DetStat) to align properly

-   Object_WorldDetect4/8/N1: Call this for object to detect against tiles of the world. The difference in calls is how many pixels into the floor the object will still consider the floor valid, i.e. "4" is within the top 4 pixel rows of the tile, "8" is within the top 8 pixel rows, and N1 (-1) essentially means ALL pixel rows.

-   Object_BumpBlocks: Interestingly, gives an object the ability to bump blocks like the Player does, i.e. smash bricks, open [?] blocks, etc. Only Boom Boom calls this and yet Boom Boom was never put into a situation to utilize it!

-   Object_Move: Links up movement and detection of object

-   Object_AboutFace: Reverses object's X velocity

-   Object_FlipFace: Reverses object horizontally

-   Object_HandleBumpUnderneath: Call this for the object to handle being "killed" by the block underneath it getting bumped

-   Object_DeleteOffScreen: Call this for the object to be removed automatically if it falls off-screen; in a non-vertical level, this is when it moves horizontally off too far. In a vertical level, it is vertical.

-   Level_PrepareNewObject: For whatever object index in 'X', initializes this slot for a "new" object.

-   Object_ShakeAndDraw/Object_ShakeAndDrawMirrored/Object_DrawWide: Handles object "shaking" (as in when a shelled enemy is "waking up") and drawing the enemy (assuming a 16x16 sprite); note a lot of objects call this routine despite the fact that they are never "shelled". The "mirrored" variant draws the object as mirrored down the center. The "wide" variant is 48x16.

-   Object_Draw16x16Sprite/Object_Draw16x32Sprite/Object_Draw48x16Sprite: Draws an object as sprites in the named size

-   Object_ToggleFrameBySpd: Toggles between frame 0 and 1 based on how fast the object is moving (common "walk" animation)

-   Object_GetRandomNearUnusedSprite: Call this to get a free sprite (semi-randomly to distribute it in case of scanline overflow)

-   Object_CalcCoarseXDiff/Object_CalcCoarseYDiff: Calculates a coarse difference between the Player and the object in units of 4 (X) or 8 (Y), taking into account the full 16-bit coordinates between them and representing them in a way that's convient for the 8-bit register size of the 6502. The result is signed and returned in Temp_Var15. Temp_Var16 holds directional information (consult the source.)

-   Object_ApplyXVel/Object_ApplyYVel/NoLimit: Moves object by its velocity. Y velocity is up to a terminating velocity (or use "NoLimit" to not limit it, but beware of overflow.) Not necessary to call if you use a macro subroutine like Object_Move.

-   Negate: Correctly negates the value in 'A'

-   Object_AnySprOffscreen: Returns non-zero if any sprite of an object is off-screen

-   ToadHouse_GiveItem: Used only by Toad House, initializes the "pop out" item you get from a Toad box

-   Object_HitTest/Object_HitTestRespond: Called by any object which needs to check if the Player is intersecting ("touching") it. Both will return Carry Flag Set if a collision occured, the "Respond" version will also call the appropriate response; see PRG001.1.C.

### PRG000.4) Glue routines (the game engine calls these, but you might want to know about their existence)

-   Object_GetAttrAndMoveTiles: Figures out the tiles that an object is interacting with

-   Object_DetectTile: Used for object to detect tile by supplied offset

-   PSwitch_SubstTileAndAttr: Logic for tiles that are temporarily swapped by an active P-Switch; the level grid data never changes with the P-Switch, so given an input tile, this function converts it to what it "actually" is (when P-Switch is active) or just passed through (when tile is not affected by P-Switch or no switch is active.)

-   Objects_HandleScrollAndUpdate: Called to actually handle running object code and also to "spawn" them as the screen scrolls; calls many additional subroutines

-   Object_DoStateAction: Performs the appropriate action for the object in its given state

-   Object_BumpOffOthers: Checks for all objects prior to the current one to see if it has "bumped" into another; given the way this works, this function is somewhat exponential and probably one of the bigger bottleneck functions

-   Player_HitEnemy: How Player touching this object is handled

-   Object_HoldKickOrHurtPlayer: Handles object being held or kicked by Player, or just hurts Player, depending on situation

-   Player_GetHurt: What happens when Player is injured!

-   Player_Die: What happens when Player dies!

-   Enemy_Kill: Not intended to be called directly, but probably could, sets up an enemy to have been killed

-   AScrlURDiag_HandleWrap: Remember 5-9, that diagonal scrolling nightmare? It's actually a (not truly diagonal) and this function handles "wrapping" everything when the screen vertically wraps aaround.

### PRG000.5) Additional data they put in here:

-   AScroll_HorizontalInitMove: Initial "movement" code index for horizontal auto-scroll levels

-   Video data to compose the "triple card match" fanfare firework graphics (Video_3CM*)

### PRG000.POI) Points of Interest

-   Search for "$C3A8"; An unused lookup table that apparently would have defined what the "brick" tile is per tileset; it expectedly contains the same value across the board.

-   Search for "$C3EA"; An unused subroutine that apparently would forcibly move the Player around based on left/right/up/down inputs. Probably was a debug routine used for exploring a level that was work-in-progress or other troubleshooting activity.

-   Search for "$C91B"; An unused debugging/development subroutine that would toggle the Player being invincible by pressing SELECT.

-   Search for "WatrHit_IsSetFlag"; Exists in "live" code, this defines a capture for the very first water splash of the game... and is never unset or used again. May have been for debugging or possibly for an object to respond to having splashed in water... ??

### PRG001) First bank for object code. Contains code for object IDs $00-$23. Of all the object banks, this one has the most gaps in IDs, and a few objects that are never used in-game with strange behaviors.

### PRG001.1) As in all object banks, it starts off with the regular, fixed-position common lookup tables as required; ObjectGroup00_InitJumpTable, ObjectGroup00_NormalJumpTable, ObjectGroup00_CollideJumpTable, ObjectGroup00_Attributes/2/3, ObjectGroup00_PatTableSel, ObjectGroup00_KillAction, ObjectGroup00_PatternStarts. Let's go through these just once, but the idea is the same for banks 1-5. The engine just assumes these lookup tables appear at particular addresses (which, as long as their position and order is left intact, they absolutely will) and this is enforced by use of ".org" tags. Just note the ".org" tags are not strictly necessary (and as long as the data is good, effectively do nothing), but are pretty much there as reminders.

-   ObjectGroup00_InitJumpTable: Per object ID, specifies a vector to a routine that is called when the object is first brought to "life" by the system, i.e. one of the only two states you have control over per object, OBJSTATE_INIT. This is the place to initialize its variables, facing direction, velocity, or whatever is appropriate.

-   ObjectGroup00_NormalJumpTable: Per object ID, specifies a vector to a routine that is called when the object is in its "normal" state, i.e. the other of the only two states you have control over per object, OBJSTATE_NORMAL. This is what the object "does" after it has been initialized until some action causes it to change to another state. Exactly what actions can happen to it depend on what subroutines you call for support (see some of them in section PRG000.3), e.g. calling Object_HandleBumpUnderneath to cause the object to get killed from a block being bumped underneath it. Actual collision with the Player is provided by the ObjectGroup00_CollideJumpTable table, which we'll get to next. Best way to figure out how to get an object to do what you want it to do is to examine existing objects and learn!

-   ObjectGroup00_CollideJumpTable: Per object ID, specifies a vector to a routine that is called when the object intersects with the Player. (NOTE: Object must be regularly calling Object_HitTestRespond for this behavior!) Enemies may cause damage (or be injured?) and pickups can be collected, etc. (For the most part, there's really not a point to this table, since the behavior is not built-in and requires explicit calls to Object_HitTestRespond; Object_HitTest and carry check works just as well.) Also, there are couple of special values (not valid as vectors) that can be used instead of a handler:\
    1) A value of "OCSPECIAL_KILLCHANGETO" OR'ed with an object ID will cause the enemy to "die into" this other ID. Used, for example, for the ceiling-drop-twirl Buzzy/Spiny to transform into regular Buzzy/Spiny after Player stomps them.

2) A value of "OCSPECIAL_HIGHSCORE" which means that this enemy gives 1000 points when killed. Pretty much only used by bosses. Collision handling sans vector can still be implement by using Object_HitTest and checking the carry flag.

-   ObjectGroup00_Attributes: Per object ID, bitfields which set palette, width and height (for sprite, not bounding box) of the object. See "OA1_*" defined constants in SMB3.ASM.

-   ObjectGroup00_Attributes2: Per object ID, bitfields which set some behavioral mods (e.g. stomping doesn't shell/squash enemy) and selects a group of offsets to use for detection of world. See "OA2_*" defined constants in SMB3.ASM.

-   ObjectGroup00_Attributes3: Per object ID, bitfields which set the "Halt Action" (what the object does when gameplay is halted, e.g. when the Player is injured or dying) and a few more behavior mods (e.g. tail attack doesn't hurt enemy.) See "OA3_*" defined constants in SMB3.ASM.

-   ObjectGroup00_PatTableSel: Per object ID, selects which pattern table bank the object desires for display. Can use OPTS_NOCHANGE to mean that the object doesn't request any particular bank to be loaded. Otherwise, use the constants OPTS_SETPT5 or OPTS_SETPT6 (the only pattern tables allows to be requested by the object) and OR in the requested bank (limited to $00-$7F, which works just fine for stock SMB3.)

-   ObjectGroup00_KillAction: What happens to the object when it is killed (i.e. hit by a weapon of some sort as opposed to being stomped.) "KILLACT_STANDARD" is the typical "flip over and fall off" action. See "KILLACT_*" defined constants in SMB3.ASM.

-   ObjectGroup00_PatternStarts: Per object frame, specifies a pair of two patterns to be the "left" and "right" 8x16 sprite of the object when it is drawn using the built-in typical routines. For more complex enemy shapes, you will have to provide your own "draw" code, or at least code in addition to the base draw. This is also the last LUT that must appear at a fixed address.

PatternStarts HACK: If object has Objects_IsGiant set OR has its ID >= OBJ_BIGGREENTROOPA, there is an assumption that the initial bytes at PatternSets form a valid JMP $xxxx instruction to go to an alternate giant shell drawing routine (since otherwise default code is used.) I really hate this one, but that's what they came up with. :)

### PRG001.POI) Points of Interest

-   Search for "ObjP1B" (patterns defined for object $1B, the block bounce object); this defines an incomplete list of tiles used by the bounce block, but the bounce block code actually handles the sprites by itself instead of using common draw code. Not sure why since it seems like it would be sufficient to draw the bounce block with the built-in routines?

[pic] all of these

-   Object ID $01: This object is not used by the game for anything. It has code that makes it move (up to a certain speed) based on Player input when the Player is "touching" it. What purpose this may have ever served is unclear. It may have just been used to test out world collision testing.

-   Object ID $02: An even more unclear object, this one is destroyed by its initialization routine if the Player is not standing on two of the same tile (??), otherwise it starts at a vertical height matching the Player +32 (roughly at Player's feet) and moves upward at some changing speed by peculiar logic.

-   Object ID $04: This object has all the basics of a ground-walking enemy. It faces the Player upon initialization, toggles its frame between 0/1 as it "walks", etc. Probably most interesting, if the Player touches the object, it "attaches" itself to the Player (by matching his velocities) and will face the same direction as the Player. Seems like some kind of "leech" enemy or something, a la the Microgoombas. This may be a lost enemy!

-   Object ID $05: Even more interesting than $04, this behaves like it would have been some kind of enemy. It walks a bit aimlessly, jumps, and (maybe a mistake/coding error) cannot be stomped. It can be killed by heavier firepower. I would definitely bet on this being a lost enemy!

-   Object ID $0A: This has frame toggling code like many others, and is "pushable" by the Player (though it doesn't check for wall blockage.)

-   Object ID $1C: Probably the least interesting (and maybe most perplexing) object, this causes something to fly off towards the Player, out into the sky. And where it started, now a mushroom is gliding along. The only thing I can think of is that this was going to be like one of those bushes in SMW where you run by them and a mushroom pops out, though the functionality is not nearly complete enough to be used that way.

-   Object ID $21 (OBJ_POWERUP_MUSHCARD), $22 (OBJ_POWERUP_FIRECARD), and $23 (OBJ_POWERUP_STARCARD): These objects provide a rapidly flashing pickup which gives you one of your cards. It cannot actually properly handle getting 3 cards, however. This does suggest that cards were going to be something you found scattered through the level at some point, or it might have just been a test object for testing the triple card matches (i.e. developer places two by the goal card and triggers the event.) Also peculiar, this object calls a subroutine I've named "Object_MoveAndReboundOffWall", which would be a great function for a marching enemy that bounces off walls (like just about every ground troop in the game!), but only this object actually calls it. (And it normally never even moves... unless, again, it was meant to be in-game and that was going to be something it did?)

### PRG002) Second bank for object code. Contains code for object IDs $24-$47.

### PRG002.1) As in all object banks, it starts off with the regular, fixed-position common lookup tables as required; see PRG001.1 for the breakdown.

### PRG003) Third bank for object code. Contains code for object IDs $48-$6B.

### PRG003.1) As in all object banks, it starts off with the regular, fixed-position common lookup tables as required; see PRG001.1 for the breakdown.

### PRG003.POI) Points of Interest

-   Object IDs $4D and $4E are unused and immediately follow the Boom Boom variants (jumper and flyer); given the usual scarcity of unused object slots after bank 1, wonder if those were reserved for a couple additional Boom Boom variants?

-   Boom Boom has an unused state 1 which just goes into state 2. This further suggests MAYBE there was reservation for an additional variant or two.

### PRG004) Third bank for object code. Contains code for object IDs $6C-$8F.

### PRG004.1) As in all object banks, it starts off with the regular, fixed-position common lookup tables as required; see PRG001.1 for the breakdown.

### PRG004.POI) Points of Interest

-   This bank contains Object ID $75, "Boss Attack", which encompasses Koopaling attacks (e.g. Lemmy's balls) and Bowser's fireball, changing dynamically as needed for the boss. It seems kind of a hack way to do business, but there it is...

-   Object $81 (OBJ_HAMMERBRO), the Hammer Brother object, will dynamically change if Level_Event = 7 (treasure box will appear, i.e. as it is when entering a battle from the map.) It changes based on which map object ID the Player entered. (So battlefields are reusable and could have any of the bro types; this was certainly an underutilized feature!) The BattleEnemy_ByEnterID lookup table determines what object the Hammer Bro should ACTUALLY be, and further interesting is that it defines a few more map object IDs (7 / World 7 Plant, 8 / Unknown marching glitch object, 9 / N-Spade) that would never instigate the battle sequence. (Well, MAYBE the plant, in another lifetime?)

-   Object $88 (OBJ_ORANGECHEEP), the lost Orange Cheep Cheep, lives here!

### PRG005) Fourth and final bank for object code. Contains code for object IDs $90-$B3, and also the special extended object IDs $B4+ that define "Cannon Fire" and "Event"

### PRG005.1) As in all object banks, it starts off with the regular, fixed-position common lookup tables as required; see PRG001.1 for the breakdown.

### PRG005.2) The "special" objects, ID $B4+, are handled with the check at PRG005_B8DB. These objects do NOT conform to the typical definitions and are basically more for triggering events and creating the special "Cannon Fire" objects.

-   Object IDs $B4 through and including $BB are subtracted from and then directly set "Level_Event" to trigger level events. See definitions starting at OBJ_CHEEPCHEEPBEGIN in SMB3.ASM.

-   Object IDs $BC through and including $D0 spawn "cannon fire" objects, special objects which are used by cannons to periodically fire cannon balls, bullet bills, etc. See definitions starting at OBJ_CFIRE_BULLETBILL.

-   Object IDs $D1 through and including $D6 are just general special purpose objects that trigger some kind of event. Examples include a couple triple enemy spawners (neither ever used in-game), horizontal auto-scroll configuration object, etc.

-   Object IDs $D7+ are invalid and unused. But I suppose nothing's stopping you from developing them...!

### PRG005.POI) Points of Interest

-   All of the Giant [?] blocks are defined here. Each reward type is a different object ID. It is well documented but still interesting that the base power-up types are available but unused -- Mushroom, Flower, and Super Leaf. There are also two IDs following the Big [?] block series -- $9B and $9C -- which could have possibly been reserved for a couple more Big [?] Blocks, but that's uncertain.

-   One of the unused "triple" enemy spawn objects would have spawned a nice school of the lost Orange Cheep Cheeps. Funny that both the Cheep Cheeps themselves are unused and that they were also assigned one of the triple spawners.

FIXME: Verify stuck\
-   Object ID $AB is an object with a bad vector assignment that is actually used once on the World 6 Airship. Its "Normal" state calls "FireJetLR_SpriteVisibleTest", which is just a sprite visibility test function for the fire jet objects... basically meaning, it's a worthless thing to call, and causes object $AB to be stuck permanently in one of the prime object slots until the next level transition! (Because an object needs to explicitly call Object_DeleteOffScreen if it should be destroyed off-screen.) Whatever it was supposed to do is long lost...

-   Object ID $B3 is a peculiar object that basically behaves similar to a Spiny Egg, but has glitched graphics. It can also curiously be "bounced" by a bounce block to turn around. Purpose unknown, maybe it was a rolling spike ball or something of that sort?

-   The "Bonus Controller" super object will define how many coins it takes to create a White Toad House ... or the unused map object ID $0C! (Looks like a white colorization of the World 8 Airship.) This adds just a little bit more to the information about the purpose of this map object, but since all it does is drop you into 7-5, whatever purpose it has is long gone.

-   In Level_ObjectsSpawnByScrollV, which spawns objects as the screen scrolls in a vertical level, the code to spawn super ID objects for Level Events (e.g. Cheep Cheep swarms, Lakitu, etc.) is broken; in attempting to read the ID, the programmer accidentally coded to read the column of the object. Fortunately, the super IDs start at $B4, and no object would appear at column $B4+ in a vertical level (since only columns $0-$F make any sense.) In any case, these objects don't exist in any of the vertical levels (although Cheep Cheep swarming in a vertical level is pretty awesome!), so the bug was never found. It's a simple one to fix, fortunately.

### PRG006) This bank actually contains the object layouts for ALL levels. Given 8192 bytes, 3 bytes per object definition, the unused start byte and $FF termination byte, and the implied limit of 48 objects per level, the maximum possible "fully filled" levels would be 8192 / (1 + 3 * 48 + 1) = 8192 / 146 = only 56 levels! But rest assured, most levels don't come anywhere near full capacity, and quite a few "alternate" levels have no object definitions at all, so it generally is not an issue.

### PRG006.POI) Point of Interest

Not even counting the known 16 unused levels, there's quite a few straggler leftover partial object sets, perhaps from unfinished levels or removed from existing "alternate" areas. So basically any object set that uses an address as its label/filename, e.g. "_C000 / C000.asm", is not assigned to any known level. They MAY belong to some of the known 16 unused levels, but it's difficult to determine. Some of them are also just empty sets which of course were probably always empty.

### PRG007) This bank is a peculiar one. It has a lot of miscellaneous gameplay functions that, I suppose, didn't fit anywhere else. But more solidly it is the defintion of all "special" and "cannon fire" objects. The former is a simplified object type -- without all the states and a lot of the overhead of the main enemy-type objects -- and the latter is a special object type that's extremely limited and only really designed to support a cannon firing.

### PRG007.1) The Miscellaneia -- all called by Gameplay_UpdateAndDrawMisc

-   Player_DoLavaDonutArrowBounce: As the name should imply, handles the Player touching lava, donut lifts, arrow lifts, or being bounced by a block.

-   ColorRotation_Do: Performs palette rotation of the rear-most BG palette color, used when the wand is grabbed at the end of the world.

-   Player_WaterOrWaterfallVizFX: Visual effects of a Player standing in a waterfall or the occasional air bubble when underwater

-   The various "update and draw" routines (or "draw and update" if it draws before updating)\
    1) BrickBusts_DrawAndUpdate: Draw and update "Brick busts", the debris that comes from busted bricks\
2) Bubbles_UpdateAndDraw: Update and draw the air bubbles\
3) Splash_UpdateAndDraw: Update and draw the water splashes\
4) CoinPUps_DrawAndUpdate: Update and draw coins that have popped out of boxes\
5) SpecialObjs_UpdateAndDraw: Update and draw Special objects\
6) CannonFire_UpdateAndDraw: Update and draw the Cannon Fires\
7) PlayerProjs_UpdateAndDraw: Update and draw Player's weapon projectiles

### PRG007.2) Note that with Special Objects there is no implicit states or a lot of the other prepackaged stuff as seen with the mainstream objects. Of note, Special Objects have no "X Hi" (i.e. no 16-bit X coordinate) and will be destroyed if they fall horizontally off-screen. They're meant to be mostly transitory, and thus typically are projectiles or occasional, short-life objects like the coins that pop out of bricks. SpecialObjs_UpdateAndDraw is the entry point for updating and drawing special objects.

### PRG007.3) Cannon Fires -- See "Objects, Special Objects, and Cannon Fires" for more information about what they are. CannonFire_UpdateAndDraw is the entry point for updating and sometimes drawing cannon fires. Most cannon fires are invisible until they actually fire.

### PRG007.POI) Points of Interest

-   The original Japanese version of SMB3 featured the "Super Suits" flying off the Player when they took damage. Special Object ID $09 (SOBJ_KURIBOSHOE), now only used for losing the Kuribo's Shoe, originally also was used for losing the suits. This was disabled because in the Americanization of the game, when they added "Super" being the intermediate step instead of going straight to "small", the pattern bank of "small" was the only one to have the suit graphics available for the fly-off. Thus they changed to the "poof" effect and this object lost some purpose.

-   There is an unused Special Object ID $03 which has some behavior and uses part of the Boomerang projectile code. Hard to say what it was meant to be at one time...

### PRG008) This bank contains a lot of the Player specific code and animation frame lookup tables. Actual drawing of the Player occurs in bank 29.

### PRG008.1) The Animation Frame LUTs -- too numerous to list, follow comments in PRG008's source. Examples of what you'll find here is "Player_WalkFramesByPUp" (walking frames by power-up), "Player_TailAttackFrames" (frames used during tail attack), etc.

### PRG008.2) Player_DoGameplay -- this is a major macro subroutine which pretty much makes all the calls to make a playable character.

-   Player_Update: The macro of the actual updating of the Player; see PRG008_A472 for a list of various related subroutines that break down Player functionality

-   Player_Control: Pretty much all response to Player's input except the actual firing of projectiles, which occurred in bank 7

-   PowerUpMovement_JumpTable: Determines how a Player moves when using a particular power-up; noteworthy if you're interested in making your own power-ups. Check these out and follow their additional subroutines to come up with behaviors.

-   Level_SetPlayerPUpPal: Sets palette based on Player's power-up; noteworthy if you're interested in making your own power-ups

-   Level_Initialize: A once-per-level call that performs some initialization for the start of the level

-   Level_InitAction_Do: Called as part of Level_Initialize, activates the queued event specified for the level, e.g. starting to slide immediately upon entering

### PRG008.3) Player_XAccel* lookup tables: this controls the Player's acceleration in different states. If you're interested in tweaking the horizontal movement rate of the Player.

### PRG008.4) Level Action Tiles: Some tiles are tiles that the Player can hit the right way to get a bounce out of them, e.g. [?] blocks, note blocks, etc.

-   LATP_PowerUps: this table defines what power up comes out of the different action tiles, if any

-   LATR_BlockResult: Defines what tile should be restored behind the one that bounced.

-   Level_ActionTiles: Defines the base tile value for each type of action tiles. Note that there are potentially multiple spanning from the base.

-   Level_ActionTiles_HitEnable: Bitfields that specify to what input a particular action tile type may respond. This is unfortunately limited by the way checks are performed, probably for performance reasons. See comments in source.

-   Level_DoBumpBlocks: The actual subroutine which checks if the Player is actually instigating any of the action tiles.\
    Object_BumpOffBlocks: The object (kicked shell) version of Level_DoBumpBlocks

-   LATP_HandleSpecialBounceTiles: After instigating an action tile, this is what determines what the result will be!

### PRG008.5) Special Tiles: Certain tiles in certain tilesets cause special things to happen, e.g. conveyor belts, spikes, slippery ice, etc. The macro subroutine is "Player_DoSpecialTiles." It has a lot of tileset-specific coding, so you'll definitely want to go through it if you want something special to happen in a particular tileset!

### PRG008.POI) Points of Interest

-   Search for "$A6B0"; Appears to be another debug function that was to forcefully set the Player's X and Y velocity to "something" in addition to a value in Temp_Var4

-   In the Level Action Tiles, there is an unused desert world breakable "pipeworks" (those thin pipes used to build rectangular structures) tile! It functions like a brick (if you hit it from underneath when you're small, it bounces; when super, it breaks.) When it breaks it uses the "cracked pipe" tile which did get used in the game. I'm guessing it was simply a gameplay idea they decided not to use because it's contrary (bricks are the breaking block everywhere except this one occasion.)

-   LevelInit_EnableSlopes: This function is called to enable slope logic when it should be available. Technically slopes not being enabled in any tileset besides 3 or 14 is true, as far as normal SMB3 goes; but there is logic for an unused variable "Level_UnusedSlopesTS5" where, if it is set to 2, slopes ARE enabled in tileset 5, used for the two "Plant Infestation" areas of World 7. This feature is never used, but it's interesting and certainly questions what they were thinking of doing there...

### PRG009) This bank is used for the 2P Vs challenge mode (when a Player challenges the other Player in a 2P game.) While this is mostly pretty straightforward, there's still a few hidden gems in here! Of note, every time a challenge is initiated, an internal counter is incremented which sets the "game type." This most commonly sends you to the "4 pipes and POW block" battle arena, but sometimes you get one of the other special ones like the "fountain" or the "ladder" area. This also includes the actual level layout data for all of the arenas, which incidentally are the same format as all other levels. This bank also includes code for auto-scroll levels, probably just a convenient bit of free space rather than any sort of organization.

### PRG009.1) Vs_EnemySetByGameType: This points to the set of enemies that will be used in this 2P Vs game type. This only applies to the "battle arena" levels; otherwise this value is unused.

### PRG009.2) Vs_2PVsInit: The macro subroutine which performs initializations for this 2P Vs game type. For all regular arenas this doesn't do much, but this does place coins in the special "statically placed coins" arena variant and the hidden coin blocks in the ladder variant.

### PRG009.3) Vs_2PVsRun: The macro subroutine which actually runs 2P Vs game logic! Again, of course, it's mostly the same for the common arena, and there's specific logic for the special variants. However, see PRG009.POI for some tidbits... Essentially the "features" of a game type are determined by different subroutines you must call to utilize a feature.

-   Vs_PlayerMove: The movement and general interaction code of the Player. Required call of all game types... if you want Player control that is.

-   Vs_SpawnEnemies: For the arenas with regular enemy spawning.

-   Vs_ObjectsUpdateAndDraw: Required call if you want to employ 2P Vs objects. (Not related to any type of gameplay objects by any stretch.)

-   Vs_BumpBlocksUpdateAndDraw: Call this to support bump blocks (used in the common battle arenas)

-   Vs_UpdateAndDrawPOW: Call this to have a POW block. Note that it's not an object or world data; it is simply a single, hardcoded block at the one location.

-   Vs_PlayerWallEnable: Not a feature subroutine, but worth pointing out, this actually enables/disables walls on certain arenas! A lot of them don't even have walls worth being concerned with, so this explicitly disables some detection logic.

### PRG009.4) 2P Vs Object related

-   Vs_ObjectDoState: Handles the simple states of 2P Vs game objects. Vs_ObjStateNormal has a lot of common behavior (responding to bounce block, etc.) and PRG009_AD80 is where ID-specific code is called.

-   Vs_ObjectDrawStyle: A lookup table that specifies how to draw the sprite for the object from a choice of 3. See comments in source.

### PRG009.5) AutoScroll_Do: Macro subroutine used for the various sorts of automatic scrolls that occur in levels that are not the normal "camera on Player" type following. "Level_AScrlSelect" chooses the type of auto-scroll that is active. Auto-scroll is activated in a level by using a Object ID $D3 (OBJ_AUTOSCROLL) which configures automatic scrolling based on its Y coordinate (used as a parameter rather than a literal coordinate location.) Auto-scroll is responsible most obviously for levels which move horizontally and sometimes vertically on their own. Other somewhat subtler effects are, for example, the spiked ceiling in the World 1 Mini-Fortress, which employs a "32 pixel partition" floor and scrolls the rest of the area for the spiked ceiling. While most of the subtler effects are hardcoded and pretty straightforward, the horizontal-driven auto-scroll is worth looking in to a little bit...

-   AScroll_Movement: The horizontally driven auto-scroll is accelerated positively and negatively to a certain rolling speed. This lookup table defines commands which specify which component to accelerate and, generally used towards the end, an optional looping command to execute. The breakdown is like this:

Bits 0-1 (0-3): This selects an acceleration factor for the vertical auto-scroll velocity. The factors are 0, 1, -1, and -1. (Likely the repeat is just to catch errors, since really only the first three are useful.) Zero is supplied since this is a macro byte value and you may not want to affect vertical.

Bits 2-3 (0-3): Same as Bits 0-1, only for the horizontal auto-scroll velocity

Bits 4-7 (0 to 15): Activate a repeating movement command; see AScroll_MovementLoop. Typical use is an Airship/Battleship/Coin ship "bobbing" forever at the end. An appropriate value from AScroll_MoveEndLoopSelect is implicitly loaded into AScroll_MovementLoop when the regular commands for auto-scroll have

-   AScroll_MovementRepeat: For each byte in AScroll_Movement, it is paired with a "repeat" value. This enables acceleration for more than a factor of 1 as appropriate without doing duplicate commands.

Example:\
AScroll_Movement: .byte $04\
AScroll_MovementRepeat: .byte $08

Translates to:\
Movement $04: Accelerate horizontal auto-scroll using factor 1 (since $04 -> %00000100 -> bits 2-3 index = 1 -> factor 1) and vertical unaffected (since bits 0-1 are zero) and does not set AScroll_MovementLoop; this is repeated 8 times

-   CoinShip_CoinGlow: This called when specifically the Coin Ship's auto scroll is active

### PRG009.POI) Point of Interest

-   Vs_UNKGAME: Just floating with all of the other subroutines that are called under Vs_2PVsRun, there's a strange, unused one. It instigates a timer which expires and then decides that Mario lost the match. This is probably incomplete logic and related to the lost battle arena... (most coins by timeout wins??) The logic actually has a call; search for "$A306." This further "proves" that at one time it was to be a subroutine belonging to some kind of game type.

-   Especially interesting in the 2P Vs code is hints of a possibility of a Player to "reappear" after dying. In the release of SMB3, death equates to losing, but perhaps an alternate idea is that death was just a momentary inconvenience and you got to try again until someone got a winning score. To reactivate this old behavior, add an RTS after the label PRG009_A431. The Player who died will reappear at their starting position, flashing with temporary invincibility.

-   The well-known "Fire Card" glitch in the ladders/question block arena has to do with the Player erroneously "collecting" the block kicked by the other Player when the game is paused, resulting in a glitched "card" (as retrieved at the end of normal gameplay levels) that uses the 2P Vs fireball sprites. The reason for that being possible starts at some logic a bit after the "PRG009_B38D" label, where it simply checks for an object with an ID greater-than-or-equal-to VSOBJID_MUSHROOMCARD. Looking at the object IDs ...

VSOBJID_MUSHROOMCARD = 8 ; Mushroom card\
VSOBJID_FLOWERCARD = 9 ; Flower card\
VSOBJID_STARCARD = 10 ; Star card\
VSOBJID_KICKEDBLOCK = 11 ; Kicked block (from [?] block match)

... the kicked block ID follows the Star Card. While normally the kicked block is handled properly, there is a timing problem in pausing that causes it to incorrectly reach this point and wrongly be considered as a pickup item. Since the range check was not more precise, herein lies the cause!

### PRG010) The first bank controlling the World Map. Contains code for the pop-up "message boxes", post-Fortress FX, and handling Player movement.

### PRG010.1) The first part of the bank contains data in SMB3's standard simple video data instruction format. These are all the "message box" type displays of the World Map. Of important note, the map scroll position only ever sits idle at $00 or $80 (i.e. "aligned" or "aligned half"), and while this may appear outwardly to be stylistic, it also simplifies (and, in this case, hardcodes) how the "message boxes" are displayed. For each box, there is a "00" version ("aligned") and an "80" version ("aligned half") so something to keep in mind if you wish to alter these.

The following is a quick reference of the "message boxes" defined, combining the 00/80 because there's not much point to listing them twice, and related functions

Video_DoWXMario00/80: "World [x]" intro box, for Mario\
Video_DoGameOver00/80: "GAME OVER!"\
Video_DoW2WZ: "WELCOME TO WARP ZONE" (no "aligned half" version since Warp Zone never scrolls)\
Video_DoWXLuigi00/80: "World [x]" intro box, for Luigi (seems like they should have just made a common one and patched in the name like for Game Over)

Map_IntroAttrSave: Saves the nametable attribute data that will be covered over by the Message Box\
GameOver_PatchPlayerName: Patches in Mario/Luigi as appropriate to the Game Over dialog\
Map_ConfigWorldIntro: Adds Player's lives count to the "World [x]" intro box (should have patched Player name in here as well?)

### PRG010.2) With that out of the way, we can move on to the proper map stuff! Of note there's the simple lookup table

-   Map_DoMap: Macro function that makes the map happen. Calls the World 5 decorative function, handles scrolling, calling functions to open/close the inventory, calls Map_DoOperation...

-   Map_DoOperation: The world map has a state variable "Map_Operation" which defines what state its in, e.g. Map_Operation = $0 is the World Intro, Map_Operation = $D is the "Normal" operation, etc. This handles the world intro, Player "skidding" across the map, level completion effects, fortress lock busting / bridge building, making bonuses (e.g. white Toad house) appear, Player movement/entering levels, and the hand traps.

-   GameOver_Loop: Game loop while Game Over is active

-   "Fortress FX" is the name given to the "explosion" that occurs after a Fortress has been (normally) completed. The selection of which to use is selected by world first (selects from a set of 4) and then is further refined by Boom Boom himself; the upper 8-bits of his Y position will determine which one to execute (subtracted by 1 in code)\
    This works by a series of lookup tables as follows:

1) FortressFX_VAddrH/L: The appropriate VRAM address (as would be visible by the Player's location when he entered the fortress) where the map tile should be visibly updated (does not effect the tile grid in this step) (Patterns set by FortressFX_Patterns)

2) FortressFX_MapCompIdx: Selects the index of Map_Completions and particular bit of that array to set to mark a "completion" of this lock/bridge (this enables persistance of the lock/bridge changing state for each re-entry to the world map)

3) FortressFX_Patterns: The actual set of four patterns to be written to the spot of the lock/bridge change (source set by FortressFX_VAddrH/L)

4) FortressFX_MapLocation/Row: "FortressFX_MapLocation" and "FortressFX_MapLocationRow" define the row and screen/column of the actual tile grid tile which must be changed. "FortressFX_MapLocation" defines both the column (in upper 4 bits) and the "screen" (in lower 4 bits)

5) FortressFX_MapTileReplace: The actual tile ID to substitute into the tile grid. This ID should match with what the patterns have changed it to, as it will be used to display the tile if the map should scroll.

6) FortressFXBase_ByWorld / FortressFX_Wx: For each world, defines a set of 4 base indicies (which index all of the aforementioned tables); which one that fires is actually set by Boom Boom, based on his Y Hi coordinate, which is propogated to the "(?) ball" that pops out after he's defeated; this is stored into Map_DoFortressFX, which later is decremented and used as the index value. (Boom Boom is always forced to Y Hi = 1 after the assignment.)

-   Map_CheckDoMove: Macro routine that defines if and how a Player is able to move around the map, including canoe!

-   World_BGM_Arrival: Defines the BGM played when the Player arrives from a Warp Whistle (the actual songs are defined in PRG030, and why they aren't common, I don't know.)

-   Map_ScrollDeltaX/Hi: Defines a 16-bit delta that controls the speed at which the map scrolls when the Player hits an edge.

-   Map_W8DarknessFill: Dark World blackness fill in for the "dark" part of the map

-   World5_Sky_AddCloudDeco: In World 5, a decorative cloud is added. Seems like kind of a silly hardcode action, maybe other map decorations were once planned?

-   MapMove_Delta[...]: A series of LUTs used to define how fast the Player moves in or out of the canoe. Kind of ridiculous.

-   Map_EnterSpecialTiles: Defines special tiles that can also be entered outside of the normal defined ranges. This table is considered first before actually checking Tile_AddrTable which determines the normal set. See also PRG010.POI for some remarks.

-   Map_PostJC_PUpPP1/2 / Map_PostJC_PUpPML: THIS IS USED *AFTER* A JUDGEM'S CLOUD ONLY. Map_PostJC_PUpPP1/2 set the second and fourth colors of the Player's palette for their power up sprite on the map. Map_PostJC_PUpPML sets the appropriate red or green for the Player. For the INITIAL palette per power-up, see PRG027's "InitPals_Per_MapPUp."

-   World_Map_Max_PanR: Defines the width of the map, in units of $10, except for Worlds 5 and 8 which have hardcoded overrides to not scroll. Interestingly, 5 and 8 use the value $00, which could have been made to check for a non-scrollable map in a cleaner way, but that's not how they did it! See after "Map_Check_PanR" to find the override code.

-   Map_Power_Pats_F1/F2 / Map_Power_Attrib_F1/2: For frame 1 and 2 of the map sprite, defines patterns and sprite attributes for the Player sprites per power-up. There are four sprites total, upper left, upper right, lower left, lower right. On the pattern side, $27 is a magic number which causes that sprite to be invisible, used for small Mario and Judgem's Cloud.

-   Map_WarpZone_Numbers / Map_WarpZone_DrawNumbers: Defining and drawing the numbers which appear in the Warp Zone... just in case you need it... or want to get rid of it.

-   Map_Object_Valid_Left/Right/Down/Up: Defines traversable path tiles. What's interesting is the duplicated left/right and up/down blocks; Nintendo may have been thinking of having one-way tiles, or at least leaving the possibility open?

-   Map_DrawBridgeCheck/V: Defines the crossable draw bridge tile and what value must be in "World3_Bridge" for the bridge to be down.

-   Map_CanoeCheckYOff/XOff/XHiOff: Offsets from the Player to "look" for a canoe to enter. Probably not useful to change, but interesting.

### PRG010.3) This bank draws to a close with DMC samples 8, 3, and 7. These samples thus only play correctly on the world map.

### PRG010.POI) Points of Interest

-   FortressFXBase_ByWorld provides offsets into FortressFX_Wx, and 4 slots are defined for each, so the pattern is, as expected, $00, $04, $08, $0C, $10, etc. While there should be one per world, the pattern actually continues significantly all the way up to $40! This is probably a mistake, or an unintended reuse of a table meant for something else at one time, not really indicative that they were making room for up to 17 or so worlds. :)

-   Search for "$CA82" / "$CD3F" / "$CD53"; an unused subroutine which executes a different short map state system. It links to another unused routine as its zero state called "Map_StateNothing" which, as named, does nothing except procede to the next state, which is "MO_NormalMoveEnter", a very much alive and in-use routine. This possibly was used to bypass a certain amount of the map intro for the original developers, but that is not clear. There are two more routines which follow "Map_StateNothing" -- "$CD3F" and "$CD53", both unused, both MAY have been additional routines called by "$CA82." Both "$CD3F" and "$CD53" also clear unused variables known as Map_Unused72C and Map_Unused7995, and both ultimately jump to the normal WorldMap_UpdateAndDraw. This certainly makes them seem to be unused map states of no known purpose (clearing lost map events that were once controlled by Map_Unused72C and Map_Unused7995 perhaps.)

-   Map_EnterSpecialTiles: While the intended function of this LUT works correctly, a bug in the check code examines up to $10 bytes ahead from the actual end of this array. The data is loads is from palette arrays, but treated as tiles, meaning that there's a lot of tiles that will be enterable on the map that shouldn't be. But given the rigid structure of the map, this generally is not a problem, which likely explains why it went undetected despite other bug fixes. Of note, however, are a few tiles that are peculiar that were included in the list such as an alternate coloring of the World 5 spiral castle (never used), a path segment which goes nowhere (??), and perhaps most interesting, the dancing flower from World 4 (!!) It's always possible of course that these were mistakes, leftover from some kind of code/graphics reorganizing at some point, or maybe there really was to be some kind of plant-themed levels.

-   Search for "$D228"; an unused routine which POSSIBLY was supposed to do something like "bounce back" the Player if they traveled left or right into "something." Even stranger is that the exact same routine appears again, still unused, in PRG011 at "$B6F6"!

### PRG011) This bank continues map functionality from PRG010. Here we see the implementation of map objects (Hammer Bros, the bonus objects, HELP bubble, Airship, etc.) "Twirling to start" (game over), warp whistle, hand trap action code...

### PRG011.1) This bank begins with several arrays that define the set of map objects. Each world is granted a fixed size set of exactly 9 map objects. If any slots are to be unused, just use the constant value of MAPOBJ_EMPTY as the ID. Note that slot 0 is expected to be the HELP bubble (or absense thereof after the Airship has begun) and slot 1 is expected to be the Airship (thus starts as MAPOBJ_EMPTY until called upon.) Other than that, the slots are free, but again, remember that there should be room for a bonus object or two (N-Spade, White Toad House, etc.)

-   Map_List_Object_Ys / XHis / XLos: Defines the pointer by world to the beginning of the array that defines the respective coordinates of the map object by index slot.

-   Map_List_Object_IDs: Defines the pointer by world to the beginning of the array that defines the ID of each map object.

-   Map_List_Object_Items: Defines the pointer by world to the beginning of the array that defines the item received by defeating this object (typically as Hammer Bro, doesn't matter for most)

### PRG011.2) In to the map code...

-   Map_Init: This calls a subroutine to prepare the Airship for traveling around the map, and copies all map objects into their respective start positions. Also clears some Player map backup variables.

-   Map_WW_IslandY/X: Defines landing spots in the Warp Zone depending on which world you originated from.

-   Map_CompleteTile: Defines the "completion" tile to use (typically [M] or [L], except for fortress et al.)

-   Map_ForcePoofTiles: Defines a set of tiles where the magic "poof" effect is used instead of panel flip or crumbling (e.g. Toad House.) Note this only effects "quadrant zero" tiles since all other quadrant tiles (excluding fortress tiles) will always use the "poof" completion.

-   Map_PanelCompletePats: Patterns used to commit the completion of level / destruction of fortress

-   Map_NoLoseTurnTiles: Defines a set of tiles where the Player does not lose their turn after completing it (e.g. Toad House)

-   Map_WhiteObjects: Defines the IDs of the bonus objects that may be existing on the map to make sure we don't create more than one (i.e. you can only have one N-Spade, one White Toad House, etc.)

-   MO_CheckForBonusRules: Subroutine which checks whether you've earned a bonus; note this is not called if you already have one, e.g. if you already have an N-Spade, doesn't even bother to check if you've earned another one!

-   Map_Object_Do_All: Does everything the particular map object needs to do, except interact with the Player, with the exception of the canoe that sets a flag that the Player is standing on it.. See MapObjects_UpdateDrawEnter for entry code. See PRG011_AE0B for the per-ID code jump table.

-   MapObject_Pat1/2 / MapObject_Attr1/2: Stores the sprite patterns and attributes for each map object by ID

-   Map_AnimSpeeds / Map_AnimCHRROM: The former specifies ticks per frame to delay world map animations (by world), the latter specifies the CHR banks used to display the animation (not by world, unfortunately?)

### PRG011.3) Airship Travel Data. The Airship can travel to any of 3 sets each containing 6 destinations. The set is random (chosen when the world starts), the destinations are visited in incremental order.

-   Map_Airship_Dest_YSets/XSets: Pointers to the sets (each containing 6) by set by world.

-   MAT_X/Y_W[1-8]A/B/C: The lookup tables of the six destinations for X and Y (including X Hi in the lower 4 bits of the X table) by each world for each of the three sets.

### PRG011.POI) Points of Interest

-   PRG011_A000: An unused array of numbers from 0 to 7. Probably something to do with worlds, but not known. Immediately followed by...

-   Map_Unused7EEA_Vals: An array of values per world that get assigned to an unused variable known as Map_Unused7EEA... which is never employed! It is completely unknown what these values may have been used for, and they range from 2 to 5, with the exception of what would be World 4's. This value sticks out at $FF, which looks like it would "disable" whatever this was. There's nothing really significant about the World 4 map though compared to some of the others, so there's not even really a guess to be made about the purpose of this value.

-   "Map_Unused7984" and "Map_Unused798A" are two unused variables, assumed to be one for each Player, cleared when the world map is first initialized. These may have just been reserved backup values for the Player similar to the coordinates and whatnot they already keep.

-   Search for "$A2AF": UNUSED macro function. What it appears to do is examine the inventory item of the current Player in the slot determined by unused variable Map_Unused738. If that inventory slot contains a mushroom, it jumps to a state table (otherwise does nothing but regular map stuff.) What state gets called depends on what inventory slot was checked; in this case it roughly split calls the regular map update or part of the world starry entrance (?), and in only once case calls the warp whistle effect. Given these strange properties, it was probably just some kind of debug routine to test map features.

-   The Warp Whistle state system does not use state 0 (skips calling this state system under assumption warp whistle has not been used), but state 0 is mapped to an unused "WWFX_WarpWhistleInit", which is similiar to the initialization now moved to the actual usage-of-whistle routine. This unused initialization is also broken in that it may attempt to go to an invalid state 5 (which points to state 1, except it completes and goes to an invalid "state 6"); probably some unmaintained leftovers or something alternate with the warp whistle.

-   There's a defined-but-unused "large Fortress" tile (TILE_LARGEFORT; similiar to SMB1's style of end fortress) which even has exception code to "crumble" after it has been completed! The only problem with this tile is that it is usually corrupted by map animations.

-   Map_WhiteObjects defines the unused map object $0C as one of the objects that could be on the map (and it can be made to appear by configuring the Bonus Controller super object the right way!)

-   Search for "$D228"; an unused routine which POSSIBLY was supposed to do something like "bounce back" the Player if they traveled left or right into "something." Even stranger is that the exact same routine appears again, still unused, in PRG011 at "$B6F6"!

### PRG012) This bank is officially the "tileset" bank for the World Maps (Tileset 0), containing the data which makes up the 16x16 tiles from their 8x8 pattern components and provides the list of tiles which are enterable (Tile_Attributes_TS0.) Also contains some of the bootstrap logic for the World map. Like all tileset banks, this is also where the World Map construction data is stored, although this is the only instance where a tile grid is not in the standard level format. World Maps are stored as a raw grid of tiles.

### PRG012.1) Palettes

-   Map_Tile_ColorSets / Map_Object_ColorSets: The former selects a set of palette colors (by world) for the map tiles, the latter for the map object sprites, both from PalSet_Maps (in PRG027)

-   Map_Removable_Tiles / Map_RemoveTo_Tiles: "Removable" tiles are tiles swapped out after they are "completed"; fortresses turn to rubble, locks turn to pathways, etc. For tiles with such special completion (instead of standard Player marker) only!

-   Map_Completable_Tiles: Similarly, these are tiles that are just marked M/L "poof" style

-   Map_Reload_with_Completions: This function is the loader for the World Map tile grid data and also what marks the tiles as complete based on current completion data

-   Map_PrepareLevel: What actually configures (based on where you stand on the map) what level you will be entering (sets tile and object layout pointers and the destination tileset)

-   Map_DoEnterViaID: A special override function; usually "entering" on the map takes your current position and goes to the level that links there. But certain objects such as the Airship require a particular level to be entered; that's where this comes in. Per map object, it applies an override as appropriate, if one is.

-   KingsRoomLayout_ByWorld / KingsRoomObjLayout: The former is per world, the latter is constant; defines a tile layout and object set per world for the throne room. Really this is not used since all the throne rooms are basically the same. Also the object layout just points to the empty set (the "Toad and King" object is inserted manually.) Interestingly, there is a slot for World 8 (where no such castle exists), although it just points to World 1's layout.

-   Airship_Layouts / Airship_Objects: Per world tile and object layouts for each world ship. Note this does include World 8; the flying contraption on the second map screen is considered World 8's "airship."

-   ToadShop_Layouts / ToadShop_Objects: Per world tile layout and Toad House configuration for White Toad Houses (the latter is named "Objects" for consistently, but this is actually a configuration value, which is also how the world map specifies it.)

-   CoinShip_Layouts / CoinShip_Objects: Per world tile and object layouts for the Coin Ship bonus. Not really used since the entries are all the same.

### PRG012.2) World Map structure data: World map structure data is broken up in the following way:

-   By Screen -- Up to 4 map screens may exist per world\
    -   By row / column -- Each valid level is a row/column entry

The intention is that you use NoDice to modify the map data, but it is raw data and a fairly simple format broken into several files to compensate for the fact that the data was made up of several tables in different locations. Check the "maps/" subdirectory and expect the following from the files:\
1) World[x]L.asm: This is the tile grid layout data in raw form. The bytes make up each screen in left-to-right order. Each screen is 9 rows by 16 columns, terminated by a $FF byte.

2) World[x]S.asm: This is the "structure" file that defines valid level panels and the map they link to. The "ByRowType" tables define the row (in upper 4 bits) and the tileset (in lower 4 bits) of each level panel in this world. The "ByScrCol" defines the column (in upper 4 bits) and the screen (in lower 4 bits) of each level panel. LevelLayout and ObjSets each define the pointers to the level layout and object layout of the destination level. Note that some tiles do not use one or the other; Toad House uses its object pointer as a configuration of what will be in the chests, and the Spade Panel actually uses both pointers as configuration of game type and host (Toad, Koopa Troopa, Hammer bro) -- see PRG022; obviously this was not actually used in SMB3!

3) World[x]O and World[x]OY/X/H/I: List of map object IDs ("O"), the map object coordinates (Y, X, XHi AKA Screen) and item for defeating this object (typically as hammer bro, does not apply to all types.)

### PRG012.POI) Points of Interest

-   Map_Completable_Tiles: The World 4 dancing flower is specifically listed as a "completable tile"; unclear if this is just some lingering mistake or it was really meant to be a level type?

-   Map_DoEnterViaID: That wily unused Map Object $0C makes another appearance in code; here the override level is specified (as 7-5's underground portion for whatever reason)

### PRG013) Tileset 14: Underground. This bank is a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information. Note that this bank can actually fully support a "Hills" level, and sometimes does.

### PRG013.POI) Points of Interest

-   There are 35 empty would-be "Underground" levels in this bank! Nintendo probably left placeholders to facilitate constructing a new level, likely done by hand after being drawn on paper.

-   An erroneous and unused partial copy of PRG015 appears at the bottom of this bank. I am unsure why this is the case, if it was an errant copy/paste effort or a glitch in the production ROM flasher or the original ROM dumper. You can (and should) delete this data if you want to free up some space.

-   Unused levels: Contains the exit for Unused Level 2

### PRG014) Tileset 18, 2P Vs. This bank is a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information. It also contains shared Tileset generators.

### PRG014.POI) Points of Interest

-   Search for "Vs_Unused"; an unused 2P Vs level! This is a level full of coins in an arc formation, probably would have been a "most coins wins" kind of competition. Although the coins can be picked up (and replace an incorrect tile), there's no code to "win" this way.\
    [pic]!!

-   LoadLevel_Generator_TS18: In the 2P Vs variable size generators, there are some peculiar entries, some of which may just be leftovers from duplicating other tileset banks, some of which are just unused content.\
    1) "LoadLevel_VsDiamondBlocks" places a run of blocks that look like the "chocolate" type flat-front diamond blocks from SMB1. These are not used in any 2P Vs arena.

2) "LoadLevel_VsCoins" places a run of regular-sized coins (not the mini-coins as seen in games which involve coins.) These were placed in the unused 2P Vs area described above. Really strange is that it employs the coin tracking memory used to prevent coins from reappearing when crossing between levels. The tracking was probably not intended to remain there, there would be no point to it.

3) Standard vertical/horizontal generators. Not to be confused with the legitimate generators used for the pipes at the edge of the standard arena, these are the pipes as seen in regular gameplay levels. There is no reason for these to be here.

4) Search for "LoadLevel_LittleCloudSolidRun"; this generator is used to produce a line of a fully solid version of the "Judgem's Cloud" type tile. This is only used in one place, in the 5-[FIXME] "ground" section. This generator mostly recycles the code provided for the non-solid run of Judgem's Clouds (and why not!), but a simple mistake produces a serious bug! To review, a common trick in 6502 programming is to take advantage of known flag states for a short jump. (The unconditional JMP instruction requires 3 bytes to complete, the conditional branch instructions Bxx only require 2 bytes). There is no unconditional branch, but it can be used in that manner if a known fixed condition exists. Here is the problem area:

LoadLevel_LittleCloudSolidRun:\
LDX #$01 ; X = 1 (use TILE1_JCLOUDSOLID)\
BEQ PRG014_CFD8 ; <-- MISTAKE!! This is never true! They wanted a BNE!!

PRG014_CFD4:\
.byte TILE1_JCLOUD, TILE1_JCLOUDSOLID

LoadLevel_LittleCloudRun:\
LDX #$00 ; X = 0 (use TILE1_JCLOUD)

PRG014_CFD8:\
LDA LL_ShapeDef\
...

The register 'X' is used in this context to select which cloud tile will be placed in the run, the regular Judgem's Cloud or the solid one. Entering from the "solid" label, there is a LDX instruction which loads the register 'X' with 1, which is fine. The intention then to jump down to PRG014_CFD8 and proceed like normal. However! They used a "BEQ" instruction, which is a "Branch if Zero" (in this context), when they ACTUALLY wanted a "BNE" (the opposite.) This means that the branch is NEVER taken and the CPU's Program Counter continues merrily along into "executing" the lookup table. If this had wound up an invalid opcode, Nintendo may have caught this, but instead it is misinterpreted as a BIT xxxx instruction (by reading the first tile as "BIT" and the second tile plus the first byte of the following LDX as a n address) followed by a BRK (the constant zero that would have been supplied to LDX), which triggers the NMI but ultimately is MOSTLY harmless... The return is in the middle of the LDA LL_ShapeDef which is now misinterpreted as an "ASL <$07"... in one trial, I got $CF as the run length! This results in the clouds at the top of this area being very thick and overrunning the level, when it should appear differently. [pic]

### PRG015) Tileset 1: Plains. This bank is a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information. Compared to some tileset banks, this one is straightforward, very packed (with levels), and does not contain any "empty" levels.

### PRG015.POI) Point of Interest

Unused levels: Contains Unused Level 1 and 2 (TCRF's numbering)

### PRG016) Tileset 3: Hills. This bank is a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information. Note that this bank can actually fully support an "Underground" level, and sometimes does.

### PRG016.POI) Points of Interest

-   There are 3 empty would-be "Hills" levels in this bank. Nintendo probably left placeholders to facilitate constructing a new level, likely done by hand after being drawn on paper.

-   Unused levels: Contains the exit for Unused Level 2 (TCRF's numbering)

-   This bank contains a couple unused "pipe junction" areas with "dirty orange" coloring, like the lost underground levels use. This suggests that once this bank was the host for both "Hills" and "Underground" levels and it was split into a new bank at some point for more levels / palettes / etc.

### PRG017) Tileset 4 and 12: High-Up and Frozen Tundra. This bank is a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information. These tilesets are ultimately different styles, so some of the shared generators only make sense in one or the other. The merge idea is that they had enough similar between them that the majority of generators would work for both of them.

### PRG017.POI) Points of Interest

-   There are 3 empty would-be "High-Up" or "Frozen Tundra" levels in this bank. Nintendo probably left placeholders to facilitate constructing a new level, likely done by hand after being drawn on paper.

-   Unused levels: Contains Unused Level 6, 7, 11, 12, and 13 (TCRF's numbering)

### PRG018) Tileset 6, 7, and 8: Underwater levels, Toad houses, and Vertical levels. This bank is a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information. These tilesets are ultimately different styles, so some of the shared generators only make sense in one or the other. The merge idea is that they had enough similar between them that the majority of generators would work for all of them. Of particular note, Tileset 8 is the one to actually include support for "pipe maze" type areas, but in fact, any SMB3 level can operate in "Vertical" mode with more or less success, depending on the generator used and other factors. The pipe maze generators however do not work in a non-vertical level as-is. ONLY Tileset 8 levels are officially stored vertical, and conversely, Tileset 8 levels are officially never non-vertical.

### PRG018.POI) Points of Interest

-   Lost Toad House gameplay differences! There's some exciting leftovers here regarding how Toad Houses apparently changed over the development of SMB3. There is leftover code in "LoadLevel_ToadChest" that shows that once Toad Houses would be given an "ID" by the map, and tracking provided for what chests had already opened, suggesting you would be able to revisit the Toad House multiple times to get more items. Maybe there was a time when there was no inventory, or it was just meant to be a handy convenience. "THouse_OpenByID" is the array that tracks open chests by Toad House ID. "THouse_ID" is the ID of the Toad House used for selecting the correct element of the array and can still be set by the map.

Further, there is actually support for FOUR chests, specifically, there is an option to generate "little" chests in "LoadLevel_ToadMiniChest." These are similar in appearance to chest found in gameplay and even have a "lid open" frame! A thought was that regular power ups would be in the little chests and super suits and the like in big ones, similar to the giant [?] blocks. This also contains the same tracking code as the big chests for having some already opened.

-   There are 54 empty would-be levels in this bank. Nintendo probably left placeholders to facilitate constructing a new level, likely done by hand after being drawn on paper.

-   Unused levels: Contains Unused Level 8 (TCRF's numbering)

### PRG019) Tileset 5, 11, and 13: Plant infested levels, Giant World, and Sky levels. This bank is a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information. These tilesets are ultimately different styles, so some of the shared generators only make sense in one or the other. The merge idea is that they had enough similar between them that the majority of generators would work for all of them.

### PRG019.POI) Points of Interest

-   LoadLevel_DoubleCloud adds interesting doubled-up cloud platforms never used in-game although present in a lost level.

-   Unused levels: Contains Unused Level 3, 9, 10, 14, and 15 (TCRF's numbering)

### PRG020) Tileset 9: Desert. This bank is a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information. Of note, this tileset has the most unused tiles and yet relatively very few levels. Seems like there were greater plans for this tileset that were never reached.

### PRG020.POI) Points of Interest

-   There are a relative LOT of unused tiles in the Desert tileset, some of which don't even have generators! For example, there are tiles that appear to have been intended to construct crumbling brick ruins or something of that nature. There is also a strange cyan log-like platform. Most interestingly, there's a breakable pipe tile for use in the characteristic "pipeworks" structures! This tile even has specially coded hit logic to bump/break in the same manner as a brick, to leave behind the cracked pipe tile (which is used.) Even more interesting is that there are bricks to break in pipeworks structures which probably replaced the usage of this block. Possibly it was dropped because of lack of room for its "bounce" sprite, possibly removed because it was counterintuitive; a block that is like a brick but doesn't look like a brick, found only in this level style. There's also some other interesting block arrangements.

-   There are 3 empty would-be levels in this bank. Nintendo probably left placeholders to facilitate constructing a new level, likely done by hand after being drawn on paper.

-   Unused levels: Contains Unused Level 8's exit area (TCRF's numbering)

### PRG021) Tileset 2: Fortress. This bank is a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information.

### PRG021.POI) Points of Interest

-   This bank contains an unused Bowser Room, identical to one of the ones actually found in the game. Unsure why this exists; perhaps it was originally a quick test area for development of the final boss code.

-   There is one empty would-be level in this bank. Nintendo probably left placeholders to facilitate constructing a new level, likely done by hand after being drawn on paper.

-   Unused levels: Contains Unused Level 5 (TCRF's numbering)

### PRG022) Tileset 15, 16, and 17: The Bonus Game bank! This bank alone has the most lost gameplay content in all of SMB3. Bonus games in SMB3 pretty much boil down to the Roulette (Spade Panel) game and the Card Matching (N-Spade) game. But originally there was a much more complex and varied system planned. This bank is also a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information.

### PRG022.1) First off, this is a tileset bank. Why? Well, the bonus introduction scene with the Mario and (typically) Toad talking about how to win at the Roulette or Card Matching is a "level" in the standard format. The background is generated, the border is generated, the table is generated, and even the Player graphic is generated from tiles! (The host, however, is a sprite construction.)

### PRG022.2) The actual gameplay logic is so intertwined with the lost content that there's not much point in describing it here as it would be redundant. See the section "Lost Bonus Games."

### PRG023) Tileset 10: Airships / Coin Ships. This bank is a standard tileset bank; see "Tiles, Tile Quadrants, Tilesets" for detailed information.

### PRG023.POI) Point of Interest

There is one empty would-be Airship level in this bank. Nintendo probably left placeholders to facilitate constructing a new level, likely done by hand after being drawn on paper.

### PRG024) This bank handles the title screen and ending sequences along with the "Toad and King" end-of-world scenarios (BEFORE Airship; after airship is PRG027)

### PRG024.1) "Toad and King" Cinematic (BEFORE Airship; after airship is PRG027); so "King" here refers to the TRANSFORMED king

-   Cinematic_ToadAndKing: Actual entry point which handles the dialog initialization and runs the loop

-   King_DoDialog: State machine which draws the box, renders the text, waits for Player to push 'A'

-   King_PatTableByWorld / King_PalByWorld / King_Spr* tables: Defines the selected pattern table CHRROM bank to load, the palette used by the king sprites, and sprites that make up the king's transformation

-   KingToad_Sprites / KingToad_Patterns: Defines the yelling/thanking Toad

-   King_Animate: Animation code for the King transformation; controls what frame is displayed/etc

### PRG024.2) Title Screen. Semi-autonomous activity sequence for the enjoyment of viewers leaving the game idle. Really should have had music/sound I think, but maybe that would get annoying for store displays?

-   Do_Title_Screen: Entry point for the title screen and all of its activities

-   Title_DoState: State machine which controls the title screen by state

-   Title_MActionScript / Title_LActionScript: Mario and Luigi's individual "action scripts", a simple format that causes them to move and jump and whatever else to cover the activities. The format is described in the source of PRG024 above these labels.

-   Title_DoEvent: Queued events such as the dropping of the logo, initialization of objects, etc.

-   Title_UpdateObj / Title_UpdateObjState1: Jump tables by title screen object ID for non-state 1 and state 1

-   Title_DoKoopaShell: This object in particular being called out since it has a lot of interaction with Mario worth looking at (getting bonked in the head, getting powered down, etc.)

### PRG024.3) Ending Sequence.

-   Rescue_Princess: Entry point for the ending, simply named

-   Ending_DoChamberScene: State machine for the chamber scene, Princess talking, etc.

-   Ending2_DoEndPic: Renders the images of the ending sequence, loading tiles and sprites as it goes

-   Ending2_EndPicPatTable[2-5]: Loads appropriate CHRROM banks per ending picture

-   Ending2_EndPicSpriteList[H/L/Len]: Points to a list of sprites to load onto the ending picture; pointers are to raw sprite data, 4 bytes each\
    ** Ending continues into PRG025 (quite literally, mid sprite list)

### PRG024.POI) Points of Interest

-   An erroneous and unused partial copy of PRG022 appears in the middle of this bank. I am unsure why this is the case, if it was an errant copy/paste effort or a glitch in the production ROM flasher or the original ROM dumper. You can (and should) delete this data if you want to free up some space.

-   The famous debug menu lives here at "Title_DebugMenu." The activation sequence appears to still partially exist at "Title_Do1P2PMenu" (so apparently it could be accessed while at the 1P/2P menu.) The trick WAS that Player 2 would hold down the A and B buttons and that would probably have made the menu appear. The only logic that is executed in that case at this point is 5 NOP instructions, which likely replaced a LDA/STA pair to change states or something.

-   The mysterious unknown map variable Map_Unused7992 only used in dead code is cleared on the way to the world map from the title screen

### PRG025) Ending Sequence continued; contains the rest of the sprite lists and all of the full-screen ending images. Also contains the graphics to load for the title screen logo (pieced apart due to limited buffer lengths)

### PRG025.1) Ending continued

-   EndPicByWorld_H/L: Pointers to simply compressed data that makes up the ending picture. Format is described in PRG025 below this label.

-   EndPic_VRAMStart_H/L: Sets starting VRAM position for each of the pictures; saves a few bytes

### PRG025.2) Title screen palette/logo graphics; see Video_Upd_Table2 in source. Standard format as described in PRG030's "Video_Upd_Table."

FIXME: $2A thru $4C on the title screen Video_Upd_Table2; maybe debug menu?? Or part of ending??

### PRG026) Player inventory management, pointers to Big [?] and Generic Exit areas and the greater Level Junctioning code, palette fade in/out routines, status bar stuff

### PRG026.1) Inventory

-   Map_DoInventory_And_PoofFX: Runs a state machine handling flipping the inventory and using items

-   InvItem_Tile_Layout: Tile layouts for each item of the inventory

-   InvItem_Pal: Sets single palette color per item (other colors assumed constant)

-   Inv_UseItem: Jump table by inventory item ID

-   InvItem_PerPowerUp_Palette/2: Sets Player palette colors after usage of an inventory item

-   RockBreak_Replace / RockBreak_TileFix: The first is the path tile to replace when a rock is broken, the second is the patterns to use to replace the broken rock

-   Map_SetCompletion_By_Poof: Sets "completion" bit where the poof is; used for when a rock is broken

### PRG026.2) Level Junctioning (and Big [?] areas)

-   LevelJctBQ_Layout / LevelJctBQ_Objects / LevelJctBQ_Tileset: Defines layout, object, and tileset pointers for all of the Big [?] areas (always tileset 14 by SMB3)

-   HandleLevelJunction: This macro function is the magic behind going through a door or pipe and "junctioning" to the "alternate" level defined for every level. Two special types of junctions -- Big [?] Areas and "Generic Exit" -- are also handled.

-   LevelJctGE_Layout / LevelJctGE_Objects / LevelJctGE_Tileset: Defines layout, object, and tileset pointers for all of the Generic Exit areas (common for all except World 4, interestingly.)

### PRG026.3) Status Bar stuff and miscellaneous video updates

-   StatusBar_Fill_Time: Draws the current time into the status bar and also updates it

-   StatusBar_Fill_Lives: Draws the current Player's lives into the status bar

-   StatusBar_Fill_Coins: Draws the current Player's coins into the status bar and applies "Coins_Earned", issuing 1-ups as coins overflow

-   StatusBar_Fill_World: Draws the current world number into the status bar

-   StatusBar_Fill_MorL: Draws the <M> or <L> into the status bar

-   StatusBar_Fill_Score: Draws the current Player's score into the status bar and applies "Score_Earned"

-   StatusBar_Fill_PowerMT: Draws the current Power meter level into the status bar

-   StatusBar_UpdateValues: The macro subroutine that calls all of the above and support structure

-   Video_Misc_Updates: The main routine that loads data as specified from table Video_Upd_Table. Cloned entirely in PRG024 as Video_Misc_Updates2 for title/ending purposes.

### PRG026.POI) Points of Interest

-   This bank contains code for the "box in" effect for entering a level from the world map. The old "box out" effect that was cut from the American version still exists at "Level_Opening_Effect"

-   While the status bar updates are more or less optimized, there are some "force update" functions that are unused; $AFFE would update all three digits of the clock, $B205 would push out the whole score. The reason for these functions is unclear, but perhaps once they would be used for some kind of debug output.

### PRG027) Toad and King cinematic AFTER the wand is recovered (So "King" is the king restored), palette data

### PRG027.1) T&K Cinematic, post-airship

-   CineKing_DoWandReturn: Main entry point after the wand has been returned

-   Letter_Palette: Defines the entire palette for use when the Princess's letter is displayed

-   LetterItem_ByWorld: Item given by world (that you're exiting)

-   Letter_Item[Pat/Attr]_[L/R]: Patterns and attribute setting that make up the given item (if any); interestingly, this defines ALL possible inventory items (although Mushroom and Frog Suit display in an incorrect blue color)

-   KingPatternTable_ByWorld: Pattern table to load for the restored king, by world

-   KingAttr_ByWorld_Top/Bottom: Defines attributes for top and bottom king sprites

-   King_Patterns / Toad_FramePatterns: Patterns that make up the king / toad sprites

-   KingAndToad_Sprites: Base sprites for King and Toad (without the patterns, supplied in aforementioned table)

-   Letter_Bodies: Pointers to each of the letter texts, by world

### PRG027.2) Palette data; this defines 8 BG palettes and 4 sprite palettes for every tileset, indexed by table Palette_By_Tileset. Use this to trace back to the set of palettes you're targeting by tileset.

-   PalSet_[Name]: Defines the set of 8 BG and 4 sprite palettes for specified tileset.

-   BonusGame_PlayerPal: Player palettes for the bonus game (and third row is for host) FIXME CHECK

-   Map_PlayerPalFix: Default Player color (when not wearing a suit which overrides it)

-   InitPal_Per_MapPowerup: Sets INITIAL palette per map power-up. This has no effect to later changes of suits.

### PRG028) The Sound Engine. See "Music Format" for how the music format is composed.

### PRG028.1) Sound Effects. Sound effects are coded wave programmer routines; they generally do not use data (except for the "map" sounds, which does have its own very primitive format); if sound effects do use data, it's data specific to the routine in question. Basically, editing sound effects is more akin to general hacking. There is no way to make a "sound effect converter" like the MusConv tool is for music.

-   The "map" group of sound effects does work on its own simple format of "note" played at "length"; see Sound_Map_LUT and "Format of Map sound data" in source

-   For the "Player" group of sound effects, see "Sound_PlayPlayer"

-   For the "Level 1" group of sound effects, see "Sound_PlayLevel1"

-   For the "Level 2" group of sound effects, see "Sound_PlayLevel2"

### PRG028.2) Music. This does have a defined format used for all songs (as defined in "Music Format".) Each segment of music has a header, one or more segments, plays from start to end segment (except Set 1, which only plays a single segment.) Tables to look for when editing music:

-   Music_Set1_Set2A_IndexOffs / Music_Set2B_IndexOffs: Defines offsets into the cooresponding Music_Set1_Set2A_Headers / Music_Set2B_Headers tables; macros in use to make this clearer. For "Set1_Set2A", the first 8 segments are the Set 1 songs. Note due to this being a single-byte-per-index table, and each header being 7 bytes long, this limits the total number of headers defineable to 36 ($24.) Of course headers can be recycled so you can have more than 36 ($24) index offsets (up to 256), but most likely this kind of reuse is rare.

-   Music_Set1_Set2A_Headers / Music_Set2B_Headers: Defines all headers for all segments. The headers select an offset into the music rest table, point to the segment data, and specify the relevant offsets into that data for each "track" within a segment. Note that due to single byte lengths here, segment data is limited to a maximum of 256 bytes total. Due to the related IndexOffs table only being byte-wide index values, the total number of headers per set is limited to 36 ($24.)

-   Music_Set2[A/B]_[Starts/Ends/Loops]: For Music Set 2A and 2B, defines which "index" (into "IndexOffs" -> "Headers" -> Segment) of segment to play at the beginning, end (inclusive), and value to restart at for looping (as songs usually don't replay their intro segment.)

### PRG028.3) Music Segment Data. This defines the actual notation/command data (again, as described in "Music Format"), and is found in the tables labeled "M12ASegData[xx]" for Set 1 / 2A and "M2BSegData[xx]" for Set2B. Note that due to a single byte offsets specified in the header, total segment data is limited to 256 bytes. Some segment data blocks flow into PRG029.

### PRG028.POI) Points of Interest

-   Toad House and P-Switch have the same song in-game, but are in fact two different queue numbers (MUS2B_TOADHOUSE/$80 and MUS2B_PSWITCH/$A0.) This doesn't definitively suggest anything other than there was at least an idea that there would be a different song for one of the two. It is possible in some early version that a song was cut from here.

-   An erroneous and unused partial copy of PRG030 appears at the bottom of this bank. I am unsure why this is the case, if it was an errant copy/paste effort or a glitch in the production ROM flasher or the original ROM dumper. You can (and should) delete this data if you want to free up some space.

### PRG029) Remainder of music segments and Player animation code. The last few segment data blocks flow into this bank. See PRG028 for more information about those. After that, we get into Player animation and action response. There is also some stuff related to the Toad House and graphics updates for gameplay tile changes.

### PRG029.1) Player sprite/animation data. "Player_Frame" is the variable which selects the frame to display for the Player.

-   SPPF_Offsets / PFxx: "Sprite Pattern Per Frame" Offsets and the pattern sets. SPPF_Offsets defines the offset into the sprite pattern sets, defined by PFxx, each set containing 6 patterns. The offset is divided by two (scaled up later) so as to provide a larger maximum. In this case, since each PFxx requires an offset of 3, the maximum possible defined frame is 85 ($55.)

-   Player_FramePageOff: Related to the above, per-frame CHRROM bank OFFSET to use for that frame. This is offset from the base, see Player_PUpRootPage below.

-   Player_PUpRootPage: Since there's a reuse of basic frames (i.e. most power-ups walk the same, jump the same, etc.), each common Player frame is rooted from a "base" CHRROM bank page, specified in this table, per-frame.

-   Player_Draw: Macro subroutine which handles "drawing" the Player (or literally, used for configuring the sprites for display.)

-   Player_DrawAndDoActions: Another major macro function which appropriate draws the Player and handles actions (coin heaven, airships, pipe movements, etc.), although none of the instigation for those actions is here.

### PRG029.2) Toad House lottery data.

-   ToadHouse_Item2Inventory: Converts a Toad House item specification value to inventory specification value. These are different for some reason.

-   ToadHouse_RandomItem: Picks for the Mushroom/Flower/Leaf or Frog/Tanooki/Hammer lottery (depending on context), with mushroom and frog having a slightly better chance than the others.

-   ToadHouse_Box_X: Defines on-screen X positions for the 3 boxes (note lack of support for the unused 4 mini-chests; see PRG018.POI)

-   ToadHouse_ChestPressB: After Player has pressed 'B' button, determine if he is in front of a chest and what item he should received based on existing context.

### PRG029.3) Blockchange: When a block is broken or otherwise changed in gameplay, the update is not automatic. The nametable must be informed of the change.

-   BlockChange_Do: Performs the block change as appropriate based on the input value.

-   PRG029_DC82: Defines a jump table per block change request on how to handle it.

-   OneTile_ChangeToPatterns: Defines the patterns to change where the block change occurs. It would have been nicer if this could reference from the Tileset bank definition of course :)

### PRG029.POI) Points of Interest

-   Search for "$DFB4": Here lies a routine for a "glowing coin" effect like SMB1 uses. Not that glowing coins are unheard of in SMB3 (they exist on the Coin Ship), but it is curious why this routine sits here, unused. It is possible that an early SMB3 would have used glowing coins prior to the implementation of level animations.

### PRG030) Primary bank! This bank always sits at $8000-$9FFF and provides a lot of skeleton support for SMB3 and runs all of the gameplay loops.

### PRG030.1) Tile memory offsets. You should probably NEVER modify these! They exist as critical lookups that define how to get to a particular tile row and column within the gameplay tile grid. It's good to know what they are and how they're important.

-   Tile_Mem_Addr: Pointer per "screen" (horizontally every 16 columns), starting offsets for 15 screens in a non-vertical level. For Player/objects, first use the "X Hi" value as your "screen" lookup. Then offset to the proper row (16 columns per row); note that for row > 16 you must +$0100 to your tile offset ("Y Hi"). Then offset to the column within the screen (should be 0 to 15.)

-   Tile_Mem_AddrVL/H: Split (not sure why, possibly just made it convenient for per-screen lookup??) parallel tables which define a pointer per "screen" (vertically every 15 rows), starting offsets for 16 screens in a vertical level. Similar to Tile_Mem_Addr, start by "screen" (which would be "Y Hi" in this context), then row (still 16 columns per row, but no overflow problem since only 15 rows per screen), and then column (again, 0 to 15.) Note that "X Hi" has no use in vertical levels since they're not wide enough at the fixed size of 16 columns.

### PRG030.2) Video Update Data table (Video_Upd_Table). This table provides pointers to blocks using a standard format which describes how to apply mostly fixed-location, fixed-content updates to the nametable VRAM. This is used mainly for things like the HUD or messages displayed at the goal. It also has a special entry at index 0 which is in RAM (under the name of "Graphics_Buffer") which allows for dynamic-location, dynamic-content updates. Dynamic content is pretty much defined on-the-fly throughout SMB3 as needed, using the same standard format as these fixed updates. Note that some of these updates are in some other bank and thus require proper context before they'll fire correctly!

### PRG030.3) Tileset data; data that is defined per-tileset. Useful especially if creating a new tileset or at least modifying an existing one.

-   ClearPattern_ByTileset: A pattern to clear the nametable with per tileset; not particularly useful, especially since levels clear it themselves at load time.

-   PAGE_A000_ByTileset / PAGE_C000_ByTileset: Sets PRGROM pages at $A000 and $C000 by tileset. The $A000 bank is typically 14 for all "normal" gameplay tilesets, containing shared functionality for level loading.

-   TileAttribute_ByTileset: Points to the attribute table (typically solidity starts per quad) by tileset

-   Level_BG_Pages1 / 2: Sets the two BG CHRROM banks by tileset. Note that most tilesets use the default animation which constantly changes the second bank, so without explicit disabling of this behavior, for tilesets that animate, the value for Level_BG_Pages2 is mostly irrelevant. However, just for reference, it is used by NoDice, so setting it to the first page of default animation is the way to make sure it shows up correctly in the editor.

-   TileLayout_ByTileset: Pointers to the layout data for each 16x16 tile by tileset

-   LevelLoad_ByTileset: Pointers to the tileset-specific "LevelLoad" routines (usually used for tileset-specific background clearning)

-   LeveLoad_Generators: Pointers to the tileset-specific variable-sized generator jump table by tileset

-   LeveLoad_FixedSizeGens: Pointers to the tileset-specific fixed-sized generator jump table by tileset

-   TileLayoutPage_ByTileset: Provides the page required for the tileset layout; note that incorrect values are stored for tilesets 15 through 18 (bonus games and 2P Vs), even though these all do use the standard level format in some way or another, but thanks to hardcoding that exists around them, this is never an issue.

### PRG030.4) Miscellaneous data / subroutines

-   PT2_Anim: This array is the normal cycled pattern table CHRROM banks that are cycled through during gameplay.

-   PlantInfest_PatTablePerACnt: For PLANT INFESTATION ONLY, a different set of CHRROM banks that are cycled through (for the munchers that go in and out of the pipe.) This array has a constant trigger counter and thus repeats the same bank to provide "delays"

-   GamePlay_YStart / GamePlay_YHiStart: Defines the valid set of vertical coordinate starting positions for the Player

-   GamePlay_VStart: Parallel to the previous tables, this defines where the vertical scroll needs to be to support that Player position

-   GamePlay_TimeStart: Stores the 4 most significant digits that are usable as a starting clock time in a level; 0, 2, 3, or 4 (i.e. unlimited time, 200, 300, or 400 seconds.)

-   GamePlay_BGM: Set of valid BGMs to be used in level. NOTE: The selection is from 10 songs, the official space for up to 16, but there's actually room to expand this to 32 or even 64 if you use the couple of additional unused bits that exist forward from the BGM definition in the level header.

-   LevelLoad: The common entry point for level loading. Performs all of the common, standard loading that is shared across all tilesets.

-   Randomize: Shakes up the pseudo random number generator a little, producing new "random" values. To retrieve random numbers, use RandomN and a few bytes ahead if you need several.

-   Level_RecordBlockHit: A routine that is called to capture hidden 1-ups and coins collected so when the engine swaps to the "alternate" level these items cannot be recollected.

### PRG030.4) Map data (not contained in the other map banks)

-   Map_Y_Starts: Defines the Y position the Player should start on the map, per world. (The X position is fixed.)\
    World_BGM

-   World_BGM: Specifies the BGM per world (PRG010 has a duplicate table "World_BGM_Restore" for post-warp whistle, although there's no good reason for that)

### PRG030.5) Game loops; depending on state...

-   IntReset_Part2: Continuation of the RESET vector (bootstrap), clears MMC3 SRAM ($6000-$7FFF) and NES RAM ($0000-$07FF, excluding the stack space at $01xx.) Then it resets the N-Spade goal score (to 80,000 points), performs initialization and runs the title screen (jumps to Do_Title_Screen) which runs until it's time to go to the world map...

-   PRG030_84A0: Initialization of world map starts here

-   WorldMap_Loop: World Map runtime loop starts here

-   PRG030_88AD: Point reached when about to enter a level or 2P Vs battle

-   PRG030_89D1: Roulette (Spade) bonus game starts here

-   PRG030_8A55: Card (N-Spade) bonus game starts here

-   PRG030_8AC0: Card (N-Spade) loop starts here (main call is NSpade_DoGame)

-   PRG030_8AE7: Main gameplay starts here

-   PRG030_8CDE: Bonus game intro (with host explaining game) starts here

-   BonusGame_Loop: Loop for bonus game intro (and lost bonus games like the rolling die) (main call is BonusGame_Do)

-   Level_MainLoop: Main gameplay loop!

-   Do_2PVsChallenge: Entry point for the 2P Vs challenge

-   PRG030_939A: Loop for 2P Vs challenge

### PRG030.POI) Points of Interest

-   Video_Upd_Table index 3 and 4 appear to be unused, and because of the addresses used, $A000 and $A06F, it is unclear where the data was supposed to be and in what context. One possibility however is the unused apparent video update data that exists at the top of PRG026 since that is at $A000. Code begins at $A06F however so it is not 100% for certain, but an interesting coincidence regardless.

-   World Map initialization still initializes some of the known and unknown unused variables: Map_Unused72C, BonusText_CharPause, Bonus_DieCnt, Unused699, Map_UnusedPlayerVal (set to $20 for both Players), Bonus_UnusedFlag

-   Unused "lost" Bonus game types of BONUS_UNUSED_DDDD and BONUS_UNUSED_2RETURN both have special code when the game exits! The former will set an unused variable Bonus_DDDD to 1. The latter will actually mark the Player as having died (calls Bonus_Return2_SetMapPos in PRG022) and will cause the Player to return to some particular spot on the map based on the value used in the Koopa Troopa Prize game. That part is especially peculiar!

-   PRG030_948F: There's a bug here; apparently after a game over, all bonus objects on the map are supposed to be removed, but the 'X' register is initiailized to a bad value and this doesn't actually happen.

-   Bonus_Prize1: A routine for the lost rolling die game which is called if you roll a 1. To recall, the translated/incomplete instruction in this case was "If 1 appears, 1", which suggests that the game designers hadn't yet decided what rolling a 1 would actually do. What Bonus_Prize1 currently does is an improper "increment" to your "card" collection, and only on the first slot, which means your first card would go Nothing -> Mushroom -> Flower -> Star -> (Glitch.) I would definitely say it is highly unlikely this was an intended result, or that the particular spot in memory was once something else, etc. Given other rolls of the die result in similar blind increments, it's likely just unfinished, which means there's not even a hint what rolling a "1" may have ever been planned to result in.

### PRG031) Primary bank, interrupt handler! This bank always sits at $E000-$FFFF and mainly provides the interrupt vector/handlers for SMB3. The bottom of the bank contains the 6502 vector table for NMI (V-Blank) "IntNMI" / Reset "IntReset" / IRQ (MMC3 Scanline) "IntIRQ". The music engine is technically interrupt driven for timing's sake, so it is mostly contained here.

### PRG031.1) Some DMC sounds and DMC Playback. The bank starts with some of the DMC sounds used in the game. It also contains the bulk of the music engine logic.

-   Music_PlayDMC / Music_StopDMC: Official start and stop functions for the DMC audio

-   DMC_MODADDR_LUT / DMC_MODLEN_LUT / DMC_MODCTL_LUT: Defines values for each of the DMC_MODxxx registers per DMC sample. This would be the DMC_MODADDR coded address (64 byte aligned), the DMC_MODLEN length specifier, and the DMC_MODCTL (speed) specifier.

### PRG031.2) Music playback. A lot of the common music engine exists here.

-   Sound_PlayMusic: The macro subroutine that actually causes music to be processed.

-   SndMus_QueueSet2B: The point at which music from Set 2B is initialized (pointers set, start/end/loop segments set, etc.)

-   SndMus2B_LoadNext: Used to queue the next segment for playback, handles looping

-   SndMus_QueueCommon: Initialization for either Set 1 or Set 2A. Note that if you want Set 2A (which mostly behaves like Set 2B), look for "PRG031_E401", otherwise see "SndMus_Queue1"

-   Music_Sq2Track / Music_Sq1Track / Music_TriTrack / Music_NseTrack / Music_PCMTrack: Each component of the NES audio hardware respective music track playback. Handles the data and special bytes. See "Music Format" for a breakdown of the music format.

### PRG031.3) Miscellaneous Data / Subroutines

-   StatusBarMTCHR / SpriteMTCHR vs. StatusBarCHR / SpriteHideCHR: The former pair are in use when on Tileset 0 (World Map) or Tileset 7 (Toad House); they provide the CHRROM bank values in use when the scanline is at the status bar level. The latter are the typical set, in which case the sprite banks are all the same, pointing at an "all clear" bank making any sprite that should happen to be there invisible. So if any attempt is made to display a sprite within the status bar area in normal gameplay it is implicitly masked.

-   PRGROM_Change_Both / PRGROM_Change_A000 / PRGROM_Change_C000: The first sets both pages at A000 and C000 together, or the others set specific pages, based on the values in the variables PAGE_A000 and PAGE_C000. There is also a "PRGROM_Change_Both2" which basically chain-calls the specific page set routines plus stores the previously executed command in PAGE_CMD; I can only assume this was for the sake of debugging because it doesn't seem to have much other purpose.

-   Player_GetCard / StatusBar_Update_Cards / Player_GetCardAndUpdate: The first function gives the Player a card as specified in the register 'A'. The second updates the status bar to actually display all cards. The third performs both; gives the card and updates.

-   Player_GetItem: Adds an item to Player's inventory. If inventory is full, it overwrites the last held item.

-   DynJump: Extremely useful jump-by-index function as described in "DynJump"

-   IntReset: While the other vectors are pretty much just quick in and out handlers for the V-Blank and scanline interrupts, this one is actually where the software begins, the main entry point.

### PRG031.POI) Points of Interest

-   Code related to the "box out" effect (the old way of bringing up the level after entering in the Japanese original version) still exists here, although it is unused since the trigger code in other banks has been deliberately broken.

-   At the bottom of this bank, just before the vector table, exists an unused ASCII string which reads "SUPER MARIO 3" followed by a series of also unused bytes. This is likely some kind of signature line but it is not used for anything.

NoDice (What a drag)
--------------------

* * * * *

![](https://sonicepoch.com/sm3mix/NoDice11.PNG)

The disassembly has a standard level editing tool specifically wired to work with it known as NoDice. The name originates after the line "Hey, you! How about lending me your clothes? No dice?! What a drag." and the fact that the "Lost Bonus Games" had a Rolling Die that was removed.

NoDice is designed to interoperate directly with the disassembly with minimal internal hardcoding. It actually contains a simple 6502 emulation core ("M6502" by Marat Fayzullin and Alex Krasivsky) and a custom basic PPU-like display which is used to emulate specific code sections of the ROM produced by the disassembly (e.g. generators) and implement the active palette settings to make NoDice truly WYSIWYG.

A test on NoDice's saving routine on EVERY SINGLE LEVEL in stock SMB3 (even the unused ones!) proved that it outputs exactly what was input except in one case: the "end area" of 1-4. This level has a floor placed at one row below the end of the screen for some reason (an invalid position) which causes it to wrap around to the next screen. Due to the limited width of this "level", the Player can never see this. In NoDice's case, it converts the wrap-around to a more correct address as if the editor intended it to be placed at the top of the adjacent screen. So given that this was bad data to begin with, I don't consider it a deficency of NoDice, and can still say that you should be able to place a great trust in the saving of a level. Of course, regular backups are never a bad idea. :)

NoDice is very loosely inspired by [Reggie](http://rvlution.net/reggie/), the level editor for NSMB Wii. There is no assocation between the projects and the games are largely different, but some of the simplest ideas of it carried over. For example, in general, right-clicking is for placement, and left-clicking is for selecting/grabbing/dragging. There was a time I wanted to implement some more features that would make it more user-friendly like dragging generators for width/height, but I didn't want to spent a huge amount of time on this. Just needed to get it where I needed it to be. After that, I feel the community can improve on the idea.

Building on the WYSIWYG element of NoDice, it directly executes game code to construct the levels. The problem is that this code is rarely error-proofed, which means that NoDice must handle errant behavior itself. There are three protections that NoDice employs; one is "timeout", another is "range checking", and the last, and very rarely used, is "generator count." In all cases, if the condition arises, the level is automatically reverted (undo) to the previous edit.

Timeout means that if your level is taking too long to be generated by the code -- usually because a generator is stuck in an infinite loop -- the internal emulation core will time out and NoDice will report as such. Note that this system isn't perfect; a host operating system stall could cause NoDice to erroneously report a timeout when really the operation would have succeeded. If you frequently receive timeout messages that don't make sense, you may have to up the timeout limit in config.xml; see the configuration section for more information.

Range Checking looks to see if a generator has written prior to the start or beyond the logical end of RAM reserved for the tile grid. An out of control generator could continue to write out tile data potentially causing a large amount of corruption, so this check will halt the internal emulation from continuing. Note that, again due to lack of error checking, a generator could write off the "bottom" of a non-vertical level and wrap around.

Generator Counting means that NoDice checks to see if you have just placed a generator that one and only one additional generator appears after it regenerates the level. Similarly that one less and only one less generator is in the total after a deletion. This one is very rarely tripped, but an out of control generator causing corruption MAY trigger it. Given the Range Checking check, this is highly unlikely. Nonetheless, I have witnessed the behavior in some early experiments, so this protection exists nonetheless.

When it first starts, NoDice has no functionality except to open a level / map. Let's first talk about opening and editing a level.

NoDice Level (Not World Map) Editing
------------------------------------\
After opening a level (NOT a World Map), three new tabs appear in the upper right corner, "Gens" (Generators), "Objs" (Sprite-based level Objects), "Starts" (Level Junction start point definitions.) The level is displayed by default in "Generator" editing mode.

### Level Properties (Edit -> Level Properties)\
The properties window configures the level. This is where you specify the "Alternate" level and several other attributes such as music selection, palette, etc. Worth a look over to get familiar with what everything does. We'll detail the default set of properties available from stock SMB3, but be aware that game.xml actually defines these properties and thus they are flexible and changeable to meet a hack's needs.

Alternate Level: This shows the current Alternate Level and allows you to change it. Note that you must sync "Tileset of Alternate Level" yourself (NoDice will tell you after changing the Alternate Level what the value should be.)

Number of 'Screens': The size of a level is specified by number of screens; this is "wide" for non-vertical (in units of 16 columns horizontally) or "tall" for vertical (in units of 15 rows vertically.) NoDice will scale its scroll system to the size of the level. Note that you can have generators and objects anywhere up to the maximum size of the level type.

2P Vs Unknown Flag: This is a flag that is set on maps belonging to 2P Vs, but it doesn't actually have any functional difference. If it ever meant anything, it's completely lost now.

Player Y position: Selects from the set of valid Y (vertical) positions for the Player to start at

BG Palette: Selects the palette number to use for the tiles. The result of this varies by tileset, and you often find unused palettes! (What I find most interesting is most outdoor tilesets seem to have a "dawn", "morning", "afternoon", "evening", "night" type of pattern in the palettes, seeming to suggest the gameplay experience would span days.)

Object Palette: Selects the palette number to use with the sprite-based objects. The result of this varies by tileset, and you often find unused palettes!

Player X position: Selects from one of the valid X (horizontal) positions for the Player to start at

Unknown flag: Another flag which has no purpose, but is sometimes set. It internally sets a variable that is never read from. Again, whatever this meant, it's gone now.

Tileset of Alternate Level: As this value is embedded in the header and not part of the level pointers, this is a separated option. Given that I want NoDice to be flexible, that means this value is not set automatically when you set an "Alternate Level", so you must keep it in sync (NoDice will remind you.)

Level is Vertical: Setting this will cause a level to operate and render in the "vertical" fashion, as seen in pipe maze levels, for example. Only one tileset fully supports this style -- 8, which actually only works in vertical. But technically any tileset can operate in vertical mode with varying results. Give it a try!

Vertical scroll: Sets the style of vertical scrolling. The common style is to lock the screen low unless the Player is climbing/flying. Another style is free scrolling as it does horizontally. Finally there is a mode which strictly locks it high or low, which can only be traversed by an inter-level vertical transit pipe.

Entering pipe does NOT exit level (MUST set for doors): This option changes gameplay behavior where, if it is NOT set, entering any pipe that would go to the Alternate Level will cause the level to end (successfully) instead. This is intended only for use with Pipe Junctions, but actually will work with any level. Note that if it is NOT set, doors will malfunction, so do not place any doors in such a level.

BG pattern bank index: This sets the CHRROM bank in use for the display of level tiles. It might seem peculiar that this is an option, especially since, in general, selecting the wrong one for the active tileset just results in a garbage display. But this actually provides an interesting and very underutilized ability to have a custom graphics set for a level without having to create a whole new tileset. The most obvious place is in 3-7, where it uses the "Plains" style like 1-1 but the would-be metal blocks have been redecorated with a grassy appearance.

Initial action: Not generally used for typical levels, except 1-5, sets the "initial action" to occurs when the level starts. Mostly for the Airship "run, jump, and catch" transition at the end of the world. The only place it is used otherwise is in 1-5, which starts the Player sliding automatically.

Level music: Selects which BGM to play in the level

Time: Select from unlimited time, 200, 300, or 400 seconds

### "Gens" Generator Editing Mode\
Generators (my term) are the fundamental building blocks of the tile grid in SMB3. Rather than storing each tile in the grid, which would consume 6,480 bytes uncompressed, Nintendo instead opted for an approach of programatically-generated geometry. Generators only require 3-4 bytes which could pattern out literally hundreds of tiles in some cases, and are grouped together by "tileset", which defines the stylization and generator availability. For more technical information about what a "tileset" is, see "Tiles, Tile Quadrants, Tilesets." For more technical information about generators, see "Level Format." Note that NoDice groups fixed-size and dynamic generators together; a fixed-size generator is basically just a generator without parameters, after all.

Left-clicking on a tile in the level will select the top-most generator it belongs to, and automatically select the generator by name from the right-hand column. While the generator is selected, you can then change its parameters (if any.) Holding the left mouse button enables dragging the generator to a new location.

Right-click places the generator currently selected on the right-hand panel with the given parameter values (if any.)

So to duplicate an existing generator as-is, left-click on it to select it, right-clik to place it with the same parameter set.

Since generators are largely used to overlap eachother, see the "Edit" menu for options regarding ordering and shortcut keys to the same. Note that some generators are "to floor" or such conditions (listed in the name), which means that they generate "downward" until they hit a non-sky tile or something similiar. Such a generator easily can be "out of control", generating "downward" until it "wraps" around to the next screen, continuing to generate on the next screen, etc. If such a generator is made the first generator (push to back), then you will get an error and revert. (Because, remember, generators are applied in order; if no floor exists in the order that the floor-seeking generator runs, then it won't find a floor.) Just an FYI. Newly added generators are always on top / in front.

Total generators to a level is really only limited by available ROM in the bank that the level is in. This is displayed at the bottom of NoDice and the editor prohibits you to exceed this space.

### "Objs" Sprite-based Object Editing Mode\
Objects are the sprite-based, sometimes hidden, objects that make up the non-static gameplay of the world. They are aligned to the tile grid during editing as this is the manner in which SMB3 operates (row and column positions are stored, not X/Y pixels, until runtime.) You should be aware of the operation of objects and strict limitations to SMB3. Have sympathy, it is powered by an 8-bit CPU running at only 1.79MHz (NTSC) / 1.66MHz (PAL.) :)

1) Objects are stored in horizontal (or vertical, if level is vertical) order. (Thus there is no useful "layering" control of objects like there is with generators.) This is to faciliate how the engine spawns new objects; it is based on the horizontal (or vertical) screen scrolling.

2) ONLY up to FIVE regular objects can be spawned (with the exception of a rare few that use the generally reserved 3 additional priority slots.) The game will simply drop any that try to spawn beyond this point.

3) The creation and destruction of objects is based on the screen scroll; new ones are brought in, old ones are destroyed. Note that for non-vertical levels, objects are spawned and maintained the entire VERTICAL height. So the five object limit must be observed for the entire active horizontal "shaft" of screen space, top of level to bottom of level. (Although if an object "falls off the bottom", it is implicitly destroyed!)

... so if you had dreams of making a level where 100 Koopa Troopas fell on the Player, you might as well get that out of your head now. Not that it matters anyway, as there's a limit of 48 objects for the entire level, due to the amount of RAM space allocated for tracking purposes.

There are some objects which perform special functions and are hidden. These objects are referred to as "special" by NoDice and are handled a little differently. They are displayed and managed on the right hand side below the object type selection. The main reason for this segragetion is that these objects give up their row position to be used as a "parameter" byte. They will thus still spawn correct in a non-vertical level as any other object (since only the horizontal position of the object is used for its creation.) Note that because of this system, these objects are almost unusable in vertical levels. Since vertical position is the spawn control of objects in a vertical level, and the row setting is to support the parameter, means the object will be created at an upredictable time. No vertical level in SMB3 has "special" objects in it.

Trying to display special objects at a particular height is meaningless, so NoDice shows them as an "infinitely" tall column that spans the entire level and occupies a width of 16 pixels (the current column it is in.) Since that would occlude other objects, the special objects will not show up unless they are specifically selected in the right-hand list. Once selected, just that object will suddenly appear, and then can be dragged to a new horizontal position like any other object. Or, while selected, you can click Add/Delete/Properties to modify the object. All newly added special objects always appear at the far left by default. They are listed by their left-to-right order. That means if one special object is moved to surpass another, it's position in the list will change. Just something to be aware of.

Left-clicking on objects selects them, which can then be dragged to a new position.

Right-clicking places a new object based on which one is selected.

Standard editing for deletion/etc. applies, but no reordering is available as they are implicitly ordered left-to-right

### "Starts" Level Junction Start Spot / Style Editing Mode\
Perhaps one of the more confusing aspects of SMB3 is its handling of level junctions. You would think ideally you could mark a location by some kind of ID and just set that as the target for the Player, but SMB3 does not have such a fine system. The system is actually very coarse; based on a "screen" (16 horizontal columns in a non-vertical level or 15 vertical rows in a vertical level), you can set a destination in the Alternate Level. This means that all doors and pipes within a "screen" can only have one defined destination into the Alternate Level.

To set a Junction Start, first you click on the "Starts" tab. A translucent mark appears denoting the current screen. On the right you can select which screen you intend to modify, and the mark will shift to that screen. Then you click the button and you are warped to the Alternate Level. (TIP: You'll want to make sure your level has a valid Alternate Level or else this part won't work too well.)

When on the Alternate Level, you right-click anywhere within it to set the destination mark. This is fortunately much more open than the initial part of the process; any column is usable as the destination point. (This is also used for vertical levels; you set the column, not the row, which unfortunately would have been more useful!) There are also other options on the right such as to set the vertical arrival height (strictly limited) which will show up as a yellow square within the marker. The arrival style (through pipe or door) is important to set correctly for your transition to operate properly. Also important is to properly set the "Is Destination Vertical" checkbox. Set this if the target level is vertical (or not if it is non-vertical) or else there will be display problems in-game.

Finally Apply the changes and you're done. Note that this requires a special type of generator (and thus consumes 3 bytes) if the level junction was not previously defined (in which case it is modified.)

NoDice World Map (Not Level) Editing
------------------------------------\
After opening a World Map (NOT a level), three new tabs appear in the upper right corner, "Tiles", "Objs" (Sprite-based Map Objects), "Links" (Linkages between a map location and its level.) The World Map is displayed by default in "Tiles" editing mode.

### "Tiles" Tile Grid Editing Mode\
Since SMB3 stores maps in a raw format, free editing of each tile is the way to go. The Tile Editor displays all tiles on the right-hand side. Left click to select one from the right-hand side (or click on the map to "grab") and then right-click to place tiles at your pointer location. Pretty simple stuff.

### "Objs" Sprite-based Object Editing Mode\
There is a strict low limit of 9 map objects, which are shown as "slots" on the right-hand side. Select one of the slots to change the type and then right-click on the map to set its location. "Deleting" a map object is actually clearing it to a "None" type. This is due to the "slot" system of map objects employed that behaves differently than the spawn-as-needed system of main gameplay. All map objects are active all of the time on the World Map.

### "Links" Tile -> Level Linkage Editing Mode\
Links are pretty straight-forward. Right-click to place. Left-click to select and move or delete. Links are designed to be placed on top of level tiles to dictate what level they go to if they should be entered. Note that non-enterable path tiles are frequently littered with links because Hammmer Bro battles happen there.

When a Link is selected, click on the Properties button on the right-hand side to edit properties. For most map tiles, this is just a destination level. For tiles specially defined in game.xml, there can be modifications that change how the tiles use their level pointers. For Toad House, for example, the level layout is kept, but what would be the object set is a parameter dictating what items Toad can give you and also an unused identification system. The Spade Panel on the other hand was designed to be highly configurable (although this all went into disuse) and doesn't specify a level at all, but bonus game type, host, etc.

NoDice Configuration
--------------------

* * * * *

The design idea of NoDice is that you can generally hack SMB3 source without requiring the source of the editor to be modified, so long as you don't change core fundamentals about level design. (Obviously all bets are off if you make substantial changes to the level format.) To this end, there are two controlling XML files which set up the behavior of the editor. Note that almost every numeric configuration element internally uses the standard C strtoul / strtol, so you can specify most notably decimal or hexadecimal (by prefixing "0x") as you see fit.

### Boot Configuration: config.xml\
This XML file is the boot configuration for NoDice.

<game value="..." /> This element specifies the directory where your disassembly is located. When NoDice starts, it will change to this directory.

<filebase value="..." /> This element specifies the base for filenames of the disassembly. In the main disassembly, this would be "smb3", which is the base for "smb3.asm", "smb3.nes", "smb3.fns", etc.

<build value="..." /> This element specifies a path to the 6502 assembler executable. Note that at this point NoDice is in the "game" directory, so if you want to use relative paths, just remember to start from there.

<builderr value="..." /> This element must be "returncode" or "texterror"; the "MAGIC KIT v2.51" assembler that this disassembly was designed to work with does not use return codes to report errors, so instead NoDice checks its output for the work "error." This is not a great solution, but works well enough for this case. Ideally the assembler would return a non-zero code, the typical way to infer that a process has failed, and this option is left available in case you retune the disassembly to work with a different assember.

<coretimeout value="..." /> This is a timeout value, in milliseconds (so 1500 = 1.5 seconds), which is how long NoDice will allow the internal 6502 core will execute until it "gives up." Generators in SMB3 aren't generally error-proof, and it is perfectly possibly that a runaway generator will freeze the system; and thus NoDice will be frozen the same. On any remotely modern computer running in the gigahertz range, no emulation operation of NoDice should take more than a fraction of a second to complete. But obviously NoDice can be stalled by the operating system it is running on, so you shouldn't make this value too tight. I find 1.5 seconds (1500 ms) to be a very usable value, but you are free to tweak this however you see fit.

<levelrangecheckhigh value="..." /> While the "coretimeout" value is the first protection against runaway generators, the second is implicit memory range checking. Generators should only be writing in the memory range beginning at NES address $6000, the beginning of MMC3 SRAM where tile data is located, and the end address should be 7950 (the larger non-vertical level has 15 screens, each 27 rows and 16 columns, so 15 * 27 * 16 = 6480 = $1950, and added to $6000 + $1950 = $7950) While loading a level, if a write to a memory address below $6000 or between $7950 and this value occurs, it will be halted and the level rolled back. From the NES POV, writing below $6000 is a dangerous operation that will probably cause a hardware crash, while writing too far ahead will begin to corrupt runtime data. The default value of $798A is compatible stock SMB3 levels and protects some variables that absolutely should never be corrupted.

### Game Configuration: game.xml\
After NoDice collects information from config.xml and changes to the specified disassembly directory, it then uses game.xml to determine what worlds, levels, tilesets, and other bits are available to it. This file is part of the magic of removing the source code of the editor from potential customizations within the game source. It tries to encompass all tricks and hacks used by the SMB3 level designers and yet remain flexible enough for you to implement your own.

The first block contains some configuration options in the <config /> block:

	<config>
		<configitem name="title" value="Super Mario Bros. 3" />

		<!-- 	Hack enable for World 9 (or otherwise specified) warp zone world
			Disables map objects and properties window will set world destination
		-->
		<configitem name="warpzone" value="9" />

		<!-- Filename for bank where object sets for levels exists -->
		<configitem name="objectsetbank" value="prg006" />
	</config>\
The <configitem /> blocks set the options as named by the "name" attribute. The following configitem options are defined:

title: This serves no real purpose to the editor other than display in the title bar; should be the title of the game you are modifying. Default is "Untitled Game".

warpzone: Sets which world map should be treated as the warp zone; this changes the behavior of the map links to act as world warp pipes only. Default is 9.

objectsetbank: Defines the filename of the bank that holds the object sets. Default is "prg006".

### Property Box Option Blocks\
The first thing that should be discussed is the "property box option blocks" that are commonly used as a way for bytes and bitfields to be presented to the user in a friendly and controlled way. These appear in a variety of sections of game.xml. The parent tag varies depending on where the element is coming from, but all the child tags are made up of an block, which works with precisely and only one byte of data, followed by individual <option /> blocks. The number of <option /> blocks will determine what type of UI element will appear. Let's look at <options /> first...

<options mask="00001100" shift="2" display="UI label:" >

The "mask" value determines what bits of the byte are going to be affected by this option. Generally this is a contiguous set of bits, but the option is left open to you to have a non-contiguous set. (Note that this never is needed in SMB3.) The mask shown here means that the UI element should only be affecting bits 2 and 3.

The "shift" value determines how far to the left to shift the option values. This enables option values to be relative numbers, e.g. with "mask" value we have, there are a possible 4 options available, which "relative" would be 0 to 3, but shifted would be (in decimal) 0, 4, 8, or 12. The options can be specified "cleanly" as 0 to 3, and "shift" will take care to shift them to the value they need to actually be.

The "display" value is simply a string which prefaces the UI element to describe it.

The "<option />" tags, if any, which are children of the "<options />" block determine what type of UI element is to be displayed. The possible UI elements are a checkbox (for single bit changes), a spin edit control (i.e. a numbers-only textbox which has a min and max value), or a combo box / drop list. First, let's look at the <option /> tag itself.

<option label="LEVEL2_XSTART_70" display="X = $70 (7 columns over)" value="1" />

The "label" value, sometimes omitted, is an assembler constant that will be used (where it applies) instead of just writing a constant value. Note that if a label is unavailable, NoDice will just write a constant number.

The "display" value is a string which describes a drop list item only.

The "value" value is the RELATIVE value that this UI element supplies. When a change is made via the UI, the existing value will be masked by the inverse of the "mask" specified in the <options /> block and then the value specified here is shifted by the "shift" value and logically-OR'ed into the final value.

Combo box / Drop list example: Player's X position; by adding several <option /> tags within the <options /> block, NoDice creates a drop list, in this case containing 4 items.

	<options mask="01100000" shift="5" display="Player X position:">
		<option label="LEVEL2_XSTART_18" display="X = $18 (1.5 columns over)" value="0" />
		<option label="LEVEL2_XSTART_70" display="X = $70 (7 columns over)" value="1" />
		<option label="LEVEL2_XSTART_D8" display="X = $D8 (13.5 columns over)" value="2" />
		<option label="LEVEL2_XSTART_80" display="X = $80 (8 columns over)" value="3" />
	</options>

Checkbox example: Level is vertical; if only a single <option /> tag exists within the <options /> block, NoDice creates a checkbox, since "obviously" the value is boolean. Note that in this case, it only makes sense that the <options /> "mask" specifies a SINGLE bit to change, and that the <option /> block omits both the "display" and "value" fields; the former is not used, and the latter is implied.

	<options mask="00010000" shift="4" display="Level is Vertical:">
		<option label="LEVEL3_VERTICAL" />
	</options>\
Spin control example: Map Junction Index. Since this is just a literal number from 0 to 31, a combo box / drop list would be an inefficient way to enable the user to change this value. By having no <option /> children, NoDice creates a spin control on the UI.\
<options mask="00011111" shift="0" display="Map jct index:" />

Each <options /> block exists within some type of "unit block", which defines a single byte. For example, the first byte of the level header is broken into the <header1 /> unit block, and within <header1 /> exists all of the children <options /> blocks that have their own <option /> children (if applicable.) This leads us to the next example, the show-if ("showif") system which only shows a particular <options /> block UI element if a condition is satisfied.

Advanced example: "id" / "showif-id" / "showif-val"

The "showif" system causes <options /> UI elements to be shown only if they make sense / apply to the context. The relationship is a simple one; within a block defining a unit value, each <options /> block can have an "id", which is a string literal that is used to identify it to other <options /> blocks in the same unit block that are participating in "showif." The best example of this is the Auto Scroll object which takes a "sub-parameter" in the lower nibble of its parameter based on which Auto Scroll type is selected.

The <options /> block that is defining itself as a source value for "showif" looks like this:

<options mask="11110000" shift="4" display="Auto-scroll function:" id="autofunc" >

The new attribute "id" gives this <options /> block the identifier of "autofunc", which can then be referenced by another <options /> block within the same unit block.

The dependent <options /> block look likes the following:

<options mask="00001111" shift="0" display="Set 0 Option:" showif-id="autofunc" showif-val="0">

The new attribute "showif-id" is set to the same identifier as the <options /> block we're depending on.\
The new attribute "showif-val" is the value the UI is looking for to enable the display of this UI element.

Thus, putting this all together, the <options /> block with the ID of "autofunc" must have a relative value of "0" selected from its available choices for the dependent <options /> block to even appear.

### <levelheader> Level Header\
The first XML block which provides UI configuration options for the five option bytes besides the pointers to the alternate level layout/objects. It is structured as follows, with each of the five option bytes specified as "header1" through "header5"; these are "unit blocks" for "Property Box Option Blocks" shown when user requests level properties (which also includes hard-coded handling for the "Alternate" level.)

<levelheader>
	<header1>
	...
		[Property Box Option Blocks]
	...
	</header1>
	<header2>
	...
		[And so on]
	...
	</header5>
</levelheader>

### <jctheader> Junction parameter header\
The Level Junction system stores a tile-aligned X coordinate for the Player and also a second byte which sets the Y coordinate among other options. This is controlled by the block. This is a "unit block" for "Property Box Option Blocks." This is visible on the right-hand side window in NoDice when editing a junction start.

### <objects> Sprite-based Objects, except World Map\
Obviously the SMB3 gameplay is not just made up of the tile world; the actors within it are the sprite based objects. Some of these are invisible controller objects with special functionality. The block contains a series of children which define the object. There is not a standard way to draw an object in SMB3, with a lot of them hardcoding their methods, so NoDice is unable to infer this directly. As such, blocks can also contain sprite piece definitions you can use to manually specify a "thumbnail" for the object that will display using the active object palette.

A typical <object /> is as follows:

	<object id="0x0B" label="OBJ_POWERUP_1UP" name="Power-Up 1-Up" desc="1-Up Mushroom">
		<sprite x="0" y="0" bank="4" pattern="0x10" palette="2" />
		<sprite x="8" y="0" bank="4" pattern="0x10" palette="2" hflip="1" />
	</object>\
The block defines the existence of the object.

The "id" value is the numeric ID value which the game engine spawns the object under.

The "label" value specifies an assembler constant to use when writing the intermediate assembler files which must equate to the "id" value.

The "name" value is what is shown in NoDice for selection.

The "desc" value is a short description of the object.

Within an object are an optional series of <sprite /> children, each one defining relative X/Y coordinates, a CHRROM bank, a pattern, a palette, and optional flip values. Setting "hflip" to 1 will horizontally flip the sprite. Setting "vflip" to 1 will vertically flip the sprite. The sprite is implicitly an 8x16 sprite, and the pattern value specifies the first of the two sequential 8x8 patterns that are displayed.

If <sprite /> tags are defined, NoDice will use these to display the thumbnail of the object. If they are not defined, NoDice displays a generic 16x16 square with the ID stamped on it. An exception to this rule is if the object is a special "hidden" controller type object.

Some objects in SMB3 are hidden objects which act as controllers to set conditions. These objects by standard exchange their "row" coordinate byte to be used as a parameter instead. (Note that these type of objects only make sense in non-vertical levels; they are basically unsupported in vertical levels, and never used in any vertical levels of stock SMB3.) To define such an object, you specify a <special /> block within the <object /> block. Note that <special /> is a "unit block" for "Property Box Option Blocks." Also, NoDice will never draw sprites for this type of object; since it loses its row position information in exchange for a parameter, it is instead displayed as a rectangular column 16 pixels wide and extending the entire width of the screen. An example of a "special" defined object, the Pipeway Controller:

<object id="0x25" label="OBJ_PIPEWAYCONTROLLER" name="Pipeway Controller" desc="Pipe Way Controller (World Map pipe-to-pipe location setter)"> <special> <options mask="00011111" shift="0" display="Map jct index:" /> </special> </object>

*** <mapobjects> Sprite-based Objects, World Map only

The <mapobjects /> block also uses the <object /> child blocks, which are basically the same as the regular gameplay <objects /> block, except the World Map does not have a concept for "special" objects. So with that exception, see the <objects /> block for information about the <object /> block.

The <mapobjects /> block also defines "item received" from a map object (technically only for Hammer Bros/W7 Plant).

The list of available items for the map objects is in a series of "<item />" blocks, an example of which is shown below:

<item id="0x03" name="Leaf" />

Pretty straightforward; the item has an ID and a name that is shown on the UI.

### <maptiles> Special Map tiles\
This block contains special property information for map tiles which don't use the layout/object pointers in the typical fashion (Toad House and Spade Panel in stock SMB3.) Each block is a <tile /> block which defines what tile ID is to be affected. An example (Toad House) is shown below:

<tile id="0x50"> <objectlayout> <low> <options mask="00001111" shift="0" display="Toad House ID (THouse_ID, Unused):" /> </low> <high> <options mask="11111111" shift="0" display="Toad House Item:"> <option display="INVALID" value="0" /> <option display="Warp Whistle" value="1" /> <option display="P-Wing" value="2" /> <option display="Frog" value="3" /> <option display="Tanooki" value="4" /> <option display="Hammer" value="5" /> <option display="Random Super Suit" value="6" /> <option display="Random Basic Item" value="7" /> </options> </high> </objectlayout> </tile>\
The <tile /> block simply takes an "id" attribute to define what tile we are describing.

Within the <tile /> block, there are two possible blocks to choose one or both bytes to modify. An <objectlayout /> block will override the pointer usually reserved for a specific object set. Note that means if the object pointer is overridden it will not be set if you select a new level. Similarly, a <tilelayout /> block overrides the pointer that usually leads to the main level format data. If one of these blocks is defined, you will not be able to select a level to load through the typical interface. You should also specify the tileset value in the "tileset" attribute, if it applies (which it most certainly does for spade panels! It must be 15!) Note that this value is limited to 4-bits (intended to be 1 to 15) by the map format; the other 4-bits are the "row" of the map link.

Within the <objectlayout /> or <tilelayout /> blocks are <low /> and <high /> blocks. These blocks simply specify which byte of the two byte pointer you are modifying in that context. These are "unit blocks" for "Property Box Option Blocks."

*** <tilesets /> Tileset / Level Definitions

The <tilesets /> block is the largest, most intricate, and probably most important block in the NoDice game.xml. This defines information about all of the "tilesets" defined in the game, what levels exist inside of them, etc. The basic structure is shown below:

<tilesets>

	<tilehints>	<-- NOTE, this one is global
		...
	</tilehints>

	vvv NOTE, tileset id="0" is special, reserved for World Maps, just like SMB3 defines it
	<tileset id="0" name="Maps" desc="World Maps">
		<levels>
			...
			<level name="Map World x" layoutfile="Worldx" layoutlabel="x" objectfile="WorldxO" objectlabel="Wx_Obj" desc="..." />
			...
		</levels>
	</tileset>

	vvv One of these exists for tileset id="1+"
	<tileset id="x" name="..." path="..." rootfile="..." desc="...">
		<vargenerators>
			...
			<generator id="x" name="..." desc="...">
				<param name="..." min="..." max="..." />
			</generator>
			...
		</vargenerators>
		<fixedgenerators>
			...
			<generator id="x" name="..." desc="..." />
			...
		</fixedgenerators>
		<levels>
			...
			<level name="..." layoutfile="..." layoutlabel="..." objectfile="..." objectlabel="..." desc="..." />
			...
		</levels>
	</tileset>

</tilesets>\
Since NoDice was designed to be a WYSIWYG editor which uses actual assembled game data to display the levels, this means that a lot of times levels are brought up in an initial state where hidden coins, blocks, and such things are just as they should be at initialization -- hidden. That certainly makes it difficult to see them and could cause them to be accidentally covered up or forgotten about. There are also tiles that are difficult to simply spot and determine purpose, like different pipe tiles which all LOOK the same, but have different functions. Enter the <tilehints /> block to help this situation...

<tilehints /> matches a tile ID to a related 16x16 PNG graphic file that provides user feedback about the nature of a particular tile. This was chosen over using CHRROM graphics since this enables a lot of better feedback about tiles that are ambiguous upon visual inspection. The PNG is displayed with user-selected opacity over top of the tile it is hinting about. Note that a <tilehints /> block exists directly under the <tilesets /> block which defines "global" tile hints. Within a tileset, you may define another <tilehints /> block which defines special tiles for that tileset only. The <tilehints /> block contains <tilehint /> children which simply match the ID to a PNG file as shown in this example:

<tilehint id="0x6D" overlay="coin10.png" />

This defines that tile ID $6D, typically a brick, is actually a brick containing the potential 10 coin bonus. Visual inspection would not clue you in to the purpose of the tile, but the tilehint does, with a graphic of a coin with a big "10" stamped on top of it.

After the initial <tilehint /> block, the next block encountered is the <tileset /> block. This is what defines generators and levels available for a given tileset. On the World Map, which stores raw data instead of generator makeup, no generators are defined. For more information about what a "tileset" is, see "Tiles, Tile Quadrants, Tilesets." For more information about generators, see "Level Format."

World Maps are a bit difficult to define because they use a lot of data spread over different banks, unlike levels which are nicely contained. So some restrictions and assumptions needed to be imposed to make NoDice usable at all. This is visible in the block. We will revisit this after describing the typical <tileset /> block.

The start of a typical tileset is a <tileset /> block which contains attributes:

id: The "id" number of the tileset, per SMB3 engine (what goes into the Level_Tileset variable)

name: Name of tileset, used for informational display and filtering in the Level Load screen

path: Relative file system path to all of the ASM files as they appear under "PRG/levels" in the disassembly

rootfile: Optionally specify the root ASM file under "PRG/levels" which includes the layout ASM files for this tileset. If this is omitted, "path" is used for this value.

desc: Describes the nature of the tileset

Within the <tileset /> block are a set of three blocks which describe necessary information. The first such block, <vargenerators />, contains information about the dynamic size / variable input generators. This block contains a series of <generator /> child blocks which define all the parameter names (and, implicitly, the optional additional byte.) Every <generator /> under <vargenerators /> MUST have at least one parameter, since one 4-bit parameter is always implied to exist. Looking at the <vargenerators /> tree:

	<vargenerators>
		...
		<generator id="x" name="..." desc="...">
			<param name="..." min="..." max="..." />
		</generator>
		...
	</vargenerators>

Within <vargenerators /> is a <generator /> child block. This has typical attributes:

id: The "id" number of the generator, per SMB3 engine

name: Name of generator, used for informational display

desc: Describes the output of this generator

Within the <generator /> block is a block. Of note, at least one of these MUST be defined for variable generators. Also, the first parameter by default has the range of 0-15 (due to being 4-bit) while additional ones have the default range of 0-255.

name: Name of parameter, displayed in UI

min: An OPTIONAL attribute which can define the minimum value of this parameter

max: An OPTIONAL attribute which can define the maximum value of this parameter

The next block is the fixed-size generator <fixedgenerators /> block which similarly contains <generator /> child blocks. However, since fixed-size generators can not take parameters, the <generator /> child is self-closed and children are not used at all.

The final block within the typical <tileset /> block is the <levels /> block which defines all available levels in this tileset. Remember, "level" is a generic term, and things like 2P Vs and the Bonus Minigame introduction area are also "levels." The <levels /> block contains a <level /> child for each level in the tileset which takes a series of attributes that connect the level and its filesystem representation with NoDice. The attributes are broken down as follows:

name: Name of level, used for informational display

layoutfile: This is the ASM file name (no extension) which contains the raw assembler source of the level. This is the file that NoDice will WRITE to when saving a map for rebuild. Note that this file is never read from (the assembled binary is instead) and will be destroyed and recreated upon saving.

layoutlabel: This is the assembly label where NoDice will seek to load the level from. The level is READ from the ASSEMBLED BINARY (not the on-disk file)

objectfile: This is the ASM file name (no extension) which contains the raw assembler list of objects for this level, to be found under "PRG/objects" in the assembly source tree. This is the file that NoDice will WRITE to when saving a map for rebuild. Note that this file is never read from (the assembled binary is instead) and will be destroyed and recreated upon saving. You can supply the name "Empty" and set the objectlabel attribute to "Empty_ObjLayout" for a level devoid of objects. This triggers a special mode in the editor that prohibits object editing.

objectlabel: This is the assembly label where NoDice will seek to load the object list from. The list is READ from the ASSEMBLED BINARY (not the on-disk file.) You can supply the name "Empty_ObjLayout" and set the objectfile attribute to "Empty" for a level devoid of objects. This triggers a special mode in the editor that prohibits object editing.

Coming back around to the World Map tileset 0, this block does not have the <vargenerators /> or <fixedgenerators /> blocks since the World Map is stored raw and doesn't use them. It does still contain a <levels /> block which is mostly the same except for some hacks in NoDice to support the spread and break-up of maps in the binary.

name: Name of World Map, typically "Map World [x]"

layoutfile: This actually defines the BASE of several ASM files to be found under "PRG/maps" since World Maps break up their information into non-contiguous blocks. You can check out the "PRG/maps" directory to see the break-up, but in any case, just understand that this is a base filename, not the full filename.

layoutlabel: In this case, this is used to SPECIFY THE WORLD NUMBER ONLY, with World 1 being "1", World 2 being "2", etc. This is due to the wide variance of labels that need to be referenced to load/save a World Map.

objectfile: This one operates the same as other <level /> blocks, but typical naming is "World[1-8]O", and loaded from "PRG/maps", not "PRG/objects"

objectlabel: This one operates the same as other <level /> blocks

desc: Same as other <level /> blocks

	<tileset id="0" name="Maps" desc="World Maps">
		<levels>
			...
			<level name="Map World x" layoutfile="Worldx" layoutlabel="x" objectfile="WorldxO" objectlabel="Wx_Obj" desc="..." />
			...
		</levels>
	</tileset>