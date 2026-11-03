<div align="center">

# 🔍 Analyse OOP

**An object-oriented system analysis library for C#**

Point it at a codebase. Get back a structured map of the classes, members, and relationships inside it.

![.NET](https://img.shields.io/badge/.NET-6.0%2B-512BD4?logo=dotnet&logoColor=white)
![Roslyn](https://img.shields.io/badge/Powered%20by-Roslyn-blueviolet)
![Language](https://img.shields.io/badge/C%23-library-239120?logo=csharp&logoColor=white)
![License](https://img.shields.io/badge/License-MIT-green)
![Status](https://img.shields.io/badge/Status-In%20development-orange)

</div>

---

## 📖 Overview

**Analyse OOP** is a library for understanding object-oriented systems. It uses the **Roslyn compiler platform** to parse C# source code, extract the details of every class (namespace, visibility, inheritance, methods, fields, properties), work out how those classes relate to one another, and write the result to a single `classes.json` file.

It is built as a foundation for deeper architectural analysis: today it answers *"what is in this codebase and how is it connected?"*, and the roadmap aims at *"what design patterns does it use, and how good is its architecture?"*

**Use it to:**

- 📚 Document a codebase automatically
- 🗺️ Get a bird's-eye view of an unfamiliar project
- 🤖 Feed project structure to other tools or LLMs
- 📈 Generate class and dependency diagrams
- 🧱 Build your own analysis tools on top of structured class data

---

## ✨ Features

- **Recursive scanning**: finds every `*.cs` file in a folder and all its subfolders
- **Parallel processing**: files are read and parsed concurrently with `Parallel.ForEachAsync`
- **Roslyn-powered parsing**: reads the real syntax tree, no fragile regex
- **Rich class details**: name, namespace, access modifier, base class, interfaces, and source file path
- **Member extraction**: methods (with parameters, modifiers and default values), fields, and properties
- **Relationship mapping**: connects each class to the parent and interface classes found in your codebase
- **Simple API**: one engine class drives the whole pipeline in three calls
- **Pretty JSON output**: indented and human-readable

---

## 🚀 Getting Started

### Prerequisites

- [.NET 6 SDK](https://dotnet.microsoft.com/download) or newer (the library uses `Parallel.ForEachAsync`)
- The Roslyn NuGet package:

```bash
dotnet add package Microsoft.CodeAnalysis.CSharp
```

### Quick start

```csharp
using ClassExtractor;

// 1. Scan a folder for C# classes
var engine = new OOPAnalyzerEngine(targetFolderPath);
await engine.RunAsyncReadSourcefiles();

// 2. Build the relationships between the classes
engine.EstablishRelationshipsBetweenClasses();

// 3. Create the JSON file containing the class information
await engine.CreateJSONClassFile();
```

The output is saved as **`classes.json`** in the application's base directory (next to the executable).

### The engine at a glance

| Method | What it does |
|---|---|
| `new OOPAnalyzerEngine(path)` | Points the engine at the folder you want to analyse |
| `RunAsyncReadSourcefiles()` | Scans and parses every `.cs` file, building the class nodes |
| `EstablishRelationshipsBetweenClasses()` | Links each class to its parent and interfaces |
| `CreateJSONClassFile()` | Writes the results to `classes.json` |

---

## 📦 What You Get

For every class, Analyse OOP records:

| Field | Description |
|---|---|
| `Name` | Class name |
| `Namespace` | Containing namespace (`Global` if none) |
| `AccessModifier` | `public`, `internal`, etc. (defaults to `internal`) |
| `ParentClass` | Base class, if any |
| `Interfaces` | Interfaces the class implements |
| `Methods` | Visibility, return type, name, and parameters |
| `Fields` | Visibility, type, and name |
| `Properties` | Visibility, type, and name |
| `FilePath` | Where the class lives on disk |

### Example output

```json
[
  {
    "Name": "OrderService",
    "Namespace": "Shop.Services",
    "AccessModifier": "public",
    "ParentClass": "BaseService",
    "Interfaces": ["IOrderService"],
    "Methods": [
      {
        "Visibility": "public",
        "ReturnType": "Task<Order>",
        "Name": "GetOrderAsync",
        "Parameters": [
          {
            "Name": "id",
            "Type": "int",
            "Modifiers": "",
            "DefaultValue": null
          }
        ]
      }
    ],
    "Fields": [
      { "Visibility": "private", "Type": "ILogger", "Name": "_logger" }
    ],
    "Properties": [
      { "Visibility": "public", "Type": "string", "Name": "ServiceName" }
    ],
    "FilePath": "C:\\Projects\\Shop\\Services\\OrderService.cs"
  }
]
```

---

## 🏗️ Architecture

```
                    ┌──────────────────────┐
                    │  OOPAnalyzerEngine   │   ← your entry point
                    └──────────┬───────────┘
                               │
        ┌──────────────────────┼──────────────────────┐
        ▼                      ▼                      ▼
┌────────────────┐   ┌──────────────────┐   ┌─────────────────────┐
│ SourceFolder   │   │ Relationship     │   │ ClassToJSON         │
│ Extractor      │──►│ Mapper           │──►│ Convertor           │
│ (Roslyn parse) │   │ (parents/ifaces) │   │ (classes.json)      │
└────────────────┘   └──────────────────┘   └─────────────────────┘
```

### Pipeline

```
 .cs files ──► Roslyn syntax tree ──► ClassNode records ──► RelationshipMapper ──► classes.json
```

1. All `.cs` files under the target folder are enumerated.
2. Each file is parsed into a Roslyn `SyntaxTree` in parallel.
3. Every `ClassDeclarationSyntax` is inspected and turned into a `ClassNode`.
4. Nodes are collected in a thread-safe `ConcurrentBag`.
5. Relationships between classes are established.
6. The results are serialized to indented JSON.

---

## 🗂️ Project Structure

| File | Purpose |
|---|---|
| `OOPAnalyzerEngine.cs` | Easy-to-use entry point that drives the whole system |
| `SourceFileExtractor.cs` | `SourceFolderExtractor`: walks the folder, parses files with Roslyn, and builds `ClassNode`s |
| `ClassNode.cs` | The record describing a single class |
| `RecordCollection.cs` | Supporting records: `MemberDetails`, `MethodDetails`, `ParameterDetails` |
| `RelationshipMapper.cs` | Links each class to its parent/interface classes found in the scanned code |
| `JSONConverter.cs` | `ClassToJSONConvertor`: serializes the results to `classes.json` |

---

## 🛣️ Development Roadmap

- [x] Read class details to a JSON file
- [x] Establish relationships between classes
- [ ] Handle complex relations, such as classes held inside data structures
- [ ] Integrate design pattern analysis
- [ ] Deliver architectural analysis based on the detected design patterns

---

## 🤝 Contributing

Ideas, issues, and pull requests are welcome. If you're planning a larger change (like new analysis passes), open an issue first so we can talk it through.

---

## 📄 License

This project is licensed under the MIT license.
