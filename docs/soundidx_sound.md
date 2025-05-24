# SOUNDIDX.MUL & SOUND.MUL

These files store sound effects used in Ultima Online, including environment sounds, item noises, and more.

## Structure

### SOUNDIDX.MUL (Index File)

- Each record is **12 bytes**:
  - **DWORD Lookup**: Offset in SOUND.MUL where the sound data begins
  - **DWORD Size**: Size of the sound data (in bytes)
  - **DWORD Unknown**: (Unused/unknown)

| Offset | Size | Type   | Description                  |
|--------|------|--------|------------------------------|
| 0x00   | 4    | DWORD  | Offset in SOUND.MUL          |
| 0x04   | 4    | DWORD  | Size of sound entry (bytes)  |
| 0x08   | 4    | DWORD  | Unknown/Unused               |

### SOUND.MUL (Data File)

- Contains raw sound data, typically in IMA ADPCM format but may vary by client version.

## Notes

- Not all entries are guaranteed to contain valid sound data; some may be unused.
- The actual format of the sound data is often IMA ADPCM, but sometimes PCM or other encodings are used.

## Reading in C#

Below is a C# code snippet for reading SOUNDIDX.MUL and SOUND.MUL entries:

```csharp
using System;
using System.IO;

public class SoundMulExample
{
    public static void ReadSound(string soundIdxPath, string soundMulPath, int soundId)
    {
        using (var idx = new BinaryReader(File.OpenRead(soundIdxPath)))
        using (var mul = new BinaryReader(File.OpenRead(soundMulPath)))
        {
            idx.BaseStream.Seek(soundId * 12, SeekOrigin.Begin);
            int offset = idx.ReadInt32();
            int length = idx.ReadInt32();
            int unused = idx.ReadInt32();

            if (offset < 0 || length <= 0)
            {
                Console.WriteLine("No sound found for this ID.");
                return;
            }

            mul.BaseStream.Seek(offset, SeekOrigin.Begin);
            byte[] soundData = mul.ReadBytes(length);

            Console.WriteLine($"SoundID {soundId}: {length} bytes read");
        }
    }
}
```

## References

- [Heptazane’s File Format Reference](https://uo.stratics.com/heptazane/fileformats.shtml)
- [Grayworld - UO File Format Documentation](http://www.grayworld.ru/uo/index_eng.html)
