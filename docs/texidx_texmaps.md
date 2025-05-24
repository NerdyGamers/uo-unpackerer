# TEXIDX.MUL & TEXMAPS.MUL

These files contain terrain texture maps used for ground surfaces in Ultima Online.

## Structure

### TEXIDX.MUL (Index File)

- Each record is **12 bytes**:
  - **DWORD Lookup**: Offset in TEXMAPS.MUL
  - **DWORD Size**: Size of texture entry
  - **DWORD Unknown**: (Unused/unknown)

| Offset | Size | Type   | Description                     |
|--------|------|--------|---------------------------------|
| 0x00   | 4    | DWORD  | Offset in TEXMAPS.MUL           |
| 0x04   | 4    | DWORD  | Size of texture entry           |
| 0x08   | 4    | DWORD  | Unknown/Unused                  |

### TEXMAPS.MUL (Data File)

- Contains bitmap textures, each typically 44x44 pixels, usually RLE-compressed.

#### Texture Entry

| Offset | Size | Type   | Description                |
|--------|------|--------|----------------------------|
| 0x00   | 2    | WORD   | Width (usually 44)         |
| 0x02   | 2    | WORD   | Height (usually 44)        |
| 0x04   | ...  | Data   | Compressed bitmap data     |

- The actual decoding of bitmap data requires RLE decompression, similar to ART.MUL and GUMPART.MUL files.

## Notes

- Used to render ground textures for terrain tiles.
- Texture IDs referenced from TILEDATA.MUL.

## Reading in C#

```csharp
using System;
using System.IO;

public class TexmapsMulExample
{
    public static void ReadTexture(string texidxPath, string texmapsPath, int textureId)
    {
        using (var idx = new BinaryReader(File.OpenRead(texidxPath)))
        using (var mul = new BinaryReader(File.OpenRead(texmapsPath)))
        {
            idx.BaseStream.Seek(textureId * 12, SeekOrigin.Begin);
            int lookup = idx.ReadInt32();
            int length = idx.ReadInt32();
            int unused = idx.ReadInt32();

            if (lookup < 0 || length <= 0)
            {
                Console.WriteLine("No texture found for this ID.");
                return;
            }

            mul.BaseStream.Seek(lookup, SeekOrigin.Begin);
            ushort width = mul.ReadUInt16();
            ushort height = mul.ReadUInt16();
            Console.WriteLine($"Texture {textureId}: width={width}, height={height}");
            // Bitmap data decoding omitted for brevity.
        }
    }
}
```

## References

- [Heptazane’s File Format Reference](https://uo.stratics.com/heptazane/fileformats.shtml)
- [Grayworld - UO File Format Documentation](http://www.grayworld.ru/uo/index_eng.html)
