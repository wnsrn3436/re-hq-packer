# HQ format

Myth of Millennium, Seven Years War 2, and Seven Years War 2+ keep their game data as a pair of files, `Data.hq` and `Data`. `Data.hq` is the header and `Data` is the body. Both files are encrypted the same way.

## Encryption

Bit 0x10 of every byte is flipped. This is the same as subtracting 16 when the bit is set and adding 16 when it is clear. Applying it once more restores the original.

```
encrypted byte = original byte XOR 0x10
```

## Header (Data.hq)

After the transform it is a single line of text. All values are decimal strings.

```
총라인수 : [file count]->name<TAB>[size]<TAB>[offset]->name<TAB>[size]<TAB>[offset]->
```

| Part | Content |
|---|---|
| `총라인수 : [N]` | File count. The Korean prefix means "total line count" |
| name | Name of the file inside the body. `folder\name` when it sits in a subfolder |
| `[size]` | File size in bytes |
| `[offset]` | Where the file starts inside the body. The first file is at 0 |

The `->` between entries is the two bytes `0D 0A` in the encrypted file. The transform turns them into `1D 1A`, which are awkward to handle as text, so HQ Packer swaps them for `->` when reading and swaps them back when packing.

Unpacking the `Data.hq` of Myth of Millennium gives this.

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

The file contents joined end to end in header order. There is no alignment or padding, so the sum of the preceding sizes is the next file's offset. The whole file carries the same transform.

## Unpacking

1. Apply the transform to both files to restore them.
2. Split the header on `->` and take the name, size, and offset of each entry.
3. Slice the body from each offset for each size and save under the name.

Packing is the reverse. Measure the file sizes, accumulate the offsets while writing the header, join the contents, and apply the transform to both files.
