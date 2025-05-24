# MAP0.MUL, MAP1.MUL, ... MAPX.MUL

These files contain terrain data for each facet (Trammel, Felucca, Ilshenar, Malas, etc.) in Ultima Online. Each map file encodes the landscape tiles, elevations, and other information for a particular world area.

## Structure

- Each map file is divided into 8x8 tile blocks.
- Each block is 196 bytes:
  - **192 bytes**: 8x8 tile data (3 bytes per tile)
  - **4 bytes**: Block header

### Tile Structure (per tile, 3 bytes)
| Offset | Size | Type  | Description      |
|--------|------|-------|------------------|
| 0x00   | 2    | WORD  | Tile ID          |
| 0x02   | 1    | BYTE  | Z height (elevation) |

### Block Structure (per 8x8 block, 196 bytes)
| Offset | Size  | Type    | Description         |
|--------|-------|---------|---------------------|
| 0x00   | 4     | DWORD   | Block header        |
| 0x04   | 192   | Tile[]  | 8x8 tiles (3 bytes each) |

- The map is arranged as a set of these blocks, covering the full dimensions of the facet.

## Notes

- Each facet's map file is named according to its index (e.g., MAP0.MUL for Felucca, MAP1.MUL for Trammel, etc.).
- The Z height value determines the elevation of each tile, affecting movement and display.
- Tile IDs reference specific terrain types as defined in TILEDATA.MUL.

## Reading in C#

Below is a sample C# code snippet for reading MAPX.MUL files:

```csharp
using System;
using System.IO;

public class MapMulExample
{
    public static void ReadMapBlock(string mapPath, int blockIndex)
    {
        const int blockSize = 196;
        using (var reader = new BinaryReader(File.OpenRead(mapPath)))
        {
            // Seek to the beginning of the desired block
            reader.BaseStream.Seek(blockIndex * blockSize, SeekOrigin.Begin);

            // Read block header
            uint header = reader.ReadUInt32();

            // Read 8x8 tiles
            for (int y = 0; y < 8; y++)
            {
                for (int x = 0; x < 8; x++)
                {
                    ushort tileId = reader.ReadUInt16();
                    sbyte z = (sbyte)reader.ReadByte();
                    Console.WriteLine($"Tile [{x},{y}]: ID={tileId}, Z={z}");
                }
            }
        }
    }
}
```

## Further Reading

- [Heptazane’s File Format Reference](https://uo.stratics.com/heptazane/fileformats.shtml)
- [Grayworld - UO File Format Documentation](http://www.grayworld.ru/uo/index_eng.html)
