# Creator Hub

Your one-stop desktop app for content creation. Each feature is a self-contained **module**
that shows up automatically in the sidebar, on the Home page and in Settings.

**Current modules**

| Module | Section | What it does |
| --- | --- | --- |
| Stream Backdrop | Stream | Animated, audio-reactive backgrounds that show up live in OBS |
| Face FX | Stream | A 3D hood that tracks your head, layered over your webcam in OBS |
| Thumbnail Maker | Create | Thumbnails with text, images, screenshots and shapes; export PNG/JPG |
| Clip Generator | Create | Finds the best moments in a VOD and exports vertical 9:16 clips with captions |

## Run it

```bash
npm install
npm run dev        # development, with hot reload
npm run build      # production build into /out
npm start          # run the production build
npm run dist       # build a Windows installer into /dist
npm run typecheck  # check for type errors
```

## Using the Stream Backdrop in OBS

1. Open **Stream Backdrop** in Creator Hub and copy the address under **Connect to OBS**
   (normally `http://127.0.0.1:7420/backdrop`).
2. In OBS: **Sources → + → Browser**, paste the address, set the size to 1920×1080.
3. Move the source to the bottom of the list so it sits behind everything.

With **Live updates** on, every change appears in OBS instantly. Turn it off to tweak privately,
then press **Send to OBS**. **Pulse** and **Flash** play effects live. **React to audio**
moves the backdrop with your mic. To react to game or music audio, pick "Stereo Mix" or a
virtual audio cable as the input.

**Match my screen** makes the backdrop follow the colors of your game. Pick the screen (or window)
your game is on. Creator Hub samples a tiny 64×36 copy of it 4 times a second, picks out the main
colors and fades the backdrop to them (**Color fade** sets how smoothly). Nothing is recorded or saved.
**Keep these colors** freezes the current palette as your own. If the colors stay dark, use borderless
windowed mode in the game, or choose the whole screen instead of the game window.

Creator Hub must stay open while streaming, because it serves the page OBS loads.
The server only listens on your own PC (127.0.0.1), not on your network.

## Using Face FX (3D hood)

1. Open **Face FX**. Your webcam starts and MediaPipe tracks your head on the GPU (about 30 fps).
   Tracking runs while you're on this page, or whenever **Hood on stream** is switched on.
2. Pick a look: built-in hood presets (Ivory, Shadow, Crimson, Midnight) or custom colours, plus
   the shadow over the eyes. Or switch to **My 3D model** and load any `.glb` (hood, helmet, hat),
   with size, position and rotation sliders.
3. **Fit to your head** nudges size and position if the hood sits a little off.
4. In OBS, add a **Browser** source `http://127.0.0.1:7420/facefx` sized like your camera
   (e.g. 1920×1080), directly **above** your webcam source, then copy the webcam's transform onto it
   (right-click → Transform → Copy / Paste Transform). Turn on "Mirrored in OBS" if your webcam source
   is flipped. Add a small **Render Delay** filter to the hood if it moves before your head.
5. Toggle **Hood on stream** in the app, or with the Stream Deck **Hood On/Off** button (it fades in
   and out). **Next Hood Style** cycles presets.

Nothing leaves your PC: the camera is read locally, and only head positions go to the OBS page.
The built-in hood is generated in code (`src/shared/facefx/hood.ts`); its shape constants at the top
of that file are easy to tweak.

## Using the Thumbnail Maker

1. Pick a size (YouTube, Full HD, Square, Vertical) and a template, or start blank.
2. Add things with the left toolbar: **Text**, **Image** (or drag files in / paste with Ctrl+V),
   **Screen** (grabs a still from a screen or game window), **Box**, **Circle**, **Arrow**.
3. Click to select, drag to move (snaps to the center and edges; hold Alt to stop snapping),
   drag the corners to resize, and drag the round handle to rotate (hold Shift to snap).
   Double-click text to edit it.
4. Style the selected item on the right: outlines, highlight boxes, shadows/glow, image brightness, etc.
5. **Export** saves a JPG (smaller) or PNG (sharpest). **Copy** puts it on the clipboard.
   Exports over YouTube's 2 MB limit get a warning.

