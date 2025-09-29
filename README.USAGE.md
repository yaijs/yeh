# YEH - Advanced Usage Patterns

Advanced patterns and techniques for YEH event delegation.

All examples work with `file://` protocol - no build, no setup, no server needed. **Just open in your browser.**

**🚀 [Interactive Examples Hub](https://eypsilon.github.io/YpsilonEventHandler-Examples/example/public/)** - Live demos of all patterns

- [🌌 Pattern: Quantum-Entangled Modules](#-pattern-quantum-entangled-modules)
- [⚡ Pattern: Reactive State Management](#-pattern-reactive-state-management)
- [🚀 Pattern: Super Delegation](#-pattern-super-delegation)
- [🏗️ Pattern: Self-Contained Module Architecture](#-pattern-self-contained-module-architecture)
- [🎯 Pattern: Inline Method Injection](#-pattern-inline-method-injection)
- [📊 Pattern: Performance Monitoring](#-pattern-performance-monitoring)
- [🎯 Key Patterns Summary](#-key-patterns-summary)
- [🚨 Advanced Patterns](#-advanced-patterns)
- [🔬 Experimental Usage Patterns](#-experimental-usage-patterns)
- [🚨 Pattern: Cross-Tab Event Broadcasting](#-pattern-cross-tab-event-broadcasting)
- [🚨 Pattern: Event-Driven CSS Toggling](#-pattern-event-driven-css-toggling)
- [🚨 Pattern: Self-Modifying Handlers](#-pattern-self-modifying-handlers)
- [🚨 Pattern: DOM-to-WebWorker Event Bridge](#-pattern-dom-to-webworker-event-bridge)

> 💡 **How to Run**: Save any example as an `.html` file and double-click to open in your browser. Works instantly!

## 🌌 **Pattern: Quantum-Entangled Modules**

**Decouple your UI and data management. Communicate via events for true modularity—no direct references, no import spaghetti.**

**Use When**: You want to build modular apps where data and UI are developed independently, or when building micro-frontend architectures.

Independent modules that communicate through event dispatch without coupling.

Event-driven architecture in minutes and zero configuration.

[Live example on JSFiddle](https://jsfiddle.net/65hdmz9v/), just because.

```js
// Module A: User Interface (can be in same file or separate files)
class MyHandler extends YEH {
    constructor() {
        super({
            // Handles clicks within its own UI section
            '#user-panel': ['click'],
            // Subscriber, subscribes to 'data.update'
            'document': [{ type: 'data.update', handler: 'onDataChange' }]
        });
    }

    handleClick(event, target, containerElement) {
        // Broadcast UI actions to any listening modules
        this.dispatch('ui.action', {
            action: target.dataset.action,
            userId: containerElement.dataset.userId
        });
    }

    // Listen for data updates from other modules
    onDataChange(event) {
        console.log('🖼️ UI updating from data change:', event.detail);
    }
}

// Module B: Data Manager (completely independent - no imports needed!)
class MyHandler extends YEH {
    constructor() {
        super({
            // Handles clicks within its own control section
            '#data-controls': ['click'],
            // Subscriber, subscribes to 'ui.action'
            'document': [{ type: 'ui.action', handler: 'onUIAction' }]
        });
    }

    handleClick(event, target, containerElement) {
        // Broadcast data updates to any listening modules
        this.dispatch('data.update', {
            timestamp: Date.now(),
            source: 'data-manager'
        });
    }

    // React to UI actions from other modules
    onUIAction(event) {
        console.log('📊 Data responding to UI action:', event.detail);
    }
}

// Zero coupling, infinite scalability! Each module operates independently
const ui = new UserInterface();
const data = new DataManager();
```

## ⚡ **Pattern: Reactive State Management**

**Build React/Vue-style reactivity with pure vanilla JavaScript. State changes instantly update the entire UI.**

**Use When**: You need reactive interfaces without framework overhead, building configuration panels, or creating dynamic dashboards.

[Live example on JSFiddle](https://jsfiddle.net/4zqem37g/), where I copy&pasted from below to there, again.

```html
<label>
    Font size
    <input
        value="16"
        data-key="fontSize"
        data-action="setConfig"
        type="number" min="10" max="40" step="1">
</label>

<label>
    Distance top
    <input
        value="10"
        data-key="marginTop"
        data-action="setConfig"
        type="number" min="5" max="100" step="1">
</label>

<pre id="show-config">Loading configuration</pre>

fontSize: <span data-config-key="fontSize">Loading value of fontSize</span>

<script src="https://cdn.jsdelivr.net/npm/@yaijs/yeh@latest/yeh.js"></script>
<script>
// Our reactive state - just a plain object
const config = {
    ui: {
        fontSize: 0,  // Will become 16 after init
        marginTop: 0, // Will become 10 after init
    }
};

class MyHandler extends YEH {
    constructor() {
        // MAGIC #1: Super delegation! Single listener handles all config inputs
        super({
            // One body listener instead of listeners on every input
            // New config form elements work automatically when added via JS
            body: [
                { type: 'change', handler: 'handleChange' }
            ]
        });

        // Cache DOM queries to avoid repeated querySelectorAll calls
        this.cache = new Map();

        this._parseConfigs();
        this._updateUiConfig();
    }

    // MAGIC #2: Super router - configurable via data-action attributes
    handleChange(event, target) {
        const action = target.dataset.action;
        if (action && this[action]) {
            this[action](target, event);
        }
    }

    // Reactive config setter - the heart of reactivity
    setConfig(target, event) {
        const key = target.dataset.key;
        config.ui[key] = this._getConfigValue(key, target);

        // DRY principled UI updater - this is where the magic happens
        this._updateUiConfig();
    }

    // Flexible value resolver - customize per config type
    _getConfigValue(key, target) {
        switch(key) {
            case 'fontSize':
            case 'marginTop': return parseInt(target.value, 10);
        }
        return null; // Handle validation/fallbacks as needed
    }

    // Initialize reactivity - existing input values become defaults
    _parseConfigs() {
        let setConfigElements = this.getCache('setConfigElements');
        if (!setConfigElements) {
            setConfigElements = document.querySelectorAll('[data-action="setConfig"]');
            this.setCache('setConfigElements', setConfigElements);
        }

        setConfigElements.forEach(element => {
            const key = element.dataset.key;
            config.ui[key] = this._getConfigValue(key, element);
        });
    }

    // The reactive core - updates happen everywhere instantly
    _updateUiConfig() {
        // Update the main config display
        let showConfig = this.getCache('showConfig');
        if (!showConfig) {
            showConfig = document.getElementById('show-config');
            this.setCache('showConfig', showConfig);
        }
        showConfig.textContent = JSON.stringify(config, null, 2);

        // Update individual elements scattered throughout the page
        let individualElements = this.getCache('individualElements');
        if (!individualElements) {
            individualElements = document.querySelectorAll('[data-config-key]');
            this.setCache('individualElements', individualElements);
        }

        individualElements.forEach(element => {
            const configKey = element.dataset.configKey;
            element.textContent = config.ui[configKey];
        });
    }

    // Simple cache management
    getCache(key) { return this.cache.get(key); }
    setCache(key, value) { this.cache.set(key, value); }
}

// Initialize and enjoy React-style reactivity in vanilla JS!
const configHandler = new ConfigHandler();
// configHandler.destroy() // Clean removal when needed
</script>
```

**React/Vue developers will be amazed - full reactivity with zero framework overhead!** ⚡


## 🚀 **Pattern: Super Delegation**

**Maximum efficiency: One listener rules them all. Perfect for large applications with dynamic content.**

**Use When**: Building large SPAs with hundreds/thousands of dynamic elements, or when you want ultimate performance with minimal memory footprint.

One listener handles infinite elements with intelligent routing.

**Something's wrong, HEY?**
"What?"
**Where is my example to copy&paste?**
"Challenge, you write your Example!"
**Say what?**

```js
class MyHandler extends YEH {
    constructor() {
        super({
            // One listener for the entire application
            body: [
                'click',
                { type: 'input',  debounce: 300 },
                { type: 'scroll', throttle: 100 }
                // 'resize', 'touchstart', 'touchmove', 'touchend'...
            ]
        });
    }

    handleClick(event, target, containerElement) {
        const action = target.dataset.action;
        const component = target.closest('[data-component]')?.dataset.component;

        // Smart routing: Component-specific handlers get priority
        if (component && this[`handle${component}Click`]) {
            this[`handle${component}Click`](event, target, containerElement);
        }
        // Fallback to action-based routing for generic behaviors
        else if (action && this[action]) {
            this[action](event, target, containerElement);
        }
    }

    // Component-specific handlers
    handleModalClick(event, target, containerElement) { console.log('🪟 Modal component click', arguments) }
    handleFormClick(event, target, containerElement)  { console.log('📋 Form component click', arguments) }

    // Action handlers
    saveData(event, target, containerElement)         { console.log('💾 Saving data...', arguments) }
    deleteItem(event, target, containerElement)       { console.log('🗑️ Deleting item...', arguments) }
}
```

## 🏗️ **Pattern: Self-Contained Module Architecture**

**Build completely independent modules that manage their own lifecycle, configuration, and event handling.**

**Use When**: Creating reusable components, building plugin systems, or developing modular applications where each module should be completely self-sufficient.

```js
function deepMerge(target, source) {
    for (const key in source) {
        if (source[key] instanceof Object && !Array.isArray(source[key])) {
            target[key] = YaiUtils.deepMerge(target[key] || {}, source[key]);
        } else {
            target[key] = source[key];
        }
    }
    return target;
}

// Self-contained module that handles everything internally
class YpsExample {
    constructor(customConfig = {}) {
        // Deep merge configuration system
        this.config = deepMerge({
            rootSelector: '[data-y-tabs]', // Module root selector
            selectors: {},                 // component-specific selectors
            events: {}                     // Event configuration overrides
        }, customConfig);

        // Own YEH instance with complete lifecycle
        this.events = new YEH({
            [this.config.rootSelector]: ['click']
        }, {
            click: { run: 'runAction' } // Event-scoped aliases
        }, {
            methods: {
                // Event-scoped method organization
                click: {
                    handleClick: (event, target, container) => {
                        const action = target.dataset.tabAction;
                        if (!action) return;

                        // Smart method resolution with fallbacks
                        const handler = this[action] ||
                                      this[this.events.resolveMethodName(action, event.type)];
                        if (handler) return handler.call(this, target, event, container);

                        console.warn(`No handler for action: ${action}`);
                    }
                }
            },
            methodsFirst: true,   // Check methods object first
            abortController: true // Enable clean abort capability
        });
    }

    // Module-specific business logic
    runAction(target, event, container) {
        console.log('🎯 Module action executed:', target.dataset.tabAction);
        // Module handles its own business logic completely independently
    }

    // Lifecycle management - critical for clean module systems
    destroy() {
        this.events.destroy();     // Complete YEH cleanup
        this.config = null;        // Clean up configuration
        // Clean up any other module resources
    }

    abort() {
        this.events.abort();       // Quick abort if abortController enabled
    }
}

// Usage: Multiple independent instances - zero conflicts!
const tabsModule1 = new YpsExample({ rootSelector: '[data-tabs-primary]' });
const tabsModule2 = new YpsExample({ rootSelector: '[data-tabs-secondary]' });

// Each module is completely self-contained and independent
```

**Perfect for building plugin architectures where each module is completely independent!** 🚀

## 🎯 **Pattern: Inline Method Injection**

**Organize your code by co-locating methods with HTML for rapid prototyping and clean separation.**

**Use When**: Building small to medium apps where you want Vue.js-style external methods without the framework overhead.

Write your code where your HTML lives and inject methods into the handler on the fly.

[Live example on JSFiddle](https://jsfiddle.net/xrju1gca/), where I copy&pasted from below to there.

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Advanced Patterns - YEH</title>
</head>
<body>
    <header><h1>Advanced Usage Patterns</h1></header>

    <div id="test-section" data-action="testRunAll">
        Test the power of inline method injection
    </div>

    <form id="user-form">
        <input type="text" data-action="validateForm" placeholder="Username">
        <button type="button" data-action="openModal">Open Settings</button>
    </form>

<script>
// 🚀 External Methods Object (Vue.js style organization)
const advancedMethods = {
    // Global methods (available to all events - think of these as universal handlers)
    testRunAll(event, target, containerElement) {
        console.log('🎯 Test initiated from:', containerElement?.id || 'unknown');
        // Dispatch custom events for inter-module communication
        this.dispatch('test.started', { target: target.dataset.action });
    },

    // Event-scoped methods (organized by event type - prevents naming conflicts!)
    click: {
        validateForm(event, target, containerElement) {
            console.log('📋 Click validation in:', containerElement?.tagName);
        },

        openModal(event, target, containerElement) {
            console.log('🪟 Opening modal from:', target.dataset.action);
            // Cross-module communication via events
            this.dispatch('modal.open', { trigger: target.dataset.action });
        }
    },

    // Same method name, different event scope = zero conflicts!
    input: {
        validateForm(event, target, containerElement) {
            console.log('⚡ Real-time input validation:', target.value);
        }
    }
};
</script>

<script src="https://cdn.jsdelivr.net/npm/@yaijs/yeh@latest/yeh.js"></script>
<script>
// 🚀 Pattern 2: Super Router with Advanced Configuration
class MyHandler extends YEH {
    constructor() {
        super({
            // Target specific sections for true component isolation
            '#test-section': ['click',        ],
            '#user-form':    ['click', 'input'],
            'document': [
                // Emit + Subscribe
                { type: 'test.started', handler: 'onTestEvent'  },
                { type: 'modal.open',   handler: 'onModalEvent' },
                { type: 'app.ready',    handler: 'onAppReady' },
            ]
        }, {}, {
            methods: advancedMethods,        // Inject external methods
            methodsFirst: true,              // Check methods object first (performance optimization)
            enableStats: true,               // Track performance metrics
            enableHandlerValidation: false   // Skip validation for production performance
        });
        // See main README.md "Configuration Options" for all available parameters

        this.cache = new Map();
    }

    // 🚀 Pattern 3: Universal Event Proxy (DRY principle)
    handleClick(event, target, containerElement) {
        this.proxyHandle(event, target, containerElement);
    }
    handleInput(event, target, containerElement) {
        this.proxyHandle(event, target, containerElement);
    }

    // DRY Proxy for data-action routing
    proxyHandle(event, target, containerElement) {
        const action = target.dataset.action;
        if (!action) return;

        // Try instance method first, then fall back to external methods
        if (typeof this[action] === 'function') {
            return this[action](event, target, containerElement);
        }

        // Let YEH resolve from methods object
        const resolvedHandler = this.resolveHandler(action, event.type);
        if (typeof resolvedHandler === 'function') {
            resolvedHandler.call(this, event, target, containerElement);
        }
    }

    // Event listeners for cross-module communication
    onTestEvent(event, target) {
        console.log('🔄 Test event received:', event.detail);
    }

    onModalEvent(event, target) {
        console.log('🔄 Modal event received:', event.detail);
    }

    onAppReady(event, target) {
        console.log('🔄 App is loaded and unlocked, ready to engage!', event.detail);
    }

    // 🚀 Advanced Caching System - Copy & Paste Ready!
    setCache(key, value) {
        this.cache.set(key, value);
    }

    // Smart element finder with caching - perfect for repeated DOM queries
    getCache(key, isMultiple = false, refresh = false, scope = document.body) {
        return this._cacheManager(key, isMultiple, refresh, scope);
    }

    getAllCache() { return this.cache }

    deleteCacheItem(key) { return this.cache.delete(key) } // Clean cache item removal

    clearCache() { this.cache.clear() }

    // Advanced cache manager - handles DOM element caching automatically
    _cacheManager(cacheKey, isMultiple, refresh, scope) {
        // cacheKey doubles as CSS selector - genius optimization!
        let getElements = refresh ? null : this.cache.get(cacheKey);

        if (!getElements || !getElements.length) {
            const select = isMultiple ? 'querySelectorAll' : 'querySelector';
            getElements = scope[select](cacheKey);
            this.cache.set(cacheKey, getElements);
        }

        return getElements;
    }
}

// Initialize the advanced handler
const handler = new AdvancedHandler();

// 🚀 Pattern 5: Static Utilities (Framework-independent)
const throttledSave = YEH.throttle(() => {
    console.log('💾 Auto-saving...');
}, 2000, 'auto-save');

// Global event broadcasting without instances
YEH.dispatch('app.ready', { timestamp: Date.now() });

console.log('📊 Performance stats:', handler.getStats());
</script>
</body>
</html>
```

## 📊 **Pattern: Performance Monitoring**

**Track your app's performance in real-time. Built-in metrics with zero overhead when disabled.**

**Use When**: Optimizing production applications, debugging performance issues, or conducting load testing.

```js
class MyHandler extends YEH {
    constructor() {
        super({
            body: ['click', 'input', 'scroll']
        }, {}, {
            enableStats: true,                 // Enable performance tracking          (default: false)
            enableDistanceCache: true,         // Cache DOM distance calculations      (default: true)
            enableHandlerValidation: false     // Skip validation for production speed (default: false)
        });
        // See main README.md "Configuration Options" for all performance settings

        // Monitor performance every 5 seconds
        setInterval(() => this.logPerformance(), 5000);
    }

    logPerformance() {
        const stats = this.getStats();
        if (stats) {
            console.log('📊 Performance Report:', {
                listeners: stats.totalListeners,
                events: stats.totalEventTypes,
                cacheHitRate: stats.distanceCache.hitRate,
                userInteracted: stats.userHasInteracted
            });
        }
    }

    handleClick(event, target, containerElement) {
        // Use throttled methods for expensive operations
        this.throttle(() => {
            this.heavyComputation();
        }, 1000, 'heavy-work');
    }

    heavyComputation() {
        console.log('🔧 Performing expensive operation...');
    }
}
```

---

## 🎯 **Key Patterns Summary**

1. **Quantum-Entangled Modules** - Independent modules with event communication
2. **Reactive State Management** - React/Vue-style reactivity without framework overhead
3. **Super Delegation** - Single listener handles unlimited elements
4. **Self-Contained Module Architecture** - Complete lifecycle management and plugin systems
5. **Inline Method Injection** - Write methods where HTML lives
6. **Performance Monitoring** - Built-in stats and optimization

---

## 🚨 **Advanced Patterns**

> ⚠️ **These patterns push the boundaries of what's possible with event delegation.**

**Experimental techniques that showcase YEH's flexibility.**

### 🔬 **Experimental Usage Patterns**

*Advanced techniques that demonstrate the library's versatility and power.*

### 🚨 **Pattern: Cross-Tab Event Broadcasting**

```js
// Broadcast events across browser tabs - because why not?
class MyHandler extends YEH {
    constructor() {
        super({ body: ['click'] });

        // Listen for events from other tabs
        window.addEventListener('storage', (e) => {
            if (e.key === 'ypsilon-event') {
                const data = JSON.parse(e.newValue);
                this.dispatch(data.type, data.detail);
            }
        });
    }

    handleClick(event, target) {
        // Broadcast to ALL open tabs instantly
        localStorage.setItem('ypsilon-event', JSON.stringify({
            type: 'cross-tab-click',
            detail: { action: target.dataset.action, timestamp: Date.now() }
        }));
        localStorage.removeItem('ypsilon-event'); // Trigger storage event
    }
}
```

### 🚨 **Pattern: Event-Driven CSS Toggling**

```js
// CSS as a service via events - completely unhinged
class MyHandler extends YEH {
    constructor() {
        super({ body: ['click', { type: 'theme-change', handler: 'applyTheme' }] });
    }

    handleClick(event, target) {
        // Dispatch CSS changes as events
        this.dispatch('theme-change', {
            theme: target.dataset.theme,
            colors: target.dataset.colors?.split(',') || []
        });
    }

    applyTheme(event) {
        const { theme, colors } = event.detail;
        // Inject CSS dynamically via events
        const style = document.createElement('style');
        style.textContent = `
            .theme-${theme} {
                --primary: ${colors[0] || '#000'};
                --secondary: ${colors[1] || '#fff'};
            }
        `;
        document.head.appendChild(style);
        document.body.className = `theme-${theme}`;
    }
}
```

### 🚨 **Pattern: Self-Modifying Handlers**

```js
// Handlers that rewrite themselves - pure madness
class MyHandler extends YEH {
    constructor() {
        super({ body: ['click'] });
        this.clickCount = 0;
    }

    handleClick(event, target) {
        this.clickCount++;

        // After 10 clicks, completely change behavior
        if (this.clickCount === 10) {
            // Replace the handler method at runtime!
            this.handleClick = function(event, target) {
                console.log('🤯 Handler has evolved!');
                // Now dispatch different events
                this.dispatch('handler-evolved', {
                    originalClicks: this.clickCount,
                    newBehavior: 'chaos-mode'
                });
            };
        }

        console.log(`Click ${this.clickCount} - ${this.clickCount < 10 ? 'Normal' : 'ERROR: Should not see this'}`);
    }
}
```

### 🚨 **Pattern: DOM-to-WebWorker Event Bridge**

Revolutionary data flow architecture bridging DOM events to WebWorker computations:

**Event Chain:**
```
DOM input → handleInput() → worker.postMessage() →
  WebWorker computation → worker.onmessage →
    this.dispatch('worker-result') → displayResult()
```

**Key Innovation:** Seamless DOM-to-WebWorker-to-DOM event bridging through YEH's custom event system. Heavy computations never block the UI.

[Live example on JSFiddle](https://jsfiddle.net/j0ev6woq/).

```js
// Bridge DOM events to WebWorkers via custom events - completely out of control
class MyHandler extends YEH {
    constructor() {
        super({ body: ['input'] });

        // Create worker for heavy computations on the fly
        this.worker = new Worker('data:application/javascript,' + encodeURIComponent(`
            self.onmessage = function(e) {
                // Simulate heavy computation
                const result = Array(1000000).fill(0).reduce((a, b, i) => a + i);
                self.postMessage({ type: 'computation-done', result, input: e.data });
            };
        `));

        this.worker.onmessage = (e) => {
            // Worker results become DOM events
            this.dispatch('worker-result', e.data);
        };
    }

    handleInput(event, target) {
        // Send DOM input to WebWorker
        this.worker.postMessage(target.value);
    }
}

// Extend to handle worker results
class WorkerResultHandler extends WebWorkerBridge {
    constructor() {
        super();
        // Add worker result handling
        this.addEvent('document', { type: 'worker-result', handler: 'displayResult' });
    }

    displayResult(event) {
        console.log('🧠 Worker computed:', event.detail);
    }
}

new WorkerResultHandler();
```

**These patterns demonstrate YEH's versatility and power.** 🚀

Advanced event delegation patterns that enable sophisticated architecture with simple implementation - all running on `file://` protocol with zero build tools required.
