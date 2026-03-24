# DevRag

**Free Local RAG for Claude Code - Save Tokens & Time**

DevRag is a lightweight RAG (Retrieval-Augmented Generation) system designed specifically for developers using Claude Code. Stop wasting tokens by reading entire documents - let vector search find exactly what you need.

## Why DevRag?

When using Claude Code, reading documents with the Read tool consumes massive amounts of tokens:

- ❌ **Wasting Context**: Reading entire docs every time (3,000+ tokens per file)
- ❌ **Poor Searchability**: Claude doesn't know which file contains what
- ❌ **Repetitive**: Same documents read multiple times across sessions

**With DevRag:**

- ✅ **40x Less Tokens**: Vector search retrieves only relevant chunks (~200 tokens)
- ✅ **15x Faster**: Search in 100ms vs 30 seconds of reading
- ✅ **Auto-Discovery**: Claude Code finds documents without knowing file names

## Features

- 🤖 **Simple RAG** - Retrieval-Augmented Generation for Claude Code
- 📝 **Markdown Support** - Auto-indexes .md files
- 🔍 **Semantic Search** - Natural language queries like "JWT authentication method"
- 🚀 **Single Binary** - No Python, models auto-download on first run
- 🖥️ **Cross-Platform** - macOS / Linux / Windows
- ⚡ **Fast** - Auto GPU/CPU detection, incremental sync
- 🌐 **Multilingual** - Supports 100+ languages including Japanese & English

## Quick Start

### 1. Install

**From source (requires Go 1.23+ and CGO):**
```bash
git clone https://github.com/0xQRx/devrag.git
cd devrag
CGO_ENABLED=1 go build -o ~/go/bin/devrag ./cmd/main.go
```

