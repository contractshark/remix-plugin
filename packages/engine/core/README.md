# Engine Core

This is the core library used to create a new plugin engine.

| Name                                           | Latest Version       |
| -----------------------------------------------| :------------------: |
| [@remixproject/engine](.)  | [![badge](https://img.shields.io/npm/v/@remixproject/engine.svg?style=flat-square)](https://www.npmjs.com/package/@remixproject/engine) |

Use this library if you want to create an engine **for a new environment**.

If you want to create an engine for an existing envrionment, use the specific library. For example : 
- Engine on the web : [@remixproject/engine-web](../web)
- Engine on node : [@remixproject/engine-node](../node)
- Engine on vscode : [@remixproject/engine-vscode](../vscode)

## Tutorial

1. [Getting Started](doc/tutorial/1-getting-started.md)
2. [Plugin Communication](doc/tutorial/2-plugin-communication.md)
3. [Host a Plugin with UI](doc/tutorial/3-hosted-plugin.md)
4. [External Plugins](doc/tutorial/4-external-plugins.md)
5. [Plugin Service](doc/tutorial/5-plugin-service.md)

## API

| API                         | Description                          |
| ----------------------------| :----------------------------------: |
| [Engine](./api/engine.md)   | Register plugins & redirect messages |
| [Manager](./api/manager.md) | Activate & Deactive plugins          |


## Connector

The plugin connector is the main component of `@remixproject/engine`, it defines how an external plugin can connect to the engine. Checkout the [documentation](./doc/connector).

--------------

## Getting started
```
npm install @remixproject/engine
```

The engine works a with two classes : 
- `PluginManager`: manage activation/deactivation
- `Engine`: manage registration & communication 

```typescript
import { PluginManager, Engine, Plugin } from '@remixproject/engine'

const manager = new PluginManager()
const engine = new Engine()
const plugin = new Plugin({ name: 'plugin-name' })

// Wait for the manager to be loaded
await engine.onload()

// Register plugins
engine.register([manager, plugin])

// Activate plugins
manager.activatePlugin('plugin-name')
```

### Registration
The registration makes the plugin available for activation in the engine.

To register a plugin you need: 
- `Profile`: The ID card of your plugin.
- `Plugin`: A class that expose the logic of the plugin.

```typescript
const profile = {
  name: 'console',
  methods: ['print']
}

class Console extends Plugin {
  constructor() {
    super(profile)
  }
  print(msg: string) {
    console.log(msg)
  }
}
const consolePlugin = new Console()

// Register plugins
engine.register(consolePlugin)
```

> In the future, this part will be manage by a `Marketplace` plugin.

### Activation
The activation process is managed by the `PluginManager`.

Activating a plugin makes it visible to other plugins. Now they can communicate.

```typescript
manager.activatePlugin('console')
```

> The `PluginManager` is a plugin itself.

### Communication
`Plugin` exposes a simple interface for communicate between plugins : 

- `call`: Call a method exposed by another plugin (This returns always a Promise).
- `on`: Listen on event emitted by another plugin.
- `emit`: Emit an event broadcasted to all listeners.

This code will call the method `print` from the plugin `console` with the parameter `'My message'`.
```typescript
plugin.call('console', 'print', 'My message')
```

### Full code example
```typescript
import { PluginManager, Engine, Plugin } from '@remixproject/engine'
const profile = {
  name: 'console',
  methods: ['print']
}

class Console extends Plugin {
  constructor() {
    super(profile)
  }
  print(msg: string) {
    console.log(msg)
  }
}

const manager = new PluginManager()
const engine = new Engine()
const emptyPlugin = new Plugin({ name: 'empty' })
const consolePlugin = new Console()

// Register plugins
engine.register([manager, plugin, consolePlugin])

// Activate plugins
manager.activatePlugin(['empty', 'console'])

// Plugin communication
emptyPlugin.call('console', 'print', 'My message')
```