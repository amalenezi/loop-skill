---
name: loop-canvas
description: Put design work on a Loop canvas, a local, shareable canvas of live frames (real HTML/CSS/JS or running pages) that people can play with, draw on, point at and comment on. Comments flow back to the agent and the frame shows it working. Use when the user mentions a "Loop canvas" or the loop CLI, says "put this on a canvas", "show me the screens side by side", "explore a few directions/variations", "make a canvas of …", or wants feedback on UI they can click through and share.
---

# Loop

Loop is a local canvas of **live frames**. Each frame is either a folder of static files (`index.html` + assets) or a live URL (for example a route on the project's running dev server). The user can interact with frames, draw on them, and point at an element and comment. Each comment is delivered to an agent together with the exact element (selector, text, HTML) and any marks drawn on it. While the agent works, the frame's title shows a loading state. When the work is done, the frame reloads in place.

## Rules (always)

- **Loop is the `loop` CLI and the canvas it serves at `http://localhost:<port>/__loop/`.** Only use it through `loop` commands. Never build your own canvas, viewer or gallery page, and never publish one as a hosted site. If `loop` can't be installed or won't run, stop and tell the user what failed.
- **Never change Loop itself.** Its toolbar, menus, tools, theme and `.loop/` files are not yours to edit or extend. Your output is the content of frames: the project's code, or a frame's own folder.
- **Never copy the app's code into frame folders.** For an existing app, frames are live routes of the running app (`loop scan`, `loop add-route`). Static frames (`loop new`) are for new designs, each one small and self-contained.

## Setup (first use only)

The CLI is `loop` (Node 18+). Check with `command -v loop`. If it's missing, install it:

```bash
npm install -g loop-canvas
```

If a global install isn't allowed, run every command below as `npx -y loop-canvas <command>` instead. Comments are handled by the agent that launched Loop: Claude Code (`claude`) or Codex (`codex`), installed and signed in.

## Make a canvas from a prompt

1. **Decide what the frames are.**
   - "Put this project (or its main screens) on a canvas": run `loop scan` from the project root. It detects the framework (Next.js, SvelteKit, Nuxt, Astro, Remix, React Router/Vite SPAs, Angular, or plain HTML), starts the project's dev server if it isn't running, and places **one live frame of the app** (its home screen, served through Loop so the user can point at exact elements). The user clicks around inside it, and duplicates it (⌘D or ⌥-drag the title) to keep another screen side by side; the copy opens on the screen they navigated to. Options: `--mobile` (phone, 393×852), `--landscape` (phone landscape, 852×393, good for games), `--w W --h H` (any size), `--both` (desktop + phone), `--url http://localhost:PORT` (use a specific running server).
     Add a specific screen only when the user asks for it: `loop add-route /settings --name "Settings"`. Don't place every screen: each live frame runs the whole app. `loop scan --all` exists, but only use it when the user explicitly asks for every screen at once.
     If the app can't run (missing env vars, backend down), say so and ask which few screens matter; then build those as small static frames (step 2), never a copy of the whole app per frame.
   - "Explore / give me N directions for X": make N frames, one per distinct direction. Make them genuinely different (layout, hierarchy, tone), not one design with small tweaks.
2. **Create a static frame:** `loop new "Pricing — editorial"` prints a folder path. Write a complete, self-contained `index.html` there: inline CSS/JS, CDN imports are fine, no build step, realistic content, working interactions. The canvas reloads the frame live as you write.
   - Sizes: `--w 1280 --h 800` desktop (default) · `--w 390 --h 844` mobile · `--w 834 --h 1112` tablet.
   - An existing folder or HTML file can be placed directly: `loop add ./prototypes/onboarding --name "Onboarding"`.
3. **Open it:** `loop open` (`loop scan` already does this). It starts Loop's server for this project in the background (state lives in `./.loop/`), prints a URL like `http://localhost:4848/__loop/`, and returns. Open **that exact URL**:
   - In Codex, open it in Codex's built-in browser view. If you can't open a browser, give the user the URL to open.
   - In Claude Code, it opens in the user's browser automatically.
   The canvas is only ever that localhost page. If you find yourself writing HTML for a canvas, toolbar or comment UI, or publishing a site, stop: that's not Loop, and comments left there reach no one.
4. **Share it, if asked:** `loop share` prints a link anyone on the same Wi-Fi can open (also in the canvas under **Share**). `loop share --public` makes a public link through a tunnel (needs `brew install cloudflared`). Guests can click through the frames and comment; their comments wait for the host's approval before reaching the agent.

## Comments come to this chat (Claude Code)

Loop is a visual canvas for you and the user to work on together. Their canvas comments are them talking to **you**, in this chat, with everything you already know about them and the project. So in Claude Code, always do this right after opening the canvas:

1. Start `loop watch` with the Monitor tool (set the longest timeout, and start it again whenever it expires) and keep it running for the rest of the session. Each canvas comment arrives as a line: `COMMENT <id> on "<frame>": "<text>" | details: <file> | look: loop shot <frameId> | when done: loop reply …`.
2. Treat each COMMENT exactly like a message from the user here. Read the details file (where they pointed, marks, attachments), look at the frame with `loop shot`, do the work the way you would for any request in this chat, check it with another screenshot, then answer with `loop reply <id> "<what you changed>" --model <your model id>`. The frame reloads and your reply shows in the canvas thread.
3. If the user talks about a frame here in the chat instead, make the change and post a short update to its canvas thread with `loop note <id> "<what you changed>"`. `loop thread <id>` shows what was said on the canvas.

Tell the user: "Comments you leave on the canvas come to me here, and I'll reply in both places." If this chat stops listening, comments wait on the canvas until it's back (the user can send one to a background agent from its thread).

## How comments reach an agent

Check the mode with `loop list` or the pill in the top right; switch it with `loop agent headless|inbox`.

Loop detects which agent launched it (Claude Code or Codex) and uses that same agent for the canvas's comments. If detection ever picks the wrong one, pass `--agent codex` or `--agent claude` to `loop open` / `loop scan`.

- **headless** (default): Loop runs `claude -p` itself in the frame's folder (or in the project root for URL frames) for every comment, streams progress to the frame title, and keeps a snapshot so the user can undo. Nothing else is needed.
- **inbox**: use this when the user wants *this* session to handle feedback. Steps:
  1. `loop inbox --wait` blocks up to 25 seconds and returns JSON `{comment, frame, prompt}`, or `{"comment": null}`. Repeat until you get a comment.
  2. Read `prompt`. It describes the request, the element the user pointed at, and the path to a PNG of any marks they drew. Make the change in `frame.src` (static frames), or find the source for that route (URL frames).
  3. While working, optionally show progress on the frame title: `loop step <commentId> "Editing the header"`.
  4. Finish with `loop reply <commentId> "One sentence on what changed" --model <your model id, e.g. claude-opus-5-5>` (add `--error` if you couldn't do it). The frame reloads, and the model shows under your reply.

## Seeing your work and offering options
- `loop shot <frameId>` saves a screenshot of a frame exactly as it renders and prints the PNG path. Look at it before and after a visual change, and fix what looks off before you reply.
- When the user asks for options or alternatives on a comment, don't change the real files. For each option run `loop option <commentId> "<short name>"`, which prints an empty folder, and write the complete new version of each changed file there at the same relative path. The user previews the options live in the frame and picks one. `loop shot <frameId> --option <commentId>:A` shows an option on a static frame before anyone picks.

## Other commands
`loop list` (frames + status) · `loop create "<brief>"` (Loop's own agent builds a new frame) · `loop rm <id>` · `loop stop`

## Notes
- Keep static frames self-contained in their folder. The comment agent is only allowed to edit inside that folder.
- Frames with a snowflake are **frozen**: read-only snapshots the user took to keep a moment (⇧F). Never edit a snapshot; a comment on one is a change to the app's source for that screen.
- Name frames by what they are ("Checkout — one page", "Checkout — stepped"), not "Frame 1".
- `.loop/canvas.json` holds the layout and comments; its folder includes a `.gitignore` for runtime files.