Shortcuts: Ctrl+Z / Ctrl+Y undo/redo · Ctrl+D duplicate · Delete remove · arrows nudge
(Shift = 10px) · `[` / `]` move back/forward · Esc deselect.

Everything autosaves. Images you add are stored once in the app's data folder (`assets/`).
To add a template, add a block to `src/renderer/src/modules/thumbnail/templates.ts`.

## Using the Clip Generator

Needs **FFmpeg full build** on your PATH (`winget install Gyan.FFmpeg`, which includes speech
recognition) and a one-time **466 MB speech model** download (the app offers it). Everything runs on
your PC; nothing is uploaded.

1. **Add video** (or drop a file anywhere on the page). Your VOD stays where it is; OBS MKVs play directly.
2. Analysis starts automatically. It extracts the audio, transcribes it on the GPU and scores every
   moment by loudness jumps plus hype phrases ("no way", "let's go", "clip that"…). About 3 s per
   minute of video on an RTX 5080.
3. Review the suggested clips (best first). Click one to preview it, drag the timeline handles to
   trim, and tick the ones to export. **Add clip here** makes one at the playhead.
4. **Edit layout** (once): drag the purple **Facecam** box over your webcam and the cyan **Gameplay**
   box over the action. Shapes stay locked to the vertical layout. "No facecam" on a clip lets
   gameplay fill the screen.
5. **Captions**: Bold Pop / Boxed / Clean, size and height. Each clip's words are editable.
6. **Export**: MP4 1080×1920, H.264 on the GPU, audio levelled toward -14 LUFS, saved to
   `Videos\Creator Hub Clips` (change it under "Save to").

How it works: `src/main/clips/` (FFmpeg jobs, transcription, export), `src/shared/clips/` (moment
scoring, captions and layout math, shared by preview and export), `src/renderer/src/modules/clips/` (UI).
Tune what counts as a highlight in `src/shared/clips/scoring.ts` (`DEFAULT_HYPE`, thresholds).

## Using Clip Studio

Record yourself over B-roll (or your screen) and cut it into a TikTok or a landscape video. It uses
the same FFmpeg and speech model as the Clip Generator. Background removal runs on your GPU with a
small (~16 MB) model downloaded on first use. Nothing is uploaded.

1. **New video** and pick **Vertical 9:16** or **Landscape 16:9**.
2. **Background tab**: choose a B-roll folder (subfolders are included), then click a clip. You can
   also use **Screen** (a window or monitor) or plain black.
   - **Start the B-roll at**: drag to the moment you want, then fine-tune with the ±0.1 / 1 / 10 s
     buttons. The preview holds that frame; **Preview** plays from it. Each take starts the B-roll
     there, and every clip remembers its own start point.
   - In **Edit takes**, the same control shifts the footage behind the selected take without moving
     your recording.
3. **Layout tab**:
   - **Cut-out** removes your background so you stand over the footage. Set size, position and edge softness.
   - **Split** stacks you and the footage (top/bottom, or side by side for landscape).
   - **No facecam** shows just the footage, full frame, with your voice. The camera stays off while
     you record (only the mic is used).
   - **Talking head scene**: just you, up close, with the gameplay blurred and darkened behind you.
     It's either a **Cut-out** or a rounded **Camera card**, with size, blur and darkening settings.
     Switch any take to it with **Main / Talking head** under the preview. To cut to it partway through
     a take, split the take first. Before recording, the same switch picks the scene for the next take.
4. **Script tab**: paste a script for the **teleprompter**. It scrolls while you record; set the speed and size.
5. **Record**: pick your camera and mic and hit **Record**. A 3-2-1 countdown runs, then press **Stop**.
   Each recording becomes a **take**, with the B-roll that was playing behind you.
6. **Edit takes** (the preview grows with the window, so maximise it for a bigger view):
   - The **whole-video timeline** under the preview shows every take, with text/pictures (purple) and
     zooms (orange) underneath. Click or drag anywhere on it to move the playhead.
   - Reorder or delete takes in the strip.
   - Trim each take on its own bar (drag the end handles; click or drag inside it to move the playhead).
   - **Split** (or press **S**) cuts the selected take in two at the playhead. To remove a section from
     the middle, split at its start, split at its end, then delete the middle piece. **Space** plays or pauses.
   - Swap a take's B-roll by clicking another clip in the Background tab.
   - Click **Play from here** to watch the whole thing.
