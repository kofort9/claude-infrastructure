---
name: terminal-ui-design
description: Create distinctive, production-grade terminal user interfaces with high design quality. Use when building CLI tools, TUI applications, or terminal-based interfaces.
source: "[[sources/x/2025-12-28-cloudtrader4-terminal-ui-skill]]"
acquired: 2025-12-29
---

# Terminal UI Design Skill

Create distinctive, production-grade terminal user interfaces with high design quality. Generate creative, polished code that avoids generic terminal aesthetics.

## Design Thinking

Before coding, understand context and commit to a BOLD aesthetic direction:

1. **Purpose**: What problem does this interface solve? Who uses it? What's the workflow?
2. **Tone**: Pick an extreme:
   - hacker/cyberpunk
   - retro-computing (80s/90s)
   - minimalist zen
   - maximalist dashboard
   - synthwave neon
   - monochrome brutalist
   - corporate mainframe
   - playful/whimsical
   - matrix-style
   - steampunk terminal
   - vaporwave
   - military/tactical
   - art deco
   - paper-tape nostalgic
3. **Constraints**: Technical requirements (Python Rich, Go bubbletea, Rust ratatui, Node.js blessed/ink, pure ANSI escape codes, ncurses)
4. **Differentiation**: What makes this UNFORGETTABLE? What's the one thing someone will remember?

Choose a clear conceptual direction and execute with precision. Dense dashboards and zen single-focus interfaces both work—the key is intentionality, not intensity.

## Box Drawing & Borders

Choose border styles that match your aesthetic:

| Style | Characters | Vibe |
|-------|------------|------|
| Single line | `┌─┐│└┘` | Clean, modern |
| Double line | `╔═╗║╚╝` | Bold, formal, retro-mainframe |
| Rounded | `╭─╮│╰╯` | Soft, friendly, modern |
| Heavy | `┏━┓┃┗┛` | Strong, industrial |
| Dashed | `┄┆` | Light, airy, informal |
| ASCII only | `+-+\|` | Retro, universal compatibility |
| Block chars | `█▀▄▌▐` | Chunky, bold, brutalist |
| Custom Unicode | `◢◣◤◥`, `●○◐◑`, `▲▼` | Unique frames |

Avoid defaulting to simple single-line boxes. Consider asymmetric borders, double-thick headers, or decorative corners like `◆`, `◈`, `✦`, `⬡`.

## Color & Theme

Commit to a cohesive palette:

- **ANSI 16**: Classic, universal. Craft distinctive combinations beyond default red/green/blue
- **256-color**: Rich palettes. Color gradients, subtle background variations
- **True color (24-bit)**: Full spectrum. Gradient text, smooth transitions
- **Monochrome**: Single color with intensity variations. Elegant constraint

Create atmosphere with:
- Background color blocks for sections
- Gradient fills using `░▒▓█`
- Color-coded semantic meaning (avoid cliché red=bad, green=good)
- Inverted/reverse video for emphasis
- Dim text for secondary, bold for primary

## Typography & Text Styling

The terminal is ALL typography. Make it count:

- **ASCII art headers**: figlet-style banners, custom letterforms, Unicode art
- **Text weight**: Bold, dim, normal — create visual hierarchy
- **Text decoration**: Underline, strikethrough, italic (where supported)
- **Letter spacing**: Simulate with spaces: `H E A D E R`
- **Case**: ALL CAPS for headers, lowercase for body
- **Unicode symbols**: Enrich with `→ • ◆ ★ λ ∴ ≡ ⌘`
- **Custom bullets**: Replace `-` with `▸ ◉ ✓ ⬢ ›` or themed symbols

## Layout & Spatial Composition

Break free from single-column output:

- **Panels & Windows**: Create distinct regions with borders
- **Columns**: Side-by-side information using careful spacing
- **Tables**: Align data meaningfully, use Unicode table characters
- **Whitespace**: Generous padding, breathing room between sections
- **Density**: Match to purpose — dashboards dense, wizards sparse
- **Hierarchy**: Clear distinction between primary content, secondary info, chrome

## Motion & Animation

Terminals support dynamic content:

- **Spinners**: Beyond `|/-\`. Use Braille `⠋⠙⠹⠸⠼⠴⠦⠧⠇⠏`, dots `⣾⣽⣻⢿⡿⣟⣯⣷`, custom sequences
- **Progress bars**: `▓░`, `█▒`, `[=====>    ]`, or `◐◓◑◒`
- **Typing effects**: Reveal text character-by-character
- **Transitions**: Wipe effects, fade with color intensity
- **Live updates**: Streaming data, real-time charts

## Anti-Patterns to Avoid

NEVER use generic terminal aesthetics:
- Plain unformatted text output
- Default colors without intentional palette
- Basic `[INFO]`, `[ERROR]` prefixes without styling
- Simple `----` dividers
- Walls of unstructured text
- Generic progress bars without personality

**The terminal is a canvas with unique constraints and possibilities. Don't just print text—craft an experience.**
