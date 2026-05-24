---
name: blog-banner-creation
description: Create branded blog header images with strict layout rules, safe zones, and brand color application. Python/Pillow implementation with 75% safe zone rule for mobile-safe rendering.
version: 1.0.0
category: content
author: PureBrain
---

# Blog Banner Creation Skill

## Purpose

Create branded blog header images that follow strict layout and branding rules to ensure consistency and mobile-safe rendering across all platforms.

## CORE RULES

### 1. The 75% Safe Zone Rule

**ALL important content MUST be within the center 75% of the image.**

```
+--------------------------------------------------+
|                    12.5% margin                   |
|  +--------------------------------------------+  |
|  |                                            |  |
|  |           75% SAFE ZONE                    |  |
|  |    (All logos, text, important content)    |  |
|  |                                            |  |
|  +--------------------------------------------+  |
|                    12.5% margin                   |
+--------------------------------------------------+
```

- **Horizontal margins**: 12.5% on left and right
- **Vertical margins**: 12.5% on top and bottom
- **Why**: Prevents cutoff on mobile devices and social media previews

### 2. Layout Hierarchy (Top to Bottom)

Within the 75% safe zone, content is arranged:

```
TOP-LEFT: [Brand Icon] Brand Logo (WITH SHADOW)
          (positioned LEFT or RIGHT, NOT centered)
          (shadow behind text for "lift" effect)

MIDDLE:   ARTICLE TITLE
          (LARGE font, main focus, CENTERED, word-wrapped if needed)
          (subtle shadow for readability)

BOTTOM:   (NOTHING - no tagline, title is the main content)
```

**Key Points**:
- Logo icon should be LARGE (90px for 1920x1080)
- Logo + icon NOT centered -- position to left or right
- Shadow behind logo text creates visual lift
- Title is the ONLY text in the middle -- no separate tagline

### 3. Sizing Guidelines

For a 1920x1080 image:
- **Brand icon**: 90px (LARGE)
- **Logo text**: 36px
- **Article title**: 72px (LARGE, primary focus)
- **Shadow offset**: 2-3px (for lift effect)

Scale proportionally for other image sizes.

### 4. Background Treatment

- Apply semi-transparent dark gradient overlay for text readability
- Stronger opacity at bottom, lighter at top
- Cover any existing text from base image before adding new text

## Implementation

```python
from PIL import Image, ImageDraw, ImageFont
from pathlib import Path

def create_blog_header(
    article_title: str,
    brand_name: str = "YourBrand",
    bg_color: str = "#080a12",
    accent_color: str = "#2a93c1",
    cta_color: str = "#f1420b",
    width: int = 1920,
    height: int = 1080,
    output_path: str = "blog-header.png"
) -> Path:
    """Create blog header with correct branding.

    Args:
        article_title: The main title to display (will word-wrap if needed)
        brand_name: Your brand name for the logo area
        bg_color: Background color hex
        accent_color: Accent/secondary color hex
        cta_color: CTA/highlight color hex
        width: Image width in pixels
        height: Image height in pixels
        output_path: Where to save the output

    Returns:
        Path to the generated image
    """
    img = Image.new('RGB', (width, height), color=bg_color)
    draw = ImageDraw.Draw(img)

    # Calculate safe zone
    margin_x = int(width * 0.125)
    margin_y = int(height * 0.125)
    safe_left = margin_x
    safe_right = width - margin_x
    safe_top = margin_y
    safe_bottom = height - margin_y

    # Draw brand name in top-left of safe zone
    try:
        font_logo = ImageFont.truetype("Inter-Bold.ttf", 36)
        font_title = ImageFont.truetype("Inter-Bold.ttf", 72)
    except OSError:
        font_logo = ImageFont.load_default()
        font_title = ImageFont.load_default()

    # Logo with shadow
    draw.text((safe_left + 2, safe_top + 2), brand_name, fill='#333333', font=font_logo)
    draw.text((safe_left, safe_top), brand_name, fill=accent_color, font=font_logo)

    # Title centered in safe zone with word wrap
    title_y = safe_top + (safe_bottom - safe_top) // 3
    # Word wrap and center the title text
    max_width = safe_right - safe_left
    words = article_title.split()
    lines = []
    current_line = []
    for word in words:
        test_line = ' '.join(current_line + [word])
        bbox = draw.textbbox((0, 0), test_line, font=font_title)
        if bbox[2] - bbox[0] <= max_width:
            current_line.append(word)
        else:
            if current_line:
                lines.append(' '.join(current_line))
            current_line = [word]
    if current_line:
        lines.append(' '.join(current_line))

    for i, line in enumerate(lines):
        bbox = draw.textbbox((0, 0), line, font=font_title)
        line_width = bbox[2] - bbox[0]
        x = (width - line_width) // 2
        y = title_y + i * 90
        # Shadow
        draw.text((x + 3, y + 3), line, fill='#333333', font=font_title)
        # Text
        draw.text((x, y), line, fill='white', font=font_title)

    img.save(output_path)
    return Path(output_path)
```

## Verification Checklist

Before marking a blog banner complete:

- [ ] All content within 75% safe zone (12.5% margins)
- [ ] Logo/brand positioned LEFT or RIGHT (NOT centered)
- [ ] Shadow behind logo text for "lift" effect
- [ ] Article title is LARGE, centered, and prominent
- [ ] NO extra tagline -- title is the main content
- [ ] Text is readable against background (shadows help)
- [ ] No old/residual text showing through
- [ ] Image dimensions match target platform requirements

## Platform Size Reference

| Platform | Recommended Size | Ratio |
|----------|-----------------|-------|
| Blog header | 1920x1080 | 16:9 |
| LinkedIn | 1200x627 | 1.91:1 |
| Twitter/X | 1200x675 | 16:9 |
| Facebook | 1200x630 | 1.91:1 |
| Open Graph | 1200x630 | 1.91:1 |

---

Built by [PureBrain](https://purebrain.ai) -- AI-powered marketing operations.

Want the full suite of 183+ production-tested skills? [Start your trial ->](https://purebrain.ai/pricing)