7. **Overlays tab** adds text and **pictures** (PNG, JPG, WebP, GIF), or drop pictures straight onto the preview.
   - New items start at the playhead. Set **Starts at** and **Shows for**, or "Show until the end".
   - **Drag** an item on the preview to move it, and use the **mouse wheel** over it to resize.
   - Pictures can have rounded corners, a white border and a shadow. Later items sit on top; reorder
     them with the arrows.
   **Effects tab**:
   - **Zoom for emphasis**: "Add zoom at playhead", then choose how far (1.1–2.5×), on you or the
     centre, smooth or punch (instant), and how long. Text and captions don't zoom.
   - **Transitions** between takes: Cut, Fade (black), Flash (white) or Zoom, with a length. Pieces of
     one take stay jump cuts unless you tick "Also between split pieces".
   - **Remove pauses** finds silent gaps longer than the length you set and cuts them out, leaving
     0.15 s around your words. Raise "Counts as silence below" if background noise hides the pauses;
     lower it if quiet words get cut. **Undo** puts the takes back.

   **Captions tab** turns on auto captions (transcribed after each take, with editable words).
8. **Export video**: MP4 (1080×1920 or 1920×1080) with audio levelled toward -14 LUFS and B-roll audio
   mixed under your voice. It's saved to `Videos\Creator Hub Studio\Exports`. Takes are kept in
   `Videos\Creator Hub Studio\<project>` until you delete the project.
9. **Exported videos** (under your projects in Clip Studio) lists everything you've exported:
   - Click the thumbnail to play a video, or use the folder icon to show it in its folder.
   - The bin icon deletes a video by moving it to the Recycle Bin, so you can still get it back.
   - Deleting a project removes its recordings, then asks whether to also send its exported videos
     to the Recycle Bin.

How it works:
- `src/main/studio.ts`: B-roll scan, take recording and FFmpeg export.
- `src/shared/studio/types.ts`: layout math shared by preview and export.
- `src/shared/studio/effects.ts`: zoom and transition timing, used by both the preview and the
  FFmpeg export (zoompan / fade), so they match.
- `src/renderer/src/modules/studio/`: UI, WebGL preview compositor, recorder and background removal.

Each take is saved as camera | mask side by side, so the cut-out can be re-rendered at export.

## Using Voice (better mic audio)

Voice cleans up your mic: less room echo ("empty room" sound), quiet between words, a less boxy tone,
softer "s" sounds and a steadier level. It works **live** (OBS, Discord) and on **Clip Studio exports**.

1. **Setup.** Pick your mic (a HyperX QuadCast is picked automatically).
   - For live use you need the free virtual mic **VB-Cable**: press **Install VB-Cable**, click
     "Install Driver" in its window and allow the Windows prompt. Restart if it doesn't show up.
   - Set "Send my cleaned voice to" to **CABLE Input**. Then in OBS and Discord choose
     **CABLE Output (VB-Audio Virtual Cable)** as your microphone.
2. **Auto-tune.** Press **Start the test** and read the lines on screen (14 s, ending with one clap).
   - It measures your room's echo time, noise, level, boxiness and "s" sounds, then sets everything.
   - **Before / After** plays your test raw and cleaned.
   - Follow the tips: on a QuadCast, the pattern dial set to **cardioid** and talking into the
     **side** (the logo) from 10–15 cm away cut the room more than any software.
3. **Live.** The big **Voice** button (or the Stream Deck **Voice On/Off** key) turns it on.
   - The sliders and presets (Auto, Dry & close, Broadcast, Natural) change the sound live, and
     **Bypass** lets you compare with the raw mic.
   - It adds about 40 ms of delay. **Turn Voice on when Creator Hub opens** starts it automatically.
     It only runs while Creator Hub is open.
   - **Level** shows how much louder or quieter the cleaned voice is while you talk. **Match my mic**
     sets Volume so it's as loud as the raw mic. In auto-tune, **Keep my level** (the default) keeps
     your volume; **Stream level** brings you up to about -18 dB.
   - **Noise gate:** mutes the mic while you're not talking.
     - **Opens at** is the white line on the In meter. Set it above the meter when you're quiet and
       below it when you talk. **Set from my room** uses your measured noise.
     - **When closed** sets how much it turns down, up to silent.
     - It listens 8 ms ahead, so word starts aren't cut off.
