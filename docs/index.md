
## Miscellaneous Information

For the rest of this documentation, we will treat the following types as described:

* CHAR : 8-bit unsigned character
* BYTE : 8-bit signed value (-128..127)
* UBYTE : 8-bit unsigned value (0..255)
* WORD : 16-bit signed value (-32768..32767)
* UWORD : 16-bit unsigned value (0..65535)
* DWORD : 32-bit signed value (-2147483648..2147483647)

Ultima Online is completely 16-bit based -- meaning each pixel is an UWORD value that can be broken down as follows:

URRRRRGGGGGBBBBB

*U=Unused, R=Red, G=Green, B=Blue

To convert to 32-bit:

```
Color32 = ( (((Color16 >> 10) & 0x1F) * 0xFF / 0x1F) |
((((Color16 >> 5) & 0x1F) * 0xFF / 0x1F) << 8) |
((( Color16 & 0x1F) * 0xFF / 0x1F) << 16));
```

## File Contents

## Definitions

### Anim.idx
This file contains an index into ANIM.MUL. To load a specific group of frames from ANIM.MUL, simply seek BNum * 12, read in the index record and use Lookup to find the group within ANIM.MUL.

Index Record (12 bytes)

|0 - 3|4 - 7|8 - B|
|:-|:-|:-|
| Lookup | Size | Unknown |

- DWORD Lookup - Is either undefined ($FFFFFFFF -1) or the file offset in ANIM.MUL
- DWORD Size - Size of the art tile
- DWORD Unknown - Unknown

### ANIM.MUL

Contains the raw animation data. It can be accessed using ANIM.IDX: 

- AnimationGroup
- WORD[256] Palette
- DWORD FrameCount
- DWORD[FrameCount] FrameOffset

**Frame**

Seek from the end of Palette plus FrameOffset[FrameNum] bytes to find the start of Frame: 

- WORD ImageCenterX
- WORD ImageCenterY
- WORD Width
- WORD Height
- data stream

**data stream:**

```
// Note: Set CenterY and CenterY to the vertical and horizontal position in
//       the bitmap at which you want the anim to be drawn.
		
PrevLineNum = $FF
Y = CenterY - ImageCenterY - Height
while not EOF
{
  Read UWORD RowHeader
  Read UWORD RowOfs
  
  if (RowHeader = 0x7FFF) or (RowOfs = 0x7FFF) then Exit
  
  RunLength = RowHeader and $FFF
  LineNum = RowHeader shr 12
  Unknown = RowOfs and $3F
  RowOfs = RowOfs sar 6
  X = CenterX + RowOfs
  if (PrevLineNum <> $FF) and (LineNum <> PrevLineNum) then Y++
  PrevLineNum = LineNum
  if Y >= 0 then
  {
    if Y >= MaxHeight then Exit;
  
    For I := 0 to RunLength-1 do 
    {
      Read(B, 1)
      Row[X+I,Y] = Palette[B]
    }
  }
  Seek(RunLength, FromCurrent)
}
```

---

### ARTIDX.MUL & ART.MUL

These files are used together for storing static art tiles (landscapes, objects, etc.).

#### ARTIDX.MUL

- Each record is 12 bytes:
  - **DWORD Lookup**: Offset in ART.MUL (file offset)
  - **DWORD Size**: Size of the art tile
  - **DWORD Unknown**: (Unused/unknown)

#### ART.MUL

- Contains the actual image data, usually as RLE-compressed bitmaps.

---

### GUMPIDX.MUL & GUMPART.MUL

Used for storing "gumps" (UI graphics, buttons, paperdolls, etc.).

#### GUMPIDX.MUL (Index File)

- Each record is 12 bytes:
  - **DWORD Lookup**: Offset in GUMPART.MUL
  - **DWORD Size**: Size of gump
  - **DWORD Unknown**: (Unused/unknown)

#### GUMPART.MUL (Data File)

- Contains bitmap data for gumps, usually RLE-compressed.

---

### HUES.MUL