**Or download a pre-built binary from [Releases](https://github.com/0xQRx/devrag/releases):**

| Platform | File |
|----------|------|
| macOS (Apple Silicon) | `devrag-macos-apple-silicon.tar.gz` |
| macOS (Intel) | `devrag-macos-intel.tar.gz` |
| Linux (x64) | `devrag-linux-x64.tar.gz` |
| Linux (ARM64) | `devrag-linux-arm64.tar.gz` |
| Windows (x64) | `devrag-windows-x64.zip` |

**macOS/Linux:**
```bash
tar -xzf devrag-*.tar.gz
chmod +x devrag-*
sudo mv devrag-* /usr/local/bin/devrag
```

**Windows:**
- Extract the zip file
- Place in your preferred location (e.g., `C:\Program Files\devrag\`)

### 2. Configure Claude Code

Add to `~/.claude.json` or `.mcp.json`:

```json
{
  "mcpServers": {
    "devrag": {
      "type": "stdio",
      "command": "/usr/local/bin/devrag"
    }
  }
}
```

**Using a custom config file:**
```json
{
  "mcpServers": {
    "devrag": {
      "type": "stdio",
      "command": "/usr/local/bin/devrag",
      "args": ["--config", "/path/to/custom-config.json"]
    }
  }
}
```

### 3. Add Your Documents

```bash
mkdir documents
cp your-notes.md documents/
```

That's it! Documents are automatically indexed on startup.

### 4. Search with Claude Code

In Claude Code:
```
"Search for JWT authentication methods"
```

## Configuration

Create `config.json`:

```json
{
  "document_patterns": [
    "./documents",
    "./notes/**/*.md",
    "./projects/backend/**/*.md"
  ],
  "db_path": "./vectors.db",
  "chunk_size": 500,
  "search_top_k": 5,
  "compute": {
    "device": "auto",
    "fallback_to_cpu": true
  },
  "model": {
    "name": "multilingual-e5-small",
    "dimensions": 384
  }
}
```

### Configuration Options

- `document_patterns`: Array of document paths and glob patterns
  - Supports directory paths: `"./documents"`
  - Supports glob patterns: `"./docs/**/*.md"` (recursive)
  - Multiple patterns: Index files from different locations
  - **Note**: Old `documents_dir` field is still supported (automatically migrated)
- `db_path`: Vector database file path
- `chunk_size`: Document chunk size in characters
- `search_top_k`: Number of search results to return
- `compute.device`: Compute device (`auto`, `cpu`, `gpu`)
- `compute.fallback_to_cpu`: Fallback to CPU if GPU unavailable
- `model.name`: Embedding model name
- `model.dimensions`: Vector dimensions

### Command-Line Options

- `--config <path>`: Specify a custom configuration file path (default: `config.json`)

**Example:**
```bash
devrag --config /path/to/custom-config.json
```

This is useful for:
- Running multiple instances with different configurations
- Testing different models or chunk sizes
- Maintaining separate dev/test/prod configurations

### Pattern Examples

```json
{
  "document_patterns": [
    "./documents",                    // All .md files in documents/
    "./notes/**/*.md",                // Recursive search in notes/
    "./projects/*/docs/*.md",         // docs/ in each project
    "/path/to/external/docs"          // Absolute path
  ]
}
```

## MCP Tools

DevRag provides the following tools via Model Context Protocol:

### search
Perform semantic vector search with optional filtering

**Parameters:**
- `query` (string, required): Search query in natural language
- `top_k` (number, optional): Maximum number of results (default: 5)
- `directory` (string, optional): Filter to specific directory (e.g., "docs/api")
- `file_pattern` (string, optional): Glob pattern for filename (e.g., "api-*.md", "*.md")

**Returns:**
Array of search results with filename, chunk content, and similarity score

**Examples:**
```
// Basic search
search(query: "JWT authentication")

// Search only in docs/api directory
search(query: "user endpoints", directory: "docs/api")

// Search only files matching pattern
search(query: "deployment", file_pattern: "guide-*.md")

// Combined filters
search(query: "authentication", directory: "docs/api", file_pattern: "auth*.md")
```

### index_markdown
Index a markdown file

**Parameters:**
- `filepath` (string): Path to the file to index

### list_documents
List all indexed documents

**Returns:**
Document list with filenames and timestamps

### delete_document
Remove a document from the index

**Parameters:**
- `filepath` (string): Path to the file to delete

### reindex_document
Re-index a document

**Parameters:**
- `filepath` (string): Path to the file to re-index

## Multiple Instances (Per-Project Databases)

You can run separate DevRag instances for different Claude Code projects, each with its own database and document set. Since DevRag communicates over stdio, each Claude Code instance spawns its own isolated process — no conflicts.

**Step 1: Create a config per project**

`project-a/config.json`:
```json
{
  "document_patterns": ["./docs"],
  "db_path": "./vectors.db"
}
```

`project-b/config.json`:
```json
{
  "document_patterns": ["./docs"],
  "db_path": "./vectors.db"
}
```

**Step 2: Point each Claude Code instance to its config**

In each project's `.mcp.json`:

```json
{
  "mcpServers": {
    "devrag": {
      "type": "stdio",
      "command": "/usr/local/bin/devrag",
      "args": ["--config", "/path/to/project-a/config.json"]
    }
  }
}
```

Each instance gets its own database, document index, and search scope — fully isolated.

## Team Development

Perfect for teams with large documentation repositories:

1. **Manage docs in Git**: Normal Git workflow
2. **Each developer runs DevRag**: Local setup on each machine
3. **Search via Claude Code**: Everyone can search all docs
4. **Auto-sync**: `git pull` automatically updates the index

Configure for your project's docs directory:

```json
{
  "document_patterns": [
    "./docs",
    "./api-docs/**/*.md",
    "./wiki/**/*.md"
  ],
  "db_path": "./.devrag/vectors.db"
}
```

## Performance

Environment: MacBook Pro M2, 100 files (1MB total)

| Operation | Time | Tokens |
|-----------|------|--------|
| Startup | 2.3s | - |
| Indexing | 8.5s | - |
| Search (1 query) | 95ms | ~300 |
| Traditional Read | 25s | ~12,000 |

**260x faster search, 40x fewer tokens**

## Development

### Run Tests

```bash
# All tests
go test ./...

# Specific packages
go test ./internal/config -v
go test ./internal/indexer -v
go test ./internal/embedder -v
go test ./internal/vectordb -v

# Integration tests
go test . -v -run TestEndToEnd
```

### Build

```bash
# Using build script
./build.sh

# Direct build
go build -o devrag cmd/main.go

# Cross-platform release build
./scripts/build-release.sh
```

### Creating a Release

```bash
# Create version tag
git tag v1.0.1

# Push tag
git push origin v1.0.1
```

GitHub Actions automatically:
1. Builds for all platforms
2. Creates GitHub Release
3. Uploads binaries
4. Generates checksums

## Project Structure

```
devrag/
├── cmd/
│   └── main.go              # Entry point
├── internal/
│   ├── config/              # Configuration
│   ├── embedder/            # Vector embeddings
│   ├── indexer/             # Indexing logic
│   ├── mcp/                 # MCP server
│   └── vectordb/            # Vector database
├── models/                  # ONNX models
├── build.sh                 # Build script
└── integration_test.go      # Integration tests
```

## Troubleshooting

### Model Download Fails

**Cause**: Internet connection or Hugging Face server issues

**Solutions**:
1. Check internet connection
2. For proxy environments:
   ```bash
   export HTTP_PROXY=http://your-proxy:port
   export HTTPS_PROXY=http://your-proxy:port
   ```
3. Manual download (see `models/DOWNLOAD.md`)
4. Retry (incomplete files are auto-removed)

### GPU Not Detected

Explicitly set CPU in `config.json`:

```json
{
  "compute": {
    "device": "cpu",
    "fallback_to_cpu": true
  }
}
```

### Won't Start

- Ensure Go 1.21+ is installed (for building)
- Check CGO is enabled: `go env CGO_ENABLED`
- Verify dependencies are installed
- Internet required for first run (model download)

### Unexpected Search Results

- Adjust `chunk_size` (default: 500)
- Rebuild index (delete vectors.db and restart)

### High Memory Usage

- GPU mode loads model into VRAM
- Switch to CPU mode for lower memory usage

## Requirements

- Go 1.21+ (for building from source)
- CGO enabled (for sqlite-vec)
- macOS, Linux, or Windows

## License

MIT License

## Credits

- Embedding Model: [intfloat/multilingual-e5-small](https://huggingface.co/intfloat/multilingual-e5-small)
- Vector Database: [sqlite-vec](https://github.com/asg017/sqlite-vec)
- MCP Protocol: [Model Context Protocol](https://modelcontextprotocol.io/)
- ONNX Runtime: [onnxruntime-go](https://github.com/yalue/onnxruntime_go)

## Contributing

Issues and Pull Requests are welcome!

## Contributors

Special thanks to all contributors who helped improve DevRag:

- **[@badri](https://github.com/badri)** - Multiple document paths with glob patterns ([#2](https://github.com/tomohiro-owada/devrag/pull/2)), `--config` CLI flag ([#3](https://github.com/tomohiro-owada/devrag/pull/3))
- **[@io41](https://github.com/io41)** - Project cleanup and documentation improvements ([#4](https://github.com/tomohiro-owada/devrag/pull/4))

Your contributions make DevRag better for everyone!

## Author

[towada](https://github.com/tomohiro-owada)