4. **Videos.** With **Clean up my voice** on (Voice page, or Clip Studio → Effects), exports use your
   Voice settings. Recordings are never changed; cleaned tracks are cached, so re-exports are quick.

How it works: `modules/voice/voice.worklet.js` runs in real time.
- **De-reverb:** late-reverb suppression, which predicts the room's echo from its measured RT60 and
  turns it down.
- **Then:** a de-esser, a soft gate, and the compressor and look-ahead limiter.
- **Around it:** `chain.ts` adds the EQ, and runs the same chain live (`engine.ts`) or offline for exports
  (`studio/voiceClean.ts`).
- **Auto-tune:** `analyze.ts`.

## Using the Content Planner

Brainstorm, plan and keep track of your videos. Anything with a post date shows up on **Home** under
**Up next** (overdue, today, this week), so you see what's due as soon as you open the app.

1. **Ideas**: type an idea and press Enter. Typing `#tags` (e.g. `#valorant #shorts`) adds tags.
   - Search, filter by tag and star your favourites.
   - **Make it a plan** moves an idea onto the board and adds a standard checklist.
   - You can also save ideas from the **Quick idea** box on Home.
2. **Board**: drag plans through Planning → Scripting → Recording → Editing → Posted.
3. **Calendar**: plans by post date. Drag a plan to another day to reschedule it, and click an empty
   day to plan something for it. Undated plans sit on the right.
4. **Plan editor** (click any plan) has these fields:
   - Stage, post date, reminder ("Morning of" / "Day before 6 pm" presets), platforms and tags.
   - Hook, outline beats, script, B-roll notes, a checklist, and notes.
   - **Send to Clip Studio** starts a Clip Studio video with your hook and script in the teleprompter.
     After that the button says **Open in Clip Studio**.
5. **Reminders**: at a plan's reminder time, Windows shows a notification while Creator Hub is running.
   Click it to open the plan. Reminders missed while the app was closed show once at the next start.
6. **AI buttons**: **Brainstorm** in Ideas, plus **Suggest hooks**, **Suggest outline** and **Title
   ideas** in a plan. These run on the free local AI (see below), or on Claude if you switch the
   Planner to Claude in Settings → AI. **Ask the Planner** opens a chat with the Planner agent.

How it works:
- `src/shared/planner/types.ts`: plan data and the "up next" / reminder rules.
- `src/renderer/src/core/plannerStore.ts`: one shared copy of your plans for the Planner and Home.
- `src/main/planner.ts`: notifications, plus the AI buttons (on the shared AI engine).
- `src/renderer/src/modules/planner/`: the Ideas / Board / Calendar views and the plan editor.

## AI assistant and agents

Free AI help everywhere in Creator Hub. It runs on your own PC with **Ollama** (model `qwen3.5:9b`,
about 7 GB, on your GPU), so there's no account and no cost per use, and nothing leaves your computer.
To set it up, go to **Settings → AI assistant**: **Install Ollama**, then **Download** the model.

Open the **Assistant** with **Assistant** in the sidebar, **Ctrl+Space**, or the Stream Deck key. It opens
beside the page (the page shrinks to fit), so you can keep using Clip Studio's tools while you chat. Drag
its left edge to resize it; double-click the edge to reset. Pick an agent:

| Agent | Headspace | What it does |
| --- | --- | --- |
| **Producer** | Your right hand | Answers quick questions and hands real work to the others, e.g. "plan a follow-up and get the open video ready to post" |
| **Planner** | Content strategist | Ideas, hooks, outlines, posting cadence, calendar gaps; proposes new ideas and plans |
| **Editor** | Hands-on editor (thinks like creator *and* viewer) | Reads the transcript and pauses of the open Clip Studio video; proposes cuts, punch-in zooms, text callouts, talking-head switches and scripts; also tunes Clip Generator clips |
| **Coach** | Performance coach | Studies videos that perform well and explains *why* (hook, pacing, words, look); coaches your videos and builds your style profile |

