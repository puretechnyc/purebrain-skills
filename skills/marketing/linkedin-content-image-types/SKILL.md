---
name: linkedin-content-image-types
description: LinkedIn image types and generation methods for each post format. Quote cards, article screenshots, brand photos, and selfie content with exact dimensions and tools.
version: 1.0.0
category: marketing
author: PureBrain
---

# LinkedIn Content Image Types

## When to Use

When generating images for LinkedIn posts. Each post type has a specific image format.

## Post Type 1: Quote Card (Twitter Repost Style)

- **Size**: 1080x1350 (portrait, 4:5)
- **Design**: Black background, headshot in accent-bordered circle, social icon centered at bottom of circle, name + handle, quote text in bold sans-serif, "FOLLOW FOR MORE" at bottom
- **Use for**: Hot takes, opinions, one-liners, motivational insights
- **Font**: Bold sans-serif for quote text, extra-bold for name

### How to Generate

Use Python with Pillow (PIL):

```python
from PIL import Image, ImageDraw, ImageFont

def create_quote_card(quote: str, name: str, handle: str, headshot_path: str, output_path: str):
    """Create a LinkedIn quote card image."""
    img = Image.new('RGB', (1080, 1350), color='#000000')
    draw = ImageDraw.Draw(img)

    # Load headshot and create circular mask
    headshot = Image.open(headshot_path).resize((200, 200))
    mask = Image.new('L', (200, 200), 0)
    mask_draw = ImageDraw.Draw(mask)
    mask_draw.ellipse((0, 0, 200, 200), fill=255)

    # Paste headshot with circular mask
    img.paste(headshot, (440, 100), mask)

    # Draw accent border circle
    draw.ellipse((435, 95, 645, 305), outline='#2a93c1', width=3)

    # Add name and handle
    draw.text((540, 330), name, fill='white', anchor='mt')
    draw.text((540, 360), handle, fill='#888888', anchor='mt')

    # Add quote text (centered, word-wrapped)
    # ... (implement word wrapping for quote)

    # Add "FOLLOW FOR MORE"
    draw.text((540, 1280), "FOLLOW FOR MORE", fill='#888888', anchor='mt')

    img.save(output_path)
```

## Post Type 2: Article Screenshot (News Reaction)

- **Size**: 1200x628 (landscape, 1.91:1)
- **Method**: Playwright with mobile viewport (480px wide, 2x device scale)
- **Process**:
  1. Navigate to article URL with Playwright
  2. Viewport: width=600, height=314, device_scale_factor=2
  3. Dismiss cookie popups
  4. Find h1 headline element, get bounding box
  5. Screenshot from y = headline_y - 10, width=600, height=314
  6. Output is 1200x628 at 2x scale (crisp text)
- **Result**: Clean headline + subtitle + date, no nav/sidebar/ads
- **Use for**: Reacting to industry news, sharing articles with your take

## Post Type 3: Brand/Topic Photo

- **Size**: 1200x627 or 1200x1200
- **Method**: Source a relevant real-world brand photo (store front, product shot, event photo)
- **Use for**: When the article screenshot isn't visually interesting

## Post Type 4: Selfie + Caption

- No image generation needed
- Use a real selfie
- **Use for**: Personal stories, behind-the-scenes, daily life content
- This is typically the highest-performing content type

## LinkedIn Image Size Reference

| Dimensions | Ratio | Notes |
|-----------|-------|-------|
| 1200 x 627 | 1.91:1 (landscape) | Most common, fills feed width |
| 1200 x 1200 | 1:1 (square) | Stands out, takes more feed space |
| 1080 x 1350 | 4:5 (portrait) | Tallest allowed, maximum feed real estate |

## Integration with Content Pipeline

When picking a story for a post:
1. Generate quote card with your hot take (Post Type 1)
2. Screenshot the article in landscape (Post Type 2)
3. Optionally grab a brand photo (Post Type 3)
4. Push all options + caption draft to your task management tool
5. Pick the best image, approve, and post

---

Built by [PureBrain](https://purebrain.ai) -- AI-powered marketing operations.

Want the full suite of 183+ production-tested skills? [Start your trial ->](https://purebrain.ai/pricing)
