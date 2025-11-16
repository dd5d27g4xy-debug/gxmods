# GX Mods AI Coding Assistant Instructions

## Project Overview
GX Mods is a collection of customization packages for Opera GX browser. Each mod is a self-contained extension that modifies the browser experience through combinations of:
- Background music (vertical remixing with multiple layers)
- Keyboard/browser sounds
- Wallpapers (light/dark, static/animated)
- CSS themes (color customization for light/dark modes)
- Web modding (CSS for specific websites like Discord)
- Shaders (GPU-accelerated visual effects in SkSL language)

## Mod Structure

Every mod follows this standardized structure:
```
ModName/
├── manifest.json          # Required: extension metadata and payload definition
├── icon_512.png           # Required: 512x512 PNG icon
├── license.txt            # Required: license information
├── music/                 # Background music (optional) - MP3, joint stereo, VBR 145-180kbps
├── keyboard/              # Keyboard sounds (optional) - WAV preferred
├── sound/                 # Browser sounds (optional) - MP3
├── shaders/               # Visual effects (optional) - SkSL format in .txt files
├── wallpaper/             # Background images (optional) - JPG preferred
├── css/                   # Theme files (optional) - applied to specific websites
└── webmodding/            # Web page CSS modifications (optional)
```

Reference examples: `mods/Cozy/`, `mods/Cyberdeck/`, `mods/GX_Aura_Takeover/`

## Key Patterns

### manifest.json Schema
- Always use `manifest_version: 3` and `schema_version: 1`
- Include developer name in `developer.name`
- Sound/music references use paths relative to manifest location
- Theme colors use HSL format: `{ h: 0-360, s: 0-100, l: 0-100 }`
- Page styles match URLs via glob patterns (e.g., `https://*.discord.com/*`)

### Sound Conventions
- **Browser sounds**: Must cover 13 event types (`CLICK`, `TAB_INSERT`, `TAB_CLOSE`, `HOVER`, etc.)
  - Can use empty string `""` to skip an event (uses default)
  - Mixing empty strings with actual sounds is valid
- **Keyboard sounds**: Support 4 types (`TYPING_LETTER`, `TYPING_ENTER`, `TYPING_SPACE`, `TYPING_BACKSPACE`)
  - Multiple sounds per type allowed (played in order, cycling)
  - Use WAV format for zero-delay responsiveness
- **Background music**: Support vertical remixing via multiple layers
  - Single layer works but volume won't scale with user activity
  - List same file multiple times to increase perceived volume

### Shaders (SkSL)
- Written in SkSL (Skia Shading Language), stored in `.txt` files
- Critical uniforms available: `iChunk` (input texture), `iChunkSize`, `iContentSize`, `iMouse`, `iTick`, `iDate`
- **Must not declare custom uniforms** - only use engine-provided ones
- Animation defined in manifest via `animation: { duration: seconds, steps: total_frames }`
- Deprecated uniform: `iFrame` (will be removed; use `-opera-args()` instead)
- Performance optimization: Set frame rate to match content update (e.g., clock shader runs at 1fps, not 60fps)

### Wallpapers
- Provide light AND dark versions (mandatory, can't block user theme switching)
- Minimum 1920x1080 resolution
- Animated: Keep at 1920x1080 max, use VP9 format if possible, keep short/looped
- Video wallpapers must include `first_frame` image (JPEG)
- Static: Use JPG format to minimize mod size
- Specify `text_color` and `text_shadow` for start page content readability

### Themes
- Always provide light AND dark versions
- Use HSL color picker (avoid extreme light/dark values)
- Colors map to: `gx_accent` (primary) and `gx_secondary_base` (secondary)

## Critical Guidelines

### Resource Optimization
- Music: MP3 format only, joint stereo, VBR 145-180kbps (follow Mod_Template volume levels)
- Keyboard sounds: Start immediately, no silence prefix (critical for responsiveness)
- Icons: 512x512 PNG for future-proofing
- All audio: Follow volume standards from `documentation/Mod_Template/`

### Wallpaper Safety
- Ensure important content avoids center (covered by UI elements)
- Test different aspect ratios and lower resolutions
- See `documentation/GXWallpaperGuidelines.pdf` for safe zones

### Shaders
- Always optimize for low GPU usage
- Check `documentation/shaders.md` for detailed SkSL reference
- Provide non-animated alternatives if animating (user choice for performance)
- Don't use `iMouse` unless necessary (performance cost)

## Development Workflow

### Live Preview Shaders
Use `tools/livepreview/livepreview.py` for real-time shader development:
```bash
$ cd tools/livepreview
$ ./livepreview.py /path/to/shader/directory
# Opens http://127.0.0.1:8888 with live reload
```
Shaders appear in both static and animated formats. Modify code, save, refresh in Opera GX.

### Loading in Opera GX
1. Open `opera:extensions`
2. Enable _Developer mode_
3. Click _Load unpacked_, select mod directory
4. View in `opera:mods`
5. Pack for distribution: _Pack extension_ → produces `.crx` file

### Distribution
- Share `.crx` files directly (drag & drop into Opera GX)
- Upload zipped mods to GX.store via https://create.gx.games/mods

## Web Modding Integration
- CSS rules in `page_styles` apply only to matched URL patterns
- Can reference Opera GX UI colors via CSS variables (check `webmodding/opera.css` template)
- Common target: Discord (`https://*.discord.com/*`) - most mods customize it

## When Creating/Editing Mods
1. Start from `documentation/Mod_Template/` as boilerplate
2. Follow guidelines in `documentation/guidelines.md` and `documentation/mods.md`
3. Test all components (sounds, themes, shaders) in actual Opera GX browser
4. Verify light/dark mode versions work correctly
5. Keep total mod size reasonable (especially wallpapers/videos)
6. Credit all third-party resources and include appropriate licenses
