# Cloudflare Website Migration Implementation Plan

> **For Claude:** REQUIRED SUB-SKILL: Use superpowers:executing-plans to implement this plan task-by-task.

**Goal:** Migrate personal website from Hugo Academic (GitHub Pages) to Astro (Cloudflare Pages) with a warm minimal UI targeting industry audiences.

**Architecture:** Single-page portfolio homepage with separate blog section. Astro generates static HTML. Tailwind CSS handles styling. Content lives in structured data files (YAML) and Markdown blog posts. Deploys to Cloudflare Pages via Git push.

**Tech Stack:** Astro 5.x, Tailwind CSS 4.x, TypeScript, Cloudflare Pages, KaTeX (math), MDX (blog)

---

### Task 1: Scaffold Astro Project

**Files:**
- Create: `package.json`
- Create: `astro.config.mjs`
- Create: `tsconfig.json`
- Create: `tailwind.config.mjs` (if needed by Tailwind v4)
- Create: `src/pages/index.astro` (placeholder)

**Step 1: Initialize Astro project in the workspace root**

The workspace is at `/Users/camellia/conductor/workspaces/xinyueruihub.io.git/lagos/`. The existing static HTML files from the old Hugo site are here. We need to scaffold Astro alongside them — old files will be cleaned up in a later task.

Run:
```bash
cd /Users/camellia/conductor/workspaces/xinyueruihub.io.git/lagos
npm create astro@latest . -- --template minimal --no-install --typescript strict
```

If prompted about existing files, allow overwriting. The old HTML files are static output and not source — we'll clean them up later.

**Step 2: Install dependencies**

```bash
npm install
npm install @astrojs/tailwind @astrojs/mdx @astrojs/sitemap @astrojs/cloudflare
npm install tailwindcss @tailwindcss/typography
npm install katex rehype-katex remark-math
```

**Step 3: Configure Astro**

Write `astro.config.mjs`:
```js
import { defineConfig } from 'astro/config';
import tailwind from '@astrojs/tailwind';
import mdx from '@astrojs/mdx';
import sitemap from '@astrojs/sitemap';
import cloudflare from '@astrojs/cloudflare';
import remarkMath from 'remark-math';
import rehypeKatex from 'rehype-katex';

export default defineConfig({
  site: 'https://camellia-rui.pages.dev',
  output: 'static',
  adapter: cloudflare(),
  integrations: [tailwind(), mdx(), sitemap()],
  markdown: {
    remarkPlugins: [remarkMath],
    rehypePlugins: [rehypeKatex],
  },
});
```

Note: Check Astro 5.x docs — the Cloudflare adapter may only be needed for SSR. For `output: 'static'`, the adapter may not be required. If the build errors, remove the adapter line.

**Step 4: Create global CSS with Tailwind and custom theme**

Create `src/styles/global.css`:
```css
@import "tailwindcss";
@import "katex/dist/katex.min.css";

@theme {
  --color-bg: #FAFAF8;
  --color-surface: #FFFFFF;
  --color-text: #1A1A1A;
  --color-text-secondary: #6B6B6B;
  --color-accent: #C4763B;
  --color-accent-hover: #A85D2A;
  --color-accent-light: #F5E6D3;
  --color-border: #E8E4DF;

  --font-sans: 'Inter', system-ui, sans-serif;
  --font-mono: 'JetBrains Mono', monospace;
}
```

**Step 5: Create minimal index page to verify build**

Write `src/pages/index.astro`:
```astro
---
---
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <title>Camellia Rui</title>
  </head>
  <body class="bg-bg text-text font-sans">
    <h1 class="text-4xl text-accent">Site scaffold works</h1>
  </body>
</html>
```

**Step 6: Verify the build succeeds**

Run:
```bash
npm run build
```

Expected: Build succeeds, outputs to `dist/`. If it fails, check error messages and fix config.

**Step 7: Verify dev server works**

Run:
```bash
npm run dev -- --port 4321 &
sleep 3
curl -s http://localhost:4321 | head -20
kill %1
```

Expected: HTML output with "Site scaffold works".

**Step 8: Commit**

```bash
git add package.json package-lock.json astro.config.mjs tsconfig.json src/ src/styles/
git commit -m "feat: scaffold Astro project with Tailwind and Cloudflare config"
```

---

### Task 2: Create Base Layout and Navigation

