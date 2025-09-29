# YEH (YpsilonEventHandler)

> A lightweight, flexible event delegation utility for modern web applications. Simplifies event management by centralizing listeners and providing advanced routing options.

YEH uses browser-native APIs (stable since 2000) for reliable, no-dependency event handling. Works on `file://` with zero build tools.

[![NPM version](https://img.shields.io/npm/v/yeh.svg)](https://npmjs.org/package/yeh)
[![License](https://img.shields.io/npm/l/yeh.svg)](https://github.com/yaijs/yeh/blob/main/LICENSE)
[![Documentation](https://img.shields.io/badge/docs-QuantumType-blueviolet)](https://github.com/yaijs/yeh/blob/main/yeh.d.ts)

---

## ✨ Features

- **Event Delegation Made Easy**: One listener handles dynamic elements with scope-based routing.
- **Automatic Target Resolution**: Handles nested elements (e.g., SVGs in buttons).
- **Throttle & Debounce Support**: Built-in performance controls.
- **Dynamic Listener Management**: Add/remove events at runtime.
- **Flexible Handler Resolution**: Class methods, external maps, or globals.
- **Multi-Handler System**: Closest-match resolution for nested components.
- **Performance Tracking**: Optional metrics for optimization.
- **No Dependencies**: ~5kB gzipped, enterprise-ready (~800 LOC).

For advanced patterns (e.g., reactive state, super delegation), see [README.USAGE.md](README.USAGE.md). For internals, see [README.TECHNICAL.md](README.TECHNICAL.md).

---

## 🚀 Quick Start

**No setup, no build step, no server, just include the file.**

**Get started in 30 seconds** – [try it live on JSFiddle](https://jsfiddle.net/L6zk29v5/)

```html
<!DOCTYPE html>
<html>
<head><title>YEH Demo</title></head>
<body>
  <div id="app">
    <button data-action="save">Save</button>
    <button data-action="delete">Delete</button>
  </div>

  <script src="https://cdn.jsdelivr.net/npm/yeh@latest"></script>
  <script>
    class MyHandler extends YEH {
      constructor() {
        super({ '#app': ['click'] }); // Falls back to handleClick()
      }

      handleClick(event, target) {
        const action = target.dataset.action;
        if (action && this[action]) this[action](target, event);
      }

      save(target) { console.log('Saving...'); }
      delete(target) { console.log('Deleting...'); }
    }

    new MyHandler(); // Adding listeners Done
  </script>
</body>
</html>
```

**30-second setup:** Create `app.html`, copy & paste the above code, then double-click to run.

> **💡 Universal Delegation Pattern**
>
> One listener on parent + `custom-selector` = handles unlimited elements within the parent

---

## 📦 Installation

### CDN (Instant Setup)
```html
<script src="https://cdn.jsdelivr.net/npm/yeh@latest"></script>
```

### npm (Build Tools)
```bash
npm install ypsilon-event-handler
```

### Direct Download
[Download latest version](ypsilon-event-handler.js) - works with file:// protocol

---

### ⚙️ Configuration Options

Pass a third argument to the constructor to enable advanced features:

| Option                 | Type      | Default | Description                                                                |
| ---------------------- | --------- | ------- | ---------------------------------------------------------------------------|
| `enableStats`          | `boolean` | `false` | Track performance metrics like event count and distance cache hits.        |
| `methods`              | `object`  | `null`  | External method map for organizing handlers by event type.                 |
| `enableGlobalFallback` | `boolean` | `false` | Fallback to global `window` functions when no method is found.             |
| `methodsFirst`         | `boolean` | `false` | Check `methods` object before class methods during handler resolution.     |
| `passiveEvents`        | `array`   | auto    | Override default passive events (scroll, touch, wheel, pointer).           |
| `abortController`      | `boolean` | `false` | Enable `AbortController` support for programmatic listener removal.        |
| `enableDistanceCache`  | `boolean` | `true`  | Cache DOM distance calculations for performance (multi-handler scenarios). |

**Example:** `new YEH(events, aliases, { enableStats: true });`

---

## 🔗 Fluent Chaining API

Chain operations for complex event orchestration:

```js
App.on('data-ready', 'handleData')
    .on('user-login', 'handleLogin')
    .emit('init-complete', { loaded: true });
```

---

### 🧹 Cleanup
`handler.destroy();` or `handler.abort();` (if enabled).

---

### 📊 Performance Metrics
With `enableStats: true`: `console.log(handler.getStats());`

---

## 🌐 Browser Support

**Chrome** | **Firefox** | **Safari** | **Edge** - all modern versions

*Works with legacy browsers via Webpack + Babel.*

---

## 📊 Comparison vs Popular Libraries

| Feature                     | YEH                 | EventEmitter3 | Redux Toolkit | jQuery         |
|-----------------------------|---------------------|---------------|---------------|----------------|
| **Bundle Size**             | 5kB gzipped         | 7kB gzipped   | 12kB+ gzipped | 30kB+ gzipped  |
| **Dependencies**            | ✅ Zero             | ✅ Zero       | ❌ Many       | ✅ Zero        |
| **Event Delegation**        | ✅ Advanced         | ❌ None       | ❌ None       | ✅ Basic       |
| **Multi-Handler System**    | ✅ Unique           | ❌ None       | ❌ None       | ❌ None        |
| **Throttle/Debounce**       | ✅ Built-in         | ❌ None       | ❌ None       | ❌ None        |
| **Native Browser API**      | ✅ Yes              | ❌ No         | ❌ No         | ❌ No          |
| **Dynamic Element Support** | ✅ Zero-config      | ❌ None       | ❌ None       | ✅ Re-bind     |
| **TypeScript Support**      | ✅ Full             | ✅ Partial    | ✅ Full       | ⚠️ Community   |
| **Memory Leak Prevention**  | ✅ Automatic        | ⚠️ Manual     | ✅ Automatic  | ⚠️ Manual      |
| **Performance**             | ✅ Native           | ⚠️ Synthetic  | ⚠️ Virtual    | ⚠️ Abstraction |
| **Custom Event Dispatch**   | ✅ Built-in         | ✅ Yes        | ✅ Yes        | ✅ Yes         |
| **Learning Curve**          | ✅ Low              | ✅ Low        | ❌ Steep      | ✅ Familiar    |

### Why YEH Stands Out
- **Smallest footprint** with advanced features like multi-handler delegation.
- **Native performance** using browser APIs, avoiding synthetic event overhead.
- **Zero dependencies** and automatic memory management for scalability.
- **Built-in utilities** (throttle, debounce, stats) eliminate external needs.


## 🚀 **See It In Action**

**[Interactive Examples Hub](https://eypsilon.github.io/YpsilonEventHandler-Examples/example/public/)**
~ Beautiful landing page with all examples organized by category

**[Feature Demonstrations](https://eypsilon.github.io/YpsilonEventHandler-Examples/example/public/features/)**
~ Interactive examples of specific capabilities

---

#### 📂 File Structure

```bash
/
├── yeh.js                        # Main library
├── yeh.d.ts                      # Main TypeScript
├── README.md                     # Quick start and core guide
├── README.USAGE.md               # Advanced patterns and techniques
└── README.TECHNICAL.md           # Implementation details and architecture
```

---

#### License

MIT License – free to use in personal or commercial projects.


##### 👥 **Authors & Contributors**

- **Claude Van DOM** - Implementation and optimization
- **Engin Ypsilon** - Original concept and architecture
- **The Y-Team** - Sunny DeepSeek & Herr Von Grokk
