<div align="center">
  <img src="docs/banner.png" alt="MCP-Markdown-RAG" width="800" style="border-radius:10px;"/>
  <h1>MCP-Markdown-RAG</h1>
  <p><em>Fork version with enhanced features</em></p>
  <p>
  <img alt="GitHub forks" src="https://img.shields.io/github/forks/Zackriya-Solutions/MCP-Markdown-RAG"/>
  <img alt="GitHub Repo stars" src="https://img.shields.io/github/stars/Zackriya-Solutions/MCP-Markdown-RAG">
  <img alt="GitHub last commit" src="https://img.shields.io/github/last-commit/Zackriya-Solutions/MCP-Markdown-RAG">
</p>
<p>
  <a href="LICENSE">
    <img src="https://img.shields.io/badge/License-Apache%202.0-green" alt="License" />
  </a>
  <img src="https://img.shields.io/badge/MCP-Server-blue"/>
</p>
</div>

A **Model Context Protocol (MCP)** server that provides a **local-first RAG engine** for your markdown documents. This server uses a file-based Milvus vector database to index your notes, enabling Large Language Models (LLMs) to perform semantic search and retrieve relevant content from your local files.

> [!NOTE]
> This is a fork version with additional features. The original project is [MCP-Markdown-RAG](https://github.com/Zackriya-Solutions/MCP-Markdown-RAG) by Zackriya Solutions.

## 🔄 Fork 版本改动

本 fork 版本在原始项目基础上增加了以下功能：

### 新增功能

1. **环境变量配置支持**
   - `MCP_RAG_STORAGE_PATH`: 自定义数据库存储路径（默认: `./.db`）
   - `MCP_RAG_DOCS_PATH`: 设置默认文档索引目录

2. **扩展文件格式支持**
   - 支持 `.mdx` 文件格式（除了原有的 `.md` 文件）
   - 适合 React/Next.js 项目中的 MDX 文档

3. **改进的递归索引**
   - 优化了递归索引的实现方式，提高性能

### 配置示例

你可以在 MCP 配置文件中通过环境变量自定义路径：

```json
{
  "mcpServers": {
    "markdown_rag": {
      "command": "uv",
      "args": [
        "--directory",
        "/ABSOLUTE/PATH/TO/MCP-Markdown-RAG",
        "run",
        "server.py"
      ],
      "env": {
        "MCP_RAG_STORAGE_PATH": "/path/to/your/database",
        "MCP_RAG_DOCS_PATH": "/path/to/your/docs"
      }
    }
  }
}
```

**配置说明：**
- `MCP_RAG_STORAGE_PATH`: 数据库和索引跟踪文件的存储位置
- `MCP_RAG_DOCS_PATH`: 默认文档目录（如果调用 `index_documents` 时不指定 `directory` 参数，将使用此路径）

## 🎯 Key Features

✅ **Local-First & Private**: All your data is processed and stored locally. Nothing is sent to a third-party service for indexing.

✅ **Semantic Search for Markdown**: Go beyond simple keyword search. Find document sections based on conceptual meaning.

✅ **MCP Compatible**: Integrates with any MCP-supported host application like Claude Desktop, Windsurf, or Cursor.

✅ **Simple Tooling**: Provides two straightforward tools (`index_documents` and `search`) for managing and querying your knowledge base.

## ⚙️ How It Works

The server operates in two main phases, exposing its functionality through MCP tools.

1.  **Indexing**:

    - The `index_documents` tool is called with a path to your markdown files.
    - The server reads the documents, splits them into logical chunks (e.g., by headings), and converts each chunk into a vector embedding.
    - These embeddings, along with their metadata (original text, file path), are stored in a local Milvus vector database.
    - You can run it in two modes:
      - **Full Reindex** (force_reindex=True): Clears and rebuilds the entire index from scratch.
      - **Incremental Update** (force_reindex=False, default): Automatically detects and re-indexes only changed files by comparing them against a tracking log. Deleted or modified chunks are pruned and replaced to keep the index up-to-date.
      - **Recursive Indexing** (recursive=False, default): Recursively indexes all subdirectories.
      - **File Format Support**: Supports both `.md` and `.mdx` file formats.

2.  **Searching**:

    - When you ask a question in a host application, it uses the `search` tool.
    - The server converts your query into a vector embedding.
    - It then performs a similarity search against the Milvus database to find the most semantically relevant document chunks.
    - The results are returned to the LLM, providing it with the context needed to answer your question accurately.

    <div align="center" >
    <img src="docs/mcp_search.png" alt="MCP Search" width="800" style="border-radius:10px;"/>
    </div>

## 🛠️ Available Tools

- `index_documents`

  - **Description**: Indexes Markdown documents for semantic search. Converts each file into structured vector chunks and inserts them into the Milvus database.
  - **Incremental Indexing**: Automatically reindexes only changed files unless force_reindex=True is passed.
  - **Supported File Formats**: `.md` and `.mdx` files
  - **Arguments**:
    - `directory` (string, optional): The path to the folder containing .md or .mdx files. Defaults to `MCP_RAG_DOCS_PATH` environment variable or current directory.
    - `force_reindex` (boolean, optional): If True, clears and rebuilds the full index. Defaults to False.
    - `recursive` (boolean, optional): If True, recursively indexes all subdirectories. Defaults to False.

- `search`
  - **Description**: Searches the indexed documents using semantic similarity.
  - **Arguments**:
    - `query` (string, required): Your natural language query.
    - `limit` (integer, optional): Max number of chunks to return (default is usually 5–10).

## 🚀 Installation & Setup

This server requires **UV** (for running the Python server).

### Step 1: Get the Server Code

Clone this repository to your local machine:

```bash
git clone https://github.com/Zackriya-Solutions/MCP-Markdown-RAG.git
```

### Step 2: Configure Your Host App

Configure your MCP host application (e.g., Cursor, Claude Desktop, Windsurf) to use the server. Add the following to your settings file:

**基础配置：**
```json
{
  "mcpServers": {
    "markdown_rag": {
      "command": "uv",
      "args": [
        "--directory",
        "/ABSOLUTE/PATH/TO/MCP-Markdown-RAG",
        "run",
        "server.py"
      ]
    }
  }
}
```

**推荐配置（使用环境变量）：**
```json
{
  "mcpServers": {
    "markdown_rag": {
      "command": "uv",
      "args": [
        "--directory",
        "/ABSOLUTE/PATH/TO/MCP-Markdown-RAG",
        "run",
        "server.py"
      ],
      "env": {
        "MCP_RAG_STORAGE_PATH": "/path/to/your/database/storage",
        "MCP_RAG_DOCS_PATH": "/path/to/your/documents"
      }
    }
  }
}
```

> **Note**: Replace `/ABSOLUTE/PATH/TO/MCP-Markdown-RAG` with the absolute path to where you cloned this repository.
> 
> **Note**: The first run will take a while and the same for the first indexing, as it needs to download the embedding model(~50MB).
> 
> **注意**: 
> - `MCP_RAG_STORAGE_PATH`: 数据库存储路径（存储 Milvus 数据库和索引跟踪文件）
> - `MCP_RAG_DOCS_PATH`: 默认文档目录（可选，如果不设置，需要在调用工具时指定 `directory` 参数）

### Step 3: 使用示例

**索引所有文档（包括子目录）：**
- 在 Cursor 或支持的 MCP 客户端中，直接说："索引所有文档，包括所有子目录"
- 或者调用工具时设置：`recursive: true`, `force_reindex: true`

**搜索文档：**
- 直接提问："搜索关于 SwiftUI 的内容"
- 系统会自动使用 `search_documents` 工具进行语义搜索

## 📈 What's Next? (Roadmap)

We are actively working on improving the server. Future plans include:

- **Performance Optimization**: Improve indexing by encoding inputs in batches, which should better manage CPU usage.
- **Flexible Embedding Models**: Add support for other embedding models, such as the `BGEM3-large` model for potentially higher accuracy.
- **Obsidian Plugin**: Explore creating a dedicated Obsidian plugin for a fully integrated experience.

## 🐛 Debugging

You can use the MCP inspector to debug the server directly. Run the following command from the repository's root directory:

```bash
npx @modelcontextprotocol/inspector uv --directory /ABSOLUTE/PATH/TO/MCP-Markdown-RAG run server.py
```

## 🤝 Contributing

Contributions are welcome! Please feel free to open an issue or submit a pull request.

## 🙏 Acknowledgments

- The **[Model Context Protocol](https://modelcontextprotocol.io/introduction)** for the open standard that makes this possible.
- The **[Milvus Project](https://milvus.io/)** for the powerful open-source vector database.