**Files:**
- Create: `src/layouts/BaseLayout.astro`
- Create: `src/components/Nav.astro`
- Create: `src/components/Footer.astro`
- Modify: `src/pages/index.astro`

**Step 1: Create the base layout**

Write `src/layouts/BaseLayout.astro`:
```astro
---
interface Props {
  title: string;
  description?: string;
}

const { title, description = "Camellia (Xinyue) Rui — PhD Candidate in Biostatistics at USC" } = Astro.props;
---
<html lang="en">
  <head>
    <meta charset="utf-8" />
    <meta name="viewport" content="width=device-width, initial-scale=1" />
    <meta name="description" content={description} />
    <link rel="preconnect" href="https://fonts.googleapis.com" />
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500&display=swap" rel="stylesheet" />
    <title>{title}</title>
  </head>
  <body class="bg-bg text-text font-sans antialiased">
    <Nav />
    <main>
      <slot />
    </main>
    <Footer />
  </body>
</html>
```

Import `Nav` and `Footer` components at the top of the frontmatter.

**Step 2: Create the navigation component**

Write `src/components/Nav.astro`:
```astro
---
const currentPath = Astro.url.pathname;
---
<nav class="fixed top-0 w-full bg-bg/80 backdrop-blur-sm border-b border-border z-50">
  <div class="max-w-4xl mx-auto px-6 h-16 flex items-center justify-between">
    <a href="/" class="text-lg font-semibold text-text hover:text-accent transition-colors">
      Camellia Rui
    </a>
    <div class="flex items-center gap-6">
      <a href="/blog" class:list={["text-sm hover:text-accent transition-colors", currentPath.startsWith('/blog') ? 'text-accent' : 'text-text-secondary']}>
        Blog
      </a>
      <a href="/cv.pdf" class="text-sm text-text-secondary hover:text-accent transition-colors">
        CV
      </a>
    </div>
  </div>
</nav>
```

**Step 3: Create the footer component**

Write `src/components/Footer.astro`:
```astro
---
const year = new Date().getFullYear();
---
<footer class="border-t border-border mt-24">
  <div class="max-w-4xl mx-auto px-6 py-12 flex flex-col items-center gap-4 text-text-secondary text-sm">
    <div class="flex gap-5">
      <a href="mailto:crui@usc.edu" class="hover:text-accent transition-colors" aria-label="Email">
        <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24"><path d="M2 6a2 2 0 012-2h16a2 2 0 012 2v12a2 2 0 01-2 2H4a2 2 0 01-2-2V6zm2 0l8 5 8-5H4zm0 2.236V18h16V8.236l-8 5-8-5z"/></svg>
      </a>
      <a href="https://www.linkedin.com/in/camellia-rui-787ba7295/" target="_blank" rel="noopener" class="hover:text-accent transition-colors" aria-label="LinkedIn">
        <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24"><path d="M20.447 20.452h-3.554v-5.569c0-1.328-.027-3.037-1.852-3.037-1.853 0-2.136 1.445-2.136 2.939v5.667H9.351V9h3.414v1.561h.046c.477-.9 1.637-1.85 3.37-1.85 3.601 0 4.267 2.37 4.267 5.455v6.286zM5.337 7.433a2.062 2.062 0 01-2.063-2.065 2.064 2.064 0 112.063 2.065zm1.782 13.019H3.555V9h3.564v11.452zM22.225 0H1.771C.792 0 0 .774 0 1.729v20.542C0 23.227.792 24 1.771 24h20.451C23.2 24 24 23.227 24 22.271V1.729C24 .774 23.2 0 22.222 0h.003z"/></svg>
      </a>
      <a href="https://twitter.com/camellia_rui" target="_blank" rel="noopener" class="hover:text-accent transition-colors" aria-label="Twitter">
        <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24"><path d="M23.953 4.57a10 10 0 01-2.825.775 4.958 4.958 0 002.163-2.723c-.951.555-2.005.959-3.127 1.184a4.92 4.92 0 00-8.384 4.482C7.69 8.095 4.067 6.13 1.64 3.162a4.822 4.822 0 00-.666 2.475c0 1.71.87 3.213 2.188 4.096a4.904 4.904 0 01-2.228-.616v.06a4.923 4.923 0 003.946 4.827 4.996 4.996 0 01-2.212.085 4.936 4.936 0 004.604 3.417 9.867 9.867 0 01-6.102 2.105c-.39 0-.779-.023-1.17-.067a13.995 13.995 0 007.557 2.209c9.053 0 13.998-7.496 13.998-13.985 0-.21 0-.42-.015-.63A9.935 9.935 0 0024 4.59z"/></svg>
      </a>
      <a href="https://www.researchgate.net/profile/Camellia_Rui" target="_blank" rel="noopener" class="hover:text-accent transition-colors" aria-label="ResearchGate">
        <svg class="w-5 h-5" fill="currentColor" viewBox="0 0 24 24"><path d="M19.586 0c-1.36 0-2.591.676-3.322 1.756a5.31 5.31 0 00-.893 2.97v.002c0 .39.043.77.125 1.135.06.274.143.54.246.795l.012.03c.756 1.844 2.56 3.136 4.632 3.136h.003c.39 0 .77-.046 1.135-.13.275-.064.542-.15.798-.255 1.844-.768 3.127-2.578 3.127-4.643v-.003c0-.39-.046-.77-.13-1.135a5.24 5.24 0 00-.255-.798C24.296 1.016 22.092 0 19.586 0zM8.414 5.2c-1.36 0-2.59.676-3.322 1.756A5.31 5.31 0 004.2 9.926v.002c0 2.769 2.246 5.016 5.014 5.016h.003c.39 0 .77-.046 1.135-.13.275-.064.542-.15.798-.255.77-.32 1.443-.82 1.966-1.45.076-.092.15-.187.22-.284l.012-.018c.24-.34.44-.71.59-1.107l.013-.03c.103-.26.186-.53.246-.808.082-.365.125-.745.125-1.135v-.003c0-2.769-2.246-5.016-5.014-5.016h-.003c-.39 0-.77.046-1.135.13a5.24 5.24 0 00-.798.255c-.77.32-1.443.82-1.966 1.45a5.25 5.25 0 00-.22.284l-.012.018a5.28 5.28 0 00-.59 1.107l-.013.03a5.1 5.1 0 00-.246.808 5.1 5.1 0 00-.125 1.135v.003c0 2.769 2.246 5.016 5.014 5.016"/></svg>
      </a>
    </div>
    <p>&copy; {year} Camellia Rui. Built with Astro.</p>
  </div>
</footer>
```

