# HQ Packer

A program that unpacks and repacks the `Data.hq` and `Data` pair in which Myth of Millennium, Seven Years War 2, and Seven Years War 2+ keep their game data. Both files are encrypted with the same per byte transform. `Data.hq` is the header listing file names, sizes, and offsets, and `Data` is the body with the file contents joined end to end. Unpacking puts the individual files under `unpacked/` with their original names, and repacking after editing gives a pair the game reads as is. The format analysis is in [docs/hq-format.md](docs/hq-format.md).

<p>
  <img src="docs/screenshots/screenshot-1.png" width="480" alt="Decrypted header">
</p>


## How to use

Download from Releases, extract it, put the executable next to `Data.hq` and `Data` in the game folder, and run it. A dialog asks "Do you want packing?".

- **No** unpacks. The individual files appear under `unpacked/` and the file list is saved to `unpacked.info`. The original `Data.hq` and `Data` are deleted.
- **Yes** packs. The files under `unpacked/` are joined in the order listed in `unpacked.info` to make a new `Data.hq` and `Data`, and then `unpacked/` and `unpacked.info` are deleted.

It is safer to keep a copy of the two original files before unpacking. A Completed! message appears at the end and the program closes.


## How it works

**The encryption flips bit 0x10 of every byte.** For both files each byte is read, and 16 is subtracted when that bit is set or added when it is clear. Applying the same transform again restores the original, so unpacking and packing share the same transform. Below is the part that encrypts the header (`i=0`) and the body when packing.

```gml
byte = file_bin_read_byte(file)
if(i=0)
{
    if(byte=45){byte=29}
    if(byte=62){byte=26}
}
if(byte & 16)
{
    file_bin_write_byte(file2, byte-16)
}
else
{
    file_bin_write_byte(file2, byte+16)
}
```

**The header is cut up with string functions.** The decoded `Data.hq` is a single line of text: `총라인수 : [file count]` followed by `name<TAB>[size]<TAB>[offset]` entries. In the encrypted file the entries are separated by the two bytes `0D 0A`, which the transform turns into the control characters `1D 1A`, awkward to handle as text. So when unpacking these are replaced with `->`, and `string_pos` then finds `->`, tabs, and brackets to read each name, size, and offset. When packing, as in the code above, `-` and `>` are turned into 29 and 26 before the transform so they come out as `0D 0A` again.

**The body is sliced from the offset for the size.** `Data` is the file contents joined in header order, so the first file's offset is 0. For each file the program seeks to its offset, copies that many bytes, and creates the folder under `unpacked/` when the name contains one. When packing it measures the sizes in `unpacked.info` order, accumulates the offsets while writing a new header, joins the contents, and encrypts both files with the same transform.


## Files

| Path | Content |
|---|---|
| `source/hq-packer.gmk` | Original project file |
| `source/split/` | Text tree produced by GmkSplitter |
| `docs/hq-format.md` | HQ format analysis |
| `docs/screenshots/` | Screenshots |
| Releases | Executable and usage notes |


## License

zlib. See [LICENSE](LICENSE).
