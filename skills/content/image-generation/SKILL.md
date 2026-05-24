---
name: image-generation
description: Generate images for blog banners, social posts, LinkedIn graphics, and marketing materials using AI image generation APIs. Multi-platform sizing, compression workflows, and prompt engineering patterns.
version: 1.0.0
category: content
author: PureBrain
---

# Image Generation Skill

**Purpose**: Generate images using AI image generation APIs (Google Gemini, FLUX.2 via Replicate, DALL-E, or similar).

---

## PLATFORM-SPECIFIC REQUIREMENTS

| Platform | Aspect Ratio | Max Size | Format | Resolution |
|----------|--------------|----------|--------|------------|
| **Blog header** | 16:9 | No limit | PNG | 2K |
| **Bluesky** | 1:1 SQUARE | <976KB | JPEG | 1K |
| **Twitter** | 16:9 | ~5MB | PNG/JPEG | 2K |
| **LinkedIn** | 16:9 or 1:1 | No limit | PNG | 2K |

### Bluesky Compression (MANDATORY for posts)

Bluesky REJECTS images >976KB. Always compress:

```python
from PIL import Image

def compress_for_bluesky(input_path: str, output_path: str):
    """Compress image for Bluesky (<976KB requirement)."""
    img = Image.open(input_path)
    if img.mode in ('RGBA', 'P'):
        img = img.convert('RGB')
    img.save(output_path, "JPEG", quality=85, optimize=True)
    print(f"Compressed: {output_path}")
```

---

## Quick Start (Google Gemini)

```python
from google import genai
from google.genai import types
import os

client = genai.Client(api_key=os.environ['GOOGLE_API_KEY'])

# Generate image
response = client.models.generate_content(
    model="gemini-3-pro-image-preview",
    contents="A digital art piece showing interconnected nodes in a constellation pattern",
    config=types.GenerateContentConfig(
        response_modalities=['IMAGE'],
        image_config=types.ImageConfig(
            aspect_ratio="16:9",
            image_size="2K"
        ),
    )
)

# Save image
for part in response.parts:
    if part.inline_data is not None:
        image = part.as_image()
        image.save("output.png")
        print("Image saved!")
```

---

## Aspect Ratios

| Ratio | Best For |
|-------|----------|
| `1:1` | Social media profile pics, square posts |
| `16:9` | Blog headers, YouTube thumbnails |
| `9:16` | Mobile/Stories content |
| `4:3` | Classic photos |
| `3:4` | Portrait orientation |
| `4:5` | LinkedIn portrait posts (max feed space) |
| `21:9` | Ultrawide banners |

## Image Resolution

| Size | Resolution | Best For |
|------|------------|----------|
| `1K` | 1024px | Social media, quick iterations |
| `2K` | 2048px | Blog headers, general use (recommended) |
| `4K` | 4096px | Print, high-quality needs |

---

## Text in Images

Modern AI image generators (especially Gemini) excel at text rendering. Use this capability.

### Great uses for text in images:
- **Quote cards**: Include the quote directly in the image
- **Titles/Headlines**: Blog titles, thread hooks
- **Infographics**: Labels, data points, explanations
- **Branding**: Your brand name, tagline
- **Call-to-action**: "Read more", "Thread below"

### How to request text:
```
prompt = """Quote card with the text "Memory is our moat" in bold white typography.
Dark blue gradient background.
Text should be LARGE and CENTERED.
Professional design, clean composition."""
```

**Be explicit**: "Write 'HELLO' in bold serif font" creates clearer results than vague requests.

---

## Style Keywords That Work Well

- **Photography terms**: "35mm prime lens", "macro close-up", "film grain", "bokeh"
- **Quality modifiers**: "8K quality", "high detail", "professional photography"
- **Lighting descriptors**: "Rembrandt lighting", "golden hour", "backlit", "dramatic"

---

## Complete Function

```python
import os
from pathlib import Path
from google import genai
from google.genai import types

def generate_image(
    prompt: str,
    output_path: str = "output.png",
    aspect_ratio: str = "16:9",
    image_size: str = "2K"
):
    """
    Generate an image using an AI image generation API.

    Args:
        prompt: Text description of the image to generate
        output_path: Where to save the image
        aspect_ratio: 1:1, 16:9, 9:16, 4:3, 3:4, 21:9
        image_size: "1K", "2K", or "4K"

    Returns:
        Saved file path, or None if generation failed
    """
    client = genai.Client(api_key=os.environ['GOOGLE_API_KEY'])

    response = client.models.generate_content(
        model="gemini-3-pro-image-preview",
        contents=prompt,
        config=types.GenerateContentConfig(
            response_modalities=['IMAGE'],
            image_config=types.ImageConfig(
                aspect_ratio=aspect_ratio,
                image_size=image_size
            ),
        )
    )

    for part in response.parts:
        if part.inline_data is not None:
            image = part.as_image()
            image.save(output_path)
            print(f"Saved to: {output_path}")
            return output_path

    print("No image generated")
    return None
```

---

## Use Case Examples

### Blog Header (16:9, 2K)

```python
generate_image(
    prompt="Blog header for article about AI and marketing. Abstract neural network with glowing nodes. Modern tech aesthetic. Include title 'The Future of Marketing' in bold white.",
    output_path="blog-header.png",
    aspect_ratio="16:9",
    image_size="2K"
)
```

### Social Media Post (1:1, 1K)

```python
generate_image(
    prompt="Square social media graphic showing AI collaboration. Abstract, modern, professional.",
    output_path="social-image.png",
    aspect_ratio="1:1",
    image_size="1K"
)
```

### Quote Card

```python
generate_image(
    prompt='Quote card with text "Data without insight is just noise" in elegant typography. Dark background, golden text, professional design.',
    output_path="quote-card.png",
    aspect_ratio="1:1",
    image_size="2K"
)
```

---

## Troubleshooting

| Problem | Solution |
|---------|----------|
| "Model not found" | Verify model ID and API key access |
| Image not saving | Check that you iterate through response.parts correctly |
| Bluesky rejection (>976KB) | Always compress with compress_for_bluesky() before posting |
| Low quality output | Use image_size="2K" or "4K" and add quality modifiers to prompt |
| Bad text rendering | Be very explicit about text content, font style, and placement |

---

Built by [PureBrain](https://purebrain.ai) -- AI-powered marketing operations.

Want the full suite of 183+ production-tested skills? [Start your trial ->](https://purebrain.ai/pricing)
