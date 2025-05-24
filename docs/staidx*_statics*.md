# STAIDX0.MUL & STATICS0.MUL

These files store static tiles for each map facet in Ultima Online, representing non-movable objects like trees, rocks, buildings, etc.

## Structure

### STAIDX0.MUL (Index File)

- Each record is **12 bytes**:
  - **DWORD Lookup**: Offset in STATICS0.MUL where the static data begins
  - **DWORD Size**: Size of the static data (in bytes)
  - **DWORD Unknown**: (Unused/unknown)

| Offset | Size | Type   | Description                        |
|--------|------|--------|------------------------------------|
| 0x00   | 4    | DWORD  | Offset in STATICS0.MUL             |
| 0x04   | 4    | DWORD  | Size of static block (bytes)       |
| 0x08   | 4    | DWORD  | Unknown/Unused                     |

### STATICS0.MUL (Data File)

- Contains static tile entries for each map block.

#### Static Tile Entry (per object, 7 bytes)

| Offset | Size | Type   | Description                |
|--------|------|--------|----------------------------|
| 0x00   | 2    | WORD   | Tile ID (from art.mul)     |
| 0x02   | 1    | BYTE   | X offset (within block)    |
| 0x03   | 1    | BYTE   | Y offset (within block)    |
| 0x04   | 1    | BYTE   | Z height (elevation)       |
| 0x05   | 2    | WORD   | Hue (color)                |

- Each block may contain multiple static objects.
- The number of entries per block is determined by the size in STAIDX0.MUL (size / 7).

## Notes

- There is one pair of STAIDX/STATICS files per map/facet (e.g., STAIDX1.MUL & STATICS1.MUL, etc.).
- Static tile IDs reference art in ART.MUL.
- Hue is used for colorization.

## Reading in C#

Below is a sample C# code snippet for reading static tile data:

```csharp
using System;
using System.IO;

public class StaticsMulExample
{
    public static void ReadStatics(string staidxPath, string staticsPath, int blockId)
    {
        using (var idx = new BinaryReader(File.OpenRead(staidxPath)))
        using (var mul = new BinaryReader(File.OpenRead(staticsPath)))
        {
            idx.BaseStream.Seek(blockId * 12, SeekOrigin.Begin);
            int offset = idx.ReadInt32();
            int length = idx.ReadInt32();
            int unused = idx.ReadInt32();

            if (offset < 0 || length <= 0)
            {
                Console.WriteLine("No static data for this block.");
                return;
            }

            mul.BaseStream.Seek(offset, SeekOrigin.Begin);
            int entries = length / 7;
            for (int i = 0; i < entries; i++)
            {
                ushort tileId = mul.ReadUInt16();
                byte x = mul.ReadByte();
                byte y = mul.ReadByte();
                sbyte z = (sbyte)mul.ReadByte();
                ushort hue = mul.ReadUInt16();

                Console.WriteLine($"Static {i}: TileID={tileId}, X={x}, Y={y}, Z={z}, Hue={hue}");
            }
        }
    }
}
```

## References

- [Heptazane’s File Format Reference](https://uo.stratics.com/heptazane/fileformats.shtml)
- [Grayworld - UO File Format Documentation](http://www.grayworld.ru/uo/index_eng.html)