**Step 4: Update index.astro to use the layout**

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
---
<BaseLayout title="Camellia Rui">
  <div class="pt-24 max-w-4xl mx-auto px-6">
    <h1 class="text-4xl font-bold">Coming soon</h1>
  </div>
</BaseLayout>
```

**Step 5: Build and verify**

```bash
npm run build
```

Expected: Build succeeds.

**Step 6: Commit**

```bash
git add src/layouts/ src/components/ src/pages/index.astro
git commit -m "feat: add base layout, nav, and footer components"
```

---

### Task 3: Create Data Files and Content Collections

**Files:**
- Create: `src/content.config.ts` (Astro 5.x content collections config)
- Create: `src/data/experience.yaml`
- Create: `src/data/accomplishments.yaml`
- Create: `src/data/publications.yaml`
- Create: `src/data/skills.yaml`
- Create: `src/data/social.yaml`
- Create: `src/content/blog/hello-world.md` (placeholder)

**Step 1: Create content collection config for blog**

Check Astro 5.x docs — in Astro 5, content collections are configured in `src/content.config.ts` (not `src/content/config.ts`). Write:

```ts
import { defineCollection, z } from 'astro:content';

const blog = defineCollection({
  type: 'content',
  schema: z.object({
    title: z.string(),
    date: z.date(),
    tags: z.array(z.string()).default([]),
    excerpt: z.string(),
    draft: z.boolean().default(false),
  }),
});

export const collections = { blog };
```

**Step 2: Create experience data**

Write `src/data/experience.yaml` — migrate from the existing HTML at `index.html:896-1015`:

```yaml
- title: "Research Assistant — PerturbVI"
  organization: "Prof. Nicholas Mancuso"
  organizationUrl: "https://www.mancusolab.com/"
  location: "Los Angeles, California"
  startDate: "Mar 2024"
  endDate: "Present"
  bullets:
    - "Helped developing a machine learning method PerturbVI that discovered gene regulatory networks with CRISPR perturbation data and single-cell RNA-seq data using Variational Inference and Jax in a team of three"
    - "Simulated model misspecification of latent variables using Python and improved 6.5% sensitivity compared to existing methods"
    - "Enabled ultra-fast inference speed with an average convergence time of 70x faster on the largest scale perturbation matrix (310,385 x 8563) than the existing method"

