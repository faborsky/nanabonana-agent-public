# Nanobanana — Claude Code Instructions

## Project Overview

CLI application for AI image generation using Google Gemini 3 Pro Image model. Supports single and batch generation, visual references, and image analysis.

## Structure

```
nanobanana/
├── nanobanana/        # Python package
│   ├── cli.py         # Typer CLI commands
│   ├── generator.py   # Gemini API wrapper
│   ├── batch.py       # Batch processing logic
│   ├── config.py      # Config, validation, constants
│   └── utils.py       # Filename generation helpers
├── jobs/              # Batch job JSON files
└── output/            # Generated images
```

## Running

```bash
./run.sh generate "prompt" -a 16:9 -r 2K
./run.sh generate "prompt" -f png -r 4K
./run.sh generate "prompt" -ref reference.jpg
./run.sh batch jobs/myjob.json
./run.sh validate jobs/myjob.json
./run.sh describe image.jpg
./run.sh describe image.jpg --detailed
```

## Creating Batch Job Files

### JSON Format

```json
{
  "defaults": {
    "aspect_ratio": "16:9",
    "resolution": "2K",
    "format": "webp"
  },
  "jobs": [
    {
      "prompt": "Detailed description of the image",
      "output_name": "optional_filename",
      "aspect_ratio": "1:1",
      "resolution": "4K",
      "format": "png"
    }
  ]
}
```

### Rules

- `defaults` is optional — sets default values for all jobs
- Each job MUST have a `prompt`
- `output_name`, `aspect_ratio`, `resolution`, `format`, `reference_path` are optional
- Valid `aspect_ratio`: `1:1`, `2:3`, `3:2`, `3:4`, `4:3`, `4:5`, `5:4`, `9:16`, `16:9`, `21:9`
- Valid `resolution`: `1K`, `2K`, `4K`
- Valid `format`: `webp` (default), `png`, `jpeg`
- Valid reference image formats: `.jpg`, `.jpeg`, `.png`, `.webp`, `.gif`

## Prompting Gemini 3 Pro Image

### Effective Prompt Structure

```
[Main subject] + [Environment/Background] + [Lighting] + [Style/Mood] + [Technical details]
```

### Examples

**Landscape:**
```
Majestic mountain range at golden hour, snow-capped peaks reflecting warm sunlight,
alpine meadow with wildflowers in foreground, dramatic clouds,
photorealistic, high detail, cinematic composition
```

**Portrait/Character:**
```
Portrait of an elderly fisherman, weathered face with deep wrinkles,
kind eyes, wearing a worn cable-knit sweater,
soft natural window light from the side, shallow depth of field,
documentary photography style
```

**Product:**
```
Minimalist product shot of a ceramic coffee mug,
matte white finish, soft shadows on light gray background,
studio lighting, clean aesthetic, commercial photography
```

### Quality Keywords

| Category | Keywords |
|----------|----------|
| Quality | `high detail`, `ultra detailed`, `sharp focus`, `8K`, `photorealistic` |
| Lighting | `golden hour`, `soft light`, `dramatic lighting`, `backlit`, `rim light`, `studio lighting` |
| Style | `cinematic`, `editorial`, `documentary`, `fine art`, `commercial` |
| Composition | `rule of thirds`, `symmetrical`, `wide angle`, `close-up`, `aerial view` |
| Mood | `moody`, `ethereal`, `vibrant`, `serene`, `dramatic` |

### Tips

- Be specific — the more detail, the better the result
- Avoid conflicting instructions ("realistic cartoon style")
- Avoid negative phrasing ("no people") — the model may not respect it
- For consistent series, use a shared style suffix across prompts

## Visual References

### Direct reference (multimodal input)

```bash
./run.sh generate "create similar image in watercolor style" -ref photo.jpg
```

### Batch job with references

```json
{
  "jobs": [
    {
      "prompt": "Transform this into anime style, vibrant colors",
      "reference_path": "references/photo1.jpg",
      "output_name": "anime_version"
    }
  ]
}
```

### The `describe` command

Automatically analyzes an image and generates a prompt-ready description:

```bash
# Brief description (under 100 words)
./run.sh describe image.jpg

# Detailed description optimized for generation
./run.sh describe image.jpg --detailed
```

## Development

### Adding a New CLI Command

1. Add a function with `@app.command()` decorator in `cli.py`
2. Use Typer annotations for parameters
3. Validate inputs using callbacks or functions from `config.py`

### Modifying Valid Values

Aspect ratios, resolutions, and output formats are defined in `config.py`:
- `AspectRatio` enum
- `Resolution` enum
- `OutputFormat` enum

### Important: Gemini SDK Image Object

`part.as_image()` from the Gemini SDK does **NOT** return a standard `PIL.Image.Image`. It returns an SDK wrapper with its own `.save(path)` method that only accepts a file path (format is derived from extension). To use PIL features (quality params, format conversion, mode conversion), first save to a temp file via the wrapper's `.save()`, then re-open with `PIL.Image.open()`.

### Testing

```bash
source .venv/bin/activate
./run.sh validate jobs/example.json  # Validation without API calls
./run.sh generate "test" -r 1K       # Quick test with small resolution
```
