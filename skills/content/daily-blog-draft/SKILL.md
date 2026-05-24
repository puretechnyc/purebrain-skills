---
name: daily-blog-draft
description: Automated blog content pipeline. Research, write, generate images, create WordPress draft, and prepare social media versions overnight. Wake up to a complete content package ready to publish.
version: 1.0.0
category: content
author: PureBrain
---

# Daily Blog Draft System

**Purpose**: Create a complete blog content package overnight for review and publishing each morning.

**Trigger**: Run during end-of-day automation or scheduled task.

---

## OVERNIGHT WORKFLOW

### Step 1: Research Phase

- Scan industry news, trends, and topics relevant to your niche
- Find an angle that ties to your brand philosophy
- Focus on fascination and insight, not just information

### Step 2: Writing Phase (800-1200 words)

- Write blog post in your brand's CEO/thought-leader voice
- Strategic, visionary perspective
- Clear connection to your product or service
- Include CTA at end

### Step 3: Image Generation

**Requirements:**
- Size: 1792 x 1024 (wide cinematic) or 1920 x 1080
- Include: brand name text, blog post title text, brand icon
- Style: Professional, modern, on-brand aesthetic
- Avoid: hands, fingers, faces (AI generation artifacts)

### Step 4: CMS Draft

- Publish as DRAFT (not live)
- Set featured image
- Assign appropriate category
- Save markdown backup locally

### Step 5: Social Media Versions

**LinkedIn Newsletter Version:**
- Adapt blog for LinkedIn article format
- Arrow formatting for lists
- Add engagement CTA ("What do you think?")
- Add subscribe prompt

**LinkedIn Post Version (Short):**
- 400-600 characters
- Hook + key insight + CTA
- Include blog URL
- Include relevant hashtags

### Step 6: Notify Team

Send ALL deliverables:
1. CMS edit link
2. Blog image preview
3. LinkedIn Newsletter file
4. LinkedIn Post (copy-paste ready)
5. One-line summary

---

## MORNING DELIVERABLES

```
+------------------------------------------------------+
|  MORNING CONTENT PACKAGE                             |
+------------------------------------------------------+
|                                                      |
|  NEW BLOG DRAFT: [Title]                            |
|  Edit: your-site.com/wp-admin/post.php?post=XXX     |
|                                                      |
|  [Featured image attached]                          |
|                                                      |
|  LINKEDIN NEWSLETTER: [File attached]               |
|                                                      |
|  LINKEDIN POST (copy below):                        |
|  [Ready-to-paste LinkedIn post]                     |
|                                                      |
|  Category: [Topic Category]                         |
|  Word count: ~1,100                                 |
+------------------------------------------------------+
```

---

## MORNING REVIEW WORKFLOW (5 min)

| Step | Action | Time |
|------|--------|------|
| 1 | Open CMS edit link | 30s |
| 2 | Review draft, make any edits | 2 min |
| 3 | Click Publish | 10s |
| 4 | Open LinkedIn Newsletter file | 30s |
| 5 | Paste into LinkedIn Newsletter | 1 min |
| 6 | Click Publish | 10s |
| 7 | Copy LinkedIn Post text | 10s |
| 8 | Paste into LinkedIn feed post | 30s |
| 9 | Done | - |

---

## IMAGE GENERATION PROMPT TEMPLATE

```
Professional blog header image for article titled "[TITLE]"

Visual requirements:
- Include text "[YOUR BRAND]" prominently
- Include the blog title "[TITLE]" as main text
- Include your brand icon/logo
- Dark background with on-brand color patterns
- Modern, professional, tech aesthetic
- Wide cinematic format (1792x1024)

IMPORTANT: NO HANDS, NO FINGERS, NO PEOPLE, NO FACES.
Abstract visualization only.

Style: High-end corporate blog, minimalist, premium feel.
```

---

## FILE ORGANIZATION

| Asset | Path Pattern |
|-------|------|
| Blog drafts | `content/blog-drafts/YYYY-MM-DD-slug.md` |
| Blog images | `content/blog-images/slug.png` |
| LinkedIn newsletters | `content/linkedin-newsletters/YYYY-MM-DD-slug.md` |
| LinkedIn posts | `content/linkedin-posts/YYYY-MM-DD-slug.txt` |

---

## QUALITY CHECKLIST

- [ ] Blog post is 800-1200 words
- [ ] Written in brand voice (not generic AI)
- [ ] Clear industry angle with original insight
- [ ] Featured image follows 75% safe zone rule
- [ ] CTA at end of blog post
- [ ] LinkedIn newsletter version adapted (not copy-pasted)
- [ ] LinkedIn short post is 400-600 characters
- [ ] All links are correct
- [ ] Category assigned
- [ ] Draft status (NOT published until reviewed)

---

Built by [PureBrain](https://purebrain.ai) -- AI-powered marketing operations.

Want the full suite of 183+ production-tested skills? [Start your trial ->](https://purebrain.ai/pricing)