- title: "Research Assistant — Single Cell Fine Mapping"
  organization: "Prof. Nicholas Mancuso & Prof. Steven Gazal"
  organizationUrl: "https://www.mancusolab.com"
  location: "Los Angeles, California"
  startDate: "Aug 2022"
  endDate: "Present"
  bullets:
    - "Developed a machine learning method SCFM that identifies gene-to-disease associations on the largest-scale single-cell RNA-seq data (4.1GB), utilizing coordinate ascent variational inference"
    - "Achieved an average of 32% improvement in sensitivity and discovered an average of 15% more genetic variants when benchmarking against the existing method"
    - "Built a new Python package implementing SCFM framework with Jax for ultra-fast computing (15x faster, 1.3s vs 20s)"
    - "Enabled robustness on calibration and model misspecification over 4000+ simulation scenarios"
    - "Accepted as the first-author abstract to American Society of Human Genetics"

- title: "Research Assistant — Worldwide Imputation Analysis"
  organization: "Prof. Charleston Chiang"
  organizationUrl: "https://chianglab.usc.edu/"
  location: "Los Angeles, CA"
  startDate: "May 2020"
  endDate: "May 2022"
  bullets:
    - "Built a statistical analysis pipeline using Python and R for accessing genotype imputation quality over 123 populations"
    - "Discovered that imputation quality fell short 6.5%–42% in imputation R² among minority populations compared to European controls"
    - "Awarded Provost Research Fellowship twice (fall 2020 and fall 2021)"
    - "Published in top tier journal (AJHG, IF=12.6)"
```

**Step 3: Create accomplishments data**

Write `src/data/accomplishments.yaml`:

```yaml
- title: "Keck School of Medicine/Graduate School Fellowship"
  institution: "University of Southern California"
  date: "Aug 2022"
  description: "For incoming PhD students whose combination of background and training will make a substantive, documentable, and unique contribution to the program"

- title: "Jennifer Battat Scholarship"
  institution: "University of Southern California"
  date: "Jun 2020"
  description: "Recognized exceptional transfer students majoring in Economics or Mathematics"

- title: "Provost's Research Fellowship"
  institution: "University of Southern California"
  date: "Sep 2019"
  description: "Awarded 100 students at USC for excellent independent research with a faculty member"
```

**Step 4: Create publications data**

Write `src/data/publications.yaml`:

```yaml
- title: "Worldwide imputation analysis"
  authors: "Rui X, et al."
  venue: "American Journal of Human Genetics (AJHG)"
  year: 2022
  url: ""
  doi: ""
```

Note: This is a placeholder. The user should fill in complete publication details. The existing site links to `camelliarui.github.io/publication/` but the actual publication content isn't in the local files.

**Step 5: Create skills data**

Write `src/data/skills.yaml`:

```yaml
- name: "Python"
- name: "R"
- name: "Statistics"
- name: "Machine Learning"
- name: "Jax"
```

**Step 6: Create social links data**

Write `src/data/social.yaml`:

```yaml
- name: "Email"
  url: "mailto:crui@usc.edu"
  icon: "email"

- name: "LinkedIn"
  url: "https://www.linkedin.com/in/camellia-rui-787ba7295/"
  icon: "linkedin"

- name: "Twitter"
  url: "https://twitter.com/camellia_rui"
  icon: "twitter"

- name: "ResearchGate"
  url: "https://www.researchgate.net/profile/Camellia_Rui"
  icon: "researchgate"

- name: "CV"
  url: "/cv.pdf"
  icon: "cv"
```

**Step 7: Create placeholder blog post**

Write `src/content/blog/hello-world.md`:

```markdown
---
title: "Hello World"
date: 2026-02-16
tags: ["meta"]
excerpt: "Welcome to my new blog — a space for technical posts, research summaries, and career reflections."
draft: false
---

Welcome to my blog! I'll be writing about statistical genetics, machine learning, and my journey in biostatistics.

