# Loop — agent skill

Loop is a canvas of live screens for your coding agent. It opens your project on a canvas as a live, clickable frame: click around inside it, and duplicate it to keep any screen side by side. Point at anything, draw on it or leave a comment, and the agent that launched Loop (Claude Code or Codex) makes the change while the frame shows it working.

## Use it

Paste this into your coding agent, inside a project:

> Run `npx skills add amalenezi/loop-skill -g -y`, then use the Loop canvas skill to put this project on a canvas.

Or install the CLI yourself:

```bash
npm install -g loop-canvas
loop scan        # open this project as a live frame on a canvas
```

## On the canvas

- **C** — comment: click to pin, drag to highlight an area
- **D** — deep select exact elements, then comment
- **P** — draw, then type what should change
- **F** — new frame (Mobile, Tablet, Desktop or any size)
- **?** — all shortcuts

Requires Node 18+ and Claude Code or Codex.
