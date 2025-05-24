# SKILLS.IDX & SKILLS.MUL

These files contain the names and properties of skills available in Ultima Online.

## Structure

### SKILLS.IDX (Index File)

- Each record is 12 bytes (index into SKILLS.MUL):

| Offset | Size | Type   | Description                     |
|--------|------|--------|---------------------------------|
| 0x00   | 4    | DWORD  | Offset in SKILLS.MUL            |
| 0x04   | 4    | DWORD  | Size of skill entry (optional)  |
| 0x08   | 4    | DWORD  | Unknown / unused                |

### SKILLS.MUL (Data File)

- Contains skill names as ASCII text, null-terminated.
- Entries may also include additional properties (client/version dependent).

#### Entry Layout (typical)

| Offset | Size        | Type   | Description            |
|--------|-------------|--------|------------------------|
| 0x00   | variable    | CHAR[] | Skill name (ASCII, null-terminated) |
| ...    | ...         | ...    | (Possibly more data per client version) |

## Notes

- The number and order of skills is subject to change between client versions.
- Skill names can be localized or extended depending on client.
- Some clients may include extra fields after the skill name (check your client version for details).

## Reading in C#

Below is a basic C# code snippet for reading skill names:

```csharp
using System;
using System.IO;
using System.Text;

public class SkillsMulExample
{
    public static void ReadSkills(string skillsIdxPath, string skillsMulPath)
    {
        using (var idx = new BinaryReader(File.OpenRead(skillsIdxPath)))
        using (var mul = new BinaryReader(File.OpenRead(skillsMulPath)))
        {
            int recordSize = 12;
            long idxLength = idx.BaseStream.Length;

            for (int i = 0; i < idxLength / recordSize; i++)
            {
                int offset = idx.ReadInt32();
                int size = idx.ReadInt32();
                int unused = idx.ReadInt32();

                if (offset < 0) continue;

                mul.BaseStream.Seek(offset, SeekOrigin.Begin);
                var nameBytes = new List<byte>();
                byte b;
                while ((b = mul.ReadByte()) != 0)
                    nameBytes.Add(b);

                string skillName = Encoding.ASCII.GetString(nameBytes.ToArray());
                Console.WriteLine($"Skill {i}: {skillName}");
            }
        }
    }
}
```

## References

- [Heptazane’s File Format Reference](https://uo.stratics.com/heptazane/fileformats.shtml)
- [Grayworld - UO File Format Documentation](http://www.grayworld.ru/uo/index_eng.html)
