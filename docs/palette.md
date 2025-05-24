# PALETTE.MUL

The `PALETTE.MUL` file in Ultima Online contains color palettes used primarily by animation frames, such as those in ANIM.MUL. These palettes define the colors available for rendering animated sprites and other graphics.

## Structure

- The palette file typically stores 256 color entries.
- Each entry is a UWORD (16-bit unsigned value).
- The file may contain multiple palettes, but documentation is limited and not all uses are fully understood.

### Palette Entry Layout

| Offset | Size | Type   | Description                |
|--------|------|--------|----------------------------|
| 0x00   | 2    | UWORD  | Palette color (see below)  |
| ...    | ...  | ...    | (next color)               |

- Each color is encoded as a UWORD in 16-bit RGB:
  - URRRRRGGGGGBBBBB (U = unused, 5 bits each for R, G, B)

## Notes

- Used by animation frames for color lookup during RLE decoding.
- The exact number of palettes and their organization may vary depending on the client version.

## Reading in C#

Below is a sample C# code snippet to read the palette colors:

```csharp
using System;
using System.IO;

public class PaletteMulExample
{
    public static void ReadPalette(string palettePath)
    {
        using (var reader = new BinaryReader(File.OpenRead(palettePath)))
        {
            // Read 256 colors (512 bytes)
            ushort[] colors = new ushort[256];
            for (int i = 0; i < 256; i++)
            {
                colors[i] = reader.ReadUInt16();
            }

            // Example: Print the first palette color
            Console.WriteLine($"First palette color (UWORD): {colors[0]}");
        }
    }
}
```

## Color Format

Each color uses a 16-bit format:

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