Stay tuned for more posts.
```

**Step 8: Install YAML loader**

Astro can import YAML files natively if configured, or use a JS/TS loader. Check if Astro 5.x supports YAML imports natively. If not:

```bash
npm install yaml
```

Alternatively, use `.json` files instead of `.yaml` — simpler with no extra dependency. Convert the YAML files to JSON if needed. The plan uses YAML for readability but JSON works identically.

**Step 9: Build and verify**

```bash
npm run build
```

Expected: Build succeeds with no content collection errors.

**Step 10: Commit**

```bash
git add src/content/ src/data/
git commit -m "feat: add content collections and structured data files"
```

---

### Task 4: Build Homepage Sections — Hero & Skills

**Files:**
- Create: `src/components/Hero.astro`
- Create: `src/components/Skills.astro`
- Create: `src/components/SocialIcons.astro`
- Modify: `src/pages/index.astro`

**Step 1: Create SocialIcons component**

Write `src/components/SocialIcons.astro` — a reusable row of social icon links. Use inline SVGs for each icon (email, linkedin, twitter, researchgate, cv). Read the icon name from the social data and render the corresponding SVG. Size them at `w-5 h-5` with `hover:text-accent` transitions.

**Step 2: Create Hero component**

Write `src/components/Hero.astro`:
```astro
---
import SocialIcons from './SocialIcons.astro';
---
<section class="pt-32 pb-16 text-center">
  <div class="max-w-2xl mx-auto px-6">
    <h1 class="text-5xl font-bold tracking-tight mb-3">
      Camellia (Xinyue) Rui
    </h1>
    <p class="text-xl text-text-secondary mb-6">
      PhD Candidate in Biostatistics · University of Southern California
    </p>
    <p class="text-base text-text-secondary leading-relaxed mb-8 max-w-lg mx-auto">
      I develop statistical and machine learning methods to elucidate genetic architecture of complex diseases. Working with
      <a href="https://www.mancusolab.com/" target="_blank" rel="noopener" class="text-accent hover:text-accent-hover">Prof. Nick Mancuso</a> and
      <a href="https://gazal-lab.org/" target="_blank" rel="noopener" class="text-accent hover:text-accent-hover">Prof. Steven Gazal</a>.
      Passionate about applying cutting-edge methods in industry.
    </p>
    <SocialIcons />
  </div>
</section>
```

**Step 3: Create Skills component**

Write `src/components/Skills.astro`:
```astro
---
import skills from '../data/skills.yaml';
---
<section id="skills" class="py-16">
  <div class="max-w-4xl mx-auto px-6">
    <h2 class="text-2xl font-semibold mb-6">Skills</h2>
    <div class="flex flex-wrap gap-3">
      {skills.map((skill) => (
        <span class="px-4 py-2 bg-accent-light text-accent text-sm font-medium rounded-full">
          {skill.name}
        </span>
      ))}
    </div>
  </div>
</section>
```

If YAML imports don't work natively, switch to importing a `.json` file or a `.ts` file that exports the data.

**Step 4: Wire into index.astro**

```astro
---
import BaseLayout from '../layouts/BaseLayout.astro';
import Hero from '../components/Hero.astro';
import Skills from '../components/Skills.astro';
---
<BaseLayout title="Camellia Rui">
  <Hero />
  <Skills />
</BaseLayout>
```

**Step 5: Build and verify**

```bash
npm run build
```

**Step 6: Visually verify with dev server**

```bash
npm run dev
```

Open in browser, check hero text is centered, skills pills render with warm colors.

**Step 7: Commit**

```bash
git add src/components/Hero.astro src/components/Skills.astro src/components/SocialIcons.astro src/pages/index.astro
git commit -m "feat: add Hero and Skills homepage sections"
```

---

### Task 5: Build Homepage Sections — Experience & Accomplishments

**Files:**
- Create: `src/components/Experience.astro`
- Create: `src/components/ExperienceCard.astro`
- Create: `src/components/Accomplishments.astro`
- Modify: `src/pages/index.astro`

**Step 1: Create ExperienceCard component**

Write `src/components/ExperienceCard.astro`:
```astro
---
interface Props {
  title: string;
  organization: string;
  organizationUrl: string;
  location: string;
  startDate: string;
  endDate: string;
  bullets: string[];
}
const { title, organization, organizationUrl, location, startDate, endDate, bullets } = Astro.props;
---
<div class="border-l-2 border-accent pl-6 pb-8">
  <h3 class="text-lg font-semibold">{title}</h3>
  <p class="text-sm text-text-secondary mt-1">
    <a href={organizationUrl} target="_blank" rel="noopener" class="text-accent hover:text-accent-hover">{organization}</a>
    <span class="mx-2">·</span>
    {startDate} – {endDate}
  </p>
  <ul class="mt-3 space-y-2 text-sm text-text-secondary list-disc list-inside">
    {bullets.map((b) => <li>{b}</li>)}
  </ul>
