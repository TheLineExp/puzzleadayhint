# A-Puzzle-A-Day Hint Generator

A web-based hint system for the DragonFjord "A-Puzzle-A-Day" puzzle. Instead of showing full solutions, it lets you progressively reveal hints by placing one piece at a time.

**Live Site:** https://www.puzzleadayhint.com (or https://icy-moss-05a1e991e.6.azurestaticapps.net)

## Features

- Select any date (month + day) to find all valid solutions
- Click a piece to see where it can be placed (highlighted in green)
- Place pieces one at a time - solutions filter as you go
- Shows count of remaining solutions after each placement
- Mobile-friendly responsive design
- Fast solver using bitmask operations (~50-200ms per date)

## The Puzzle

The A-Puzzle-A-Day puzzle has:
- **Board**: 7x7 grid with 6 blocked cells (43 usable cells)
- **8 Pieces**: 1 rectangle (6 cells) + 7 pentominoes (5 cells each) = 41 cells
- **Goal**: Cover all cells except the current month and day

## Project Structure

```
puzzle-hint-generator/
├── index.html                 # Single-file app (HTML + CSS + JS)
├── staticwebapp.config.json   # Azure Static Web Apps config
├── README.md                  # This file
└── .github/
    └── workflows/
        └── azure-static-web-apps-icy-moss-05a1e991e.yml  # Auto-deploy workflow
```

## Local Development

Just open `index.html` in a browser - no build step or server needed.

```bash
# Or use a local server
npx serve .
```

## Deployment

The site auto-deploys to Azure Static Web Apps on every push to `master`.

### Manual Deployment

```bash
# Get deployment token from Azure
az staticwebapp secrets list --name puzzleadayhint --resource-group Puzzleadayhint_group --query "properties.apiKey" -o tsv

# Deploy
swa deploy . --deployment-token "<token>" --env production
```

### Azure Resources

- **Resource Group**: `Puzzleadayhint_group`
- **Static Web App**: `puzzleadayhint`
- **Region**: West US 2
- **SKU**: Free
- **GitHub Repo**: https://github.com/TheLineExp/puzzleadayhint

## Custom Domain Setup

### Current Domains
- `www.puzzleadayhint.com` - CNAME to `icy-moss-05a1e991e.6.azurestaticapps.net`
- `puzzleadayhint.com` - Requires TXT validation for apex domain

### DNS Records (at your registrar)

| Type  | Name | Value |
|-------|------|-------|
| CNAME | www  | icy-moss-05a1e991e.6.azurestaticapps.net |
| ALIAS | @    | icy-moss-05a1e991e.6.azurestaticapps.net |

For apex domain validation:
```bash
az staticwebapp hostname show -n puzzleadayhint -g Puzzleadayhint_group --hostname puzzleadayhint.com --query "validationToken"
```
Add returned value as TXT record at `@`.

## Code Overview

### Piece Definitions

8 pieces defined as coordinate arrays, with rotations/reflections auto-generated:

```javascript
const PIECE_BASES = [
    { name: 'Rectangle', base: [[0,0],[0,1],[0,2],[1,0],[1,1],[1,2]], reflect: false },
    { name: 'U-shape',   base: [[0,0],[0,1],[1,0],[2,0],[2,1]], reflect: false },
    { name: 'Corner',    base: [[0,0],[0,1],[0,2],[1,0],[2,0]], reflect: false },
    { name: 'S-shape',   base: [[0,0],[1,0],[1,1],[2,1],[3,1]], reflect: true },
    { name: 'L-shape',   base: [[0,0],[1,0],[2,0],[3,0],[3,1]], reflect: true },
    { name: 'Z-shape',   base: [[0,0],[0,1],[1,1],[2,1],[2,2]], reflect: true },
    { name: 'T-shape',   base: [[0,0],[0,1],[0,2],[0,3],[1,1]], reflect: true },
    { name: 'P-shape',   base: [[0,0],[0,1],[1,0],[1,1],[2,0]], reflect: true }
];
```

### Solver Algorithm

1. **Bitmask representation**: Board state as 64-bit BigInt for O(1) collision detection
2. **Pre-computed placements**: All valid positions for each piece orientation calculated once
3. **Pruning**: Flood-fill detects isolated regions too small for remaining pieces
4. **Backtracking**: Recursive search through all piece placements

### Key Functions

| Function | Description |
|----------|-------------|
| `findAllSolutions(month, day)` | Returns array of all valid solutions |
| `generateOrientations(shape)` | Creates all rotations/reflections of a piece |
| `hasIsolatedRegion(board)` | Pruning check for unsolvable states |
| `showValidPlacements(pieceIdx)` | Highlights valid cells for selected piece |
| `placePiece(pieceIdx, placement)` | Places piece and filters remaining solutions |

## Making Changes

### Update Piece Shapes
Edit `PIECE_BASES` array in `index.html` (around line 242)

### Change Colors
Edit CSS variables and `.color-N` classes (lines 8-169)

### Modify Board Layout
Edit `BOARD_LABELS` and `BLOCKED` arrays (lines 185-196)

### Adjust Mobile Breakpoints
Edit `@media` queries at end of `<style>` block (lines 171-196)

## Troubleshooting

### Solver finds no solutions
- Check piece definitions total 41 cells
- Verify blocked cells leave 43 usable cells
- Ensure target date cells are valid

### Deployment fails
- Check GitHub Actions logs: https://github.com/TheLineExp/puzzleadayhint/actions
- Verify `app_location: "/"` in workflow file
- Ensure deployment token is valid

### Custom domain not working
- DNS propagation can take up to 48 hours
- Verify CNAME points to correct Azure hostname
- Check Azure Portal for validation status

## Resources

- [DragonFjord A-Puzzle-A-Day](https://www.dragonfjord.com/product/a-puzzle-a-day/)
- [Azure Static Web Apps Docs](https://docs.microsoft.com/en-us/azure/static-web-apps/)
- [Reference Solver (Rust)](https://github.com/anowell/today-puzzle)
