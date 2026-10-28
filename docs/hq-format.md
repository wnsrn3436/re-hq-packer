# HQ format

Myth of Millennium, Seven Years War 2, and Seven Years War 2+ keep their game data as a pair of files, `Data.hq` and `Data`. `Data.hq` is the header and `Data` is the body, and both files are encrypted the same way.

## Encryption

Bit 0x10 of every byte is flipped. This is the same as subtracting 16 when the bit is set and adding 16 when it is clear. Applying it once more restores the original.

```
encrypted byte = original byte XOR 0x10
```

## Header (Data.hq)

Undoing the transform gives a single line of text in which every value is a decimal string.

```
총라인수 : [file count]->name<TAB>[size]<TAB>[offset]->name<TAB>[size]<TAB>[offset]->
```

| Part | Content |
|---|---|
| `총라인수 : [N]` | File count |
| name | Name of the file inside the body. `folder\name` when it sits in a subfolder |
| `[size]` | File size in bytes |
| `[offset]` | Where the file starts inside the body. The first file is at 0 |

The `->` between entries is the two bytes `0D 0A` in the encrypted file. The transform turns these two bytes into `1D 1A`, which are awkward to handle as a string, so HQ Packer replaces them with `->` when reading and puts them back when packing.

Unpacking Myth of Millennium's `Data.hq` gives the following.

| Name | Size | Offset |
|---|---|---|
| `CastleType.txt` | 1108 | 0 |
| `EffectSound.dat` | 35457 | 1108 |
| `FontInfo_chi.dat` | 371 | 36565 |
| `FontInfo_kor.dat` | 1461 | 36936 |
| `FontInfo_tai.dat` | 393 | 38397 |
| `ImgFile.dat` | 12200 | 38790 |
| `NewBlkType.txt` | 711 | 50990 |
| `SeaType.txt` | 2361 | 51701 |
| `Text_chi.dat` | 16175 | 54062 |
| `Text_eng.dat` | 15683 | 70237 |
| `Text_kor.dat` | 20959 | 85920 |
| `Text_tai.dat` | 16232 | 106879 |
| `TileFile.txt` | 4597 | 123111 |

## Body (Data)

The body is the file contents joined end to end in header order, unchanged. There is no alignment or padding, so the sum of the preceding files' sizes is the next file's offset. The same transform is applied to the whole file.

## Unpacking

1. Apply the transform to both files to restore them.
2. Split the header on `->` and take the name, size, and offset of each entry.
3. Cut the body from each offset for its size and save it under its name.

Packing is the reverse. Measure the file sizes, accumulate the offsets while writing the header, join the contents, and then apply the transform to both files.
