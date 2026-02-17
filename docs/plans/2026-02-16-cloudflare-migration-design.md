# Cloudflare Website Migration & Redesign — Design Document

**Date:** 2026-02-16
**Status:** Approved

## Goal

Migrate personal academic website from Hugo Academic (GitHub Pages) to a modern Astro-based site on Cloudflare Pages, with a clean minimal UI targeting industry audiences.

## Audience

Recruiters and hiring managers at tech/biotech companies. The site should convey technical credibility while being approachable and easy to scan.

## Architecture

### Framework & Tooling

- **Astro** with content collections for blog posts (Markdown/MDX)
- **Tailwind CSS** for styling
- **TypeScript** (optional, recommended)
- Zero client JS by default (Astro islands only where needed)

### Pages

```
/                  → Single-page portfolio (Hero, Skills, Experience, Awards, Publications)
/blog              → Blog listing (paginated)
/blog/[slug]       → Individual blog posts
/cv.pdf            → Static CV download
```

### Deployment

- Git repo auto-deploys to Cloudflare Pages on push
- `*.pages.dev` subdomain initially; custom domain when ready
- Astro Cloudflare adapter for optimal builds

### Content Management

- Blog posts as `.md` or `.mdx` files in `src/content/blog/`
- Experience, publications, skills defined in structured data files (JSON/YAML)
- No CMS — edit files directly in Git

## Visual Design

### Color Palette (Warm Minimal)

| Role           | Color     | Usage                          |
|----------------|-----------|--------------------------------|
| Background     | `#FAFAF8` | Page background                |
| Surface        | `#FFFFFF` | Cards, blog post backgrounds   |
| Text Primary   | `#1A1A1A` | Headings, body text            |
| Text Secondary | `#6B6B6B` | Subtitles, dates, metadata     |
| Accent         | `#C4763B` | Links, buttons, highlights     |
| Accent Hover   | `#A85D2A` | Interactive states             |
| Accent Light   | `#F5E6D3` | Tags, badges, subtle BGs       |
| Border         | `#E8E4DF` | Dividers, card borders         |

### Typography

- **Headings & Body:** Inter
- **Code blocks:** JetBrains Mono
- Large heading sizes with generous line-height

### Navigation

- Fixed top bar: `Camellia Rui` (left) + `Blog` `CV` (right)
- Smooth scroll to homepage sections
- Collapsed hamburger on mobile

## Homepage Sections (Top to Bottom)

### 1. Hero/Bio (no avatar photo)

- "Camellia (Xinyue) Rui" — large heading, centered
- "PhD Candidate in Biostatistics · USC" — subtitle
- One-liner industry-oriented bio
- Social icons row: Email, LinkedIn, Twitter, ResearchGate, CV

### 2. Skills

- Clean pill-shaped tags or 3-column minimal grid
- Tags: Python, R, Statistics, Machine Learning, Jax
- Warm-toned badge styling using accent-light color

### 3. Experience

- Clean card layout (no timeline dots)
- Each card: Title, Organization, Date range, 2-3 bullet points
- Subtle terracotta left-border accent
- Most recent first

### 4. Accomplishments

- Compact list — no cards
- Award name + institution + year in clean rows
- Subtle dividers between entries

### 5. Publications

- Academic-style listing: Title (linked), authors, venue, year
- Optional small "PDF" / "DOI" pill buttons

### 6. Footer

- Name, copyright, social icons, "Built with Astro"

## Blog Design

### Listing Page (`/blog`)

- Simple list: title (linked), date + read time, 1-2 line excerpt, tag pills
- No thumbnail images
- 10 posts per page, paginated

### Post Page (`/blog/[slug]`)

- Max-width ~680px for readability
- Post title, date + read time + tags
- Markdown-rendered with styled code blocks, KaTeX math support, image captions
- "Back to blog" link at top
- No comments section

### Content Structure

```
src/content/blog/
  my-first-post.md       ← frontmatter: title, date, tags, excerpt
  research-summary.mdx   ← MDX for interactive elements if needed
```

## Content Migration

Migrate from current Hugo Academic site:
- Bio text (update: "PhD student" → "PhD candidate")
- 3 experience entries (PerturbVI, SCFM, Worldwide Imputation)
- 3 accomplishments (Keck Fellowship, Battat Scholarship, Provost Fellowship)
- Publications listing
- CV PDF file
- Skills (Python, R, Statistics)
- Social links (email, Twitter, LinkedIn, ResearchGate)

**Removed:** AboutMe page (personal photos section)
**Added:** Blog/Writing section (technical posts, research summaries, career reflections)

## Sections NOT Included

- AboutMe page with personal photos (removed per request)
- Netlify CMS admin panel
- Search functionality (not needed for this scale)
- Dark mode (not requested)
- Comments on blog posts
