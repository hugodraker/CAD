ICOEDTV - Vector & GDI Polygon Icon Editor
`ICOEDTV` is a lightweight, pure C vector drawing tool and icon generator tailored for Windows GDI. Unlike traditional bitmap icon editors, `ICOEDTV` constructs graphics out of vector polygons, polylines, lines, text nodes, and hierarchical external references (`EXT_REF`). It saves graphic databases directly as C source code snippets (`case N: { ... }`) designed for direct inclusion into GDI-based Windows applications.
This documentation covers both the Win32 GCC build and the Win16 (Windows 3.1x / Open Watcom / Borland C++ 3.1) variant.
---
1. Key Features
GDI C Code Generation: Outputs native Windows GDI primitives (`Polygon`, `Polyline`, `MoveToEx`/`LineTo`, `CreateSolidBrush`, `CreatePen`).
Multi-Icon / Page Database: Store up to 20 icons/pages per `.c` file partitioned into switchable `case N:` blocks.
Hierarchical References (`EXT_REF`):
Re-use other vector files (C data files or SVGs) as sub-components with dynamic translation, rotation, and scaling.
Parameterized strings and text tags (`TAG_TEXT` / `TAG_TEXT_ROT`) pass variable values down through references.
Interactive pack/unpack tools (`ToRef` / `FromRef`).
Precision CAD Tools:
Interactive dimensioning system (`DIMENSION`) supporting aligned, horizontal, and vertical dimensions with configurable units (`mm`, `cm`, `m`, `inches`, `feet`, `feet-inches`).
Constraint tools: node alignment (horizontal/vertical), explicit distance/length setting, width/height scaling, and angle locking (`Lock Axis`).
Polygon boolean union / merging (`Merge`).
Import / Export Support:
Ingestion and path parsing for Scalable Vector Graphics (`.svg` paths, rects, and polylines).
Drag-and-drop file support (`WM_DROPFILES`).
---
2. Compilation & Toolchains
Win32 Target (GCC / MinGW)
Build using GCC (MinGW-w64 or classic MinGW32):
```sh
gcc -O2 -mwindows icoedtv.c -o icoedtv.exe -lcomdlg32
```
For debugging output with symbols:
```sh
gcc -g -mwindows icoedtv.c -o icoedtv.exe -lcomdlg32
```
Win16 Target (Windows 3.1x / Windows for Workgroups)
The Win16 version is tailored to run on 16-bit segmented real/protected mode memory models (Windows 3.1+). Because Win16 GDI handles 16-bit integer parameters for coordinates and lacks certain Win32 APIs, the following differences and flags apply:
Porting & Compatibility Notes
API Differences:
Win32 `SetWindowLongPtr(...)` with `GWLP_WNDPROC` is replaced with `SetWindowLong(..., GWL_WNDPROC, ...)`.
`MoveToEx` is used, or fallback `MoveTo` for earlier 16-bit targets.
Text rendering uses `GetTextExtent` instead of `GetTextExtentPoint32`.
`DragAcceptFiles` requires linking with `shell.lib` / `<shellapi.h>`.
Memory Model: Compile using the Large memory model (`-ml` / `/AL`) due to heap allocations and string caches.
Open Watcom C/C++ 16-bit (Recommended)
```sh
wcl -q -bt=windows -l=windows -ml -bcl=windows icoedtv16.c commdlg.lib shell.lib
```
Borland C++ 3.1 / 4.52 (Command Line)
```sh
bcc -W -ml -2 -O2 -w- icoedtv16.c
```
(Ensure `COMMDLG.LIB` and `SHELL.LIB` are in your library search path).
---
3. UI and Keyboard Controls
Toolbar Commands
Button	Function
Select/Edit	Select nodes or whole shapes; drag to move vertices or objects.
Rotate	Rotate selected shapes around their center of mass.
Scale	Proportionally scale selected elements.
Polygon / Line / Polyline	Interactive vector drawing modes. Click canvas to drop vertices.
Pan	Scroll canvas view area.
Flood Fill	Apply active fill color to closed polygons under the cursor.
Undo / Clear / Delete	Undo buffer (`Ctrl+Z`), blank active icon, or delete selected node/shape (`Del`).
Import SVG	Read SVG vectors into the current shape table.
Import Ref	Embed external `.c` or `.svg` file as an `EXT_REF` component.
Open .C / Save .C	Load and persist multiple `case N:` icon databases.
P<->L	Convert closed Polygon to open Polyline or vice versa.
Merge	Compute boolean union of two selected closed polygons.
Move Up / Move Down	Reorder z-index of shapes within the active icon.
Align Vert / Align Horz	Snap 2 selected nodes to the same X or Y plane.
Set Dist / Width / Height	Precise numeric editing via pop-up text field.
Set Angle	Numeric angle adjustment for the vector between 2 selected nodes.
Dimension	Place associative CAD dimension lines between points.
Page Size	Configure canvas dimensions, physical units, and paper bounds.
Tag Editor / Add Tag	Manage dynamic template strings.
(Lock Axis)	Lock dragging to constrained 90° / along existing segment axes.
Show All	Reset pan and zoom factors to default.
ToRef / FromRef	Extract selected shapes into an external component or unpack a reference.
Text	Place plain or rotated label text on the canvas.
Hotkeys & Mouse Actions
Left Mouse Click:
Click node / edge: Select element.
Click & drag: Move vertices, rotate, or scale depending on active mode.
Double-click edge: Insert node at cursor position along the polygon boundary.
Double-click text / tag / dimension: Open in-place editor.
Double-click palette swatches: Open `ChooseColor` common dialog.
Right Mouse Click:
On node: Delete vertex.
On drawing in progress: Finish current polygon or polyline.
On overlapping references: Cycle selection through stacked `EXT_REF` components.
Shift + Click: Add/remove vertices to selection or cycle text alignment (Left, Center, Right).
Ctrl + Drag: Constrain axis dragging (horizontal / vertical only) or duplicate shapes.
Enter: Finalize active drawing shape or accept numeric input in pop-up boxes.
Esc: Cancel current tool operation or dismiss edit boxes.
---
4. File Data Format Specification
`ICOEDTV` writes readable C files that double as code macros and visual data blocks. An exported C database follows this pattern:
```c
case 1: {
    PAGE_DEF("Main Title", 32, 32, "A4 (210x297)", "mm", 1);
    
    // Solid filled polygon
    HBRUSH hBr = CreateSolidBrush(RGB(128,128,128)); HGDIOBJ oBr = SelectObject(hdc, hBr);
    HPEN hPen = CreatePen(PS_SOLID, 1, RGB(0,0,0)); HGDIOBJ oPen = SelectObject(hdc, hPen);
    POINT p0[] = { PT(4,4), PT(28,4), PT(28,28), PT(4,28) }; POLY(p0);
    SelectObject(hdc, oBr); DeleteObject(hBr);
    SelectObject(hdc, oPen); DeleteObject(hPen);

    // Dimension line
    DIMENSION(0, 0, 0, 1, 3.5, 0.5, 0);

    // Dynamic Tagged Text
    TAG_TEXT(16, 16, "TAG1", C, 24);

    // Sub-component external reference
    EXT_REF(10, 10, "sub_part.c", "Custom Value|120V");

    break;
}
```
Macro Definition Helper Header
To compile or render output files directly in your own application, supply the following helper macros:
```c
#define PT(x, y) { (int)(x), (int)(y) }
#define POLY(pts) Polygon(hdc, pts, sizeof(pts)/sizeof(pts[0]))
#define L(x1, y1, x2, y2) do { \
    MoveToEx(hdc, (int)(x1), (int)(y1), NULL); \
    LineTo(hdc, (int)(x2), (int)(y2)); \
} while(0)
#define PAGE_DEF(title, gh, gw, paper, unit, scale) /* Drawing configuration */
#define DIMENSION(s1, p1, s2, p2, off, tpos, mode)  /* CAD dimension data */
#define TAG_TEXT(x, y, txt, align, size)            /* Text string */
#define TAG_TEXT_ROT(x, y, txt, align, size, rot)   /* Rotated text */
#define EXT_REF(x, y, path, ...)                    /* Nested reference */
```
---
5. Technical Limits & Data Structures
Parameter	Win32 Limit	Win16 Limit
Max Points per Shape	64	64
Max Shapes per Icon	40	40
Undo Levels	10	10
Max Stored Icons per File	20	20
Max Dimensions per Icon	32	32
Max External Ref Depth	3 nested levels	3 nested levels
Coordinate System	Double precision	Single precision / Fixed-point
Memory Handling	Dynamic (`calloc`/`free`)	Global/Local Heap (`HGLOBAL`)
---
6. License
This program has been released into the Public Domain. You may use, modify, distribute, and compile it for both commercial and non-commercial purposes without restriction.
