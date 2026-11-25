# Zernio Library Skills

A library of Claude Code skills for working with [Zernio](https://zernio.com). It currently contains five skills — `zernio-publish` (end-to-end social publishing), `zernio-comment-to-dm` (comment-to-DM automations + DM sequences), `meta-ads-launch` (Meta ad batches through Zernio's ads API, launch-then-pause), `zernio-workflow-creator` (Zernio conversation Workflows / WhatsApp text AI agents), and `zernio-voice-agent` (WhatsApp AI front desk — text agent + inbound calls routed via `forwardTo` to a Retell AI voice agent on one number). More Zernio-related skills (analytics, account management, content calendaring) get added to `.claude/skills/` as they're built.

This is a **Claude Code skill repo**, not a runtime or SDK project. The agent (Claude) has its native toolkit available — bash, curl, ffmpeg, vision, transcript extraction, web fetching. The skill is *instructions* for using those tools well; it isn't a framework around them.

## Clip Machine project (how we run it here)

This folder is also the working home of **the Clip Machine**: one long video in, native short clips out for TikTok, Instagram Reels and YouTube Shorts. The framework is `.claude/skills/clip-machine/` (SKILL.md + `reference/best-moments-prompt.md` + `reference/platform-cheat-sheet.md`); posting rides `.claude/skills/zernio-publish/`.

**The four stations:** CUT (WhisperX word-level transcript, best moments scored with the framework, cut on word boundaries, reframe to 1080x1920) → CAPTION (burned word captions inside the safe band + different caption text per platform) → COVER (cover is frame 0; Reels title inside the center square) → POST (one Zernio request per clip, scheduled 2-3 min ahead, posted serially, verified after).

**How we work together**
- User hands over the long video (upload or Drive link). Claude transcribes, proposes the clip list with scores and anchors, and waits for the pick.
- Claude produces the clips, then shows the full package per clip (file, captions, title, cover, schedule). **Nothing publishes without an explicit OK.**
- Space clips 1-2 per day. Every clip points back to the long video.

**Where things live** (all gitignored)
- `media/source/` long videos · `media/transcripts/` WhisperX JSON · `media/clips/` finished 9:16 clips + covers · `drafts/` per-clip post bodies awaiting approval · `posts/` publish logs
- `.venv/` Python env with WhisperX + yt-dlp: `. .venv/bin/activate` then `HF_HUB_DISABLE_XET=1 whisperx file.mp4 --model large-v3 --device cpu --compute_type int8 --output_format json --output_dir media/transcripts`
- `.env` holds `ZERNIO_API_KEY` (skip-worktree is set, so the real key never gets staged). Load with `set -a; . ./.env; set +a`. API base: `https://zernio.com/api/v1`.

## Project layout

```
zernio-library-skills/
├── .claude/
│   └── skills/
│       ├── clip-machine/                      ← long video → native TikTok/Reels/Shorts (CUT→CAPTION→COVER→POST)
│       │   ├── SKILL.md
│       │   └── reference/{best-moments-prompt,platform-cheat-sheet,initiation-prompt}.md
│       │
│       ├── zernio-publish/                    ← end-to-end social publishing
│       │   ├── SKILL.md                          the workflow + Zernio API knowledge
│       │   ├── reference/                        deep docs (loaded on demand)
│       │   │   ├── zernio-api.md
│       │   │   ├── zernio-upload.md
│       │   │   ├── zernio-post.md
│       │   │   ├── principles.md
│       │   │   ├── platforms.md
│       │   │   ├── platforms/{platform}.md        per-platform deep dives (13 of them)
│       │   │   └── zernio-openapi.yaml            canonical 17K-line OpenAPI spec
│       │   └── templates/manifest.json           reference shape of a Zernio POST body
│       │
│       ├── zernio-comment-to-dm/              ← comment-to-DM automations + sequences
│       │   ├── SKILL.md
│       │   ├── reference/zernio-automations-api.md
│       │   └── templates/automation.json
│       │
│       ├── meta-ads-launch/                   ← Meta ad batches via Zernio's ads API
│       │   ├── SKILL.md                          launch-then-pause co-pilot (operator decides)
│       │   ├── reference/zernio-ads-api.md       complete verified ads API reference
│       │   └── templates/campaign-plan.json      multi-creative request template
│       │
│       ├── zernio-workflow-creator/           ← Zernio Workflows / WhatsApp text AI agents
│       │   ├── SKILL.md
│       │   ├── reference/zernio-workflows-api.md  full node/edge contract (16 node types)
│       │   └── templates/whatsapp-customer-service-agent.json
│       │
│       ├── zernio-voice-agent/                ← WhatsApp AI FRONT DESK (Zernio + Retell AI)
│       │   ├── SKILL.md                          text + voice on one number, inbound-first
│       │   ├── reference/whatsapp-calling-api.md  verified calling endpoints + gates + start_call
│       │   ├── reference/retell-voice-api.md      Retell wiring (inbound_agents / SIP / wss)
│       │   └── templates/
│       │       ├── whatsapp-ai-front-desk.workflow.json  THE proven graph (text + tap-to-call)
│       │       ├── whatsapp-voice-agent.workflow.json    outbound start_call variant
│       │       ├── retell-voice-agent.base.md            fill-in-the-blank voice prompt
│       │       └── retell-voice-agent.example.md         a filled voice prompt (booking)
│       │
│       └── skill-creator/                     ← Anthropic's official skill-creator (bundled)
│                                                  use this when adding more skills to the library
│
├── apps/white-label-app/                     ← the finished white-label app from the videos (Node 22.13+: npm ci, npm test, npm run demo)
├── .env                                       ← ZERNIO_API_KEY (placeholder by default; user fills in)
├── CLAUDE.md                                  ← this file
├── README.md
├── LICENSE
└── .gitignore
```

## How to use this library

1. Drop the `.claude/skills/zernio-publish/` folder into any Claude Code project, OR open this repo directly as a Claude Code project.
2. Edit `.env` to put your real `ZERNIO_API_KEY` in place of the placeholder. Then `git update-index --skip-worktree .env` to keep your key out of commits.
3. Talk to Claude. Drop a video file, paste a Drive link, describe what you want to publish — Claude reads `SKILL.md`, follows the workflow, and ships.

The agent uses its own tools — `source .env` to load the key, `curl -L` for URLs and Drive single-file shares, `ffmpeg` for video format conversion, vision to read images, transcript tools for video/audio. There are no helper scripts in this repo; the agent doesn't need them.

## Adding more skills to the library

The point of the "library" name: more Zernio-related skills land here as they're built. To add one:

1. Create `.claude/skills/{skill-name}/SKILL.md` with YAML frontmatter and the skill's instructions.
2. Optional: add `reference/` for deep docs the skill pulls on demand, `templates/` for reusable shapes.
3. The bundled `skill-creator` at `.claude/skills/skill-creator/` is Anthropic's official tool for designing and improving skills — invoke it when building a new one.

## Hard rules embedded in `zernio-publish`

These four constraints live inside `SKILL.md`. They prevent real damage, not autonomy:

- **Approval before publish.** Show the full package, wait for an explicit OK.
- **Verify after publish.** Zernio's 200 OK ≠ landed on the platform.
- **`scheduledFor` 2-3 min ahead** for multi-platform. Never `publishNow: true`.
- **Zernio only.** No Late, Buffer, Hootsuite, or any alternative posting tool.

Everything else is judgment.
