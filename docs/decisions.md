# Decision Log

Running record of decisions made while building this task.
Chronological, newest at the bottom. Each entry: what was decided,
why, and (where it matters) what it costs.

Not every choice is an entry. Only ones a reviewer could reasonably
challenge. Routine mechanics like running `dotnet new` or installing
xUnit do not belong here.

---

## 2026-10-16, Day 1: Environment and project skeleton

### Target framework: .NET 10
- **Decision:** target `net10.0` across the solution.
- **Why:** .NET 10 shipped November 2025 and is the current LTS
  release. It has a longer support window than .NET 9. I had not used
  it before, so this is a good chance to work with the newest version.
- **Cost:** opening the solution needs VS 2022 17.10 or newer, or VS
  2026. Not an issue here.
- **Reference:** [.NET support policy](https://dotnet.microsoft.com/en-us/platform/support/policy/dotnet-core)

### Two projects only (`StockAllocation.Domain`, `StockAllocation.Domain.Tests`)
- **Decision:** no `Application`, `Infrastructure`, or `Contracts` project.
- **Why:** this is a pure domain library. No REST API, no database,
  no messaging. There is nothing to mock. Extra layers would exist
  only to abstract things that are not there. An empty project is a
  review red flag.
- **Cost:** if I need persistence later I would add `Infrastructure`
  then. The brief forbids it, so the cost is theoretical.

### Folders are named after the business, not after patterns
- **Decision:** `Orders/`, `Stock/`, `Allocation/`. Not `Entities/`,
  `Services/`, `ValueObjects/`.
- **Why:** the code should use the same words the business uses. Open
  `src/` and the three folders show what the task is about. Pattern
  folders tell the reader what the code is, not what it is about. The
  reader has to assemble the problem from scratch.
- **Cost:** a class that touches more than one subject does not have an
  obvious home. The allocation logic reads orders and stock, so where
  does it go. I pick one, the reviewer may disagree.

### `Directory.Build.props` with warnings as errors and nullable enabled
- **Decision:** shared build settings live in `Directory.Build.props`
  at the repo root: `TargetFramework`, `Nullable`, `ImplicitUsings`,
  `TreatWarningsAsErrors`, `LangVersion`. Each `.csproj` inherits.
- **Why:** one file means no drift between projects when a setting
  changes. `Nullable=enable` turns "is this field optional?" into a
  compile-time decision. Writing `Sku Sku` on a line item says the SKU
  is required. Writing `Sku? Sku` says it may be absent. The compiler
  forces me to pick. `TreatWarningsAsErrors=true` means a forgotten
  decision breaks the build.
- **Cost:** experimenting is slower. Half-written code with a warning
  will not build. Third-party warnings I cannot fix need an explicit
  suppression in code.
- **Reference:** [Customize your build (Directory.Build.props)](https://learn.microsoft.com/en-us/visualstudio/msbuild/customize-your-build), [Nullable reference types](https://learn.microsoft.com/en-us/dotnet/csharp/nullable-references)

### Solution file is `.slnx`, not `.sln`
- **Decision:** kept the default `.slnx` emitted by `dotnet new sln` on
  .NET 10.
- **Why:** the XML format reads and diffs cleanly. It is the SDK
  default on .NET 10. VS 2022 17.10 and newer and VS 2026 both open it.
- **Cost:** anyone on an older VS cannot open the solution. Not an
  issue for this submission.

### Solution created from the terminal, not the New Project wizard
- **Decision:** scaffolded the solution with `dotnet new sln`,
  `dotnet new classlib`, and `dotnet new xunit` from the terminal.
- **Why:** the README must list the exact build and test commands. I
  can only document commands I actually ran. The wizard adds files I
  did not pick, like a `Properties/` folder and `AssemblyInfo.cs`. "The
  wizard added it" is not a good answer in a review.
- **Cost:** slightly more typing on Day 1. Nothing else.
- **Reference:** [`dotnet new` command](https://learn.microsoft.com/en-us/dotnet/core/tools/dotnet-new)

---