Contains color tables for hues (used for item coloring, etc.).

- 375 hues, each 708 bytes:
  - 8 bytes: Header (group name, etc.)
  - 384 bytes: 64 colors per group, 3 groups per hue, 2 bytes per color (UWORD)
  - 20 bytes: Footer (group name, etc.)

---

### MAP0.MUL, MAP1.MUL, ... MAPX.MUL

Terrain data for each facet (Trammel, Felucca, etc.).

- Each 8x8 block is 196 bytes:
  - 192 bytes: 8x8 tiles (3 bytes per tile)
  - 4 bytes: Header
- **Tile:**
  - 2 bytes: Tile ID
  - 1 byte: Z height

---

### MULTI.IDX & MULTI.MUL

Defines multi-objects (houses, boats, etc.).

#### MULTI.IDX (Index File)
- 4 bytes per record: Offset into MULTI.MUL

#### MULTI.MUL (Data File)
- Contains arrays of multi-tile structures:
  - 2 bytes: Tile ID
  - 2 bytes: X offset
  - 2 bytes: Y offset
  - 2 bytes: Z offset
  - 4 bytes: Flags

---

### PALETTE.MUL

Contains palettes used by animation frames.
- Typically stores 256 color entries (UWORD).
- Details are not well-documented.

---

### RADARCOL.MUL

Contains color mapping for the radar map.

- 0x4000 (16,384) colors
- 2 bytes per color (UWORD)

---

### SKILLS.IDX & SKILLS.MUL

Contains skill names and properties.

#### SKILLS.IDX (Index File)
- 12 bytes per record (index into SKILLS.MUL)

#### SKILLS.MUL (Data File)
- Contains skill names (ASCII text, null-terminated)

---

### SOUNDIDX.MUL & SOUND.MUL

Stores sound effects.

#### SOUNDIDX.MUL (Index File)
- 12 bytes per record:
  - **DWORD Lookup**: Offset in SOUND.MUL
  - **DWORD Size**: Size of sound data
  - **DWORD Unknown**: (Unused)

#### SOUND.MUL (Data File)
- Raw sound data, often in IMA ADPCM format

---

### STAIDX0.MUL & STATICS0.MUL

Static tiles for each map facet.

#### STAIDX0.MUL (Index File)
- 12 bytes per record

#### STATICS0.MUL (Data File)
- Contains static tile data:
  - 2 bytes: Tile ID
  - 1 byte: X offset
  - 1 byte: Y offset
  - 1 byte: Z height
  - 1 byte: Hue

---

### TILEDATA.MUL

Tile properties and flags (walkable, impassable, etc.).

- Contains item and land tile data:
  - Each land tile: 26 bytes (flags, name, etc.)
  - Each item tile: 37 bytes (flags, weight, quality, etc.)

---

### TEXIDX.MUL & TEXMAPS.MUL

Texture maps for terrain.

#### TEXIDX.MUL (Index File)

- 12 bytes per record

#### TEXMAPS.MUL (Data File)

- Contains 44x44 bitmap textures, usually RLE-compressed

---

### VERDATA.MUL

Patch file for overriding or adding entries in other MUL files.

- Contains records with:
  - File ID
  - Block ID
  - Position
  - Length
  - Replacement data
 
---

## Credits
Most of the file formats are reverse engineered by great UO Community members. They deserve a special thank you.

anim.mul, anim.idx, art.mul, artidx.mul, palette.mul, texmaps.mul, texidx.mul, tiledata.mul, gumpidx.mul, gumpart.mul, hues.mul by Alazane

map0, radarcol, staidx0, statics0 by Tim Walters

skills.idx, skills.mul by Sir Valgriz

soundidx, sound by Steve Dang

**verdata** by Cironian
---

### Extra Credits

**References:**  
- [uo.stratics.com/heptazane/fileformats.shtml](https://uo.stratics.com/heptazane/fileformats.shtml)  
- [Grayworld - UO File Format Documentation](http://www.grayworld.ru/uo/index_eng.html)