</div>
```

**Step 2: Create Experience section**

Write `src/components/Experience.astro`:
```astro
---
import ExperienceCard from './ExperienceCard.astro';
import experience from '../data/experience.yaml';
---
<section id="experience" class="py-16">
  <div class="max-w-4xl mx-auto px-6">
    <h2 class="text-2xl font-semibold mb-8">Experience</h2>
    <div class="space-y-2">
      {experience.map((exp) => (
        <ExperienceCard {...exp} />
      ))}
    </div>
  </div>
</section>
```

**Step 3: Create Accomplishments section**

Write `src/components/Accomplishments.astro`:
```astro
---
import accomplishments from '../data/accomplishments.yaml';
---
<section id="accomplishments" class="py-16">
  <div class="max-w-4xl mx-auto px-6">
    <h2 class="text-2xl font-semibold mb-8">Accomplishments</h2>
    <div class="divide-y divide-border">
      {accomplishments.map((a) => (
        <div class="py-4">
          <div class="flex flex-col sm:flex-row sm:items-baseline sm:justify-between gap-1">
            <h3 class="text-base font-medium">{a.title}</h3>
            <span class="text-sm text-text-secondary whitespace-nowrap">{a.date}</span>
          </div>
          <p class="text-sm text-text-secondary mt-1">{a.institution}</p>
        </div>
      ))}
    </div>
  </div>
</section>
```

**Step 4: Wire into index.astro**

Add imports for `Experience` and `Accomplishments` and add them after `<Skills />`.

**Step 5: Build and verify**

```bash
npm run build
```

**Step 6: Commit**

```bash
git add src/components/Experience.astro src/components/ExperienceCard.astro src/components/Accomplishments.astro src/pages/index.astro
git commit -m "feat: add Experience and Accomplishments homepage sections"
```

---

### Task 6: Build Homepage Section — Publications

**Files:**
- Create: `src/components/Publications.astro`
- Modify: `src/pages/index.astro`

**Step 1: Create Publications component**

Write `src/components/Publications.astro`:
```astro
---
import publications from '../data/publications.yaml';
---
<section id="publications" class="py-16">
  <div class="max-w-4xl mx-auto px-6">
    <h2 class="text-2xl font-semibold mb-8">Publications</h2>
    <div class="space-y-6">
      {publications.map((pub) => (
        <div>
          <h3 class="text-base font-medium">
            {pub.url ? (
              <a href={pub.url} target="_blank" rel="noopener" class="text-accent hover:text-accent-hover">
                {pub.title}
              </a>
            ) : pub.title}
          </h3>
          <p class="text-sm text-text-secondary mt-1">
            {pub.authors} · {pub.venue}, {pub.year}
          </p>
          {(pub.doi || pub.url) && (
            <div class="flex gap-2 mt-2">
              {pub.doi && (
                <a href={`https://doi.org/${pub.doi}`} target="_blank" rel="noopener" class="inline-block px-3 py-1 text-xs bg-accent-light text-accent rounded-full hover:bg-accent hover:text-white transition-colors">
                  DOI
                </a>
              )}
              {pub.url && (
                <a href={pub.url} target="_blank" rel="noopener" class="inline-block px-3 py-1 text-xs bg-accent-light text-accent rounded-full hover:bg-accent hover:text-white transition-colors">
                  PDF
                </a>
              )}
            </div>
          )}
        </div>
      ))}
    </div>
  </div>
</section>
```

**Step 2: Wire into index.astro**

Add `Publications` after `Accomplishments`.

**Step 3: Build and verify**

```bash
npm run build
```

**Step 4: Commit**

```bash
git add src/components/Publications.astro src/pages/index.astro
git commit -m "feat: add Publications homepage section"
```

---

### Task 7: Build Blog Pages

**Files:**
- Create: `src/pages/blog/index.astro`
- Create: `src/pages/blog/[slug].astro`
- Create: `src/layouts/BlogPost.astro`

**Step 1: Create blog listing page**

Write `src/pages/blog/index.astro`:
```astro
---
import BaseLayout from '../../layouts/BaseLayout.astro';
import { getCollection } from 'astro:content';

const posts = (await getCollection('blog', ({ data }) => !data.draft))
  .sort((a, b) => b.data.date.valueOf() - a.data.date.valueOf());
