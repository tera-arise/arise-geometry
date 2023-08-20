# TERA Arise World Geometry Data

This repository contains world geometry data for the TERA Arise server.

Due to the nature of the data, this is a very large repository; consider a
[shallow clone](https://git-scm.com/docs/git-clone#Documentation/git-clone.txt---depthltdepthgt)
or
[shallow submodule](https://git-scm.com/docs/git-submodule#Documentation/git-submodule.txt---depth)
if possible.

## Details

Every zone (as defined by `AreaList.Continent.Area.Zones.Zone` in the data
center) has a corresponding zone geometry file describing its accessible space.
The file name is given by the `x` and `y` coordinates on the `Zone` node. For
example, `x1002y1006.zone` would be the zone geometry file for
`Velika Eastern Gate`.

Every zone consists of 120x120 squares. Every square consists of 8x8 cells. A
given cell contains a positive number of volumes describing the accessible space
in that cell. A volume consists of a Z (vertical) position and a height; its
width is the same as the width of the cell.

A cell's width is equivalent to 16 in-game position units. An in-game 'meter' is
equivalent to 25 in-game position units.

## Format

Zone geometry files are compressed with zlib's Deflate algorithm configured for
maximum compression. The format after decompression can be described with this
C-like structure:

```c
struct Zone
{
    Square squares[120][120];
};

struct Square
{
    Cell cells[8][8];
};

struct Cell
{
    uint8_t volume_count;
    Volume volumes[volume_count];
};

struct Volume
{
    uint16_t z;
    uint16_t height;
};
```

Note that, in order to improve Deflate compression, the `z` and `height` fields
are written as differences from the corresponding values on the volume written
prior to the current volume, if any.
