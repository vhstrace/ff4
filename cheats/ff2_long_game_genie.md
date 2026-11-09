# Final Fantasy II (SNES) Game Genie codes: long cheats

These cheats need more than 5 codes, so they are kept apart from
`ff2_game_genie.md`. A real Game Genie can't hold them all; they are
meant for emulators that take any number of codes. Like the short codes,
they were made from the USA 1.0 release (Final Fantasy II (U) v1.0,
CRC32 `65D0A825`). SNES Game Genie codes have no compare byte; the
original byte in each raw form is documentation only. All effects come
from the disassembly. They have not been tested in play unless an entry
says so.

## Fight hits every selected target

**Codes:** (use all 64)
`DD81-A7DA` + `EE81-A70A` + `7FAD-D463` + `EEAD-D4A3` + `C2ED-ADDE` +
`A9ED-AD0E` + `70ED-AD6E` + `EDED-ADAE` + `D7ED-AFDE` + `DCED-AF0E` +
`BDED-AF6E` + `E2ED-AFAE` + `62ED-A4DE` + `5BED-A40E` + `4AED-A46E` +
`06ED-A4AE` + `4DED-A7DE` + `6BED-A70E` + `A0ED-A76E` + `16ED-A7AE` +
`EDEF-ADDE` + `F3EF-AD0E` + `31EF-AD6E` + `A3EF-ADAE` + `DCEF-AFDE` +
`BDEF-AF0E` + `E8EF-AF6E` + `62EF-AFAE` + `5BEF-A4DE` + `4AEF-A40E` +
`06EF-A46E` + `C9EF-A4AE` + `A3EF-A7DE` + `4DEF-A70E` + `17EF-A76E` +
`8FEF-A7AE` + `CDE4-ADDE` + `5EE4-AD0E` + `DDE4-AD6E` + `8FE4-ADAE` +
`6DE4-AFDE` + `BBE4-AF0E` + `DDE4-AF6E` + `45E4-AFAE` + `66E4-A4DE` +
`FDE4-A40E` + `E6E4-A46E` + `4DE4-A4AE` + `E8E4-A7DE` + `A0E4-A70E` +
`6DE4-A76E` + `2EE4-A7AE` + `1DE7-ADDE` + `C2E7-AD0E` + `5BE7-AD6E` +
`4AE7-ADAE` + `2DE7-AFDE` + `D7E7-AF0E` + `0AE7-AF6E` + `13E7-AFAE` +
`ACE7-A4DE` + `58E7-A40E` + `1DE7-A46E` + `DDCE-0708`
**Raw form:** `03B36C:00` (orig `$89`) + `03B36D:FF` (orig `$C4`) +
`03C80A:31` (orig `$6E`) + `03C80B:FF` (orig `$CA`), then 59 bytes at
`03FF00`-`03FF3A` (orig all `$FF`), then `02ADFD:00` (orig `$04`):

```
03FF00:AD 03FF01:C5 03FF02:34 03FF03:F0 03FF04:03 03FF05:0A 03FF06:90
03FF07:FD 03FF08:8D 03FF09:79 03FF0A:2C 03FF0B:48 03FF0C:20 03FF0D:89
03FF0E:C4 03FF0F:68 03FF10:F0 03FF11:1E 03FF12:E6 03FF13:CE 03FF14:0A
03FF15:90 03FF16:FB 03FF17:8D 03FF18:79 03FF19:2C 03FF1A:48 03FF1B:A5
03FF1C:CE 03FF1D:20 03FF1E:63 03FF1F:B1 03FF20:A0 03FF21:7F 03FF22:00
03FF23:B1 03FF24:80 03FF25:99 03FF26:00 03FF27:27 03FF28:88 03FF29:10
03FF2A:F8 03FF2B:20 03FF2C:FB 03FF2D:C4 03FF2E:80 03FF2F:DF 03FF30:60
03FF31:AD 03FF32:79 03FF33:2C 03FF34:D0 03FF35:03 03FF36:4C 03FF37:6E
03FF38:CA 03FF39:7B 03FF3A:60
```

