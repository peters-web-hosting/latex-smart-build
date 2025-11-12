# smart build for latex
A GitHub Action that intelligently compiles LaTeX documents with automatic dependency tracking. Only rebuilds documents affected by changes, saving time in complex multi-document projects.

## Features
- 🎯 **Smart Dependency Tracking** – Automatically detects which main documents need rebuilding when included files change
- 📁 **Deep Nesting Support** – Handles `\include{}` and `\input{}` commands at any directory depth
- ⚡ **Efficient Builds** – Only compiles documents affected by changes, not the entire project
- 📚 **Multiple Document Support** – Works with repositories containing multiple LaTeX documents
- 🗂️ **Automatic Versioning** – Keeps timestamped PDFs with configurable retention
- 📖 **Bibliography Support** – Optional Biber processing for citations
- 📝 **Optional Word Count** – Automatically updates `\wordcount{}` in specified top-level TeX files
- 🚫 **Selective Exclusion** – Exclude specific documents from being recompiled (e.g., archived or finalized documents)
- 🛠️ **Flexible Configuration** – Choose your compiler and customize settings

## Usage

### Breaking Changes in v2.0.0
**Removed `wordcount` input**  
The `wordcount` boolean input has been removed. Word count functionality is now controlled solely by the `wordcount-files` input:
- If `wordcount-files` contains a list of files, word counts will be updated
- If `wordcount-files` is empty or omitted, no word counts will be updated

#### Migration Guide
```yaml
# Old (v1.x)
wordcount: true
wordcount-files: 'thesis.tex,paper.tex'

# New (v2.x) 
wordcount-files: 'thesis.tex,paper.tex'
```

### Basic Usage
```yaml
name: Build LaTeX Documents
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: PeterTSmith1997/smart-build-for-latex@2.0.0
```

### Advanced Configuration
```yaml
name: Build LaTeX Documents
on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: PeterTSmith1997/smart-build-for-latex@2.0.0
        with:
          compiler: 'pdflatex'
          max-drafts: '5'
          output-dir: 'pdf-builds'
          biber: 'true'
          commit-message: 'Build PDFs'
          wordcount-files: 'main.tex,chapter1.tex'
          exclude-files: 'archived/thesis_v1.0,old_submissions'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `compiler` | LaTeX compiler to use (`xelatex`, `pdflatex`, `lualatex`) | No | `xelatex` |
| `max-drafts` | Maximum number of draft PDFs to keep per document | No | `3` |
| `output-dir` | Directory to store compiled PDFs | No | `drafts` |
| `biber` | Run Biber for bibliography processing | No | `true` |
| `commit-message` | Commit message for PDF updates | No | `Update PDFs` |
| `wordcount-files` | Comma-separated list of top-level TeX files to update wordcount for | No | `''` |
| `exclude-files` | Comma-separated list of TeX files to never recompile (e.g., archived documents, stable releases) | No | `''` |

## How It Works

1. **Change Detection** – Detects changed `.tex` files in the commit
2. **Dependency Analysis** – Determines if the changed file is a main document or included file, and finds affected main documents
3. **Exclusion Check** – Filters out any documents specified in `exclude-files` from the build set
4. **Smart Compilation** – Only rebuilds affected documents that aren't excluded
5. **Word Count** – If enabled, updates `\wordcount{}` only in specified top-level files (excluding those in the exclude list)
6. **Version Management** – PDFs are timestamped and old drafts cleaned up
7. **Auto-commit** – Generated PDFs and updated TeX files are pushed back

## Supported Project Structures

```
repository/
├── main.tex
├── chapters/
│   ├── chapter1.tex
│   └── chapter2.tex
├── thesis/
│   ├── thesis.tex
│   └── sections/methodology/data.tex
├── archived/
│   ├── thesis_v1.0.tex
│   └── old_paper_2023.tex
└── shared/
    └── preamble.tex
```

Supports deeply nested `\include{}` or `\input{}` paths.

## Examples

### Dissertation/Thesis Repository
```yaml
uses: PeterTSmith1997/smart-build-for-latex@2.0.0
with:
  compiler: 'xelatex'
  max-drafts: '10'
  output-dir: 'thesis-builds'
  wordcount-files: 'thesis/thesis.tex,thesis/chapters/ch1.tex'
```

### Multiple Papers Repository
```yaml
uses: PeterTSmith1997/smart-build-for-latex@2.0.0
with:
  compiler: 'pdflatex'
  max-drafts: '3'
  output-dir: 'papers'
  commit-message: 'Compile papers'
  wordcount-files: 'paper1.tex,paper2.tex'
```

### Book/Manual with Complex Structure
```yaml
uses: PeterTSmith1997/smart-build-for-latex@2.0.0
with:
  compiler: 'lualatex'
  biber: 'true'
  max-drafts: '5'
