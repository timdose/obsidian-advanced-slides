# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
npm run dev        # Development build with watch mode (esbuild + SCSS + asset copying)
npm run build      # Production build (minified, no source maps)
npm run test       # Run Jest tests
npm run coverage   # Generate test coverage reports
npm run lint       # ESLint with auto-fix
npm run docs       # Serve Hugo documentation locally
```

To run a single test file:
```bash
npx jest test/filename.test.ts
```

## Architecture

**obsidian-advanced-slides** is an Obsidian plugin that transforms markdown notes into reveal.js presentations. It runs an embedded Express server to serve presentations and provides a live preview pane within Obsidian.

### Core Pipeline

The central abstraction is a **multi-pass processor chain** in `src/obsidianMarkdownPreprocessor.ts`. Markdown flows through 21+ sequential processors (each in `src/processors/`), covering:
- File inclusion: `MultipleFileProcessor`, `TemplateProcessor`
- Content transformation: `ImageProcessor`, `InternalLinkProcessor`, `LatexProcessor`, `MermaidProcessor`, `ExcalidrawProcessor`
- Slide formatting: `FormatProcessor`, `BlockProcessor`, `CalloutProcessor`, `GridProcessor`
- Special features: `FragmentProcessor`, `ChartProcessor`, `FootnoteProcessor`, `EmojiProcessor`

Processing iterates up to 9 times (circuit breaker) to handle template expansion that produces more templates.

### Key Files

| File | Role |
|---|---|
| `src/main.ts` | Plugin entry: Obsidian lifecycle, UI commands, ribbon, settings, autocomplete |
| `src/revealRenderer.ts` | Orchestrates the full render flow (YAML parse → processor chain → Mustache template → HTML) |
| `src/obsidianMarkdownPreprocessor.ts` | Assembles and runs the processor chain |
| `src/revealServer.ts` | Express server (default port 3000) that serves rendered presentations |
| `src/revealPreviewView.ts` | Obsidian webview pane with live preview |
| `src/revealExporter.ts` | PDF/HTML export |

### Transformers

`src/transformers/` implements a **visitor pattern** for HTML attribute transformation. The `Properties` class is a fluent builder; `AttributeTransformers` runs 11+ transformers (`ClassTransformer`, `StyleTransformer`, `BackgroundTransformer`, `GridTransformer`, `BorderTransformer`, etc.) in sequence on parsed element attributes.

### State / Singletons

`YamlStore` and `ImageCollector` use `getInstance()` singletons to share state across the processor pipeline during a single render pass.

### Build Output

esbuild compiles TypeScript to `main.js` and copies reveal.js, plugins, fonts, MathJax, and compiled SCSS themes into the plugin directory. A test vault is set up locally for manual testing.

### Tests

Jest with `ts-jest`. Six test files in `/test/` cover basic/extended syntax, templates, grids, comments, and split components.
