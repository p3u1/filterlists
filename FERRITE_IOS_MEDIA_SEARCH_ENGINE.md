# Ferrite — iOS Media Search Engine (Plugin-first)

Ferrite is a native iOS media search engine with a modular plugin API for extending search providers, metadata enrichers, parsing logic, and cloud/debrid integrations.

> **Disclaimer**  
> This project is developed for hobbyist/educational purposes. Ferrite does not host content and is not intended to do so. It indexes/searches third-party sources through user-installed plugins.

---

## Product Vision

Finding shows and movies is usually possible, but many existing websites have poor UX and weak filtering. Ferrite aims to provide a polished, native iOS interface with:

- fast, composable search filters
- a clear “search → inspect → stream/download” flow
- user-controlled providers via plugins
- optional debrid cloud integration

---

## Core Features

- Ad-free experience
- Clean UI with native performance
- Powerful search with an intuitive filter system
- Modular plugin system
- Debrid provider integrations
- Flexible parser system written in Swift
- Local library (bookmarks + history)
- Debrid cloud manager
- Optional Kodi integration

---

## iOS Support Policy

Ferrite follows an **n - 2** support model:

- `v0.8+`: iOS 16+
- `v0.7+`: iOS 15+
- `v0.6.x and below`: iOS 14+

---

## Supported Debrid Providers

- RealDebrid
- AllDebrid
- Premiumize
- TorBox
- OffCloud

---

## Architecture

### 1) App Layers

- **UI Layer (SwiftUI/UIKit):** search, filters, details, bookmarks, history, cloud
- **Domain Layer:** use-cases (query orchestration, plugin dispatch, ranking)
- **Data Layer:** provider clients, plugin runtime, cache, persistence
- **Integration Layer:** debrid APIs, optional external player/Kodi endpoints

### 2) Data Flow

1. User enters a query and optional filters.
2. Search coordinator fans out requests to enabled plugins.
3. Plugins normalize provider responses into shared models.
4. Ranking engine merges/sorts results.
5. User opens result details and can trigger cloud/debrid actions.

---

## Plugin API (Swift)

Ferrite’s extension point is intentionally narrow and typed.

### Plugin Manifest

Each plugin declares metadata and capabilities.

```json
{
  "id": "com.example.source.myprovider",
  "name": "MyProvider",
  "version": "1.0.0",
  "apiVersion": "1",
  "capabilities": ["search", "resolve", "metadata"],
  "minFerriteVersion": "0.8.0"
}
```

### Swift Protocols

```swift
import Foundation

public struct SearchRequest: Sendable {
    public let query: String
    public let page: Int
    public let filters: [String: String]
}

public struct MediaResult: Codable, Sendable, Identifiable {
    public let id: String
    public let title: String
    public let year: Int?
    public let type: String
    public let source: String
    public let score: Double?
}

public struct ResolveRequest: Sendable {
    public let resultID: String
}

public struct StreamLink: Codable, Sendable {
    public let url: URL
    public let quality: String?
    public let sizeBytes: Int64?
    public let codec: String?
}

public protocol FerritePlugin: Sendable {
    var id: String { get }
    var name: String { get }
    var version: String { get }

    func search(_ request: SearchRequest) async throws -> [MediaResult]
    func resolve(_ request: ResolveRequest) async throws -> [StreamLink]
}
```

### Host Contracts

- Plugins run in a sandboxed execution context.
- All plugin network access is brokered by the host.
- Plugin settings are namespaced by plugin ID.
- Versioned API contracts enforce compatibility.
- The host can disable plugins that violate policy/timeouts.

---

## Security Model

- Signed plugin packages (recommended)
- Manifest validation + capability gating
- Strict timeout, rate-limit, and retry budgets
- Audit logging for plugin actions
- Optional “safe mode” with only trusted first-party plugins

---

## Storage

- **SQLite/CoreData/SwiftData** for local metadata and history
- **Keychain** for sensitive tokens
- **On-disk cache** for artwork and transient provider responses

---

## Distribution

Current approach:

- Distribute as `.ipa` artifact through CI builds
- Not planning App Store/TestFlight release in this model

---

## Suggested Repository Layout

```text
Ferrite/
  App/
  Features/
    Search/
    Details/
    Library/
    Cloud/
    Plugins/
  Core/
    PluginAPI/
    Networking/
    Persistence/
    Logging/
  Integrations/
    Debrid/
    Kodi/
  Plugins/
    Builtin/
```

---

## Developer + Community

- Creator/Developer: kingbri
- Website: kingbri.dev
- Discord: kingbri#6666
- Support Discord: Ferrite Discord
- Reference repository: https://github.com/p3u1/ferritestreams

---

## Next Milestones

1. Stabilize Plugin API v1 and publish docs/examples.
2. Add plugin test harness + conformance suite.
3. Implement plugin marketplace feed (self-hostable JSON index).
4. Add advanced filters (HDR, codec, release group, language).
5. Improve ranking with per-user signals (local-only).