```

### Repository with Archived Documents
```yaml
uses: PeterTSmith1997/smart-build-for-latex@2.0.0
with:
  compiler: 'xelatex'
  exclude-files: 'submissions/final_2023, archived/thesis_v1.0, old_drafts/chapter_deprecated'
  wordcount-files: 'current/main.tex'
```

In this example:
- `submissions/final_2023.tex` won't be recompiled even if its dependencies change
- `archived/thesis_v1.0.tex` is preserved as-is without rebuilding
- `old_drafts/chapter_deprecated.tex` is excluded from the build process
- Only `current/main.tex` will have its word count updated

## Performance

The action reduces build times by only compiling affected documents:

| Scenario | Traditional Approach | Smart Build |
|----------|---------------------|-------------|
| Edit shared preamble (`shared/preamble.tex`) | Rebuilds all documents | Rebuilds only documents that include the preamble (excluding those in `exclude-files`) |
| Edit one chapter (`chapters/chapter2.tex`) | Rebuilds entire main document | Only rebuilds main document(s) that include that chapter |
| Add new independent paper | Rebuilds all papers | Builds only the new paper |
| Modify archived document | Would rebuild document | Skips if listed in `exclude-files` |

## Requirements

- Main documents identified with `\documentclass`
- Standard `\include{}` or `\input{}` usage
- Files in a Git repository with `.tex` extensions

## Troubleshooting

- **PDFs not generating** – Check `\documentclass` and paths
- **Wrong documents compiled** – Verify `\include{}`/`\input{}` paths
- **Old PDFs not cleaned** – Only applies to `{basename}_{timestamp}.pdf`
- **Document still being compiled** – Check it's correctly listed in `exclude-files` (can use with or without `.tex` extension)
- **Excluded file not being skipped** – Verify the path matches (relative paths from repository root work best)

## Contributing

1. Fork the repo
2. Create branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add feature'`)
4. Push and create PR

## License

MIT – see [LICENSE](LICENSE)

## Acknowledgments

