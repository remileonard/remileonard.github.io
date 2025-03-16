---
layout: post
title:  "Strike Commander Remake, Let's dive into the mission"
date:   2025-03-16 12:18:59 +0100
categories: LibRealSpace
use_mermaid: true
---


When I wanted to integrate *Strike Commander* maps into my port of *Flight* (SGI Dogfight), I initially managed to load the map background but without the objects. Upon examining the code, I noticed that the objects were identified but not loaded. So, I modified the code to handle object loading and display. It was during this first execution that I noticed something strange: the objects were indeed present, but they appeared to be in the wrong positions.

Here's how I solved this problem.

<!--more-->

## The Coordinate System

As is often the case, when encountering a bug, the root cause is likely to be found... between the chair and the keyboard. To ensure that I hadn’t made any mistakes, I needed to better understand the inner workings of the system to verify that the data matched what I was using for display.

### First Clues:

In the December 1994 issue of [GDM](/assets/docs/GDM_December_1994.pdf), Wayne Sike analyzes the coordinate system used in *Pacific Strike* and provides initial documentation of the various *chunks* encountered in the game's IFF files.

Using the article's information, I implemented a first version. In doing so, I discovered a key difference from the original code: coordinates were not stored as 16-bit integers but as 24-bit integers for two axes (X, Y), while the Z-axis was 16-bit. The article also mentioned that coordinates are relative to the game zones where objects should be positioned. I implemented the data structures to store these zone coordinates accordingly.

Once everything was recompiled, a problem arose: some objects referenced a zone with an ID of 255. Since this zone didn’t exist, the program crashed spectacularly. The issue was that not all objects used relative coordinates. Some had fixed coordinates, which was signaled by a zone ID of 255.

After fixing this bug, everything seemed good to go—though the article might suggest otherwise! 😉


### Endianness

