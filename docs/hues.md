# HUES.MUL

The `HUES.MUL` file in Ultima Online contains color table data, defining hues used for item coloring, character clothing, effects, and other visual elements.

## Structure

- The file contains 375 hue entries.
- Each hue entry is 708 bytes and consists of:
  - **8 bytes**: Header (often includes group name, etc.)
  - **384 bytes**: 192 colors (3 groups per hue, 64 colors per group, 2 bytes per color, UWORD)
  - **20 bytes**: Footer (may include group name, metadata, etc.)

### Entry Layout

| Offset | Size   | Type   | Description                              |
|--------|--------|--------|------------------------------------------|
| 0x00   | 8      | CHAR[] | Header (group name/identifier)           |
| 0x08   | 384    | UWORD  | 192 colors (3 groups × 64 colors)        |
| 0x188  | 20     | CHAR[] | Footer (group info/metadata)             |

- **UWORD** is a 16-bit unsigned integer (0..65535).
- Each color is stored as a UWORD (2 bytes) in 5-6-5 RGB format or as described in [docs/index.md](index.md).

## Notes

- Hues are used to tint objects, mobiles, and effects throughout Ultima Online.
- The file structure allows for groupings of hues, which can control how colors are blended and applied.

## Reading in C#

Below is a sample C# code snippet to read the hues and extract their color palettes.

```csharp
using System;
using System.IO;

public class HuesMulExample
{
    public static void ExtractHues(string huesPath)
    {
        using (var reader = new BinaryReader(File.OpenRead(huesPath)))
        {
            int huesCount = 375;
            int hueSize = 708;

            for (int i = 0; i < huesCount; i++)
            {
                // Read header
                byte[] header = reader.ReadBytes(8);

                // Read color palette (192 UWORDs = 384 bytes)
                ushort[] colors = new ushort[192];
                for (int j = 0; j < 192; j++)
                    colors[j] = reader.ReadUInt16();

                // Read footer
                byte[] footer = reader.ReadBytes(20);

                // Example: display the first color in the hue
                Console.WriteLine($"Hue {i}: First color (UWORD) = {colors[0]}");
            }
        }
    }
}
```

## Color Format

Each color is a UWORD in 16-bit format:

- **URRRRRGGGGGBBBBB**
  - U = Unused
  - R = Red (5 bits)
  - G = Green (5 bits)
  - B = Blue (5 bits)

To convert to 32-bit RGB:

```c
Color32 = ( (((Color16 >> 10) & 0x1F) * 0xFF / 0x1F) |
            (((Color16 >> 5) & 0x1F) * 0xFF / 0x1F) << 8 |
            (( Color16 & 0x1F) * 0xFF / 0x1F) << 16 );
```

## References

- [Heptazane’s File Format Reference](https://uo.stratics.com/heptazane/fileformats.shtml)
- [Grayworld - UO File Format Documentation](http://www.grayworld.ru/uo/index_eng.html)