- Uses [tj-actions/changed-files](https://github.com/tj-actions/changed-files)
- Inspired by academic LaTeX projects

## Support

- 🐛 [Report bugs](https://github.com/PeterTSmith1997/smart-build-for-latex/issues)
- 💡 [Request features](https://github.com/PeterTSmith1997/smart-build-for-latex/issues)
- 📖 [Documentation](https://github.com/PeterTSmith1997/smart-build-for-latex)# smart build for latex
A GitHub Action that intelligently compiles LaTeX documents with automatic dependency tracking. Only rebuilds documents affected by changes, saving time in complex multi-document projects.

## Features
- 🎯 **Smart Dependency Tracking** – Automatically detects which main documents need rebuilding when included files change
- 📁 **Deep Nesting Support** – Handles `\include{}` and `\input{}` commands at any directory depth
- ⚡ **Efficient Builds** – Only compiles documents affected by changes, not the entire project
- 📚 **Multiple Document Support** – Works with repositories containing multiple LaTeX documents
- 🗂️ **Automatic Versioning** – Keeps timestamped PDFs with configurable retention
- 📖 **Bibliography Support** – Optional Biber processing for citations
- 📝 **Optional Word Count** – Automatically updates `\wordcount{}` in specified top-level TeX files
- 🚫 **Selective Exclusion** – Exclude specific documents from being recompiled (e.g., archived or finalized documents)
- 🛠️ **Flexible Configuration** – Choose your compiler and customize settings

## Usage

### Breaking Changes in v2.0.0
**Removed `wordcount` input**  
The `wordcount` boolean input has been removed. Word count functionality is now controlled solely by the `wordcount-files` input:
- If `wordcount-files` contains a list of files, word counts will be updated
- If `wordcount-files` is empty or omitted, no word counts will be updated

#### Migration Guide
```yaml
# Old (v1.x)
wordcount: true
wordcount-files: 'thesis.tex,paper.tex'

# New (v2.x) 
wordcount-files: 'thesis.tex,paper.tex'
```

### Basic Usage
```yaml
name: Build LaTeX Documents
on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: PeterTSmith1997/smart-build-for-latex@2.0.0
```

### Advanced Configuration
```yaml
name: Build LaTeX Documents
on:
  push:
    branches: [main]

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: PeterTSmith1997/smart-build-for-latex@2.0.0
        with:
          compiler: 'pdflatex'
          max-drafts: '5'
          output-dir: 'pdf-builds'
          biber: 'true'
          commit-message: 'Build PDFs'
          wordcount-files: 'main.tex,chapter1.tex'
          exclude-files: 'archived/thesis_v1.0,old_submissions'
```

## Inputs

| Input | Description | Required | Default |
|-------|-------------|----------|---------|
| `compiler` | LaTeX compiler to use (`xelatex`, `pdflatex`, `lualatex`) | No | `xelatex` |
| `max-drafts` | Maximum number of draft PDFs to keep per document | No | `3` |
| `output-dir` | Directory to store compiled PDFs | No | `drafts` |
| `biber` | Run Biber for bibliography processing | No | `true` |
| `commit-message` | Commit message for PDF updates | No | `Update PDFs` |
| `wordcount-files` | Comma-separated list of top-level TeX files to update wordcount for | No | `''` |
| `exclude-files` | Comma-separated list of TeX files to never recompile (e.g., archived documents, stable releases) | No | `''` |

## How It Works

1. **Change Detection** – Detects changed `.tex` files in the commit
2. **Dependency Analysis** – Determines if the changed file is a main document or included file, and finds affected main documents
3. **Exclusion Check** – Filters out any documents specified in `exclude-files` from the build set
4. **Smart Compilation** – Only rebuilds affected documents that aren't excluded
5. **Word Count** – If enabled, updates `\wordcount{}` only in specified top-level files (excluding those in the exclude list)
6. **Version Management** – PDFs are timestamped and old drafts cleaned up
7. **Auto-commit** – Generated PDFs and updated TeX files are pushed back

## Supported Project Structures

```
repository/
├── main.tex
├── chapters/
│   ├── chapter1.tex
│   └── chapter2.tex
├── thesis/
│   ├── thesis.tex
│   └── sections/methodology/data.tex
├── archived/
│   ├── thesis_v1.0.tex
│   └── old_paper_2023.tex
└── shared/
    └── preamble.tex
```

Supports deeply nested `\include{}` or `\input{}` paths.

## Examples

### Dissertation/Thesis Repository
```yaml
uses: PeterTSmith1997/smart-build-for-latex@2.0.0
with:
  compiler: 'xelatex'
  max-drafts: '10'
  output-dir: 'thesis-builds'
  wordcount-files: 'thesis/thesis.tex,thesis/chapters/ch1.tex'
```

### Multiple Papers Repository
```yaml
uses: PeterTSmith1997/smart-build-for-latex@2.0.0
with:
  compiler: 'pdflatex'
  max-drafts: '3'
  output-dir: 'papers'
  commit-message: 'Compile papers'
  wordcount-files: 'paper1.tex,paper2.tex'
```

### Book/Manual with Complex Structure
```yaml
uses: PeterTSmith1997/smart-build-for-latex@2.0.0
with:
  compiler: 'lualatex'
  biber: 'true'
  max-drafts: '5'
```

### Repository with Archived Documents
```yaml
uses: PeterTSmith1997/smart-build-for-latex@2.0.0
with:
  compiler: 'xelatex'
  exclude-files: 'submissions/final_2023, archived/thesis_v1.0, old_drafts/chapter_deprecated'
  wordcount-files: 'current/main.tex'
```

In this example:
- `submissions/final_2023.tex` won't be recompiled even if its dependencies change
- `archived/thesis_v1.0.tex` is preserved as-is without rebuilding
- `old_drafts/chapter_deprecated.tex` is excluded from the build process
- Only `current/main.tex` will have its word count updated

## Performance

The action reduces build times by only compiling affected documents:

| Scenario | Traditional Approach | Smart Build |
|----------|---------------------|-------------|
| Edit shared preamble (`shared/preamble.tex`) | Rebuilds all documents | Rebuilds only documents that include the preamble (excluding those in `exclude-files`) |
| Edit one chapter (`chapters/chapter2.tex`) | Rebuilds entire main document | Only rebuilds main document(s) that include that chapter |
| Add new independent paper | Rebuilds all papers | Builds only the new paper |
| Modify archived document | Would rebuild document | Skips if listed in `exclude-files` |

## Requirements

- Main documents identified with `\documentclass`
- Standard `\include{}` or `\input{}` usage
- Files in a Git repository with `.tex` extensions

## Troubleshooting

- **PDFs not generating** – Check `\documentclass` and paths
- **Wrong documents compiled** – Verify `\include{}`/`\input{}` paths
- **Old PDFs not cleaned** – Only applies to `{basename}_{timestamp}.pdf`
- **Document still being compiled** – Check it's correctly listed in `exclude-files` (can use with or without `.tex` extension)
- **Excluded file not being skipped** – Verify the path matches (relative paths from repository root work best)

## Contributing

1. Fork the repo
2. Create branch (`git checkout -b feature/AmazingFeature`)
3. Commit changes (`git commit -m 'Add feature'`)
4. Push and create PR

## License

MIT – see [LICENSE](LICENSE)

## Acknowledgments

- Uses [tj-actions/changed-files](https://github.com/tj-actions/changed-files)
- Inspired by academic LaTeX projects

## Support

- 🐛 [Report bugs](https://github.com/PeterTSmith1997/smart-build-for-latex/issues)
- 💡 [Request features](https://github.com/PeterTSmith1997/smart-build-for-latex/issues)
- 📖 [Documentation](https://github.com/PeterTSmith1997/smart-build-for-latex)
