# VERDATA.MUL

The `VERDATA.MUL` file is a patch file used by the Ultima Online client to override or supplement content from other `.MUL` files. It allows the client to update existing resources or add new ones without replacing the entire resource file.

## Structure

- Contains a list of records, each specifying a patch or override for a file entry.
- Each record contains:
  - **File ID:** Target file type being patched (e.g., ART, GUMP, MAP, etc.).
  - **Block ID:** Index or identifier for the specific entry in the target file.
  - **Position:** Offset in the original file (if applicable).
  - **Length:** Length of the replacement data (in bytes).
  - **Data:** Replacement or additional data.

### Record Layout

| Offset | Size | Type   | Description                           |
|--------|------|--------|---------------------------------------|
| 0x00   | 4    | INT32  | File ID (target file type)            |
| 0x04   | 4    | INT32  | Block ID (target entry index)         |
| 0x08   | 4    | INT32  | Position (offset in original file)    |
| 0x0C   | 4    | INT32  | Length (size of replacement data)     |
| 0x10   | ...  | BYTE[] | Replacement data                      |

- The record count is typically stored at the beginning of the file (4 bytes).

## Notes

- The client checks `VERDATA.MUL` before loading entries from other resource files. If a match is found, the patched data is used instead.
- Used for hotfixes, map updates, art changes, etc., without requiring a full file replacement.

## Reading in C#

Below is a sample C# code snippet to enumerate and read records from `VERDATA.MUL`:

```csharp
using System;
using System.IO;

public class VerdataMulExample
{
    public static void ReadVerdata(string verdataPath)
    {
        using (var reader = new BinaryReader(File.OpenRead(verdataPath)))
        {
            int recordCount = reader.ReadInt32();
            Console.WriteLine($"Records in verdata: {recordCount}");

            for (int i = 0; i < recordCount; i++)
            {
                int fileId = reader.ReadInt32();
                int blockId = reader.ReadInt32();
                int position = reader.ReadInt32();
                int length = reader.ReadInt32();

                byte[] data = reader.ReadBytes(length);

                Console.WriteLine($"Patch {i}: FileID={fileId}, BlockID={blockId}, Position={position}, Length={length}");
            }
        }
    }
}
```