- **Nothing changes until you click Apply.** Agents turn every change into a card: **Apply**,
  **Dismiss** or **Apply all**. Each applied card has **Undo**, which only reverses that one change.
- Agents see what you have open (shown under the agent picker), so "this video" or "this plan" just works.
- **Ask the Editor** and **Coach** buttons sit at the top of Clip Studio.
- **Performance Coach** (sidebar, under Analyze):
  - Add videos that perform well, plus your own, and type their views, likes and watch %.
  - Each one is measured automatically: when talking starts, the opening words, cuts/min, longest
    shot, pauses, speaking speed, transcript and call to action.
  - **Why does this work?**, **Coach this video** and **Build from my references** (your style profile)
    open the Coach.
  - Library videos can't be edited, so for those the Coach gives advice in its reply. For an open
    Clip Studio video it also makes Apply cards.
- **Engines:** each agent can use the local engine or **Claude** (your own API key, set in Settings →
  AI). In Settings → AI you can also pick another downloaded Ollama model, or have the model unload
  sooner to free up GPU memory.
- **Freeing memory when you close Creator Hub:** Settings → AI → **When I close Creator Hub**.
  **Quit Ollama** (the default) unloads the model and stops Ollama, so it uses no memory.
  **Unload the model** frees the ~6 GB but leaves the small server running. **Leave it running**
  changes nothing. Ollama starts again on its own the next time you use the Assistant.

**Teaching the agents your taste:**
- **About my channel** (Settings → AI assistant): your niche, games, audience, tone and things to
  avoid. Every agent and the Planner's AI buttons read it. Filling this in helps the most.
- **Ideas from videos you like:** on the Performance Coach page, give each reference video a title,
  what it's about, tags and **Why I like it**. Then:
  - **Ideas like these** (all references) or **Ideas like this** (one video) asks the Planner for
    ideas in that style, as Apply cards. Each says which video inspired it and what it borrowed.
  - In the Content Planner's Brainstorm box, turn on **Like my reference videos**. The topic is then
    optional.
  - The Planner borrows the format, hook style, pace and angle, never the video itself.
- **It learns from your choices:** applied cards and ideas you add count as *kept*; dismissed,
  undone and unadded ones count as *passed on*. The agents get the recent ones, suggest more like what
  you keep, and never re-suggest ideas you already have or decided on. Settings → AI → **Forget**
  clears this.

How it works:
- `src/main/ai/`: engines (`ollama.ts`, `claude.ts`), setup, and frame grabs for agents.
- `src/shared/ai/agents.ts`: the agents' personas and which tools each may use.
- `src/renderer/src/core/agentRunner.ts`: the agent loop. It runs tools, collects proposals, hands
  work between agents, and stops after 8 steps.
- `src/renderer/src/core/agentTools.ts`: `useAgentTool` / `useAgentContext`.
  - Each feature registers its own tools: `modules/planner/agentTools.ts`, `modules/studio/agentTools.ts`,
    `modules/clips/agentTools.ts`, `modules/coach/agentTools.ts`.
  - To give agents a new ability, add a tool there. Name it `propose_*` and return a Proposal to make
    it an Apply card.
- `src/main/coach.ts`: measures videos with FFmpeg + Whisper (cached).
- `src/shared/ai/memory.ts`: the "About my channel" note and your kept / passed-on choices
  (`ai.choices`), added to every agent's instructions.

## Stream Deck

The `streamdeck/` folder holds a Stream Deck plugin. Install or update it with:

```powershell
npm run streamdeck:install
```

(This closes the Stream Deck app for a few seconds, copies the plugin in and restarts it.)
Then drag actions from the **Creator Hub** category onto your keys:

| Button | What it does |
| --- | --- |
| Open Creator Hub | Brings the app forward, or starts it if it's closed |
| Open Stream Backdrop / Face FX / Clip Studio / Thumbnail Maker / Clip Generator | Opens the app on that page (starts it if needed) |
| Hood On/Off, Next Hood Style | Shows/hides the Face FX hood (the key shows on/off), cycles hood styles |
| Capture Screenshot | Saves a full-resolution screenshot to Thumbnail Maker → Captured moments |
| Backdrop Pulse / Flash | Plays the effect live in OBS |
| Next / Previous Backdrop Look | Cycles through presets |
| Match Screen On/Off, React to Audio On/Off | Toggles; the key shows on/off |
| Content Plans | Shows how many plans are due today or late (e.g. "2 due"); press to open the Content Planner |
| Ask Assistant | Opens the AI Assistant |
| Voice On/Off | Turns live mic clean-up on or off (the key shows on/off) |