(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 15:50:27

**Mechanism:** Normally Fight hits only the first selected target. When
`DoAttack` (`battle/attack.asm`) runs an action, it puts all the targets
in `$34c5` (`SetTargets`), sets `$ce` to the first one, and copies that
target's properties to `$2700`. `DoFightCmd` (`battle/fight.asm`) then
works only on `$ce`. It adds the damage to the damage buffer at `$34d4`
and calls `ApplyDmg`, which applies the whole buffer:

```
@c7fb:  lda     $cd
        bpl     @c809       ; branch if character attacker
        ldx     $2896
        lda     $33c2,x
        cmp     #$f8
        bne     @c80c       ; branch if no battle message ???
@c809:  jsr     ApplyDmg
```

The cheat has three parts:

1. The last code is `DDCE-0708` from `ff2_game_genie.md` ("Any command
   can target all enemies or all allies"). It lets you select every
   monster or every character with Fight.
2. 59 codes write a new routine into unused bytes at `$03:FF00`. The
   game fills unused space in bank `$03` with `$FF` (`fill = yes,
   fillval = $ff` in `ff4-en.lnk`). The routine is:

   ```
   FF00:  lda $34c5        ; all targets
          beq FF08
   FF05:  asl              ; drop bits up to the first target
          bcc FF05
   FF08:  sta $2c79        ; nonzero: more targets to come
          pha
          jsr $c489        ; DoFightCmd on the first target
   FF0F:  pla
          beq FF30         ; done if no targets are left
   FF12:  inc $ce          ; next slot
          asl
          bcc FF12         ; until a selected one
          sta $2c79
          pha
          lda $ce
          jsr $b163        ; GetObjPtr
          ldy #$007f
   FF23:  lda ($80),y      ; copy target properties to $2700
          sta $2700,y
          dey
          bpl FF23
          jsr $c4fb        ; DoFightCmd after its setup
          bra FF0F
   FF30:  rts

   FF31:  lda $2c79        ; ApplyDmg, or skip it if more targets
          bne FF39         ;   are to come
          jmp $ca6e        ; ApplyDmg
   FF39:  tdc              ; clear A, as ApplyDmg does
          rts
   ```

   The bits of `$34c5` are in the same order `DoAttack` uses to find
   `$ce` (bit 7 is slot 0), so `inc $ce` and `asl` step through the
   slots together. The copy loop is the same one `DoAttack` uses for the
   first target. The first target gets the whole `DoFightCmd`; later
   targets enter at `$c4fb`, after the arrow use and the graphics script
   setup, so those happen once per attack.
3. Four codes connect it. `CmdTbl` entry `$00` changes from `Cmd_00`
   (`$C489`) to `$FF00`, so the Fight command runs the new routine.
   The `jsr ApplyDmg` at `$03:C809` changes to `jsr $FF31`.

`ApplyDmg` doesn't clear the damage buffer, so calling it once per
target would apply the first target's damage again for each later
target. `$FF31` skips it while `$2c79` says more targets are left. The
last target's pass has `$2c79` = 0, so `ApplyDmg` runs once and applies
the damage for every target. `$2c79` is otherwise only cleared at the
start of each action (`stz $2c79` in `DoAttack`) and never read. Focus,
Aim and Jump call `DoFightCmd` directly (`jsr $c489` at `$03:E80E`,
`$03:E831` and `$03:E98D`), not through `CmdTbl`. They see `$2c79` = 0,
so they don't change.

**Evidence:** The `fight.asm` labels match the USA ROM; the `attack.asm`
and `damage.asm` labels used here match too. `ff2_original.smc` has:

- `89 C4` (`Cmd_00`) at `$03:B36C`, the first entry of `CmdTbl`.
- `9C C7 34` (`stz $34c7`) at `$03:C489` and `64 C7 9C FC 38` (`stz $c7`,
  `stz $38fc`) at `$03:C4FB`.
- `20 6E CA` (`jsr ApplyDmg`) at `$03:C809`, and `C2 20 7B AA`
  (`longa`, `clr_ax`) at `$03:CA6E`.
- `48 29 7F 85 DF` (`GetObjPtr`) at `$03:B163`.
- `A0 7F 00 B1 80 99 00 27` (the copy loop) at `$03:AE66` and
  `$03:AFDB`.
- `$FF` in every byte from `$03:FEE3` to `$03:FFFF`.
- `79 2C` in bank `$03` only at `$03:AD9F` (`stz $2c79`).
- `20 89 C4` at `$03:E80E`, `$03:E831` and `$03:E98D` only.

`DoAttack` runs with an 8-bit A and 16-bit X and Y (`ldy #$007f` is
`A0 7F 00`), and the routine is written for that. Every code decodes
back to its raw form with `snes_gg_encode.py`, and the 59 new bytes
were checked with a disassembler.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Monsters' Fight also goes through the new routine, but monsters
  usually pick one target.
- A selected target that died earlier in the battle is still in
  `$34c5`. `ApplyDmg` skips targets that are no longer present
  (`$3540`), but the attack may still show on that slot.
- Status effects from the weapon are checked for each target before
  any damage is applied, so a target that dies from this attack can
  still get the status.
- The attack animation and damage numbers come from the graphics code,
  which was not checked for a Fight with several targets.
- Each later target gets the full damage, not a split share.

## Every battle has 8 enemies

**Codes:** (use all 62)
`4D6A-0FAA` + `5D6A-04DA` + `EE6A-040A` + `3C6A-046A` + `C4E5-ADDE` +
`D4E5-AD0E` + `DDE5-AD6E` + `82E5-ADAE` + `ACE5-AFDE` + `4BE5-AF0E` +
`2DE5-AF6E` + `D9E5-AFAE` + `ACE5-A4DE` + `FDE5-A40E` + `E6E5-A46E` +
`6DE5-A4AE` + `46E5-A7DE` + `CDE5-A70E` + `D5E5-A76E` + `DDE5-A7AE` +
`8BE6-ADDE` + `89E6-AD0E` + `4BE6-AD6E` + `ABE6-ADAE` + `EEE6-AFDE` +
`2DE6-AF0E` + `F8E6-AF6E` + `6CE6-AFAE` + `BBE6-A4DE` + `89E6-A40E` +
`4BE6-A46E` + `BBE6-A4AE` + `82E6-A7DE` + `4BE6-A70E` + `E3E6-A76E` +
`ACE6-A7AE` + `4BEB-ADDE` + `8BEB-AD0E` + `C9EB-AD6E` + `4BEB-ADAE` +
`ABEB-AFDE` + `EEEB-AF0E` + `2DEB-AF6E` + `DCEB-AFAE` + `2CEB-A4DE` +
`88EB-A40E` + `8EEB-A46E` + `5DEB-A4AE` + `BDEB-A7DE` + `D3EB-A70E` +
`ECEB-A76E` + `BBEB-A7AE` + `C9EC-ADDE` + `4BEC-AD0E` + `66EC-AD6E` +
`FDEC-ADAE` + `28EC-AFDE` + `F6EC-AF0E` + `C2EC-AF6E` + `ACEC-AFAE` +
`4BEC-A4DE` + `1DEC-A40E`
**Raw form:** `0381C7:20` (orig `$18`) + `0381C8:70` (orig `$AD`) +
`0381C9:FF` (orig `$CA`) + `0381CA:EA` (orig `$29`), then 58 bytes
at `03FF70`-`03FFA9` (orig all `$FF`):

```
03FF70:A2 03FF71:02 03FF72:00 03FF73:BD 03FF74:CA 03FF75:29 03FF76:D0
03FF77:05 03FF78:CA 03FF79:10 03FF7A:F8 03FF7B:80 03FF7C:28 03FF7D:A0
03FF7E:07 03FF7F:00 03FF80:B9 03FF81:B5 03FF82:29 03FF83:C9 03FF84:FF
03FF85:D0 03FF86:1B 03FF87:8A 03FF88:99 03FF89:B5 03FF8A:29 03FF8B:99
03FF8C:BD 03FF8D:29 03FF8E:FE 03FF8F:CA 03FF90:29 03FF91:B9 03FF92:A5
03FF93:29 03FF94:C9 03FF95:FF 03FF96:D0 03FF97:0A 03FF98:DA 03FF99:BB
03FF9A:BF 03FF9B:70 03FF9C:90 03FF9D:0E 03FF9E:FA 03FF9F:99 03FFA0:A5
03FFA1:29 03FFA2:88 03FFA3:10 03FFA4:DB 03FFA5:18 03FFA6:AD 03FFA7:CA
03FFA8:29 03FFA9:60
```

(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:13:18

Tags: `challenge`

**Mechanism:** The battle setup in `battle/battle.asm` reads the
battle's 3 monster types and a 2-bit count for each into `$29ca`-`$29cc`.
It writes the type of each of the 8 monster slots to `$29b5` and
`$29bd` (`$FF` = empty slot), and loads 8 slot positions from
`MonsterPos` to `$29a5` (`$FF` = no position). Then it adds up the
total:

```
        clc
        lda     $29ca                   ; calculate total monster count
        adc     $29cb
        adc     $29cc
        sta     $29cd
```

The first four codes change `clc / lda $29ca` to `jsr $ff70 / nop`. The
other 58 write this into unused bytes at `$03:FF70`:

```
FF70:  ldx #2           ; find the last type with a count
FF73:  lda $29ca,x
       bne FF7D
       dex
       bpl FF73
       bra FFA5
FF7D:  ldy #7           ; for each slot
FF80:  lda $29b5,y
       cmp #$ff
       bne FFA2         ; skip slots that have a monster
       txa
       sta $29b5,y      ; put the last type in the slot
       sta $29bd,y
       inc $29ca,x      ; and count it
       lda $29a5,y
       cmp #$ff
       bne FFA2
       phx
       tyx
       lda $0e9070,x    ; no position: use MonsterPos entry 14
       plx
       sta $29a5,y
FFA2:  dey
       bpl FF80
FFA5:  clc              ; the code the jsr replaced
       lda $29ca
       rts
```

The game's code then works out the new total and copies the counts to
`$38f0`. So every empty slot gets one more of the battle's last monster
type, and every battle has 8 monsters. MonsterPos entry 14 (`$0E:9070`,
`04 0B 53 5A A4 AB F3 FA`) is a layout with a position for all 8 slots.
The new monsters use its positions for the slots they fill. The
monsters that were already in the battle keep theirs.

**Evidence:** `ff2_original.smc` has `18 AD CA 29 6D CB 29 6D CC 29 8D
CD 29` at `$03:81C7`, after the `MonsterPos` load
(`BF 00 90 0E 99 A5 29` at `$03:81B9`). The slot fill at
`$03:818E` and the empty-slot reset at `$03:817E` match
`battle/battle.asm`. The code there runs with an 8-bit A and a 16-bit
X and Y. `$FF` fills `$03:FEE3`-`$03:FFFF`. The monster init loop
(`battle/init.asm` @8be4) reads all 8 slots from `$29b5`. The monster
cursor code (`btlgfx/cursor.asm`) skips to the first live monster when
a monster has no cursor link. Every code decodes back to its raw form
with `snes_gg_encode.py`, and the new bytes were checked with a
disassembler.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- It uses `$03:FF70`-`$03:FFA9`. "Fight hits every selected target"
  (`$03:FF00`-`$03:FF3A`) uses other bytes, so both can be on at once.
- Boss and story battles get 8 too, for example 8 Mist Dragons. Some of
  these may not end the normal way, or may freeze.
- The new monsters can overlap each other or the first ones, since their
  positions come from a different layout. Large monsters will overlap
  a lot.
- The new monsters have no cursor links (`MonsterCursor` is not
  changed). The target cursor may skip some of them until other
  monsters die.
- They give experience, gil and items like normal monsters.

## Any able party member covers allies who are low on HP

**Codes:** (use all 84)
`4DC6-A4DE` + `DDC6-A40E` + `E4C6-A46E` + `2CED-6DDA` + `9CED-6D0A` +
`CDED-6D6A` + `DDED-6DAA` + `DDED-6FDA` + `C4ED-6F0A` + `DDED-6F6A` +
`DDED-6FAA` + `8BED-64DA` + `0DED-640A` + `79ED-646A` + `2DED-64AA` +
`49ED-67DA` + `B6ED-670A` + `A9ED-676A` + `A3ED-67AA` + `EDEF-6DDA` +
`4DEF-6D0A` + `A9EF-6D6A` + `A2EF-6DAA` + `EDEF-6FDA` + `FAEF-6F0A` +
`82EF-6F6A` + `D7EF-6FAA` + `4DEF-64DA` + `4BEF-640A` + `ADEF-646A` +
`2DEF-64AA` + `F9EF-67DA` + `82EF-670A` + `D0EF-676A` + `4DEF-67AA` +
`4BE4-6DDA` + `7AE4-6D0A` + `2DE4-6D6A` + `D3E4-6DAA` + `82E4-6FDA` +
`D9E4-6F0A` + `4DE4-6F6A` + `4BE4-6FAA` + `9DE4-64DA` + `2DE4-640A` +
`D5E4-646A` + `82E4-64AA` + `D1E4-67DA` + `4DE4-670A` + `4BE4-676A` +
`DFE4-67AA` + `EDE7-6DDA` + `F9E7-6D0A` + `A4E7-6D6A` + `4DE7-6DAA` +
`6CE7-6FDA` + `F6E7-6F0A` + `1BE7-6F6A` + `6DE7-6FAA` + `DDE7-64DA` +
`CCE7-640A` + `34E7-646A` + `4DE7-64AA` + `58E7-67DA` + `A6E7-670A` +
`ADE7-676A` + `D9E7-67AA` + `DDE0-6DDA` + `2DE0-6D0A` + `A9E0-6D6A` +
`CBE0-6DAA` + `EEE0-6FDA` + `6DE0-6F0A` + `DFE0-6F6A` + `B6E0-6FAA` +
`62E0-64DA` + `93E0-640A` + `79E0-646A` + `5CE0-64AA` + `ECE0-67DA` +
`C2E0-670A` + `93E0-676A` + `79E0-67AA` + `1DE9-6DDA`
**Raw form:** `03AF88:20` (orig `$AD`) + `03AF89:00` (orig `$5E`) +
`03AF8A:F2` (orig `$35`), then 81 bytes at `03F200`-`03F250` (orig
all `$FF`):

```
03F200:DA 03F201:5A 03F202:A0 03F203:00 03F204:00 03F205:A2 03F206:00
03F207:00 03F208:B9 03F209:40 03F20A:35 03F20B:D0 03F20C:25 03F20D:98
03F20E:C5 03F20F:CE 03F210:F0 03F211:20 03F212:C5 03F213:CD 03F214:F0
03F215:1C 03F216:BD 03F217:03 03F218:20 03F219:29 03F21A:C0 03F21B:D0
03F21C:15 03F21D:BD 03F21E:04 03F21F:20 03F220:29 03F221:3C 03F222:D0
03F223:0E 03F224:BD 03F225:05 03F226:20 03F227:29 03F228:50 03F229:D0
03F22A:07 03F22B:BD 03F22C:06 03F22D:20 03F22E:29 03F22F:01 03F230:F0
03F231:15 03F232:C2 03F233:20 03F234:8A 03F235:18 03F236:69 03F237:80
03F238:00 03F239:AA 03F23A:E2 03F23B:20 03F23C:7B 03F23D:C8 03F23E:C0
03F23F:05 03F240:00 03F241:D0 03F242:C5 03F243:A9 03F244:FF 03F245:80
03F246:01 03F247:98 03F248:8D 03F249:5E 03F24A:35 03F24B:7A 03F24C:FA
03F24D:AD 03F24E:5E 03F24F:35 03F250:60
```

(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:26:54

**Mechanism:** When a Fight attack targets a party member who is low on
HP (critical, `$2706` bit 0) or being covered, the attack code in
`battle/attack.asm` moves the attack to the covering character. Only
Paladin Cecil can cover: `battle/init.asm` puts his slot in `$355e`
(`$FF` = none):

```
@af81:  lda     $26d1
        cmp     #$c0
        bne     @afeb       ; branch if not fight
        lda     $355e
        bmi     @afeb       ; branch if paladin cecil is not present
```

The first three codes change `lda $355e` to `jsr $f200`. The other 81
write this into unused bytes at `$03:F200`. It picks the first party
slot that is present, isn't the target or the attacker, and passes the
same status checks the game makes for Cecil (not dead, stone,
paralyzed, asleep, charmed, berserk, stopped or critical):

```
F200:  phx
       phy
       ldy #0           ; party slot
       ldx #0           ; slot * $80
F208:  lda $3540,y
       bne F232         ; slot is empty
       tya
       cmp $ce
       beq F232         ; slot is the target
       cmp $cd
       beq F232         ; slot is the attacker
       lda $2003,x
       and #$c0
       bne F232         ; dead or stone
       lda $2004,x
       and #$3c
       bne F232         ; paralyze, sleep, charm, berserk
       lda $2005,x
       and #$50
       bne F232
       lda $2006,x
       and #$01
       beq F247         ; not critical: use this slot
F232:  rep #$20
       txa
       clc
       adc #$0080
       tax
       sep #$20
       tdc
       iny
       cpy #5
       bne F208
       lda #$ff         ; no one can cover
       bra F248
F247:  tya
F248:  sta $355e        ; covering slot
       ply
       plx
       lda $355e        ; the load the jsr replaced
       rts
```

The game's code then makes its usual checks and moves the attack to
that slot.

**Evidence:** `ff2_original.smc` has `AD D1 26 C9 C0 D0 63 AD 5E 35 30
5E` at `$03:AF81`, and `AD D1 26 C9 C0 D0` occurs only once. Party
slot status is at `$2000 + slot * $80`, as `GetFrontRowChars`
(`battle/ai.asm`) reads it, and `$3540,slot` is 0 when the slot is
present. The cover code reads `$355e` with `ldx` (16-bit), so
`$355f` must stay 0; the routine writes only `$355e`. `$FF` fills
`$03:F1FB`-`$03:F27F`, between the end of the `battle_code`
segment and `btlgfx_code2`, which `ff4-en.lnk` starts at `$03:F280`.
Every code decodes back to its raw form with `snes_gg_encode.py`, and
the new bytes were checked with a disassembler.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Only Fight attacks are covered, and only for a target who is low on
  HP or is the target of Cecil's Cover command.
- The cover animation (`UpdateHitCharPose`, `btlgfx/weapon.asm`) was
  made for Cecil. It hasn't been checked for other characters and may
  look wrong.
- The first able slot always covers, so slot 1 covers most often.
- It uses `$03:F200`-`$03:F250`, which no other code here uses.
- "Attacks that Cecil covers always miss" works with this too, so then
  every covered attack misses.

## Every character has the Jump command

**Codes:** (use all 17)
`D1E9-09AE` + `D1E1-01DE` + `D1E1-050E` + `D1E5-006E` + `D1E5-096E` +
`D1E5-05DE` + `D1E6-000E` + `D1E6-096E` + `D1E6-01AE` + `D1EB-00DE` +
`D1EB-09DE` + `D1EB-010E` + `D1EC-09DE` + `D1EC-010E` + `D1E8-00AE` +
`D1E8-016E` + `D1E8-05AE`
**Raw form:** `$06` in place of one `$FF` in each of 17 class
command lists. The class each code changes:

- `D1E9-09AE`: `13FD57:06` (orig `$FF`), Dark Knight Cecil, class `$01`
- `D1E1-01DE`: `13FD68:06` (orig `$FF`), Tellah, class `$04`
- `D1E1-050E`: `13FD6D:06` (orig `$FF`), Edward, class `$05`
- `D1E5-006E`: `13FD72:06` (orig `$FF`), Rosa, class `$06`
- `D1E5-096E`: `13FD76:06` (orig `$FF`), Yang, class `$07`
- `D1E5-05DE`: `13FD7C:06` (orig `$FF`), Palom, class `$08`
- `D1E6-000E`: `13FD81:06` (orig `$FF`), Porom, class `$09`
- `D1E6-096E`: `13FD86:06` (orig `$FF`), Tellah (2nd time), class `$0A`
- `D1E6-01AE`: `13FD8B:06` (orig `$FF`), Paladin Cecil, class `$0B`
- `D1EB-00DE`: `13FD90:06` (orig `$FF`), Tellah (3rd time), class `$0C`
- `D1EB-09DE`: `13FD94:06` (orig `$FF`), Yang (2nd time), class `$0D`
- `D1EB-010E`: `13FD99:06` (orig `$FF`), Cid, class `$0E`
- `D1EC-09DE`: `13FDA4:06` (orig `$FF`), Rosa (2nd time), class `$10`
- `D1EC-010E`: `13FDA9:06` (orig `$FF`), adult Rydia, class `$11`
- `D1E8-00AE`: `13FDB3:06` (orig `$FF`), FuSoYa, class `$13`
- `D1E8-016E`: `13FDBA:06` (orig `$FF`), Golbez, class `$15`
- `D1E8-05AE`: `13FDBF:06` (orig `$FF`), Anna, class `$16`

(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:28:13

**Mechanism:** At the start of each battle, `battle/init.asm` copies 5
command ids per character from `ClassCmd` (`$13:FD55`, 5 bytes per
class, `$FF` = empty) to the battle command list at `$3303`:

```
@8b99:  lda     f:ClassCmd,x            ; battle commands for each class
        sta     $3303,y
        cmp     #$ff
        bne     @8ba7
```

Each code puts `$06`, Jump (`Cmd_06` in `battle/cmd.asm`), in the first
empty slot of one class's list. The three Kain classes (`$02`, `$0F`,
`$14`) already have Jump. Child Rydia (`$03`) and Edge (`$12`) have
no empty slot, so they don't get it. Each code works on its own; the
first one alone gives Dark Knight Cecil Jump from the first battle.

**Evidence:** `ff2_original.smc` has `BF 55 FD 13 99 03 33` (`lda
$13fd55,x / sta $3303,y`) at `$03:8B99`. The 22 lists at `$13:FD55`
match the classes' commands (Kain's `00 06 01 FF FF`, Edge's `00 16 17
18 01`) and the class ids in the `ff4j` RAM map. Entry `$06` of the
command table at `$03:B36C` is `$E93F`. Every code decodes back to its
raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Characters other than Kain may not have jump graphics, so the jump
  may look wrong.
- Jump does weapon damage, so a character with a weak weapon does
  little damage.
- The Paladin Cecil code (`13FD8B`) uses the same byte as "Paladin
  Cecil has the Dark Wave command"; use only one of them.

## Every character can equip any weapon

**Codes:** (use all 41)
`44A9-6760` + `5DA9-67A0` + `E2A1-6DD0` + `DDA1-6D00` + `8EE5-0DD7` +
`D1E5-0D07` + `BFE5-0D67` + `DEE5-0DA7` + `06E5-0FD7` + `2CE5-0F07` +
`C4E5-0F67` + `DDE5-0FA7` + `DDE5-04D7` + `C2E5-0407` + `7BE5-0467` +
`F8E5-04A7` + `ABE5-07D7` + `1DE5-0707` + `BDE5-0767` + `D1E5-07A7` +
`36E6-0DD7` + `ABE6-0D07` + `BAE6-0D67` + `BDE6-0DA7` + `DFE6-0FD7` +
`36E6-0F07` + `8EE6-0F67` + `CDE6-0FA7` + `E2E6-04D7` + `DDE6-0407` +
`7DE6-0467` + `D9E6-04A7` + `ECE6-07D7` + `16E6-0707` + `CBE6-0767` +
`DDE6-07A7` + `18EB-0DD7` + `ECEB-0D07` + `16EB-0D67` + `18EB-0DA7` +
`DDEC-0DD7`
**Raw form:** `01C25E:22` (orig `$BF`) + `01C25F:70` (orig `$06`) +
`01C260:FD` (orig `$91`) + `01C261:00` (orig `$0F`), then 36 bytes
at `00FD70`-`00FD93` (orig all `$FF`), then `00FDA0:00` (orig `$FF`):

```
00FD70:BF 00FD71:06 00FD72:91 00FD73:0F 00FD74:48 00FD75:DA 00FD76:A2
00FD77:00 00FD78:00 00FD79:AD 00FD7A:39 00FD7B:1B 00FD7C:C9 00FD7D:60
00FD7E:90 00FD7F:06 00FD80:E8 00FD81:C9 00FD82:9C 00FD83:90 00FD84:01
00FD85:E8 00FD86:BF 00FD87:A0 00FD88:FD 00FD89:00 00FD8A:30 00FD8B:05
00FD8C:FA 00FD8D:68 00FD8E:A9 00FD8F:00 00FD90:6B 00FD91:FA 00FD92:68
00FD93:6B
```

(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:33:28

**Optional code:** add `DDC5-076A` (`03A17E:00`, orig `$1F`) so
any character can also pick any weapon or shield from the item list in
battle.

**Mechanism:** `CheckClassEquip` in `menu/equip.asm` decides if the
character can equip the item `$1B39`. It is used by the equip menu,
the shop and the fat chocobo. It takes the item's class group from
byte 6 of `EquipProp` and looks up a 16-bit class mask in
`ItemClasses`:

```
        lda     f:EquipProp+6,x         ; item properties, byte 6
        and     #$1f
        asl
        jsr     Tax16
        longa
        lda     f:ItemClasses,x   ; item equipability
```

The codes change the `lda f:EquipProp+6,x` into `jsl $00:FD70`, a
new routine in free space. The routine loads the same byte, then picks
an on/off switch by item id: `$00:FDA0` for weapons (`$00`-`$5F`),
`$00:FDA1` for shields and armor (`$60`-`$9B`) and `$00:FDA2`
for gauntlets and rings (`$9C`-`$AF`). A switch byte with bit 7 clear
is on. When the switch is on, the routine returns 0, so the game uses
group 0, whose mask is `$FFFF` (every class). Otherwise it returns
the item's real group. This entry turns on only the weapon switch.

The optional code changes `and #$1f` to `and #$00` in
`CheckCanUseItem` in `battle/char.asm`. It reads the same table to
grey out items in the battle item list, but only for weapons and
shields.

**Evidence:** `ff2_original.smc` has `BF 06 91 0F 29 1F 0A 20 B4 87
C2 20 BF 50 A5 0F` at `$01:C25E` and `29 1F 0A AA BF 50 A5 0F` at
`$03:A17D`. `$00:FD31`-`$00:FFBB` is all `$FF`, and no other entry uses
that space. The first two bytes of `ItemClasses`
(`$0F:A550`) are `FF FF`. A patched copy disassembles as the routine
above. Every code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The first 40 codes are the same in "Every character can equip any
  armor" and "Every character can equip any gauntlet or ring". To use
  more than one of them, enter the 40 codes once and add each entry's
  last code.
- The optional code also lets any character use any shield in battle.
- How a character looks when attacking with a weapon type that their
  class can't normally use was not checked.

## Every character can equip any armor

**Codes:** (use all 41)
`44A9-6760` + `5DA9-67A0` + `E2A1-6DD0` + `DDA1-6D00` + `8EE5-0DD7` +
`D1E5-0D07` + `BFE5-0D67` + `DEE5-0DA7` + `06E5-0FD7` + `2CE5-0F07` +
`C4E5-0F67` + `DDE5-0FA7` + `DDE5-04D7` + `C2E5-0407` + `7BE5-0467` +
`F8E5-04A7` + `ABE5-07D7` + `1DE5-0707` + `BDE5-0767` + `D1E5-07A7` +
`36E6-0DD7` + `ABE6-0D07` + `BAE6-0D67` + `BDE6-0DA7` + `DFE6-0FD7` +
`36E6-0F07` + `8EE6-0F67` + `CDE6-0FA7` + `E2E6-04D7` + `DDE6-0407` +
`7DE6-0467` + `D9E6-04A7` + `ECE6-07D7` + `16E6-0707` + `CBE6-0767` +
`DDE6-07A7` + `18EB-0DD7` + `ECEB-0D07` + `16EB-0D67` + `18EB-0DA7` +
`DDEC-0D07`
**Raw form:** `01C25E:22` (orig `$BF`) + `01C25F:70` (orig `$06`) +
`01C260:FD` (orig `$91`) + `01C261:00` (orig `$0F`), then 36 bytes
at `00FD70`-`00FD93` (orig all `$FF`), then `00FDA1:00` (orig `$FF`):

```
00FD70:BF 00FD71:06 00FD72:91 00FD73:0F 00FD74:48 00FD75:DA 00FD76:A2
00FD77:00 00FD78:00 00FD79:AD 00FD7A:39 00FD7B:1B 00FD7C:C9 00FD7D:60
00FD7E:90 00FD7F:06 00FD80:E8 00FD81:C9 00FD82:9C 00FD83:90 00FD84:01
00FD85:E8 00FD86:BF 00FD87:A0 00FD88:FD 00FD89:00 00FD8A:30 00FD8B:05
00FD8C:FA 00FD8D:68 00FD8E:A9 00FD8F:00 00FD90:6B 00FD91:FA 00FD92:68
00FD93:6B
```

(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:34:00

**Optional code:** add `DDC5-076A` (`03A17E:00`, orig `$1F`) so
any character can also pick any shield or weapon from the item list in
battle.

**Mechanism:** `CheckClassEquip` in `menu/equip.asm` decides if the
character can equip the item `$1B39`. It is used by the equip menu,
the shop and the fat chocobo. It takes the item's class group from
byte 6 of `EquipProp` and looks up a 16-bit class mask in
`ItemClasses`:

```
        lda     f:EquipProp+6,x         ; item properties, byte 6
        and     #$1f
        asl
        jsr     Tax16
        longa
        lda     f:ItemClasses,x   ; item equipability
```

The codes change the `lda f:EquipProp+6,x` into `jsl $00:FD70`, a
new routine in free space. The routine loads the same byte, then picks
an on/off switch by item id: `$00:FDA0` for weapons (`$00`-`$5F`),
`$00:FDA1` for shields and armor (`$60`-`$9B`) and `$00:FDA2`
for gauntlets and rings (`$9C`-`$AF`). A switch byte with bit 7 clear
is on. When the switch is on, the routine returns 0, so the game uses
group 0, whose mask is `$FFFF` (every class). Otherwise it returns
the item's real group. This entry turns on only the armor switch.

The optional code changes `and #$1f` to `and #$00` in
`CheckCanUseItem` in `battle/char.asm`. It reads the same table to
grey out items in the battle item list, but only for weapons and
shields.

**Evidence:** `ff2_original.smc` has `BF 06 91 0F 29 1F 0A 20 B4 87
C2 20 BF 50 A5 0F` at `$01:C25E` and `29 1F 0A AA BF 50 A5 0F` at
`$03:A17D`. `$00:FD31`-`$00:FFBB` is all `$FF`, and no other entry uses
that space. The first two bytes of `ItemClasses`
(`$0F:A550`) are `FF FF`. A patched copy disassembles as the routine
above. Every code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The first 40 codes are the same in "Every character can equip any
  weapon" and "Every character can equip any gauntlet or ring". To use
  more than one of them, enter the 40 codes once and add each entry's
  last code.
- The optional code also lets any character use any weapon in battle.
- Helmets and body armor can't be changed in battle, so the optional
  code only matters for shields.

## Every character can equip any gauntlet or ring

**Codes:** (use all 41)
`44A9-6760` + `5DA9-67A0` + `E2A1-6DD0` + `DDA1-6D00` + `8EE5-0DD7` +
`D1E5-0D07` + `BFE5-0D67` + `DEE5-0DA7` + `06E5-0FD7` + `2CE5-0F07` +
`C4E5-0F67` + `DDE5-0FA7` + `DDE5-04D7` + `C2E5-0407` + `7BE5-0467` +
`F8E5-04A7` + `ABE5-07D7` + `1DE5-0707` + `BDE5-0767` + `D1E5-07A7` +
`36E6-0DD7` + `ABE6-0D07` + `BAE6-0D67` + `BDE6-0DA7` + `DFE6-0FD7` +
`36E6-0F07` + `8EE6-0F67` + `CDE6-0FA7` + `E2E6-04D7` + `DDE6-0407` +
`7DE6-0467` + `D9E6-04A7` + `ECE6-07D7` + `16E6-0707` + `CBE6-0767` +
`DDE6-07A7` + `18EB-0DD7` + `ECEB-0D07` + `16EB-0D67` + `18EB-0DA7` +
`DDEC-0D67`
**Raw form:** `01C25E:22` (orig `$BF`) + `01C25F:70` (orig `$06`) +
`01C260:FD` (orig `$91`) + `01C261:00` (orig `$0F`), then 36 bytes
at `00FD70`-`00FD93` (orig all `$FF`), then `00FDA2:00` (orig `$FF`):

```
00FD70:BF 00FD71:06 00FD72:91 00FD73:0F 00FD74:48 00FD75:DA 00FD76:A2
00FD77:00 00FD78:00 00FD79:AD 00FD7A:39 00FD7B:1B 00FD7C:C9 00FD7D:60
00FD7E:90 00FD7F:06 00FD80:E8 00FD81:C9 00FD82:9C 00FD83:90 00FD84:01
00FD85:E8 00FD86:BF 00FD87:A0 00FD88:FD 00FD89:00 00FD8A:30 00FD8B:05
00FD8C:FA 00FD8D:68 00FD8E:A9 00FD8F:00 00FD90:6B 00FD91:FA 00FD92:68
00FD93:6B
```

(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:34:20

**Mechanism:** `CheckClassEquip` in `menu/equip.asm` decides if the
character can equip the item `$1B39`. It is used by the equip menu,
the shop and the fat chocobo. It takes the item's class group from
byte 6 of `EquipProp` and looks up a 16-bit class mask in
`ItemClasses`:

```
        lda     f:EquipProp+6,x         ; item properties, byte 6
        and     #$1f
        asl
        jsr     Tax16
        longa
        lda     f:ItemClasses,x   ; item equipability
```

The codes change the `lda f:EquipProp+6,x` into `jsl $00:FD70`, a
new routine in free space. The routine loads the same byte, then picks
an on/off switch by item id: `$00:FDA0` for weapons (`$00`-`$5F`),
`$00:FDA1` for shields and armor (`$60`-`$9B`) and `$00:FDA2`
for gauntlets and rings (`$9C`-`$AF`). A switch byte with bit 7 clear
is on. When the switch is on, the routine returns 0, so the game uses
group 0, whose mask is `$FFFF` (every class). Otherwise it returns
the item's real group. This entry turns on only the gauntlet and ring
switch. In this game, the accessory slot holds gauntlets and rings.

**Evidence:** `ff2_original.smc` has `BF 06 91 0F 29 1F 0A 20 B4 87
C2 20 BF 50 A5 0F` at `$01:C25E`. `$00:FD31`-`$00:FFBB` is all
`$FF`. Of the other entries, only "Start a new game with 99 of each
healing item" uses that space, at `$00:FD40`-`$00:FD60`. The first
two bytes of `ItemClasses` (`$0F:A550`) are `FF FF`. A patched copy
disassembles as the routine above. Every code decodes back to its raw
form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The first 40 codes are the same in "Every character can equip any
  weapon" and "Every character can equip any armor". To use more than
  one of them, enter the 40 codes once and add each entry's last code.

## Every character has the Dart command

**Codes:** (use all 20)
`F1E9-09AE` + `F1E9-050E` + `F1E1-01DE` + `F1E1-050E` + `F1E5-006E` +
`F1E5-096E` + `F1E5-05DE` + `F1E6-000E` + `F1E6-096E` + `F1E6-01AE` +
`F1EB-00DE` + `F1EB-09DE` + `F1EB-010E` + `F1EB-056E` + `F1EC-09DE` +
`F1EC-010E` + `F1E8-00AE` + `F1E8-09AE` + `F1E8-016E` + `F1E8-05AE`
**Raw form:** `$16` in place of one `$FF` in each of 20 class
command lists. The class each code changes:

- `F1E9-09AE`: `13FD57:16` (orig `$FF`), Dark Knight Cecil, class `$01`
- `F1E9-050E`: `13FD5D:16` (orig `$FF`), Kain, class `$02`
- `F1E1-01DE`: `13FD68:16` (orig `$FF`), Tellah, class `$04`
- `F1E1-050E`: `13FD6D:16` (orig `$FF`), Edward, class `$05`
- `F1E5-006E`: `13FD72:16` (orig `$FF`), Rosa, class `$06`
- `F1E5-096E`: `13FD76:16` (orig `$FF`), Yang, class `$07`
- `F1E5-05DE`: `13FD7C:16` (orig `$FF`), Palom, class `$08`
- `F1E6-000E`: `13FD81:16` (orig `$FF`), Porom, class `$09`
- `F1E6-096E`: `13FD86:16` (orig `$FF`), Tellah (2nd time), class `$0A`
- `F1E6-01AE`: `13FD8B:16` (orig `$FF`), Paladin Cecil, class `$0B`
- `F1EB-00DE`: `13FD90:16` (orig `$FF`), Tellah (3rd time), class `$0C`
- `F1EB-09DE`: `13FD94:16` (orig `$FF`), Yang (2nd time), class `$0D`
- `F1EB-010E`: `13FD99:16` (orig `$FF`), Cid, class `$0E`
- `F1EB-056E`: `13FD9E:16` (orig `$FF`), Kain (2nd time), class `$0F`
- `F1EC-09DE`: `13FDA4:16` (orig `$FF`), Rosa (2nd time), class `$10`
- `F1EC-010E`: `13FDA9:16` (orig `$FF`), adult Rydia, class `$11`
- `F1E8-00AE`: `13FDB3:16` (orig `$FF`), FuSoYa, class `$13`
- `F1E8-09AE`: `13FDB7:16` (orig `$FF`), Kain (3rd time), class `$14`
- `F1E8-016E`: `13FDBA:16` (orig `$FF`), Golbez, class `$15`
- `F1E8-05AE`: `13FDBF:16` (orig `$FF`), Anna, class `$16`


(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:37:10

**Mechanism:** At the start of each battle, `battle/init.asm` copies 5
command ids per character from `ClassCmd` (`$13:FD55`, 5 bytes per
class, `$FF` = empty) to the battle command list at `$3303`:

```
@8b99:  lda     f:ClassCmd,x            ; battle commands for each class
        sta     $3303,y
        cmp     #$ff
        bne     @8ba7
```

Each code puts `$16`, Dart (Throw in other versions, `Cmd_16` in
`battle/cmd.asm`), in the first empty slot of one class's list. Only
Edge has Dart. The battle menu opens the item list for it, and
`Cmd_16` does damage of byte 1 of the item's `EquipProp` entry times
the attacker's level (`$2682`), plus a random amount, up to 9,999. Child
Rydia (`$03`) has no empty slot, so she doesn't get it. Each code
works on its own; the first one alone gives Dark Knight Cecil Dart
from the first battle.

**Evidence:** `ff2_original.smc` has `BF 55 FD 13 99 03 33` (`lda
$13fd55,x / sta $3303,y`) at `$03:8B99` and `C9 16 F0 ?? C9 18 F0`
(the Dart and Ninja checks in the battle menu) at `$02:B9BD`. Edge's
list at `$13:FDAA` is `00 16 17 18 01` (Fight, Dart, Sneak, Ninja,
Item). Every code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Which items the list lets you throw, and whether they are used up,
  were not checked.
- The codes use the same bytes as "Every character has the Jump
  command", and the Paladin
  Cecil code uses the same byte as "Paladin Cecil has the Dark Wave
  command"; use only one of them.

## Every character has the Sneak command

**Codes:** (use all 20)
`F5E9-09AE` + `F5E9-050E` + `F5E1-01DE` + `F5E1-050E` + `F5E5-006E` +
`F5E5-096E` + `F5E5-05DE` + `F5E6-000E` + `F5E6-096E` + `F5E6-01AE` +
`F5EB-00DE` + `F5EB-09DE` + `F5EB-010E` + `F5EB-056E` + `F5EC-09DE` +
`F5EC-010E` + `F5E8-00AE` + `F5E8-09AE` + `F5E8-016E` + `F5E8-05AE`
**Raw form:** `$17` in place of one `$FF` in each of 20 class
command lists. The class each code changes:

- `F5E9-09AE`: `13FD57:17` (orig `$FF`), Dark Knight Cecil, class `$01`
- `F5E9-050E`: `13FD5D:17` (orig `$FF`), Kain, class `$02`
- `F5E1-01DE`: `13FD68:17` (orig `$FF`), Tellah, class `$04`
- `F5E1-050E`: `13FD6D:17` (orig `$FF`), Edward, class `$05`
- `F5E5-006E`: `13FD72:17` (orig `$FF`), Rosa, class `$06`
- `F5E5-096E`: `13FD76:17` (orig `$FF`), Yang, class `$07`
- `F5E5-05DE`: `13FD7C:17` (orig `$FF`), Palom, class `$08`
- `F5E6-000E`: `13FD81:17` (orig `$FF`), Porom, class `$09`
- `F5E6-096E`: `13FD86:17` (orig `$FF`), Tellah (2nd time), class `$0A`
- `F5E6-01AE`: `13FD8B:17` (orig `$FF`), Paladin Cecil, class `$0B`
- `F5EB-00DE`: `13FD90:17` (orig `$FF`), Tellah (3rd time), class `$0C`
- `F5EB-09DE`: `13FD94:17` (orig `$FF`), Yang (2nd time), class `$0D`
- `F5EB-010E`: `13FD99:17` (orig `$FF`), Cid, class `$0E`
- `F5EB-056E`: `13FD9E:17` (orig `$FF`), Kain (2nd time), class `$0F`
- `F5EC-09DE`: `13FDA4:17` (orig `$FF`), Rosa (2nd time), class `$10`
- `F5EC-010E`: `13FDA9:17` (orig `$FF`), adult Rydia, class `$11`
- `F5E8-00AE`: `13FDB3:17` (orig `$FF`), FuSoYa, class `$13`
- `F5E8-09AE`: `13FDB7:17` (orig `$FF`), Kain (3rd time), class `$14`
- `F5E8-016E`: `13FDBA:17` (orig `$FF`), Golbez, class `$15`
- `F5E8-05AE`: `13FDBF:17` (orig `$FF`), Anna, class `$16`


(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:37:46

**Mechanism:** At the start of each battle, `battle/init.asm` copies 5
command ids per character from `ClassCmd` (`$13:FD55`, 5 bytes per
class, `$FF` = empty) to the battle command list at `$3303`:

```
@8b99:  lda     f:ClassCmd,x            ; battle commands for each class
        sta     $3303,y
        cmp     #$ff
        bne     @8ba7
```

Each code puts `$17`, Sneak (Steal in other versions, `Cmd_17` in
`battle/cmd.asm`), in the first empty slot of one class's list. Only
Edge has Sneak. `Cmd_17` uses the attacker's own level, so it works
the same for anyone:

```
; steal rate is (attacker level + 50) - (monster level + 10)
; so 40% if attacker and monster are at the same level
; after a failed steal, there is a (monster level + 10) chance
; that edge takes damage of (max hp) / 16
```

Child Rydia (`$03`) has no empty slot, so she doesn't get it. Each
code works on its own; the first one alone gives Dark Knight Cecil
Sneak from the first battle.

**Evidence:** `ff2_original.smc` has `BF 55 FD 13 99 03 33` (`lda
$13fd55,x / sta $3303,y`) at `$03:8B99`, and `Cmd_17` starts at
`$03:E1B7` with `A9 17 8D C8 34` (`lda #$17 / sta $34c8`). Edge's
list at `$13:FDAA` is `00 16 17 18 01` (Fight, Dart, Sneak, Ninja,
Item). Every code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- A failed steal can hurt the character who tried it, as it does Edge.
- The codes use the same bytes as "Every character has the Jump
  command" and "Every character has the Dart command", and the Paladin
  Cecil code uses the same byte as "Paladin Cecil has the Dark Wave
  command"; use only one of them.

## Every character has the Sing command

**Codes:** (use all 19)
`D6E9-09AE` + `D6E9-050E` + `D6E1-01DE` + `D6E5-006E` + `D6E5-096E` +
`D6E5-05DE` + `D6E6-000E` + `D6E6-096E` + `D6E6-01AE` + `D6EB-00DE` +
`D6EB-09DE` + `D6EB-010E` + `D6EB-056E` + `D6EC-09DE` + `D6EC-010E` +
`D6E8-00AE` + `D6E8-09AE` + `D6E8-016E` + `D6E8-05AE`
**Raw form:** `$08` in place of one `$FF` in each of 19 class
command lists. The class each code changes:

- `D6E9-09AE`: `13FD57:08` (orig `$FF`), Dark Knight Cecil, class `$01`
- `D6E9-050E`: `13FD5D:08` (orig `$FF`), Kain, class `$02`
- `D6E1-01DE`: `13FD68:08` (orig `$FF`), Tellah, class `$04`
- `D6E5-006E`: `13FD72:08` (orig `$FF`), Rosa, class `$06`
- `D6E5-096E`: `13FD76:08` (orig `$FF`), Yang, class `$07`
- `D6E5-05DE`: `13FD7C:08` (orig `$FF`), Palom, class `$08`
- `D6E6-000E`: `13FD81:08` (orig `$FF`), Porom, class `$09`
- `D6E6-096E`: `13FD86:08` (orig `$FF`), Tellah (2nd time), class `$0A`
- `D6E6-01AE`: `13FD8B:08` (orig `$FF`), Paladin Cecil, class `$0B`
- `D6EB-00DE`: `13FD90:08` (orig `$FF`), Tellah (3rd time), class `$0C`
- `D6EB-09DE`: `13FD94:08` (orig `$FF`), Yang (2nd time), class `$0D`
- `D6EB-010E`: `13FD99:08` (orig `$FF`), Cid, class `$0E`
- `D6EB-056E`: `13FD9E:08` (orig `$FF`), Kain (2nd time), class `$0F`
- `D6EC-09DE`: `13FDA4:08` (orig `$FF`), Rosa (2nd time), class `$10`
- `D6EC-010E`: `13FDA9:08` (orig `$FF`), adult Rydia, class `$11`
- `D6E8-00AE`: `13FDB3:08` (orig `$FF`), FuSoYa, class `$13`
- `D6E8-09AE`: `13FDB7:08` (orig `$FF`), Kain (3rd time), class `$14`
- `D6E8-016E`: `13FDBA:08` (orig `$FF`), Golbez, class `$15`
- `D6E8-05AE`: `13FDBF:08` (orig `$FF`), Anna, class `$16`


(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:40:55

**Mechanism:** At the start of each battle, `battle/init.asm` copies 5
command ids per character from `ClassCmd` (`$13:FD55`, 5 bytes per
class, `$FF` = empty) to the battle command list at `$3303`:

```
@8b99:  lda     f:ClassCmd,x            ; battle commands for each class
        sta     $3303,y
        cmp     #$ff
        bne     @8ba7
```

Each code puts `$08`, Sing (`Cmd_08` in `battle/cmd.asm`), in the
first empty slot of one class's list. Only Edward has Sing. The menu
check in `battle/char.asm` turns Sing off unless the character holds a
harp:

```
; sing (must have a harp equipped)
@a2ff:  lda     $2033,x
        beq     @a30d
        cmp     #$44
        bcc     @a36e
        cmp     #$4d
        bcs     @a36e
```

Child Rydia (`$03`) and Edge (`$12`) have no empty slot, so they
don't get it. Each code works on its own; the first one alone gives
Dark Knight Cecil Sing from the first battle.

**Evidence:** `ff2_original.smc` has `BF 55 FD 13 99 03 33` (`lda
$13fd55,x / sta $3303,y`) at `$03:8B99`, and the harp check `BD 33 20
F0 ?? C9 44 90` at `$03:A2FF`. Edward's list at
`$13:FD69` is `00 08 09 01 FF` (Fight, Sing, Hide, Item). Every code
decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- Sing is greyed out until the character holds a harp (items `$44`-
  `$4C`). For a class that can't equip harps, add "Every character can
  equip any weapon".
- The codes use the same bytes as "Every character has the Jump
  command" and the other entries that add a command, and the Paladin
  Cecil code uses the same byte as "Paladin Cecil has the Dark Wave
  command"; use only one of them.

## Every character has the Hide command

**Codes:** (use all 19)
`DBE9-09AE` + `DBE9-050E` + `DBE1-01DE` + `DBE5-006E` + `DBE5-096E` +
`DBE5-05DE` + `DBE6-000E` + `DBE6-096E` + `DBE6-01AE` + `DBEB-00DE` +
`DBEB-09DE` + `DBEB-010E` + `DBEB-056E` + `DBEC-09DE` + `DBEC-010E` +
`DBE8-00AE` + `DBE8-09AE` + `DBE8-016E` + `DBE8-05AE`
**Raw form:** `$09` in place of one `$FF` in each of 19 class
command lists. The class each code changes:

- `DBE9-09AE`: `13FD57:09` (orig `$FF`), Dark Knight Cecil, class `$01`
- `DBE9-050E`: `13FD5D:09` (orig `$FF`), Kain, class `$02`
- `DBE1-01DE`: `13FD68:09` (orig `$FF`), Tellah, class `$04`
- `DBE5-006E`: `13FD72:09` (orig `$FF`), Rosa, class `$06`
- `DBE5-096E`: `13FD76:09` (orig `$FF`), Yang, class `$07`
- `DBE5-05DE`: `13FD7C:09` (orig `$FF`), Palom, class `$08`
- `DBE6-000E`: `13FD81:09` (orig `$FF`), Porom, class `$09`
- `DBE6-096E`: `13FD86:09` (orig `$FF`), Tellah (2nd time), class `$0A`
- `DBE6-01AE`: `13FD8B:09` (orig `$FF`), Paladin Cecil, class `$0B`
- `DBEB-00DE`: `13FD90:09` (orig `$FF`), Tellah (3rd time), class `$0C`
- `DBEB-09DE`: `13FD94:09` (orig `$FF`), Yang (2nd time), class `$0D`
- `DBEB-010E`: `13FD99:09` (orig `$FF`), Cid, class `$0E`
- `DBEB-056E`: `13FD9E:09` (orig `$FF`), Kain (2nd time), class `$0F`
- `DBEC-09DE`: `13FDA4:09` (orig `$FF`), Rosa (2nd time), class `$10`
- `DBEC-010E`: `13FDA9:09` (orig `$FF`), adult Rydia, class `$11`
- `DBE8-00AE`: `13FDB3:09` (orig `$FF`), FuSoYa, class `$13`
- `DBE8-09AE`: `13FDB7:09` (orig `$FF`), Kain (3rd time), class `$14`
- `DBE8-016E`: `13FDBA:09` (orig `$FF`), Golbez, class `$15`
- `DBE8-05AE`: `13FDBF:09` (orig `$FF`), Anna, class `$16`


(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:41:31

**Mechanism:** At the start of each battle, `battle/init.asm` copies 5
command ids per character from `ClassCmd` (`$13:FD55`, 5 bytes per
class, `$FF` = empty) to the battle command list at `$3303`:

```
@8b99:  lda     f:ClassCmd,x            ; battle commands for each class
        sta     $3303,y
        cmp     #$ff
        bne     @8ba7
```

Each code puts `$09`, Hide (`Cmd_09` in `battle/cmd.asm`), in the
first empty slot of one class's list. Only Edward has Hide. `Cmd_09`
works on the acting character (`$a6`): it sets bit 7 of `$2006`,
turns the Hide command into `$1C` and turns off the other commands:

```
Cmd_09:
@e879:  ldx     $a6
        lda     $2006,x
        ora     #$80
        sta     $2006,x
        ldx     $3534
        stz     $a9
@e888:  lda     $3303,x
        cmp     #$09
        bne     @e896
        lda     #$1c
        sta     $3303,x
```

Child Rydia (`$03`) and Edge (`$12`) have no empty slot, so they
don't get it. Each code works on its own; the first one alone gives
Dark Knight Cecil Hide from the first battle.

**Evidence:** `ff2_original.smc` has `BF 55 FD 13 99 03 33` (`lda
$13fd55,x / sta $3303,y`) at `$03:8B99`, and `Cmd_09` starts at
`$03:E861` with `A6 A6 BD 06 20 09 80 9D 06 20`. Edward's list at
`$13:FD69` is `00 08 09 01 FF` (Fight, Sing, Hide, Item). Every code
decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- If every party member hides at once, what the battle does was not
  checked.
- The codes use the same bytes as "Every character has the Jump
  command" and the other entries that add a command, and the Paladin
  Cecil code uses the same byte as "Paladin Cecil has the Dark Wave
  command"; use only one of them.

## Party members never leave

**Codes:** (use all 16)
`403A-6D0F` + `2D3B-0F0F` + `E23B-0F6F` + `C1E2-0DD7` + `72E2-0D07` +
`3DE2-0D67` + `0DE2-0DA7` + `DFE2-0FD7` + `EDE2-0F07` + `D7E2-0F67` +
`0AE2-0FA7` + `6DE2-04D7` + `39E2-0407` + `0AE2-0467` + `27E2-04A7` +
`3DE2-07D7`
**Raw form:** `00E6C1:24` (orig `$F0`) + `00E595:D0` (orig `$80`) +
`00E596:FD` (orig `$E5`), then 13 bytes at `00FDD0`-`00FDDC` (orig
all `$FF`):

```
00FDD0:A6 00FDD1:3D 00FDD2:E0 00FDD3:40 00FDD4:01 00FDD5:F0 00FDD6:03
00FDD7:4C 00FDD8:80 00FDD9:E5 00FDDA:4C 00FDDB:D3 00FDDC:E0
```

(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:48:19

**Mechanism:** Story events remove a member with event command `$E8`
(`EventCmd_e8` in `field/event.asm`). It looks through the 5 party
slots for the character id:

```
@e93f:  ldy     $3d
        lda     $1000,y
        and     #$1f
        cmp     $09d5,x
        beq     @e962
```

The first code changes that `beq` into `bit $17`, which does nothing
here, so the search never finds the member and the command ends.

The other codes keep a full party safe. Event command `$E7`
(`EventCmd_e7`, add a member) looks for the first empty slot with no
end check:

```
@e808:  ldx     $3d
        lda     $1000,x
        beq     @e81f       ; find an empty character slot
        ...
        jmp     @e808
```

Since nobody leaves, the party fills up, and the search would go past
slot 4 into `$1140`, where the game keeps the data of characters who
left. The codes point the `jmp @e808` at a new routine in free space
at `$00:FDD0`. It goes back to `@e808` unless `$3d` is `$0140`
(past slot 4). Then it ends the command with `jmp WaitVblankEvent`,
the same way the command normally ends, so the new member isn't added.

**Evidence:** `ff2_original.smc` has `B9 00 10 29 1F DD D5 09 F0 17` at
`$00:E6B9` and `4C 80 E5` (`jmp $e580`) at `$00:E594`, the end of
the `$E7` loop that starts at `$00:E580`. `WaitVblankEvent` is at
`$00:E0D3`; the `$E8` command ends with `4C D3 E0` at `$00:E6D7`.
`$00:FD31`-`$00:FFBB` is all `$FF`. A patched copy disassembles as
above. Every code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The story's scenes don't change; they play as if the member left.
  Members who leave and come back are added again, so a party can have
  two of the same character.
- Once the party has 5 members, the story can't add anyone. For
  example, if the party is full when Paladin Cecil should join, he
  isn't added and Dark Knight Cecil stays.
- Some members come back from the data that `$E8` saves when they
  leave (`CharRemoveTbl` entries with bit 7 set). Since that data is
  never saved, such a member comes back blank if there is room.
- Solo battles hide the other members and bring them back after; that
  doesn't change.

## Every character has the Cry command

**Codes:** (use all 20)
`F4E9-09AE` + `F4E9-050E` + `F4E1-01DE` + `F4E1-050E` + `F4E5-006E` +
`F4E5-096E` + `F4E5-05DE` + `F4E6-000E` + `F4E6-096E` + `F4E6-01AE` +
`F4EB-00DE` + `F4EB-09DE` + `F4EB-010E` + `F4EB-056E` + `F4EC-09DE` +
`F4EC-010E` + `F4E8-00AE` + `F4E8-09AE` + `F4E8-016E` + `F4E8-05AE`
**Raw form:** `$12` in place of one `$FF` in each of 20 class
command lists. The class each code changes:

- `F4E9-09AE`: `13FD57:12` (orig `$FF`), Dark Knight Cecil, class `$01`
- `F4E9-050E`: `13FD5D:12` (orig `$FF`), Kain, class `$02`
- `F4E1-01DE`: `13FD68:12` (orig `$FF`), Tellah, class `$04`
- `F4E1-050E`: `13FD6D:12` (orig `$FF`), Edward, class `$05`
- `F4E5-006E`: `13FD72:12` (orig `$FF`), Rosa, class `$06`
- `F4E5-096E`: `13FD76:12` (orig `$FF`), Yang, class `$07`
- `F4E5-05DE`: `13FD7C:12` (orig `$FF`), Palom, class `$08`
- `F4E6-000E`: `13FD81:12` (orig `$FF`), Porom, class `$09`
- `F4E6-096E`: `13FD86:12` (orig `$FF`), Tellah (2nd time), class `$0A`
- `F4E6-01AE`: `13FD8B:12` (orig `$FF`), Paladin Cecil, class `$0B`
- `F4EB-00DE`: `13FD90:12` (orig `$FF`), Tellah (3rd time), class `$0C`
- `F4EB-09DE`: `13FD94:12` (orig `$FF`), Yang (2nd time), class `$0D`
- `F4EB-010E`: `13FD99:12` (orig `$FF`), Cid, class `$0E`
- `F4EB-056E`: `13FD9E:12` (orig `$FF`), Kain (2nd time), class `$0F`
- `F4EC-09DE`: `13FDA4:12` (orig `$FF`), Rosa (2nd time), class `$10`
- `F4EC-010E`: `13FDA9:12` (orig `$FF`), adult Rydia, class `$11`
- `F4E8-00AE`: `13FDB3:12` (orig `$FF`), FuSoYa, class `$13`
- `F4E8-09AE`: `13FDB7:12` (orig `$FF`), Kain (3rd time), class `$14`
- `F4E8-016E`: `13FDBA:12` (orig `$FF`), Golbez, class `$15`
- `F4E8-05AE`: `13FDBF:12` (orig `$FF`), Anna, class `$16`


(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 16:52:23

**Mechanism:** At the start of each battle, `battle/init.asm` copies 5
command ids per character from `ClassCmd` (`$13:FD55`, 5 bytes per
class, `$FF` = empty) to the battle command list at `$3303`:

```
@8b99:  lda     f:ClassCmd,x            ; battle commands for each class
        sta     $3303,y
        cmp     #$ff
        bne     @8ba7
```

Each code puts `$12`, Cry (`Cmd_12` in `battle/cmd.asm`), in the
first empty slot of one class's list. No class in the USA version has
Cry (the source's comments name Porom as its user), but `Cmd_12` is
still in the ROM. It works on the acting character (`$a6`): for each
monster present, it lowers the monster's level stat (`$202F`) by half
of the user's level stat, to no less than 1:

```
        ldx     $a6
        lda     $202f,x     ; porom's (level + 10) / 2
        lsr
        sta     $a9
        ...
        sec
        lda     $202f,x
        sbc     $a9
        beq     @eb17
        bcs     @eb19
@eb17:  lda     #$01
@eb19:  sta     $202f,x     ; set level stat to 1
```

Child Rydia (`$03`) and Edge (`$12`) have no empty slot, so they
don't get it. Each code works on its own; the first one alone gives
Dark Knight Cecil Cry from the first battle.

**Evidence:** `ff2_original.smc` has `BF 55 FD 13 99 03 33` (`lda
$13fd55,x / sta $3303,y`) at `$03:8B99`. Entry `$12` of the command
table at `$03:B36C` is `$EAD1`, and `$03:EAD1` has `A9 12 8D C8 34`
(`lda #$12 / sta $34c8`), the start of `Cmd_12`. Porom's list at
`$13:FD7D` is `00 02 10 01 FF` (Fight, White, Twin, Item). Every code
decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The USA command name for `$12` is "Dummy", so the menu shows Dummy.
- What the battle message after Cry (`$13`) says in the USA version
  was not checked.
- The codes use the same bytes as "Every character has the Jump
  command" and the other entries that add a command, and the Paladin
  Cecil code uses the same byte as "Paladin Cecil has the Dark Wave
  command"; use only one of them.

## Every character's Fight is Aim

**Codes:** (use all 22)
`DAE9-090E` + `DAE9-016E` + `DAE9-05AE` + `DAE1-09DE` + `DAE1-010E` +
`DAE1-056E` + `DAE5-00AE` + `DAE5-01DE` + `DAE5-050E` + `DAE6-006E` +
`DAE6-09AE` + `DAE6-05DE` + `DAEB-000E` + `DAEB-096E` + `DAEB-01AE` +
`DAEC-00DE` + `DAEC-090E` + `DAEC-016E` + `DAEC-05AE` + `DAE8-09DE` +
`DAE8-010E` + `DAE8-056E`
**Raw form:** `$0C` in place of the `$00` (Fight) at the start of
each of the 22 class command lists. The class each code changes:

- `DAE9-090E`: `13FD55:0C` (orig `$00`), Dark Knight Cecil, class `$01`
- `DAE9-016E`: `13FD5A:0C` (orig `$00`), Kain, class `$02`
- `DAE9-05AE`: `13FD5F:0C` (orig `$00`), child Rydia, class `$03`
- `DAE1-09DE`: `13FD64:0C` (orig `$00`), Tellah, class `$04`
- `DAE1-010E`: `13FD69:0C` (orig `$00`), Edward, class `$05`
- `DAE1-056E`: `13FD6E:0C` (orig `$00`), Rosa, class `$06`
- `DAE5-00AE`: `13FD73:0C` (orig `$00`), Yang, class `$07`
- `DAE5-01DE`: `13FD78:0C` (orig `$00`), Palom, class `$08`
- `DAE5-050E`: `13FD7D:0C` (orig `$00`), Porom, class `$09`
- `DAE6-006E`: `13FD82:0C` (orig `$00`), Tellah (2nd time), class `$0A`
- `DAE6-09AE`: `13FD87:0C` (orig `$00`), Paladin Cecil, class `$0B`
- `DAE6-05DE`: `13FD8C:0C` (orig `$00`), Tellah (3rd time), class `$0C`
- `DAEB-000E`: `13FD91:0C` (orig `$00`), Yang (2nd time), class `$0D`
- `DAEB-096E`: `13FD96:0C` (orig `$00`), Cid, class `$0E`
- `DAEB-01AE`: `13FD9B:0C` (orig `$00`), Kain (2nd time), class `$0F`
- `DAEC-00DE`: `13FDA0:0C` (orig `$00`), Rosa (2nd time), class `$10`
- `DAEC-090E`: `13FDA5:0C` (orig `$00`), adult Rydia, class `$11`
- `DAEC-016E`: `13FDAA:0C` (orig `$00`), Edge, class `$12`
- `DAEC-05AE`: `13FDAF:0C` (orig `$00`), FuSoYa, class `$13`
- `DAE8-09DE`: `13FDB4:0C` (orig `$00`), Kain (3rd time), class `$14`
- `DAE8-010E`: `13FDB9:0C` (orig `$00`), Golbez, class `$15`
- `DAE8-056E`: `13FDBE:0C` (orig `$00`), Anna, class `$16`

(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:07:14

**Mechanism:** Aim (`Cmd_0c`, `battle/cmd.asm`) is a Fight attack
that can't miss. It saves the attacker's hit rate (`$269C`) and the
target's evade (`$2729`), sets them to 255 and 0, runs `DoFightCmd`,
then puts them back:

```
Cmd_0c:
@e839:  lda     $269c
        pha
        lda     $2729
        pha
        clr_a
        sta     $2729
        dec
        sta     $269c
        jsr     DoFightCmd
```

`DoFightCmd` doesn't know which weapon is held, and no flag says that
a character holds a bow when a Fight attack starts, so a code for bow
attacks alone was not possible. Instead, each code puts `$0C`, Aim,
in place of `$00`, Fight, in one class's list in `ClassCmd`
(`$13:FD55`, 5 bytes per class). `battle/init.asm` copies these lists
to the battle command list at `$3303` at the start of each battle.
Every party attack, with a bow or any other weapon, then goes through
`Cmd_0c`. Monsters use `Cmd_00`, not the class lists, so they don't
change.

**Evidence:** `ff2_original.smc` has the 22 lists at `$13:FD55`, each
starting with `$00` (Dark Knight Cecil's `00 01 FF FF FF`, Rosa's `00
02 0C 01 FF`). `BF 55 FD 13 99 03 33` (`lda $13fd55,x / sta $3303,y`)
is at `$03:8B99`. Entry `$0C` of the command table at `$03:B36C` is
`$E821`, and the bytes there (`AD 9C 26 48 AD 29 27 48 7B 8D 29 27
3A 8D 9C 26 20 89 C4 68 8D 29 27 68 8D 9C 26 60`) match `Cmd_0c`.
Every code decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The first two codes (Cecil and Kain) show from the first battle.
  Each code works on its own.
- The menu shows "Aim" in place of "Fight". Rosa already has Aim, so
  she shows it twice.
- Aim does not always hit. `CalcHits` hits when `Rand99` (0 to 98)
  is less than the hit rate. `DoFightCmd` gives the character in
  party slot 1 (Cecil at the start) +25% hit rate with an 8-bit
  result: 255 * 5 / 4 = 318, which keeps only `$3E`, 62. That
  character hits about 62% of the time, as with the normal Aim.
  Others hit every time, unless blind and in the back row with a
  back-row-penalty weapon (255 halved twice is 63). The target's
  evade is 0, so it never dodges. Barrier and image still block hits.
- The codes use other bytes than the entries that add a command (they
  change `$FF` slots), so they work together.

## Mute, toad and pig never block magic

**Codes:** (use all 8)
`DDC8-0D6A` + `DDC8-070A` + `DDCA-04DA` + `DDEF-050E` + `DDEF-05AE` +
`DDE4-000E` + `DDE0-010E` + `6DA5-6DAE`
**Raw form:**

- `DDC8-0D6A`: `03A1B2:00` (orig `$80`), mute doesn't grey out spells
- `DDC8-070A`: `03A1BD:00` (orig `$80`), toad doesn't grey out spells
- `DDCA-04DA`: `03A1C8:00` (orig `$80`), pig doesn't grey out spells
- `DDEF-050E`: `13FD1D:00` (orig `$2C`), White stays enabled
- `DDEF-05AE`: `13FD1F:00` (orig `$04`), Black stays enabled
- `DDE4-000E`: `13FD21:00` (orig `$2C`), Summon stays enabled
- `DDE0-010E`: `13FD49:00` (orig `$2C`), Ninja stays enabled
- `6DA5-6DAE`: `03CE73:80` (orig `$D0`), the spell is cast anyway

(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:14:05

**Mechanism:** Status blocks magic in three places.

1. `UpdateEnabledCmds` (`battle/char.asm`) sets `$A9`, `$AB` and
   `$AC` to `$80` if the character is mute (status 1 bit `$04`), a
   toad (`$20`) or a pig (`$08`). The spell loop ORs them into bit 7
   of each spell's flag byte at `$2C7A`, which greys the spell out:

   ```
   @a1aa:  lda     $2003,x
           and     #$04
           beq     @a1b5
           lda     #$80
           sta     $a9
   ```

   The first three codes change each `lda #$80` to `lda #$00`. The
   loop then only greys out spells the character lacks MP for. The
   toad and pig cases that un-grey the Toad and Pig spells have
   nothing left to do.

2. `CheckCmdEnabled` disables a command when the character's status
   1 byte has a bit set in byte 0 of the command's entry in
   `DisableCmdStatusTbl` (`$13:FD19`, 2 bytes per command). The next
   four codes clear byte 0 for `$02` White, `$03` Black, `$04` Summon
   and `$18` Ninja (`$2C` is toad, pig and mute; `$04` is mute).

3. When the spell goes off, `battle/magic.asm` checks the status
   again and makes the spell fail (`@cea1`):

   ```
   @ce6b:  lda     $352a
           bne     @cea7       ; branch if an item
           lda     $38eb
           bne     @cea7       ; branch if weapon magic
           lda     $2683
           and     #$04
           beq     @ce83       ; branch if attacker is not mute
   ```

   The last code changes the second `bne @cea7` to `bra @cea7`, so
   the mute, toad and pig checks are always skipped. Items and weapon
   magic took this path already.

**Evidence:** `ff2_original.smc` has `BD 03 20 29 04 F0 04 A9 80 85 A9
BD 03 20 29 20 F0 04 A9 80 85 AB BD 03 20 29 08 F0 04 A9 80 85 AC` at
`$03:A1AA`, so the three `$80` bytes are at `$03:A1B2`, `$03:A1BD`
and `$03:A1C8`. `DisableCmdStatusTbl` at `$13:FD19` starts `00 00 00
00 2C 00 04 00 2C 00`, and entry `$18` (`$13:FD49`) is `2C 00`. The
cast check `AD 2A 35 D0 37 AD EB 38 D0 32` is at `$03:CE6B`; `D0 32`
at `$03:CE73` jumps to `$03:CEA7`, the same place as `D0 37`. Every
code decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- A toad or pig keeps its low stats, so its spells may be weak.
- The cast check also runs for monsters, so a mute monster can cast
  too.
- Recall, Twin and the other commands that status blocks are not
  changed. Status 2 (byte 1 of each entry) is not changed either.
- The first three codes alone only un-grey the spells. Without the
  table codes the command is still disabled, and without the last
  code the spell still fails.

## Press Select on the world map to ride a Chocobo

**Codes:** (use all 6)
`D7BF-D767` + `4DB4-DDD7` + `91B4-DFD7` + `91B4-D407` + `DDBA-D467` +
`DDB2-DD67`
**Raw form:**

- `D7BF-D767`: `009C1E:03` (orig `$02`), read the second joypad byte
- `4DB4-DDD7`: `009C20:20` (orig `$80`), test Select, not A
- `91B4-DFD7`: `009C24:56` (orig `$54`), wait for Select to be let go
- `91B4-D407`: `009C29:56` (orig `$54`), mark Select as used
- `DDBA-D467`: `009CCA:00` (orig `$0B`), no need for a Chocobo to be there
- `DDB2-DD67`: `009CD2:00` (orig `$03`), no need to stand on it

(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:17:23

**Mechanism:** `ChangeVehicle` (`field/vehicle.asm`) runs each frame
on the world map. It boards or leaves a vehicle when A is newly
pressed: A is bit `$80` of `$02`, and `$54` is set until A is let go
(`UpdateCtrl` clears it in `field/hardware.asm`):

```
ChangeVehicle:
@9e3b:  lda     $02
        and     #JOY_A
        beq     @9e45       ; return if A button is not pressed
        lda     $54
        beq     @9e46       ; return if A button is not reset
@9e45:  rts
@9e46:  inc     $54         ; reset A button
```

The first four codes make it use Select instead (bit `$20` of `$03`,
with `$56` as its flag). When the party is on foot, standing still and
on the overworld (`$1701` = 0), the board code then checks that a
Chocobo is shown (`$170F`) and that the party is on its tile (`$1710`):

```
@9edd:  lda     $1701
        cmp     #$00
        bne     @9ef4
        lda     $170f
        beq     @9ef4       ; branch if chocobo is not visible
        ldx     $1706
        cpx     $1710       ; chocobo position
        bne     @9ef4
        jmp     BoardChoco
```

The last two codes set the offsets of the `beq` and the second `bne`
to 0, so the code always reaches `jmp BoardChoco`. When the party
gets off, `LandChoco` makes the Chocobo run away as usual.

**Evidence:** `ff2_original.smc` has `A5 02 29 80 F0 04 A5 54 F0 01 60
E6 54` (`ChangeVehicle`) at `$00:9C1D`, and `AD 01 17 C9 00 D0 10 AD
0F 17 F0 0B AE 06 17 EC 10 17 D0 03 4C 20 9E` at `$00:9CBF`. The
`jmp` goes to `$00:9E20`, which has `A9 01 8D 04 17 85 AC 20 3F 8B 60`
(`BoardChoco`). `field/field.asm` sets `$54` and `$56` to 1 together
at start-up, and nothing else on the field uses `$56`. Every code
decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The party must stand still on the overworld. It does nothing in the
  underground or on the moon.
- A no longer boards or leaves any vehicle; Select does all of it.
  Select also gets off the Chocobo, on a tile where that is allowed.
- Select boards the yellow Chocobo even when a vehicle, such as the
  airship, is under the party.
- The black chocobo's flight home ends with a pretend A press. With
  these codes, press Select to land when it gets home.
- Don't use with "Press Select on the world map to ride a flying
  Chocobo". With both, Select always boards the yellow one.

## Press Select on the world map to ride a flying Chocobo

**Codes:** (use all 7)
`D7BF-D767` + `4DB4-DDD7` + `91B4-DFD7` + `91B4-D407` + `DDB3-DD07` +
`DDB3-D407` + `C2B4-64A7`
**Raw form:**

- `D7BF-D767`: `009C1E:03` (orig `$02`), read the second joypad byte
- `4DB4-DDD7`: `009C20:20` (orig `$80`), test Select, not A
- `91B4-DFD7`: `009C24:56` (orig `$54`), wait for Select to be let go
- `91B4-D407`: `009C29:56` (orig `$54`), mark Select as used
- `DDB3-DD07`: `009CE1:00` (orig `$0B`), no need for a black chocobo
  to be there
- `DDB3-D407`: `009CE9:00` (orig `$03`), no need to stand on it
- `C2B4-64A7`: `009E2B:AD` (orig `$EE`), never fly home on its own

(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-24 21:17:23

**Mechanism:** The first four codes are the same as in "Press Select
on the world map to ride a Chocobo": `ChangeVehicle`
(`field/vehicle.asm`) boards and leaves vehicles on a new Select press
in place of A. The board code then checks for the black chocobo the
same way it checks for the yellow one (`$1712` shown, `$1713` its
tile):

```
@9ef4:  lda     $1701
        cmp     #$00
        bne     @9f0b
        lda     $1712
        beq     @9f0b
        ldx     $1706
        cpx     $1713
        bne     @9f0b
        jmp     BoardBkChoco
```

Codes 5 and 6 set the offsets of the `beq` and the second `bne` to 0,
so the code reaches `jmp BoardBkChoco`.

`BoardBkChoco` counts rides in `$1715`. On the first ride it saves the
start tile in `$1716` and `$1717`. When `$1715` is 2,
`CheckPlayerMoveWorld` (`field/move.asm`) flies the bird back to that
tile by itself, and `LandBkChoco` then removes it. The last code
changes `inc $1715` (`EE 15 17`) to `lda $1715` (`AD 15 17`), so the
count stays 0 and the bird never flies home. `LandBkChoco` only lands
on a forest tile (`$A1` bit `$08`), as usual.

**Evidence:** `ff2_original.smc` has `A5 02 29 80 F0 04 A5 54 F0 01 60
E6 54` (`ChangeVehicle`) at `$00:9C1D`, and `AD 01 17 C9 00 D0 10 AD
12 17 F0 0B AE 06 17 EC 13 17 D0 03 4C 2B 9E` at `$00:9CD6`. The `jmp`
goes to `$00:9E2B`, which has `EE 15 17 AD 15 17 C9 01 D0`
(`BoardBkChoco`). Every code decodes back to its raw form with
`snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The party must stand still on the overworld, and not on the yellow
  Chocobo. It does nothing in the underground or on the moon.
- A no longer boards or leaves any vehicle; Select does all of it.
- It lands only on a forest tile. Press Select over a forest to get
  off.
- The black chocobo you find in the forests also never flies home.
- Don't use with "Press Select on the world map to ride a Chocobo".

## Sound effects play half an octave higher

**Codes:** (use all 7)
`73B7-D4F4` + `73B7-D494` + `73B7-D4B4` + `73B7-D424` + `73B7-D7F4` +
`73B7-D794` + `1CB7-D7B4`
**Raw form:** `049838:3E` + `049839:3E` + `04983A:3E` + `04983B:3E` +
`04983C:3E` + `04983D:3E` (orig all `$E1`) + `04983E:6A` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-29 00:41:05

**Mechanism:** The sound program (`notes/ff4-spc.asm`) works out the
pitch of a note the same way for music and sound effects, so no single
byte changes only the sound effects. But each sample has its own
frequency multiplier, and the sound effects use their own samples
(`$00`-`$06`). Sound command `$DB` sets a channel's sample and its
multiplier:

```
0F13: D5 C1 02  MOV   $02C1+X, A      ; set sample id
0F16: FD        MOV   Y, A
0F17: F6 00 FF  MOV   A, $FF00+Y
0F1A: D5 00 03  MOV   $0300+X, A      ; set sample frequency multiplier
```

When a note plays (`$0A87`), a multiplier `m` from `$00` to `$7F` makes
the frequency `f * (1 + m/256)`, and `m` from `$80` to `$FF` makes it
`f * m/256`. The multipliers of the sound effect samples, at SPC
`$FF00`, are uploaded once by `InitSound` from the ROM. Samples `$00`-`$05`
have `$E1` (`f * 0.879`) and sample `$06` has `$00` (`f * 1.0`). The
codes change them to `$3E` (`f * 1.242`) and `$6A` (`f * 1.414`). That
makes every sound effect note 1.414 times (6 semitones) higher than
before. Music uses the multipliers at `$FF40`, which each song loads
for itself, so it doesn't change.

**Evidence:** `ff2_original.smc` has the `SPCData` upload block for
SPC `$FF00` (`$0020` bytes) at `$04:9834`, so its data starts at
`$04:9838`: `E1 E1 E1 E1 E1 E1 00 00 ...`. The sample ids come from
the `$DB` commands in the sound effect scripts (SPC `$B300`-`$CA6F`,
read from the upload block at `$04:A258`): about 240 of them, all for
samples `$00`-`$06` (sample `$00` alone is about half). The 7 sample
pointers at SPC `$1E00` agree. Each code decodes back to its raw form
with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The system sound effects (such as the menu cursor) use the same
  samples, so they change too.
- Sound effects that use the noise generator instead of a sample keep
  their pitch.
- The sound effect scripts use about 240 `$DB` commands, and 2 of them
  (`$DB $43`, `$DB $44`) may be song samples or other data. Those
  sounds, if real, don't change.
- About half an octave is the most this can go up. `$7F`
  (`f * 1.496`) is the largest multiplier, and sample `$06` would need
  more for a bigger step.
- Don't use with "Sound effects play half an octave lower".

## Sound effects play half an octave lower

**Codes:** (use all 7)
`BEB7-D4F4` + `BEB7-D494` + `BEB7-D4B4` + `BEB7-D424` + `BEB7-D7F4` +
`BEB7-D794` + `89B7-D7B4`
**Raw form:** `049838:9F` + `049839:9F` + `04983A:9F` + `04983B:9F` +
`04983C:9F` + `04983D:9F` (orig all `$E1`) + `04983E:B5` (orig `$00`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-29 00:41:05

**Mechanism:** The same as "Sound effects play half an octave higher",
but the multipliers go down. Samples `$00`-`$05` get `$9F`
(`f * 0.621`, from `f * 0.879`) and sample `$06` gets `$B5`
(`f * 0.707`, from `f * 1.0`). That makes every sound effect note 0.707
times (6 semitones) lower than before.

**Evidence:** Same as "Sound effects play half an octave higher". Each
code decodes back to its raw form with `snes_gg_encode.py`.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested.
- The system sound effects, such as the menu cursor, change too, and
  noise sound effects don't.
- `$80` (`f * 0.5`) is the smallest multiplier, so samples `$00`-`$05`
  can go at most about 10 semitones lower.
- Don't use with "Sound effects play half an octave higher".

## Start with Dark Cecil, Paladin Cecil, Kain and FuSoYa

**Codes:** (use all 6)
`B864-D0D6` + `D164-D006` + `3D6A-D16C` + `336A-D1AC` + `356B-D5AC` +
`3568-D5AC`
**Raw form:** `128020:9B` (orig `$E0`) + `128021:06` (orig `$01`) +
`1288CA:E0` (orig `$FD`) + `1288CB:EE` (orig `$09`) + `12889F:E7`
(orig `$F5`) + `1288BF:E7` (orig `$F5`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-31 11:41:20

Tags: `curated` `demo-ready`

**Mechanism:** The first 4 codes are "Skip the opening and start outside
Baron". A new game runs the tail of event `$15` (at `$12:889B`), which is
`f2 04 e7 02` (set an event switch, add Kain) and then a run of `f5 NN`
(clear npc switch `NN`) commands. The last 2 codes change two of those
`f5` bytes to `e7` (add character). `f5 13` at `$12:889F` becomes
`e7 13` (add FuSoYa), and `f5 0b` at `$12:88BF` becomes `e7 0b` (add
Paladin Cecil). Event command `$E7` (`field/event.asm`) loads each one
into the next free party slot. Dark Cecil is in slot 0, Kain in slot 1,
FuSoYa in slot 2 and Paladin Cecil in slot 3.

**Evidence:** `ff2_original.smc` has `F5 13` at `$12:889F` and `F5 0B`
at `$12:88BF`. `CharRemoveTbl` maps id `$13` to `CharProp` entry `$0C`
(FuSoYa) and id `$0B` to entry `$09` (Paladin Cecil). The other codes
are the codes of "Skip the opening and start outside Baron".
Each code decodes back to its raw form with `snes_gg_encode.py`.
Tested in Mesen2 with the old Package codes of "Skip the opening and
start outside Baron" (`3B6A-D16C` + `336E-6DDD`, before 2026-10-31): a
new game starts outside Baron with all four in the party.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Not tested since the Package codes changed to `3D6A-D16C` +
  `336A-D1AC`.
- Only whether all four join was tested. Later story events were not
  checked.
- npc switches `$13` and `$0B` are not cleared. Which npcs they control
  was not checked, so an npc may show up where it shouldn't.
- There are two Cecils. When the story changes Dark Cecil into Paladin
  Cecil or adds FuSoYa, event command `$E7` doesn't check for a
  duplicate. What happens then was not checked.
- Don't use with a "New game: ... is in the party" entry.

## Start a new game with every item that events give

**Codes:** (use all 44)
`5764-D0D6` + `D164-D006` + `3D6A-D16C` + `336A-D1AC` + `3D65-D0AC` +
`E665-D9DC` + `3D65-D90C` + `3E65-D96C` + `3D65-D9AC` + `ED65-D1DC` +
`3D65-D10C` + `F665-D16C` + `3D65-D1AC` + `7E65-D5DC` + `3D65-D50C` +
`7D65-D56C` + `3D65-D5AC` + `E066-D0DC` + `3D66-D00C` + `EF66-D06C` +
`3D66-D0AC` + `E466-D9DC` + `3D66-D90C` + `4E66-D96C` + `3D66-D9AC` +
`4666-D1DC` + `3D66-D10C` + `E766-D16C` + `3D66-D1AC` + `E966-D5DC` +
`3D66-D50C` + `3066-D56C` + `3D66-D5AC` + `AC6B-D0DC` + `3D6B-D00C` +
`F86B-D06C` + `3D6B-D0AC` + `E56B-D9DC` + `3D6B-D90C` + `BC6B-D96C` +
`3D6B-D9AC` + `A66B-D1DC` + `3D6B-D10C` + `736B-D16C`
**Raw form:** `128020:73` (orig `$E0`) + `128021:06` (orig `$01`) + `1288CA:E0` (orig
`$FD`) + `1288CB:EE` (orig `$09`) + `128873:E0` (orig `$EB`) + `128874:F8`
(orig `$03`) + `128875:E0` (orig `$04`) + `128876:EF` (orig `$55`) +
`128877:E0` (orig `$67`) + `128878:F0` (orig `$C2`) + `128879:E0` (orig
`$B2`) + `12887A:18` (orig `$EB`) + `12887B:E0` (orig `$07`) + `12887C:3F`
(orig `$02`) + `12887D:E0` (orig `$C2`) + `12887E:30` (orig `$B2`) +
`12887F:E0` (orig `$EB`) + `128880:F4` (orig `$01`) + `128881:E0` (orig
`$02`) + `128882:F1` (orig `$33`) + `128883:E0` (orig `$41`) + `128884:F2`
(orig `$EB`) + `128885:E0` (orig `$01`) + `128886:2F` (orig `$02`) +
`128887:E0` (orig `$35`) + `128888:28` (orig `$47`) + `128889:E0` (orig
`$34`) + `12888A:F3` (orig `$E9`) + `12888B:E0` (orig `$04`) + `12888C:F5`
(orig `$D5`) + `12888D:E0` (orig `$E9`) + `12888E:E4` (orig `$04`) +
`12888F:E0` (orig `$EB`) + `128890:CA` (orig `$01`) + `128891:E0` (orig
`$02`) + `128892:1B` (orig `$36`) + `128893:E0` (orig `$46`) + `128894:F7`
(orig `$EB`) + `128895:E0` (orig `$05`) + `128896:9A` (orig `$02`) +
`128897:E0` (orig `$C2`) + `128898:C8` (orig `$B2`) + `128899:E0` (orig
`$E9`) + `12889A:3E` (orig `$0E`)
(address:new-byte — the original byte is documentation, not part of the code;
SNES Game Genie has no compare byte)
**Recorded:** 2026-10-31 13:10:01

Tags: `demo-ready`

**Mechanism:** Event command `$E0 nn` (`EventCmd_e0`, `field/event.asm`)
gives item `nn` with `GiveItem`. `ExecEvent` copies each command into a
buffer: commands below `$DB` are 1 byte, `$E2` is 3 bytes, `$FE` is 5,
`$EB` is 3 plus its count byte, and the rest are 2. Walking all 256
scripts in `EventScriptPtrs` (`$12:8000`) with these lengths finds 23
`$E0` commands, for these 20 items and the Package:

- Pan `$F8` (event `$09`), Baron key `$EF` (`$21`), SandRuby `$F0`
  (`$45`), Black sword `$18` (`$52`), Crystal sword `$3F` (`$58`),
  Masamune `$30` (`$59`), TwinHarp `$F4` (`$71`), Earth crystal `$F1`
  (`$74`), Magma key `$F2` (`$7C`), Murasame `$2F` (`$A3`), White spear
  `$28` (`$A4`), Luca key `$F3` (`$A6`), Darkness crystal `$F5` (`$AF`),
  Book `$E4` (`$DE`), item `$CA` (`$DF`, `$E0` and `$F8`), Excalbur `$1B`
  (`$E1`), Adamant `$F7` (`$E2`), Adamant armor `$9A` (`$E3`), Crystal
  `$C8` (`$E7`) and Spoon `$3E` (`$F9`).

This is "Skip the opening and start outside Baron" with the event `$10`
pointer moved 40 bytes earlier, from `$069B` to `$0673` (`$12:8873`).
The 40 bytes before the tail of event `$15` become `e0 nn` for each of
the 20 items, so a new game gives them all and then runs the tail
(Kain joins, the Package from the last 2 skip codes, the overworld).

**Evidence:** `ff2_original.smc` has `F2 04 E7 02` at `$12:889B` and the
pointer `$01E0` at `$12:8020`. The 40 replaced bytes (`$12:8873` to
`$12:889A`) are the end of event `$15`, which a new game doesn't run with
these codes. Each item name was read from the item name table at
`$0F:8000`; the kind in parentheses is from the name's icon. Every code
decodes back to its raw form with `snes_gg_encode.py`. Tested in Mesen2:
the items are in the item list on a new game.

Item `$CA` is おうぎのしょ ("book of secret techniques") in the Japanese
ROMs. Its `ItemProp` entry (`$0F:9680` + 6 × `$1A`) is `00 00 E4 00 00
80`, so it has no effect in battle, and the field menu only uses items
`$CE`-`$EA`. Events `$DF`, `$E0` and `$F8` (`e0 ca fb 37 f1 f0 ..
f5 nn`) give it, play sound `$37`, show message `$F0` of the bank 1 high
dialogue table and clear the npc's switch. Event `$DE` gives the Book
the same way. They are run by the scripts of npcs `$16C`, `$16D`, `$191`
and `$16B` (`NPCScript`, `ff nn`). In both Japanese ROMs (1.0 and 1.1)
those npcs are on maps `$141` (Town of Monsters weapon shop), `$142`
(armor shop), `$149` (Sylvan Cave, Sylphs' house) and `$12B` (Developers'
Room 3). The USA `NPCProp` lists for those maps don't have them, so in
the USA ROM these 4 events never run and items `$CA` and `$E4` can't be
found.

**Revisions:** USA 1.0 only; 1.1 not checked.

**Caveats:**

- Only whether the items show up was tested.
- Key items (keys, crystals, the TwinHarp, the SandRuby, Adamant, the Pan
  and Spoon, the Crystal) come early. Story events that check for them,
  or take them, may act early or wrongly. Not checked.
- Item `$CA` shows as `Dummy` and does nothing. What message `$F0` says
  was not checked.
- Events that give items later still give another copy.
- Treasure chests and shops are not events and are not included.
- Event `$15` is broken by these codes if it ever runs. Without the
  pointer codes the normal intro runs it.
- Uses the same bytes as "Skip the opening and start outside Baron" and
  "Start with Dark Cecil, Paladin Cecil, Kain and FuSoYa"; use only one.
  The Sort slot is free, so a "Start a new game with 99 ..." entry can be
  added.
