# Copilot Workspace Instructions

**Soc Ops** is a Social Bingo game built with Blazor WebAssembly (.NET 10). Players find people who match the questions to mark squares and get 5 in a row. This is an interactive workshop demonstrating GitHub Copilot capabilities for multi-agent development.

---

## Project Overview

### Purpose
- **Workshop Lab**: Demonstrates GitHub Copilot agents in practical scenarios (context engineering, design, custom agents, multi-agent workflows)
- **Game**: Social Bingo mixer game for in-person events
- **Live Demo**: https://dotnet-presentations.github.io/vscode-github-copilot-agent-lab/

### Key Technologies
- **Framework**: Blazor WebAssembly (.NET 10)
- **Architecture**: Component-based (MVVM-ish) with services for state & logic
- **Storage**: Browser localStorage (via JSInterop)
- **Styling**: Custom CSS utility classes (Tailwind-like patterns)

---

## Project Structure

```
SocOps/
├── Program.cs                # WebAssembly entry point, service registration
├── App.razor                 # Root component router
├── Components/
│   ├── StartScreen.razor     # Initial UI for new games
│   ├── GameScreen.razor      # Main game board container
│   ├── BingoBoard.razor      # 5x5 grid renderer
│   ├── BingoSquare.razor     # Individual square with toggle state
│   └── BingoModal.razor      # Victory/Bingo announcement modal
├── Pages/
│   ├── Home.razor            # Main page route
│   ├── Counter.razor         # Example component
│   └── Weather.razor         # Example component
├── Services/
│   ├── BingoGameService.cs   # State management, localStorage persistence, events
│   └── BingoLogicService.cs  # Core game logic (board generation, win detection)
├── Models/
│   ├── GameState.cs          # Enum: Start, Playing, Won
│   ├── BingoSquareData.cs    # DTO: Id, Question, IsMarked
│   └── BingoLine.cs          # DTO: Row/Column/Diagonal winner
├── Data/
│   └── Questions.cs          # Static list of bingo questions
├── Layout/
│   └── MainLayout.razor      # App shell with nav
├── wwwroot/
│   ├── index.html            # HTML entry point
│   ├── css/app.css           # Utility classes & component styles
│   └── lib/bootstrap/        # Bootstrap CSS (included by default)
└── Properties/
    └── launchSettings.json   # Dev server config (port 5166)
```

### Support Folder: Workshop Lab Guide
```
workshop/
├── 00-overview.md            # Lab checklist & intro
├── 01-setup.md               # Workspace instructions (Part 1)
├── 02-design.md              # Design-first frontend (Part 2)
├── 03-quiz-master.md         # Custom quiz agent (Part 3)
├── 04-multi-agent.md         # Multi-agent development (Part 4)
└── GUIDE.md                  # Quick reference
```

---

## Development Setup

