THIS IS THE TRANSLATED VERSION OF AN OLD README
ScratchEdit3DS (prototype)

Touchscreen editor for the Nintendo 3DS (homebrew) featuring multiple sprites, a costume editor (pixel art), a sound editor, and a block editor that saves projects in .sb3 format (Scratch 3). This is not a port of the official Scratch editor: the official editor (Scratch 3) is written in JavaScript/HTML and cannot fit on a 3DS, so this is a new editor written in C using libctru + citro2d that generates .sb3 files.

    Unofficial project; "Scratch" is a trademark of MIT / the Scratch Foundation.

Status: Read Before Use
Component	Status
.sb3 writer (source/sb3_writer.c)	Uncompressed ZIP containing project.json + PNG/WAV files named by MD5, with MD5/CRC32/PNG implemented from scratch. Verified with the real Scratch 3 engine: the sample project passes scratch-parser (official schema), loads in scratch-vm, and when the green flag is clicked, sprites move and variables change as they do in the editor; all 114 opcodes in the table appear in all_blocks.sb3, and tools/sb3check.py checks references, MD5 hashes, and reachability. I have not opened it in the Scratch web interface.
.sb writer (Scratch 1.x, sb_writer.c)	No longer used by the editor; retained with its tests. Not verified against Scratch 1.4.
Editor, interpreter, assets, recorder, importer	Tested on PC (tools/test_*.c).
Interface (source/main.c)	Touchscreen logic was tested on PC by compiling main.c with mock libraries (tools/test_ui.c). Not compiled against the real libctru/citro2d libraries or tested on a console.
Microphone (mic* from libctru)	Written from memory based on the API; a simulated microphone is used in tests. 
Numeric keypad (SWKBD_TYPE_NUMPAD)	Calls follow the libctru API; a simulated keyboard is used in tests.
Audio (ndsp)	Written according to the libctru API, but untested. Requires sd:/3ds/dspfirm.cdc, including in emulators (the emulator's sdmc/3ds/ folder); if ndspInit fails, the editor displays the error code in the status line.

Differences when exporting to Scratch 3 (format limitations, not interface limitations):

forever if is saved as forever { if }; loud? is saved as loudness > 10; variables and lists are global (belonging to the Stage), and lists start empty; costumes are saved as 64×64 PNGs (each editor pixel becomes 2×2 pixels); synthesized tones are saved as WAV files. Pen and music blocks use Scratch 3's pen and music extensions, which are enabled automatically when blocks of those types are present. touching color / color is touching are simulated (see below).
Building

    Install devkitPro and the libraries: sudo dkp-pacman -S 3ds-dev (includes citro2d/citro3d).
    Set export DEVKITPRO=/opt/devkitpro and export DEVKITARM=$DEVKITPRO/devkitARM.
    Run make → generates scratch3ds.3dsx.
    Copy scratch3ds.3dsx to sd:/3ds/ and launch it from the Homebrew Launcher.

CIA (installable with FBI): In addition to the above, you need makerom and bannertool in your PATH (binaries from 3DSGuy/Project_CTR and Epicpkmn11/bannertool). Then run make cia to generate scratch3ds.cia (uses resources/icon.png, banner.png, audio.wav, and app.rsf). Install it with FBI. Audio still requires sd:/3ds/dspfirm.cdc. The .rsf and make cia have not been tested: if makerom complains about a key, the problem is in the .rsf file.

Projects are saved to sd:/3ds/scratch-editor/<name>.sb3 and can be opened in Scratch 3 (scratch.mit.edu or the desktop editor: File → Load from your computer).
Usage

The bottom screen has three tabs (Scripts, Costumes, Sounds), followed by the sprite selector and, at the bottom, the Flag / Stop / Save / Del / Undo / Redo / Dup buttons. The top screen displays the Stage (Scripts), an enlarged costume (Costumes), or the waveform (Sounds).

Sprites (up to 12): < and > switch sprites, + creates a new one, - deletes the current one, and tapping the name opens the sprite list. The Spr button opens the sprite menu: rename, duplicate (copies scripts, costumes, and recordings), move within the list / layer order, edit x, y, direction, size (%), and visibility (initial values saved in the .sb3), and import.

Backdrops (Stage): Tap the sprite name and select Stage (backdrops) at the bottom of the list. With the Stage selected, the Costumes tab edits backdrops (same canvas, palette, New, Name, Flip, Delete; the selected backdrop is displayed), and Sounds edits its sounds. The first backdrop is white; transparent areas appear white. < > / L R return to the sprites. The Stage has no scripts: pressing Flag while the Stage is selected switches the editor to the first sprite. In the .sb3, each backdrop is saved as a 480×360 SVG.

Importing sprites from a folder (for example, one created by a game or a PC tool):

Place folders in sd:/3ds/scratch-editor/sprites/ (created by the editor), then use Spr → Import sprite... to import one sprite or Import ALL to import them all.

sprites/
  Hero/              one folder = one sprite named "Hero"
    sprite.txt       optional: name=  x=  y=  direction=  size=  visible=  costume=  (key=value lines, # comments)
    walk1.png        .png / .bmp / .ppm images -> costumes, alphabetically (max. 6)
    walk2.png
    jump.wav         .wav PCM 8/16-bit, mono or stereo -> sounds (max. 6; reduced to ≤ 11025 Hz and 4 seconds)
  coin.png           a standalone image = a sprite with one costume

Images are resized to fit within 32×32 pixels while preserving their aspect ratio, and converted to the 15-color palette. Alpha values below 128 are transparent; for BMP/PPM, magenta #FF00FF is also transparent. Interlaced PNGs and compressed BMPs are not supported. Files that fail to import are skipped, and the editor reports how many failed. Duplicate names are numbered (Hero, Hero2, ...). tools/make_fixtures.py DIR generates an example folder.

Scripts: 9 categories (Motion, Looks, Sound, Pen, Control, Sensing, Operators, Variables, Lists) with 123 blocks (the 117 from Scratch 1.x, two PicoBoard blocks, three clone blocks, and pen down?, which do not exist in Scratch). Tap a block in the palette to add it after the selected block (hat blocks create a new script; forever, repeat, if, if else, etc. create their bodies with end/else markers). The ^/v buttons scroll the palette.

Arguments: The bottom bar of the script area displays the three input slots for the selected block. Tap a slot to focus it, then tap it again to edit it using the keyboard or a menu (costume, sound, key, message, variable, list, effect, pen color, sprite, etc.). If you tap a reporter (round, ( )) or Boolean (< >) block in the palette while a slot is focused, it is plugged into that slot, and you enter the reporter to fill in its own inputs (^ moves up one level). Delete removes the plugged-in reporter. Reporters can be nested up to 16 levels per script.

Variables and lists (up to 10 variables and 4 lists): Use Make a variable / Make a list in the corresponding category. Each variable has a reporter block and set/change/show/hide blocks; lists have add/delete/insert/replace blocks, and so on.

Execution display: The purple Run:T / Run:B button in the sprite bar selects where the project runs.

    Run:T (default): the Stage is displayed on the top screen, and the bottom screen acts as a touchscreen panel (mouse).
    Run:B: the Stage fills the touchscreen (480×360 → 320×240, tapping = mouse), while the top screen displays positions, variables, the timer, and the answer.

The setting is saved in sd:/3ds/scratch-editor/settings.txt.

Run (Flag/Y): The touchscreen becomes a mouse input panel; the D-pad, A, X, Y, L, and R act as keys; B stops execution. Supported features include pen blocks (trails and stamps), sensing blocks (mouse, keyboard, timer, ask/answer, microphone volume, when loudness > n), operators, lists, and broadcast and wait.

Costumes (up to 6 per sprite, 32×32 pixels, 16-color palette): Tap a color and draw on the canvas with the pencil (Pen) or fill an area (Fill). The x color erases. Flip flips the image, Clear clears it, New creates a costume by copying the current one, Name renames it, and < > switch costumes.

Sounds (up to 6 per sprite): Generate tones (sine / square / sawtooth / noise) with adjustable frequency, duration, and volume, or make microphone recordings. Rec (microphone) starts recording; tap it again to stop (maximum 4 seconds at 10910 Hz; recording stops automatically when full and discards recordings shorter than 50 ms). Recordings can be trimmed (Trim silence), normalized, reversed, and have their volume adjusted. They are saved in the .sb3 as WAV files and can be played with play sound.
Button	Action
L / R	Previous / next sprite
D-pad ↑ ↓	Move the block selection (Scripts tab)
D-pad ← →	Move focus between input slots
A	Edit the focused input slot
X	Delete (block, reporter, script, costume, or sound, depending on the tab)
(screen) Undo / Redo / Dup	Undo, redo, duplicate (see below)
Y / B	Run (green flag) / stop
START	Save .sb3
SELECT	Exit

Beside the script area, two ^ and v buttons scroll through pages without changing the selected block (along with Top and End).
Stage Boundaries

As in Scratch, a sprite can never leave the Stage completely: at least part of it (up to 15 px) always remains inside. Thus, forever move 10 stops at the edge instead of moving away forever.
Light / Dark Theme

D-pad left (when not running a project) switches the interface between dark and light themes. The Stage, costumes, and pen retain their colors. During execution, D-pad left continues to act as the project's left-arrow key. Inside a block, D-pad right advances to the next argument and, after the last one, returns to the first.
About Tab and Version History

The fourth tab, About, displays the name (ScratchEdit3DS), version, credits, controls, and legal notice (the version history is in CHANGELOG.md). The text is in source/about.h. To publish a new version, change APP_VERSION there (format 3.0.0BR: BR = beta release, BE = beta, NS = not stable, S = stable; see CHANGELOG.md) and add the corresponding entries. Scroll with ^ v or the D-pad.
Stage on the Bottom Screen, Eyedropper, and Moving Sprites

    D-pad up (when nothing is running): moves the Stage to the touchscreen. Drag to position the selected sprite; the position is saved as its initial position. Press D-pad up, B, or Done to return. (This is why D-pad up no longer selects the previous block: tap the block or use D-pad down, Top, and End.)
    In touching color and color is touching, tapping the color box opens an eyedropper: the Stage appears on the bottom screen, and tapping a pixel selects its color from the palette. List opens the usual color menu, and Cancel exits.

Pointer and Long Scripts

    go to mouse-pointer, point towards mouse-pointer, touching mouse-pointer?, and distance to mouse-pointer only work while the touchscreen is being touched. If no finger is touching the screen, the block is skipped (the sprite stays where it is) instead of moving to a corner.
    Below the palette, next to < > (previous/next script), the Top and End buttons jump to the first and last blocks of the script.

PicoBoard-Style Sensors

<channel> sensor value and sensor <…>? (Sensing category) read the console's inputs while the project is running:
Channel	Input
slider	Circle Pad horizontal axis (0–100, center 50)
resistance-A	Circle Pad vertical axis (0–100, center 50)
resistance-B / resistance-C	Touchscreen X / Y position (0–100)
sound	Microphone (same as loudness)
sensor button pressed?	A button
sensor A/B/C connected?	Always true (D is false)

This has not been tested on a real 3DS (hidCircleRead and touchscreen input have only been tested in the PC simulation).
Dragging Blocks (Touchscreen)

In the Scripts tab, you can drag blocks using the stylus or finger:

    Palette → script: Drop a block between two rows (a white line indicates where it will be inserted). Dropping a hat block in the script area creates a new script.
    Reporters/Booleans (operators, variables, etc.): Drop them onto an argument slot in the bottom bar or onto a script row (they are inserted into the first available compatible slot).
    Reordering: Drag a script row to another position. A C-shaped block (if, forever, etc.) moves together with its entire body. Dropping it inside itself does nothing. Dropping it directly below the header of a C-shaped block inserts it into that block.
    Deleting: Drag a row to the palette (it turns red).
    Near the top or bottom edge of the script area, scrolling happens automatically. A tap without movement still works as before (select/add). Hat blocks cannot be dragged.

    Tested only with PC tests (simulated screen); not run on a 3DS or emulator.

Undo, Redo, and Duplicate

    Undo / Redo (bottom buttons): undo and redo changes to the sprite's scripts (adding, deleting, moving, or duplicating blocks; plugging in reporters; changing a value; creating or deleting scripts). There are 20 history steps. A new edit clears the redo history; deleting or reordering sprites clears the entire history. Costumes, sounds, variables, and lists are not covered.
    Dup (bottom button): in Scripts, duplicates the selected block (including its body if it is a C-shaped block and any plugged-in reporters). If a hat block is selected, the entire script is duplicated. In Costumes and Sounds, the current costume or sound is duplicated.

Clones

In Control: when I start as a clone, create clone of [myself / sprite], and delete this clone. A clone copies the original's position, direction, size, costume, and effects; runs its sprite's scripts; responds to messages and key presses; and can touch or be touched by other sprites. Up to 16 clones can exist at once; they disappear when Stop (or stop all) is used. They are exported to .sb3 as standard Scratch 3 blocks (control_create_clone_of...).
Keyboard

Tapping an input slot already opened the system keyboard. Now, numeric slots and values in the sprite menu (x, y, direction, size, frequency, etc.) open the numeric keypad (digits, -, and .). In block input slots, the Abc button opens the full keyboard (for typing last, all, etc.).
Sound Effects

Sounds → Sound effects... adds one of 10 synthesized effects as a regular sound (exported as WAV):

laser, jump, coin, power-up, explosion, hit, blip, siren, fall, click.
Templates

Spr → New from template... (asks for confirmation and replaces the current project; save first):

    Blank project
    Catch the apples
    Pong
    Space shooter (clones)
    Pen sketch
    Music keys

These are regular projects with editable sprites, costumes, sounds, and scripts. In PC tests, they run in the interpreter and their .sb3 files pass scratch-parser validation. Five of the six loaded in the real Scratch engine (scratch-vm, without a display; collisions and bounces cannot be tested there), and create clone of was executed in the space shooter template. Music keys was not tested there. The prebuilt .sb3 files are in examples/templates/.
Prototype Limitations

    Maximum of 12 sprites, 8 scripts per sprite, 40 blocks per script, 6 costumes, and 6 sounds per sprite.
    The Stage supports backdrops (up to 6) and sounds, but no scripts. Backdrops are 32×32 pixels stretched to 480×360, and costumes are saved enlarged to 64×64.
    touching color / color touching color only check sprites and the backdrop (not the pen trails, stamps, or rotation); visual sprite rotation is not simulated.
    Excluded: video blocks. PicoBoard blocks (… sensor value, sensor …?) are simulated using the console (see above) and do not exist in Scratch 3: they are omitted when exporting to .sb3 (the default value remains); they are saved in Scratch 1.x .sb files. Clones are not part of Scratch 1.x (they were added at the user's request).
    Saves .sb3 (Scratch 3); does not open existing project files, only creates and saves new ones.

Testing and Format Verification (on PC)

cd tools
./run_tests.sh                                  # editor, assets, importer, .sb/.sb3 writers, and interface with mock libraries
python3 sb3check.py ../examples/all_blocks.sb3 --coverage   # validates .sb3: ZIP, MD5, block references, opcodes
# optional, with the real Scratch 3 engine (requires npm i scratch-vm scratch-storage):
node sb3_vm_run.js ../examples/sample.sb3       # loads the project, clicks the green flag, and displays positions and variables

Alternative for Playing Projects

If you want to play Scratch projects on the 3DS, there is Scratch Everywhere! (a runtime for .sb3). You can create the project on a PC using Scratch 3 and copy it to 3ds/scratch-everywhere.
Project Structure

source/project.h      data model + BLOCK_LIST table (all 123 blocks)
source/assets.c/h     palette, costumes, sound synthesis, recorder/trimming/normalization (no libctru)
source/importer.c/h   imports sprites from folders (PNG with custom inflate, BMP, PPM, WAV, sprite.txt)
source/editor.c/h     project editing: scripts, reporters, variables, lists (no libctru)
source/runtime.c/h    cooperative multi-sprite interpreter with clones (no libctru)
source/templates.c/h  project templates (built using the same editing API)
source/sb3_writer.c/h .sb3 serializer (ZIP + project.json + PNG/WAV)
source/sb_writer.c/h  Scratch 1.x .sb serializer (no longer used by the editor)
source/main.c         3DS interface (citro2d, ndsp, microphone)
tools/                sb3check.py, sb3_vm_run.js, sbdump.py, PC tests, run_tests.sh, stubs/ (mock libctru/citro2d)
examples/             sample.sb3 (catch-the-ball game), all_blocks.sb3 (one block of each type), and the same projects in .sb format
