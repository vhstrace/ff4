# Final Fantasy II (SNES) Game Genie codes

These codes were made from the USA 1.0 release (Final Fantasy II (U)
v1.0, CRC32 `65D0A825`). SNES Game Genie codes have no compare byte; the
original byte in each raw form is documentation only. All effects come
from the disassembly. They have not been tested in play unless an entry
says so.

## Skip the title screen

**Code:** `4A67-D70D`
**Raw form:** `00803D:2C` (orig `$20`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 09:45:52

Tags: `utility`

**Mechanism:** This is the FF2 version of the "skip the title screen"
codes for ALTTP, SMW and Super Metroid. `Reset` (`field/field.asm`) ends
with:

```
        jsl     InitCtrl_ext
        jsr     ShowTitle

SoftReset:
@8040:  jsl     GameLoadMenu_ext
```

`ShowTitle` (`field/title_en.asm`) starts the Prelude, shows the title
screen and waits for the A button. The code changes the `jsr` (`$20`) to
`bit abs` (`$2C`). `bit $85FA` only reads a byte of ROM, so the game goes
straight from power-on to `GameLoadMenu_ext`: the load menu, or a new
game at once when there is no save file (`GameLoadMenu` skips the menu
when `ValidateSRAM` fails).

**Evidence:** `ff2_original.smc` has `20 FA 85` at `$00:803D` (file
offset `$00023D`), and `22 0C 80 01` (`jsl GameLoadMenu_ext`) right after
it at `$00:8040`. `ShowTitle` has no other caller. The game over path in
`field/field.asm` (`jsl ClearRAM`, then `jmp SoftReset`) already enters
the load menu without the title screen, so the menu doesn't need it.
The code decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The Prelude that `ShowTitle` starts doesn't play, so the load menu may
  be silent.
- The Red Wings opening still plays. It is event `$10`, run by
  `lda #$10 : jsr ExecEvent` in the new-game path. The event scripts are
  data that the source doesn't hold, so a clean code to skip it was not
  possible. Leaving out the event would leave no map loaded.

## The prologue text scrolls past 32 times as fast

**Code:** `DD87-076D`
**Raw form:** `00B13E:00` (orig `$0D`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 09:47:20

**Mechanism:** This is the FF2 version of the Zelda 1 and ALTTP "fastest
text" codes. Normal dialogue in FF2 already shows a whole page at once
and waits for a button, so the only slow text is the auto-scrolling
prologue. `Special_09` (`field/special.asm`, "special effect $09:
prologue") sets `$cb` ("enable auto-scrolling dialogue") and then calls
`OpenDlgWindow` (`field/window.asm`). Each page there scrolls in through
this loop:

```
@b372:  jsr     WaitVblankLong
        lda     $cb
        bne     @b386                   ; branch if auto-scrolling dialogue
        inc     $bb
        inc     $bb
        inc     $bb
        inc     $bb
        dec     $07
        jmp     @b396
@b386:  lda     $7a
        and     #$07
        bne     @b396
        inc     $bb
        lda     $7a
        and     #$1f
        bne     @b396
        dec     $07
```

`$07` starts at `$10`. A normal page scrolls 4 pixels a frame and is in
place after 16 frames. The prologue scrolls 1 pixel every 8 frames, so
each page takes 512 frames (about 8.5 seconds). The code sets the `bne`
offset to 0, so the prologue takes the normal path too: each page takes
16 frames. The prologue still skips the wait for a button, because that
check (`lda $cb : bne`) comes earlier in `OpenDlgWindow`.

**Evidence:** The `@xxxx` labels in `window.asm` don't match the USA
ROM here. `ff2_original.smc` has the loop at `$00:B138`: `A5 CB D0 0D`
at `$00:B13B` (the `bne` offset is at `$00:B13E`), the four `E6 BB` at
`$00:B13F`, and `A5 7A 29 07 D0 0A E6 BB A5 7A 29 1F` at `$00:B14C`. The
byte pattern occurs only once in the ROM. `$cb` is set only by
`Special_09` and cleared right after it. The code decodes back to its raw
form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The prologue text goes by too fast to read: a page every 16 frames.
  In effect, it skips the prologue.
- Which event runs `Special_09`, and when the prologue shows up, isn't
  in the source (the event scripts are data). It is probably part of the
  opening.
- Normal dialogue doesn't change. It was already one page at a time.

## Screen fades on the map are up to 8 times as fast

**Codes:** `DD6C-AFA4` + `DD62-AF64` (use both)
**Raw form:** `008BA7:00` (orig `$F0`) + `008BD6:00` (orig `$E4`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 09:48:47

**Mechanism:** This is the FF2 version of the SMW "every fade-in is
instant" and "every fade-out is instant" codes. `FadeIn` and `FadeOut`
(`field/field.asm`) take a fade rate in A ("frame counter mask") and step
the screen brightness `$80` once each time the frame counter `$79` hits
a multiple of rate + 1:

```
@8db6:  jsr     WaitVblankShort
        lda     $80
        sta     $2100                   ; set screen brightness
        inc     $79
        lda     $79
        and     $82
        bne     @8db6
        inc     $80
```

`FadeOut` has the same `and $82 : bne` test. Each fade has 16 steps, so
rate `$07` takes 128 frames (about 2 seconds) and rate `$03` takes 64.
Event command `$da` ("fade in/out", `EventCmd_da`) always uses rate
`$07`, and so do most cutscene fades in `field/special.asm`. The codes
set the two `bne` offsets to 0, so the brightness steps every frame and
every fade takes 16 frames.

**Evidence:** The `@xxxx` labels here don't match the USA ROM. In
`ff2_original.smc`, `FadeIn` is at `$00:8B92` and `FadeOut` at
`$00:8BB3`. `ff2_original.smc` has `25 82 D0 F0` at `$00:8BA4` and
`25 82 D0 E4` at `$00:8BD3`. `lda #$07 : jsr FadeIn` (`A9 07 20 92 8B`)
is at `$00:D231` and `$00:E7C8`, and the same for `FadeOut` at `$00:D24C`
and `$00:E7D0`. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Fades are not instant: each still has 16 steps of one frame each. A
  fade with rate 0 (the menu and treasure fades) doesn't change.
- Fading in after a map change is done in the NMI handler (`$ca`), one
  step per frame. It was already fast and doesn't change.
- Battle and menu screens have their own fades, which don't change.
- Scripted scenes wait for each fade to end, so they move on sooner.
  Music and sound effects keep their own timing.

## Cecil starts with the Excalibur equipped

**Code:** `F8CD-ADE3`
**Raw form:** `0FAB03:1B` (orig `$16`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 09:52:37

Tags: `demo-ready` `curated`

**Mechanism:** This is the FF2 version of the ALTTP "Link's uncle gives
the Master Sword" and Zelda 1 Magical Sword codes. A new game sets up
Cecil with `InitCharProp` (`field/field.asm`), which calls
`InitMainCharEquip` (`field/event.asm`). That copies 7 bytes from
`CharInitEquip` entry 0 (helmet, armor, accessory, right hand, right
hand count, left hand, left hand count) to `$1030`:

```
@e905:  lda     f:CharInitEquip,x   ; initial character equipment
        sta     $1030,y
```

Cecil's right hand holds item `$16`, the Shadow sword (attack 10). The
code changes it to item `$1B`, "Excalbur" (attack 160). The battle code
(`UpdateCharStats`, `LoadEquipProp`) reads the weapon's stats from
`EquipProp` and doesn't check who can equip it.

**Evidence:** `ff2_original.smc` has `6E 82 9D 16 01 62 01` at
`$0F:AB00` (`char_init_equip` starts at `$0FAB00` in `ff4-en.lnk`
through `field/field_data.asm`). `BF 00 AB 0F 99 30 10`
(`lda f:CharInitEquip,x : sta $1030,y`) is at `$00:E67D`. The item names
at `$0F:8000` give `$16` = "Shadow" and `$1B` = "Excalbur".
`EquipProp` (`$0F:9100`, 8 bytes per item) has attack `$0A` for `$16`
and `$A0` for `$1B`. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only a new game changes. An old save keeps its weapon.
- `CharInitEquip` entry 0 is read again if an event re-adds the Dark
  Knight Cecil with fresh equipment. He would get the Excalbur again.
- A Dark Knight can't equip the Excalbur in the menu. If you take it
  off, you can't put it back on until Cecil is a Paladin.
- Excalbur's other stats (holy element, stat bonus) come with it.

## Cecil starts with 9,928 HP

**Codes:** `41CD-0473` + `41CD-0483` (use both)
**Raw form:** `0FA908:26` (orig `$00`) + `0FA90A:26` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, Kain too:** `41C4-0473` + `41C4-0483`
(`0FA928:26` + `0FA92A:26`, both orig `$00`)
**Recorded:** 2026-10-24 09:52:38

Tags: `demo-ready` `curated`

**Mechanism:** This is the FF2 version of the Zelda 1 "second quest
grants 16 heart containers" code. `CharProp` (`$0FA900`, 32 bytes per
character) holds each character's starting stats. Bytes 7-8 are current
HP and bytes 9-10 are max HP. A new game copies Cecil's entry (entry 0)
to `$1000` in `InitCharProp`:

```
@94f4:  lda     f:CharProp,x
        sta     $1000,y
```

Cecil starts with 200/200 HP (`$00C8`). The codes change the high
bytes of current and max HP from `$00` to `$26`, so he has 9,928/9,928
HP (`$26C8`). Kain is entry 1, which `LoadCharProp` (`field/event.asm`)
copies when he joins. He has 190 HP (`$00BE`), so the optional codes
give him 9,918.

**Evidence:** `ff2_original.smc` has `81 00 0A 00 00 00 00 C8 00 C8 00`
at `$0F:A900` and `42 01 0A 00 00 00 00 BE 00 BE 00` at `$0F:A920`
(`char_prop` starts at `$0FA900` in `ff4-en.lnk`). The byte layout is
from `ff4-en-rip.json` (`CharProp`: current HP at byte 7, max HP at byte
9) and the WRAM map (`$1007` current HP, `$1009` max HP).
`BF 00 A9 0F 99 00 10` (`lda f:CharProp,x : sta $1000,y`) is at
`$00:92D6` (`InitCharProp`) and `$00:E623` (`LoadCharProp`). The codes
decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only a new game changes. An old save keeps its HP.
- Both codes are needed. With only the max HP code, Cecil starts with
  200/9,928 HP.
- Level-ups add HP on top, up to the 9,999 limit.
- Cecil's other stats, level and attack don't change.

## Winning a battle sets your gil to 9,999,999

**Code:** `DD36-0F6E`
**Raw form:** `03ED86:00` (orig `$0F`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, for treasure and event gil too:** `D3BE-A40F`
(`0097F9:0E`, orig `$1D`)
**Recorded:** 2026-10-24 09:53:50

Tags: `demo-ready` `curated`

**Mechanism:** This is the FF2 version of the Zelda 1 "Collect any
Rupies, get 255" code. After a won battle, `WinUpdate`
(`battle/win.asm`) adds the battle's gil (`$3594`) to your gil
(`$16a0-$16a2`) and then caps it at 9,999,999 (`$98967F`):

```
        sec
        lda     $16a0
        sbc     #$7f
        lda     $16a1
        sbc     #$96
        lda     $16a2
        sbc     #$98
        bcc     @edb0
        lda     #$7f
        sta     $16a0
        ...
```

The code sets the `bcc` offset to 0, so the cap always runs and every
won battle leaves you with 9,999,999 gil.

The optional code does the same for gil from treasure chests and events
(event command `$e5`). Both call `GiveGil` (`field/trigger.asm`),
which checks the cap one byte at a time:

```
        cmp     #$98
        bcc     @9a35
        ...
        lda     #$7f
        sta     $16a0
```

The optional code points the first `bcc` at the `lda #$7f` instead of
the `rts`. Then any gil under 9,961,472 (`$980000`) goes to the cap. Gil
above that still goes through the normal checks.

**Evidence:** The `@xxxx` labels here don't match the USA ROM.
`ff2_original.smc` has
`38 AD A0 16 E9 7F AD A1 16 E9 96 AD A2 16 E9 98 90 0F A9 7F` at
`$03:ED75` (the `bcc` offset is at `$03:ED86`). This pattern occurs only
once. `GiveGil` is at `$00:97DD`. It has `C9 98 90 1D` at `$00:97F6`
(the `bcc` offset is at `$00:97F9`) and `A9 7F 8D A0 16` at `$00:9808`,
which is `$00:97FA` + `$0E`. The codes decode back to their raw forms
with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The optional code was not tested.
- The victory message still shows the normal gil for the battle. The
  total in the menu is 9,999,999.
- Running away gives no gil and doesn't change it.
- Shops and inns still take gil, but you can't run out after one more
  battle.

## No music

**Code:** `1D66-0D2D`
**Raw form:** `048183:60` (orig `$8D`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 09:54:31

Tags: `demo-ready`

**Mechanism:** This is the FF2 version of the "no music" codes for
Metroid, Zelda 1, SMB1, SMW, ALTTP and Super Metroid. Every part of the
game sends sound commands through `ExecSound` (`sound/sound.asm`). It
reads the command at `$1e00`: commands 1, 3 and 4 call `PlaySong`, 2
plays a sound effect, `$10-$1E` play a system sound effect, and `$80`
and up are sent to the SPC as is. `PlaySong` starts with:

```
PlaySong:
@816f:  shorta
        sta     $08
        sta     $04
        lda     #$00
        xba
        lda     $01
        cmp     $05
        bne     @817f                   ; branch if song isn't playing
        rts
@817f:  shorta
        sta     $05
        sta     hWRMPYA                 ; multiply by 3
```

After that it sends the song script (and its samples) to the SPC. The
code changes the `sta hWRMPYA` opcode (`$8D`) to `rts` (`$60`). The
game still records the command (`$1e04`) and the song number (`$1e05`)
as if the song were playing, but no song is sent. `ExecSound` restores
the registers with `plp` after the call.

The `rts` has to come after `sta $05`. `PlayBattleSong`
(`battle/battle.asm`) sends the battle song and then loops until `$1e05`
holds that song number and `$1e04` is 1. With an `rts` at the start of
`PlaySong`, every battle would hang there.

**Evidence:** `ff2_original.smc` has
`E2 20 85 08 85 04 A9 00 EB A5 01 C5 05 D0 01 60 E2 20 85 05 8D 02 42`
at `$04:816F`, which matches `PlaySong` and its label (`sound_code` is
in bank `$04` in `ff4-en.lnk`). The `sta hWRMPYA` is at `$04:8183`.
`C9 01 D0 05 20 6F 81` (`cmp #$01 : bne : jsr PlaySong`) is at
`$04:8122`. The three `jsr PlaySong` in `ExecSound` are its only
callers. `PlayBattleSong`'s test
(`AD 05 1E C5 A9 D0 .. AD 04 1E C9 01 D0`) is at `$13:FF2B`. The code
decodes back to its raw form with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Sound effects go through `PlayGameSfx` and `PlaySystemSfx`, which
  don't change. If some sound effects use samples that only come with a
  song, they may be silent or sound wrong.
- An earlier version of this entry used `1D61-072D` (`04816F:60`). That
  code would hang the game at the start of each battle. Don't use it.

## Cecil walks 4 times as fast

**Codes:** `D7CE-6DD4` + `D0CE-6FD4` (use both)
**Raw form:** `00AAF0:03` (orig `$0F`) + `00AAF4:04` (orig `$01`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 09:55:24

Tags: `demo-ready` `curated`

**Mechanism:** This is the FF2 version of the Zelda 1, ALTTP and SMW
"walk faster" codes. The party's movement speed `$ac` is an index into
two tables in `field/move.asm`:

```
; trigger check frequency for each movement speed
TriggerRateTbl:
@ad0e:  .byte   $0F,$07,$03,$01

; player movement speeds (pixels per frame)
PlayerMoveRateTbl:
@ad12:  .byte   $01,$02,$04,$08
```

Walking on foot is index 0: 1 pixel per frame, with a trigger check every
16 frames (once per tile). The codes change entry 0 of both tables to the
values of entry 2: 4 pixels per frame, with a trigger check every 4 frames.
A 16-pixel step then takes 4 frames instead of 16 and the trigger checks
still run once per tile.

An earlier version of this entry changed `PlayerSpeedTbl` (`$00:830C`)
instead. `UpdatePlayerSpeed` reads that table only on a new game and after
a battle, and `LoadSubMap` clears `$ac` on every town, castle or cave map.
The speed was lost when entering a town and stayed lost on the world map
until the next battle. The tables above are read on every step, so this
version doesn't lose it.

**Evidence:** The `@xxxx` labels here don't match the USA ROM.
`ff2_original.smc` has `0F 07 03 01 01 02 04 08` at `$00:AAF0`, and the
pattern occurs only once. `TriggerRateTbl` is read at `trigger.asm` `@9bd7`
and `@9d3c`, and `PlayerMoveRateTbl` six times in `move.asm`, all indexed by
`$ac`. The codes decode back to their raw forms with `snes_gg_encode.py`.
The old `PlayerSpeedTbl` version was 4 times as fast on the world map and
normal in a town and after leaving it. This version was tested in Mesen2
together with the 4 codes of "Skip the opening and start outside Baron": Cecil
stays 4 times as fast in Baron and on the world map after leaving it.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- It works on foot on every map. Vehicles use other table entries and keep
  their own speeds.
- Event command `$D7` (fast player movement) sets index 1, which isn't
  changed.
- Speed 3 (8 pixels per frame, the airship speed) would be 8 times as fast. It
  wasn't chosen because walking indoors at airship speed is hard to control.
- Random battles come per step, so they come 4 times as often per
  second.

## No random battles

**Code:** `6D64-0D04`
**Raw form:** `008921:80` (orig `$D0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 09:56:42

Tags: `demo-ready`

**Mechanism:** This is the FF2 version of the Zelda 1 "Prevent Zoras
from appearing" code. The source has a `NO_RAND_BATTLES` option that
replaces both `jsr CheckBattle` calls in `MainLoop` with `nop3`.
`CheckBattle` (`field/rand_battle.asm`) starts with:

```
CheckBattle:
@8b3c:  lda     $1704
        bne     @8b56
        lda     $a2
        and     #$40
        beq     @8b56
```

`$1704` is the vehicle id. On foot it is 0, and the check goes on to the
tile's "enable random battles" bit, the step counters and the random
number test. In a vehicle the `bne` jumps to `@8b56`, which ends the
check with no battle. The code changes the `bne` (`$D0`) to `bra`
(`$80`), so every step ends there. This is one code where
`NO_RAND_BATTLES` changes two `jsr`s.

**Evidence:** The `@xxxx` labels here don't match the USA ROM.
`ff2_original.smc` has `AD 04 17 D0 15 A5 A2 29 40 F0 0F` at `$00:891E`.
The pattern occurs only once. `20 1E 89` (`jsr CheckBattle`) is at
`$00:80E2` (world map) and `$00:8142` (sub-map). The branch target
`$00:8938` is `lda $c0 : beq`, then `rts`. The code decodes back to its
raw form with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Scripted battles (bosses, event battles) and battles from treasure
  chests and NPCs don't use `CheckBattle`, so they still happen.
- The Alarm item (`$c0`, "used siren/alert item") no longer starts a
  battle. `@8b56` clears it and plays a sound effect.
- You get no experience from random battles, so the party stays at a low
  level.

## A random battle on every step

**Code:** `6D6B-0D04`
**Raw form:** `008991:80` (orig `$90`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, for caves and dungeons too:** `6D69-6764`
(`008A5E:80`, orig `$90`)
**Recorded:** 2026-10-24 09:56:43

Tags: `demo-ready` `challenge`

**Mechanism:** This is the FF2 version of the Zelda 1 "More overworld
enemies spawn per screen" code. On each step on a tile with the "enable
random battles" bit, `CheckBattle` (`field/rand_battle.asm`) compares a
random number with the area's battle rate. On the world map:

```
        lda     $86
        tax
        lda     f:RNGTbl,x   ; rng table
        clc
        adc     $17ef
        cmp     $06
        bcc     @8bb2
        rts
@8bb2:  phy
        plx
        lda     f:OverworldBattleGrp,x
```

The world map rates (`OverworldBattleRate`, `$0EC300`) are 7 to 10 out
of 256, so a battle comes about once in 25-35 steps. The code changes
the `bcc` (`$90`) to `bra` (`$80`), so every step starts a battle with a
group from the area's normal list. The optional code does the same for
sub-maps (`cmp $06 : bcc @8c91`). Sub-maps with a rate of 0 (towns,
castles) branch away before that test, so they still have no battles.

**Evidence:** The `@xxxx` labels here don't match the USA ROM.
`ff2_original.smc` has `BF 00 EE 14 18 6D EF 17 C5 06 90 01 60` at
`$00:8987` (the `bcc` is at `$00:8991`) and
`BF 42 C3 0E F0 15 85 06 A5 C0 D0 22 A5 86 AA BF 00 EE 14 18 6D EF 17
C5 06 90 13` at `$00:8A45` (the `bcc` is at `$00:8A5E`). The
`OverworldBattleRate` bytes at `$0E:C300` are all `$07`, `$08` or `$0A`.
The codes decode back to their raw forms with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron" and 7 other cheats in one test group, without the
optional code.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The optional code was not tested.
- Only tiles that allow random battles (the tile's `$40` bit) and only
  steps on foot count. In a vehicle there are still no battles.
- The underground and the moon have their own tests, which don't
  change.
- After each battle you have to take one more step to start the next
  one. Running away doesn't stop the next one.

## Cecil walks through walls in towns, castles and caves

**Code:** `DDC1-DF04`
**Raw form:** `00A865:00` (orig `$0D`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 09:58:26

**Mechanism:** This is the FF2 version of the walk-through-walls codes
for Metroid, Zelda 1, ALTTP and Super Metroid. On a sub-map,
`CheckPlayerMoveSub` (`field/move.asm`) calls `CheckTilePass` for the
pressed direction. It returns 0 if Cecil can move and 1 if not:

```
@aa7d:  lda     $06a1,y
        and     $0a
        beq     @aa91
        lda     #$00
        rts
...
@aa91:  lda     #$01
        rts
```

`$06a1,y` is the passability of the tile in that direction and `$0a` is
the bit for Cecil's current z-level. A wall has no bit for that level,
so the `beq` returns 1. The code sets the `beq` offset to 0, so every
tile returns 0. NPCs are checked before this (`jsr CheckNPCBlock`), so
they still block Cecil.

**Evidence:** The `@xxxx` labels here don't match the USA ROM.
`CheckTilePass` is at `$00:A83A`. `ff2_original.smc` has
`B9 A1 06 25 0A F0 0D A9 00 60` at `$00:A85F` (the `beq` offset is at
`$00:A865`) and `A9 01 60` at `$00:A873`. The pattern occurs only once.
`CheckTilePass` is called only from `CheckPlayerMoveSub`. The code
decodes back to its raw form with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron" and the other walking cheat.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only sub-maps change. For the world map, see "Walk on water and mountains
  on the world map".
- On bridge tiles (`$a1` bit 2) the game uses another test, which
  doesn't change. Walls next to a bridge still block.
- You can walk off the map into empty space, or into places where an
  event expects you not to be. Triggers and exits still work when you
  step on them.

## Cecil walks through people and other NPCs

**Code:** `6DCC-D7A4`
**Raw form:** `00A8AF:80` (orig `$10`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 09:58:50

**Mechanism:** This is the FF2 version of the Zelda 1 "Walk right
through shopkeepers and old men" code. Before the tile test,
`CheckTilePass` (`field/move.asm`) calls `CheckNPCBlock`. It looks up
the tile in front of Cecil in the NPC map at `$7F4C00`:

```
        jsr     GetNPCMapPtr
        ldx     $3d
        lda     $7f4c00,x
        bpl     @aad4       ; branch if tile is empty
        sta     $ee
        jmp     @aad6
@aad4:  lda     #$00
@aad6:  plx
        rts
```

If an NPC is there, it returns the NPC id with the high bit set, and
`CheckTilePass` returns 1 (blocked). The code changes the `bpl` (`$10`)
to `bra` (`$80`), so the tile always reads as empty.

**Evidence:** The `@xxxx` labels here don't match the USA ROM.
`CheckNPCBlock` is at `$00:A876`. `ff2_original.smc` has
`BF 00 4C 7F 10 05 85 EE` at `$00:A8AB` (the `bpl` is at `$00:A8AF`).
The pattern occurs only once. `CheckNPCBlock` is called only from
`CheckTilePass`. `$ee` is "npc blocking player movement" in the WRAM
map. `GetNPCPtr` (`field/npc.asm`) uses it only to stop that NPC from
walking. The code decodes back to its raw form with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron" and the other walking cheat.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only sub-maps (towns, castles, caves) have NPCs, so only they change.
- Talking to an NPC still works. Face the NPC and press A, as usual.
- Cecil can stand on the same tile as an NPC. An NPC that walks into
  Cecil may stop there or walk through him.
- Walls still block. See "Cecil walks through walls in towns, castles
  and caves".

## Walk on water and mountains on the world map

**Code:** `DDC7-67A4`
**Raw form:** `00AA3F:00` (orig `$07`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 09:59:30

Tags: `demo-ready` `curated`

**Mechanism:** This is the FF2 version of the Zelda 1 and ALTTP "walk on
water" codes. On the world map, `CheckPlayerMoveWorld`
(`field/move.asm`) tests the tile in the pressed direction. In a vehicle
it uses `VehiclePassBit`. On foot it uses:

```
@ac56:  lda     $a1,x
        and     #$41                  ; check passability (no vehicle)
        and     $d2
        beq     @ac65
@ac5e:  jsr     CheckVehicleBlock
        lda     $0a
        beq     @ac68       ; branch if not blocked
@ac65:  stz     $ab
        rts
```

`$d2` holds the same two bits (`$41`) of the tile Cecil stands on. If
the next tile shares none of them (water, mountains, and so on), the
`beq` stops him. The code sets the `beq` offset to 0, so on foot every
tile passes. `CheckVehicleBlock` returns at once when Cecil is on foot.

**Evidence:** The `@xxxx` labels here don't match the USA ROM.
`ff2_original.smc` has `B5 A1 29 41 25 D2 F0 07 20 5F AA A5 0A F0 03 64
AB 60` at `$00:AA38` (the `beq` offset is at `$00:AA3F`). The pattern
occurs only once. The vehicle test just before it
(`and VehiclePassBit,y : beq`, at `$00:AA30`) is a separate branch. The
code decodes back to its raw form with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron" and 7 other cheats in one test group.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only walking on the world map, the underground and the moon changes.
  Vehicles keep their own rules.
- Random battles only come on tiles with the "enable random battles"
  bit, so walking on water or mountains may give few or no battles.
- Walking onto a town, cave or castle tile still enters it.

## The party's weapon attacks do 4 times the damage

**Code:** `DDAB-6FD2`
**Raw form:** `03C694:00` (orig `$18`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, add for 20 times the damage:** `46AB-6F62`
(`03C696:28`, orig `$08`)
**Recorded:** 2026-10-24 10:01:12

Tags: `demo-ready` `curated`

**Mechanism:** This is the FF2 version of the Zelda 1, ALTTP and Super
Metroid (Power Beam) "more damage" codes. `DoFightCmd`
(`battle/fight.asm`) sets a creature type damage multiplier `$38ff`.
For a character hitting a monster, it is 4x when the weapon is strong
against the monster's type (a holy weapon on an undead monster, for
example):

```
; character vs. monster
        lda     $269a       ; attack strong vs. creature type
        and     $2740       ; creature type
        beq     @c6ad
        lda     #$08        ; 4x creature type multiplier
        sta     $38ff
```

The code sets the `beq` offset to 0, so every character attack on a
monster gets the 4x multiplier. `ApplyDmgMult` (`battle/damage.asm`)
multiplies the damage by the value / 2 (`$02` is 1x). The optional code
changes `#$08` to `#$28`, which is 20x. Damage is still capped at 9,999.

**Evidence:** The labels here match the USA ROM. `ff2_original.smc` has
`A5 CD 30 13 A5 CE 10 20 AD 9A 26 2D 40 27 F0 18 A9 08 8D FF 38` at
`$03:C685` (the `beq` offset is at `$03:C694` and the `#$08` at
`$03:C696`). The pattern occurs only once. `DoFightCmd` is also called
by the Focus, Aim and Jump commands (`battle/cmd.asm`). The codes decode
back to their raw forms with `snes_gg_encode.py`.
Tested in Mesen2 together with the 3 codes of "Skip the opening and start
outside Baron" and the 2 codes of "New game: Rosa is in the party".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only Fight and the commands that use it (Jump, Aim, Focus) change.
  Spells, items and monster attacks don't.
- `ApplyDmgMult` keeps only 16 bits of the product. 20x is the most that
  can't overflow: the highest damage before this step is 765 x 4
  (elemental) = 3,060.
- With 20x, nearly every monster early in the game dies in one hit.
  Monsters with more than 9,999 HP don't.

## Enemy weapon attacks always miss the party

**Code:** `25A4-6F62`
**Raw form:** `03C626:D7` (orig `$05`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 10:01:42

Tags: `demo-ready` `curated`

**Mechanism:** This is the FF2 version of the "enemies never hurt the
player" codes for Metroid, SMB3, ALTTP and Super Metroid. `DoFightCmd`
(`battle/fight.asm`) works out the net number of hits (attack hits
minus hits defended). With no net hits it goes to `@c5fe`, which sets
the miss flag. With one or more, it checks the target:

```
; nonzero net hits
@c623:  lda     $ce
        bpl     @c62c       ; branch if target is a character
        lda     $3881
        bne     @c5fe       ; branch if monsters are invincible ???
@c62c:  ...
```

The code changes the `bpl` offset from `$05` (`@c62c`) to `$D7`
(`@c5fe`), so every Fight attack on a character counts as zero hits and
misses.

**Evidence:** The labels here match the USA ROM. `ff2_original.smc` has
`A5 CE 10 05 AD 81 38 D0 D2` at `$03:C623` (the `bpl` offset is at
`$03:C626`) and `7B 8D FC 38 1A 8D C8 34` (`clr_a : sta $38fc : inc :
sta $34c8`) at `$03:C5FE`. `$03:C627` - `$29` = `$03:C5FE`. The pattern
occurs only once. The code decodes back to its raw form with
`snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron" and 7 other cheats in one test group.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only Fight attacks miss. Enemy spells and special attacks still hurt
  the party.
- A Fight attack that misses can't cause a status effect, so enemy
  Fight attacks with poison and the like stop working too.
- A confused party member's Fight attack on another party member also
  misses.

## Enemy weapon attacks do only 1 damage to the party

**Codes:** `DDAC-6FA2` + `DDAC-6402` (use both)
**Raw form:** `03C6A7:00` (orig `$05`) + `03C6A9:00` (orig `$01`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 10:01:56

Tags: `demo-ready`

**Mechanism:** This is the FF2 version of the Metroid and Super Metroid
"enemies barely hurt the player" codes. `DoFightCmd`
(`battle/fight.asm`) sets a creature type damage multiplier `$38ff`.
For a monster hitting a character, it is 1/2x when the character's armor
is strong against the monster's type:

```
; monster vs. character
        lda     $26c0
        and     $2727       ; defense strong vs. creature type
        beq     @c6ad
        lda     #$01        ; 1/2x creature type multiplier
        sta     $38ff
```

The first code sets the `beq` offset to 0, so every monster Fight
attack on a character gets the multiplier. The second changes it from
`#$01` to `#$00`. `ApplyDmgMult` (`battle/damage.asm`) sets the damage
to 0 for a multiplier of 0. At the end, `CalcDmg` raises a damage of 0
to 1 (`ldx $a4 : bne : inc $a4`).

**Evidence:** The labels here match the USA ROM. `ff2_original.smc` has
`A5 CE 30 0D AD C0 26 2D 27 27 F0 05 A9 01 8D FF 38` at `$03:C69C` (the
`beq` offset is at `$03:C6A7` and the `#$01` at `$03:C6A9`). The pattern
occurs only once. The codes decode back to their raw forms with
`snes_gg_encode.py`.
Tested in Mesen2 together with the 3 codes of "Skip the opening and start
outside Baron" and the 2 codes of "New game: Rosa is in the party".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only Fight attacks change. Enemy spells and special attacks do full
  damage.
- A critical hit adds its bonus after the multiplier, so an enemy
  critical can still do more than 1 damage.
- With only the first code, enemy Fight attacks do half damage.
- Don't use this with "The party takes double damage from enemy weapon
  attacks". Both change `$03:C6A9`.

## The party takes double damage from enemy weapon attacks

**Codes:** `DDAC-6FA2` + `D0AC-6402` (use both)
**Raw form:** `03C6A7:00` (orig `$05`) + `03C6A9:04` (orig `$01`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 10:02:11

Tags: `challenge`

**Mechanism:** This is the FF2 version of the Super Metroid "Samus
always takes double damage" code. It changes the same lines as "Enemy
weapon attacks do only 1 damage to the party". `DoFightCmd`
(`battle/fight.asm`) sets the creature type multiplier `$38ff` to 1/2x
(`#$01`) when a character's armor is strong against the monster's type:

```
; monster vs. character
        lda     $26c0
        and     $2727       ; defense strong vs. creature type
        beq     @c6ad
        lda     #$01        ; 1/2x creature type multiplier
        sta     $38ff
```

The first code sets the `beq` offset to 0, so every monster Fight attack
on a character gets the multiplier. The second changes it to `#$04`,
which `ApplyDmgMult` (`battle/damage.asm`) treats as 2x.

**Evidence:** The labels here match the USA ROM. `ff2_original.smc` has
`A5 CE 30 0D AD C0 26 2D 27 27 F0 05 A9 01 8D FF 38` at `$03:C69C` (the
`beq` offset is at `$03:C6A7` and the `#$01` at `$03:C6A9`). `#$04` is
the value `DoFightCmd` uses for 2x elemental damage. The codes decode
back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Only Fight attacks change. Enemy spells and special attacks do normal
  damage.
- The party's defense is taken off after the multiplier, so the damage
  you see is often more than double.
- Don't use this with "Enemy weapon attacks do only 1 damage to the
  party". Both change `$03:C6A9`.

## Run away the moment a battle starts

**Code:** `BA66-676A`
**Raw form:** `03828E:9C` (orig `$8D`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 10:03:56

**Mechanism:** To run, you hold L and R. `CheckBattleEnd`
(`battle/battle.asm`) checks the run counter `$38d6` each time:

```
@a08d:  lda     $38d6
        cmp     #$ff
        beq     @a0f6
        lda     $38d6
        beq     @a0a3
        lda     $38f3
        bne     @a0a3       ; branch if used escape ???
        dec     $38d6
        bra     @a0f6
@a0a3:  lda     $38e5
        and     #$01
        bne     @a0de       ; branch if can't run
```

`InitRAM` sets `$38d6` to `$ff` at the start of every battle, and `$ff`
means you can't run yet. It becomes 0 only when the first character's
turn comes up (`GetCharAttack`, `battle/char.asm`). So you have to wait
for the first ATB gauge to fill before L and R work. The code changes
the `sta $38d6` in `InitRAM` to `stz $38d6` (`$8D` to `$9C`), so the
counter starts at 0. Holding L and R then runs away at once.

**Evidence:** The labels here match the USA ROM. `ff2_original.smc` has
`8D 01 36 8D 02 36 8D D6 38 A9 1A` at `$03:8288`, the end of the
`lda #$ff` stores in `InitRAM` (the `sta $38d6` is at `$03:828E`).
`AD D6 38 C9 FF F0 .. AD D6 38 F0` is at `$03:A08D`. `stz $38d6`
(`9C D6 38`) is already used at `$03:A396`. The code decodes back to its
raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Battles you can't run from (bosses, `$38e5` bit 0) don't change.
- Monsters of level 97-99 set their own run counter (3, 7 or 10, from
  `MonsterRunTbl`), so running from them still takes a while.
- The first character who isn't dead, stone, asleep, paralyzed, stopped
  or jumping has to be able to run, as before.

## Every battle starts with a first strike for the party

**Codes:** `DD6F-046E` + `DD6F-07DE` (use both)
**Raw form:** `038D1A:00` (orig `$29`) + `038D1C:00` (orig `$08`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 14:42:14

**Mechanism:** At the start of a battle, `battle/init.asm` compares the
party's average agility (`$38d4`) with the monsters' (`$38d5`). Each
side rolls against its value, which leaves `$a9` (party) and `$aa`
(monsters) at 1 or 2:

```
@8d15:  lda     $a9
        cmp     $aa
        beq     @8d44
        bcc     @8d25
        inc     $38d7
        inc     $38d8
        bra     @8d44
@8d25:  lda     #$80
        sta     $38d7
        sta     $38d8
```

A tie is a normal battle, a higher party roll is a first strike
(`$38d7` = 1, "first strike chance!"), and a lower one is a surprise
attack (`$38d7` = `$80`) or a back attack. The codes set both branch
offsets to 0, so every roll ends in the first strike path.

**Evidence:** The `@xxxx` labels here don't match the USA ROM.
`ff2_original.smc` has `A5 A9 C5 AA F0 29 90 08 EE D7 38 EE D8 38` at
`$03:8D15` (the offsets are at `$03:8D1A` and `$03:8D1C`). The pattern
occurs only once. `CalcTimerDur` and the code after `@8e1d` read `$38d8`
to give the party a head start. The codes decode back to their raw forms
with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Battles with a set back attack, boss battles and battles you can't
  run from skip this test (`@8c66`-`@8c84`), so they don't change.
- No random battle is a surprise or back attack.

## Every battle is a back attack

**Code:** `CB6F-0FAA`
**Raw form:** `038117:A9` (orig `$29`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 14:42:27

Tags: `demo-ready` `challenge`

**Mechanism:** When a battle starts, `battle/battle.asm` loads its 8
bytes of `BattleProp`. Bit 3 of the first byte forces a back attack:

```
        lda     $299c
        sta     $29a4
        and     #$08
        sta     $3581                   ; forced back attack if set
```

The code changes `and #$08` (`$29`) to `lda #$08` (`$A9`), so `$3581`
is always `$08`. `battle/init.asm` then takes the forced back attack
path (`@8c66`): the monsters get the head start (`$38d8` = `$80`), the
party's rows are swapped, and the "back attack!" message shows.

**Evidence:** `ff2_original.smc` has `AD 9C 29 8D A4 29 29 08 8D 81 35`
at `$03:8111` (the `and` opcode is at `$03:8117`). The pattern occurs
only once. `AD 81 35 29 08 F0 08 A9 80 8D D8 38` (the forced back attack
test) is at `$03:8C66`. The code decodes back to its raw form with
`snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron" and 7 other cheats in one test group.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Boss and event battles are back attacks too.
- The party's front and back rows are swapped for the battle, so Cecil
  and Kain fight from the back row. `DoFightCmd` halves the hit rate of
  a back row attacker unless the weapon has no back row penalty.
- Don't use this with "Every battle starts with a first strike for the
  party". A forced back attack skips that test.

## Enemies never get a turn

**Code:** `6D8E-AFAA` + `478E-A4DA`
**Raw form:** `03B3F7:80` (orig `$D0`), `03B3F8:23` (orig `$29`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 14:45:23

**Mechanism:** When a monster's action timer runs out, `InitAction`
picks action type 1, and `GetMonsterAttack` (`battle/ai.asm`) chooses
the monster's command. It first sets the command to `$e1` (do nothing),
then tests for berserk or confusion:

```
        lda     #$e1
        sta     $2051,x
        lda     $2004,x
        and     #$0c
        bne     @b422
```

If the monster isn't berserk or confused, the code goes on to the
"monster runs away" test, and then runs the monster's AI script. The
codes change `bne @b422` to `bra` to the end of the run away test:

```
        jsr     InitAIDelay
        jmp     @b68c
```

So every monster keeps the "do nothing" command, `InitAIDelay` restarts
its action timer, and its AI script never runs.

**Evidence:** `ff2_original.smc` has `A9 E1 9D 51 20 BD 04 20 29 0C D0
29` at `$03:B3ED` (the branch is at `$03:B3F7`). The pattern occurs only
once. `20 DE BB 4C 8C B6` (`jsr InitAIDelay`, `jmp @b68c`) is at
`$03:B41C`, `$23` bytes after the end of the branch. `InitAIDelay` is at
`$03:BBDE`. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Monsters never run away either.
- Counterattacks don't go through `GetMonsterAttack`, so a monster may
  still counter a hit.
- Boss and event battles that end through the monster's AI script (a
  boss that changes form, or a battle you are meant to lose) may never
  end.

## Cecil's turns come up 16 times as fast

**Code:** `DFCF-04E3`
**Raw form:** `0FA91B:01` (orig `$10`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, Kain too:** `DFC7-04E3`
(`0FA93B:01`, orig `$10`)
**Recorded:** 2026-10-24 14:47:16

**Mechanism:** This is the FF2 version of the Metroid "no delay before
auto-fire starts" code. Byte 27 of each character's `CharProp` entry is
the speed multiplier. A new game copies bytes 23-31 of Cecil's entry to
`$1037` in `InitCharProp`, so byte 27 lands in `$103B`:

```
@9516:  lda     f:CharProp,x
        sta     $1037,y
```

`LoadCharProp` (`battle/init.asm`) copies the 64-byte record to `$2000`
when a battle starts. `CalcTimerDur` reads the multiplier from `$203b`,
and `ApplySpeedMod` works out the timer length as:

```
duration = base duration * multiplier / 16
```

Every character starts with a multiplier of 16 (`$10`), so the base
duration is used as is. The code sets Cecil's to 1, so his action timer
and the delays after his commands are 1/16 as long. `TimerDur_00` keeps
the action timer at 1 tick or more. Kain is entry 1, which
`LoadCharProp` (`field/event.asm`) copies when he joins.

**Evidence:** `ff2_original.smc` has `B8 0B 00 00 10` at `$0F:A917`
(Cecil) and `FC 08 00 00 10` at `$0F:A937` (Kain). The RAM map lists
`$103B` as the "speed multiplier (haste: -3, slow: +8)".
`BF 00 A9 0F 99 37 10` (`InitCharProp`) is at `$00:92F8`, and
`BF 17 A9 0F 99 37 10` (the field `LoadCharProp`) is at `$00:E64D`.
`BD 3B 20` (`lda $203b,x`, the only reader in battle) is at `$03:9E45`.
The codes decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Only a new game changes. An old save keeps its speed.
- Cecil's turns don't come up at once. The game still waits at least 1
  tick, and the battle speed setting sets the length of a tick.
- Timers for his status effects (sleep, paralysis) use the same
  multiplier, so they wear off sooner.
- Haste and Slow change the multiplier to 12 or 9 (`MagicEffect_0f`
  keeps it between 12 and 32), so they slow Cecil down.

## Every random battle is the area's rarest enemy group

**Code:** `6D63-6FD4`
**Raw form:** `008AE4:80` (orig `$D0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 14:48:04

Tags: `demo-ready`

**Mechanism:** This is the FF2 version of the ALTTP "every enemy is
replaced by one enemy type" code. Each area has a list of 8 battle
groups. `ChooseRandBattle` (`field/rand_battle.asm`) rolls a random
number and picks a group by the table in its header: groups 0-3 come up
16.8% of the time each, and group 7 only 1.56%. If `$c0` is set, it uses
`$ff` instead of the roll, which always picks group 7:

```
        lda     $c0
        bne     @8d1f
        ...
@8d1f:  lda     #$ff
```

The code changes `bne` to `bra`, so every random battle, on the world
map and in caves and dungeons, uses group 7 of the area's list. The
battle ID is then read from `WorldBattles` or `SubBattles` as usual.

**Evidence:** The `@xxxx` labels here don't match the USA ROM.
`ff2_original.smc` has `A5 C0 D0 1B A5 87 AA BF 00 EE 14` at `$00:8AE2`
(the branch is at `$00:8AE4`). The pattern occurs only once.
`A9 FF A6 3D C9 2B` (`lda #$ff`, then the first compare) is at
`$00:8B01`, which the branch reaches. Menu event 6 sets `$c0`
(`field/field.asm`). The code decodes back to its raw form with
`snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The group differs from area to area. Within one area every battle is
  the same group.
- The rarest group is often harder than the others in the area.
- The random number generator for battles (`$87`) no longer moves
  forward.

## Any command can target all enemies or all allies (but most effects still only act on one)

**Code:** `DDCE-0708`
**Raw form:** `02ADFD:00` (orig `$04`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 14:50:17

**Mechanism:** When you pick a command, `SelectDefaultTarget`
(`btlgfx/cursor.asm`) reads its targeting flags. Bit 5 says whether it
can target all, and the result goes in `$ef8e`:

```
@adf9:  lda     $ef8c
        and     #$20
        beq     @ae04       ; branch if can't target all
        lda     #$01
        bra     @ae05
@ae04:  clr_a
@ae05:  sta     $ef8e
```

The code changes the `beq` offset from 4 to 0, so it never skips the
`lda #$01`. Every command can then target all. Moving the cursor past
the last monster selects all monsters, and moving it past the last
character selects all characters (`MoveTargetCursorBack` and the code
before it read `$ef8e`).

**Evidence:** The `@xxxx` labels here don't match the USA ROM.
`ff2_original.smc` has `AD 8C EF 29 20 F0 04 A9 01 80 01 7B 8D 8E EF`
at `$02:ADF7` (the offset is at `$02:ADFD`). The pattern occurs only
once. `btlgfx_code` is in bank `$02` in `ff4-en.lnk`. The code decodes
back to its raw form with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only the cursor changes. Spells and items act on every selected target
  (`SetTargets` sets the multi-target flag), but Fight and most other
  commands use only the first target (`$ce`). Confirmed in Mesen2: a Cure1
  potion could be spread over all enemies or all allies, but Fight and Jump
  hit only one enemy even with all targeted.
- Spells and items that target all may do less to each target, the same
  as spells that can already target all.

## Battle messages close after 17 frames

**Code:** `DDA0-07DC` + `DFED-D481` (use both)
**Raw form:** `02C94C:00` (orig `$04`) + `0DF80A:01` (orig `$C0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 14:51:27

**Mechanism:** This is the FF2 version of the SMW instant fades and the
Zelda 1 fastest text. `WaitMsg` (`btlgfx/btlgfx.asm`) waits 16 frames
after a battle message opens. Then, unless the message waits for a key,
`WaitMsgAuto` waits again for a time from `MsgDurTbl`, which the battle
message speed (`$16ad`, 0-5) picks:

```
WaitMsgAuto:
@c94a:  lda     $f49c
        beq     @c953
        lda     #$05        ; use slowest message speed (5)
        bra     @c956
@c953:  lda     $16ad       ; battle message speed
@c956:  asl
        tax
        longa
        lda     f:MsgDurTbl,x   ; battle message durations
```

`MsgDurTbl` (`btlgfx/btlgfx_data.asm`) is 32, 64, 96, 128, 160 and 192
frames. A new game sets the message speed to 2 (96 frames). The first
code changes the `beq` offset from 4 to 0, so the speed is always 5 for
any setting. The second code changes speed 5 from 192 frames (`$00C0`)
to 1 frame (`$0001`). Each message then closes after 16 + 1 frames,
about 0.3 seconds.

**Evidence:** The `@xxxx` labels here don't match the USA ROM.
`ff2_original.smc` has `AD 9C F4 F0 04 A9 05 80 03 AD AD 16 0A AA C2 20
BF 00 F8 0D` at `$02:C948` (the offset is at `$02:C94C`). The pattern
occurs only once. `20 00 40 00 60 00 80 00 A0 00 C0 00` (`MsgDurTbl`)
is at `$0D:F800`. `A9 00 ... 1A 8D AD 16` at `$01:9B50` sets the message
speed to 2 for a new game. `WaitX` decreases X before it tests it, so a
value of 0 would wait 65,536 frames; the code uses 1. The codes decode
back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Messages that wait for a button press don't change.
- Boss dialogue in battle (`$f49c` set) also closes after 17 frames, so
  it may be too fast to read.
- The message speed setting in the config menu has no effect.
- Attack animations don't change.

## Fight from the back row with no penalty

**Code:** `6DAF-0DA2`
**Raw form:** `03C513:80` (orig `$10`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 14:52:18

**Mechanism:** `DoFightCmd` (`battle/fight.asm`) halves the attacker's
hit rate if the attacker is in the back row, unless the weapon has no
back row penalty (bows, for example) or the attacker is jumping:

```
@c510:  lda     $2681
        bpl     @c526       ; branch if in front row
        lda     $2681
        and     #$20
        bne     @c526       ; branch if no back row penalty
        lda     $2685
        and     #$02
        bne     @c526       ; branch if jumping
        lsr     $38f8       ; halve hit rate
@c526:
```

The hit rate sets how many of the attacker's hits land (`CalcHits`), so
a back row attacker does about half the damage. The code changes `bpl`
to `bra`, so the hit rate is never halved for the attacker's row.

**Evidence:** `ff2_original.smc` has `AD 81 26 10 11 AD 81 26 29 20 D0
0A` at `$03:C510` (the branch is at `$03:C513`). The pattern occurs only
once. The code decodes back to its raw form with `snes_gg_encode.py`.
Tested in Mesen2 together with the 3 codes of "Skip the opening and start
outside Baron" and the 2 codes of "New game: Rosa is in the party".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Monsters attack through `DoFightCmd` too, so a monster's row doesn't
  matter either.
- The target's row still counts. A target in the back row halves the hit
  rate (`@c545`) and doubles its defense (`@c6f1`).
- This works well with "Every battle is a back attack", which swaps the
  party's rows.

## Each defeated monster gives 65,536 more experience

**Code:** `DF3B-D76E`
**Raw form:** `03EC9E:01` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 14:53:17

Tags: `demo-ready` `curated`

**Mechanism:** This is the FF2 version of the SMB1 "coins per 1-up:
every 5 coins instead of 100" code. After a won battle, `WinUpdate`
(`battle/win.asm`) adds each defeated monster's experience from
`MonsterXP` to a 3-byte total at `$3591`:

```
        clc
        lda     f:MonsterXP,x
        adc     $3591
        sta     $3591
        lda     f:MonsterXP+1,x
        adc     $3592
        sta     $3592
        lda     $3593
        adc     #$00
        sta     $3593
```

The code changes `adc #$00` to `adc #$01`, so each monster adds 65,536
more. `DivXP` splits the total among the characters who are still
standing, and the level up loop (`@ef06`) repeats until a character's
experience is below the next level, so Cecil and Kain gain many levels
after the first battle.

**Evidence:** The `@xxxx` labels here don't match the USA ROM.
`ff2_original.smc` has `BF C0 A1 0E 6D 91 35 8D 91 35 BF C1 A1 0E 6D 92
35 8D 92 35 AD 93 35 69 00 8D 93 35` at `$03:EC86` (the `adc` operand is
at `$03:EC9E`). `AD 93 35 69 00 8D 93 35` occurs only once. The level up
loop ends with `jmp @ef06` after each level. The code decodes back to
its raw form with `snes_gg_encode.py`. Tested in Mesen2 with a new game
and adult Rydia (the 7 codes of "New game: adult Rydia is in the party" and
"Skip the opening and start outside Baron"): the party gained many levels
after a battle.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only the level gain after a battle was tested.
- With two characters, each one gets at least 32,768 experience per
  monster.
- Experience stops at 9,999,999, and levels stop at 99.
- The "received experience" message may show only the low part of the
  number.

## Each defeated monster gives 65,536 more gil

**Code:** `DF3A-D46E`
**Raw form:** `03ECCA:01` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 14:53:32

Tags: `demo-ready` `curated`

**Mechanism:** This is the FF2 version of the SMW score values code.
After a won battle, `WinUpdate` (`battle/win.asm`) adds each defeated
monster's gil from `MonsterGil` to a 3-byte total at `$3594`:

```
        clc
        lda     f:MonsterGil,x
        adc     $3594
        sta     $3594
        lda     f:MonsterGil+1,x
        adc     $3595
        sta     $3595
        lda     $3596
        adc     #$00
        sta     $3596
```

The code changes `adc #$00` to `adc #$01`, so each monster adds 65,536
more gil. The total is then added to the party's gil at `$16a0`, which
stops at 9,999,999.

**Evidence:** The `@xxxx` labels here don't match the USA ROM.
`ff2_original.smc` has `BF 00 A0 0E 6D 94 35 8D 94 35 BF 01 A0 0E 6D 95
35 8D 95 35 AD 96 35 69 00 8D 96 35` at `$03:ECB2` (the `adc` operand is
at `$03:ECCA`). `AD 96 35 69 00 8D 96 35` occurs only once. The code
decodes back to its raw form with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron" and 7 other cheats in one test group.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- A battle with three monsters gives at least 196,608 gil. It takes
  about 153 monsters to reach 9,999,999.
- For the most gil at once, use "Winning a battle sets your gil to
  9,999,999" instead.
- Gil dropped when you run away doesn't change.

## Every defeated monster drops an item

**Code:** `CB32-D4DE`
**Raw form:** `03ECD8:A9` (orig `$29`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 14:54:17

Tags: `demo-ready` `curated`

**Mechanism:** This is the FF2 version of the Super Metroid "every killed
enemy drops an item" code. In `WinUpdate` (`battle/win.asm`), the top 2
bits of each monster's item byte (`$358e`) set its drop rate: `$c0`
always, `$40` 5%, `$80` 25%, and 0 never. The low 6 bits pick one of 64
item sets in `MonsterItems`:

```
        lda     $358e,x
        and     #$c0
        cmp     #$c0
        beq     @ed0f
```

The code changes `and #$c0` to `lda #$c0`, so every defeated monster
takes the "always" path. `@ed0f` then picks one of the set's 4 items
(about 50%, 31%, 17% and 1.6%) and adds it to the battle's spoils.

**Evidence:** The `@xxxx` labels here don't match the USA ROM.
`ff2_original.smc` has `BD 8E 35 29 C0 C9 C0 F0 17 C9 40` at `$03:ECD5`
(the `and` opcode is at `$03:ECD8`). The pattern occurs only once.
`BF 00 9F 0E 99 9C 28` (the `MonsterItems` read) is at `$03:ED00`, and
item set 0 at `$0E:9F00` is `CE CE CE CE` (Cure1). The code decodes
back to its raw form with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- A battle holds at most 8 items (`cpx #$0008`).
- Monsters that never drop items still use the item set in their low 6
  bits. Many of these may be set 0, which gives Cure1.
- Rare items stay rare. Only the drop itself is certain.

## Shops sell everything for 0 gil

**Code:** `DDAD-0709`
**Raw form:** `01C50D:00` (orig `$0A`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, for items that cost more than 1,270 gil:** `DDAA-DF09`
(`01C4C5:00`, orig `$FA`)
**Recorded:** 2026-10-24 14:55:17

Tags: `demo-ready` `curated`

**Mechanism:** This is the FF2 version of the Zelda 1 "buy shop items
regardless of Rupee count" code. `LoadShopList` (`menu/shop.asm`) works
out each item's price from its `ItemPrice` byte. If bit 7 is clear, the
price is the byte times 10 (up to 1,270 gil). If it is set, the price is
the low 7 bits times 1,000 (times 250, then times 4):

```
; 10x price multiplier
@c533:  sta     f:hWRMPYA
        lda     #$0a        ; multiply by 10
        sta     f:hWRMPYB
```

The code changes the 10 to 0, and the optional code changes the 250 to
0. The prices that `LoadShopList` stores at `$1b56` and shows in the
list are then 0. The buy code multiplies that price by the quantity and
takes it from your gil, so it takes nothing and the "not enough gil"
test always passes.

**Evidence:** The `@xxxx` labels here don't match the USA ROM.
`ff2_original.smc` has `BF 50 A4 0F 10 4A 29 7F 8F 02 42 00 A9 FA 8F 03
42 00` at `$01:C4B8` (the 250 is at `$01:C4C5`) and `8F 02 42 00 A9 0A
8F 03 42 00` at `$01:C508` (the 10 is at `$01:C50D`). The 10x pattern
occurs only once. The codes decode back to their raw forms with
`snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Selling isn't changed (it reads `ItemPrice` itself), so you can buy
  items for 0 and sell them for gil.
- Inns aren't changed.

## Spells cost no MP in battle

**Code:** `DDA7-67AE` + `DD61-64A3` (use both)
**Raw form:** `03CE3F:00` (orig `$7F`) + `038A6B:00` (orig `$7F`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 14:57:18

Tags: `demo-ready` `curated`

**Mechanism:** The MP cost is the low 7 bits of byte 5 of a spell's
`AttackProp` entry. When a character casts a spell, `battle/magic.asm`
reads it and takes it from the caster's MP:

```
        lda     $28a1
        and     #$7f        ; mp cost
        sta     $a9
        sec
        lda     $268b       ; subtract from attacker's mp
        sbc     $a9
```

The first code changes `and #$7f` to `and #$00`, so the cost is 0. At
the start of a battle, `battle/init.asm` copies each known spell's cost
to the spell list the same way (`lda $28a1`, `and #$7f`,
`sta $2c7d,x`). `battle/char.asm` grays out spells that cost more than
the caster's MP, and the battle menu shows the cost from this list. The
second code makes these costs 0 too, so every spell can be picked.

**Evidence:** `ff2_original.smc` has `AD A1 28 29 7F 85 A9 38 AD 8B 26`
at `$03:CE3B` (the mask is at `$03:CE3F`) and `AD A1 28 29 7F 9D 7D 2C`
at `$03:8A67` (the mask is at `$03:8A6B`). Each pattern occurs only
once. The codes decode back to their raw forms with
`snes_gg_encode.py`.
Tested in Mesen2 together with the 3 codes of "Skip the opening and start
outside Baron", the 2 codes of "New game: Rydia is in the party" and
"Rydia starts with Bahamut" or "Leviathan".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Cecil and Kain know no spells at the start of the game, so this shows
  only once a spell caster joins.
- Twin spells (Palom and Porom) and Recall check MP in their own code
  (`$03:E5C4`, `$03:EA94`), which this doesn't change.
- Spells cast from the field menu still cost MP.

## Spells cost no MP on the field menu

**Code:** `DD84-DFA9`
**Raw form:** `01B427:00` (orig `$7F`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 14:58:15

**Mechanism:** The Magic menu (`menu/magic.asm`) gets a spell's MP cost
from byte 5 of its `AttackProp` entry and keeps it in `$1b93`:

```
        lda     $0f97a5,x
        and     #$7f
        sta     $1b93
```

The menu shows `$1b93` as the MP needed, `_01ba05` takes it from the
caster's MP, and `MagicEffect_14` gives it back if the spell does
nothing. The code changes `and #$7f` to `and #$00`, so the cost is
always 0.

**Evidence:** The `@xxxx` labels here don't match the USA ROM.
`ff2_original.smc` has `BF A5 97 0F 29 7F 8D 93 1B` at `$01:B422` (the
mask is at `$01:B427`). The pattern occurs only once. The battle code
loads `AttackProp` from `$0F:97A0` (`A2 A0 97 86 80 A9 0F 85 82` at
`$03:8A4E`), so `$0F:97A5` is byte 5 of entry 0. The code decodes back
to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Cecil and Kain know no spells at the start of the game, so this shows
  only once a spell caster joins.
- Spells cast in battle still cost MP. Use "Spells cost no MP in battle"
  for those.

## Healing items in battle refill all HP

**Code:** `D32F-D462`
**Raw form:** `03D41A:0E` (orig `$1E`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 14:59:16

**Mechanism:** This is the FF2 version of the Metroid, ALTTP and Super
Metroid energy pickups that refill everything. Cure spells and healing
items both use `RestoreHP` (`battle/magic.asm`). It already has a full
heal path, which only Cure4 (`$11`) on one target uses:

```
_d416:  lda     $352a
        bne     @d439
        lda     $26d2
        cmp     #$11
        bne     @d439
        lda     $3906
        cmp     #$01
        bne     @d439
        longa
        sec
        lda     $2709
        sbc     $2707
        sta     $a4
        shorta0
        bra     @d43c
@d439:  jsr     CalcDmg
```

Everything else gets the normal amount from `CalcDmg`. The code changes
the first branch offset from `$1E` to `$0E`, so an item (`$352a` set)
goes to `longa` and heals max HP (`$2709`) minus current HP (`$2707`).

**Evidence:** The `@xxxx` labels here don't match the USA ROM.
`ff2_original.smc` has `AD 2A 35 D0 1E AD D2 26 C9 11 D0 17 AD 06 39 C9
01 D0 10 C2 20 38 AD 09 27 ED 07 27` at `$03:D416` (the offset is at
`$03:D41A`). The pattern occurs only once. `longa` is at `$03:D429`,
`$0E` bytes after the end of the branch. The code decodes back to its
raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Only items used in battle change. Items used from the field menu don't.
- `RestoreHP` also runs when a target absorbs an item's element, so a
  monster that absorbs an attack item is fully healed.

## Healing spells in battle refill all HP

**Codes:** `D524-DD02` + `DD24-D4D2` (use both)
**Raw form:** `03D421:07` (orig `$17`) + `03D428:00` (orig `$10`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 14:59:41

**Mechanism:** `RestoreHP` (`battle/magic.asm`) heals the normal amount
from `CalcDmg`, except for Cure4 (`$11`) on one target, which heals the
target's max HP minus its current HP. See "Healing items in battle
refill all HP" for the code. The first code changes the offset of the
`cmp #$11` branch from `$17` to `$07`, so every other spell goes to the
full heal path at `longa`. The second code changes the offset of the
one target test from `$10` to `$00`, so Cure4 on several targets also
heals fully.

**Evidence:** The `@xxxx` labels here don't match the USA ROM.
`ff2_original.smc` has `AD D2 26 C9 11 D0 17 AD 06 39 C9 01 D0 10 C2 20`
at `$03:D41B` (the offsets are at `$03:D421` and `$03:D428`). `longa` is
at `$03:D429`, `$07` bytes after the first branch and right after the
second. The codes decode back to their raw forms with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Cecil and Kain know no spells at the start of the game.
- Monster healing spells fully heal monsters too.
- `RestoreHP` also runs when a target absorbs a spell's element
  (`MagicEffect_01`, `_14`, `_17` and others), so a monster that absorbs
  an attack spell is fully healed.
- Spells cast from the field menu don't change.

## Weapon attacks never miss

**Codes:** `DDAB-0FA3` + `CBA3-0FA2` + `DDA3-04D2` (use all three)
**Raw form:** `03C997:00` (orig `$03`) + `03C5E7:A9` (orig `$AD`) +
`03C5E8:00` (orig `$FD`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 15:53:14

**Mechanism:** `CalcHits` (`battle/damage.asm`) rolls once for each
base hit in `$38fb` and counts the rolls under the rate in `$38fa`:

```
@c990:  jsr     Rand99
        cmp     $38fa       ; check vs. hit rate
        bcs     @c99b
        inc     $38fd       ; increment number of hits
@c99b:  dey
        bne     @c990
```

`DoFightCmd` (`battle/fight.asm`) calls it for the attacker's hits and
again for the hits the target defends, and subtracts the second from the
first. Zero or fewer net hits is a miss:

```
@c5e4:  jsr     CalcHits
        lda     $38fd
@c5ea:  sta     $38f9       ; set number of hits defended
        sec
        lda     $38f8
        sbc     $38f9       ; subtract from number of hits
```

The first code changes the `bcs` offset from 3 to 0, so every roll
counts and `CalcHits` returns all of `$38fb`. The attacker gets every
base hit (the attack multiplier). The other two codes change
`lda $38fd` (`AD FD 38`) to `lda #$00` and `sec` (`A9 00 38`), so the
target defends no hits. `sec` comes next anyway, so the extra one does
nothing.

**Evidence:** The `fight.asm` and `damage.asm` labels match the USA ROM.
`ff2_original.smc` has `B0 03 EE FD 38 88 D0 F2` at `$03:C996` (the
offset is at `$03:C997`) and `20 87 C9 AD FD 38 8D F9 38 38` at
`$03:C5E4`. Each pattern occurs only once. The codes decode back to
their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Monsters' weapon attacks never miss the party either. Both sides use
  `DoFightCmd`.
- Every attack does its full number of hits, so weapon damage goes up
  for both sides. Blind and the back row no longer lower the hit count.
- `CalcHits` is shared, so every roll that uses it succeeds:
  - Weapon status effects always land (unless the target is immune, a
    boss, or the hit rate is not above its defense rate).
  - Spells get all their base hits, and monsters block all of their
    magic defense hits. A spell can then miss a monster with high
    magic defense more often than before.
- Barrier and Image still block the attack.

## Damage never takes HP below 1

**Codes:** `DFA8-6DD3` + `DDA8-6D03` + `6DAA-6463` + `37AA-64A3` (use all
four)
**Raw form:** `03CAB0:01` (orig `$FF`) + `03CAB1:00` (orig `$FF`) +
`03CACA:80` (orig `$E2`) + `03CACB:E3` (orig `$20`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 15:53:32

**Mechanism:** This is the FF2 version of the Metroid "Samus can't die"
code. `ApplyDmg` (`battle/damage.asm`) subtracts each target's damage
from its HP. If the result is zero or less, it counts a death and sets
the dead status:

```
@ca9e:  lda     $34d4,y
        bpl     @cab4       ; branch if attack did damage
; restore hp
        clc
        lda     $34d4,y
        and     #$3fff
        adc     $2007,x     ; add to target hp
        bcc     @cae6
        lda     #$ffff      ; max $ffff
        bra     @cae6
; damage hp
@cab4:  lda     $34d4,y
        and     #$bfff      ; clear miss flag
        sta     $2898
        sec
        lda     $2007,x
        beq     @cad1
        sbc     $2898       ; subtract from target hp
        beq     @caca
        bcs     @cae6
; hp reached zero
@caca:  shorta
        inc     $3907       ; increment number of targets that died
```

The first two codes change `lda #$ffff` to `lda #$0001`. The other two
change `shorta` at `@caca` to `bra` back to that `lda`. When HP would
reach zero or less, it is set to 1 and stored at `@cae6`, the same as
any other new HP value.

**Evidence:** The `damage.asm` labels match the USA ROM.
`ff2_original.smc` has `90 37 A9 FF FF 80 32` at `$03:CAAD` (the `lda`
is at `$03:CAAF`) and `F0 02 B0 1C E2 20 EE 07 39` at `$03:CAC6`. Each
pattern occurs only once. The new `bra` at `$03:CACA` goes to
`$03:CACC` - `$1D` = `$03:CAAF`. The codes decode back to their raw
forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Monsters can't die from damage either. Battles can then only end by
  running away, or by a death or stone effect. The party can't win
  normal battles with this code on.
- It covers every damage that goes through `ApplyDmg`: weapons, spells,
  items and poison. Death, stone and other status effects still work.
- The `lda #$ffff` was the cap for healing past 65,535 HP. A heal that
  would go past that now sets HP to 1. Only monsters have enough HP for
  that.
- A target already at 0 HP still dies.

## Start a new game with 99 Elixirs

**Codes:** `276E-6DDD` + `176E-646D` (use both)
**Raw form:** `0082F0:D3` (orig `$FE`) + `0082FA:63` (orig `$01`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:02:29

Tags: `demo-ready`

**Mechanism:** The inventory is 48 slots of item ID and quantity at
`$1440`. A new game starts with it empty except for the last two slots.
`InitNewGame` (`field/field.asm`) puts the Sort entry (item `$FE`) and
the TrashCan (item `$FF`) there:

```
        lda     #$fe
        sta     $149c
        lda     #$ff
        sta     $149e
        lda     #1
        sta     $149d
        sta     $149f
        rts
```

The first code changes `lda #$fe` to `lda #$d3`, Elixir. The second
changes `lda #1` to `lda #99`, the most one slot can hold. The
second-to-last slot then holds 99 Elixirs from the start of the game.

**Evidence:** The `@xxxx` labels here match the USA ROM.
`ff2_original.smc` has `A9 FE 8D 9C 14 A9 FF 8D 9E 14 A9 01 8D 9D 14 8D
9F 14 60` at `$00:82EF`. `InitNewGame` is called from `$00:8059` only
when `$17FB` is 0 (a new game). The item name table at `$0F:8000` has
`Elixir` for item `$D3`, `Sort` for `$FE` and `TrashCan` for `$FF`. The
codes decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The Sort entry is gone, so the item menu can't sort.
- The TrashCan's quantity becomes 99 too. What that does was not
  checked.
- Only a new game changes.

## Rydia starts with every white and black spell

**Code:** `2CA0-6920`
**Raw form:** `15C247:DA` (orig `$C0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:03:35

Tags: `demo-ready`

**Mechanism:** A new game fills the 13 spell lists at `$1560` (24 bytes
each) from `SpellListInit` (`$0F:C8C0`). `InitSpellLists`
(`field/field.asm`) reads it as one stream: it copies spell IDs until a
`$FF`, then pads the rest of that list with zeros. A list that gets 24
spells ends without a `$FF`.

```
@c246:  lda     f:SpellListInit,x
        cmp     #$ff
        beq     @c25f
        sta     $1560,y
```

The code moves the start of the stream from `$0F:C8C0` to `$0F:C8DA`,
26 bytes later, so every list reads the data of a later list. FuSoYa's
white and black lists then land on Rydia. The lists become (names from
`notes/ff4j-sfc-ram-map.txt`):

- List 0, Cecil (white): Cure1 becomes Fire1, Ice1, Lit1, Sleep and
  Venom (Palom's list).
- List 1 (unknown): empty becomes Cure1, Hold, Peep, Slow and Sight
  (Porom's list).
- List 2, Rydia (white): empty becomes FuSoYa's white list: Hold, Mute,
  Charm, Blink, Slow, Fast, Bersk, Wall, White, Peep, Cure1 to
  Cure4, Heal, Life1, Life2, Size, Exit, Sight and Float.
- List 3, Rydia (black): empty becomes FuSoYa's black list, all 24 from
  Toad to Nuke, with Meteo.
- List 4, Rydia (call): Chocb becomes Flame (Edge's list).
- Lists 5-12 (Tellah, Rosa, Palom, Porom, FuSoYa, Edge): all empty.

**Evidence:** The `@xxxx` labels here don't match the USA ROM.
`InitSpellLists` is at `$15:C23E`, and `ff2_original.smc` has
`BF C0 C8 0F C9 FF F0 11 99 60 15` at `$15:C246` (the low byte of the
address is at `$15:C247`). It is called from `$00:805F`, right after
`InitNewGame`, only for a new game. The pattern `C0 C8 0F` occurs only
once. The stream at `$0F:C8C0` is `0E FF FF FF FF 35 FF 0F 03 04 12 13
16 FF 1D 20 23 2C 2E FF 0E 01 0D 07 17 FF 1D 20 23 29 1C FF 0E 01 0D 07
17 FF`, then FuSoYa's white list (`01`-`04`, `07`-`18`, `FF`) at
`$0F:C8E6`, his black list (`19`-`30`) and Edge's (`42 FF`) at
`$0F:C8FC`, then only `$FF` to `$0F:C9FF`. The spell names come from
the table at `$0F:8900` (6 bytes each). The code decodes back to its
raw form with `snes_gg_encode.py`.
Tested in Mesen2 together with the 3 codes of "Skip the opening and start
outside Baron" and the 2 codes of "New game: Rydia is in the party": the
White and Black menus showed every spell.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Rydia joins after the village of Mist, so it shows from then on. Cecil and
  Kain have no Magic command.
- Child Rydia has little MP, so most of these spells cost more than she
  has. "Spells cost no MP in battle" fixes that.
- Tellah, Rosa, Palom, Porom, FuSoYa and Edge start with no spells.
  They still learn spells by level and by story events.
- Rydia loses Chocobo. Her Call list has Flame instead.
- Only a new game changes.

## Rydia starts with Bahamut

**Code:** `7EAA-DF53`
**Raw form:** `0FC8C5:3F` (orig `$35`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:05:09

Tags: `demo-ready` `curated`

**Mechanism:** A new game fills the spell lists at `$1560` from
`SpellListInit` (`$0F:C8C0`); see "Rydia starts with every white and
black spell" for how `InitSpellLists` reads it. The first lists are
`0E FF FF FF FF 35 FF`: Cure1 for Cecil, three empty lists, and
Chocobo (`$35`, `Chocb`) for Rydia's Call list (list 4). The code
changes Chocobo to `$3F`, Bahamut. It is a summon (IDs `$31`-`$3F`), so it
belongs in the Call list anyway. `DoMagicCmd` (`battle/magic.asm`) decides
white, black or summon from the spell ID, not from the menu.

**Evidence:** `ff2_original.smc` has `0E FF FF FF FF 35 FF` at
`$0F:C8C0` (the `35` is at `$0F:C8C5`). The spell name table at
`$0F:8900` (6 bytes each) has `Baham` for spell `$3F`. The code
decodes back to its raw form with `snes_gg_encode.py`.
Tested in Mesen2 together with the 3 codes of "Skip the opening and start
outside Baron", the 2 codes of "New game: Rydia is in the party" and
"Spells cost no MP in battle": Bahamut was in her Call list and cast.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Rydia joins after the village of Mist, so it shows from then on.
- She may not have enough MP to cast it. "Spells cost no MP in battle"
  removes the cost.
- Rydia loses Chocobo.
- Only a new game changes.

## Rydia starts with Leviathan

**Code:** `72AA-DF53`
**Raw form:** `0FC8C5:3D` (orig `$35`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:05:13

Tags: `demo-ready`

**Mechanism:** This works like "Rydia starts with Bahamut". The code
changes Chocobo (`$35`) in Rydia's Call list in `SpellListInit` to
`$3D`, Leviathan. It is a summon (IDs `$31`-`$3F`), so it
belongs in the Call list anyway.

**Evidence:** `ff2_original.smc` has `0E FF FF FF FF 35 FF` at
`$0F:C8C0` (the `35` is at `$0F:C8C5`). The spell name table at
`$0F:8900` (6 bytes each) has `Levia` for spell `$3D`. The code
decodes back to its raw form with `snes_gg_encode.py`.
Tested in Mesen2 together with the 3 codes of "Skip the opening and start
outside Baron", the 2 codes of "New game: Rydia is in the party" and
"Spells cost no MP in battle": Leviathan was in her Call list and cast.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Rydia joins after the village of Mist, so it shows from then on.
- She may not have enough MP to cast it. "Spells cost no MP in battle"
  removes the cost.
- Rydia loses Chocobo.
- Only a new game changes.

## Every battle has only 1 enemy

**Codes:** `CB67-04DA` + `0D67-040A` + `3C67-046A` (use all three)
**Raw form:** `038138:A9` (orig `$AD`) + `038139:40` (orig `$A0`) +
`03813A:EA` (orig `$29`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:14:14

Tags: `demo-ready`

**Mechanism:** The battle setup in `battle/battle.asm` reads how many
of each of the battle's 3 monster types there are from `$29a0`, 2 bits
per type, first type in the top 2 bits:

```
        lda     $29a0                   ; monster type counts (2 bits each)
        sta     $ab
@813d:  stz     $29ca,x
        asl     $ab
        rol     $29ca,x
        asl     $ab
        rol     $29ca,x
```

The codes change `lda $29a0` to `lda #$40 / nop`. `$40` is binary
`01 00 00 00`: 1 of the first type and none of the other two. The slot
fill then puts that 1 monster in slot 0, and the total count is 1.

**Evidence:** `ff2_original.smc` has `AD A0 29 85 AB 9E CA 29` at
`$03:8138`, and the pattern occurs only once. The code runs with an
8-bit A. The low 2 bits of `$29a0` are read again in
`battle/init.asm` from `$29a0` itself, so they don't change. Each
code decodes back to its raw form with `snes_gg_encode.py`, and the new
bytes were checked with a disassembler. Tested in Mesen2 together with the 4
codes of "Skip the opening and start outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The 1 monster is always the battle's first type. In a battle where a
  boss is the second or third type, the boss is gone and only the
  first type is left.
- Some story battles use a monster that is hidden at first, or that has
  to be there for a script. If that monster is the one that is left,
  or the one that is removed, the battle may not end the normal way.
- Fewer monsters give less experience and gil.
- Don't use it with "Every battle has 8 enemies"; that cheat then fills
  the other 7 slots with the same monster.

## Every battle is the one 32 places later in the battle list

**Code:** `6F6D-0F0A`
**Raw form:** `038105:81` (orig `$80`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, instead, 64 places later:** `646D-0F0A` (`038105:82`)
**Optional, instead, 128 places later:** `606D-0F0A` (`038105:84`)
**Recorded:** 2026-10-24 16:15:55

Tags: `demo-ready` `curated` `challenge`

**Mechanism:** The battle setup in `battle/battle.asm` multiplies the
battle number (`$1800`) by 8 and copies that battle's 8 bytes from
`BattleProp` (`$0E:8000`, 512 battles) to `$299c`:

```
@80ec:  lda     $1800
        sta     $393d                   ; get pointer to battle properties
        lda     #8
        sta     $393f
        jsr     Mult16
        shorta0
        ldx     $3941
        clr_ay
@8103:  lda     f:BattleProp,x
        sta     $299c,y                 ; load battle properties
```

The code changes the middle byte of the `BattleProp` address from
`$80` to `$81`, so the table starts `$100` bytes (32 battles) later.
Every battle loads the whole record of battle number + 32: its monster
types and counts, positions, cursor data, music and flags. The
optional codes use `$82` (+64) or `$84` (+128).

In the first area, battle 0 (3 Imps) becomes battle 32 (3 Imps and an
Imp Captain), battle 1 (2 FloatEyes) becomes battle 33 (a Turtle and 2
Imps), and battle 3 (3 Eagles) becomes battle 35 (a Basilisk and 3
Imps).

**Evidence:** `ff2_original.smc` has `BF 00 80 0E 99 9C 29` at
`$03:8103`, and `BF 00 80 0E` occurs only once. The battle list was
read from `$0E:8000` (byte 4 holds the 2-bit counts, bytes 1-3 the
types) with the monster names at `$0E:9800`. The `ff4j` ROM map lists
512 battles of 8 bytes at `$0E:8000`-`$0E:8FFF`. Each code decodes
back to its raw form with `snes_gg_encode.py`. Tested
in Mesen2 (the base code and the 64-place and 128-place options) together with the
4 codes of "Skip the opening and start outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Boss and story battles change too. The code that checks the battle
  number (`$1800`) still sees the old number, so scripted events that
  expect a certain monster may not end the normal way.
- The last 32 battles (64 or 128 with the optional codes) read past the
  end of the table, into the monster position data at `$0E:9000`.
  These battles are garbage and may freeze.
- The new battles can be much harder than the area's normal ones.

## The whole party fights from the back row

**Codes:** `6DB2-0D02` + `DBBF-6FA2` + `6DBF-64D2` + `DBBF-67A2` +
`6DB4-6DD2` (use all five)
**Raw form:** `0395D1:80` (orig `$D0`) + `039617:09` (orig `$29`) +
`039618:80` (orig `$7F`) + `03961F:09` (orig `$29`) +
`039620:80` (orig `$7F`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:17:23

**Mechanism:** At the start of each battle, `InitCharRows`
(`battle/init.asm`) sets bit 7 of each party slot's `$2001` (1 = back
row) from the row setting `$16a8`. With `$16a8` = 0, slots 1-3 are
in front and slots 4-5 in back; otherwise slots 1-3 are in back and
slots 4-5 in front:

```
@95ce:  lda     $16a8       ; row setting
        bne     @95fc
        ...
@95fc:  lda     $2001
        ora     #$80
        sta     $2001
        ...                 ; same for $2081 and $2101
        lda     $2181
        and     #$7f
        sta     $2181
        lda     $2201
        and     #$7f
        sta     $2201
        rts
```

The first code changes `bne` to `bra`, so the second half always
runs. The other four change its two `and #$7f` to `ora #$80`, so all
five slots get the back row bit.

In the back row, `DoFightCmd` (`battle/fight.asm`) halves the
attacker's hit rate (not for bows, or when jumping), and a back row
target's defense is doubled. So the party does about half the damage
with most weapons, and takes about half the damage from weapon attacks.

**Evidence:** `ff2_original.smc` has `AD A8 16 D0` at `$03:95CE` and
`AD 81 21 29 7F 8D 81 21 AD 01 22 29 7F 8D 01 22 60` at `$03:9614`.
Each pattern occurs only once. `InitCharRows` is called from
`InitObjects` (`battle/init.asm`) and from `battle/win.asm`. Each code
decodes back to its raw form with `snes_gg_encode.py`, and the new
bytes were checked with a disassembler.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The character sprites don't move. `InitCharSprites`
  (`btlgfx/sprite.asm`) places them from `$16a8`, not from `$2001`.
  The change shows only in the damage numbers.
- The Change command (`Cmd_1a`, `battle/cmd.asm`) flips every slot's
  row bit, so the whole party then moves to the front row.
- Monster attacks that target "all front row characters" find no one.

## Battles give twice the experience

**Code:** `4FE5-A1AE`
**Raw form:** `13FF7B:21` (orig `$20`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, instead, 4 times:** `44E5-A1AE` (`13FF7B:22`)
**Optional, instead, 16 times:** `40E5-A1AE` (`13FF7B:24`)
**Recorded:** 2026-10-24 16:19:36

**Mechanism:** After a won battle, `WinUpdate` (`battle/win.asm`) adds
up the defeated monsters' experience at `$3591` and counts the living
party members in `$ab`. `DivXP` (`battle/battle.asm`) then divides
the total by the count, one quotient bit per loop:

```
        longa
        clc
        ldx     #$0020
@ff7d:  rol     $a9         ; shift the next total bit into $b5
        rol     $ab
        rol     $b5
        rol     $b7
        sec
        lda     $b5
        sbc     $ad         ; subtract the member count
        ...
@ffa1:  rol     $b1         ; next quotient bit
        rol     $b3
        dex
        bne     @ff7d
```

The 32 loops use up the 32-bit total. The code makes it 33: the extra
loop shifts in a 0 bit, so the result is `2 * total / count`, twice
each member's share. Each extra loop doubles it again: `$22` (34
loops) is 4 times and `$24` (36 loops) is 16 times.

**Evidence:** `ff2_original.smc` has `A2 20 00 26 A9 26 AB` at
`$13:FF7A`, and the pattern occurs only once. `DivXP` is at
`$13:FF5F` and its one caller is the `jsl $13ff5f` at `$03:EE0E`
(`battle/win.asm`). A model of the loop in Python gives 3, 7, 14 and
56 for 7 experience and 2 members with 32, 33, 34 and 36 loops. Each
code decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Gil doesn't change.
- The battle's "experience points" message shows the new amount.
- Each character still stops at 9,999,999 experience.

## Every party member gets the battle's full experience

**Code:** `6D3E-0DAE`
**Raw form:** `03EDF3:80` (orig `$F0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:20:06

**Mechanism:** After a won battle, `WinUpdate` (`battle/win.asm`) adds
up the defeated monsters' experience at `$3591` and counts the living
party members in `$ab`. Then it divides the total by the count, unless
the total is 0:

```
        lda     $3591
        ora     $3592
        ora     $3593
        beq     @ee40
        ...
@ee28:  jsl     DivXP
        ...
@ee36:  ldx     $ad
        stx     $3591
        lda     $af
        sta     $3593
@ee40:  jsr     InitGfxScript
```

The code changes `beq` to `bra`, so the division is always skipped.
`$3591` keeps the whole total, and each living member gets all of it,
as if they were the only one in the party.

**Evidence:** `ff2_original.smc` has `AD 91 35 0D 92 35 0D 93 35 F0 31`
at `$03:EDEA` (the `beq` is at `$03:EDF3`), and `0D 93 35 F0 31`
occurs only once. The branch goes to `jsr $b2fd` at `$03:EE26`
(`InitGfxScript`), past the store of the result. The code decodes back
to its raw form with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- With Cecil and Kain, each gets twice the normal amount.
- Gil doesn't change.
- "Battles give twice the experience" changes `DivXP`, which this code
  skips, so the two don't add up.

## Attacks that Cecil covers always miss

**Codes:** `40A4-6F02` + `A1A4-64D2` + `70A4-6402` (use all three)
**Raw form:** `03C625:24` (orig `$10`) + `03C628:C6` (orig `$81`) +
`03C629:34` (orig `$38`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:21:54

**Mechanism:** When a Fight attack targets an ally who is low on HP or
is being covered, the attack code in `battle/attack.asm` moves the
attack to Paladin Cecil (`$355e`) and saves the old target mask:

```
        lda     $34c5
        sta     $34c6       ; targets being covered
        ldx     $355e
        txa
        sta     $ce         ; change target to paladin cecil
```

`InitGfxScript` clears `$34c2`-`$34d3` at the start of each action, so
`$34c6` is nonzero only when Cecil covers. `DoFightCmd`
(`battle/fight.asm`) checks the target after it works out the net hits:

```
; nonzero net hits
@c623:  lda     $ce
        bpl     @c62c       ; branch if target is a character
        lda     $3881
        bne     @c5fe       ; branch if monsters are invincible ???
```

The codes change `bpl @c62c` to `bit $05` (which does nothing here) and
`lda $3881` to `lda $34c6`. Every target now goes through the check,
and when Cecil covers, the attack goes to `@c5fe`, which sets zero hits
and the miss flag.

**Evidence:** `ff2_original.smc` has `A5 CE 10 05 AD 81 38 D0 D2` at
`$03:C623`, and the pattern occurs only once. The cover code is at
`@af6f` in `battle/attack.asm`, and it is the only code that stores
`$34c6`. `InitGfxScript` clears `$34c2`-`$34d3` (`ldx #$0011 / stz
$34c2,x`). Each code decodes back to its raw form with
`snes_gg_encode.py`, and the new bytes were checked with a
disassembler.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Cecil becomes a Paladin and gets cover later in the game (Mount
  Ordeals).
- Only Fight attacks can be covered, so only they miss.
- The `$3881` check is gone. `InitObjects` (`battle/init.asm`) sets it
  for battles `$1B6` and `$1B7` at the end of the game, so Fight attacks
  there no longer miss before the story says they should.
- It uses the byte at `$03:C626`, like "Enemy weapon attacks always miss
  the party". With both on, that code's effect is lost.

## Paladin Cecil has the Dark Wave command

**Code:** `D9E6-01AE`
**Raw form:** `13FD8B:05` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:24:28

**Mechanism:** At the start of each battle, the monster and character
init in `battle/init.asm` copies 5 command ids per character from
`ClassCmd` (`$13:FD55`, 5 bytes per class) to the battle command list
at `$3303`:

```
@8b99:  lda     f:ClassCmd,x            ; battle commands for each class
        sta     $3303,y
```

Class `$0B`, Paladin Cecil, has `00 02 13 01 FF` (Fight, White,
Cover, Item, none) at `$13:FD87`. The code changes the last `$FF` to
`$05`, Dark Wave (`Cmd_05` in `battle/cmd.asm`). The USA release
took Dark Wave away from Dark Knight Cecil, but its command code is
still in the ROM. For Dark Knight Cecil, see "Dark Knight Cecil has the
Dark Wave command".

Dark Wave takes 1/8 of the user's max HP, then hits every enemy through
`DoMultiAttack` ("do multi-attack (kick/dark wave)"):

```
Cmd_05:
@e9ec:  lda     $cd
        bmi     @ea16
        longa
        ldx     $a6
        lda     $2009,x
        jsr     Lsr_3
        sta     $a9
        sec
        lda     $2007,x
        sbc     $a9
        sta     $2007,x
```

**Evidence:** `ff2_original.smc` has `BF 55 FD 13 99 03 33` (`lda
$13fd55,x / sta $3303,y`) at `$03:8B99`. The 22 class lists at
`$13:FD55` match the classes' commands (Rydia's is `00 02 03 04 01`,
Edge's `00 16 17 18 01`). `00 02 13 01 FF` occurs only once. Entry
`$05` of the command table at `$03:B36C` is `$E9D4`, and the bytes there
(`A5 CD 30 26 C2 20 A6 A6 BD 09 20 20 85 84`) match `Cmd_05`. The code
decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The USA command name for `$05` is "Dummy", so the menu shows Dummy.
- Paladin Cecil comes later in the game (Mount Ordeals).
- Dark Wave can kill Cecil: if his HP is less than 1/8 of his max HP, it
  drops to 0 and he dies.
- The USA ROM may lack the Dark Wave attack data or animation, so the
  attack may look or work wrong.

## Sneak always takes the monster's rarest item

**Code:** `D730-6D6A`
**Raw form:** `03E242:03` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, also steal from monsters that drop nothing:** `6D37-6D0A`
(`03E231:80`, orig `$D0`)
**Recorded:** 2026-10-24 16:39:06

**Mechanism:** When Sneak (`Cmd_17` in `battle/cmd.asm`; Steal in other
versions) works, it takes the monster's item set from `$2773` and reads
item 0 of the set's 4 items in `MonsterItems` (`$0E:9F00`):

```
        lda     $2773       ; monster item set
        and     #$3f
        jsr     Asl_2
        tax
        lda     f:MonsterItems,x        ; monster items (always choose 1st item)
```

Item 0 is the common drop. `WinUpdate` in `battle/win.asm` picks a
battle drop from the same 4 items, and picks item 3 only on a random
byte of `$FC` or more (1 in 64). The code changes the operand from
`$0E:9F00` to `$0E:9F03`, so Sneak takes item 3, the rare drop.

The optional code changes the `bne` after the drop-rate check into
`bra`:

```
@e242:  lda     $2773       ; item drop rate
        and     #$c0
        cmp     #$c0
        bne     @e24c       ; branch if nonzero
        rts
```

So Sneak also works on monsters whose drop rate bits are `$C0` (no
drop).

**Evidence:** `ff2_original.smc` has `AD 73 27 29 C0 C9 C0 D0 01 60` at
`$03:E22A` and `BF 00 9F 0E F0 DD` at `$03:E241`. The set index is
at most `$3F`, so the last byte read is `$0E:9FFF`, still in the
table. Set 1 is `Cure1 Cure1 Cure1 Cure2` and set 2 is `Heal Heal Heal
Tent`. Each code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Only Edge has Sneak. Use "Every character has the Sneak command" in
  `ff2_long_game_genie.md` to give it to Cecil and Kain.
- Battle drops don't change.
- With the optional code, the item from a no-drop monster is whatever
  its set index points to.

## Edward's Sing always plays a song

**Code:** `6D3D-0DD3`
**Raw form:** `03E900:80` (orig `$90`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:40:17

**Mechanism:** `Cmd_08` in `battle/cmd.asm` (Sing) rolls a random
byte and picks a spell to cast by its value:

```
@e913:  jsr     Rand
        cmp     #$c0
        bcc     @e923
        lda     #$00
        sta     $34ca
        lda     #$00
        bra     @e944
@e923:  cmp     #$80
```

A roll of `$C0` or more (1 in 4) casts spell `$00` with message `$00`
("nothing happened"). `$80`-`$BF` casts Mute (`$02`), `$40`-`$7F` Charm
(`$03`) and below `$40` Sleep (`$29`). The code changes the `bcc`
into `bra`, so every roll goes on to the song checks. A roll of `$C0`
or more is then also Mute: Mute 1 in 2, Charm 1 in 4, Sleep 1 in 4.

**Evidence:** `ff2_original.smc` has `20 93 85 C9 C0 90 09` (`jsr
Rand / cmp #$c0 / bcc`) at `$03:E8FB`, and the pattern occurs only
once. The spell ids are `A9 02`, `A9 03` and `A9 29` at `$03:E914`,
`$03:E921` and `$03:E92A`; their names in the spell name table are
Mute, Charm and Sleep. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The code only removes the roll that plays nothing. Whether each song
  can still miss was not checked.
- If bit `$20` of the singer's status byte `$2683` is set, Sing
  casts Toad (`$19`) instead, before the roll, as before.
- Sing needs a harp equipped (`battle/char.asm`).
- Edward joins after the first battles.

## New-game spell lists come from the level-up spell table

**Codes:** `DDA0-6920` + `A5A0-61F0`
**Raw form:** `15C247:00` (orig `$C0`) + `15C248:C7` (orig `$C8`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:50:25

**Mechanism:** A new game fills the 13 spell lists at `$1560` from
`SpellListInit` (`$0F:C8C0`); see "Rydia starts with every white and
black spell" for how `InitSpellLists` reads it. It copies spell ids
until a `$FF`, then pads the rest of that 24-byte list with zeros:

```
@c246:  lda     f:SpellListInit,x
        cmp     #$ff
        beq     @c25f
        sta     $1560,y
```

The codes change the address from `$0F:C8C0` to `$0F:C700`, the start
of `SpellListLearned` (`battle/battle_data.asm`), which `LevelUpMagic`
(`battle/win.asm`) reads as pairs of a level and a spell id, with
`$FF` at the end of each list. Read as spell lists, the levels become
spells too. The lists become (spell names from the table at
`$0F:8900`):

- List 0, Cecil (white): Charm, Sight, Fast, Peep, Cure2, Cure2, Life1,
  Exit, Float, Heal.
- List 2, Rydia (white): Charm, Cure1, Blink, Sight, Slow, Hold.
- List 3, Rydia (black): Mute, Ice1, Armor, Lit1, Fast, Sleep, Wall,
  Venom, Dummy, Warp, Peep, Toad, Cure2, Stop, Life2, Piggy, Piggy,
  Virus, Fire3, Psych, Lit1, Drain, Virus, Ice3.
- List 4, Rydia (call): Quake, Fire3, Stone, Lit3, Stop, Quake, Psych,
  Stone, Nuke, Weak, Imp, Fatal, Bomb, Nuke, Odin, Meteo.
- List 8, Palom (black): 24 spells from White to Wall.
- List 9, Porom (white): Virus, Cure4, Stone, Life2, Nuke, White.
- Lists 10 and 11, FuSoYa: 24 and 14 spells, with Meteo.
- List 12, Edge: 24 spells from White to Float.
- Lists 1 and 5-7 (unknown, Tellah, Rosa): empty.

`DoMagicCmd` (`battle/magic.asm`) picks white, black or summon from
the spell id, not from the menu, so a white spell in a black list still
works.

**Evidence:** `ff2_original.smc` has `BF C0 C8 0F C9 FF` at
`$15:C246`. The data at `$0F:C700` starts `03 17 08 0D 0F 0F 13 16
18 12 FF FF 03 0E 04 17 07 01 FF`. The lists above come from a Python
model of the `InitSpellLists` loop run on the ROM bytes; the 13 lists
end at `$0F:C79C`, inside `SpellListLearned`. Each code decodes back
to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Only a new game changes. Rydia joins after the village of Mist, so it
  shows from then on. Dark Knight Cecil has no Magic command.
- Child Rydia has little MP, so most of these spells cost more than she
  has. "Spells cost no MP in battle" fixes that.
- Characters still learn their normal spells by level.
- Uses the same byte as "Rydia starts with every white and black
  spell"; use only one of them.
- `Dummy` (`$0C`) is the Japanese Dispel. See "Rydia can cast Dispel
  (the unused "Dummy" spell)".

## Dark Wave costs no HP

**Code:** `6D32-0F63`
**Raw form:** `03E9D6:80` (orig `$30`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:51:11

**Mechanism:** Dark Wave (`Cmd_05` in `battle/cmd.asm`) is the only
command that costs the user HP. When a party member uses it, it takes
1/8 of the max HP (`$2009`) from the current HP (`$2007`), and
sets the dead status if HP would go below 0. A monster attacker skips
that:

```
Cmd_05:
@e9ec:  lda     $cd
        bmi     @ea16
        longa
        ldx     $a6
        lda     $2009,x
        jsr     Lsr_3
```

The code changes `bmi` into `bra`, so the HP cost is always skipped
and the attack goes on to `DoMultiAttack` as before.

**Evidence:** `ff2_original.smc` has `A5 CD 30 26 C2 20 A6 A6 BD 09 20`
at `$03:E9D4`, and the pattern occurs only once. Entry `$05` of the
command table at `$03:B36C` is `$E9D4`. The code decodes back to its
raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- In the USA version no character has Dark Wave. Use it with "Paladin
  Cecil has the Dark Wave command" or "Dark Knight Cecil has the Dark
  Wave command".

## Monsters attack other monsters

**Code:** `D680-AF02`
**Raw form:** `03B745:08` (orig `$05`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:55:15

**Mechanism:** When a monster's script does not name a target,
`SetMonsterTarget` (`battle/ai.asm`) picks one with
`GetDefaultAITarget`. `$AB` says which side: 5 picks a random
character and 8 picks a random monster:

```
@b744:  lda     #5                      ; random character target
@b746:  sta     $ab
        jsr     GetDefaultAITarget
```

The code changes the 5 to 8. Monster attacks with no set target, and
black magic, then go to a random monster that is still in the battle.
White magic already used 8.

**Evidence:** `ff2_original.smc` has `A9 05 85 AB 20 31 BC 60` at
`$03:B744` (`jsr $BC31` is `GetDefaultAITarget`), and the pattern
occurs only once. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- A monster can pick itself.
- Script commands that name a target (`GetAITarget`), Target/Search
  (`$3883`) and berserk monsters still pick the party.
- A monster alone in a battle can only attack itself.

## Monsters take poison damage all the time

**Code:** `9D62-AF0E`
**Raw form:** `038FD5:50` (orig `$40`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:58:06

**Mechanism:** `InitMonster` (`battle/init.asm`) turns on each
monster's action timer by writing `$40` to its timer flags at
`$29EB`:

```
InitMonster:
@8fca:  lda     $b9
        sta     $35a0
        asl
        clc
        adc     #$0a
        tax
        lda     #$40                    ; enable action timer
        sta     $29eb,x
```

Bit `$10` of the same byte is the poison timer (`ora #$10` in
`InitPoison` and `MagicEffect_17`). The code writes `$50`, so each
monster also starts with its poison timer on. When that timer runs out,
`TimerEffect_03` takes 1/8 of the monster's max HP and shows the
damage. Battle start clears `$2000-$397D`, so the timer's duration
and its reload value (`$2B2A`) are 0. The timer loop in
`battle/battle.asm` marks a timer with duration 0 as run out, so the
poison comes back at once each time.

**Evidence:** `ff2_original.smc` has
`A5 B9 8D A0 35 0A 18 69 0A AA A9 40 9D EB 29` at `$03:8FCA`, and the
pattern occurs only once. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- How often the damage comes was not checked; it may be as often as
  the timers are updated, so monsters may die in seconds. Bosses are
  hit too.
- The monster's poison status bit is not set, so there is no poison
  status on the monster.
- Monsters that join a battle later may not use `InitMonster`; not
  checked.

## Start a new game with the Tower key

**Code:** `EC6E-6DDD`
**Raw form:** `0082F0:FA` (orig `$FE`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:58:46

**Mechanism:** The source has a debug new game, `DebugInit` in
`field/field.asm`. It is only assembled with `SKIP_INTRO=1`, so it
is not in the ROM. Its first step puts the key to the Tower of Babil
(item `$FA`) in the first inventory slot:

```
DebugInit:
        lda     #$fa            ; give key to tower of babil
        sta     $1440
        lda     #1
        sta     $1441
```

A normal new game runs `InitNewGame`, which puts the Sort entry (item
`$FE`, quantity 1) in slot `$149C`:

```
        lda     #$fe
        sta     $149c
        lda     #$ff
        sta     $149e
        lda     #1
        sta     $149d
        sta     $149f
        rts
```

The code changes `lda #$fe` to `lda #$fa`, so that slot holds one
key instead.

**Evidence:** `ff2_original.smc` has `A9 FE 8D 9C 14 A9 FF 8D 9E 14
A9 01 8D 9D 14 8D 9F 14 60` at `$00:82EF`. The item name table at
`$0F:8000` has `Tower` for item `$FA`. The code decodes back to its
raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Only the key. `DebugInit` also gives 50,000 gil, the four vehicles,
  the hook and the tunnel to the underground; this code does not.
- The Sort entry is gone, so the item menu can't sort.
- Uses the same byte as "Start a new game with 99 Elixirs"; use only
  one of them.

## Bow attacks don't use up arrows

**Code:** `DDA1-0FA3`
**Raw form:** `03C967:00` (orig `$01`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:59:32

**Mechanism:** After a Fight attack, `DecArrows` (`battle/fight.asm`)
finds the hand that holds arrows and takes 1 from their quantity. If the
quantity reaches 0, it sets a flag so `ValidateArrows` removes them:

```
@c962:  sec
        lda     $32dc,x     ; decrement item quantity
        sbc     #$01
        bne     @c978
```

The code changes `sbc #$01` to `sbc #$00`. The quantity is stored
back unchanged, and it never reaches 0.

**Evidence:** `ff2_original.smc` has `38 BD DC 32 E9 01 D0 0E A5 CD AA
BD DC 38` at `$03:C962`, and the pattern occurs only once.
`DecArrows` is called only from `battle/fight.asm`. The code decodes
back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- No character has a bow in the first battle.

## Meteo is cast with no wait

**Code:** `ADB8-D483`
**Raw form:** `0F98BA:C0` (orig `$CA`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 17:00:26

**Mechanism:** `TimerDur_03` (`battle/timer_dur.asm`) sets the wait
before a spell goes off. It takes the low 5 bits of byte 0 of the
spell's 6-byte `AttackProp` entry and doubles it:

```
        lda     f:AttackProp,x   ; magic action delay
_9eab:  and     #$1f
        tax
        stx     $a9
        asl     $a9         ; multiply by 2
```

In the USA ROM this `lda` reads `$0F:97A0`. Meteo is spell `$2F`, so
its entry is at `$0F:98BA`, and byte 0 is `$CA` (wait 10). The code
writes `$C0`: the same upper 3 bits, wait 0.

**Evidence:** `ff2_original.smc` has `BF A0 97 0F 29 1F` at
`$03:9EA7` and `CA C8 64 00 05 E3` at `$0F:98BA`. The spell name
table lists `Meteo` as spell `$2F`. The code decodes back to its raw
form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Only Tellah (after he learns it) and FuSoYa have Meteo.
- Monsters that cast Meteo also cast it with no wait.

## Spells and slow commands go off with no wait

**Codes:** `DDBC-67DE` + `DDBB-A70E` (use both)
**Raw form:** `039EAC:00` (orig `$1F`) + `039F9D:00` (orig `$03`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, Jump too:** `DDBB-ADAE` (`039F93:00`, orig `$04`)
**Recorded:** 2026-10-24 17:40:48

**Mechanism:** When a character picks an action, `CalcTimerDur`
(`battle/timer_dur.asm`) sets the wait before it goes off. Spells use
`TimerDur_03`, which takes the low 5 bits of byte 0 of the spell's
`AttackProp` entry:

```
        lda     f:AttackProp,x   ; magic action delay
_9eab:  and     #$1f
```

The first code changes `and #$1f` to `and #$00`, so every spell's
wait is 0. This is the general form of "Meteo is cast with no wait".

Other commands use `TimerDur_0c`, which reads `CmdDelayTbl`
(`$13:FE89`). In the USA 1.0 ROM, Recall, Aim, Kick, Dart and Twin
have a wait, and so do the commands that the USA release took away
(Dark Wave, Salve, Pray and Focus). A wait of 0 goes to `TimerDur_01`, the "immediate" case:

```
        lda     f:CmdDelayTbl,x   ; action delay for each command
        bne     @9fa1
        jmp     TimerDur_01
```

The second code changes the `bne` offset from `$03` to `$00`, so the
branch goes to the next instruction, and every command takes the
immediate path.

The optional code is for the wind-up before Jump, which
`TimerDur_0c` sets with `ldx #4` before the table read. It changes
the 4 to 0.

**Evidence:** `ff2_original.smc` has `BF A0 97 0F 29 1F AA 86 A9` at
`$03:9EA7`, `AD 7B 39 C9 06 D0 05 A2 04 00 80 33 AA BF 89 FE 13 D0
03 4C 71 9E` at `$03:9F8B`, and the USA `CmdDelayTbl` bytes at
`$13:FE89`. Each pattern occurs only once. The codes decode back to
their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Cecil and Kain have no spells at the start. Kain's Jump uses the
  optional code. Dark Wave comes later.
- The time the jumper stays in the air (`Cmd_06`, 4) does not change.
- FuSoYa's Regen ends when its 10-turn timer runs out, and that timer
  comes from `CmdDelayTbl` too. With the second code it ends at
  once, so Regen gives no HP.
- No USA class has Focus. With "Yang has a Power command that hits
  at once", its hit comes at once, with the charge bonus.
- Monster spells that use `TimerDur_03` are cast with no wait too.

## Tellah joins with 858 MP

**Codes:** `D7C1-0773` + `D7C1-0783` (use both)
**Raw form:** `0FA96C:03` (orig `$00`) + `0FA96E:03` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 17:42:22

**Mechanism:** This works like "Cecil starts with 9,928 HP".
`CharProp` (`$0FA900`, 32 bytes per entry) holds each character's
starting stats. Bytes 11-12 are current MP and bytes 13-14 are max
MP. Tellah is entry 3. When he first joins, `EventCmd_e7`
(`field/event.asm`) runs `LoadCharProp`, which copies his entry to
the party:

```
@e8ab:  lda     f:CharProp,x   ; character properties
        sta     $1000,y
```

Tellah has 90/90 MP (`$005A`). The codes change the high bytes of
current and max MP from `$00` to `$03`, so he has 858/858 MP
(`$035A`). When Tellah joins the 2nd and 3rd time, `CharRemoveTbl`
sends `EventCmd_e7` to `RestoreCharProp`, which brings back the stats
he had when he left, so he keeps the extra MP.

**Evidence:** `ff2_original.smc` has `84 03 14 00 00 00 00 54 01 54
01 5A 00 5A 00` at `$0F:A960` (Tellah, level 20, 340 HP, 90 MP).
`BF 00 A9 0F 99 00 10` (`lda f:CharProp,x : sta $1000,y`) is at
`$00:92D6` (`InitCharProp`) and `$00:E623` (`LoadCharProp`).
`CharRemoveTbl` has `$81` (restore slot 1) for character ids `$0A`
and `$0C`, Tellah's 2nd and 3rd times. The codes decode back to their
raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Tellah joins after the Mist Cave and Kaipo, not at the start.
- The codes are read when Tellah first joins. A save made after that
  keeps his old MP.
- The number was kept below the 999 shown in menus. A higher high
  byte (`$26` for 9,818) may not show right.

## Yang has a Power command that hits at once

**Codes:** `D2E5-096E` + `DDEC-61DE` (use both)
**Raw form:** `13FD76:0D` (orig `$FF`) + `13FEA8:00` (orig `$82`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 17:44:10

**Mechanism:** In the Japanese game, Yang's Focus ("Power" in other
translations) charges for 2 turns, then hits with double attack.
The USA 1.0 release took it away: Yang's list in `ClassCmd`
(`$13:FD55`, 5 bytes per class) is `00 0E 01 FF FF` (Fight, Kick,
Item) at `$13:FD73`. The first code puts command `$0D` (Focus) in
his fourth slot. The same idea is in "Paladin Cecil has the Dark Wave
command".

`Cmd_0d` (`battle/cmd.asm`) sets the focus status bit and queues
command `$1F`, `Cmd_1f`, with the wait from `CmdDelayTbl`:

```
Cmd_0d:
@e7f3:  inc     $390a
        lda     #$1f
        sta     $397b
        lda     #$0c
        sta     $d6
        lda     $cd
        jsr     CalcTimerDur
```

In the USA 1.0 ROM, entry `$1F` of `CmdDelayTbl` (`$13:FE89`) is
`$82`, 2 turns. The second code makes it `$00`. `TimerDur_0c` then
takes the immediate path, so `Cmd_1f` runs `DoFightCmd` at once. With
the focus bit set, `battle/fight.asm` doubles the attack ("attack
power +100%") and halves the target's defense.

**Evidence:** `ff2_original.smc` has `00 0E 01 FF FF` at `$13:FD73`
(class `$07`, Yang) and `00 00 ... 82 81 00 8A` (`CmdDelayTbl`, 35
bytes) at `$13:FE89`. In the command table at `$03:B36C`, entry `$0D`
is `$E7DB` and entry `$1F` is `$E80E`. The bytes there (`EE 0A 39 A9
1F 8D 7B 39 A9 0C 85 D6 A5 CD 20 2C 9E` and `20 89 C4 A5 CD 20 89
84`) match `Cmd_0d` and `Cmd_1f`. The codes decode back to their raw
forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked. The source shows that
1.1 changed `CmdDelayTbl` (entry `$1F` is `$01` there).

**Caveats:**

- Not tested.
- Yang joins at Mt. Hobs, not at the start.
- The USA command name for `$0D` is "Dummy", so the menu shows Dummy.
  The message `Cmd_0d` shows may also be blank or wrong in the USA
  ROM.
- The second code alone changes nothing in the USA game, since no
  class has Focus.
- Uses the same table as "Spells and slow commands go off with no
  wait", whose second code already makes Focus hit at once.
- The first code uses the same byte (`13FD76`, Yang's fourth slot) as
  the Yang code of the long entries that add a command, such as
  "Every character has the Jump command"; use only one of them.

## Kick hits every enemy with full power

**Code:** `6D3D-A702`
**Raw form:** `03E70D:80` (orig `$D0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:00:01

**Mechanism:** Kick (`Cmd_0e`, `battle/cmd.asm`) clears `$C1` and
runs `DoMultiAttack`, which hits every enemy. Kick's damage does not
change with the number of enemies. Instead, because it hits them all,
`DoMultiAttack` always halves the attack multiplier (`$201B`) for
Kick. Dark Wave sets `$C1` to 1 and skips this:

```
@e71c:  ldx     $a6
        lda     $201b,x
        sta     $df
        lda     $c1
        bne     @e72f
        lsr     $df
        lda     $df
        bne     @e72f
        inc     $df
```

The code changes `bne` (`$D0`) to `bra` (`$80`), so the `lsr` is
skipped, and Kick uses the full multiplier, like one Fight hit on
each enemy. The same halved value is used for the defense part of
the damage (`$84`), so that doubles too, but the damage still goes up
by about 2 times.

**Evidence:** `ff2_original.smc` has `BD 1B 20 85 DF A5 C1 D0 08 46
DF A5 DF D0 02 E6` at `$03:E706`, and the pattern occurs only once.
Entry `$0E` of the command table at `$03:B36C` is `$E69F`, and the
bytes there (`64 C1 4C`) match `Cmd_0e` (`stz $c1 : jmp
DoMultiAttack`). The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Only Yang has Kick. He joins at Mt. Hobs.
- Monsters that use Kick hit harder too.
- Dark Wave does not change.

## Healing hurts every monster, as if it were undead

**Codes:** `DD2D-D702` + `7E2F-DDD2` + `2D2F-DD02` (use all three)
**Raw form:** `03D40D:00` (orig `$40`) + `03D410:3F` (orig `$80`) +
`03D411:D0` (orig `$F0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:03:54

**Mechanism:** Bit 7 of creature type (`$2040`) is undead. Monsters
get it in `InitMonster` (`battle/init.asm`) only when their data has
a creature-type field, so there is no one byte that sets it for every
monster. Instead, the codes change the check in `RestoreHP` (magic
effect `$02`, `battle/magic.asm`), the code for Cure spells and other
HP healing:

```
RestoreHP:
@d40c:  lda     $2740
        and     #$80
        beq     _d416
        jmp     _d3ae
```

`$2740` is the target's creature type in the target buffer at
`$2700`. If the undead bit is set, the code jumps to `_d3ae`, `jsr
CalcDmg`, and the healing is done as damage. The codes change this to
`lda $2700 : and #$3f : bne _d416`. Byte 0 of a character holds its
character id (`rlcccccc`, never 0). Monsters never get a byte 0:
`InitMonster` does not write it and battle start clears it. So a
character target takes the normal healing path, and every monster
target takes the undead (damage) path.

**Evidence:** `ff2_original.smc` has `AD 40 27 29 80 F0 03 4C AE D3`
at `$03:D40C`, and the pattern occurs only once. `$03:D3AE` has `20 9F
C9 60` (`_d3ae: jsr CalcDmg : rts`). The RAM map gives `$2040` bit 7 as
undead and `$2000` as `rlcccccc` (same as `$1000`). The codes decode
back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- All three codes are needed. With only some of them, healing can
  hurt the party or work as before.
- Healing items use `RestoreHP` too, so a Cure1 thrown at a monster
  hurts it. Monsters that cure themselves or each other take damage.
- Spells that absorb an element (`CheckStrongElem`) also jump to
  `RestoreHP`. A monster that absorbs an element now takes damage from
  it.
- Only HP healing changes. Life still works as before on monsters
  that are not undead, and drain attacks do not change.
- Rosa joins later; at the start, only Cure items can heal.
- Works with "Healing items in battle refill all HP" and "Healing
  spells in battle refill all HP": a monster then takes damage, not
  a full heal, since those codes change the healing path.

## Rydia can cast Dispel (the unused "Dummy" spell)

**Code:** `DAAA-DF53`
**Raw form:** `0FC8C5:0C` (orig `$35`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:04:00

**Mechanism:** White spell `$0C` is named `Dummy` in the USA spell
name table. In the Japanese game it is Dispel, which FuSoYa learns.
The USA release took it out of FuSoYa's list in `SpellListInit`
(his list goes `0B 0D`, White then Peep), but the spell still works.
FuSoYa's auto-battle script against Zemus in `battle/battle_data.asm`
still casts it:

```
AutoBattle_0007:
@fe5d:  .byte   $00,$2f                 ; use magic: meteo
@fe5f:  .byte   $00,$0c                 ; use magic: dispel
```

Its `AttackProp` entry is intact: `A1 00 C6 10 00 0C`, effect `$10`,
12 MP. `MagicEffect_10` (`battle/magic.asm`) clears berserk (bit 2 of
status 2 at `$2704`), keeps only bits 7, 1 and 0 of status 4 at
`$2706` (so it removes reflect, barrier, image and HP leak), and
turns off the target's stop and sap timers:

```
@d8c9:  lda     $2706
        and     #$83
        sta     $2706
        ...
        lda     $29eb,x     ; disable stop and sap timers
        and     #$5f
```

The code puts `$0C` in place of Chocobo (`$35`) in Rydia's Call list
in `SpellListInit`, as in "Rydia starts with Bahamut". `DoMagicCmd`
casts IDs below `$19` as white magic.

**Evidence:** `ff2_original.smc` has `0E FF FF FF FF 35 FF` at
`$0F:C8C0`. The spell name table at `$0F:8900` has `Dummy` for spell
`$0C`. `AttackProp` (`$0F:97A0`, 6 bytes each) has `A1 00 C6 10 00
0C` at `$0F:97E8`. FuSoYa's white list at `$0F:C8E6` is `01 02 03 04
07 08 09 0A 0B 0D ...` (no `$0C`). The code decodes back to its raw
form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The menu shows the spell as "Dummy".
- Rydia joins after the village of Mist. She loses Chocobo.
- The code uses the same byte as the other "Rydia starts with ..."
  entries; use only one of them.
- Only a new game changes.

## Rydia can summon Cockatrice (the unused "Dummy" summon)

**Code:** `77AA-DF53`
**Raw form:** `0FC8C5:33` (orig `$35`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:04:05

**Mechanism:** Summon `$33` is named `Dummy` in the USA spell name
table. It sits between Bomb (`$32`) and Mage (`$34`, Mindflayer), the
slot of the Japanese Cockatrice. Its monster-call item (`$E9`) is
`Dummy` too. For a summon, `DoMagicCmd` calls `GetSummonAttack`
(`battle/magic.asm`), which turns spell `$33` into attack `$33 - $31 +
$4D = $4F`:

```
@e071:  sec
        sbc     #$31
        clc
        adc     #$4d
@e077:  sta     $26d2
```

Attack `$4F` is intact: its `AttackProp` entry is `80 00 B2 18 16
8F`, effect `$18` (`MagicEffect_18`, "magic effect $18: cockatrice"),
which deals damage and adds the attack's status. The summon graphics
are still there too: `LoadSummonGfx` uses entry `$4F - $4D = 2` of
`SummonPalTbl` and `SummonGfxPtrs`, and `CockatriceAnim`
(`btlgfx/summon.asm`) is in the ROM. The code puts `$33` in place of
Chocobo (`$35`) in Rydia's Call list in `SpellListInit`.

**Evidence:** `ff2_original.smc` has `0E FF FF FF FF 35 FF` at
`$0F:C8C0`. The spell name table has `Dummy` for `$33`, the attack
name table (`$0F:8AB0`, 8 bytes each from attack `$48`) has `Dummy`
for `$4F`, and `AttackProp` has `80 00 B2 18 16 8F` at `$0F:997A`.
`SummonPalTbl` at `$0D:FE4C` is `00 55 24 3F ...` and `SummonGfxPtrs`
at `$0D:87F0` has `$8B18` for entry 2. The code decodes back to its
raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The menu and the attack name window show "Dummy".
- Status byte `$16` was not decoded here; the Japanese Cockatrice
  turns targets to stone.
- Rydia joins after the village of Mist. She loses Chocobo.
- The code uses the same byte as the other "Rydia starts with ..."
  entries; use only one of them.
- Only a new game changes.

## The field shows the party's map position (unused debug display)

**Codes:** `516F-A4D7` + `5164-AFA7` (use both)
**Raw form:** `008F18:76` (orig `$3D`) + `008F27:76` (orig `$3D`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:04:10

**Mechanism:** `DrawPos` (`field/field.asm`, "draw player's current
position") is debug code. The source notes it is "unused but works in
the english translation". It writes the X and Y position (`$1706`,
`$1707`) and the map ID (`$1702`) in hex to VRAM `$2882` and `$28C2`,
on BG3 (`hBG3SC` = `$29`, the dialog window layer). Its only caller,
`_15c23d`, calls it only in a `DEBUG` build. In the retail ROM
`_15c23d` is a lone `rtl`, and `DrawPos` follows at `$15:C276`.

`WaitVblankLong` and `WaitVblankShort` call `_15c23d` once a frame:

```
@9131:  lda     $7d
        bne     @9131
        jsl     _15c23d
```

The codes change the low byte of both `jsl` operands from `$3D` to
`$76`, so each frame calls `DrawPos` instead.

**Evidence:** The `@xxxx` labels don't match the USA ROM.
`ff2_original.smc` has `A9 01 85 7D A5 7D D0 FC 22 3D C2 15 E6 7D 60`
at `$00:8F0F` (`WaitVblankLong`) and the same with `$7E` at `$00:8F1E`
(`WaitVblankShort`). `22 3D C2 15` occurs only at these two places.
`$15:C23D` is `6B` (`rtl`), and `$15:C276` starts `A9 80 8D 15 21`
(`lda #$80 / sta $2115`), the start of `DrawPos`. The codes decode back
to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The numbers are hex: X and Y on one row, the map ID below them.
- `DrawPos` writes VRAM right after the NMI, without forced blank. If
  the NMI work runs past vblank, the writes may land wrong or not at
  all.
- Dialog windows also use BG3, so a window can cover or erase the
  numbers. The digits may not show if BG3 is off on a map.
- With only one code, the display shows only on frames that use that
  wait routine.

## Battles keep running while you choose a spell or item (Active mode)

**Code:** `6DC4-AD0C`
**Raw form:** `02AB21:80` (orig `$D0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:04:15

Tags: `demo-ready` `curated`

**Mechanism:** The Japanese config menu has an Active/Wait battle
mode, `$16BE` (`ToggleActiveWait` in `menu/config.asm`). The USA
release builds the menu with `SIMPLE_CONFIG`, which leaves the option
out, so `$16BE` stays 0 (Wait). The battle code still checks it.
`PauseBattleWait` (`btlgfx/menu.asm`) runs when a spell or item
window opens, and pauses the battle in Wait mode:

```
PauseBattleWait:
@ab20:  lda     $16be
        bne     @ab28
        inc     $38da
@ab28:  rts
```

The code changes `bne` (`$D0`) to `bra` (`$80`), so the pause is
always skipped, as in Active mode. Monsters keep acting while the
window is open.

**Evidence:** `ff2_original.smc` has `AD BE 16 D0 03 EE DA 38 60` at
`$02:AB1E` (`PauseBattleWait`), and `AD BE 16 D0 03 9C DA 38 60`
(`UnpauseBattleWait`) right after it. The first pattern occurs only
once. The source has no write to `$16BE` outside the config menu. The
code decodes back to its raw form with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron" and 7 other cheats in one test group.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- `UnpauseBattleWait` still clears `$38DA` when the window closes;
  with nothing paused, that has no effect.
- The battle speed setting still applies.

## Menus remember the cursor position (unused cursor memory option)

**Codes:** `6D61-D4A5` + `6D6D-D4A1` (use both)
**Raw form:** `018C6B:80` (orig `$D0`) + `01880B:80` (orig `$D0`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:04:20

**Mechanism:** The Japanese config menu has a cursor
reset/memory option, `$16B7`. The USA release builds the menu with
`SIMPLE_CONFIG`, which leaves it out, so `$16B7` stays 0 (reset). The
menu code still checks it. `ClearCursorMem` (`menu/menu.asm`) clears
the saved menu cursors at `$1B00`-`$1BEB` unless cursor memory is on,
and `MainMenuMain` resets the main menu cursor (`$1A76`, `$1A77`):

```
ClearCursorMem:
@8c65:  pha
        lda     $16b7
        bne     @8c7b       ; branch if memory cursor
...
MainMenuMain:
...
        lda     $16b7
        bne     @8813
        stz     $1a76
        stz     $1a77
```

The codes change both `bne` (`$D0`) to `bra` (`$80`), so the menus
act as if cursor memory were on. The first code keeps the sub-menu
cursors (items, magic, equip ...). The second keeps the main menu
cursor.

**Evidence:** `ff2_original.smc` has `48 AD B7 16 D0 10` at `$01:8C67`
(`ClearCursorMem`) and `AD B7 16 D0 06 9C 76 1A` at `$01:8808`
(`MainMenuMain`). These are the only two `AD B7 16 D0` in the ROM. The
codes decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Only the field menus change. The battle menu doesn't read `$16B7`.
- Each code works on its own.

## Tellah has the Recall command

**Code:** `D5E1-01DE`
**Raw form:** `13FD68:07` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:04:25

**Mechanism:** This works like "Paladin Cecil has the Dark Wave
command". The USA release took Recall (command `$07`) away from Tellah,
but `Cmd_07` (`battle/cmd.asm`) is still in the ROM and in `CmdTbl`. At
the start of each battle, `battle/init.asm` copies 5 command ids per
class from `ClassCmd` (`$13:FD55`) to `$3303`, and the targeting flags
from `CmdTargetTbl`. Tellah's list is `00 02 03 01 FF` (Fight, White,
Black, Item) at `$13:FD64`. The code puts `$07` in the first empty slot.

`Cmd_07` picks a random number from 0 to `$9F`. Below `$80` it casts one
of 8 black spells: Stone, Virus, Weak, Fatal (1 in 20 each), or Fire1,
Ice-1, Lit-1, Toad (3 in 20 each). From `$80` up (1 in 5) it fails with
a message. It casts only if the user has enough MP, and it uses the MP
(`inc $355d`):

```
Cmd_07:
@ea1d:  clr_ax
        lda     #$a0
        jsr     RandXA
        cmp     #$08
        bcs     @ea2c
        lda     #$2a
```

**Evidence:** `ff2_original.smc` has `00 02 03 01 FF` at `$13:FD64`.
Entry `$07` of `CmdTbl` (`$03:B36C`) is `$EA05`, and `$03:EA05` has `7B
AA A9 A0 20 79 83`, the start of `Cmd_07`. `CmdTargetTbl` at `$13:FDC3`
matches the source. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The USA command name for `$07` is "Dummy", so the menu shows Dummy.
- Tellah joins after the Mist Cave and Kaipo. His later classes (`$0A`,
  `$0C`) don't get it.
- If Tellah has too little MP for the spell it picks, nothing happens.
- What the battle messages for Recall say in the USA version was not
  checked.
- The code uses the same byte as the "Every character has the ...
  command" entries in `ff2_long_game_genie.md`.

## Edward has the Salve command

**Code:** `DCE1-050E`
**Raw form:** `13FD6D:0A` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:04:30

**Mechanism:** This works like "Paladin Cecil has the Dark Wave
command". The USA release took Salve (command `$0A`) away from Edward,
but `Cmd_0a` (`battle/cmd.asm`) is still in the ROM and in `CmdTbl`. At
the start of each battle, `battle/init.asm` copies 5 command ids per
class from `ClassCmd` (`$13:FD55`) to `$3303`, and the targeting flags
from `CmdTargetTbl`. Edward's list is `00 08 09 01 FF` (Fight, Sing,
Hide, Item) at `$13:FD69`. The code puts `$0A` in the first empty slot.

`Cmd_0a` looks for item `$CE` (Cure1) in the battle inventory at
`$321B`. If it finds one, it takes one away and uses it through
`DoMagicAttack`, with the targets from `CmdTargetTbl` (`$20` for Salve,
the same as Pray). With no Cure1 it shows a message:

```
Cmd_0a:
@e2df:  lda     #$c1
        sta     $33c4
        clr_axy
@e2e7:  lda     $321b,x
        cmp     #$ce
        beq     @e302
```

**Evidence:** `ff2_original.smc` has `00 08 09 01 FF` at `$13:FD69`.
Entry `$0A` of `CmdTbl` (`$03:B36C`) is `$E2C7`, and `$03:E2C7` has `A9
C1 8D C4 33`, the start of `Cmd_0a`. `CmdTargetTbl` at `$13:FDC3`
matches the source. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The USA command name for `$0A` is "Dummy", so the menu shows Dummy.
- Edward joins at Damcyan.
- Salve uses up a Cure1 potion each time. It does nothing without one.
- What the battle messages for Salve say in the USA version was not
  checked.
- The code uses the same byte as the "Every character has the ...
  command" entries in `ff2_long_game_genie.md`.

## Rosa has the Pray command

**Code:** `D8E5-006E`
**Raw form:** `13FD72:0B` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:04:35

**Mechanism:** This works like "Paladin Cecil has the Dark Wave
command". The USA release took Pray (command `$0B`) away from Rosa, but
`Cmd_0b` (`battle/cmd.asm`) is still in the ROM and in `CmdTbl`. At the
start of each battle, `battle/init.asm` copies 5 command ids per class
from `ClassCmd` (`$13:FD55`) to `$3303`, and the targeting flags from
`CmdTargetTbl`. Rosa's list is `00 02 0C 01 FF` (Fight, White, Aim,
Item) at `$13:FD6E`. The code puts `$0B` in the first empty slot.

`Cmd_0b` casts Cure1 (spell `$0E`) with no MP cost half of the time, and
shows a fail message the other half:

```
Cmd_0b:
@e855:  jsr     Rand
        cmp     #$80
        bcc     @e866
        ...
@e866:  lda     #$0e
        sta     $26d2
        jsr     DoMagicAttack
```

**Evidence:** `ff2_original.smc` has `00 02 0C 01 FF` at `$13:FD6E`.
Entry `$0B` of `CmdTbl` (`$03:B36C`) is `$E83D`, and `$03:E83D` has `20
93 85 C9 80 90 0A`, the start of `Cmd_0b`. `CmdTargetTbl` at `$13:FDC3`
matches the source. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The USA command name for `$0B` is "Dummy", so the menu shows Dummy.
- Rosa joins after the Antlion Cave. Her 2nd class (`$10`) doesn't get
  it.
- What the battle messages for Pray say in the USA version was not
  checked.
- The code uses the same byte as the "Every character has the ...
  command" entries in `ff2_long_game_genie.md`.

## Yang has the Brace command

**Code:** `DEE5-096E`
**Raw form:** `13FD76:0F` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:04:40

**Mechanism:** This works like "Paladin Cecil has the Dark Wave
command". The USA release took Brace (command `$0F`) away from Yang, but
`Cmd_0f` (`battle/cmd.asm`) is still in the ROM and in `CmdTbl`. At the
start of each battle, `battle/init.asm` copies 5 command ids per class
from `ClassCmd` (`$13:FD55`) to `$3303`, and the targeting flags from
`CmdTargetTbl`. Yang's list is `00 0E 01 FF FF` (Fight, Kick, Item) at
`$13:FD73`. The code puts `$0F` in the first empty slot.

`Cmd_0f` casts spell `$05`, Armor (the comment calls it "barrier
spell"), through `DoMagicAttack`, with no MP cost:

```
Cmd_0f:
@e699:  lda     #$05        ; barrier spell (armor)
        sta     $26d2
        stz     $33c4
        jsr     DoMagicAttack
```

**Evidence:** `ff2_original.smc` has `00 0E 01 FF FF` at `$13:FD73`.
Entry `$0F` of `CmdTbl` (`$03:B36C`) is `$E681`, and `$03:E681` has `A9
05 8D D2 26 9C C4 33`, the start of `Cmd_0f`. `CmdTargetTbl` at
`$13:FDC3` matches the source. The code decodes back to its raw form
with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The USA command name for `$0F` is "Dummy", so the menu shows Dummy.
- Yang joins at Mt. Hobs. His 2nd class (`$0D`) doesn't get it.
- "Yang has a Power command that hits at once" uses the same byte. To
  use both, use `DEE5-09AE` (`13FD77:0F`, his 5th slot) instead of this
  code.
- What the battle messages for Brace say in the USA version was not
  checked.
- The code uses the same byte as the "Every character has the ...
  command" entries in `ff2_long_game_genie.md`.

## Palom has the Bluff command

**Code:** `FFE5-05DE`
**Raw form:** `13FD7C:11` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:04:45

**Mechanism:** This works like "Paladin Cecil has the Dark Wave
command". The USA release took Bluff (command `$11`) away from Palom,
but `Cmd_11` (`battle/cmd.asm`) is still in the ROM and in `CmdTbl`. At
the start of each battle, `battle/init.asm` copies 5 command ids per
class from `ClassCmd` (`$13:FD55`) to `$3303`, and the targeting flags
from `CmdTargetTbl`. Palom's list is `00 03 10 01 FF` (Fight, Black,
Twin, Item) at `$13:FD78`. The code puts `$11` in the first empty slot.

`Cmd_11` adds 16 to the user's modified intellect (`$2017`), up to 99,
so his black magic hits harder for the rest of the battle:

```
Cmd_11:
        ldx     $a6
        clc
        lda     $2017,x
        adc     #$10
        cmp     #$63
        bcc     @eade
        lda     #$63
@eade:  sta     $2017,x
```

**Evidence:** `ff2_original.smc` has `00 03 10 01 FF` at `$13:FD78`.
Entry `$11` of `CmdTbl` (`$03:B36C`) is `$EAAE`, and `$03:EAAE` has `A9
11 8D C8 34 A9 10 8D C7 34`, the start of `Cmd_11`. `CmdTargetTbl` at
`$13:FDC3` matches the source. The code decodes back to its raw form
with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The USA command name for `$11` is "Dummy", so the menu shows Dummy.
- Palom joins at Mysidia.
- What the battle messages for Bluff say in the USA version was not
  checked.
- The code uses the same byte as the "Every character has the ...
  command" entries in `ff2_long_game_genie.md`.

## Porom has the Cry command

**Code:** `F4E6-000E`
**Raw form:** `13FD81:12` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:04:50

**Mechanism:** This works like "Paladin Cecil has the Dark Wave
command". The USA release took Cry (command `$12`) away from Porom, but
`Cmd_12` (`battle/cmd.asm`) is still in the ROM and in `CmdTbl`. At the
start of each battle, `battle/init.asm` copies 5 command ids per class
from `ClassCmd` (`$13:FD55`) to `$3303`, and the targeting flags from
`CmdTargetTbl`. Porom's list is `00 02 10 01 FF` (Fight, White, Twin,
Item) at `$13:FD7D`. The code puts `$12` in the first empty slot.

`Cmd_12` lowers each monster's level stat (`$202F`) by half of the
user's level stat, to no less than 1. See "Every character has the Cry
command" (`ff2_long_game_genie.md`) for the code. This entry gives it
back only to Porom, its user in the Japanese game.

**Evidence:** `ff2_original.smc` has `00 02 10 01 FF` at `$13:FD7D`.
Entry `$12` of `CmdTbl` (`$03:B36C`) is `$EAD1`, and `$03:EAD1` has `A9
12 8D C8 34`, the start of `Cmd_12`. `CmdTargetTbl` at `$13:FDC3`
matches the source. The code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The USA command name for `$12` is "Dummy", so the menu shows Dummy.
- Porom joins at Mysidia.
- The code is also in "Every character has the Cry command".
- What the battle messages for Cry say in the USA version was not
  checked.
- The code uses the same byte as the "Every character has the ...
  command" entries in `ff2_long_game_genie.md`.

## FuSoYa has the Regen command

**Code:** `FBE8-00AE`
**Raw form:** `13FDB3:19` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:04:55

**Mechanism:** This works like "Paladin Cecil has the Dark Wave
command". The USA release took Regen (command `$19`) away from FuSoYa,
but `Cmd_19` (`battle/cmd.asm`) is still in the ROM and in `CmdTbl`. At
the start of each battle, `battle/init.asm` copies 5 command ids per
class from `ClassCmd` (`$13:FD55`) to `$3303`, and the targeting flags
from `CmdTargetTbl`. FuSoYa's list is `00 02 03 01 FF` (Fight, White,
Black, Item) at `$13:FDAF`. The code puts `$19` in the first empty slot.

`Cmd_19` starts the regen counter (`$357C`, "hp gain every 5 ticks")
with 10 HP per gain (`$357D`), and queues command `$22` (`Cmd_22`, end
regen) on the user's action timer:

```
Cmd_19:
@e3fd:  stz     $357c
        ldx     #10         ; 10 hp gain per tick
        stx     $357d
        ...
        lda     #$22
        sta     $397b
```

`Cmd_22` sets `$357C` back to `$FF` (off).

**Evidence:** `ff2_original.smc` has `00 02 03 01 FF` at `$13:FDAF`.
Entry `$19` of `CmdTbl` (`$03:B36C`) is `$E3E5`, and `$03:E3E5` has `9C
7C 35 A2 0A 00 8E 7D 35`, the start of `Cmd_19`. `CmdTargetTbl` at
`$13:FDC3` matches the source. The code decodes back to its raw form
with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The USA command name for `$19` is "Dummy", so the menu shows Dummy.
- FuSoYa joins late, at the Lunar Subterrane.
- With the second code of "Spells and slow commands go off with no
  wait", Regen ends at once and gives no HP.
- What the battle messages for Regen say in the USA version was not
  checked.
- The code uses the same byte as the "Every character has the ...
  command" entries in `ff2_long_game_genie.md`.

## Running away never drops gil

**Code:** `1060-DFD3`
**Raw form:** `038844:64` (orig `$32`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:05:04

**Mechanism:** After the party runs away, the battle end code in
`battle/battle.asm` calls `DropGil` ("drop gil from running away").
It drops gil only half the time:

```
DropGil:
@8840:  jsr     Rand99
        cmp     #50                     ; 50% chance to drop gil
        bcs     @884a
        jmp     @8920
```

`Rand99` returns 0 to 98 (`RandXA` with 0 and 98). The code changes
`cmp #50` to `cmp #100`. The random number is then always less, so
`bcs` is never taken, and `DropGil` always jumps to its `rts` at
`@8920`. No gil is lost and no "dropped gil" message shows.

**Evidence:** `ff2_original.smc` has `20 8B 85 C9 32 B0 03 4C 20 89`
at `$03:8840`. `$03:858B` has `7B AA A9 62 20 79 83 60` (`Rand99`:
`clr_ax : lda #98 : jsr RandXA : rts`), and `$03:8920` is `60`
(`rts`). The code decodes back to its raw form with
`snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Running away works as before; only the gil drop is gone.
- Works with "Run away the moment a battle starts".

## Time stops while any battle menu is open

**Codes:** `0C6E-0D02` + `DD6E-0D62` + `DDC1-6D6C` (use all three)
**Raw form:** `0385F1:4A` (orig `$DA`) + `0385F2:00` (orig `$38`) +
`02AA62:00` (orig `$01`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:11:23

**Mechanism:** `BattleMain` (`battle/battle.asm`) runs the battle
clock one frame at a time, and stops it while either of two flags is
set:

```
@85e8:  lda     #$02                    ; battle graphics $02: wait one frame
        jsr     ExecBtlGfx
        lda     $38d9
        ora     $38da
        bne     @85e8                   ; branch if paused or wait mode
```

`$38D9` is the Start pause. `$38DA` is the wait-mode flag, which
`PauseBattleWait` (`btlgfx/menu.asm`) sets only when the item or a
spell list opens. The time keeps running while the command list is
open.

`$4A` has one bit for each open battle menu window: `$02` command
list, `$04` inventory, `$08` equipment, `$10`/`$20`/`$40` the spell
lists, `$80` MP cost (`OpenCmdWindow`, `OpenInventoryWindow` and the
others set them with `tsb $4a` and clear them with `trb $4a`). The
first two codes change `ora $38da` to `ora $004a`, so the clock stops
while any of these windows is open. Bit `$01` is the name window,
which `OpenNameWindow` opens for the whole battle (`CloseNameWindow`
is unused). The third code changes its `lda #$01 : tsb $4a` to `lda
#$00`, so the bit is never set and the clock can still run when no
menu is open. No code reads bit `$01`.

**Evidence:** `ff2_original.smc` has `A9 02 20 85 80 AD D9 38 0D DA
38 D0 F3` at `$03:85E8` and `A9 01 04 4A 7B 20 69 A9 A9 02 8D 1E 18`
(`OpenNameWindow`) at `$02:AA61`. `A2 00 00 86 5F A9 02 04 4A` at
`$02:AA78` matches the start of `OpenCmdWindow`. Battle code runs
with the data bank at `$7E` and the direct page at 0, so `$7E:004A`
is the same byte as `$4A`. The codes decode back to their raw forms
with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- All three codes are needed. Without the third code, the clock
  never runs and the battle can't go on.
- Monsters only act while no character's command list is open. When
  several characters are ready, time stops until each one has picked
  a command.
- The Start pause works as before.

## Blind, toad and mini don't weaken Fight attacks

**Codes:** `DDAD-0462` + `DDAC-0402` + `DDA2-0762` + `DDA0-AD02` +
`DDA9-AD02` (use all five)
**Raw form:** `03C50A:00` (orig `$02`) + `03C5A9:00` (orig `$02`) +
`03C5DE:00` (orig `$30`) + `03C741:00` (orig `$30`) + `03C751:00`
(orig `$30`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:15:17

**Mechanism:** The Fight attack code (`battle/fight.asm`) tests status
1 of the attacker (`$2683`) and the target (`$2703`) in five places
and lowers a stat when a bit is set. Bit `$02` is blind; `$30` is toad
and mini. Each code changes the `and #mask` operand to `#$00`, so the
`beq` after it always skips the penalty:

- `DDAD-0462`: blind attacker. `lsr $38f8` would halve the hit rate.
- `DDAC-0402`: blind target. `lsr $38f9` would halve the defense rate.
- `DDA2-0762`: toad or mini target. `stz $38fb` would zero the defense
  multiplier.
- `DDA0-AD02`: toad or mini attacker. The attack power would be set to
  1 and the critical hit flag cleared.
- `DDA9-AD02`: toad or mini target. The attack power would be doubled
  and the defense set to 0.

```
@c73d:  lda     $2683
        and     #$30
        beq     @c74d       ; branch if attacker is not toad or mini
        ldx     #$0001
        stx     $3902       ; set attack power to 1
        stz     $3900       ; clear crit flag
```

**Evidence:** `ff2_original.smc` has `AD 83 26 29 02 F0 03 4E` at
`$03:C506`, `AD 03 27 29 02 F0 03 4E` at `$03:C5A5`, `AD 03 27 29 30
F0 03 9C` at `$03:C5DA`, `AD 83 26 29 30 F0 09 A2` at `$03:C73D` and
`AD 03 27 29 30 F0 0B 0E` at `$03:C74D`. The mask bytes are at offset
4 of each. Every code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The code is shared by the party and the monsters. Blind, toad and
  mini monsters keep their stats too, and a toad or mini character no
  longer takes double damage. Use only the first and fourth codes to
  change the attacker side alone.
- Only Fight-type attacks (Fight, Aim, Kick, Jump and others that run
  `DoFightCmd`) change. Spells are not affected.
- The status still has its other effects: toads and pigs can only use
  some commands, and blind still shows.

## Start a new game with 99 of an unused attack item

**Codes:** Bomblet: `8D6E-6DDD` + `176E-646D`; BigBomb: `8F6E-6DDD` +
`176E-646D`; Notus: `846E-6DDD` + `176E-646D`; Boreas: `876E-6DDD` +
`176E-646D`; ThorRage: `806E-6DDD` + `176E-646D`; ZeusRage: `896E-6DDD` +
`176E-646D`; Stardust: `816E-6DDD` + `176E-646D`; Succubus: `856E-6DDD` +
`176E-646D`; Vampire: `866E-6DDD` + `176E-646D`; WhiteFang: `AF6E-6DDD` +
`176E-646D`; Kamikaze: `A06E-6DDD` + `176E-646D`; EarthDrum: `A56E-6DDD` +
`176E-646D`; Coffin: `AB6E-6DDD` + `176E-646D` (use one pair)
**Raw form:** `0082F0:<item>` (orig `$FE`) + `0082FA:63` (orig `$01`).
The items are Bomblet `$B0`, BigBomb `$B1`, Notus `$B2`, Boreas `$B3`,
ThorRage `$B4`, ZeusRage `$B5`, Stardust `$B6`, Succubus `$B7`, Vampire
`$B8`, WhiteFang `$C1`, Kamikaze `$C4`, EarthDrum `$C7` and Coffin `$C9`.
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:42:20

**Mechanism:** The USA release took these Japanese battle items out of
the game. Their names in the item name table (`$0F:8000`) were changed
to `Dummy`, but their battle data in `ItemProp` (`$0F:9680`, 6 bytes per
item from `$B0`) was kept.

The Japanese names, from the item name table of the Japanese 1.0 and
1.1 ROMs (the same address, `$0F:8000`):

| Item | Japanese name | Meaning |
|---|---|---|
| `$B0` Bomblet | ボムのかけら | bomb fragment |
| `$B1` BigBomb | ボムのみぎうで | bomb's right arm |
| `$B2` Notus | なんきょくのかぜ | antarctic wind |
| `$B3` Boreas | ほっきょくのかぜ | arctic wind |
| `$B4` ThorRage | ゼウスのいかり | Zeus's wrath |
| `$B5` ZeusRage | かみがみのいかり | wrath of the gods |
| `$B6` Stardust | ほしのすな | star sand |
| `$B7` Succubus | リリスのくちづけ | Lilith's kiss |
| `$B8` Vampire | バンパイアのきば | vampire fang |
| `$C1` WhiteFang | しろいきば | white fang |
| `$C4` Kamikaze | ボムのたましい | bomb's soul |
| `$C7` EarthDrum | だいちのドラム | earth drum |
| `$C9` Coffin | クアールのひげ | coeurl whisker |

The code names are the English names from the Free Enterprise
randomizer's item list (`github.com/goladus/ff4fepr`, `items.yml`), but
`$C1` is named from its Japanese name: it is the white (ice) one of the
three fangs. FireBomb (`$C0`) and Lit-Bolt (`$C2`) are the red and blue
fangs.

The codes use the new-game Sort slot, like "Start a new game with 99
Elixirs". `InitNewGame` (`field/field.asm`) puts item `$FE` with
quantity 1 in slot `$149C`:

```
        lda     #$fe
        sta     $149c
        lda     #$ff
        sta     $149e
        lda     #1
        sta     $149d
        sta     $149f
        rts
```

The first code of each pair changes `lda #$fe` to the item ID. The
second changes `lda #1` to `lda #99`.

In battle, the item menu lets a character use any item from `$B0` to
`$DD` (`CheckCanUseItem` in `battle/char.asm`: `cmp #$de` / `cmp
#$b0`), and the item command runs the effect in its `ItemProp` row.
Byte 3 of the row is the magic effect and byte 1 is its power:

| Item | Row | Effect |
|---|---|---|
| Bomblet, BigBomb | `C0 03 64 00 01 00`, `C0 10 64 00 01 00` | `$00` magic damage, power 3 / 16, element `$01` (as FireBomb) |
| Notus, Boreas | `C0 03 64 00 02 00`, `C0 10 64 00 02 00` | `$00` magic damage, power 3 / 16, element `$02` |
| ThorRage, ZeusRage | `C0 03 64 00 03 00`, `C0 10 64 00 03 00` | `$00` magic damage, power 3 / 16, element `$03` (as Lit-Bolt) |
| Stardust | `C0 14 64 00 00 80` | `$00` magic damage, power 20, no element |
| Succubus | `A0 08 64 05 07 00` | `$05` drain MP, power 8 |
| Vampire | `A0 10 64 04 07 00` | `$04` drain HP, power 16 |
| WhiteFang | `C0 02 64 14 02 80` | `$14`, the FireBomb and Lit-Bolt effect, element `$02` |
| Kamikaze | `A0 01 64 25 80 80` | `$25` explode |
| EarthDrum | `C0 14 64 00 00 80` | `$00` magic damage, power 20, no element |
| Coffin | `A0 00 E4 06 17 00` | `$06` set status `$17` |

**Evidence:** `ff2_original.smc` has `A9 FE 8D 9C 14 A9 FF 8D 9E 14 A9
01 8D 9D 14 8D 9F 14 60` at `$00:82EF`. The item name table at
`$0F:8000` has `Dummy` for each of these items, `FireBomb` for `$C0` and
`Lit-Bolt` for `$C2`. The `ItemProp` rows above were read from
`$0F:9680`. The Japanese 1.0 ROM (`Final Fantasy IV (Japan).sfc`, CRC32
`21027C5D`) and 1.1 ROM (`(Rev A)`, CRC32 `CAA15E97`) have the same
`ItemProp` rows and the same item animation IDs (`ItemMagicAnim`,
`$0F:D4E0`). The animation scripts and frames (`$0F:D67E`-`$0F:FEBE`) are
the same as in the USA ROM, and `AnimProp` (`$0F:A050`) differs only in
entries `$4F`, `$59` and `$5A`, which these items don't use.
`CheckCanUseItem` is at `$03:A158` in the USA ROM (`C9 6D 90 0A C9 DE
B0`). Every code decodes back to its raw form with
`snes_gg_encode.py`. Bomblet and Vampire were tested in Mesen2 together
with the 3 codes `B864-D0D6` + `D164-D006` + `3B6A-D16C` of "Skip the
opening and start outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only Bomblet and Vampire were tested. The other items were not.
- The item shows as `Dummy` in the menus and in battle, and the USA item
  descriptions for it are blank or say "Dummy".
- The English code names are not in any ROM. Only the Japanese names
  are.
- Kamikaze uses the explode effect; the user may die or be removed from
  battle, as with a monster's Explode. Coffin may be death; status
  `$17` was not checked.
- The Sort entry is gone, so the item menu can't sort. The TrashCan's
  quantity becomes 99 too.
- Only a new game changes. Uses the same bytes as "Start a new game with
  99 Elixirs", "Start a new game with the Tower key" and the other
  unused-item entries; use only one of them.

## Start a new game with 99 of an unused support item

**Codes:** Bacchus: `8B6E-6DDD` + `176E-646D`; Hermes: `8C6E-6DDD` +
`176E-646D`; Hourglass1: `886E-6DDD` + `176E-646D`; Hourglass2:
`8A6E-6DDD` + `176E-646D`; Hourglass3: `826E-6DDD` + `176E-646D`;
SilkWeb: `836E-6DDD` + `176E-646D`; Illusion: `8E6E-6DDD` + `176E-646D`;
StarVeil: `A76E-6DDD` + `176E-646D`; MoonVeil: `A96E-6DDD` + `176E-646D`;
MuteBell: `A16E-6DDD` + `176E-646D`; Grimoire: `AC6E-6DDD` + `176E-646D`;
Bestiary: `A86E-6DDD` + `176E-646D`; AlarmClock: `AA6E-6DDD` +
`176E-646D`; UnicornHorn: `A26E-6DDD` + `176E-646D` (use one pair)
**Raw form:** `0082F0:<item>` (orig `$FE`) + `0082FA:63` (orig `$01`).
The items are Bacchus `$B9`, Hermes `$BA`, Hourglass1 `$BB`, Hourglass2
`$BC`, Hourglass3 `$BD`, SilkWeb `$BE`, Illusion `$BF`, StarVeil `$C3`,
MoonVeil `$C5`, MuteBell `$C6`, Grimoire `$CA`, Bestiary `$CB`,
AlarmClock `$CC` and UnicornHorn `$CD`.
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:42:25

**Mechanism:** These are more of the Japanese battle items that the USA
release renamed `Dummy` but kept in `ItemProp` (`$0F:9680`). Their
Japanese names:

| Item | Japanese name | Meaning |
|---|---|---|
| `$B9` Bacchus | バッカスのさけ | Bacchus's wine |
| `$BA` Hermes | エルメスのくつ | Hermes's shoes |
| `$BB`-`$BD` Hourglass1-3 | どう / ぎん / きんのすなどけい | bronze / silver / gold hourglass |
| `$BE` SilkWeb | くものいと | spider thread |
| `$BF` Illusion | スケープドール | scapegoat doll |
| `$C3` StarVeil | ひかりのカーテン | curtain of light |
| `$C5` MoonVeil | つきのカーテン | moon curtain |
| `$C6` MuteBell | せいじゃくのかね | bell of silence |
| `$CA` Grimoire | おうぎのしょ | book of secret arts |
| `$CB` Bestiary | かいじゅうずかん | monster encyclopedia |
| `$CC` AlarmClock | めざましどけい | alarm clock |
| `$CD` UnicornHorn | ユニコーンのつの | unicorn horn |

The codes
put 99 of one in the new-game Sort slot, as in "Start a new game with 99
of an unused attack item". The battle item menu allows every item from
`$B0` to `$DD`.

| Item | Row | Effect |
|---|---|---|
| Bacchus | `00 00 E4 06 0A 80` | `$06` set status `$0A` (berserk, by its name) |
| Hermes | `00 F8 64 0F 00 80` | `$0F` haste/slow, power `$F8` |
| Hourglass1, 2, 3 | `C0 32 E4 91 15 00`, `C0 64 ...`, `C0 C8 ...` | `$11` stop, power 50 / 100 / 200 |
| SilkWeb | `C0 10 64 0F 00 00` | `$0F` haste/slow, power 16 |
| Illusion | `00 00 64 08 00 00` | `$08` blink |
| StarVeil | `00 0A 64 09 0D 00` | `$09` reflect (wall) |
| MoonVeil | `00 14 64 09 32 00` | `$09` reflect (wall) |
| MuteBell | `C0 00 E4 06 12 00` | `$06` set status `$12` |
| Grimoire | `00 00 E4 00 00 80` | a random summon (see below) |
| Bestiary | `A0 00 64 12 00 80` | `$12` libra (peep) |
| AlarmClock | `40 00 64 0B 1A 00` | `$0B` esuna (cure status) |
| UnicornHorn | `40 00 64 0B 19 00` | `$0B` esuna (cure status) |

The Grimoire's row is empty because the item command handles it before
the row is used:

```
        lda     $26d2
        cmp     #$ca        ; summon book (call)
        bne     @ebc7
        jsr     RandSummon
```

**Evidence:** The new-game bytes at `$00:82EF` and the name table are as
in "Start a new game with 99 of an unused attack item". The `ItemProp`
rows above were read from `$0F:9680`. `ff2_original.smc` has `C9 CA D0
05` (`cmp #$ca`, `bne`) at `$03:EBA4`. The Japanese names, rows and
animations were checked as in the attack item entry. Every code
decodes back to its raw form with `snes_gg_encode.py`.
Hermes and MuteBell were tested in Mesen2 together with the 3 codes
`B864-D0D6` + `D164-D006` + `3B6A-D16C` of "Skip the opening and start
outside Baron". MuteBell put mute on all enemies.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only Hermes and MuteBell were tested. The other items were not.
- The item shows as `Dummy` in the menus and in battle.
- The English code names are from the Free Enterprise item list. The
  status numbers in the table were not checked, but the rows are the
  same as in the Japanese ROMs.
- These items can't be used from the field menu, which only allows
  items `$CE`-`$EA`.
- The Sort entry is gone, so the item menu can't sort. The TrashCan's
  quantity becomes 99 too.
- Only a new game changes. Uses the same bytes as the other new-game
  item entries; use only one of them.

## Start a new game with 99 of an unused status cure

**Codes:** Soft: `296E-6DDD` + `176E-646D`; MaidenKiss: `216E-6DDD` +
`176E-646D`; TinyHammer: `256E-6DDD` + `176E-646D`; DietFood:
`266E-6DDD` + `176E-646D`; EchoHerb: `2B6E-6DDD` + `176E-646D`; EyeDrop:
`2C6E-6DDD` + `176E-646D`; Antidote: `286E-6DDD` + `176E-646D`; Cross:
`2A6E-6DDD` + `176E-646D` (use one pair)
**Raw form:** `0082F0:<item>` (orig `$FE`) + `0082FA:63` (orig `$01`).
The items are Soft `$D5`, MaidenKiss `$D6`, TinyHammer `$D7`, DietFood
`$D8`, EchoHerb `$D9`, EyeDrop `$DA`, Antidote `$DB` and Cross `$DC`.
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:42:30

**Mechanism:** The Japanese game has one item for each bad status. The
USA release kept only Heal (`$DD`), which cures them all, and renamed
the other eight `Dummy`. Their Japanese names are きんのはり (gold
needle), おとめのキッス (maiden's kiss), うちでのこづち (magic mallet),
ダイエットフード (diet food), やまびこそう (echo herb), めぐすり (eye
drops), どくけし (antidote) and じゅうじか (cross). Their code and data
are still there. The codes put 99 of one in the new-game Sort slot, as
in "Start a new game with 99 of an unused attack item".

From the field menu, items `$D5`-`$DC` all run one handler (`menu/item.asm`).
It clears one bit of the character's status with `ItemStatusMaskTbl`:

```
@a79a:  lda     $1b3b
        sec
        sbc     #$d5
        asl
        sta     $43
        ldx     $43
        ldy     #$0003
        longa
        lda     ($60),y
        sta     $45
        and     f:ItemStatusMaskTbl,x   ; status mask for items $d5-$dd
        sta     ($60),y
```

The masks are `$FFBF`, `$FFDF`, `$FFEF`, `$FFF7`, `$FFFB`, `$FFFD`,
`$FFFE` and `$7FFF`. They clear stone, toad, mini, pig, mute, blind,
poison and curse, in item order. In battle, each item's `ItemProp` row
(`20 00 64 0B 1B 80` to `20 00 64 0B 22 80`) runs magic effect `$0B`
(esuna) with status `$1B` to `$22`.

**Evidence:** The new-game bytes at `$00:82EF` and the name table are as
in "Start a new game with 99 of an unused attack item". The USA field
item jump table at `$01:A654` points `$D5`-`$DC` all to `$01:A772`,
which has `AD 3B 1B 38 E9 D5 0A 85 43` (`lda $1b3b`, `sec`, `sbc #$d5`,
`asl`, `sta $43`). The masks were read at `$1E:FEC3`. Every code decodes
back to its raw form with `snes_gg_encode.py`.
EchoHerb was tested in Mesen2 together with the 3 codes `B864-D0D6` +
`D164-D006` + `3B6A-D16C` of "Skip the opening and start outside
Baron": the 99 EchoHerbs were there and could be used. No one was
muted, so the cure itself was not seen.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only EchoHerb was tested, and not on a muted character. The other
  items were not tested.
- The item shows as `Dummy` in the menus and in battle.
- The English code names are from the Free Enterprise item list. The
  mask order matches the Japanese names.
- The battle status numbers `$1B`-`$22` were not checked. The rows are
  the same as in the Japanese ROMs.
- The Sort entry is gone, so the item menu can't sort. The TrashCan's
  quantity becomes 99 too.
- Only a new game changes. Uses the same bytes as the other new-game
  item entries; use only one of them.

## Start a new game with 99 of an unused field item

**Codes:** Cockatrice: `3B6E-6DDD` + `176E-646D`; Siren: `236E-6DDD` +
`176E-646D`; AgApple: `3D6E-6DDD` + `176E-646D`; SomaDrop: `3F6E-6DDD` +
`176E-646D`; Book: `306E-6DDD` + `176E-646D`; Exit: `396E-6DDD` +
`176E-646D`; Bread: `316E-6DDD` + `176E-646D` (use one pair)
**Raw form:** `0082F0:<item>` (orig `$FE`) + `0082FA:63` (orig `$01`).
The items are Cockatrice `$E9`, Siren `$DE`, AgApple `$E0`, SomaDrop
`$E1`, Book `$E4`, Exit `$E5` and Bread `$E6`.
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:42:35

Tags: `demo-ready`

**Mechanism:** These are the Japanese items used from the field menu.
Their Japanese names are コカトリス (Cockatrice, with the summon icon),
アラーム (alarm), ぎんのリンゴ (silver apple), ソーマのしずく (soma
drop), エッチなほん (naughty book), ひじょうぐち (emergency exit) and
こびとのパン (dwarf bread).
The USA release renamed them `Dummy`, but the menu's item jump table
(`ItemEffect_tbl`, `menu/item.asm`) still points each one at its
handler. AuApple has its own entry, "Start a new game with 99 AuApple". SomaDrop
also has its own entry, "Start a new game with 99 SomaDrop".
The field menu allows items `$CE`-`$EA`. The codes put 99 of
one in the new-game Sort slot, as in "Start a new game with 99 of an
unused attack item".

- Cockatrice (`ItemEffect_e9`): like the Imp, Bomb and Mage summon
  items. It teaches Rydia spell `$E9 - $B6 = $33`, the unused Cockatrice
  summon.
- Siren (`ItemEffect_de`): plays a sound and returns menu event 6. The
  field sets `$C0`. On the next step where random battles can happen, a
  battle starts at once with the rarest monster group of the area
  (`ChooseRandBattle` uses `$FF` in place of the random number). On a
  map with no random battles, `CheckBattle` plays a sound and clears
  `$C0`.
- AgApple (`ItemEffect_e0`): adds 50 to a character's max HP.
- SomaDrop (`ItemEffect_e1`): adds 10 to a character's max MP (up to
  999).
- Book (`ItemEffect_e4`, the "naughty book"): only on a town or
  dungeon map. It runs field event `$DB`. The event fades out the
  music (`d8`), waits (`e9`), plays song `$22` (`ea 22`), and shows
  message `$83` of the bank 1 high dialogue table (`f1 83`). In
  Japanese the message is 「ドキドキしてきた‥‥！」 ("my heart's
  pounding...!"). In the USA ROM it is only `D` (`45 00`).
- Exit (`ItemEffect_e5`, "emergency exit"): only where the Exit spell
  works. It returns menu event 4, the same as the Exit spell
  (`MagicEffect_16`).
- Bread (`ItemEffect_e6`, "dwarven bread" in the source): only where
  the Sight spell works. It returns menu event 5, the same as the Sight
  spell (`MagicEffect_17`).

**Evidence:** The new-game bytes at `$00:82EF` and the name table are as
in "Start a new game with 99 of an unused attack item". The USA item
jump table is at `$01:A654`. It points `$DE` to `$01:A866` (`A9 3C 20 62
87 A9 06`), `$E0` to `$01:A74B`
(`... 18 69 32 00`), `$E1` to `$01:A758` (`A0 0D 00 ... 69 0A 00`), `$E4`
to `$01:A835` (`AD 04 1A 29 80 F0 EB A9 07`), `$E5` to `$01:A840` (`29
20`), `$E6` to `$01:A849` (`29 40`) and `$E9` to `$01:A86F`. The field
code has `A9 DB 20 63 DF` (`lda #$db`, `jsr ExecEvent`) at `$00:8285` for
menu event 7. The pointer for message `$183` (at `$10:8306`) is `$647E`
in the Japanese ROM and `$634E` in the USA ROM, so the text is at
`$10:E87E` and `$10:E74E`. The Exit spell's check at `$01:B993` has `AD
04 1A 29 20 F0 E8 A9 04`. Every code decodes back to its raw form with `snes_gg_encode.py`.
Exit was tested in Mesen2 together with the 3 codes
`B864-D0D6` + `D164-D006` + `3B6A-D16C` of "Skip the opening and start
outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only Exit was tested. The other items were not.
- The item shows as `Dummy` in the menu.
- The English code names are not in any ROM.
- The Book's message says only `D`. Event `$DB` itself is the same as
  in the Japanese 1.0 ROM (`C4 D8 E9 10 C7 E9 04 C5 E9 02 C7 E9 02 C4 EA
  22 F9 57 E9 10 F1 83 E9 10 D8 F9 00 FF`). What the `C4`-`C7` and
  `F9` commands do was not checked.
- An item used where it can't work may still be used up. This was not
  checked.
- These items can't be used in battle. The battle code skips items
  `$DE` and up.
- The Sort entry is gone, so the item menu can't sort. The TrashCan's
  quantity becomes 99 too.
- Only a new game changes. Uses the same bytes as the other new-game
  item entries; use only one of them.

## Everyone's turns come up with no delay

**Codes:** `10B7-676E` + `10B0-6DAE` (use both)
**Raw form:** `039E3E:64` (orig `$85`) + `039E43:64` (orig `$85`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-25 20:34:21

**Mechanism:** `CalcTimerDur` (`battle/timer_dur.asm`) sets the length
of a battle timer. It first copies the character's or monster's base
timer duration (`$2060`, 16 bits) to `$a9`:

```
        lda     $2060,x     ; base timer duration
        sta     $a9
        lda     $2061,x
        sta     $aa
```

`TimerDur_00` (the wait until the next turn) is `base * speed / 16`,
at least 1. `TimerDur_0c` (the wait for slow commands) is a part or a
multiple of the base. The codes change both `sta` (`$85`) to `stz`
(`$64`), so the base is always 0. Now every character and monster gets
a new turn 1 tick after the last one, and slow commands go off at
once. The other timer types (spells, sleep, poison, stop, ...) put
their own value in `$a9`, so they don't change.

**Evidence:** `ff2_original.smc` has `20 89 84 A6 A6 BD 60 20 85 A9 BD
61 20 85 AA BD 3B 20` at `$03:9E36` (PC `$019E36`). `BD 60 20 85 A9 BD
61 20 85 AA` appears only once in the ROM. The codes decode back to
their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- This is for everyone. Monsters also attack after every tick, so
  battles are much harder. There is no party-only version: the party
  and the monsters use the same timer code.
- Turns still wait 1 tick, and the battle speed setting sets the
  length of a tick. Spell waits don't change. Use "Spells and slow
  commands go off with no wait" for those.
- Speed, Haste and Slow no longer matter, so "Cecil's turns come up 16
  times as fast" does nothing with this code.
- FuSoYa's Regen ends at once (its timer is a multiple of the base),
  so it gives no HP.

## Skip the opening and start outside Baron

**Codes:** `B864-D0D6` + `D164-D006` + `3D6A-D16C` + `336A-D1AC` (use all 4)
**Raw form:** `128020:9B` (orig `$E0`) + `128021:06` (orig `$01`) +
`1288CA:E0` (orig `$FD`) + `1288CB:EE` (orig `$09`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, story switches:** `BEED-69D6` + `2EED-6966` + `8EED-61A6` + `53ED-6506` + `D8E0-60D6` + `D0E0-69D6`
**Raw form of the story switch codes:** `12F204:9F` (orig `$BF`) + `12F206:DF` (orig `$FF`) +
`12F20B:BF` (orig `$FF`) + `12F20D:7E` (orig `$7C`) + `12F240:0B` (orig `$00`) +
`12F244:04` (orig `$00`)
**Recorded:** 2026-10-27 21:08:47

Tags: `demo-ready` `curated` `utility` `play-start`

**Mechanism:** A new game ends `SoftReset` (`field/field.asm`) by running
the intro, event `$10`, and then entering the main loop:

```
        lda     #$10                    ; intro
        jsr     ExecEvent
        stz     $b1                     ; no event
```

`ExecEvent` (`field/event.asm`) looks up the script of event `$10` in
`EventScriptPtrs` (`$12:8000`, 2 bytes per event, relative to the first
script at `$12:8200`). Event `$10` is the Red Wings opening and the Baron
castle scenes, 864 bytes long. It ends in the throne room with Cecil
holding the Package (`e0 ee`, item `$EE`).

Event `$15`, the last Baron castle scene, ends with these commands:

```
        f2 04           set event switch $04
        e7 02           add character: Kain
        f5 13 ... f5 1c clear 20 npc switches
        f4 40           set npc switch $40
        da              fade out
        fd 09           special effect $09: prologue
        fe fb 66 9e 00  load map: overworld at (102, 158)
        c9 fa 0d da ff  walk, play song $0D, fade, end
```

The first two codes change the pointer of event `$10` from `$01E0` to
`$069B`, which is the `f2 04` above. The new game then runs only this end
of event `$15`: Kain joins Cecil, and the party appears on the overworld
at (102, 158), one tile south of the Baron castle gate (the world triggers
at (101-102, 156-157) load map `$24`, the castle). The third and fourth
codes change `fd 09` (special effect: prologue) to `e0 ee` (give item
`$EE`, the Package). That skips the prologue text scroll and gives the
Package, which the intro normally gives and the Mist plot needs. Event
command `$E0` (`EventCmd_e0`, `field/event.asm`) calls `GiveItem`. The
Sort entry stays, so the new-game item entries can be used with this one.

Before 2026-10-31 this entry used `3B6A-D16C` (`1288CA:E9`, `fd` to `e9`,
pause 9) and `336E-6DDD` (`0082F0:EE`, `lda #$fe` in `InitNewGame` to
`lda #$ee`), which put the Package in the Sort slot.

The `fd 09` bytes are also used by the "New game: <name> is in the party"
entries, which change them to `e7 NN` to add a character. Here they are
changed to `e0 ee` to give the Package.

The story switch codes change the starting tables `InitNPCSwitch` (`$12:F200`)
and `InitEventSwitch` (`$12:F240`) that `InitNewGame` copies to RAM. They
set the event switches `$00`, `$01`, `$03` and `$22` and the npc switch
`$69`, and clear the npc switches `$5E`, `$35` and `$25`. These are the
switch changes made by events `$10` to `$14`, which this code skips
(event `$15` makes the others itself).

**Evidence:** `ff2_original.smc` has `F2 04 E7 02 F5 13 F5 14` once, at
`$12:889B` (script offset `$069B`). `DA FD 09 FE FB 66 9E 00` appears once,
at `$12:88C9`. Item `$EE` is "Package" in the item name table at
`$0F:8000`, and `E0 EE` (give item) is in event `$10`. The world triggers for
the overworld (`$19:FE60`) send (101, 156), (102, 156), (101, 157) and
(102, 157) to map `$24`, the same map that event `$15` loads before it
leaves. The codes decode back to their raw forms with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The 4 core codes were tested in Mesen2 together with "Start a new
  game with 99 SomaDrop": the party starts on the overworld outside Baron
  with Cecil and Kain, and the item list has the Package and 99
  SomaDrop. The old codes (`3B6A-D16C` + `336E-6DDD`) were tested too.
  The optional codes were not tested.
- Switch changes made by trigger scripts in Baron castle, and the order
  of events `$11` to `$14`, were not checked. Without the optional codes
  the castle may still act as if the opening was not played if you go
  back inside.
- Uses the same bytes as the "New game: <name> is in the party"
  entries. Those entries say how to use them with the first 2 codes.
- The song, the Red Wings flight and the castle scenes do not play. The
  title screen still does; use "Skip the title screen" for that.

## New game: Rydia is in the party

**Codes:** `356A-D16C` + `D76A-D1AC` (use both)
**Raw form:** `1288CA:E7` (orig `$FD`) + `1288CB:03` (orig `$09`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-27 21:42:32

Tags: `demo-ready`

**Mechanism:** This is a split-out version of the Rydia option of "Skip the
opening and start outside Baron", for a normal game
with the full intro. Event `$15` (`EventScriptPtrs` at `$12:8000`, script
offset `$05C1`-`$06D5`) is the last Baron castle scene, the one where Kain
joins.
It ends with these commands:

```
        f2 04           set event switch $04
        e7 02           add character: Kain
        f5 13 ... f5 1c clear 20 npc switches
        f4 40           set npc switch $40
        da              fade out
        fd 09           special effect $09: prologue
        fe fb 66 9e 00  load map: overworld at (102, 158)
        c9 fa 0d da ff  walk, play song $0D, fade, end
```

The codes change `fd 09` to `e7 03`, so the event adds one more
character right after Kain. Event command `$E7` (`field/event.asm`)
looks up `CharRemoveTbl[3 - 1]`, which is `CharProp` entry `$02`
(`$0F:A900`, 32 bytes each), and loads it into the next free party slot
with `LoadCharProp`. That entry is Rydia (the name comes from
`CharNameTbl` in `menu/menu.asm`), level 1, as the child. Cecil is in
slot 0 and Kain in slot 1, so Rydia takes slot 2 (`$1080`). The prologue
text scroll that `fd 09` started no longer plays.

**Evidence:** `ff2_original.smc` has `DA FD 09 FE FB 66 9E 00` once, at
`$12:88C9`. `CharProp` entry `$02` at `$0F:A940` starts `83 02 01`: id
`$03`, graphic `$02`, level 1. The codes decode back to their raw forms
with `snes_gg_encode.py`. Tested in Mesen2 with only these two codes on
and the full intro playing: Rydia is in the party.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Rydia joins at level 1 with the `CharProp` starting equipment.
- The prologue text scroll is skipped.
- When the story adds Rydia later, event command `$E7` doesn't check for
  a duplicate, so there may be two copies of Rydia. What happens then was not
  checked.
- Uses the same bytes as the third and fourth codes (`3D6A-D16C` +
  `336A-D1AC`) of "Skip the opening and start outside Baron". To use both,
  take these codes, the first 2 codes of that entry (`B864-D0D6` +
  `D164-D006`) and `336E-6DDD` (`0082F0:EE`, the Package in the Sort slot).
  This is what that entry's old Rydia option was.

## New game: Tellah is in the party

**Codes:** `356A-D16C` + `D06A-D1AC` (use both)
**Raw form:** `1288CA:E7` (orig `$FD`) + `1288CB:04` (orig `$09`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-27 21:44:48

Tags: `demo-ready`

**Mechanism:** This is the same idea as "New game: Rydia is in the party", for
Tellah. Event `$15` (`EventScriptPtrs` at `$12:8000`, script offset
`$05C1`-`$06D5`) is the last Baron castle scene, the one where Kain joins. It
ends with `e7 02` (add character: Kain), a run of npc switch changes, then
`da` (fade out), `fd 09` (special effect `$09`: the prologue text) and
`fe fb 66 9e 00` (load the overworld at (102, 158)). The code changes `fd`
to `e7` and `09` to `04`. The event then adds Tellah right after Kain. Event command `$E7`
(`field/event.asm`) looks up `CharRemoveTbl[4 - 1]`, which is `CharProp`
entry `$03` (`$0F:A900`, 32 bytes each), and loads it into the next free
party slot with `LoadCharProp`. Cecil is in slot 0 and Kain in slot 1, so
Tellah takes slot 2 (`$1080`). The prologue text scroll no longer plays.

**Evidence:** `ff2_original.smc` has `DA FD 09 FE FB 66 9E 00` once, at
`$12:88C9`. `CharProp` entry `$03` at `$0F:A960` starts `84 03 14`: id
`$04`, graphic `$03`, level 20, 21,505 HP and 23040 MP. `CharNameTbl` maps id `$04` to the
name "Tellah". The `CharInitEquip` row (`$0F:AB00`, 7 bytes each) for this
character is `00 8E 00 0F 01 00 00`. The codes decode back to their raw forms with
`snes_gg_encode.py`. Tested in Mesen2 together with
the 3 codes `B864-D0D6` + `D164-D006` + `336E-6DDD` of "Skip the opening and
start outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Tellah joins at level 20 with the `CharProp` starting equipment. The
  starting spells come from `SpellListInit`, which `InitSpellLists` fills
  before the character joins; whether Tellah has the spells from the story
  was not checked.
- The prologue text scroll is skipped.
- When the story adds Tellah later, event command `$E7` doesn't check for a
  duplicate, so there may be two copies of Tellah. What happens then was not
  checked.
- Uses the same bytes as the third and fourth codes (`3D6A-D16C` +
  `336A-D1AC`) of "Skip the opening and start outside Baron". To use both,
  take these codes, the first 2 codes of that entry (`B864-D0D6` +
  `D164-D006`) and `336E-6DDD` (`0082F0:EE`, the Package in the Sort slot).
  Only one "New game: ... is in the party" entry can be used at a time.

## New game: Edward is in the party

**Codes:** `356A-D16C` + `D96A-D1AC` (use both)
**Raw form:** `1288CA:E7` (orig `$FD`) + `1288CB:05` (orig `$09`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-27 21:44:49

Tags: `demo-ready`

**Mechanism:** This is the same idea as "New game: Rydia is in the party", for
Edward. Event `$15` (`EventScriptPtrs` at `$12:8000`, script offset
`$05C1`-`$06D5`) is the last Baron castle scene, the one where Kain joins. It
ends with `e7 02` (add character: Kain), a run of npc switch changes, then
`da` (fade out), `fd 09` (special effect `$09`: the prologue text) and
`fe fb 66 9e 00` (load the overworld at (102, 158)). The code changes `fd`
to `e7` and `09` to `05`. The event then adds Edward right after Kain. Event command `$E7`
(`field/event.asm`) looks up `CharRemoveTbl[5 - 1]`, which is `CharProp`
entry `$04` (`$0F:A900`, 32 bytes each), and loads it into the next free
party slot with `LoadCharProp`. Cecil is in slot 0 and Kain in slot 1, so
Edward takes slot 2 (`$1080`). The prologue text scroll no longer plays.

**Evidence:** `ff2_original.smc` has `DA FD 09 FE FB 66 9E 00` once, at
`$12:88C9`. `CharProp` entry `$04` at `$0F:A980` starts `85 04 05`: id
`$05`, graphic `$04`, level 5, 15,360 HP and 0 MP. `CharNameTbl` maps id `$05` to the
name "Edward". The `CharInitEquip` row (`$0F:AB00`, 7 bytes each) for this
character is `78 97 A8 44 01 00 00`. The codes decode back to their raw forms with
`snes_gg_encode.py`. Tested in Mesen2 together with
the 3 codes `B864-D0D6` + `D164-D006` + `336E-6DDD` of "Skip the opening and
start outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Edward joins at level 5 with the `CharProp` starting equipment. The
  starting spells come from `SpellListInit`, which `InitSpellLists` fills
  before the character joins; whether Edward has the spells from the story
  was not checked.
- The prologue text scroll is skipped.
- When the story adds Edward later, event command `$E7` doesn't check for a
  duplicate, so there may be two copies of Edward. What happens then was not
  checked.
- Uses the same bytes as the third and fourth codes (`3D6A-D16C` +
  `336A-D1AC`) of "Skip the opening and start outside Baron". To use both,
  take these codes, the first 2 codes of that entry (`B864-D0D6` +
  `D164-D006`) and `336E-6DDD` (`0082F0:EE`, the Package in the Sort slot).
  Only one "New game: ... is in the party" entry can be used at a time.

## New game: Rosa is in the party

**Codes:** `356A-D16C` + `D16A-D1AC` (use both)
**Raw form:** `1288CA:E7` (orig `$FD`) + `1288CB:06` (orig `$09`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-27 21:44:50

Tags: `demo-ready`

**Mechanism:** This is the same idea as "New game: Rydia is in the party", for
Rosa. Event `$15` (`EventScriptPtrs` at `$12:8000`, script offset
`$05C1`-`$06D5`) is the last Baron castle scene, the one where Kain joins. It
ends with `e7 02` (add character: Kain), a run of npc switch changes, then
`da` (fade out), `fd 09` (special effect `$09`: the prologue text) and
`fe fb 66 9e 00` (load the overworld at (102, 158)). The code changes `fd`
to `e7` and `09` to `06`. The event then adds Rosa right after Kain. Event command `$E7`
(`field/event.asm`) looks up `CharRemoveTbl[6 - 1]`, which is `CharProp`
entry `$05` (`$0F:A900`, 32 bytes each), and loads it into the next free
party slot with `LoadCharProp`. Cecil is in slot 0 and Kain in slot 1, so
Rosa takes slot 2 (`$1080`). The prologue text scroll no longer plays.

**Evidence:** `ff2_original.smc` has `DA FD 09 FE FB 66 9E 00` once, at
`$12:88C9`. `CharProp` entry `$05` at `$0F:A9A0` starts `86 05 0A`: id
`$06`, graphic `$05`, level 10, 38,400 HP and 20480 MP. `CharNameTbl` maps id `$06` to the
name "Rosa". The `CharInitEquip` row (`$0F:AB00`, 7 bytes each) for this
character is `77 8E A7 54 32 4D 01`. The codes decode back to their raw forms with
`snes_gg_encode.py`. Tested in Mesen2 together with
the 3 codes `B864-D0D6` + `D164-D006` + `336E-6DDD` of "Skip the opening and
start outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Rosa joins at level 10 with the `CharProp` starting equipment. The
  starting spells come from `SpellListInit`, which `InitSpellLists` fills
  before the character joins; whether Rosa has the spells from the story
  was not checked.
- The prologue text scroll is skipped.
- When the story adds Rosa later, event command `$E7` doesn't check for a
  duplicate, so there may be two copies of Rosa. What happens then was not
  checked.
- Uses the same bytes as the third and fourth codes (`3D6A-D16C` +
  `336A-D1AC`) of "Skip the opening and start outside Baron". To use both,
  take these codes, the first 2 codes of that entry (`B864-D0D6` +
  `D164-D006`) and `336E-6DDD` (`0082F0:EE`, the Package in the Sort slot).
  Only one "New game: ... is in the party" entry can be used at a time.

## New game: Yang is in the party

**Codes:** `356A-D16C` + `D56A-D1AC` (use both)
**Raw form:** `1288CA:E7` (orig `$FD`) + `1288CB:07` (orig `$09`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-27 21:44:51

Tags: `demo-ready`

**Mechanism:** This is the same idea as "New game: Rydia is in the party", for
Yang. Event `$15` (`EventScriptPtrs` at `$12:8000`, script offset
`$05C1`-`$06D5`) is the last Baron castle scene, the one where Kain joins. It
ends with `e7 02` (add character: Kain), a run of npc switch changes, then
`da` (fade out), `fd 09` (special effect `$09`: the prologue text) and
`fe fb 66 9e 00` (load the overworld at (102, 158)). The code changes `fd`
to `e7` and `09` to `07`. The event then adds Yang right after Kain. Event command `$E7`
(`field/event.asm`) looks up `CharRemoveTbl[7 - 1]`, which is `CharProp`
entry `$06` (`$0F:A900`, 32 bytes each), and loads it into the next free
party slot with `LoadCharProp`. Cecil is in slot 0 and Kain in slot 1, so
Yang takes slot 2 (`$1080`). The prologue text scroll no longer plays.

**Evidence:** `ff2_original.smc` has `DA FD 09 FE FB 66 9E 00` once, at
`$12:88C9`. `CharProp` entry `$06` at `$0F:A9C0` starts `C7 06 0A`: id
`$07`, graphic `$06`, level 10, 11,265 HP and 0 MP. `CharNameTbl` maps id `$07` to the
name "Yang". The `CharInitEquip` row (`$0F:AB00`, 7 bytes each) for this
character is `00 8E 00 01 01 00 00`. The codes decode back to their raw forms with
`snes_gg_encode.py`. Tested in Mesen2 together with
the 3 codes `B864-D0D6` + `D164-D006` + `336E-6DDD` of "Skip the opening and
start outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Yang joins at level 10 with the `CharProp` starting equipment. The
  starting spells come from `SpellListInit`, which `InitSpellLists` fills
  before the character joins; whether Yang has the spells from the story
  was not checked.
- The prologue text scroll is skipped.
- When the story adds Yang later, event command `$E7` doesn't check for a
  duplicate, so there may be two copies of Yang. What happens then was not
  checked.
- Uses the same bytes as the third and fourth codes (`3D6A-D16C` +
  `336A-D1AC`) of "Skip the opening and start outside Baron". To use both,
  take these codes, the first 2 codes of that entry (`B864-D0D6` +
  `D164-D006`) and `336E-6DDD` (`0082F0:EE`, the Package in the Sort slot).
  Only one "New game: ... is in the party" entry can be used at a time.

## New game: Palom is in the party

**Codes:** `356A-D16C` + `D66A-D1AC` (use both)
**Raw form:** `1288CA:E7` (orig `$FD`) + `1288CB:08` (orig `$09`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-27 21:44:52

Tags: `demo-ready`

**Mechanism:** This is the same idea as "New game: Rydia is in the party", for
Palom. Event `$15` (`EventScriptPtrs` at `$12:8000`, script offset
`$05C1`-`$06D5`) is the last Baron castle scene, the one where Kain joins. It
ends with `e7 02` (add character: Kain), a run of npc switch changes, then
`da` (fade out), `fd 09` (special effect `$09`: the prologue text) and
`fe fb 66 9e 00` (load the overworld at (102, 158)). The code changes `fd`
to `e7` and `09` to `08`. The event then adds Palom right after Kain. Event command `$E7`
(`field/event.asm`) looks up `CharRemoveTbl[8 - 1]`, which is `CharProp`
entry `$07` (`$0F:A900`, 32 bytes each), and loads it into the next free
party slot with `LoadCharProp`. Cecil is in slot 0 and Kain in slot 1, so
Palom takes slot 2 (`$1080`). The prologue text scroll no longer plays.

**Evidence:** `ff2_original.smc` has `DA FD 09 FE FB 66 9E 00` once, at
`$12:88C9`. `CharProp` entry `$07` at `$0F:A9E0` starts `48 07 0A`: id
`$08`, graphic `$07`, level 10, 25,600 HP and 12800 MP. `CharNameTbl` maps id `$08` to the
name "Palom". The `CharInitEquip` row (`$0F:AB00`, 7 bytes each) for this
character is `78 8E A7 00 00 07 01`. The codes decode back to their raw forms with
`snes_gg_encode.py`. Tested in Mesen2 together with
the 3 codes `B864-D0D6` + `D164-D006` + `336E-6DDD` of "Skip the opening and
start outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Palom joins at level 10 with the `CharProp` starting equipment. The
  starting spells come from `SpellListInit`, which `InitSpellLists` fills
  before the character joins; whether Palom has the spells from the story
  was not checked.
- The prologue text scroll is skipped.
- When the story adds Palom later, event command `$E7` doesn't check for a
  duplicate, so there may be two copies of Palom. What happens then was not
  checked.
- Uses the same bytes as the third and fourth codes (`3D6A-D16C` +
  `336A-D1AC`) of "Skip the opening and start outside Baron". To use both,
  take these codes, the first 2 codes of that entry (`B864-D0D6` +
  `D164-D006`) and `336E-6DDD` (`0082F0:EE`, the Package in the Sort slot).
  Only one "New game: ... is in the party" entry can be used at a time.

## New game: Porom is in the party

**Code:** `356A-D16C` (use this one code)
**Raw form:** `1288CA:E7` (orig `$FD`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-27 21:44:53

Tags: `demo-ready`

**Mechanism:** This is the same idea as "New game: Rydia is in the party", for
Porom. Event `$15` (`EventScriptPtrs` at `$12:8000`, script offset
`$05C1`-`$06D5`) is the last Baron castle scene, the one where Kain joins. It
ends with `e7 02` (add character: Kain), a run of npc switch changes, then
`da` (fade out), `fd 09` (special effect `$09`: the prologue text) and
`fe fb 66 9e 00` (load the overworld at (102, 158)). The code changes `fd`
to `e7`. The second byte of `fd 09` is already `$09`, which is the number for
Porom, so only the first byte changes. The event then adds Porom right after
Kain. Event command `$E7`
(`field/event.asm`) looks up `CharRemoveTbl[9 - 1]`, which is `CharProp`
entry `$08` (`$0F:A900`, 32 bytes each), and loads it into the next free
party slot with `LoadCharProp`. Cecil is in slot 0 and Kain in slot 1, so
Porom takes slot 2 (`$1080`). The prologue text scroll no longer plays.

**Evidence:** `ff2_original.smc` has `DA FD 09 FE FB 66 9E 00` once, at
`$12:88C9`. `CharProp` entry `$08` at `$0F:AA00` starts `89 08 0A`: id
`$09`, graphic `$08`, level 10, 28,160 HP and 12800 MP. `CharNameTbl` maps id `$09` to the
name "Porom". The `CharInitEquip` row (`$0F:AB00`, 7 bytes each) for this
character is `78 8E A7 0F 01 00 00`. The codes decode back to their raw forms with
`snes_gg_encode.py`. Tested in Mesen2 together with
the 3 codes `B864-D0D6` + `D164-D006` + `336E-6DDD` of "Skip the opening and
start outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Porom joins at level 10 with the `CharProp` starting equipment. The
  starting spells come from `SpellListInit`, which `InitSpellLists` fills
  before the character joins; whether Porom has the spells from the story
  was not checked.
- The prologue text scroll is skipped.
- When the story adds Porom later, event command `$E7` doesn't check for a
  duplicate, so there may be two copies of Porom. What happens then was not
  checked.
- Uses the same bytes as the third and fourth codes (`3D6A-D16C` +
  `336A-D1AC`) of "Skip the opening and start outside Baron". To use both,
  take these codes, the first 2 codes of that entry (`B864-D0D6` +
  `D164-D006`) and `336E-6DDD` (`0082F0:EE`, the Package in the Sort slot).
  Only one "New game: ... is in the party" entry can be used at a time.

## New game: Cid is in the party

**Codes:** `356A-D16C` + `D36A-D1AC` (use both)
**Raw form:** `1288CA:E7` (orig `$FD`) + `1288CB:0E` (orig `$09`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-27 21:44:54

Tags: `demo-ready`

**Mechanism:** This is the same idea as "New game: Rydia is in the party", for
Cid. Event `$15` (`EventScriptPtrs` at `$12:8000`, script offset
`$05C1`-`$06D5`) is the last Baron castle scene, the one where Kain joins. It
ends with `e7 02` (add character: Kain), a run of npc switch changes, then
`da` (fade out), `fd 09` (special effect `$09`: the prologue text) and
`fe fb 66 9e 00` (load the overworld at (102, 158)). The code changes `fd`
to `e7` and `09` to `0E`. The event then adds Cid right after Kain. Event command `$E7`
(`field/event.asm`) looks up `CharRemoveTbl[14 - 1]`, which is `CharProp`
entry `$0A` (`$0F:A900`, 32 bytes each), and loads it into the next free
party slot with `LoadCharProp`. Cecil is in slot 0 and Kain in slot 1, so
Cid takes slot 2 (`$1080`). The prologue text scroll no longer plays.

**Evidence:** `ff2_original.smc` has `DA FD 09 FE FB 66 9E 00` once, at
`$12:88C9`. `CharProp` entry `$0A` at `$0F:AA40` starts `8E 0A 14`: id
`$0E`, graphic `$0A`, level 20, 5,123 HP and 0 MP. `CharNameTbl` maps id `$0E` to the
name "Cid". The `CharInitEquip` row (`$0F:AB00`, 7 bytes each) for this
character is `00 96 00 4B 01 00 00`. The codes decode back to their raw forms with
`snes_gg_encode.py`. Tested in Mesen2 together with
the 3 codes `B864-D0D6` + `D164-D006` + `336E-6DDD` of "Skip the opening and
start outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Cid joins at level 20 with the `CharProp` starting equipment. The
  starting spells come from `SpellListInit`, which `InitSpellLists` fills
  before the character joins; whether Cid has the spells from the story
  was not checked.
- The prologue text scroll is skipped.
- When the story adds Cid later, event command `$E7` doesn't check for a
  duplicate, so there may be two copies of Cid. What happens then was not
  checked.
- Uses the same bytes as the third and fourth codes (`3D6A-D16C` +
  `336A-D1AC`) of "Skip the opening and start outside Baron". To use both,
  take these codes, the first 2 codes of that entry (`B864-D0D6` +
  `D164-D006`) and `336E-6DDD` (`0082F0:EE`, the Package in the Sort slot).
  Only one "New game: ... is in the party" entry can be used at a time.

## New game: Edge is in the party

**Codes:** `356A-D16C` + `F46A-D1AC` (use both)
**Raw form:** `1288CA:E7` (orig `$FD`) + `1288CB:12` (orig `$09`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-27 21:44:55

Tags: `demo-ready`

**Mechanism:** This is the same idea as "New game: Rydia is in the party", for
Edge. Event `$15` (`EventScriptPtrs` at `$12:8000`, script offset
`$05C1`-`$06D5`) is the last Baron castle scene, the one where Kain joins. It
ends with `e7 02` (add character: Kain), a run of npc switch changes, then
`da` (fade out), `fd 09` (special effect `$09`: the prologue text) and
`fe fb 66 9e 00` (load the overworld at (102, 158)). The code changes `fd`
to `e7` and `09` to `12`. The event then adds Edge right after Kain. Event command `$E7`
(`field/event.asm`) looks up `CharRemoveTbl[18 - 1]`, which is `CharProp`
entry `$0B` (`$0F:A900`, 32 bytes each), and loads it into the next free
party slot with `LoadCharProp`. Cecil is in slot 0 and Kain in slot 1, so
Edge takes slot 2 (`$1080`). The prologue text scroll no longer plays.

**Evidence:** `ff2_original.smc` has `DA FD 09 FE FB 66 9E 00` once, at
`$12:88C9`. `CharProp` entry `$0B` at `$0F:AA60` starts `D2 0C 19`: id
`$12`, graphic `$0C`, level 25, 5,635 HP and 15360 MP. `CharNameTbl` maps id `$12` to the
name "Edge". The `CharInitEquip` row (`$0F:AB00`, 7 bytes each) for this
character is `7E 99 A1 2B 01 2B 01`. The codes decode back to their raw forms with
`snes_gg_encode.py`. Tested in Mesen2 together with
the 3 codes `B864-D0D6` + `D164-D006` + `336E-6DDD` of "Skip the opening and
start outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Edge joins at level 25 with the `CharProp` starting equipment. The
  starting spells come from `SpellListInit`, which `InitSpellLists` fills
  before the character joins; whether Edge has the spells from the story
  was not checked.
- The prologue text scroll is skipped.
- When the story adds Edge later, event command `$E7` doesn't check for a
  duplicate, so there may be two copies of Edge. What happens then was not
  checked.
- Uses the same bytes as the third and fourth codes (`3D6A-D16C` +
  `336A-D1AC`) of "Skip the opening and start outside Baron". To use both,
  take these codes, the first 2 codes of that entry (`B864-D0D6` +
  `D164-D006`) and `336E-6DDD` (`0082F0:EE`, the Package in the Sort slot).
  Only one "New game: ... is in the party" entry can be used at a time.

## New game: FuSoYa is in the party

**Codes:** `356A-D16C` + `F76A-D1AC` (use both)
**Raw form:** `1288CA:E7` (orig `$FD`) + `1288CB:13` (orig `$09`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-27 21:44:56

Tags: `demo-ready`

**Mechanism:** This is the same idea as "New game: Rydia is in the party", for
FuSoYa. Event `$15` (`EventScriptPtrs` at `$12:8000`, script offset
`$05C1`-`$06D5`) is the last Baron castle scene, the one where Kain joins. It
ends with `e7 02` (add character: Kain), a run of npc switch changes, then
`da` (fade out), `fd 09` (special effect `$09`: the prologue text) and
`fe fb 66 9e 00` (load the overworld at (102, 158)). The code changes `fd`
to `e7` and `09` to `13`. The event then adds FuSoYa right after Kain. Event command `$E7`
(`field/event.asm`) looks up `CharRemoveTbl[19 - 1]`, which is `CharProp`
entry `$0C` (`$0F:A900`, 32 bytes each), and loads it into the next free
party slot with `LoadCharProp`. Cecil is in slot 0 and Kain in slot 1, so
FuSoYa takes slot 2 (`$1080`). The prologue text scroll no longer plays.

**Evidence:** `ff2_original.smc` has `DA FD 09 FE FB 66 9E 00` once, at
`$12:88C9`. `CharProp` entry `$0C` at `$0F:AA80` starts `93 0D 32`: id
`$13`, graphic `$0D`, level 50, 27,655 HP and 48640 MP. `CharNameTbl` maps id `$13` to the
name "FuSoYa". The `CharInitEquip` row (`$0F:AB00`, 7 bytes each) for this
character is `7A 92 AD 13 01 00 00`. The codes decode back to their raw forms with
`snes_gg_encode.py`. Tested in Mesen2 together with the 3 codes
`B864-D0D6` + `D164-D006` + `336E-6DDD` of "Skip the opening and start
outside Baron": FuSoYa is in the party with Cecil and Kain.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only whether FuSoYa joins was tested, and only together with the skip
  cheat. The spells and equipment were not checked.
- FuSoYa joins at level 50 with the `CharProp` starting equipment. The
  starting spells come from `SpellListInit`, which `InitSpellLists` fills
  before the character joins; whether FuSoYa has the spells from the story
  was not checked.
- The prologue text scroll is skipped.
- When the story adds FuSoYa later, event command `$E7` doesn't check for a
  duplicate, so there may be two copies of FuSoYa. What happens then was not
  checked.
- Uses the same bytes as the third and fourth codes (`3D6A-D16C` +
  `336A-D1AC`) of "Skip the opening and start outside Baron". To use both,
  take these codes, the first 2 codes of that entry (`B864-D0D6` +
  `D164-D006`) and `336E-6DDD` (`0082F0:EE`, the Package in the Sort slot).
  Only one "New game: ... is in the party" entry can be used at a time.

## New game: Golbez is in the party

**Codes:** `356A-D16C` + `F96A-D1AC` (use both)
**Raw form:** `1288CA:E7` (orig `$FD`) + `1288CB:15` (orig `$09`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-27 21:44:57

Tags: `demo-ready`

**Mechanism:** This is the same idea as "New game: Rydia is in the party", for
Golbez. Event `$15` (`EventScriptPtrs` at `$12:8000`, script offset
`$05C1`-`$06D5`) is the last Baron castle scene, the one where Kain joins. It
ends with `e7 02` (add character: Kain), a run of npc switch changes, then
`da` (fade out), `fd 09` (special effect `$09`: the prologue text) and
`fe fb 66 9e 00` (load the overworld at (102, 158)). The code changes `fd`
to `e7` and `09` to `15`. The event then adds Golbez right after Kain. Event command `$E7`
(`field/event.asm`) looks up `CharRemoveTbl[21 - 1]`, which is `CharProp`
entry `$0D` (`$0F:A900`, 32 bytes each), and loads it into the next free
party slot with `LoadCharProp`. Cecil is in slot 0 and Kain in slot 1, so
Golbez takes slot 2 (`$1080`). The prologue text scroll no longer plays.

**Evidence:** `ff2_original.smc` has `DA FD 09 FE FB 66 9E 00` once, at
`$12:88C9`. `CharProp` entry `$0D` at `$0F:AAA0` starts `95 0F 32`: id
`$15`, graphic `$0F`, level 50, 32,523 HP and 46080 MP. `CharNameTbl` maps id `$15` to the
name "Golbez". The `CharInitEquip` row (`$0F:AB00`, 7 bytes each) for this
character is `7A 92 AD 13 01 00 FF`. The codes decode back to their raw forms with
`snes_gg_encode.py`. Tested in Mesen2 with only these two codes and
the full intro playing: Golbez is in the party.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only whether Golbez joins was tested. The spells and equipment were not
  checked.
- Golbez joins at level 50 with the `CharProp` starting equipment. The
  starting spells come from `SpellListInit`, which `InitSpellLists` fills
  before the character joins; whether Golbez has the spells from the story
  was not checked.
- The prologue text scroll is skipped.
- When the story adds Golbez later, event command `$E7` doesn't check for a
  duplicate, so there may be two copies of Golbez. What happens then was not
  checked.
- The `CharInitEquip` row ends in `FF` where the others end in `00`; what
  that does was not checked.
- Uses the same bytes as the third and fourth codes (`3D6A-D16C` +
  `336A-D1AC`) of "Skip the opening and start outside Baron". To use both,
  take these codes, the first 2 codes of that entry (`B864-D0D6` +
  `D164-D006`) and `336E-6DDD` (`0082F0:EE`, the Package in the Sort slot).
  Only one "New game: ... is in the party" entry can be used at a time.

## Sage class has the Ninja command

**Code:** `F6E1-01DE`
**Raw form:** `13FD68:18` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-27 22:51:08

**Mechanism:** At the start of each battle,
`battle/init.asm` copies 5 command ids per character from `ClassCmd`
(`$13:FD55`, 5 bytes per class, `$FF` = empty) to the battle command list
at `$3303`. Class `$04`, the class Tellah has when he joins (title "Sage"),
has Fight, White, Black, Item and one empty slot. The code puts `$18`,
Ninja, in that empty slot. When Tellah picks it, the battle menu in
`btlgfx/cursor.asm` opens his black magic list, the same as Black.

**Evidence:** In `ff2_original.smc` the `ClassCmd` row for class `$04` is at
`$13:FD64`: `00 02 03 01 FF` (Fight, White, Black, Item, empty). The code
changes its fifth byte, `$13:FD68`. Tellah's `CharProp` entry (`$0F:A960`) starts
`84 03 14`: id `$04`, title index 3, which is "Sage" in the class name
table. The code decodes back to its raw form with `snes_gg_encode.py`.
Tested in Mesen2 together with the 3 codes `B864-D0D6` + `D164-D006` +
`336E-6DDD` of "Skip the opening and start outside Baron" and the 2 codes
of "New game: Tellah is in the party": Tellah has Ninja in battle.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Tellah has no other class row with Ninja here. His later rows (classes
  `$0A` and `$0C`) are not changed.
- The Ninja list is Tellah's black magic list, so it shows the same spells
  as Black.
- Uses the same byte as "Tellah has the Recall command" and as the entries
  in `ff2_long_game_genie.md` that add a command to every class; use only one
  of them.

## Rydia's black magic spells replaced with Ninja spells

**Codes:** `DAE3-09DE` + `DAEE-05AE` (use both)
**Raw form:** `13FDE4:0C` (orig `$03`) + `13FDFF:0C` (orig `$03`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, Tellah too:** `DAE3-09AE` (`13FDE7:0C`, orig `$06`)
**Optional, Palom too:** `DAEE-00AE` (`13FDF3:0C`, orig `$08`)
**Optional, FuSoYa too:** `DAED-690E` (`13FE05:0C`, orig `$0B`)
**Recorded:** 2026-10-27 22:59:43

Tags: `demo-ready`

**Mechanism:** At the start of each battle, `battle/init.asm` builds each
character's three spell lists (white, black, summon) from `SpellListTbl`
(`$13:FDDD`, 3 bytes per character, indexed by the low nibble of the
character's `$1001`; `$FF` = no list). The rows with a black list are child
Rydia (`02 03 04`), Tellah (`05 06 FF`), Palom (`FF 08 FF`), adult Rydia
(`FF 03 04`) and FuSoYa (`0A 0B FF`). Each code changes one black list to
`$0C`, the list Edge uses for his ninja spells. The Black command (and Ninja,
if the character has it) opens the black spell list, so it now shows the
ninja spells. The two Rydia codes are for child and adult Rydia; they use the
same list, `$03`, so the Rydia codes only differ in the row they change.
Level-ups use the same table (`LevelUpMagic` in `battle/win.asm`), so each
character learns from the `$0C` schedule in `SpellListLearned` (`$0F:C700`):
ids `$46` at level 27, `$45` at level 33 and `$47` at level 38. The table is
indexed by character, not class, so each code covers every class row of its
character.

**Evidence:** `ff2_original.smc` has `FF FF FF FF FF FF 02 03 04 05 06 FF FF FF
FF 07 FF FF FF FF FF FF 08 FF 09 FF FF 00 FF FF FF FF FF FF 03 04 FF 0C FF 0A
0B FF 0A 0B FF` at `$13:FDDD`. Every code decodes back to its raw form with
`snes_gg_encode.py`. Tested in Mesen2 as part of the 5 codes `DAE3-09DE` +
`DAE3-09AE` + `DAEE-00AE` + `DAEE-05AE` + `DAED-690E`, together with the 3
codes `B864-D0D6` + `D164-D006` + `336E-6DDD` of "Skip the opening and start
outside Baron" and the 2 codes of "New game: Tellah is in the party":
Tellah's black magic menu showed only Flame. The two base codes `DAE3-09DE` +
`DAEE-05AE` were also tested alone, together with the 3 codes of the skip entry
and the 2 codes of "New game: Rydia is in the party": child Rydia's black magic
menu showed the Ninja spells.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Tellah was checked with all 5 codes on. Child Rydia was checked with the
  two base codes only. The adult Rydia, Palom and FuSoYa rows were matched to
  the characters by their position in the table and not checked in battle.
- The old black spells are gone from the menu, not just hidden.
- The ninja list is shared with Edge and with every character whose black
  list is set to `$0C`.
- Golbez's row (`0A 0B FF`) has a black list too, but Golbez has no Black
  command, so he is not covered.
- Where Flame comes from, and the full starting set, was not checked. The
  level-up schedule above was read from the ROM; it was not tested.

## New game: Paladin Cecil is in the party

**Codes:** `356A-D16C` + `D86A-D1AC` (use both)
**Raw form:** `1288CA:E7` (orig `$FD`) + `1288CB:0B` (orig `$09`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-27 23:06:39

Tags: `demo-ready`

**Mechanism:** This is the same idea as "New game: Rydia is in the party",
for Paladin Cecil. Event `$15` (`EventScriptPtrs` at `$12:8000`, script
offset `$05C1`-`$06D5`) is the last Baron castle scene, the one where Kain
joins. It ends with `e7 02` (add character: Kain), a run of npc switch
changes, then `da` (fade out), `fd 09` (special effect `$09`: the prologue
text) and `fe fb 66 9e 00` (load the overworld at (102, 158)). The codes
change `fd 09` to `e7 0b`. The event then adds Paladin Cecil right after
Kain. Event command `$E7` (`field/event.asm`) looks up
`CharRemoveTbl[11 - 1]`, which is `CharProp` entry `$09` (`$0F:A900`, 32
bytes each), and loads it into the next free party slot with
`LoadCharProp`. Cecil is in slot 0 and Kain in slot 1, so Paladin Cecil takes
slot 2 (`$1080`). The prologue text scroll no longer plays.

**Evidence:** `ff2_original.smc` has `DA FD 09 FE FB 66 9E 00` once, at
`$12:88C9`. `CharProp` entry `$09` at `$0F:AA20` starts `8B 09 01`: id
`$0B`, graphic `$09`, level 1. `CharNameTbl` maps id `$0B` to the name
"Cecil". The `CharInitEquip` row (`$0F:AB00`, 7 bytes each) for this
character is `00 8D 00 19 01 00 00`. The codes decode back to their raw
forms with `snes_gg_encode.py`. Tested in Mesen2 together with the 3 codes
`B864-D0D6` + `D164-D006` + `336E-6DDD` of "Skip the opening and start
outside Baron": Paladin Cecil is in the party with Cecil and Kain.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only whether Paladin Cecil joins was tested, and only together with the
  skip cheat. The spells and equipment were not checked.
- The party has two characters named Cecil: the Dark Knight in slot 0 and
  the Paladin in slot 2. Nothing in the story removes the Dark Knight, and
  what the game does when the story turns Cecil into a Paladin was not
  checked.
- Paladin Cecil joins at level 1 with the `CharProp` starting equipment. The
  starting spells come from `SpellListInit`, which `InitSpellLists` fills
  before the character joins; whether Paladin Cecil has the spells from the
  story was not checked.
- The prologue text scroll is skipped.
- Uses the same bytes as the third and fourth codes (`3D6A-D16C` +
  `336A-D1AC`) of "Skip the opening and start outside Baron". To use both,
  take these codes, the first 2 codes of that entry (`B864-D0D6` +
  `D164-D006`) and `336E-6DDD` (`0082F0:EE`, the Package in the Sort slot).
  Only one "New game: ... is in the party" entry can be used at a time.

## New game: adult Rydia is in the party

**Codes:** `356A-D16C` + `D76A-D1AC` + `BFC0-0D73` + `D8C0-0D53` (use all 4)
**Raw form:** `1288CA:E7` (orig `$FD`) + `1288CB:03` (orig `$09`) +
`0FA940:91` (orig `$83`) + `0FA941:0B` (orig `$02`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-27 23:06:40

Tags: `demo-ready`

**Mechanism:** Adult Rydia has no `CharProp` entry of her own. The story
makes her by restoring Rydia's saved data with event command `$E7` and then
patching it: if the class id is `$11`, the graphic is set to `$0B`
(`field/event.asm`). A new game has no saved data to restore, so these codes
build her from child Rydia's entry instead. The first two codes are the ones
from "New game: Rydia is in the party": they change `fd 09` in event `$15`
(the last Baron castle scene) to `e7 03`, which adds child Rydia's `CharProp`
entry `$02` after Kain. The other two codes change that entry's first two
bytes: the id byte `$83` to `$91` (class id `$11`, adult Rydia, with the same
flag bits) and the graphic `$02` to `$0B`. `SpellListTbl` is indexed by the
graphic, so graphic `$0B` uses the adult Rydia row (`FF 03 04`: no white
magic, black list `$03`, summon list `$04`).

**Evidence:** `ff2_original.smc` has `DA FD 09 FE FB 66 9E 00` once, at
`$12:88C9`. `CharProp` entry `$02` at `$0F:A940` starts `83 02 01`: id
`$03`, graphic `$02`, level 1. The adult Rydia check in `EventCmd_e7`
(`and #$1f / cmp #$11 / bne / lda #$0b / sta $1001,y`) is
`29 1F C9 11 D0 05 A9 0B 99 01 10` at `$00:E5B5`. `SpellListTbl` (`$13:FDDD`) row
11 is `FF 03 04`. The codes decode back to their raw forms with
`snes_gg_encode.py`. Tested in Mesen2 together with the 3 codes
`B864-D0D6` + `D164-D006` + `336E-6DDD` of "Skip the opening and start
outside Baron": adult Rydia is in the party with Cecil and Kain.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only whether adult Rydia joins was tested, and only together with the skip
  cheat. Her commands, spells and equipment were not checked.
- She joins at level 1 with child Rydia's stats and starting equipment, not
  the stats she has in the story. Her spells come from `SpellListInit`.
- The two data codes change the `CharProp` entry the story also uses for
  child Rydia, so when the story adds Rydia later she is an adult too.
- The prologue text scroll is skipped.
- Uses the same bytes as the third and fourth codes (`3D6A-D16C` +
  `336A-D1AC`) of "Skip the opening and start outside Baron". To use both,
  take these codes, the first 2 codes of that entry (`B864-D0D6` +
  `D164-D006`) and `336E-6DDD` (`0082F0:EE`, the Package in the Sort slot).
  Only one "New game: ... is in the party" entry can be used at a time.

## Recall always casts and costs no MP

**Codes:** `DD3B-64D3` + `DDA7-67AE` (use both)
**Raw form:** `03EA98:00` (orig `$7F`) + `03CE3F:00` (orig `$7F`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 21:30:23

**Mechanism:** Recall (command `$07`, `Cmd_07` in `battle/cmd.asm`)
picks one of 8 black spells, then checks the spell's MP cost against the
caster's MP in its own code. If the cost is higher, it does nothing:

```
@eaac:  lda     $28a1
        and     #$7f        ; mp cost
        cmp     $a9         ; caster's mp
        beq     @eabb
        bcc     @eabb
        pla
        jmp     @eac5
@eabb:  pla
        sta     $26d2
        inc     $355d       ; consume mp
        jsr     DoMagicCmd
```

The first code changes `and #$7f` to `and #$00`, so the cost it compares
is 0 and the check always passes. Because Recall sets `$355d`, the spell
then goes through the normal MP subtract in `battle/magic.asm`, which
stops it with "not enough MP" if the cost is higher than the caster's MP.
The second code is the first code of "Spells cost no MP in battle": it
changes the `and #$7f` there to `and #$00`, so the cost is 0 and the
spell casts. With both codes, Recall casts even with 0 MP. This lets Dark
Knight Cecil, who has 0 MP at every level, use Recall from "Dark Knight
Cecil has the Recall command".

**Evidence:** `ff2_original.smc` has `AD A1 28 29 7F C5 A9 F0 06 90 04 68`
at `$03:EA94` (the mask is at `$03:EA98`). `AD A1 28 29 7F C5 A9` occurs
only once. It has `AD A1 28 29 7F 85 A9 38 AD 8B 26` at `$03:CE3B` (the
mask is at `$03:CE3F`), which also occurs only once. Dark Knight Cecil's
`CharProp` entry at `$0F:A900` has 0/0 MP, and byte 2 of every
`LevelUpProp_0000` entry (levels 10 to 69) has an MP gain of 0. The
codes decode back to their raw forms with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron" and "Dark Knight Cecil has the Recall command".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- No one has Recall in the USA version. Use this with "Tellah has the
  Recall command" or "Dark Knight Cecil has the Recall command".
- The second code makes every spell cost no MP in battle, not only
  Recall's. Spells that cost more than the caster's MP are still grayed
  out in the menu. Add `DD61-64A3` (the other code of "Spells cost no MP
  in battle") to pick them too.
- Recall still fails 1 time in 5, as in the original code.
- `DDA7-67AE` is also in "Spells cost no MP in battle". It writes the
  same byte with the same value, so the two can be used together.

## Dark Knight Cecil has the Recall command

**Code:** `D5E9-09AE`
**Raw form:** `13FD57:07` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 21:35:13

Tags: `demo-ready`

**Mechanism:** This works like "Tellah has the Recall command", which
has the details of `Cmd_07` (`battle/cmd.asm`). At the start of each
battle, `battle/init.asm` copies 5 command ids per class from `ClassCmd`
(`$13:FD55`) to `$3303`. Dark Knight Cecil's list is `00 01 FF FF FF`
(Fight, Item) at `$13:FD55`. The code puts `$07` in his third slot, so
Recall shows from the first battle. Recall casts one of 8 black spells,
or fails 1 time in 5. It checks the spell's MP cost against the caster's
MP first.

**Evidence:** `ff2_original.smc` has `00 01 FF FF FF` at `$13:FD55`, so
the byte at `$13:FD57` is `$FF`. The code decodes back to its raw form
with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron" and "Recall always casts and costs no MP".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The USA command name for `$07` is "Dummy", so the menu shows Dummy.
- Only Dark Knight Cecil (class `$01`) gets it. Paladin Cecil (class
  `$0B`) doesn't.
- Dark Knight Cecil has 0 MP at every level, so Recall always does
  nothing. Use it with "Recall always casts and costs no MP".
- What the battle messages for Recall say in the USA version was not
  checked.
- Uses the same byte (`$13:FD57`) as the other "Dark Knight Cecil has
  the ... command" entries and the first code of the "Every character
  has the ... command" entries in `ff2_long_game_genie.md`. Use only one
  of them.

## Dark Knight Cecil has the Salve command

**Code:** `DCE9-09AE`
**Raw form:** `13FD57:0A` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 21:35:13

Tags: `demo-ready`

**Mechanism:** This works like "Edward has the Salve command", which has
the details of `Cmd_0a` (`battle/cmd.asm`). At the start of each battle,
`battle/init.asm` copies 5 command ids per class from `ClassCmd`
(`$13:FD55`) to `$3303`. Dark Knight Cecil's list is `00 01 FF FF FF`
(Fight, Item) at `$13:FD55`. The code puts `$0A` in his third slot, so
Salve shows from the first battle. Salve uses up one Cure1 potion from
the battle inventory and uses it.

**Evidence:** `ff2_original.smc` has `00 01 FF FF FF` at `$13:FD55`, so
the byte at `$13:FD57` is `$FF`. The code decodes back to its raw form
with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The USA command name for `$0A` is "Dummy", so the menu shows Dummy.
- Only Dark Knight Cecil (class `$01`) gets it. Paladin Cecil (class
  `$0B`) doesn't.
- Salve uses up a Cure1 potion each time. It does nothing without one.
- What the battle messages for Salve say in the USA version was not
  checked.
- Uses the same byte (`$13:FD57`) as the other "Dark Knight Cecil has
  the ... command" entries and the first code of the "Every character
  has the ... command" entries in `ff2_long_game_genie.md`. Use only one
  of them.

## Dark Knight Cecil has the Pray command

**Code:** `D8E9-09AE`
**Raw form:** `13FD57:0B` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 21:35:13

Tags: `demo-ready`

**Mechanism:** This works like "Rosa has the Pray command", which has
the details of `Cmd_0b` (`battle/cmd.asm`). At the start of each battle,
`battle/init.asm` copies 5 command ids per class from `ClassCmd`
(`$13:FD55`) to `$3303`. Dark Knight Cecil's list is `00 01 FF FF FF`
(Fight, Item) at `$13:FD55`. The code puts `$0B` in his third slot, so
Pray shows from the first battle. Pray casts Cure1 with no MP cost half
of the time, and fails the other half. It doesn't need MP, so it works
for Dark Knight Cecil.

**Evidence:** `ff2_original.smc` has `00 01 FF FF FF` at `$13:FD55`, so
the byte at `$13:FD57` is `$FF`. The code decodes back to its raw form
with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The USA command name for `$0B` is "Dummy", so the menu shows Dummy.
- Only Dark Knight Cecil (class `$01`) gets it. Paladin Cecil (class
  `$0B`) doesn't.
- What the battle messages for Pray say in the USA version was not
  checked.
- Uses the same byte (`$13:FD57`) as the other "Dark Knight Cecil has
  the ... command" entries and the first code of the "Every character
  has the ... command" entries in `ff2_long_game_genie.md`. Use only one
  of them.

## Dark Knight Cecil has the Brace command

**Code:** `DEE9-09AE`
**Raw form:** `13FD57:0F` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 21:35:13

**Mechanism:** This works like "Yang has the Brace command", which has
the details of `Cmd_0f` (`battle/cmd.asm`). At the start of each battle,
`battle/init.asm` copies 5 command ids per class from `ClassCmd`
(`$13:FD55`) to `$3303`. Dark Knight Cecil's list is `00 01 FF FF FF`
(Fight, Item) at `$13:FD55`. The code puts `$0F` in his third slot, so
Brace shows from the first battle. Brace casts Armor with no MP cost. It
doesn't need MP, so it works for Dark Knight Cecil.

**Evidence:** `ff2_original.smc` has `00 01 FF FF FF` at `$13:FD55`, so
the byte at `$13:FD57` is `$FF`. The code decodes back to its raw form
with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The USA command name for `$0F` is "Dummy", so the menu shows Dummy.
- Only Dark Knight Cecil (class `$01`) gets it. Paladin Cecil (class
  `$0B`) doesn't.
- What the battle messages for Brace say in the USA version was not
  checked.
- Uses the same byte (`$13:FD57`) as the other "Dark Knight Cecil has
  the ... command" entries and the first code of the "Every character
  has the ... command" entries in `ff2_long_game_genie.md`. Use only one
  of them.

## Dark Knight Cecil has the Bluff command

**Code:** `FFE9-09AE`
**Raw form:** `13FD57:11` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 21:35:13

**Mechanism:** This works like "Palom has the Bluff command", which has
the details of `Cmd_11` (`battle/cmd.asm`). At the start of each battle,
`battle/init.asm` copies 5 command ids per class from `ClassCmd`
(`$13:FD55`) to `$3303`. Dark Knight Cecil's list is `00 01 FF FF FF`
(Fight, Item) at `$13:FD55`. The code puts `$11` in his third slot, so
Bluff shows from the first battle. Bluff adds 16 to the user's intellect
for the rest of the battle, up to 99.

**Evidence:** `ff2_original.smc` has `00 01 FF FF FF` at `$13:FD55`, so
the byte at `$13:FD57` is `$FF`. The code decodes back to its raw form
with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The USA command name for `$11` is "Dummy", so the menu shows Dummy.
- Only Dark Knight Cecil (class `$01`) gets it. Paladin Cecil (class
  `$0B`) doesn't.
- Intellect only makes black magic stronger. Dark Knight Cecil has no
  black magic, and Fight doesn't use intellect, so Bluff has no effect
  you can see.
- What the battle messages for Bluff say in the USA version was not
  checked.
- Uses the same byte (`$13:FD57`) as the other "Dark Knight Cecil has
  the ... command" entries and the first code of the "Every character
  has the ... command" entries in `ff2_long_game_genie.md`. Use only one
  of them.

## Dark Knight Cecil has the Cry command

**Code:** `F4E9-09AE`
**Raw form:** `13FD57:12` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 21:35:13

**Mechanism:** This works like "Porom has the Cry command", which has
the details of `Cmd_12` (`battle/cmd.asm`). At the start of each battle,
`battle/init.asm` copies 5 command ids per class from `ClassCmd`
(`$13:FD55`) to `$3303`. Dark Knight Cecil's list is `00 01 FF FF FF`
(Fight, Item) at `$13:FD55`. The code puts `$12` in his third slot, so
Cry shows from the first battle. Cry lowers each monster's level by half
of the user's level, to no less than 1.

**Evidence:** `ff2_original.smc` has `00 01 FF FF FF` at `$13:FD55`, so
the byte at `$13:FD57` is `$FF`. The code decodes back to its raw form
with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The USA command name for `$12` is "Dummy", so the menu shows Dummy.
- Only Dark Knight Cecil (class `$01`) gets it. Paladin Cecil (class
  `$0B`) doesn't.
- The code is also the first code of "Every character has the Cry
  command" (`ff2_long_game_genie.md`).
- What the battle messages for Cry say in the USA version was not
  checked.
- Uses the same byte (`$13:FD57`) as the other "Dark Knight Cecil has
  the ... command" entries and the first code of the "Every character
  has the ... command" entries in `ff2_long_game_genie.md`. Use only one
  of them.

## Dark Knight Cecil has the Regen command

**Code:** `FBE9-09AE`
**Raw form:** `13FD57:19` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 21:35:13

Tags: `demo-ready`

**Mechanism:** This works like "FuSoYa has the Regen command", which has
the details of `Cmd_19` (`battle/cmd.asm`). At the start of each battle,
`battle/init.asm` copies 5 command ids per class from `ClassCmd`
(`$13:FD55`) to `$3303`. Dark Knight Cecil's list is `00 01 FF FF FF`
(Fight, Item) at `$13:FD55`. The code puts `$19` in his third slot, so
Regen shows from the first battle. Regen gives the user 10 HP every 5
ticks for a while.

**Evidence:** `ff2_original.smc` has `00 01 FF FF FF` at `$13:FD55`, so
the byte at `$13:FD57` is `$FF`. The code decodes back to its raw form
with `snes_gg_encode.py`. Tested in Mesen2.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The USA command name for `$19` is "Dummy", so the menu shows Dummy.
- Only Dark Knight Cecil (class `$01`) gets it. Paladin Cecil (class
  `$0B`) doesn't.
- With the second code of "Spells and slow commands go off with no
  wait", Regen ends at once and gives no HP.
- What the battle messages for Regen say in the USA version was not
  checked.
- Uses the same byte (`$13:FD57`) as the other "Dark Knight Cecil has
  the ... command" entries and the first code of the "Every character
  has the ... command" entries in `ff2_long_game_genie.md`. Use only one
  of them.

## Dark Knight Cecil has the Dark Wave command

**Code:** `D9E9-09AE`
**Raw form:** `13FD57:05` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 21:36:44

Tags: `demo-ready`

**Mechanism:** This works like "Paladin Cecil has the Dark Wave
command", which has the details of `Cmd_05` (`battle/cmd.asm`). The USA
release took Dark Wave away from Dark Knight Cecil. At the start of each
battle, `battle/init.asm` copies 5 command ids per class from `ClassCmd`
(`$13:FD55`) to `$3303`. Dark Knight Cecil's list is `00 01 FF FF FF`
(Fight, Item) at `$13:FD55`. The code puts `$05` in his third slot, so
Dark Wave shows from the first battle. Dark Wave takes 1/8 of the user's
max HP, then hits every enemy.

**Evidence:** `ff2_original.smc` has `00 01 FF FF FF` at `$13:FD55`, so
the byte at `$13:FD57` is `$FF`. The code decodes back to its raw form
with `snes_gg_encode.py`. Tested in Mesen2.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The USA command name for `$05` is "Dummy", so the menu shows Dummy.
- Only Dark Knight Cecil (class `$01`) gets it. Paladin Cecil (class
  `$0B`) doesn't.
- Dark Wave can kill Cecil: if his HP is less than 1/8 of his max HP, it
  drops to 0 and he dies. "Dark Wave costs no HP" removes the cost.
- The USA ROM may lack the Dark Wave attack data or animation, so the
  attack may look or work wrong.
- Uses the same byte (`$13:FD57`) as the other "Dark Knight Cecil has
  the ... command" entries and the first code of the "Every character
  has the ... command" entries in `ff2_long_game_genie.md`. Use only one
  of them.

## Baron's item shop sells 5 unused field items

**Codes:** `2ECB-A1AA` + `3FCB-A5DA` + `30CB-A50A` + `31CB-A56A` +
`3BCB-A5AA` (use all 5)
**Raw form:** `13A39B:DF` (orig `$EB`) + `13A39C:E1` (orig `$DD`) +
`13A39D:E4` (orig `$D1`) + `13A39E:E6` (orig `$FF`) + `13A39F:E9` (orig
`$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 22:25:07

Tags: `demo-ready` `curated`

**Mechanism:** `ShopProp` (`$13:A300`) has 8 item ids per shop, with
`$FF` for an empty slot. Event command `$ED nn` opens shop `nn`. The
Baron item shop is shop `$13` (event `$26`), at `$13:A398`:
`CE D4 E2 EB DD D1 FF FF` (Cure1, Life, Tent, Carrot, Heal, Ether1). The
codes put AuApple (`$DF`), SomaDrop (`$E1`) and Book (`$E4`) in place of
Carrot, Heal and Ether1, and Bread (`$E6`) and Cockatrice (`$E9`) in the
two empty slots. `LoadShopList` in `menu/shop.asm` stops at the first
`$FF`, so the shop then lists all 8 items. Prices come from `ItemPrice`
(`$0F:A450`, one byte per item: bit 7 set means (byte & `$7F`) × 1000,
else byte × 10).

**Evidence:** `ff2_original.smc` has `CE D4 E2 EB DD D1 FF FF` at
`$13:A398`, and this pattern is found nowhere else in the ROM. The codes
decode back to their raw forms with `snes_gg_encode.py`.
Tested in Mesen2 together with "Winning a battle sets your gil to
9,999,999", with AgApple (`$E0`, code `3DCB-A5DA`) in place of SomaDrop.
`ItemPrice` has `$01` for SomaDrop, so it costs 10 gil.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested since AgApple was replaced with SomaDrop.
- The unused items have no USA names, so all 5 show as "Dummy". In
  shop order they are AuApple (1,000 gil), SomaDrop (10), Book (580),
  Bread (100) and Cockatrice (300), after Cure1, Life and Tent.
- Carrot, Heal and Ether1 are no longer sold in Baron.
- The codes are in one entry because the shop list stops at the first
  empty slot: the Cockatrice code (`3BCB-A5AA`) shows nothing without
  the Bread code (`31CB-A56A`).
- AuApple adds 100 max HP and SomaDrop adds 10 max MP. Book works only on town
  and dungeon maps and shows a short message. Bread works only where the
  Sight spell works. Cockatrice teaches Rydia the unused summon, and does
  nothing if she isn't in the party. See
  "Start a new game with 99 of an unused field item" for the
  details.
- Only the Baron item shop changes. Other shops are not affected.

## Dark Knight Cecil has the Jump command

**Code:** `D1E9-09AE`
**Raw form:** `13FD57:06` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 23:00:01

Tags: `demo-ready` `curated`

**Mechanism:** The USA release gives Jump (command `$06`) to Kain. At
the start of each battle, `battle/init.asm` copies 5 command ids per
class from `ClassCmd` (`$13:FD55`) to `$3303`. Dark Knight Cecil's list
is `00 01 FF FF FF` (Fight, Item) at `$13:FD55`. The code puts `$06` in
his third slot, so Jump shows from the first battle. `Cmd_06`
(`battle/cmd.asm`) makes the user jump out of sight for a while. When
the timer runs out, `Cmd_1e` (jump 2) does a Fight attack and the user
lands.

**Evidence:** `ff2_original.smc` has `00 01 FF FF FF` at `$13:FD55`, so
the byte at `$13:FD57` is `$FF`. The code decodes back to its raw form
with `snes_gg_encode.py`. Tested in Mesen2.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only Dark Knight Cecil (class `$01`) gets it. Paladin Cecil (class
  `$0B`) doesn't.
- Uses the same byte (`$13:FD57`) as the other "Dark Knight Cecil has
  the ... command" entries and the first code of the "Every character
  has the ... command" entries in `ff2_long_game_genie.md`. Use only one
  of them.

## Dark Knight Cecil has the Kick command

**Code:** `D3E9-09AE`
**Raw form:** `13FD57:0E` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 23:00:01

Tags: `demo-ready`

**Mechanism:** The USA release gives Kick (command `$0E`) to Yang. At
the start of each battle, `battle/init.asm` copies 5 command ids per
class from `ClassCmd` (`$13:FD55`) to `$3303`. Dark Knight Cecil's list
is `00 01 FF FF FF` (Fight, Item) at `$13:FD55`. The code puts `$0E` in
his third slot, so Kick shows from the first battle. `Cmd_0e`
(`battle/cmd.asm`) does a Fight attack on every monster
(`DoMultiAttack`).

**Evidence:** `ff2_original.smc` has `00 01 FF FF FF` at `$13:FD55`, so
the byte at `$13:FD57` is `$FF`. The code decodes back to its raw form
with `snes_gg_encode.py`. Tested in Mesen2.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only Dark Knight Cecil (class `$01`) gets it. Paladin Cecil (class
  `$0B`) doesn't.
- Uses the same byte (`$13:FD57`) as the other "Dark Knight Cecil has
  the ... command" entries and the first code of the "Every character
  has the ... command" entries in `ff2_long_game_genie.md`. Use only one
  of them.

## Dark Knight Cecil has the Aim command

**Code:** `DAE9-09AE`
**Raw form:** `13FD57:0C` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 23:00:01

**Mechanism:** The USA release gives Aim (command `$0C`) to Rosa. At the
start of each battle, `battle/init.asm` copies 5 command ids per class
from `ClassCmd` (`$13:FD55`) to `$3303`. Dark Knight Cecil's list is `00
01 FF FF FF` (Fight, Item) at `$13:FD55`. The code puts `$0C` in his
third slot, so Aim shows from the first battle. `Cmd_0c`
(`battle/cmd.asm`) sets the user's hit rate (`$269C`) to 255 and the
target's defense rate (`$2729`) to 0, does a Fight attack
(`DoFightCmd`), then puts both back. But `CheckCmdEnabled`
(`battle/char.asm`) grays out Aim unless the user has a bow in the right
hand and arrows in the left hand, and Dark Knight Cecil can't equip a
bow.

**Evidence:** `ff2_original.smc` has `00 01 FF FF FF` at `$13:FD55`, so
the byte at `$13:FD57` is `$FF`. The code decodes back to its raw form
with `snes_gg_encode.py`. Tested in Mesen2 together with
"Aim works without a bow".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Aim is grayed out. Use it with "Aim works without a bow", and then it
  is a Fight attack with his sword that always hits.
- Only Dark Knight Cecil (class `$01`) gets it. Paladin Cecil (class
  `$0B`) doesn't.
- Uses the same byte (`$13:FD57`) as the other "Dark Knight Cecil has
  the ... command" entries and the first code of the "Every character
  has the ... command" entries in `ff2_long_game_genie.md`. Use only one
  of them.

## Dark Knight Cecil has the Cover command

**Codes:** `F7E9-09AE` + `FA69-046E` (use both)
**Raw form:** `13FD57:13` (orig `$FF`) + `038D5A:1C` (orig `$1F`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 23:00:01

Tags: `demo-ready`

**Mechanism:** The USA release gives Cover (command `$13`) to Paladin
Cecil. At the start of each battle, `battle/init.asm` copies 5 command
ids per class from `ClassCmd` (`$13:FD55`) to `$3303`. Dark Knight
Cecil's list is `00 01 FF FF FF` (Fight, Item) at `$13:FD55`. The code
puts `$13` in his third slot, so Cover shows from the first battle.
`Cmd_13` (`battle/cmd.asm`) makes the user take hits for the chosen
partner, and changes the command to `$1D`, which turns it off again.

When a monster's Fight attack targets a character who is covered or has
low HP, `battle/attack.asm` changes the target to the character in slot
`$355E`, if that character can act. At the start of each battle,
`battle/init.asm` puts the slot of the class `$0B` character (Paladin
Cecil) there. Dark Knight Cecil (class `$01`) skips it:

```
@8d49:  lda     $3540,y
        bne     @8d5f
        lda     $c7
        bne     @8d7a
        lda     $2000,x
        and     #$1f
        cmp     #$01
        beq     @8d7a
        cmp     #$0b
        beq     @8d77
        ...
@8d77:  sty     $355e
@8d7a:  lda     $2015,x
```

The second code changes the first `beq` offset from `$1F` to `$1C`, so
class `$01` also goes to `@8d77` and Dark Knight Cecil takes the hits.

**Evidence:** `ff2_original.smc` has `00 01 FF FF FF` at `$13:FD55`, so
the byte at `$13:FD57` is `$FF`. It has
`BD 00 20 29 1F C9 01 F0 1F C9 0B F0 18` at `$03:8D52`, so the first
`beq` offset is at `$03:8D5A`. This pattern occurs only once. The codes
decode back to their raw forms with `snes_gg_encode.py`. Tested in Mesen2.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Cover needs another party member to cover.
- With the second code, Dark Knight Cecil also takes Fight hits for
  party members with low HP, like Paladin Cecil does.
- Only monsters' Fight attacks are covered.
- Only Dark Knight Cecil (class `$01`) gets it. Paladin Cecil (class
  `$0B`) doesn't.
- Uses the same byte (`$13:FD57`) as the other "Dark Knight Cecil has
  the ... command" entries and the first code of the "Every character
  has the ... command" entries in `ff2_long_game_genie.md`. Use only one
  of them.

## Dark Knight Cecil has the Sneak command

**Code:** `F5E9-09AE`
**Raw form:** `13FD57:17` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 23:00:01

Tags: `demo-ready`

**Mechanism:** The USA release gives Sneak (command `$17`) to Edge. At
the start of each battle, `battle/init.asm` copies 5 command ids per
class from `ClassCmd` (`$13:FD55`) to `$3303`. Dark Knight Cecil's list
is `00 01 FF FF FF` (Fight, Item) at `$13:FD55`. The code puts `$17` in
his third slot, so Sneak shows from the first battle. `Cmd_17`
(`battle/cmd.asm`) steals from a monster. The chance is (user level +
50) - (monster level + 10), from 1 to 99 percent. If it fails, there is
a (monster level + 10) percent chance that the monster sees him and he
takes 1/16 of his max HP.

**Evidence:** `ff2_original.smc` has `00 01 FF FF FF` at `$13:FD55`, so
the byte at `$13:FD57` is `$FF`. The code decodes back to its raw form
with `snes_gg_encode.py`. Tested in Mesen2.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- A steal takes the first item of the monster's item set (`MonsterItems`),
  or 10 of it if it is an arrow. Monsters whose item drop rate bits
  (`$2773` & `$C0`) are `$C0` have nothing to steal, and the steal also
  fails if the inventory is full.
- Only Dark Knight Cecil (class `$01`) gets it. Paladin Cecil (class
  `$0B`) doesn't.
- Uses the same byte (`$13:FD57`) as the other "Dark Knight Cecil has
  the ... command" entries and the first code of the "Every character
  has the ... command" entries in `ff2_long_game_genie.md`. Use only one
  of them.

## Aim works without a bow

**Code:** `D9C6-670A`
**Raw form:** `03A28D:05` (orig `$0C`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 23:05:13

**Mechanism:** Before a character's turn, `CheckCmdEnabled`
(`battle/char.asm`) sends Dark Wave, Sing, Aim and Twin through
`CheckCmdReq`, which grays out the command if its requirement isn't met:

```
CheckCmdEnabled:
        ...
        cmp     #$05
        beq     @a294
        cmp     #$08
        beq     @a294
        cmp     #$0c
        beq     @a294
        cmp     #$10
        bne     @a29b
; dark wave, sing, aim, twin
@a294:  jsr     CheckCmdReq
```

For Aim, `CheckCmdReq` needs a bow (item `$4D` to `$60`) in the right
hand and something (arrows) in the left hand. The code changes `cmp
#$0c` to `cmp #$05`, a second check for Dark Wave, so Aim skips
`CheckCmdReq`. `Cmd_0c` (`battle/cmd.asm`) doesn't check the weapon
itself. It sets the hit rate to 255 and the target's defense rate to 0
and does a Fight attack with whatever the user holds.

**Evidence:** `ff2_original.smc` has
`C9 05 F0 0C C9 08 F0 08 C9 0C F0 04 C9 10 D0 07` at `$03:A284`, so the
`$0C` operand is at `$03:A28D`. This pattern occurs only once. The code
decodes back to its raw form with `snes_gg_encode.py`. Tested in
Mesen2 together with "Dark Knight Cecil has the Aim command".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Rosa can also use Aim without a bow.
- Status effects that disable Aim (toad, mini, pig) still disable it.
- Use it with "Dark Knight Cecil has the Aim command" to give Dark
  Knight Cecil a sure-hit sword attack.

## Every monster has the items from 32 places later in the monster item list

**Codes:** `6D3D-0D0E` + `6D30-6D6A` (use both)
**Raw form:** `03ED01:80` (orig `$00`) + `03E242:80` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 23:18:59

**Mechanism:** Byte 7 of each monster's properties holds its item drop
rate (bits 6-7) and item set (bits 0-5). `MonsterItems` (`$0E:9F00`)
has 64 sets of 4 items. After a won battle, `battle/win.asm` multiplies
the set number by 4 and copies that set's 4 items, then picks one at
random:

```
@ed0f:  asl     $b3
        asl     $b3
        lda     $b3
        tax
        clr_ay
        sty     $a9
@ed1a:  lda     f:MonsterItems,x
        sta     $289c,y
```

The steal command (`Cmd_17` in `battle/cmd.asm`) reads the first item
of the same set:

```
@e24c:  clr_a
        inc
        sta     $aa         ; qty: 1
        lda     $2773       ; monster item set
        and     #$3f
        jsr     Asl_2
        tax
        lda     f:MonsterItems,x        ; monster items (always choose 1st item)
```

Each code changes the low byte of one `MonsterItems` address from `$00`
to `$80`, so the table starts `$80` bytes (32 sets) later. The first
code is for drops and the second is for steals. The drop rate doesn't
change.

Some examples: FloatEye (set 3) drops Cure1 instead of Heal. Larva and
Cave Bat (set 16) drop Heal, Ether2, Life or Ribbon instead of Cure1 or
Cure2. Sand Man (set 2) drops Cure, Rune or Silence instead of Heal or
Tent.

**Evidence:** `ff2_original.smc` has `BF 00 9F 0E 99 9C 28` at
`$03:ED00` and `BF 00 9F 0E F0 DD` at `$03:E241`. `BF 00 9F 0E` occurs
only at these two places. The item sets were read from `$0E:9F00`, the
monster item set bytes from the properties that `MonsterPropPtrs`
(`$0E:A6A0`) points to, and the item names from `$0F:8000`. The `ff4j`
ROM map lists 64 sets of 4 bytes at `$0E:9F00`-`$0E:9FFF`. Each code
decodes back to its raw form with `snes_gg_encode.py`.
Tested in Mesen2 together with "Dark Knight Cecil has the Sneak command".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Sets 32-63 read past the end of the table, into the monster gil table
  at `$0E:A000`. Many monsters use these sets, for example the Imp (set
  56) and the Eagle (set 57). Their items are garbage: some slots are
  `$00` (probably no item), and others are weapons, armor or unused
  dummy items.
- Use both codes, or only the first for drops. With only the second,
  only steals change.
- This changes the items of every monster, bosses too.

## Dark Knight Cecil has Porom's white magic

**Codes:** `D4E9-09AE` + `DBE2-050E` + `DBEC-A0B7` (use all 3)
**Raw form:** `13FD57:02` (orig `$FF`) + `13FDDD:09` (orig `$FF`) +
`14FFA2:09` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 23:28:29

**Mechanism:** At the start of each battle, `battle/init.asm` copies 5
command ids per class from `ClassCmd` (`$13:FD55`) to `$3303`. Dark
Knight Cecil's list is `00 01 FF FF FF` (Fight, Item). The first code
puts White (`$02`) in his third slot. `battle/init.asm` also builds each
character's white, black and summon lists from `SpellListTbl`
(`$13:FDDD`, 3 bytes per character, indexed by the low nibble of
`$1001`). Dark Knight Cecil's row (graphic `$00`) is `FF FF FF`. The
second code puts Porom's white list (`$09`) in it. The field menu has
its own copy of the table (`$14:FFA2`, `menu/system.asm`), and
`menu/magic.asm` skips a character whose row is `FF FF FF`. The third
code puts `$09` there too. Spell lists are stored in RAM by list number
(24 spells each, from `$1560`), so Cecil and Porom share this list.
`LevelUpMagic` (`battle/win.asm`) uses the same row, so Cecil's
level-ups teach the `$09` schedule in `SpellListLearned`
(`$0F:C700`): white spells at levels 11 to 56.

**Evidence:** `ff2_original.smc` has `00 01 FF FF FF` at `$13:FD55`, and
`FF FF FF FF FF FF 02 03 04 05 06 FF` at both `$13:FDDD` and `$14:FFA2`.
List `$09` in `SpellListInit` (`$0F:C8C0`) is `0E 01 0D 07 17` on a new
game. The codes decode back to their raw forms with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron", "Dark Knight Cecil starts with 200 MP" and "Dark
Knight Cecil gains 4 max MP per level".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Dark Knight Cecil has 0 MP. Use it with "Dark Knight Cecil starts with
  200 MP" and "Dark Knight Cecil gains 4 max MP per level".
- Can be used together with "Dark Knight Cecil has Palom's black magic".
  Then his commands are Fight, Item, White, Black.
- Cecil and Porom share the spells they know. When Porom joins, Porom
  knows every spell Cecil learned, and the other way around.
- Only Dark Knight Cecil (class `$01`) gets it. Paladin Cecil (class
  `$0B`) and his own spell list don't change.
- Uses the same byte (`$13:FD57`) as the other "Dark Knight Cecil has
  the ... command" entries and the first code of the "Every character
  has the ... command" entries in `ff2_long_game_genie.md`. Use only one
  of them.

## Dark Knight Cecil has Palom's black magic

**Codes:** `D7E9-01DE` + `D6E2-056E` + `D6EC-A027` (use all 3)
**Raw form:** `13FD58:03` (orig `$FF`) + `13FDDE:08` (orig `$FF`) +
`14FFA3:08` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 23:28:29

**Mechanism:** At the start of each battle, `battle/init.asm` copies 5
command ids per class from `ClassCmd` (`$13:FD55`) to `$3303`. Dark
Knight Cecil's list is `00 01 FF FF FF` (Fight, Item). The first code
puts Black (`$03`) in his fourth slot. `battle/init.asm` also builds
each character's white, black and summon lists from `SpellListTbl`
(`$13:FDDD`, 3 bytes per character, indexed by the low nibble of
`$1001`). Dark Knight Cecil's row (graphic `$00`) is `FF FF FF`. The
second code puts Palom's black list (`$08`) in it. The field menu has
its own copy of the table (`$14:FFA2`, `menu/system.asm`), and
`menu/magic.asm` skips a character whose row is `FF FF FF`. The third
code puts `$08` there too. Spell lists are stored in RAM by list number
(24 spells each, from `$1560`), so Cecil and Palom share this list.
`LevelUpMagic` (`battle/win.asm`) uses the same row, so Cecil's
level-ups teach the `$08` schedule in `SpellListLearned`
(`$0F:C700`): black spells at levels 11 to 52.

**Evidence:** `ff2_original.smc` has `00 01 FF FF FF` at `$13:FD55`, and
`FF FF FF FF FF FF 02 03 04 05 06 FF` at both `$13:FDDD` and `$14:FFA2`.
List `$08` in `SpellListInit` (`$0F:C8C0`) is `1D 20 23 29 1C` on a new
game. The codes decode back to their raw forms with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron", "Dark Knight Cecil starts with 200 MP" and "Dark
Knight Cecil gains 4 max MP per level".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Dark Knight Cecil has 0 MP. Use it with "Dark Knight Cecil starts with
  200 MP" and "Dark Knight Cecil gains 4 max MP per level".
- Can be used together with "Dark Knight Cecil has Porom's white magic".
  Then his commands are Fight, Item, White, Black.
- Cecil and Palom share the spells they know. When Palom joins, Palom
  knows every spell Cecil learned, and the other way around.
- Only Dark Knight Cecil (class `$01`) gets it. Paladin Cecil (class
  `$0B`) and his own spell list don't change.
- Without the white magic entry, his third command slot is empty, and
  the cursor skips it.

## Dark Knight Cecil has Rydia's summons

**Codes:** `D0E9-010E` + `D0E2-05AE` + `D0EC-A9F7` (use all 3)
**Raw form:** `13FD59:04` (orig `$FF`) + `13FDDF:04` (orig `$FF`) +
`14FFA4:04` (orig `$FF`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 23:39:11

**Mechanism:** This works like "Dark Knight Cecil has Porom's white
magic". The first code puts Call (`$04`) in the fifth slot of his
`ClassCmd` list (`$13:FD55`). The second code puts Rydia's summon list
(`$04`) in the third byte of his `SpellListTbl` row (`$13:FDDD`), and
the third code does the same in the field menu's copy (`$14:FFA2`).
Spell lists are stored in RAM by list number, so Cecil and Rydia share
this list. List `$04` has no level-up schedule in `SpellListLearned`, so
Cecil only gets the summons Rydia gets from the story.

**Evidence:** `ff2_original.smc` has `00 01 FF FF FF` at `$13:FD55`, and
`FF FF FF FF FF FF 02 03 04 05 06 FF` at both `$13:FDDD` and `$14:FFA2`.
List `$04` in `SpellListInit` (`$0F:C8C0`) is `35` on a new game, and
its `SpellListLearned` (`$0F:C700`) entry is empty. The codes decode
back to their raw forms with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron", "Dark Knight Cecil starts with 200 MP" and "Dark
Knight Cecil gains 4 max MP per level".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Dark Knight Cecil has 0 MP. Use it with "Dark Knight Cecil starts with
  200 MP" and "Dark Knight Cecil gains 4 max MP per level".
- Can be used together with "Dark Knight Cecil has Porom's white magic"
  and "Dark Knight Cecil has Palom's black magic". Then his commands are
  Fight, Item, White, Black, Call.
- Without those entries, his third and fourth command slots are empty,
  and the cursor skips them.
- Cecil and Rydia share the summons they know.
- Only Dark Knight Cecil (class `$01`) gets it. Paladin Cecil (class
  `$0B`) doesn't.

## Dark Knight Cecil starts with 200 MP

**Codes:** `A6CD-04E3` + `A6CD-0753` (use both)
**Raw form:** `0FA90B:C8` (orig `$00`) + `0FA90D:C8` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 23:23:50

**Mechanism:** This works like "Tellah joins with 858 MP". `CharProp`
(`$0FA900`, 32 bytes per entry) holds each character's starting stats.
Bytes 11-12 are current MP and bytes 13-14 are max MP. Dark Knight Cecil
is entry 0, with 0/0 MP. The codes change the low bytes to `$C8`, so he
has 200/200 MP. `InitCharProp` copies the entry to the party when a new
game starts.

**Evidence:** `ff2_original.smc` has `81 00 0A 00 00 00 00 C8 00 C8 00
00 00 00 00` at `$0F:A900` (Dark Knight Cecil, level 10, 200 HP, 0 MP).
The codes decode back to their raw forms with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron", "Dark Knight Cecil has Porom's white magic" and "Dark Knight Cecil
gains 4 max MP per level".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only works on a new game. A saved game keeps its MP.
- His level-ups give 0 MP. "Dark Knight Cecil gains 4 max MP per level"
  fixes that.
- Paladin Cecil has his own `CharProp` entry, so he doesn't get the MP.

## Dark Knight Cecil gains 4 max MP per level

**Codes:** `FD8D-0D72` + `AD8D-0D52` (use both)
**Raw form:** `0FB500:10` (orig `$FB`) + `0FB501:C0` (orig `$B4`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-28 23:23:50

**Mechanism:** At a level-up, `battle/win.asm` reads 5 bytes for the new
level from the class's table, through `LevelUpPropPtrs` (`$0F:B500`, one
word per class). `LevelUp` adds the low 5 bits of the third byte to max
MP, plus a random 0 to 1/8 of it (`LevelUpHPMP`). Dark Knight Cecil's
table has 0 there at every level. The codes change his pointer from
`$B4FB` to `$C010`, Paladin Cecil's table, which has 4 at every level.
The table is indexed by level, so he uses Paladin Cecil's row for his
level.

**Evidence:** `ff2_original.smc` has `FB B4 2F B6` at `$0F:B500`, and
the class `$0B` word (`$0F:B514`) is `$C010`. Paladin Cecil's rows at
levels 10 to 13 are `A1 18 04 EA 02`, `09 18 04 B1 03`, `B1 18 04 9C 04`
and `00 18 04 AB 05`. Dark Knight Cecil's are `E1 14 00 A3 03`,
`C1 14 00 A2 04`, `B1 14 00 CE 05` and `E1 14 00 2A 07`. The codes
decode back to their raw forms with `snes_gg_encode.py`.
Tested in Mesen2 together with the 4 codes of "Skip the opening and start
outside Baron", "Dark Knight Cecil has Porom's white magic" and "Dark Knight Cecil
starts with 200 MP".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- He also gets Paladin Cecil's HP and stat gains and his experience
  needs. At level 10 to 13 he needs less experience for each level and
  gains 24 HP in place of 20.
- Max MP goes up. Current MP doesn't.
- Works on saved games too.

## Monsters always drop their rarest item

**Codes:** `EE3F-07DE` + `D734-0DDE` (use both)
**Raw form:** `03ED1C:FF` (orig `$80`) + `03ED20:03` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-29 00:03:46

Tags: `curated`

**Mechanism:** When a monster drops an item, `WinUpdate`
(`battle/win.asm`) copies the 4 items of its set from `MonsterItems`
and picks one with a random number from 0 to 255:

```
        jsr     Rand
        cmp     #$80
        bcs     @ed3d
        lda     #$00
        bra     @ed4f
@ed3d:  cmp     #$d0
        bcs     @ed45
        lda     #$01
        bra     @ed4f
@ed45:  cmp     #$fc
        bcs     @ed4d
        lda     #$02
        bra     @ed4f
@ed4d:  lda     #$03
@ed4f:  tax
        lda     $289c,x
```

So the slots come up 128, 80, 44 and 4 times in 256 (about 50%, 31%,
17% and 1.6%). The first code changes `cmp #$80` to `cmp #$ff`, so every
number but `$FF` takes the first branch, and the second code changes its
`lda #$00` to `lda #$03`. `$FF` still goes through the other compares to
`@ed4d`. Every drop is the fourth item.

**Evidence:** The `@xxxx` labels here don't match the USA ROM.
`ff2_original.smc` has
`20 93 85 C9 80 B0 04 A9 00 80 12 C9 D0 B0 04 A9 01 80 0A C9 FC B0 04 A9
02 80 02 A9 03 AA BD 9C 28` at `$03:ED18`. The pattern occurs only once.
The codes decode back to their raw forms with `snes_gg_encode.py`.
Tested in Mesen2 together with "Every defeated monster drops an item".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only the item choice changes. The drop rate doesn't: most monsters
  still drop nothing most of the time. Add "Every defeated monster drops
  an item" (`CB32-D4DE`) to make every kill drop one.
- Some sets may have no item or a weak one in the fourth slot. The sets
  were not checked one by one.
- "Sneak always takes the monster's rarest item" does the same for
  steals.

## Every monster has the items from 47 places later in the monster item list

**Codes:** `8A3D-0D0E` + `8A30-6D6A` (use both)
**Raw form:** `03ED01:BC` (orig `$00`) + `03E242:BC` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-29 00:05:50

Tags: `demo-ready` `curated`

**Mechanism:** The same as "Every monster has the items from 32 places
later in the monster item list", but each code sets the low byte of the
`MonsterItems` address to `$BC`, so the table starts `$BC` bytes (47
sets) later. The first code is for drops and the second is for steals.
The drop rate doesn't change.

The sets are not in level order, so 47 was picked for set 0, which 82 of
the 224 monsters use, from level 1 to 99. Set 0 (mostly Cure1) becomes
set 47: Headband, Strength, Power or Avenger. Set 1 becomes set 48
(Heal, Ether, Life or Ribbon), and set 16 becomes set 63 (Elixir).

**Evidence:** `ff2_original.smc` has `BF 00 9F 0E` at `$03:ED00` and
`$03:E241`, and nowhere else. The item sets were read from `$0E:9F00`,
the monster item set bytes from the properties that `MonsterPropPtrs`
(`$0E:A6A0`) points to, and the item names from `$0F:8000`. Each code
decodes back to its raw form with `snes_gg_encode.py`.
Tested in Mesen2 together with "Dark Knight Cecil has the Sneak command".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Sets 17-63 read past the end of the table, into the monster gil table
  at `$0E:A000`. 81 of the 224 monsters use these sets, so their items
  are garbage: no item, or weapons, armor or unused dummy items.
- Use both codes, or only the first for drops. With only the second,
  only steals change.
- Don't use it with the 32-places cheat. They write the same 2 bytes.
- This changes the items of every monster, bosses too.

## Every sound effect is the one after it in the sound effect list

**Code:** `D0B0-DFBF`
**Raw form:** `049446:04` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, instead, 2 places later:** `D6B0-DFBF` (`049446:08`)
**Optional, instead, 4 places later:** `FDB0-DFBF` (`049446:10`)
**Recorded:** 2026-10-29 00:07:52

Tags: `demo-ready` `curated` `play-start`

**Mechanism:** This is the FF2 version of the ALTTP code "Every sound
effect comes from the wrong sound list". FF2 has only one list of game
sound effects, so this code shifts that list instead of swapping two.

Every part of the game plays a sound effect the same way, like
`PlaySfx` in `field/field.asm`: it puts `$02` in `$1e00` and the sound
effect id in `$1e01`, then calls `ExecSound_ext`. `PlayGameSfx`
(`sound/sound.asm`) sends the id to the sound chip on `hAPUIO1`. The
sound program (`notes/ff4-spc.asm`) has a table of 128 sound effects at
SPC `$FD00`, 4 bytes each (two channel script pointers). Interrupt
`$02` finds the entry like this:

```
15BE: 8F 00 22  MOV   $22, #$00       ; $FD00 (pointers to game sound effects)
15C1: 8F FD 23  MOV   $23, #$FD
15C4: E4 F5     MOV   A, $F5          ; sound effect id
15C6: 8D 04     MOV   Y, #$04
15C8: CF        MUL   YA
15C9: 7A 22     ADDW  YA, $22
```

The sound program is part of the ROM, and `InitSound` uploads it to
the sound chip at reset. The code changes the `#$00` to `#$04`, so
the table starts at `$FD04`, and sound effect `n` plays sound effect
`n+1`. The optional codes use `#$08` (`n+2`) or `#$10` (`n+4`). Some
of them (ids from the `PlaySfx` calls in `field/`):

| Sound | ID | Now plays |
|---|---|---|
| Door opening (`OpenDoor`) | `$45` | `$46` |
| Battle starting on the world map (`BattleWorld`) | `$3F` | `$40` |
| Opening a treasure chest | `$30` | `$31` |
| Special effects such as the bomb ring | `$23`, `$51` | `$24`, `$52` |

Battle sound effects go through the same table and shift too. Music
and the system sound effects (interrupts `$10`-`$1F`, table at SPC
`$18E0`) use other tables and are not changed.

**Evidence:** `ff2_original.smc` has `8F 00 22 8F FD 23 E4 F5` once in
the ROM, at `$04:9445`. That is SPC `$15BE` in the first upload block
of `SPCData` (`$04:8683`), which loads `$11A9` bytes at SPC `$0800`
starting from `$04:8687`. The block for SPC `$FD00` (`$0200` bytes, at
`$04:B9CC`) has 128 entries, all pointing into the sound effect data at
`$B300`-`$CA6F`. Each code decodes back to its raw form with
`snes_gg_encode.py`. Tested in Mesen2.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The optional codes were not tested.
- The table only gives which id plays, not what each sound is, so the
  "Now plays" column has ids, not names.
- The last id (`$7F`, or the last 2 or 4 with the optional codes)
  reads past the table into the sample frequency multipliers at SPC
  `$FF00` (`E1 E1 E1 E1`). That gives a channel script pointer of
  `$E1E1`, inside the echo buffer, so that sound plays garbage and may
  hang the sound program. It's not known which events use these ids.
- Sound effect `$00` is empty. With the code, the effect that plays
  `$01` plays `$02`, and nothing plays `$00` any more.

## Every song is the one after it in the song list

**Codes:** `4DAD-DDFD` + `4EAD-DFBD` (use both)
**Raw form:** `04C000:20` (orig `$1D`) + `04C006:2F` (orig `$0F`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, instead, 2 places later:** `47AD-DDFD` + `0EAD-DFBD`
(`04C000:23` + `04C006:4F`)
**Recorded:** 2026-10-29 00:34:56

Tags: `demo-ready` `curated` `play-start`

**Mechanism:** This is the music version of "Every sound effect is the
one after it in the sound effect list". Every part of the game plays a
song by putting `$01` in `$1e00` and the song id in `$1e01` and calling
`ExecSound_ext`, like `PlayMapSong` in `field/field.asm`. `PlaySong`
(`sound/sound.asm`) uses two tables, and finds each one through a
3-byte offset at the start of `SoundData` (`$04:C000`):

```
SongScriptPtrsOffset:
        .faraddr SongScriptPtrs - SoundData

BRRSamplePtrsOffset:
        .faraddr BRRSamplePtrs - SoundData

SongSamplesOffset:
        .faraddr SongSamples - SoundData
```

It multiplies the song id by 3 to read the song script pointer from
`SongScriptPtrs`, and by 32 to read the song's list of instrument
samples from `SongSamples`. The first code adds 3 to the low byte of
`SongScriptPtrsOffset` (`$1D` to `$20`), and the second adds 32 to the
low byte of `SongSamplesOffset` (`$0F` to `$2F`). Song `n` now plays
song `n+1`, with song `n+1`'s instruments. The optional codes add 6
and 64 (`n+2`).

Some of them (ids from `WorldSongTbl` and `VehicleSongTbl` in
`field/field.asm`, and the title screen):

| Song | ID | Now plays |
|---|---|---|
| Overworld, and the hovercraft | `$0D` | `$0E`, the Big Whale song |
| Chocobo | `$04` | `$05`, the black chocobo song |
| Underground | `$06` | `$07` |
| Enterprise and Falcon | `$18` | `$19` |
| Big Whale | `$0E` | `$0F` |
| Moon | `$31` | `$32` |
| The prelude (title screen) | `$15` | `$16` |

Battle songs and the songs that event scripts play go through the same
`PlaySong` and shift too. Sound effects use another table and don't
change.

**Evidence:** `ff2_original.smc` has `1D 32 01 42 09 00 0F 00 00` at
`$04:C000`: `SongScriptPtrsOffset` = `$01321D`, `BRRSamplePtrsOffset`
= `$000942` and `SongSamplesOffset` = `$00000F`. That matches the
`ff4j` ROM map (song pointers at `$06:F21D`, 70 songs of 32 bytes of
samples at `$04:C00F`). `PlaySong` is the only code that reads the two
offsets. Adding 3 and 32 doesn't carry into the next byte. The 70
pointers were read from `$06:F21D`. Each code decodes back to its raw
form with `snes_gg_encode.py`. Tested in Mesen2.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The optional codes were not tested.
- Use both codes. With only the first, each song plays with the
  instruments of the song it replaces, so it plays the right notes
  with the wrong instruments.
- Songs `$38`-`$3D` point to the same script as song `$00`. So song
  `$37` (and `$36` with the optional codes) plays song `$00`.
- The last song (`$45`, or `$44` and `$45` with the optional codes)
  reads past the table and gets a garbage script. The upload can
  overwrite the sound program and stop all sound, or freeze the game.
  It's not known which event uses that id.

## Every song plays with the instruments of the next song

**Code:** `4EAD-DFBD`
**Raw form:** `04C006:2F` (orig `$0F`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, instead, 2 songs later:** `0EAD-DFBD` (`04C006:4F`)
**Optional, instead, 4 songs later:** `6EAD-DFBD` (`04C006:8F`)
**Recorded:** 2026-10-29 00:37:12

Tags: `demo-ready` `play-start`

**Mechanism:** This is the second code of "Every song is the one after
it in the song list", used alone. `PlaySong` (`sound/sound.asm`) finds
each song's list of instrument samples (16 two-byte sample ids, 32
bytes per song) through `SongSamplesOffset` at `$04:C006`:

```
        lda     .loword(SongSamplesOffset)
        sta     $20
        ...
        lda     $01
        longa
        asl5
        clc
        adc     $10
```

It loads those samples into the sound chip, and the song script picks
its instruments by their place in the list. The code changes the low
byte of `SongSamplesOffset` from `$0F` to `$2F`. Each song still reads
its own script, so it plays the same notes, but it loads the samples
of the next song, and each instrument becomes the sample in the same
place in that song's list. The optional codes use the list 2 songs
later (`$4F`) or 4 songs later (`$8F`).

The sample ids are the sound program's own, so each sample keeps its
own loop point and tuning (`SampleLoopStart`, `SampleFreqMult`). Only
which instrument plays which part changes. For example, the overworld
(song `$0D`, 7 samples) uses the 6 samples of the Big Whale song
(`$0E`).

**Evidence:** `ff2_original.smc` has `0F 00 00` at `$04:C006`
(`SongSamplesOffset` = `$00000F`). That matches the `ff4j` ROM map (70
songs of 32 bytes of samples at `$04:C00F`). `PlaySong` is the only
code that reads it, and adding 32 (or 64, or 128) doesn't carry into
the next byte. The sample counts were read from the table. The codes
decode back to their raw forms with `snes_gg_encode.py`. Tested in
Mesen2.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The optional codes were not tested.
- When the new list has fewer samples than the song uses (the
  overworld uses 7 and gets 6), the extra instruments have no sample of
  their own. They may be silent, or play a sample left over from an
  earlier song.
- Many short songs (`$38`-`$45`, and `$00`, `$10`, `$12`, `$2A`) have
  only 1 sample. Songs that get one of their lists play every part with
  the same instrument, or lose parts.
- The last song (`$45`, more with the optional codes) reads past the
  table into `SampleLoopStart` and gets garbage sample ids, which may
  load garbage or stop the sound.
- Don't use with "Every song is the one after it in the song list".
  That cheat already uses this code.

## Music and sound effects play half an octave higher

**Codes:** `D062-DDB4` + `D06C-04B4` (use both)
**Raw form:** `0488D2:04` (orig `$0A`) + `0489AA:04` (orig `$0A`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, instead, half an octave lower:** `FD62-DDB4` + `FD6C-04B4`
(`0488D2:10` + `0489AA:10`)
**Optional, instead, an octave higher:** `E362-DDB4` + `E36C-04B4`
(`0488D2:FE` + `0489AA:FE`)
**Optional, instead, an octave lower:** `F162-DDB4` + `F16C-04B4`
(`0488D2:16` + `0489AA:16`)
**Recorded:** 2026-10-29 00:42:37

Tags: `demo-ready` `play-start`

**Mechanism:** When a channel plays a note, the sound program
(`notes/ff4-spc.asm`) turns the octave and note into a frequency. It
takes octave * 12 + note, subtracts 10, and splits the result into an
octave and one of the 12 frequency values at SPC `$1877`:

```
0A3E: F5 C0 02  MOV   A, $02C0+X      ; octave
0A41: 8D 0C     MOV   Y, #$0C
0A43: CF        MUL   YA
0A44: 60        CLRC
0A45: 84 26     ADC   A, $26
0A47: C4 8E     MOV   $8E, A
0A49: 80        SETC
0A4A: A8 0A     SBC   A, #$0A
...
0A5D: F6 77 18  MOV   A, $1877+Y      ; frequency value
```

The same steps are at `$0B22` for a song channel's pitch envelope (the
note the pitch slides to). Each step less in the `#$0A` makes every
note one semitone higher. The codes change both to `#$04`, 6 semitones
higher. The optional codes use `#$10` (6 lower), `#$FE` (12 higher) or
`#$16` (12 lower).

This code is shared: song channels fall through to `$0A3B`, and sound
effect channels branch to it (`09F3: BCS $0A3B ; branch if a sound
effect`). No byte changes only the music, so the sound effects change
too. Use it with "Sound effects play half an octave lower" to move only
the music up.

**Evidence:** `ff2_original.smc` has `A8 0A` at `$04:88D1` and at
`$04:89A9`. These are SPC `$0A4A` and `$0B22` in the first `SPCData`
upload block, which loads SPC `$0800` from `$04:8687`. No other
`SBC A,#$0A` in the sound program feeds the frequency table. Each code
decodes back to its raw form with `snes_gg_encode.py`. The
octave-higher codes were tested in Mesen2.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only the octave-higher codes were tested.
- Use both codes. With only the first, song notes with a pitch envelope
  slide from the new pitch back toward the old one.
- Going up, the highest notes can pass the DSP's largest pitch and
  wrap around to a wrong note. The octave-higher codes have the most
  of these.
- Going down an octave, notes below octave 1 have a value under -12,
  which the code doesn't handle, so they play a wrong note. Few songs
  go that low.
- Sound effects that use the noise generator keep their pitch.

## Music and sound effects play with 5 of the 12 notes out of place

**Codes:** `D8BD-AD9F` + `DABD-AF2F` + `D6BD-A42F` + `DBBD-A72F` +
`DABF-AF9F` (use all 5)
**Raw form:** `049701:0B` (orig `$08`) + `049707:0C` (orig `$0A`) +
`04970B:08` (orig `$0B`) + `04970F:09` (orig `$0D`) + `049715:0C`
(orig `$10`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-29 00:51:01

Tags: `demo-ready` `play-start`

**Mechanism:** The sound program (`notes/ff4-spc.asm`) gets the pitch
of each note from a table of 12 frequency values at SPC `$1877`, one
per semitone, and doubles or halves it for the octave (see "Music and
sound effects play half an octave higher"). The table is read at
`$0A5D` for a note and at `$0B37` for a pitch envelope:

```
1877: 0879 08FA 0983 0A14 0AAD 0B50 0BFC 0CB2
      0D74 0E41 0F1A 1000
```

The codes change the high byte of 5 of the values, so those notes play
at the wrong pitch in every octave and the other 7 stay put:

| Value | Was | Now | Change |
|---|---|---|---|
| 2 | `$08FA` | `$0BFA` | 5 semitones up |
| 5 | `$0AAD` | `$0CAD` | 3 semitones up |
| 7 | `$0BFC` | `$08FC` | 5 semitones down |
| 9 | `$0D74` | `$0974` | 6 semitones down |
| 12 | `$1000` | `$0C00` | 5 semitones down |

Moving every note by the same amount keeps a tune easy to recognize.
This moves only some of them, by different amounts, so melodies jump
around and chords clash. The 5 are spread over the scale, so every key
gets some wrong notes. No new value is above `$1000`, so this adds no
notes past the DSP's largest pitch.

**Evidence:** `ff2_original.smc` has
`79 08 FA 08 83 09 14 0A AD 0A 50 0B FC 0B B2 0C 74 0D 41 0E 1A 0F 00
10` at `$04:96FE`, and nowhere else. That is SPC `$1877` in the first
`SPCData` upload block, which loads SPC `$0800` from `$04:8687`, and
it matches the table in `notes/ff4-spc.asm`. Each code decodes back to
its raw form with `snes_gg_encode.py`. Tested in Mesen2.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Sound effects use the same table, so some of them change pitch too.

## Music and sound effects play one semitone higher

**Codes:** `DB62-DDB4` + `DB6C-04B4` (use both)
**Raw form:** `0488D2:09` (orig `$0A`) + `0489AA:09` (orig `$0A`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, instead, one semitone lower:** `D862-DDB4` + `D86C-04B4`
(`0488D2:0B` + `0489AA:0B`)
**Recorded:** 2026-10-29 00:55:02

Tags: `demo-ready` `play-start`

**Mechanism:** The same as "Music and sound effects play half an octave
higher", but with a shift of 1 semitone instead of 6. The sound program
subtracts `#$0A` from octave * 12 + note before it looks up the
frequency, at SPC `$0A4A` for a note and `$0B22` for a pitch
envelope. The codes change both to `#$09`, so every note is one
semitone higher. The optional codes use `#$0B`, one semitone lower.

**Evidence:** `ff2_original.smc` has `A8 0A` at `$04:88D1` and at
`$04:89A9` (SPC `$0A4A` and `$0B22`), the same bytes as the
half-octave entry. Each code decodes back to its raw form with
`snes_gg_encode.py`. Tested in Mesen2.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The optional codes were not tested.
- Use both codes. With only the first, song notes with a pitch envelope
  slide from the new pitch back toward the old one.
- No byte changes only the music, so sound effects move one semitone
  too.
- A one-semitone shift is small. Many players won't notice it unless
  they know the songs well.
- These write the same 2 bytes as the half-octave cheat. Use only one.

## Music and sound effects play a whole tone higher

**Codes:** `D662-DDB4` + `D66C-04B4` (use both)
**Raw form:** `0488D2:08` (orig `$0A`) + `0489AA:08` (orig `$0A`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, instead, a whole tone lower:** `DA62-DDB4` + `DA6C-04B4`
(`0488D2:0C` + `0489AA:0C`)
**Recorded:** 2026-10-29 00:57:08

Tags: `demo-ready` `play-start`

**Mechanism:** The same as "Music and sound effects play half an octave
higher", but with a shift of 2 semitones instead of 6. The sound program
subtracts `#$0A` from octave * 12 + note before it looks up the
frequency, at SPC `$0A4A` for a note and `$0B22` for a pitch
envelope. The codes change both to `#$08`, so every note is 2
semitones higher. The optional codes use `#$0C`, 2 semitones lower.

**Evidence:** `ff2_original.smc` has `A8 0A` at `$04:88D1` and at
`$04:89A9` (SPC `$0A4A` and `$0B22`), the same bytes as the
half-octave entry. Each code decodes back to its raw form with
`snes_gg_encode.py`. Tested in Mesen2.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The optional codes were not tested.
- Use both codes. With only the first, song notes with a pitch envelope
  slide from the new pitch back toward the old one.
- No byte changes only the music, so sound effects move a whole tone
  too.
- These write the same 2 bytes as the half-octave and one-semitone
  cheats. Use only one.

## Music and sound effects play 2 whole tones higher

**Codes:** `D162-DDB4` + `D16C-04B4` (use both)
**Raw form:** `0488D2:06` (orig `$0A`) + `0489AA:06` (orig `$0A`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Optional, instead, 2 whole tones lower:** `D362-DDB4` + `D36C-04B4`
(`0488D2:0E` + `0489AA:0E`)
**Recorded:** 2026-10-29 00:58:51

Tags: `demo-ready` `play-start`

**Mechanism:** The same as "Music and sound effects play half an octave
higher", but with a shift of 4 semitones instead of 6. The sound program
subtracts `#$0A` from octave * 12 + note before it looks up the
frequency, at SPC `$0A4A` for a note and `$0B22` for a pitch
envelope. The codes change both to `#$06`, so every note is 4
semitones higher. The optional codes use `#$0E`, 4 semitones lower.

**Evidence:** `ff2_original.smc` has `A8 0A` at `$04:88D1` and at
`$04:89A9` (SPC `$0A4A` and `$0B22`), the same bytes as the
half-octave entry. Each code decodes back to its raw form with
`snes_gg_encode.py`. Tested in Mesen2.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The optional codes were not tested.
- Use both codes. With only the first, song notes with a pitch envelope
  slide from the new pitch back toward the old one.
- No byte changes only the music, so sound effects move 2 whole tones
  too.
- Going up, the highest notes can pass the DSP's largest pitch and
  wrap around to a wrong note.
- These write the same 2 bytes as the other music and sound effect
  pitch cheats. Use only one.
- For 3 whole tones, use "Music and sound effects play half an octave
  higher". 3 whole tones are 6 semitones, which is half an octave.

## Start a new game with 99 AuApple

**Codes:** `2E6E-6DDD` + `176E-646D` (use both)
**Raw form:** `0082F0:DF` (orig `$FE`) + `0082FA:63` (orig `$01`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-31 12:44:59

Tags: `demo-ready`

**Mechanism:** AuApple (item `$DF`) is a Japanese item used from the
field menu. Its Japanese name is きんのリンゴ (golden apple). The USA
release renamed it `Dummy`, but the menu's item jump table
(`ItemEffect_tbl`, `menu/item.asm`) still points it at its handler,
`ItemEffect_df`. That handler adds 100 to a character's max HP (up to
9999). The codes put 99 of it in the new-game Sort slot, as in "Start a
new game with 99 of an unused attack item". This was split out of
"Start a new game with 99 of an unused field item".

```
ItemEffect_df:
@a759:  longa
        ldy     #$0009
        lda     ($60),y
        clc
        adc     #100         ; add 100 to max hp
```

**Evidence:** The new-game bytes at `$00:82EF` and the name table are as
in "Start a new game with 99 of an unused attack item". The USA item
jump table at `$01:A654` points `$DF` to `$01:A731` (`... 18 69 64 00`).
Both codes decode back to their raw forms with `snes_gg_encode.py`.
Tested in Mesen2 together with the 3 codes `B864-D0D6` + `D164-D006` +
`3B6A-D16C` of "Skip the opening and start outside Baron".

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- The item shows as `Dummy` in the menu.
- The English name is not in any ROM.
- The item can't be used in battle. The battle code skips items `$DE`
  and up.
- The Sort entry is gone, so the item menu can't sort. The TrashCan's
  quantity becomes 99 too.
- Only a new game changes. Uses the same bytes as the other new-game
  item entries; use only one of them.

## Start a new game with 99 SomaDrop

**Codes:** `3F6E-6DDD` + `176E-646D` (use both)
**Raw form:** `0082F0:E1` (orig `$FE`) + `0082FA:63` (orig `$01`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-31 12:46:12

Tags: `demo-ready` `curated`

**Mechanism:** SomaDrop (item `$E1`) is a Japanese item used from the
field menu. Its Japanese name is ソーマのしずく (soma drop). The USA
release renamed it `Dummy`, but the menu's item jump table
(`ItemEffect_tbl`, `menu/item.asm`) still points it at its handler,
`ItemEffect_e1`. That handler adds 10 to a character's max MP (up to
999). The codes put 99 of it in the new-game Sort slot, as in "Start a
new game with 99 of an unused attack item". This was split out of
"Start a new game with 99 of an unused field item".

**Evidence:** The new-game bytes at `$00:82EF` and the name table are as
in "Start a new game with 99 of an unused attack item". The USA item
jump table at `$01:A654` points `$E1` to `$01:A758` (`A0 0D 00 ... 69
0A 00`). Both codes decode back to their raw forms with
`snes_gg_encode.py`. Tested in Mesen2 together with "Skip the opening
and start outside Baron" (`B864-D0D6` + `D164-D006` + `3D6A-D16C` +
`336A-D1AC`): the item list has the Package and 99 SomaDrop.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only whether the items show up was tested. Using a SomaDrop was not
  checked.
- The item shows as `Dummy` in the menu.
- The English name is not in any ROM.
- The item can't be used in battle. The battle code skips items `$DE`
  and up.
- The Sort entry is gone, so the item menu can't sort. The TrashCan's
  quantity becomes 99 too.
- Only a new game changes. Uses the same bytes as the other new-game
  item entries; use only one of them.
