# Nanobanana

CLI tool for AI image generation powered by **Google Gemini 3 Pro Image** model. Designed to be orchestrated by AI coding agents like [Claude Code](https://docs.anthropic.com/en/docs/claude-code), but works great standalone too.

Supports single image generation, batch processing from JSON job files, visual references, and image-to-prompt analysis.

> **Course material** for [Vibe Coding for Marketers](https://vibecodingformarketers.com) — a hands-on course teaching marketers to build real tools with AI.

## Features

- **Single generation** — generate one image from a text prompt
- **Batch processing** — generate multiple images from a JSON job file with progress tracking
- **Visual references** — use a reference image to guide style and content
- **Image analysis** — describe an existing image to create reusable prompts
- **Flexible output** — choose aspect ratio (10 options), resolution (1K/2K/4K), and format (WebP/PNG/JPEG)
- **Retry logic** — automatic exponential backoff on API failures
- **Graceful interrupts** — Ctrl+C stops batch processing cleanly after the current job

## Requirements

- Python 3.11+
- Google Gemini API key (free to get — see below)

## Getting your API key

1. Go to [Google AI Studio](https://aistudio.google.com/app/apikey)
2. Sign in with your Google account
3. Click **"Create API key"**
4. Select or create a Google Cloud project (the free tier is sufficient)
5. Copy the generated key — it starts with `AIzaSy...`

The API key gives you access to all Gemini models including image generation. Google offers a generous free tier (~500 images/day). See [Pricing](#pricing) for details.

## Installation

```bash
git clone https://github.com/faborsky/nanabonana-agent-public.git
cd nanabonana-agent-public

# Run setup (creates venv, installs dependencies)
./setup.sh
```

The setup script creates a `.env` file from the template. Open it and paste your API key:

```bash
nano .env
```

The file should look like this:

```env
# Gemini API Key
# Get your key from: https://aistudio.google.com/app/apikey
GEMINI_API_KEY=AIzaSy...your-key-here
```

That's it. The app reads the key from `.env` automatically.

## Usage

### Generate a single image

```bash
./run.sh generate "sunset over ocean, cinematic lighting, high detail" --aspect 16:9 --resolution 2K
```

| Option | Short | Default | Description |
|--------|-------|---------|-------------|
| `--aspect` | `-a` | `1:1` | Aspect ratio |
| `--resolution` | `-r` | `1K` | Resolution (1K / 2K / 4K) |
| `--output` | `-o` | `./output` | Output directory |
| `--name` | `-n` | auto | Custom filename |
| `--format` | `-f` | `webp` | Output format (webp / png / jpeg) |
| `--reference` | `-ref` | — | Reference image for style guidance |

**Supported aspect ratios:** `1:1`, `2:3`, `3:2`, `3:4`, `4:3`, `4:5`, `5:4`, `9:16`, `16:9`, `21:9`

### Batch processing

```bash
# Run a batch job
./run.sh batch jobs/example.json

# Custom delay between API calls
./run.sh batch jobs/example.json --delay 2.0

# Validate a job file without running it
./run.sh validate jobs/example.json
```

### Batch job file format

```json
{
  "defaults": {
    "aspect_ratio": "16:9",
    "resolution": "2K",
    "format": "webp"
  },
  "jobs": [
    {
      "prompt": "Majestic mountain range at golden hour, snow-capped peaks, cinematic composition",
      "output_name": "mountains"
    },
    {
      "prompt": "Tropical beach with crystal clear water, aerial view, vibrant colors",
      "aspect_ratio": "21:9",
      "resolution": "4K"
    },
    {
      "prompt": "Transform this into watercolor style",
      "reference_path": "references/photo.jpg"
    }
  ]
}
```

- `defaults` is optional — sets fallback values for all jobs
- Each job must have a `prompt`
- All other fields are optional and inherit from defaults

### Image analysis

Analyze an existing image and get a text description you can use as a prompt:

```bash
# Brief description
./run.sh describe image.jpg

# Detailed prompt-ready description
./run.sh describe image.jpg --detailed
```

## Pricing

This tool uses the Google Gemini API. Approximate cost per image (as of early 2026):

| Resolution | Cost per image |
|------------|---------------|
| 1K (1024px) | ~$0.04 |
| 2K (2048px) | ~$0.13 |
| 4K (4096px) | ~$0.24 |

There is also a **free tier** with up to ~500 images/day through Google AI Studio.

The Gemini API [Batch API](https://ai.google.dev/gemini-api/docs/batch) offers a **50% discount** for non-real-time requests (24h processing window). This tool does not implement batch API yet, but it could be added.

For current pricing, always check the [official Gemini API pricing page](https://ai.google.dev/gemini-api/docs/pricing).

## How it works with AI agents

Nanobanana is designed as a CLI tool that AI coding agents can orchestrate. When used with Claude Code, the agent can:

1. Read your requirements and craft optimized prompts
2. Create batch job JSON files
3. Run generation commands
4. Analyze results and iterate

Example workflow in Claude Code:
```
You: "Generate 5 hero images for a restaurant website, 16:9, 2K"
Claude Code: Creates a batch JSON → runs ./run.sh batch → delivers results
```

## Project structure

```
nanabonana-agent-public/
├── nanobanana/           # Python package
│   ├── cli.py            # Typer CLI commands (generate, batch, validate, describe)
│   ├── generator.py      # Gemini API wrapper with retry logic
│   ├── batch.py          # Batch job parsing and processing
│   ├── config.py         # Configuration, enums, validation
│   └── utils.py          # Filename generation utilities
├── jobs/                 # Batch job JSON files
│   └── example.json      # Example batch job
├── output/               # Generated images (gitignored)
├── CLAUDE.md             # Instructions for AI coding agents
├── setup.sh              # Installation script
├── run.sh                # Entry point wrapper
└── .env                  # API key (never committed!)
```

## License

MIT

## Author

Built by [Jindrich Faborsky](https://www.faborsky.com) with Claude Code.

---

## Nanobanana — Informace v cestine

Tento nastroj je soucasti vyukoveho materialu dvou kurzu:

### Vibe Coding for Marketers

[Vibe Coding for Marketers](https://vibecodingformarketers.com) je prakticky kurz pro marketery, kteri chteji stavet vlastni nastroje, weby a automatizace pomoci AI — bez potreby programatorskych znalosti. Nanobanana je priklad realneho nastroje postaveného behem kurzu.

### AI First

Nanobanana se take objevuje v kurzu [AI First](https://aifirst.cz) v **lekci c. 7: Vibe codujeme marketing — veskerá grafika z Claude Code**. V teto lekci se studenti uci, jak pomoci Claude Code a vlastnich CLI nastroju automatizovat tvorbu marketingove grafiky.

Kurz AI First je prakticky online kurz pro ceske podnikatele, marketery a OSVC o vyuziti AI v podnikani. Vice informaci na [aifirst.cz](https://aifirst.cz).
