> 📘 This is the English version of the documentation.  
> 📗 [Read this documentation in Russian](./README_RU.md)

# Contents  

- [Introduction](#introduction)  
  - [Resources](#resources)
  - [Key Features](#key-features)
  - [Additional Features](#additional-features)
  - [Technical Details](#technical-details)
- [Demo project](#demo-project) 
  - [How to launch](#How-to-launch-the-demo)
  - [Potential Issues](#Potential-Issues)
- [Usage](#usage)  
  - [Getting Started](#getting-started)  
  - [Saving the Graph and Configuring Components](#saving-and-configuration)  
  - [Validation](#validation)  
- [Customization](#customization)  
  - [Custom Port Implementation](#custom-port-implementation)  
  - [PortsDropdown Class](#portsdropdown-class)  
  - [Pushing Out of Passages with PushData](#pushing-out-of-passages)  
  - [Custom TransitionManager Launch Implementation](#custom-transitionmanager-launch-implementation)  
  - [Events and Asynchronous Code Execution](#events-and-asynchronous-code-execution)  
- [Roadmap](#roadmap)  

# Introduction  

World Graph Editor — is a user-friendly, node-based tool that makes it easy to define connections between scenes, streamlining the creation of multi-scene projects. The tool also includes built-in transition mechanics with support for awaiting asynchronous tasks.

## Resources

- [AssetStore Page](https://assetstore.unity.com/packages/slug/306228)  
- [YouTube Tutorial](https://www.youtube.com/watch?v=-ATVm00ctiA)  

## Key Features

- Intuitive and easy-to-use editor window  
- Define one of three connection types between scenes
- Visual highlighting and labeling of GameObjects that represent ports in the graph
- Automatically adds scenes to Build Settings
- Data validation when entering Play Mode or building the project
- Fully compatible with both 2D and 3D projects
- Undo/Redo support
- Unity 6 support

## Additional Features

- TransitionManager prefab for managing scene transitions at runtime
- Option to automatically spawn a player prefab when a new scene loads
- Supports asynchronous code execution during scene transitions
- Includes a demo sample with 10 scenes
- Easily extendable with your own custom components

## Technical details

**Supported OS:**
Windows, macOS, Linux

The tool runs inside the Unity Editor and has been tested on Windows and macOS. Linux compatibility is expected but not extensively tested.

**Supported platforms:**
Windows, macOS, WebGL

No platform-specific limitations were found for other platforms, but no dedicated testing was performed on them.

# Demo Project

The asset is not dependent on any render pipeline; however, the demo scenes use URP and the old input system.

## How to launch the demo:

1. Import **World Graph Editor** into your project.
2. Navigate to `Assets/WorldGraphEditor/Resources/TransitionManager.prefab`  
   and click the `Refresh Build Settings` button under the `Container` field.  
   This will automatically add the required scenes to your Build Settings.
3. Open the `Example 1` scene located at:  
   `Assets/WorldGraphEditor/Examples/Scenes/Example 1.unity`
4. Press Play to run the game.

## Potential Issues

After importing the demo project, the player character may not be able to jump.  
This is usually due to missing or misconfigured ground layer settings.

### How to fix:

1. Create a new physics layer, e.g., `MyGround`.
2. Open the player prefab at:  
   `Assets/WorldGraphEditor/Examples/Prefabs/Player.prefab`  
   and set your new layer in the `LayerMask` field.
3. For each demo scene (`Assets/WorldGraphEditor/Examples/Scenes`),  
   assign the `MyGround` layer to all child objects of the `Ground` GameObject.

Once this is done, the player's jump should work as expected.

# Usage
## Getting Started
### Creating a World Graph Container  

1. In the project folder, create a `WorldGraphContainer`:  
   - **Right-click → Create → World Graph Editor → World Graph Container**  
2. Double-click the created object to open the editor window.  

### Adding Scenes  

Drag and drop the desired scenes into the editor window to create nodes. Each node represents a scene, and each port on the node represents a connection to another scene.

> [!WARNING]  
> Avoid duplicate scenes, as this will cause errors. Duplicates will be highlighted in red.  

### Port Types  

When creating ports using the "+" button on a node, a context menu opens, allowing you to create one of the following port types:  
- **Left Passage**, **Right Passage**, **Top Passage**, **Bottom Passage** — Connection ports (passages) used to connect nodes.  
- **Additional Port** — A port not intended for connections, used for cases like fast-travel systems.  

### Creating Ports and Connections  

1. Click "+" on the selected node and create a connection port (passage).  
2. Drag a connection from the port of the first node to the center of the second node.  
3. If done correctly, the nodes will be connected.  

This way, you define how scenes are connected via passages. Additional port settings are available in the **Inspector** window. Select a node to modify its parameters.  

> [!WARNING]  
> Each port on a node must have a unique name, and the name cannot be empty or consist only of spaces. Otherwise, errors will occur.  

### Types of Node Connections  

- **Undirected** — Default connection. Transitions between passages are allowed in both directions.
- **Shortcut** — Indicates that the passage is initially accessible from only one side (e.g., a locked door that opens from the other side). Can be used in [custom port implementations](#custom-port-implementation) to block transitions based on conditions.  
- **One-Way** — A connection that allows movement only in the specified direction.  

The connection type can be configured in the context menu, which opens by right-clicking an existing connection.  

## Saving and Configuration  
### Saving the Graph  

When you finish working with the graph, click the "Save" button to save it.  

> [!NOTE]  
> When saving, all unconnected ports (except Additional Ports) will be removed.  

> [!NOTE]  
> Upon saving, you will be prompted to add all used scenes to **Build Settings**, as this is required for further work.  

### Configuring TransitionManager  

1. Ensure the manager exists at: `Assets/WorldGraphEditor/Resources/TransitionManager`. If not, create it via:  
   - **Tools → World Graph Editor → Create Transition Manager Prefab**  
2. Drag the saved `WorldGraphContainer` into the corresponding field in `TransitionManager`. This specifies which container will be used.  
3. Set `AutoLoad` to `true`. This allows the manager to load automatically when the game starts.  
4. Enable the checkbox next to the `PlayerPrefab` field and drag the player prefab into it.  

This ensures the player object is automatically created on each scene. If this loading method doesn't suit your needs, you can [use a custom implementation](#custom-transitionmanager-launch-implementation).  

### Scene Configuration  

To represent ports in scenes, add the corresponding components. By default, the following are available:  
- **Passage2D** — Moves the player to the opposite port based on the defined connection in the graph.  
- **Teleport2D** — Moves the player to any other existing port in the graph.  

You can also set a default spawn position by adding an object with the `DefaultSpawnPosition` component to the scene. If no `OutputTransitionComponent` is found, the player will spawn at this position.  

### Component Configuration  

- **Passage2D**  
  - In the `AssignedPort` field, select the name of the passage corresponding to the port in the graph.  

- **Teleport2D**  
  - In the `AssignedPort` field, select the name of the passage corresponding to the port in the graph.  
  - In the `GoTo` field, select the port to which the transition should occur.  

> [!TIP]  
> Each port in the `Assigned Port` field should refer to one port on the node.  

For advanced use cases, you can implement [custom ports](#custom-port-implementation).  

## Validation  
### Expected Behavior  

- On launch, a `TransitionManager` object is automatically created in **DontDestroyOnLoad**.  
- When interacting with a port object, the player moves to another scene at the position corresponding to the target port.  

### Errors  

If the target port is missing in the scene:  
- The player object will spawn at the position defined in `DefaultSpawnPoint` or at (0, 0, 0) if not specified.  
- An error message will be logged to the console.  

> [!WARNING]  
> The game will automatically stop if:  
> - **TransitionManager** is missing at: `Assets/WorldGraphEditor/Resources/TransitionManager`.  
> - The `Container` field in `TransitionManager` is empty.  
> - The container contains errors.  
> - Scenes in **BuildSettings** are deleted, disabled, or reordered.  
>  
> An error with instructions will be logged to the console.  

# Customization  

## Custom Port Implementation  

### General Guidelines  

To create a custom implementation:  
- Inherit from `PassageBase`, `TeleportBase`, or implement `ITransitionComponent`.  
- Use the [PortsDropdown class](#portsdropdown-class) and [related methods](#populating-portsdropdown-with-graph-data) in `WorldGraphContainer` to create dropdowns for initializing the current port or selecting any port in the graph.  
- Implement the `GetGuid()` method, which returns the `Guid` of the current port.  
- Use the `Refresh()` method to update fields with data from `TransitionManager` and `WorldGraphContainer`.  

> [!TIP]  
> Wrap the `Refresh()` method in `#if UNITY_EDITOR`, as it's only needed in the editor. You can call `Refresh()` on all scene components using `TransitionManager.RefreshPorts()`.  

> [!NOTE]  
> `Refresh()` may return `null` if `TransitionManager` is missing at the specified path.  

### Transition Methods in TransitionManager  

```csharp  
Task LoadScene(int buildIndex)    
```
Loads a scene by its build index.  

Parameters:  

| Name         | Type  | Description       |
| ------------ | ----- | ----------------- |
| `buildIndex` | `int` | Scene build index |

---  

```csharp  
void GoFrom(string currentPortGuid, bool ignoreShortcuts)  
```
Moves the player to the opposite port, **based on** graph connections.  

Parameters:  

| Name              | Type     | Description                 |
| ----------------- | -------- | --------------------------- |
| `currentPortGuid` | `string` | `Guid` of the current port  |
| `ignoreShortcuts` | `bool`   | Whether to ignore shortcuts |

---  

```csharp  
void GoTo(string currentPortGuid, string targetPortGuid)    
```
Moves the player to any selected port. **Does not rely on** graph connections.  

Parameters:  

| Name              | Type     | Description                |
| ----------------- | -------- | -------------------------- |
| `currentPortGuid` | `string` | `Guid` of the current port |
| `targetPortGuid`  | `string` | `Guid` of the target port  |

---  

#### Example  

Example of a teleport implementation with destination selection:  

```csharp  
using System.Linq;  
using UnityEngine;  
using UnityEngine.SceneManagement;  

namespace WorldGraphEditor.Examples  
{  
    public class MyExampleTeleport : MonoBehaviour, ITransitionComponent, IInteractable  
    {  
        [SerializeField] private PortsDropdown _assignedPort = new();  
        [SerializeField, ReadOnlyField] private string _currentGuid;  

#if UNITY_EDITOR  
        private WorldGraphContainer _container;  

        public void Refresh(TransitionManager manager)  
        {  
            if (manager != null)  
                _container = manager.Container;  

            if (_container == null)  
                return;  

            // Get current scene build index  
            var sceneBuildIndex = SceneManager.GetActiveScene().buildIndex;  
            
            // Get data about all ports on this scene  
            var data = _container.GetPortsDropdownData(sceneBuildIndex);  
            
            // Set values in dropdown  
            _assignedPort.SetData(data);  

            // Get selected value from dropdown  
            _currentGuid = _assignedPort.GetSelectedValue();  
        }  
#endif  

        private void OnValidate()  
        {  
            TransitionManager.RefreshPorts();  
        }  

        public Vector3 GetSpawnPosition() => transform.position;  
        public string GetGuid() => _currentGuid;  

        public void Interact()  
        {  
            // Example logic to get info about visited teleports  
            var data = MyRuntimeData.VisitedTeleports  
                .Where(item => item.guid != GetGuid())  
                .ToArray();  
            
            // Show window to select teleport destination  
            TeleportSelectionWindow.Instance.ShowDialog(data, OnSelect);  
        }  

        private void OnSelect(string selectedGuid)  
        {  
            // Close teleport selection window  
            TeleportSelectionWindow.Instance.CloseDialog();  
            
            if (selectedGuid == string.Empty)  
                return;  
            
            // Call transition method  
            TransitionManager.Instance.GoTo(GetGuid(), selectedGuid);  
        }  
    }  
}    
```
---  

### PortsDropdown Class  

The `PortsDropdown` class is designed to create dropdowns linked to ports in the graph. It ensures selected values persist even if ports are renamed or their count changes.  

The class includes two methods:  

```csharp  
string GetSelectedValue()   
```

Returns the selected value.  

Return Values:  

| Type     | Description                  |
| -------- | ---------------------------- |
| `string` | The selected value (`Guid`). |

---  

```csharp  
void SetData(IEnumerable<(string Guid, string DisplayName)> data)    
```
Updates dropdown data with `Guid` and display name.  

Parameters:  

| Name   | Type                                             | Description                      |
| ------ | ------------------------------------------------ | -------------------------------- |
| `data` | `IEnumerable<(string Guid, string DisplayName)>` | Data containing `Guid` and name. |

---  

### Populating PortsDropdown with Graph Data  

`WorldGraphContainer` provides **EditorOnly** methods to simplify `PortsDropdown` population:  

```csharp  
IEnumerable<(string Guid, string Name)> GetTransitionsDropdownData(int buildIndex) 
```

Parameters:  

| Name         | Type  | Description                    |
| ------------ | ----- | ------------------------------ |
| `buildIndex` | `int` | Scene index to fetch data for. |

Return Values:  

| Type                                      | Description                      |
| ----------------------------------------- | -------------------------------- |
| `IEnumerable<(string Guid, string Name)>` | Data for all ports on the scene. |

#### Example  

```csharp  
using UnityEngine;  
using UnityEngine.SceneManagement;  

namespace WorldGraphEditor.Examples  
{  
    public class MyComponentWithDropdown : MonoBehaviour, ITransitionComponent  
    {  
        // Dropdown   
        [SerializeField] private PortsDropdown _assignedPortDropdown = new();  
        private string _currentPortGuid;  

#if UNITY_EDITOR  
        private WorldGraphContainer _container;  

        public void Refresh(TransitionManager manager)  
        {  
            if (manager != null)  
                _container = manager.Container;  

            if (_container == null)  
                return;  

            // Get current scene build index  
            var currentSceneBuildIndex = SceneManager.GetActiveScene().buildIndex;  
            // Get data about all ports on this scene  
            var scenePortsData = _container.GetPortsDropdownData(currentSceneBuildIndex);  
            
            // Set data in dropdown  
            _assignedPortDropdown.SetData(scenePortsData);  
        
            // Get selected value from dropdown  
            _currentPortGuid = _assignedPortDropdown.GetSelectedValue();  
        }  
#endif  
    }  
}   
```

---  

```csharp  
IEnumerable<(string Guid, string Path)> GetAllPortsDropdownData()   
```

Returns data for all ports in the graph, grouped by scenes.  

Return Values:  

| Type                                      | Description               |
| ----------------------------------------- | ------------------------- |
| `IEnumerable<(string Guid, string Path)>` | Data for all graph ports. |

#### Example  

```csharp  
using UnityEngine;  

namespace WorldGraphEditor.Examples  
{  
    public class MyComponentWithDropdown : MonoBehaviour, ITransitionComponent  
    {  
        // Dropdown 
        [SerializeField] private PortsDropdown _targetPortDropdown = new();  

        private string _targetPortGuid;  
        
#if UNITY_EDITOR  
        private WorldGraphContainer _container;  
        
        public void Refresh(TransitionManager manager)  
        {  
            if (manager != null)  
                _container = manager.Container;  

            if (_container == null)  
                return;  
        
            // Get data about all ports in the graph  
            var allPortsData = _container.GetAllPortsDropdownData();  
        
            // Set data in dropdown  
            _targetPortDropdown.SetData(allPortsData);  
        
            // Get selected value from dropdown  
            _targetPortGuid = _targetPortDropdown.GetSelectedValue();  
        }  
#endif  
    }  
}  
```

---  

### Pushing Out of Passages  

You can add "push force" to port by implementing `IPusher` and the `GetPushData()` method, which returns `PushData`.  

`PushData` is a struct with two fields:  
- `Force` — The push force.  
- `Exists` — Indicates if the force exists.  

```csharp  
public readonly struct PushData  
{  
    public readonly Vector3 Force;  
    public readonly bool Exists;  

    public PushData(Vector3 force)  
    {  
        Force = force;  
        Exists = true;  
    }  
}   
```

#### Example  

```csharp  
using UnityEngine;  

namespace WorldGraphEditor.Examples  
{  
    public class MyPushingPassage : PassageBase, IPusher  
    {  
        [SerializeField] private Vector2 _pushForce;  
        
        public PushData GetPushData()  
        {  
            // If force is (0, 0), return default PushData  
            if (_pushForce == Vector2.zero)  
                // Returns default value where Exists will be false  
                return default;  
            
            // If force exists, pass it to constructor. Exists will become true  
            return new PushData(_pushForce);  
        }  
        
        // Other methods  
        public override Vector3 GetSpawnPosition() //...  
        private void OnValidate() //...  
    }  
}    
```

To apply this force to the player, use `TransitionManager.Instance.PushData`.  

#### Example  

```csharp  
using UnityEngine;  

namespace WorldGraphEditor.Examples  
{  
    public class PlayerController : MonoBehaviour  
    {  
        [SerializeField] private Rigidbody2D _rigidbody;  

        private void OnTransitionEnded()  
        {  
            // Get push force  
            var pushData = TransitionManager.Instance.PushData;  
            
            // Apply force to Rigidbody only if it exists  
            if (pushData.Exists)  
                _rigidbody.AddForce(pushData.Force, ForceMode2D.Impulse);  
        }  

        private void Awake()  
        {  
            TransitionManager.OnTransitionEnded += OnTransitionEnded;  
        }  
        
        private void OnDestroy()  
        {  
            TransitionManager.OnTransitionEnded -= OnTransitionEnded;  
        }  
    }  
}   
```

## Custom TransitionManager Launch Implementation  

By default, `TransitionManager` loads automatically on game start, but you can create it at any time.  

### Disabling Auto-Load  

1. Set `AutoLoad` to `false`.  
2. Call `TransitionManager.CreateInstance()` to create the manager.  

#### Example  
```csharp  
using UnityEngine;  

namespace WorldGraphEditor.Examples  
{  
    public class MyCustomTransitionManagerLoader : MonoBehaviour  
    {  
        [SerializeField] private float _loadDelay = 2f;  
        
        private void Start()  
        {  
            Invoke(nameof(MyLoadMethod), _loadDelay);  
        }  

        private void MyLoadMethod()  
        {  
            // Create manager  
            TransitionManager.CreateInstance();  
        }  
    }  
}    
```
> [!NOTE]  
> `TransitionManger` also contains `LoadFromResources()`, but it only returns the class instance from **Resources** and doesn't create a **DontDestroyOnLoad** object. Use `CreateInstance()` for proper handling.  

---  

## Events and Asynchronous Code Execution  

`TransitionManager` provides events for transition phases:  

- **OnInitialized**: Called after successful `TransitionManager` initialization.  
- **OnPortEntered**: Called when `GoFrom()` or `GoTo()` is invoked.  
- **OnTransitionStarted**: Called before loading the next scene.  
- **OnSceneLoaded**: Called after the scene has been loaded and after the "output port" `OutputTransitionComponent` has been found and the player's spawn position `PlayerSpawnPosition` has been determined.
- **OnTransitionEnded**: Called after spawning the player prefab (or immediately if no prefab is set). 
- **OnPortLeaved**: Called after `OnTransitionEnded`.  

### Delaying Transition Phases  

You can set a delay between transition phases before calling asynchronous methods by using `EventCallConfig`

1. Create:  
   - **Right-click → Create → World Graph Editor → Event Call Config**  
2. Select a delay type:  
   - **This Frame** — In the current frame.  
   - **Next Frame** — In the next frame.  
   - **Physics Update** — In `FixedUpdate()`.  
   - **Custom Delay** — After a specified time.  
1. Drag the `EventCallConfig` into the corresponding `TransitionManager` field.  

### Awaiting Asynchronous Methods  

If you need to wait for operations such as level generation to complete, you can register asynchronous methods with a specified priority using `TransitionManager.RegisterAsyncHandler()`. The higher the priority value, the earlier the method will be invoked.

> [!TIP]  
> The order of execution during each transition phase will be as follows:
>- Invocation of the transition event (**OnPortEntered / OnTransitionStarted / OnSceneLoaded / OnTransitionEnded**).
>- Waiting for the delay defined by `EventCallConfig`.
>- Waiting for the execution of asynchronous methods.

The type of event after which the registered asynchronous methods begin execution is specified using the `EventType` enum.

Events for which you can register asynchronous methods:

```csharp  
namespace WorldGraphEditor  
{  
    public enum EventType  
    {  
        OnPortEntered,  
        OnTransitionStarted,  
        OnSceneLoaded,  
        OnTransitionEnded,  
    }  
}    
```

---  

### Working with Asynchronous Methods  

```csharp  
static void RegisterAsyncHandler(EventType eventType, Func<Task> func, int priority)    
```

Parameters:  

| Name        | Type         | Description                                |
| ----------- | ------------ | ------------------------------------------ |
| `eventType` | `EventType`  | Event triggering the async methods.        |
| `func`      | `Func<Task>` | The `Task` — returning method to register. |
| `priority`  | `int`        | Execution priority (higher = earlier).     |

---  

```csharp  
static void UnregisterAsyncHandler(EventType eventType, Func<Task> func)   
```

Parameters:  

| Name        | Type         | Description                             |
| ----------- | ------------ | --------------------------------------- |
| `eventType` | `EventType`  | Event to unregister from.               |
| `func`      | `Func<Task>` | The `Task`— returning method to remove. |

---  

> [!NOTE]  
> `TransitionManager` automatically clears all registered methods on `OnDestroy()`.  

#### Example  

```csharp  
using System.Threading.Tasks;  
using UnityEngine;  
using UnityEngine.UI;  

namespace WorldGraphEditor.Examples  
{  
    public class MyWorldGenerator : MonoBehaviour  
    {  
        [SerializeField] private Text _worldTextMeshStatus;  
        [SerializeField] private Text _lootTextMeshStatus;  
        [SerializeField] private Text _enemiesTextMeshStatus;  

        [SerializeField] private float _worldGenerationTime;  
        [SerializeField] private float _spawnLootTime;  
        [SerializeField] private float _spawnEnemiesTime;  
        
        // Event after which async operations will execute. Here - after scene load  
        private EventType _eventType = EventType.OnSceneLoaded;  

        private void Awake()  
        {  
            // Will execute first due to high priority  
            TransitionManager.RegisterAsyncHandler(_eventType, GenerateWorld, 2);  
            
            // Will execute after, in registration order (same priority)  
            TransitionManager.RegisterAsyncHandler(_eventType, SpawnLoot, 1);  
            TransitionManager.RegisterAsyncHandler(_eventType, SpawnEnemies, 1);  
            
            _lootTextMeshStatus.text = "Waiting...";  
            _enemiesTextMeshStatus.text = "Waiting...";  
        }  
        
        // Async methods  
        // Each method first updates the text, then waits for the specified time, and updates the text again
        private async Task GenerateWorld()  
        {  
            _worldTextMeshStatus.text = "Generating...";  
            await Awaitable.WaitForSecondsAsync(_worldGenerationTime);  
            _worldTextMeshStatus.text = "Complete!";  
        }  
        
        private async Task SpawnLoot()  
        {  
            _lootTextMeshStatus.text = "Spawning...";  
            await Awaitable.WaitForSecondsAsync(_spawnLootTime);  
            _lootTextMeshStatus.text = "Complete!";  
        }  

        private async Task SpawnEnemies()  
        {  
            _enemiesTextMeshStatus.text = "Spawning...";  
            await Awaitable.WaitForSecondsAsync(_spawnEnemiesTime);  
            _enemiesTextMeshStatus.text = "Complete!";  
        }  
        
        private void OnDisable()  
        {  
            // Remove methods  
            TransitionManager.UnregisterAsyncHandler(_eventType, GenerateWorld);  
            TransitionManager.UnregisterAsyncHandler(_eventType, SpawnLoot);  
            TransitionManager.UnregisterAsyncHandler(_eventType, SpawnEnemies);  
        }  
    }  
}    
```
---  

## Roadmap  

- Support for multiple scenes (variants) per node.  
- Validation of all scenes in the graph (detecting unassigned ports, duplicates, etc.).  
- Support for custom `TransitionManager` implementations.  
- Node customization.  
- Pathfinding algorithm for selected scenes.  