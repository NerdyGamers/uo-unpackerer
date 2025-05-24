# TILEDATA.MUL

The `TILEDATA.MUL` file in Ultima Online contains properties and flags for both land and item tiles, defining how each tile behaves in the game (e.g., walkable, impassable, animated).

## Structure

- The file is divided into land tile and item tile sections.
- Each land tile entry is **26 bytes**.
- Each item tile entry is **37 bytes**.

### Land Tile Entry (26 bytes)

| Offset | Size | Type    | Description                |
|--------|------|---------|----------------------------|
| 0x00   | 4    | DWORD   | Flags                      |
| 0x04   | 2    | WORD    | Texture ID                 |
| 0x06   | 20   | CHAR[20]| Name (ASCII, null-padded)  |

### Item Tile Entry (37 bytes)

| Offset | Size | Type    | Description                |
|--------|------|---------|----------------------------|
| 0x00   | 4    | DWORD   | Flags                      |
| 0x04   | 1    | BYTE    | Weight                     |
| 0x05   | 2    | WORD    | Quality                    |
| 0x07   | 2    | WORD    | Unknown                    |
| 0x09   | 1    | BYTE    | Quantity                   |
| 0x0A   | 1    | BYTE    | Animation ID               |
| 0x0B   | 1    | BYTE    | Unknown                    |
| 0x0C   | 1    | BYTE    | Unknown                    |
| 0x0D   | 1    | BYTE    | Value                      |
| 0x0E   | 1    | BYTE    | Height                     |
| 0x0F   | 20   | CHAR[20]| Name (ASCII, null-padded)  |

- **Flags**: Control properties such as walkable, impassable, surfacable, etc.
- The number of tiles depends on client version.

## Notes

- The file is used to determine in-game physics, rendering, and interactions for each tile.
- Tile IDs reference entries in ART.MUL (for graphics) and MAP files (for terrain).

## Reading in C#

```csharp
using System;
using System.IO;
using System.Text;

public class TiledataMulExample
{
    public static void ReadLandTile(string tiledataPath, int landTileId)
    {
        const int landEntrySize = 26;
        using (var reader = new BinaryReader(File.OpenRead(tiledataPath)))
        {
            reader.BaseStream.Seek(landTileId * landEntrySize, SeekOrigin.Begin);
            uint flags = reader.ReadUInt32();
            ushort textureId = reader.ReadUInt16();
            byte[] nameBytes = reader.ReadBytes(20);
            string name = Encoding.ASCII.GetString(nameBytes).TrimEnd('\0');
            Console.WriteLine($"Land TileID={landTileId}: Name={name}, Flags={flags}, TextureID={textureId}");
        }
    }
}
```

## References

- [Heptazane’s File Format Reference](https://uo.stratics.com/heptazane/fileformats.shtml)
- [Grayworld - UO File Format Documentation](http://www.grayworld.ru/uo/index_eng.html)
