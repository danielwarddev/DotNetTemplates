# .NET Templates

My personal `dotnet new` templates with the packages, project layout, and setup scripts I usually like to start from. Also wraps the project creation in a bash script that updates the Nuget packages to the latest stable version afterwards.

For the most part, they are completely unrelated to AI and can be used separately. I only them "AI-ready" for 2 reasons:

1. The templates include a premade `CLAUDE.md` and `.claude` folder with project instructions, skills, and helper scripts I like to use.
2. The `scripts` folder provides a more reliable way for AI to create projects for me. I find that without them, what AI creates for new projects is a bit of a gamble (old .NET versions, old Nuget package versions, different patterns, etc.).
3. Bonus reason - it sounds cool.

## Available templates

| Template | Short name | What it is | Script |
| --- | --- | --- | --- |
| AI-Ready .NET Console App | `ai-ready-console` | Console app template with a sibling unit-test project and shared AI/project instructions. | `scripts\new-ai-ready-console-app` |
| AI-Ready MudBlazor App | `ai-ready-mudblazor` | Blazor/MudBlazor app template with a sibling unit-test project, shared AI/project instructions, and Blazor-specific guidance. | `scripts\new-ai-ready-mudblazor-app` |

## How to use

Install the templates you want with `dotnet new install`:

```powershell
dotnet new install .\AiReadyConsoleApp
dotnet new install .\AiReadyMudBlazorApp
```

Then, copy files in the `scripts` folder (or the folder itself) somewhere on your `PATH`.

You can then create projects with the wrapper scripts:

```bash
new-ai-ready-console-app -n MyConsoleApp
new-ai-ready-mudblazor-app -n MyBlazorApp
```

Both scripts also accept `-o` / `--output` to choose the parent output directory. You probably don't need to do this, since by default it creates an output directory for you with the name of the app.

### Adding to an existing solution

Both scripts accept `--skip-solution`. This omits the solution-level scaffolding (the `.slnx` solution, `Directory.Build.props`, `CLAUDE.md`, and `.claude/`) and emits only the two project folders, so you can drop them into a solution you already have:

```bash
new-ai-ready-console-app -n MyConsoleApp --skip-solution
dotnet sln path/to/Existing.slnx add \
    MyConsoleApp/MyConsoleApp/MyConsoleApp.csproj \
    MyConsoleApp/MyConsoleApp.UnitTests/MyConsoleApp.UnitTests.csproj
```

## Info on the scripts

`scripts\new-ai-ready-app` is the internal base script behind the template-specific wrappers. It's not intended to be run manually; use the wrapper scripts (eg `new-ai-ready-console-app` or `new-ai-ready-mudblazor-app`) instead.

Each wrapper also runs `scripts\update-packages` after project creation. This solves one of the weakness of `dotnet new` templates: they cannot automatically pull the latest stable package versions of Nuget packages at creation time. This script fixes that by scanning the generated folder for all `.csproj` files, reading the `PackageReference` names from each one, and running `dotnet add package` for each one, which will set the package version to the latest stable.

## Included skills

| Skill | Description |
| --- | --- |
| `coding-standards` | General .NET architecture, C# conventions, project structure, and focused implementation guidance. |
| `testing-standards` | Unit/integration test conventions, test project selection, naming, assertions, and verification workflow. |
| `spec-creation` | Creating small, reviewable feature specs under `.specs/`. |
| `spec-execution` | Implementing full specs or individual numbered spec sections. |
| `blazor-frontend` (if relevant) | Blazor Server, MudBlazor, Razor component patterns, UI states, bUnit tests, rendering performance, and browser verification. |
