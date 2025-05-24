# ARTIDX.MUL & ART.MUL

These files store static art tiles (landscapes, objects, etc.) in Ultima Online.  
- **ARTIDX.MUL** is the index file. Each record is 12 bytes:
  - **DWORD Lookup**: Offset in ART.MUL
  - **DWORD Size**: Size of the art tile (in bytes)
  - **DWORD Unknown**: (Unused/unknown)
- **ART.MUL** contains the actual image data, typically as RLE-compressed bitmaps.

## Structure

### ARTIDX.MUL entry (12 bytes per art tile)
| Offset | Size  | Type   | Description            |
|--------|-------|--------|------------------------|
| 0x00   | 4     | DWORD  | Offset in ART.MUL      |
| 0x04   | 4     | DWORD  | Size of art entry      |
| 0x08   | 4     | DWORD  | Unknown/Unused         |

### ART.MUL entry (variable size)
| Offset | Size  | Type   | Description            |
|--------|-------|--------|------------------------|
| 0x00   | 2     | WORD   | Width                  |
| 0x02   | 2     | WORD   | Height                 |
| 0x04   | ...   | Data   | Compressed bitmap data |

## Reading in C#

Below is a sample C# code snippet for reading ARTIDX.MUL and ART.MUL to extract metadata and decompress the image.  

```csharp
using System;
using System.Drawing;
using System.IO;

public class ArtMulExample
{
    public static void ExtractArt(string artIdxPath, string artMulPath, int artId)
    {
        using (var idx = new BinaryReader(File.OpenRead(artIdxPath)))
        using (var mul = new BinaryReader(File.OpenRead(artMulPath)))
        {
            // Seek to the artId-th entry in the index (12 bytes per entry)
            idx.BaseStream.Seek(artId * 12, SeekOrigin.Begin);
            int lookup = idx.ReadInt32();
            int length = idx.ReadInt32();
            int unused = idx.ReadInt32();

            if (lookup < 0 || length <= 0)
            {
                Console.WriteLine("No art found for this ID.");
                return;
            }

            // Seek to the art data in art.mul
            mul.BaseStream.Seek(lookup, SeekOrigin.Begin);
            ushort width = mul.ReadUInt16();
            ushort height = mul.ReadUInt16();

            Console.WriteLine($"ArtID {artId}: width={width}, height={height}");

            // Example: Read and decode pixel data (pseudo-code, real decoding is RLE)
            // This just shows reading the header; actual RLE decoding is more complex.
            // For full details, see: https://uo.stratics.com/heptazane/fileformats.shtml

            // You may want to use ServUO's Ultima.Art.GetStatic for real use.
        }
    }
}
```

## Image Data Stream

The image data in ART.MUL is usually RLE compressed. Decoding it requires reading row headers, offsets, and run-lengths, and then mapping palette indices to colors.  
A simplified pseudo-code for the decompression process (from Heptazane’s docs and ServUO):

```pseudo
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

For more details or a full C# RLE decoder, see ServUO’s [Ultima/Art.cs](https://github.com/ServUO/ServUO/tree/pub57/Ultima/Art.cs).