*Strike Commander* is a PC game designed for [x86](https://en.wikipedia.org/wiki/X86) processors, which use little-endian representation for internal data. However, in the game's IFF files, chunk sizes are stored in big-endian format.

Did I mention the problem between the chair and the keyboard?

Since some data was already loaded in big-endian format, I implemented data loading in big-endian. Big mistake! When I checked the coordinate values, everything was completely nonsensical. I turned to my trusty hex editor, [HxD](https://mh-nexus.de/en/), and took a closer look. One of the editor's features allowed me to convert data from big to little-endian. Once I did that, everything started making sense. I updated my code to handle endianness correctly.


### Y or Z?

The objects were now visible on the map, but they were still misplaced. The issue stemmed from the representation of the X, Y, and Z axes. *RealSpace* uses the Z-axis for height, while OpenGL uses the Y-axis for the same purpose. After swapping the axes, I finally had objects on the ground instead of floating.

Still, something wasn’t right. Some objects appeared to be correctly placed—especially those near the center of the map—but as I moved further away, they seemed to be on the opposite side of their intended locations.


### Drawing the Map

To resolve this final issue, I decided to use the game itself for reference. The game holds “the truth,” and by correlating in-game data with map positions, I could determine the correct placement for objects.

My first step was to get the map. Easy enough—I launched the game and took a screenshot.

Next, I delved into the coordinate system. Returning to Wayne Sike’s article, I noted that the center of the map corresponds to (0,0) and that each block represents a 20,000-unit square.

![Drawing the map](/assets/img/mauritania.png)

With the map in hand, I compared my coordinate system with the game’s. This allowed me to confirm that the values were being loaded correctly and to pinpoint the source of the positioning issue: the Z-axis. 

In my coordinate system, which uses a right-handed frame of reference, negative Z-values indicate forward movement. Once I inverted the Z-values, everything fell into place—the objects were finally in their correct locations.

Well... almost. But that's a story for another time, when we tackle the mission script system!


## To conclude, a small diagram

Here is a snippet of the class diagram for the implementation of the mission system in *Strike Commander*.

<div class="diagram-container" id="diagram-container">
<pre class="mermaid">
classDiagram
    
    class MISN:::RSFILE {
        uint16_t version;
        uint8_t info[];
        uint8_t tune;
        std::string name;
        std::string world_filename;
        vector~AREA~ areas;
        vector~SPOT~ spots;
        vector ~string~ messages;
        vector ~uint8_t~ flags;
        CAST * casting[];
        uint8_t prog[];
        uint8_t nums[];
        MISN_PART * parts[];
        uint8_t team[];
        uint8_t scenes[][];
        uint8_t load[];
    }

    class AREA:::DATATYPE {
        int id;
        unsigned char AreaType;
        char AreaName[33];
        long XAxis;
        long YAxis;
        long ZAxis;
        unsigned int AreaWidth;
        unsigned int AreaHeight;
        unsigned char Unknown[5];
    }
    class MISN_PART:::DATATYPE {
        uint8_t id;
        std::string member_name;
        std::string member_name_destroyed;
        std::string weapon_load;
        uint8_t area_id;      
        uint8_t unknown1;
        uint16_t unknown2;
        int32_t x;
        int32_t y;
        uint16_t z;
        uint16_t azymuth;
        uint16_t roll;
        uint16_t pitch;
        uint8_t> unknown_bytes;
        RSEntity *entity;
        bool alive;
    }
    class SPOT:::DATATYPE {
        int id;
        short unknown;

        long XAxis;
        long YAxis;
        long ZAxis;
    }
    class MSGS:::DATATYPE {
        char message[255];
        int id;
    }
    class CAST:::DATATYPE {
        std::string actor;
        RSProf *profile;
    }

    MISN --> AREA
    MISN --> SPOT
    MISN --> CAST
    MISN --> MISN_PART
    MISN --> MSGS

        class CHLD:::DATATYPE {
        std::string name;
        int32_t x;
        int32_t y;
        int32_t z;
        uint8_t data[];
        RSEntity *objct;
    }
    class EXPL:::DATATYPE {
        std::string name;
        int16_t x;
        int16_t y;
        RSEntity *objct;
    }
    class WDAT:::DATATYPE {
        uint16_t damage;
        uint16_t radius;
        uint8_t unknown1;
        uint8_t weapon_id;
        uint8_t weapon_category;
        uint8_t radar_type;
        uint8_t weapon_aspec;
        uint32_t target_range;
        uint8_t tracking_cone;
        uint32_t effective_range;  
        uint8_t unknown6;
        uint8_t unknown7;
        uint8_t unknown8;
    }
    class DYNN_MISS:::DATATYPE {
        uint32_t turn_degre_per_sec;
        uint32_t velovity_m_per_sec;
        uint32_t proximity_cm;
    }
    class WEAPS:::DATATYPE {
        int nb_weap;
        std::string name;
        RSEntity *objct;
    }
    class HPTS:::DATATYPE {
        uint8_t id;
        int32_t x;
        int32_t y;
        int32_t z;
    }
    class MapVertex {
        Point3D v;

        uint8_t flag;
        uint8_t type;
        uint8_t lowerImageID;
        uint8_t upperImageID;

        float color[4];

    }
    MapVertex --> Point3D

    class BoudingBox {
        Point3D min;
        Point3D max;
    }

    BoudingBox --> Point3D

    class UV {
        uint8_t u;
        uint8_t v;
    }
    class uvxyEntry {
        uint8_t triangleID;
        uint8_t textureID;
        UV uvs[3];
    }

    class Triangle {
        uint8_t property;
        uint8_t ids[3];
        uint8_t color;
        uint8_t flags[3];
    }
    class Lod {
        uint32_t dist;
        uint16_t numTriangles;
        uint16_t triangleIDs[256];
    }
    class RSEntity:::RSFILE {
        RSImage * images[];
        Point3D vertices[];
        uvxyEntry uvs[];
        Lod lods[];
        Triangle triangles[];
        WEAPS * weaps[];
        HPTS * hpts[];
        CHLD * chld[];
        enum Property;
        EXPL *explos;
        int32_t thrust_in_newton;
        int32_t weight_in_kg;
        WDAT *wdat;
        DYNN_MISS *dynn_miss;
        bool gravity;

        uint16_t life;
        map[string, map[string, uint16_t] sysm;
        Point3D position;
        Quaternion orientation;
        bool prepared;
    }

    RSEntity --> WDAT
    RSEntity --> DYNN_MISS
    RSEntity --> WEAPS
    RSEntity --> HPTS
    RSEntity --> CHLD
    RSEntity --> EXPL
    RSEntity --> Triangle
    RSEntity --> Lod
    RSEntity --> uvxyEntry
    RSEntity --> MapVertex
    RSEntity --> BoudingBox
    RSEntity --> Point3D
    RSEntity --> Quaternion
    RSEntity --> RSImage

    MISN_PART --> RSEntity
    CHLD --> RSEntity
    EXPL --> RSEntity
    WEAPS --> RSEntity
    uvxyEntry --> UV

    MISN_PART --> AREA
    MISN_PART --> CAST

    class RSArea:::RSFILE {
        std::vector~MapObject~ objects;
        std::vector~AreaOverlay~ objectOverlay;
        float elevation[BLOCKS_PER_MAP];
        AreaBlock blocks[NUM_LODS][BLOCKS_PER_MAP];
    }

    class MapObject{
        char name[9];
        char destroyedName[9];
        int32_t position[3];
        RSEntity* entity;
    }

    class AreaBlock{
        size_t width;
        size_t height;
        int sideSize;
        MapVertex vertice[400];   
    }
    class AreaOverlayTriangles {
        int verticesIdx[3];
        uint8_t color;
        uint8_t u0, u1, u2, u3, u4;
        uint8_t u5, u6, u7, u8, u9;
        uint8_t u10,u11;
    }
    class AoVPoints {
        int x;
        int y;
        int z;
        int u0;
        int u1;
        int u2;
    }
    class AreaOverlay {
        AoVPoints* vertices;
        AreaOverlayTriangles trianles[400];
        int lx, ly, hx, hy;
        int nbTriangles;
    }
    MapObject --> RSEntity
    RSArea --> MapObject
    RSArea --> AreaBlock
    RSArea --> AreaOverlay
    AreaOverlay --> AoVPoints
    AreaOverlay --> AreaOverlayTriangles
    AreaBlock --> MapVertex
    MISN --> RSArea

    classDef RSFILE fill:#f96;
    classDef DATATYPE fill:#f66,stroke:#333,stroke-width:4px;

</pre>
</div>

Merci pour votre lecture, à bientôt.