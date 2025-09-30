# Iframe Initialization Data Attributes

**Secure, synchronous child iframe access to parent data attributes**

## Problem Statement

Iframes are used by [MCP-UI](https://github.com/idosal/mcp-ui) to embed UI into agents in a simple and secure way. MCP-UI is a specification for bringing interactive web components to the [Model Context Protocol](https://modelcontextprotocol.io/docs/getting-started/intro).

MCP-UI frequently needs to pass initialization data into iframes, such as:

- CSS custom properties for component theming
- Configuration parameters

### Current Limitations

Today, data can reach an iframe only through limited mechanisms:

1. **URL Query Parameters**: Awkward for complex data, limited by URL length (~2000 characters), and exposes data in browser history and server logs.

2. **`postMessage` API**: Requires complex coordination where:
   - The parent must wait for the child to signal readiness
   - The child must delay rendering to prevent flash of unstyled content (FOUC)
   - Timing issues can cause race conditions
   - Requires additional JavaScript overhead

These limitations result in poor user experience and complex implementation patterns.

## Proposal

Introduce a secure, synchronous mechanism for iframes to read declarative data attributes from their parent at load time.

### Key Features

- **Secure by design**: Parent explicitly opts into data sharing per attribute
- **Synchronous access**: Data available immediately when child loads
- **Non-reactive**: Initial data only - updates should use existing `postMessage`
- **Granular control**: Parent chooses exactly which data to expose

### Mechanism

The parent opts into sharing data using new `childdata-*` attributes:

```html
<iframe childdata-theme="dark" childdata-config="..."></iframe>
```

The child frame accesses this data synchronously via `window.parent.dataset`:

```javascript
const theme = window.parent.dataset.theme; // "dark"
```

## Complete Example

### Parent Frame (https://parent.com)

```html
<iframe
  src="https://child.com/widget.html"
  childdata-css='{
    "body": {
      "background": "#121212",
      "color": "#eeeeee"
    },
    "button": {
      "color": "#ff0088",
      "border": "1px solid #333"
    }
  }'
  childdata-theme="dark"
></iframe>
```

### Child Frame (https://child.com/widget.html)

```html
<!DOCTYPE html>
<html>
  <head>
    <title>Child Widget</title>
  </head>
  <body>
    <script>
      // Synchronous access to parent data - no waiting required
      const theme = window.parent.dataset.theme ?? "light";
      const cssConfig = window.parent.dataset.css;
    </script>
  </body>
</html>
```

## Benefits

- **Eliminates FOUC**: No flash of unstyled content since data is available immediately
- **Reduces complexity**: No need for complex `postMessage` coordination
- **Improves performance**: Eliminates round-trip communication overhead

## Alternative Approaches Considered

### Option 1: Single Opt-in Attribute

Instead of `childdata-*` prefixes, use a single attribute to expose all `data-*` attributes:

```html
<iframe
  src="https://child.com/widget.html"
  allowchilddataread
  data-css="..."
  data-theme="dark"
></iframe>
```

**Rejected because:**

- Lacks granular control over which attributes to expose
- Conflicts with reactive nature of existing `data-*` attributes
- All-or-nothing approach is less secure