A green check on the key means it worked; a warning triangle means Creator Hub isn't running or
something failed (details in the plugin's `logs/creatorhub.log`).

**How it connects:** Creator Hub serves a small API on `http://127.0.0.1:7420/api/` (your PC only).
Calls need the token from `%APPDATA%\Creator Hub\connection.json`. Any tool can use it:
`POST /api/action/<id>` with header `X-Creator-Hub-Token`. The list is at `GET /api/actions`.

**Adding a button:** add the action id to `src/shared/actions.ts`, handle it in a feature with
`useAction('<id>', () => …)`, add an entry to `streamdeck/actions.json` (any icon name from
lucide.dev that is imported in `streamdeck/icons.tsx`), then run `npm run streamdeck:install`.

## How the project is organised

```
src/
  main/             Electron "back end": window, data storage, OBS server
  preload/          The safe bridge between the UI and the back end (window.api)
  shared/           Code used by both sides (types, the backdrop engine + scenes)
  renderer/
    index.html      The app window
    overlay.html    The page OBS loads
    src/
      core/         App plumbing: module type, settings/theme, saved state
      components/   Shared UI kit (Panel, Field, Slider, Toggle, Button, ...)
      pages/        Home and Settings
      modules/      ⭐ One folder per feature
        registry.ts ⭐ The list of all features
        backdrop/   Stream Backdrop feature
      styles/       Theme colors and shared styles
```

## Adding a new feature (module)

1. **Create a folder** in `src/renderer/src/modules/`, e.g. `content-planner/`.
2. **Build the page** in it, using the shared UI kit so it matches the rest of the app:

   ```tsx
   // modules/content-planner/ContentPlanner.tsx
   import { CalendarDays } from 'lucide-react'
   import { PageHeader, Panel } from '../../components/ui'
   import { useStoredState } from '../../core/useStoredState'

   export default function ContentPlanner() {
     // Saved automatically and restored next launch. Prefix keys with your module id.
     const [ideas, setIdeas] = useStoredState<string[]>('content-planner.ideas', [])
     return (
       <div className="page">
         <PageHeader icon={CalendarDays} title="Content Planner" subtitle="Plan what you post." />
         <Panel title="Ideas">{/* ... */}</Panel>
       </div>
     )
   }
   ```

3. **Describe it** in an `index.ts`:

   ```ts
   import { CalendarDays } from 'lucide-react'
   import type { CreatorModule } from '../../core/types'
   import ContentPlanner from './ContentPlanner'

   export const contentPlannerModule: CreatorModule = {
     id: 'content-planner',          // never change this after release
     name: 'Content Planner',
     description: 'Plan, schedule and track every post.',
     icon: CalendarDays,             // any icon from https://lucide.dev/icons
     section: 'Plan',                // Stream | Create | Plan | Publish | Analyze
     component: ContentPlanner
   }
   ```

4. **Register it** in `modules/registry.ts`:

   ```ts
   export const MODULES: CreatorModule[] = [backdropModule, contentPlannerModule]
   ```

That's it. The sidebar groups it under its section, it appears on Home, and users can
reorder or hide it in Settings. If a module needs the back end (files, network, OS features),
add an `ipcMain.handle(...)` in `src/main/index.ts` and expose it in `src/preload/index.ts`.

## Adding a new backdrop scene

Create a file in `src/shared/backdrop/scenes/` (copy `waves.ts` as a starting point) and add it to
the `SCENES` list in `scenes/index.ts`, plus an icon in `SCENE_ICONS` in `BackdropStudio.tsx`.
Scenes get the palette, speed, density, glow and an `energy` value (audio + effects) to react to.

## Customizing the look

All colors live as variables at the top of `src/renderer/src/styles/app.css` (one block for dark,
one for light). Accent color, theme, text size and compact sidebar are user settings.