---
<BaseLayout title="Blog — Camellia Rui">
  <div class="pt-28 pb-16 max-w-2xl mx-auto px-6">
    <h1 class="text-3xl font-bold mb-12">Blog</h1>
    <div class="space-y-10">
      {posts.map((post) => (
        <article>
          <a href={`/blog/${post.id}`} class="group">
            <h2 class="text-xl font-semibold group-hover:text-accent transition-colors">
              {post.data.title}
            </h2>
          </a>
          <p class="text-sm text-text-secondary mt-1">
            {post.data.date.toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' })}
          </p>
          <p class="text-text-secondary mt-2">{post.data.excerpt}</p>
          <div class="flex gap-2 mt-3">
            {post.data.tags.map((tag) => (
              <span class="px-2 py-0.5 text-xs bg-accent-light text-accent rounded-full">{tag}</span>
            ))}
          </div>
        </article>
      ))}
    </div>
  </div>
</BaseLayout>
```

Note: In Astro 5.x, the `slug` field may be replaced by `id`. Check the Astro 5 content collections API. Use `post.id` or `post.slug` as appropriate.

**Step 2: Create blog post layout**

Write `src/layouts/BlogPost.astro`:
```astro
---
import BaseLayout from './BaseLayout.astro';

interface Props {
  title: string;
  date: Date;
  tags: string[];
}

const { title, date, tags } = Astro.props;
---
<BaseLayout title={`${title} — Camellia Rui`}>
  <article class="pt-28 pb-16 max-w-[680px] mx-auto px-6">
    <a href="/blog" class="text-sm text-text-secondary hover:text-accent transition-colors mb-8 inline-block">
      &larr; Back to blog
    </a>
    <h1 class="text-3xl font-bold mt-4 mb-3">{title}</h1>
    <div class="flex items-center gap-3 text-sm text-text-secondary mb-8">
      <time>{date.toLocaleDateString('en-US', { year: 'numeric', month: 'long', day: 'numeric' })}</time>
      <div class="flex gap-2">
        {tags.map((tag) => (
          <span class="px-2 py-0.5 text-xs bg-accent-light text-accent rounded-full">{tag}</span>
        ))}
      </div>
    </div>
    <div class="prose prose-lg max-w-none prose-headings:text-text prose-a:text-accent prose-a:no-underline hover:prose-a:text-accent-hover prose-code:font-mono prose-code:text-sm prose-pre:bg-surface prose-pre:border prose-pre:border-border">
      <slot />
    </div>
  </article>
</BaseLayout>
```

**Step 3: Create dynamic blog post page**

Write `src/pages/blog/[slug].astro`:
```astro
---
import { getCollection, render } from 'astro:content';
import BlogPost from '../../layouts/BlogPost.astro';

export async function getStaticPaths() {
  const posts = await getCollection('blog');
  return posts.map((post) => ({
    params: { slug: post.id },
    props: { post },
  }));
}

const { post } = Astro.props;
const { Content } = await render(post);
---
<BlogPost title={post.data.title} date={post.data.date} tags={post.data.tags}>
  <Content />
</BlogPost>
```

Note: In Astro 5.x, `render()` is imported from `astro:content` and called as `render(entry)` rather than `entry.render()`. Verify against Astro 5 docs.

**Step 4: Build and verify**

```bash
npm run build
```

Check that `/blog/index.html` and `/blog/hello-world/index.html` are generated in `dist/`.

**Step 5: Commit**

```bash
git add src/pages/blog/ src/layouts/BlogPost.astro
git commit -m "feat: add blog listing and post pages"
```

---

### Task 8: Static Assets and Cleanup

**Files:**
- Copy: `file/cv.pdf` → `public/cv.pdf`
- Create: `public/favicon.svg` (or copy existing favicon)
- Remove: Old Hugo-generated files (index.html at root, css/, js/, admin/, etc.)

**Step 1: Set up public directory**

```bash
mkdir -p public
cp file/cv.pdf public/cv.pdf
```

**Step 2: Create a simple favicon**

Create `public/favicon.svg` — a simple terracotta "C" monogram, or copy the existing favicon from `images/`. For now a simple SVG:

```svg
<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 32 32">
  <rect width="32" height="32" rx="6" fill="#C4763B"/>
  <text x="16" y="22" font-family="Inter, sans-serif" font-size="18" font-weight="700" fill="white" text-anchor="middle">C</text>
</svg>
```

Add the favicon reference to BaseLayout.astro `<head>`:
```html
<link rel="icon" type="image/svg+xml" href="/favicon.svg" />
```

**Step 3: Add robots.txt**

Create `public/robots.txt`:
```
User-agent: *
Allow: /
Sitemap: https://camellia-rui.pages.dev/sitemap-index.xml
```

**Step 4: Clean up old Hugo files**

Remove the old static site files that are no longer needed. Be careful — only remove files that are clearly Hugo output, not the new Astro source:

```bash
# Remove old Hugo output files
rm -f index.html 404.html index.json index.xml sitemap.xml index.webmanifest manifest.webmanifest
rm -rf css/ js/ admin/ images/ img/ author/ authors/ post/ publication/ publication_types/ talk/ tags/ categories/ aboutme/ file/ files/ camelliarui.github.io/
```

**Important:** Run `git status` after this to verify only old Hugo files were removed. The `src/`, `public/`, `docs/`, `package.json`, `astro.config.mjs`, etc. should NOT be affected.

**Step 5: Update .gitignore**

Check if `.gitignore` exists. If not, create one:
```
node_modules/
dist/
.astro/
.DS_Store
.context/
```

**Step 6: Build and verify everything still works**

```bash
npm run build
```

**Step 7: Commit**

```bash
git add -A
git commit -m "chore: add static assets and remove old Hugo files"
```

---

### Task 9: Cloudflare Pages Configuration

**Files:**
- Create: `wrangler.toml` (optional, for Cloudflare settings)
- Verify: build settings for Cloudflare Pages

**Step 1: Verify build command works for Cloudflare**

Cloudflare Pages needs:
- **Build command:** `npm run build`
- **Build output directory:** `dist`
- **Node.js version:** 22.x (set via environment variable or `.node-version`)

Create `.node-version`:
```
22
```

**Step 2: Test production build**

```bash
npm run build
ls dist/
```

Verify `dist/` contains:
- `index.html`
- `blog/index.html`
- `blog/hello-world/index.html`
- `cv.pdf`
- `favicon.svg`

**Step 3: Commit**

```bash
git add .node-version
git commit -m "chore: add Cloudflare Pages build configuration"
```

---

### Task 10: Final Polish and Verification

**Step 1: Run dev server and visually verify all sections**

```bash
npm run dev
```

Check:
- [ ] Navigation fixed at top, transparent/blurred background
- [ ] Hero section: name, subtitle, bio, social icons
- [ ] Skills: warm-toned pill badges
- [ ] Experience: cards with terracotta left border
- [ ] Accomplishments: clean list with dividers
- [ ] Publications: linked titles with pill buttons
- [ ] Footer: social icons, copyright
- [ ] Blog listing: posts with titles, dates, excerpts, tags
- [ ] Blog post: readable width, styled prose, back link
- [ ] CV link works (downloads PDF)
- [ ] Mobile responsive (check at 375px width)

**Step 2: Fix any visual issues**

Adjust spacing, colors, or typography as needed.

**Step 3: Final build**

```bash
npm run build
```

**Step 4: Commit any final fixes**

```bash
git add -A
git commit -m "polish: final visual adjustments"
```

---

## Deployment

After all tasks are complete, deploy to Cloudflare Pages:

1. Push the branch to GitHub
2. Go to Cloudflare Dashboard → Pages → Create a project
3. Connect your GitHub repo
4. Set build command: `npm run build`
5. Set build output: `dist`
6. Deploy

The site will be available at `https://<project-name>.pages.dev`.

---

## Summary of Tasks

| Task | Description | Key Files |
|------|-------------|-----------|
| 1 | Scaffold Astro project | package.json, astro.config.mjs, global.css |
| 2 | Base layout & navigation | BaseLayout, Nav, Footer |
| 3 | Data files & content collections | YAML data, blog collection, hello-world post |
| 4 | Hero & Skills sections | Hero, Skills, SocialIcons components |
| 5 | Experience & Accomplishments | ExperienceCard, Experience, Accomplishments |
| 6 | Publications section | Publications component |
| 7 | Blog pages | Blog listing, post layout, dynamic routes |
| 8 | Static assets & cleanup | CV PDF, favicon, remove old Hugo files |
| 9 | Cloudflare config | .node-version, build verification |
| 10 | Final polish | Visual verification, responsive check |