### Prerequisites
- .NET 10 SDK ([download](https://dotnet.microsoft.com/download/dotnet/10.0))
- VS Code with GitHub Copilot Chat
- Git

### Quick Start
```bash
# Restore dependencies & build
cd SocOps
dotnet restore && dotnet build

# Run dev server (opens http://localhost:5166)
dotnet run

# OR from workspace root:
dotnet run --project SocOps
```

---

## Key Commands

| Command | Purpose |
|---------|---------|
| `dotnet build` | Compile project |
| `dotnet run` | Start dev server on port 5166 |
| `dotnet run --no-build` | Run without recompiling |
| `dotnet test` | Run tests (when implemented) |
| `dotnet clean` | Remove build artifacts |
| `dotnet publish` | Release build for deployment |

---

## Architecture & Patterns

### Service Layer (State Management)
**BingoGameService** is the centralized state container:
- Manages `CurrentGameState`, `Board`, `WinningLine`, `ShowBingoModal`
- Persists state to browser localStorage via JSInterop
- Publishes `OnStateChanged` event for component subscriptions
- Components call public methods like `StartGame()`, `MarkSquare()`, `Reset()`

**Anti-pattern**: Don't store game state in component @code blocks; use BingoGameService instead.

### Game Logic Layer
**BingoLogicService** contains pure, stateless logic:
- `GenerateBoard()` - Create 5x5 bingo card with random questions
- `ToggleSquare()` - Mark/unmark a square
- `CheckForWin()` - Detect 5-in-a-row (rows, columns, diagonals)
- `GetWinningSquareIds()` - Return highlighted winning cells

### Component Hierarchy
```
<App>                      # Router
  <MainLayout>             # App shell
    <HomeRoute>
      <StartScreen>        # Show if GameState == Start
      <GameScreen>         # Show if GameState == Playing
        <BingoBoard>
          <BingoSquare> (25x)
        <BingoModal>       # Victory announcement
```

---

## Styling Conventions

### CSS Architecture
- **Utility-first**: Use classes like `.flex`, `.p-4`, `.mb-2` (Tailwind-inspired)
- **Component scoped**: `.razor.css` files co-located with components
- **Global styles**: `wwwroot/css/app.css`

### Common Classes
```css
/* Layout */
.flex, .grid, .items-center, .justify-center, .gap-2, .gap-4

/* Spacing */
.p-2, .p-4, .mb-2, .mb-4, .mt-2, .mx-auto

/* Colors */
.bg-accent     /* Primary action color */
.bg-marked     /* Selected/marked square */
.bg-gray-100   /* Light backgrounds */
.text-gray-700 /* Text color */

/* Interactive */
.cursor-pointer
.hover:bg-opacity-80
.transition
```

**Naming**: Use kebab-case (`.my-class`), no underscores.

---

## Naming Conventions

| Item | Convention | Example |
|------|-----------|---------|
| C# Classes | PascalCase | `BingoGameService`, `GameState` |
| C# Methods | PascalCase | `StartGame()`, `MarkSquare()` |
| C# Properties | PascalCase | `CurrentGameState`, `Board` |
| C# Fields (private) | _camelCase | `_jsRuntime`, `_logger` |
| Razor Components | PascalCase `.razor` | `BingoBoard.razor` |
| CSS Classes | kebab-case | `.bg-marked`, `.flex` |
| Events (C#) | PascalCase | `OnStateChanged` |
| Routes | lowercase | `/` (home), `/counter` |

---

## State Flow Example

```
1. User clicks StartScreen "Play"
   ↓
2. BingoGameService.StartGame() called
   ├─ Set Board = random 25 squares
   ├─ Set CurrentGameState = Playing
   ├─ Save to localStorage
   └─ Call NotifyStateChanged()
   ↓
3. Components re-render via event subscription
   ├─ GameScreen shows BingoBoard
   └─ BingoBoard renders BingoSquare components
   ↓
4. User clicks square
   ├─ BingoSquare calls service MarkSquare()
   ├─ Service updates board[i].IsMarked = !IsMarked
   ├─ Checks if any 5-in-a-row exists
   ├─ If won: WinningLine set, ShowBingoModal = true
   └─ NotifyStateChanged()
   ↓
5. Components re-render with new state
```

---

## Common Tasks

### Add a New Bingo Question
**File**: `Data/Questions.cs`
```csharp
public static class Questions
{
    public static List<string> GetQuestions() => new()
    {
        "Has attended a tech conference",
        "YOUR_NEW_QUESTION",  // Add here
        // ...
    };
}
```

### Add a New UI Component
1. Create `Components/MyComponent.razor` (PascalCase)
2. Add `@inject BingoGameService GameService` if needs state
3. Subscribe to state changes: `protected override async Task OnInitializedAsync() { GameService.OnStateChanged += StateHasChanged; }`
4. Add styling in `MyComponent.razor.css`
5. Import in `_Imports.razor` if shared across multiple pages

### Modify Game Logic
**File**: `Services/BingoLogicService.cs` - Keep stateless, pure functions.
**Testing**: Add unit tests in `SocOps.Tests/BingoLogicServiceTests.cs` (TDD pattern).

### Change Dev Server Port
**File**: `Properties/launchSettings.json`, update `"applicationUrl": "https://localhost:XXXX"`

---

## Development Best Practices

### Before Committing
- [ ] `dotnet build` passes with no errors
- [ ] Code uses PascalCase for public members
- [ ] No unused using statements
- [ ] Components properly dispose of event subscriptions
- [ ] CSS follows kebab-case convention
- [ ] Browser dev tools show no console errors

### Performance Tips
- Component reuse: Reuse components like `<BingoSquare>` rather than duplicating markup
- Event debouncing: If many rapid state changes, consider debouncing
- Lazy loading: Consider lazy-loading heavy components (Counter, Weather pages not needed at startup)

### Common Issues & Solutions

| Issue | Solution |
|-------|----------|
| Changes not reflecting in browser | Hard refresh (Ctrl+Shift+R), check localStorage in DevTools |
| "Service not found" error | Ensure service registered in `Program.cs` with `AddScoped<>()` |
| Component not re-rendering | Verify event subscription in `OnInitializedAsync()` and disposal in `Dispose()` |
| localhost:5166 already in use | Change port in `launchSettings.json` or `dotnet run -- --urls "https://localhost:XXXX"` |
| Build fails with CS0103 | Missing `using` statement or misspelled type name |

---

## Workshop Context

This project supports a multi-part lab:

| Part | Focus | Agent Tasks |
|------|-------|------------|
| **01** | Context Engineering | Generate instructions, run linting, update README |
| **02** | Design-First Frontend | AI-designed UI themes, CSS refactoring |
| **03** | Custom Quiz Agent | Themed bingo questions generator |
| **04** | Multi-Agent Development | Scavenger Hunt (TDD), Card Deck (Pixel Jam), UX Review |

### Running Lab Parts with Copilot
- `/setup` - Bootstrap dev environment
- `@workspace` - Reference project context in chat
- Generate steps incrementally, commit each part

---

## Resources

- [Lab Guide](../workshop/GUIDE.md)
- [Live Demo](https://dotnet-presentations.github.io/vscode-github-copilot-agent-lab/)
- [GitHub Actions Deploy](../../.github/workflows/) - Publishes to GitHub Pages
- [C# Naming Guidelines](https://learn.microsoft.com/en-us/dotnet/csharp/fundamentals/coding-style/naming-conventions)
- [Blazor Documentation](https://learn.microsoft.com/en-us/aspnet/core/blazor)

---

## Last Updated
March 2026 | .NET 10 | Blazor WebAssembly
