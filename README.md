# TERA Arise World Data

This repository contains world geometry and path data for the TERA Arise server.

Due to the nature of the data, this is a very large repository; consider a
[shallow clone](https://git-scm.com/docs/git-clone#Documentation/git-clone.txt---depthltdepthgt)
or
[shallow submodule](https://git-scm.com/docs/git-submodule#Documentation/git-submodule.txt---depth)
if possible.

## Zone Geometry Data (ZGD) Format

Every zone (as defined by `AreaList.Continent.Area.Zones.Zone` in the data
center) has a corresponding zone geometry data file describing its accessible
space. The file name is given by the `x` and `y` attributes on the `Zone` node.
For example, `x1002y1006.zgd` would be the zone geometry file for the singular
zone of the `Velika Eastern Gate` area.

Every zone consists of 120x120 squares. Every square consists of 8x8 cells. A
given cell contains a number of volumes describing the accessible space in that
cell. A volume consists of a Z (vertical) position and a height; its width is
the same as the width of the cell.

A cell's width is equivalent to 16 in-game position units. For comparison, an
in-game 'meter' is equivalent to 25 in-game position units.

Zone geometry data files are compressed with the
[Brotli algorithm](https://datatracker.ietf.org/doc/html/rfc7932) configured for
maximum compression. The format after decompression can be described with this
C-like structure:

```c
struct ZoneGeometryData
{
    Square squares[120, 120];
};

struct Square
{
    Cell cells[8, 8];
};

struct Cell
{
    uint8_t volume_count;
    Volume volumes[volume_count];
};

struct Volume
{
    int16_t z;
    uint16_t height;
};
```

Note that:

* In order to improve compression, the `z` and `height` fields on `Volume` are
  written as differences from the corresponding values on the volume written
  prior to the current volume, or zero.

## Area Path Data (APD) Format

Every area (as defined in `AreaList.Continent.Area` in the data center) has a
corresponding area path data file describing the possible paths that non-player
units can take when doing non-trivial navigation through the world. The file
name is given by the `name` attribute on the `Area` node. For example,
`vk_dq_sd_03_p.apd` would be the area path data file for the
`Velika Eastern Gate` area.

An area consists of a rectangle of zones. Multiple areas can use the same
underlying zone(s). Yet, importantly, one area might have more zones than
another, even if some are shared between the two. For this reason, path data
cannot simply be integrated into the zone geometry data files; it must be
generated per area.

For every square in each zone of an area, there is a number of nodes. These
nodes usually sit right in the center of a volume, on the floor. Pathfinding
generally works by finding a node near the moving unit and a node at the goal,
and then plotting a path through the edges that connect these nodes. Nodes can
be connected both across squares and across zones. The organization of nodes
into squares is arbitrary and primarily intended to enable quick searches for
start and end nodes.

Area path data files are compressed with the
[Brotli algorithm](https://datatracker.ietf.org/doc/html/rfc7932) configured for
maximum compression. The format after decompression can be described with this
C-like structure:

```c
struct AreaPathData
{
    uint32_t node_count;
    Node nodes[node_count];
    uint32_t zone_count;
    Zone zones[zone_count];
};

struct Node
{
    float position[3];
    uint8_t directions;
    uint32_t edge_node_indexes[popcount(directions)];
};

struct Zone
{
    uint32_t position[2];
    Square squares[120, 120];
};

struct Square
{
    uint32_t node_index;
    uint8_t node_count;
};
```

Note that:

* The `position` field on `Zone` contains the `x` and `y` values used to locate
  the relevant zone geometry data file.
* The `node_index` and `node_count` fields on `Square` together represent the
  range of nodes for the square that should be read from the `nodes` field.
* The `position` field on `Node` is measured in in-game position units.
* The `directions` field on `Node` has one bit for every secondary intercardinal
  direction, starting from west-southwest and going counterclockwise, indicating
  whether a node index (i.e. an edge) is present for that direction.
