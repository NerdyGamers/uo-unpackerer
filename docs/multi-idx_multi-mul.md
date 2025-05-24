# MULTI.IDX & MULTI.MUL

These files define multi-objects in Ultima Online, such as houses, boats, and other structures that are made up of multiple tiles.

## Structure

### MULTI.IDX (Index File)

- Each record is **4 bytes**:
  - **DWORD Offset**: Offset in MULTI.MUL where the multi-object data begins

| Offset | Size | Type  | Description                |
|--------|------|-------|----------------------------|
| 0x00   | 4    | DWORD | Offset in MULTI.MUL        |

### MULTI.MUL (Data File)

- Contains arrays of multi-tile structures for each multi-object.
- Each entry describes one tile of the multi-object.

| Offset | Size | Type   | Description            |
|--------|------|--------|------------------------|
| 0x00   | 2    | WORD   | Tile ID                |
| 0x02   | 2    | WORD   | X offset (relative)    |
| 0x04   | 2    | WORD   | Y offset (relative)    |
| 0x06   | 2    | WORD   | Z offset (relative)    |
| 0x08   | 4    | DWORD  | Flags                  |

- Multi-object entries are terminated by a special record (often Tile ID = 0xFFFF).

## Notes

- The **Tile ID** references the specific art tile as defined in ART.MUL.
- Offsets are relative to the multi-object's origin.
- **Flags** can describe properties such as impassability, movability, etc.

## Reading in C#

Below is a sample C# code snippet for reading MULTI.IDX and MULTI.MUL:

```csharp
using System;
using System.IO;

public class MultiMulExample
{
    public static void ExtractMulti(string multiIdxPath, string multiMulPath, int multiId)
    {
        using (var idx = new BinaryReader(File.OpenRead(multiIdxPath)))
        using (var mul = new BinaryReader(File.OpenRead(multiMulPath)))
        {
            // Seek to the multiId-th entry in the index (4 bytes per entry)
            idx.BaseStream.Seek(multiId * 4, SeekOrigin.Begin);
            int offset = idx.ReadInt32();

            if (offset < 0)
            {
                Console.WriteLine("No multi-object found for this ID.");
                return;
            }

            // Seek to the multi-object data in multi.mul
            mul.BaseStream.Seek(offset, SeekOrigin.Begin);

            while (true)
            {
                ushort tileId = mul.ReadUInt16();
                if (tileId == 0xFFFF) break; // End of multi-object

                short xOffset = mul.ReadInt16();
                short yOffset = mul.ReadInt16();
                short zOffset = mul.ReadInt16();
                uint flags = mul.ReadUInt32();

                Console.WriteLine($"TileID={tileId}, X={xOffset}, Y={yOffset}, Z={zOffset}, Flags={flags}");
            }
        }
    }
}
```

## References

- [Heptazane’s File Format Reference](https://uo.stratics.com/heptazane/fileformats.shtml)
- [Grayworld - UO File Format Documentation](http://www.grayworld.ru/uo/index_eng.html)
