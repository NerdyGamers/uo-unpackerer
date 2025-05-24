# RADARCOL.MUL

The `RADARCOL.MUL` file in Ultima Online provides the color mapping for the in-game radar map. This file allows the client to display different terrain and object types using appropriate colors on the minimap.

## Structure

- Contains **0x4000 (16,384)** color entries.
- Each entry is a **UWORD** (16-bit unsigned value).

### Entry Layout

| Offset | Size | Type   | Description               |
|--------|------|--------|---------------------------|
| 0x00   | 2    | UWORD  | Color value (see below)   |
| ...    | ...  | ...    | Next color                |

- Colors are stored as UWORDs in 16-bit RGB (URRRRRGGGGGBBBBB).

## Notes

- Each map tile can reference a palette index from this file for its minimap color.
- The mapping is often hardcoded for land and static tiles in the client.

## Reading in C#

Below is a sample C# code snippet for reading `RADARCOL.MUL`:

```csharp
using System;
using System.IO;

public class RadarcolMulExample
{
    public static void ReadRadarcol(string radarcolPath)
    {
        using (var reader = new BinaryReader(File.OpenRead(radarcolPath)))
        {
            ushort[] colors = new ushort[0x4000];
            for (int i = 0; i < colors.Length; i++)
                colors[i] = reader.ReadUInt16();

            Console.WriteLine($"First radar color (UWORD): {colors[0]}");
        }
    }
}
```

## Color Format

Each color is a 16-bit UWORD:

- **URRRRRGGGGGBBBBB**

To convert to 32-bit RGB:

```c
Color32 = ( (((Color16 >> 10) & 0x1F) * 0xFF / 0x1F) |
            (((Color16 >> 5) & 0x1F) * 0xFF / 0x1F) << 8 |
            (( Color16 & 0x1F) * 0xFF / 0x1F) << 16 );
```

## References

- [Heptazane’s File Format Reference](https://uo.stratics.com/heptazane/fileformats.shtml)
- [Grayworld - UO File Format Documentation](http://www.grayworld.ru/uo/index_eng.html)
