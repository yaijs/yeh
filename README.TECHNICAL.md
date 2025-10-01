# YEH - Technical Documentation

> Implementation details, architecture decisions, and performance characteristics

This document covers the technical internals of YEH (YpsilonEventHandler) for developers who want to understand how it works under the hood.

**🔬 [Feature Demonstrations](https://eypsilon.github.io/YpsilonEventHandler-Examples/example/public/features/)** - Interactive examples of technical concepts

---

## 🎯 Core Philosophy: Event Abstraction Layer

YEH is fundamentally designed around **event delegation** - the concept of using a single listener to handle multiple, or even dynamically added elements efficiently and above all lossless. This isn't just an optimization; it's the entire architectural foundation.

### Traditional Approach vs Event Delegation

**Traditional: One listener per element (even with shared handlers, still individual registrations):**

```js
button1.addEventListener('click', handler1);
button2.addEventListener('click', handler2);
button3.addEventListener('click', handler3);
// Result: 1000 buttons = 1000 listeners
// Need to add buttons? Must add listeners. ❌
```

**Event Delegation: One listener on a container element handles all events triggered within:**

```js
document.body.addEventListener('click', (e) => {
  if (e.target.matches('button')) {
    // Handle any button click and route automatically to
    // appropriate methods via attributes, mappings, etc.
  }
});
// Result: 1000 buttons = 1 listener
// Need to add buttons? Throw as many as the browser can render.
// Still 1, in fact, still the same listener. Not even a refresh. ✅
```

This is especially useful if you have a lot of form elements (e.g., editable configurations via a GUI). Instead of adding a new listener for each input element and then properly removing them when you're done, we simply inject our HTML snippets/templates into the page. They trigger their corresponding events as well that we catch on the parent level and process. When done, we remove the HTML snippets the same way we inserted them - without a second thought. Where "without a second thought" doesn't mean "thoughtless," but rather "it's simply not necessary."

### Event-as-a-Service (EaaS) Architecture

YEH treats events as a **service system**:

- **Single registration point** - One listener per container per event type
- **Automatic routing** - Events are intelligently routed to appropriate handlers
- **Dynamic element support** - New elements work automatically without re-registration
- **Service-level features** - Throttling, debouncing, and performance monitoring built-in

```js
// One service instance handles unlimited elements
new YEH({
  body:   ['click', 'input'] // 3 listeners for
  window: ['scroll']         // entire application
});
// Auto-routed to main handlers
handleClick(e, t);
handleInput(e, t);
handleScroll(e, t);
```

**This is the fundamental paradigm shift:** Instead of managing individual element listeners, you manage **event services** that handle entire application sections.

---

## 🏗️ **Core Architecture**

### **Multi-Handler System with Closest-Match Resolution**

The revolutionary feature that sets YEH apart from all other event libraries:

```js
// Multiple handlers for the same event type
new YEH({
  'body': [
    { type: 'click', handler: 'handleGeneralClick' }
  ],
  '.modal': [
    { type: 'click', handler: 'handleModalClick' }
  ],
  '#specific-button': [
    { type: 'click', handler: 'handleSpecificClick' }
  ]
});
```

**How it works:**
1. When an event occurs, the system finds ALL registered handlers for that event type
2. Calculates DOM distance from event target to each handler's container
3. Selects the handler with the CLOSEST container to the event target
4. Only ONE handler executes - the most specific one

### **Native handleEvent Interface**

Uses the browser's built-in `handleEvent` interface for optimal performance:

```js
// YEH instances are registered directly
element.addEventListener('click', this, options);

// Browser calls this.handleEvent(event)
handleEvent(event) {
  // Route to appropriate method based on configuration
}
```

**Benefits:**
- Zero synthetic event overhead
- Direct browser API usage
- Automatic `this` binding
- Native performance characteristics

---

## ⚡ **Performance Optimizations**

### **Automatic Passive Listeners**

Automatically applies passive listeners to scroll-related events:

```js
const PASSIVE_EVENTS = ['scroll', 'touchstart', 'touchmove', 'wheel'];

// Applied automatically - no configuration needed
element.addEventListener('scroll', handler, { passive: true });
```

### **Distance Calculation Caching**

DOM distance calculations are cached when multiple handlers exist:

```js
// Cached to avoid repeated DOM traversal
const distance = this.distanceCache.get(cacheKey) ||
  this.calculateDOMDistance(target, container);
```

### **Handler Validation**

Method existence is validated during registration (development mode):

```js
// Warns about missing handlers during setup
if (typeof this[handlerName] !== 'function') {
  console.warn(`Handler method '${handlerName}' not found`);
}
```

---

## 🧠 **Memory Management**

### **Dual Map System**

Uses both Map and WeakMap for optimal memory management:

```js
// Strong references for cleanup
this.eventListeners = new Map();

// Weak references for garbage collection
this.elementHandlers = new WeakMap();
```

**Why both?**
- `Map` allows complete cleanup via `destroy()`
- `WeakMap` automatically releases memory when elements are removed from DOM

### **Automatic Cleanup**

The `destroy()` method provides comprehensive cleanup:

```js
destroy() {
  // Remove all event listeners
  for (const [element, listeners] of this.eventListeners) {
    for (const config of listeners) {
      element.removeEventListener(config.type, this, config.options);
    }
  }

  // Clear all maps
  this.eventListeners.clear();
  this.eventHandlerMap.clear();

  // WeakMap clears automatically
}
```

---

## 🎯 **Event Routing System**

### **Method Resolution Priority**

Handler method resolution follows this priority:

1. **Explicit handler mapping**: `{ type: 'click', handler: 'myHandler' }`
2. **External methods object**: `options.methods.click.myHandler`
3. **Class methods**: `this.handleClick`
4. **Global fallback**: `window.handleClick` (if enabled)

### **Container Element Resolution**

Each handler receives three parameters:

```js
handleClick(event, target, containerElement) {
  // event: Native event object
  // target: Element that triggered the event
  // containerElement: Element where listener was registered
}
```

**Container resolution logic:**
```js
const containerElement = this.findContainer(event.target, selector);
// Returns the matching parent element for the given selector
```

---

## 🔧 **Browser Compatibility Implementation**

### **Element.closest() Polyfill**

For browsers that don't support `Element.closest()`:

```js
findClosest(element, selector) {
  if (element.closest) {
    return element.closest(selector);
  }

  // Manual traversal fallback
  let current = element;
  while (current && current !== document) {
    if (this.matches(current, selector)) {
      return current;
    }
    current = current.parentElement;
  }
  return null;
}
```

### **Passive Support Detection**

One-time global detection with caching:

```js
static isPassiveSupported() {
  if (YEH._passiveSupportCache !== undefined) {
    return YEH._passiveSupportCache;
  }

  try {
    const opts = Object.defineProperty({}, 'passive', {
      get: () => { YEH._passiveSupportCache = true; }
    });
    window.addEventListener('test', null, opts);
    window.removeEventListener('test', null, opts);
  } catch (e) {
    YEH._passiveSupportCache = false;
  }

  return YEH._passiveSupportCache;
}
```

---

## 📊 **Performance Characteristics**

### **Memory Usage**

- **Per instance**: ~2KB base memory footprint
- **Per listener**: ~100 bytes additional overhead
- **WeakMap entries**: 0 bytes (automatic garbage collection)

### **Event Processing Speed**

- **Handler resolution**: O(1) for single handlers, O(n) for multiple handlers
- **DOM distance calculation**: O(d) where d = DOM depth
- **Method dispatch**: O(1) after resolution

### **Scalability**

- **Elements handled**: Unlimited (single listener per event type per container)
- **Event types**: Unlimited (each type managed independently)
- **Dynamic elements**: Automatic (no re-registration needed)

---

## 🎛️ **Configuration Deep Dive**

### **Event Mapping Structure**

```js
const eventMapping = {
  'selector': [
    'eventType',  // Convention: eventType → handleEventType
    { type: 'eventType', handler: 'customHandler' },
    { type: 'scroll', throttle: 250, passive: true },
    { type: 'input', debounce: 300 },
    { type: 'click', options: { once: true } }
  ]
};
```

### **Options Object**

```js
const options = {
  enableStats: false,            // Performance tracking
  enableDistanceCache: true,     // Cache DOM calculations
  enableHandlerValidation: true, // Warn about missing methods
  methods: {},                   // External method mapping
  methodsFirst: false,           // Priority: methods before class
  enableGlobalFallback: false,   // Fallback to window functions
  passiveEvents: [...],          // Custom passive event list
  abortController: false         // AbortController support
};
```

### **Available Configuration Options (Parameter 3 with defaults)**

```json
{
  // Handler Resolution System
  handlerPrefix:           'handle', // Auto-generated method prefix (handle → handleClick)
  enableGlobalFallback:    false,    // Search window/global scope for missing handlers
  methods:                 null,     // External methods object (Vue.js style)
  methodsFirst:            false,    // Check methods object before instance methods

  // Modern Event Management
  abortController:         false,    // Enable AbortController for easy cleanup

  // Smart target resolution
  autoTargetResolution:    false,    // Automatically resolve actionable targets
  targetResolutionEvents:  [         // Events that should use smart target resolution (e.target || e.currentTarget)
    'click', 'touchstart', 'touchend', 'mousedown', 'mouseup'
  ],

  // Performance Optimization
  enableStats:             false,    // Track performance metrics
  enableDistanceCache:     true,     // Enable DOM distance caching (default: true)
  enableConfigValidation:  true,     // Enable comprehensive configuration validation (default: true)

  // Actionable Target Configuration (NEW v1.6.6+)
  enableActionableTargets: true,            // Enable actionable target system
  actionableAttributes:    ['data-action'], // Custom actionable attributes
  actionableClasses:       ['actionable'],  // Custom actionable CSS classes
  actionableTags:          ['BUTTON', 'A'], // Custom actionable HTML tags

  // Event Behavior
  passiveEvents: [ // Events that should be passive for performance
    'scroll', 'touchstart', 'touchmove', 'touchend', 'touchcancel',
    'wheel', 'mousewheel', 'pointermove', 'pointerenter', 'pointerleave',
    'resize', 'orientationchange', 'load', 'beforeunload', 'unload'
  ],
}
```

---

## 🚀 **Advanced Implementation Notes**

### **Why No Synthetic Events?**

YEH deliberately avoids synthetic events:

- **Performance**: No wrapper object creation
- **Compatibility**: Works with all browser APIs
- **Debugging**: Native events in DevTools
- **Memory**: No additional object allocation

### **Why handleEvent Interface?**

The native `handleEvent` interface provides:

- **Automatic `this` binding**: No `.bind()` needed
- **Single entry point**: Centralized event routing
- **Browser optimization**: Native code path
- **Standard compliance**: Following web standards

### **Why Map/WeakMap Combination?**

- **Map**: Allows iteration for cleanup
- **WeakMap**: Automatic garbage collection
- **Performance**: O(1) lookups in both
- **Memory safety**: No dangling references

---

## 🔬 **Debugging and Development**

### **Performance Statistics**

When `enableStats: true`:

```js
const stats = handler.getStats();
/*
{
  totalListeners: 8,
  totalEventTypes: 3,
  eventsProcessed: 1247,
  distanceCache: {
    hits: 1150,
    misses: 97,
    hitRate: 0.922
  },
  handlerResolutionTime: {
    avg: 0.12,
    max: 2.3,
    min: 0.05
  },
  userHasInteracted: true
}
*/
```

### **Debug Output**

Enable detailed logging for development:

```js
// In browser console
YEH.DEBUG = true;

// Shows handler resolution, distance calculations, method routing
```

---

## 💡 **Design Philosophy**

### **Principles**

1. **Browser-native first**: Use standard APIs whenever possible
2. **Zero dependencies**: No external libraries required
3. **Performance by default**: Optimizations should be automatic
4. **Memory safety**: Prevent leaks through design
5. **Developer experience**: Clear errors, helpful warnings

### **Trade-offs**

- **Bundle size vs features**: Comprehensive but compact
- **Performance vs flexibility**: Optimized common paths
- **Standards compliance vs convenience**: Standards win
- **Compatibility vs modernity**: Support what matters

---

**The technical implementation reflects 10+ years of JavaScript event handling evolution, distilled into a single, powerful, standards-compliant library.** 🎯
