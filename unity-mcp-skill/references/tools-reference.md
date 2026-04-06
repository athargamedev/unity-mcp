# Unity-MCP Tools Reference

Complete reference for all MCP tools. Each tool includes parameters, types, and usage examples.

> **Template warning:** Examples in this file are skill templates and may be inaccurate for some Unity versions, packages, or project setups. Validate parameters and payload shapes against your active tool schema and runtime behavior.

## Table of Contents

- [Infrastructure Tools](#infrastructure-tools)
- [Scene Tools](#scene-tools)
- [GameObject Tools](#gameobject-tools)
- [Script Tools](#script-tools)
- [Asset Tools](#asset-tools)
- [Material & Shader Tools](#material--shader-tools)
- [UI Tools](#ui-tools)
- [Editor Control Tools](#editor-control-tools)
- [Testing Tools](#testing-tools)
- [Camera Tools](#camera-tools)
- [Graphics Tools](#graphics-tools)
- [Package Tools](#package-tools)
- [Physics Tools](#physics-tools)
- [ProBuilder Tools](#probuilder-tools)
- [Profiler Tools](#profiler-tools)
- [Docs Tools](#docs-tools)

---

## Project Info Resource

Read `mcpforunity://project/info` to detect project capabilities before making assumptions about UI, input, or rendering setup.

**Returned fields:**

| Field | Type | Description |
|-------|------|-------------|
| `projectRoot` | string | Absolute path to project root |
| `projectName` | string | Project folder name |
| `unityVersion` | string | e.g. `"2022.3.20f1"` |
| `platform` | string | Active build target e.g. `"StandaloneWindows64"` |
| `assetsPath` | string | Absolute path to Assets folder |
| `renderPipeline` | string | `"BuiltIn"`, `"Universal"`, `"HighDefinition"`, or `"Custom"` |
| `activeInputHandler` | string | `"Old"`, `"New"`, or `"Both"` |
| `packages.ugui` | bool | `com.unity.ugui` installed (Canvas, Image, Button, etc.) |
| `packages.textmeshpro` | bool | `com.unity.textmeshpro` installed (TMP_Text, TMP_InputField) |
| `packages.inputsystem` | bool | `com.unity.inputsystem` installed (InputAction, PlayerInput) |
| `packages.uiToolkit` | bool | Always `true` for Unity 2021.3+ (UIDocument, VisualElement, UXML/USS) |
| `packages.screenCapture` | bool | `com.unity.modules.screencapture` enabled (ScreenCapture API for screenshots) |

**Key decision points:**

- **UI system**: If `packages.uiToolkit` is true (always for Unity 2021+), use `manage_ui` for UI Toolkit workflows (UXML/USS). If `packages.ugui` is true, use Canvas + uGUI components via `batch_execute`. UI Toolkit is preferred for new UI — it uses a frontend-like workflow (UXML for structure, USS for styling).
- **Text**: If `packages.textmeshpro` is true, use `TextMeshProUGUI` instead of legacy `Text`.
- **Input**: Use `activeInputHandler` to decide EventSystem module — `StandaloneInputModule` (Old) vs `InputSystemUIInputModule` (New). See [workflows.md — Input System](workflows.md#input-system-old-vs-new).
- **Shaders**: Use `renderPipeline` to pick correct shader names — `Standard` (BuiltIn) vs `Universal Render Pipeline/Lit` (URP) vs `HDRP/Lit` (HDRP).

---

## Infrastructure Tools

### `batch_execute`
Run multiple tool calls in one request.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `commands` | `list[object]` | yes | Max 25 by default; each item: `{ "tool": str, "params": object }` |
| `parallel` | `bool` | no | Advisory hint; Unity may still execute sequentially |
| `fail_fast` | `bool` | no | Stop on first command failure |
| `max_parallelism` | `int` | no | Max parallel workers |

```python
batch_execute(commands=[
    {"tool": "manage_gameobject", "params": {"action": "create", "name": "CubeA", "primitive_type": "Cube"}},
    {"tool": "manage_gameobject", "params": {"action": "create", "name": "CubeB", "primitive_type": "Cube"}}
], fail_fast=True)
```

### `set_active_instance`
Route subsequent tool calls to a specific Unity editor instance.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `instance` | `str` | yes | `Name@hash` or hash prefix |

### `refresh_unity`
Refresh assets and optionally request compilation.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `mode` | `str` | no | `if_dirty` or `force` |
| `scope` | `str` | no | `assets`, `scripts`, or `all` |
| `compile` | `str` | no | `none` or `request` |
| `wait_for_ready` | `bool` | no | Wait until editor is ready |

```python
refresh_unity(mode="force", scope="scripts", compile="request", wait_for_ready=True)
```

## Scene Tools

### `manage_scene`
Scene operations and hierarchy/screenshot queries.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `action` | `str` | yes | `get_hierarchy`, `screenshot`, `get_active`, `get_build_settings`, `create`, `load`, `save` |
| `page_size` | `int` | no | For `get_hierarchy`; default 50, max 500 |
| `cursor` | `int` | no | Pagination cursor |
| `parent` | `str|int` | no | Parent filter |
| `include_transform` | `bool` | no | Include transform data |
| `name` | `str` | no | For `create` |
| `path` | `str` | no | For `create`/`load` |

```python
manage_scene(action="get_hierarchy", page_size=50, cursor=0)
manage_scene(action="screenshot")
```

### `find_gameobjects`
Search for GameObjects and return IDs.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `search_term` | `str` | yes | Search value |
| `search_method` | `str` | no | `by_name`, `by_tag`, `by_layer`, `by_component`, `by_path`, `by_id` |
| `include_inactive` | `bool|str` | no | Include inactive objects |
| `page_size` | `int` | no | Default 50, max 500 |
| `cursor` | `int` | no | Pagination cursor |

```python
find_gameobjects(search_term="Player", search_method="by_name", page_size=50)
```

## GameObject Tools

### `manage_gameobject`
Create, modify, delete, duplicate, or move GameObjects.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `action` | `str` | yes | `create`, `modify`, `delete`, `duplicate`, `move_relative` |
| `name` | `str` | no | Name for `create` |
| `primitive_type` | `str` | no | `Cube`, `Sphere`, `Capsule`, `Cylinder`, `Plane`, `Quad` |
| `target` | `str|int` | no | Name/path/instance ID |
| `search_method` | `str` | no | Target lookup strategy |
| `position` | `list[float]|str` | no | World position |
| `rotation` | `list[float]|str` | no | Euler rotation |
| `scale` | `list[float]|str` | no | Scale |
| `set_active` | `bool` | no | Active state |
| `layer` | `str|int` | no | Layer name/index |
| `components_to_add` | `list[str]` | no | Components to add |
| `components_to_remove` | `list[str]` | no | Components to remove |
| `component_properties` | `dict` | no | Nested component property payload |
| `save_as_prefab` | `bool` | no | Save created object as prefab |
| `prefab_path` | `str` | no | Prefab destination |
| `new_name` | `str` | no | For `duplicate` |
| `offset` | `list[float]|str` | no | Duplicate position offset |
| `reference_object` | `str|int` | no | For `move_relative` |
| `direction` | `str` | no | `left`, `right`, `up`, `down`, `forward`, `back` |
| `distance` | `float` | no | Move distance |
| `world_space` | `bool` | no | Relative move in world space |
| `parent` | `str|int` | no | Parent object for creation |

```python
manage_gameobject(action="create", name="MyCube", primitive_type="Cube", position=[0,1,0])
manage_gameobject(action="modify", target=12345, scale=[2,2,2])
```

### `manage_components`
Add/remove components or set component properties.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `action` | `str` | yes | `add`, `remove`, `set_property` |
| `target` | `str|int` | yes | Prefer instance ID |
| `component_type` | `str` | yes | Component class/type name |
| `search_method` | `str` | no | Lookup strategy when target is not ID |
| `property` | `str` | no | Single property key |
| `value` | `any` | no | Value for single property |
| `properties` | `dict` | no | Multiple property key/value pairs |

```python
manage_components(action="add", target=12345, component_type="Rigidbody")
manage_components(action="set_property", target=12345, component_type="Rigidbody", property="mass", value=5.0)
```

## Script Tools

### `create_script`
Create a new C# script file.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `path` | `str` | yes | Assets-relative C# file path |
| `contents` | `str` | yes | Full script content |
| `script_type` | `str` | no | Optional hint, e.g. `MonoBehaviour` |
| `namespace` | `str` | no | Optional namespace |

```python
create_script(path="Assets/Scripts/MyScript.cs", contents="using UnityEngine;\npublic class MyScript : MonoBehaviour {}")
```

### `script_apply_edits`
Apply structured code edits by operations.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `name` | `str` | yes | Script name without `.cs` |
| `path` | `str` | yes | Folder path |
| `edits` | `list[object]` | yes | Edit ops: `replace_method`, `insert_method`, `delete_method`, `anchor_insert`, `regex_replace`, `prepend`, `append` |

```python
script_apply_edits(name="PlayerController", path="Assets/Scripts", edits=[
    {"op": "replace_method", "methodName": "Update", "replacement": "void Update() { }"}
])
```

### `apply_text_edits`
Apply exact text edits by line/column ranges (1-indexed).

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `uri` | `str` | yes | File URI, usually `mcpforunity://path/...` |
| `edits` | `list[object]` | yes | Items with `startLine`, `startCol`, `endLine`, `endCol`, `newText` |
| `precondition_sha256` | `str` | no | Prevent stale-file writes |
| `strict` | `bool` | no | Enable stricter checks |

```python
apply_text_edits(uri="mcpforunity://path/Assets/Scripts/MyScript.cs", edits=[
    {"startLine": 10, "startCol": 1, "endLine": 10, "endCol": 5, "newText": "void FixedUpdate()"}
], strict=True)
```

### `validate_script`
Validate C# script syntax/semantics.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `uri` | `str` | yes | Script URI |
| `level` | `str` | no | `basic` or `standard` |
| `include_diagnostics` | `bool` | no | Include detailed diagnostics |

```python
validate_script(uri="mcpforunity://path/Assets/Scripts/MyScript.cs", level="standard", include_diagnostics=True)
```

### `get_sha`
Return file SHA without fetching file content.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `uri` | `str` | yes | File URI |

### `delete_script`
Delete a script file.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `uri` | `str` | yes | Script URI |

## Asset Tools

### `manage_asset`
Search/import/create/modify/move/rename/delete assets.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `action` | `str` | yes | `search`, `get_info`, `create`, `duplicate`, `move`, `rename`, `create_folder`, `delete` |
| `path` | `str` | no | Path or search scope |
| `search_pattern` | `str` | no | Glob or filter expression |
| `filter_type` | `str` | no | Asset type filter |
| `page_size` | `int` | no | Search pagination size |
| `page_number` | `int` | no | Search pagination page (1-based) |
| `generate_preview` | `bool` | no | Include previews |
| `asset_type` | `str` | no | For `create` |
| `properties` | `dict` | no | Create/modify payload |
| `destination` | `str` | no | For `duplicate`/`move`/`rename` |

```python
manage_asset(action="search", path="Assets", search_pattern="*.prefab", page_size=25, page_number=1)
manage_asset(action="create_folder", path="Assets/Materials")
```

### `manage_prefabs`
Headless prefab inspection and modification.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `action` | `str` | yes | `get_info`, `get_hierarchy`, `create_from_gameobject`, `modify_contents` |
| `prefab_path` | `str` | yes | Assets-relative prefab path |
| `target` | `str|int` | no | Scene object or prefab child target |
| `allow_overwrite` | `bool` | no | For prefab creation |
| `position` | `list[float]|str` | no | For `modify_contents` |
| `components_to_add` | `list[str]` | no | For `modify_contents` |

```python
manage_prefabs(action="get_hierarchy", prefab_path="Assets/Prefabs/Player.prefab")

# Create prefab from scene GameObject
manage_prefabs(
    action="create_from_gameobject",
    target="Player",             # GameObject in scene
    prefab_path="Assets/Prefabs/Player.prefab",
    allow_overwrite=False
)

# Modify prefab contents (headless)
manage_prefabs(
    action="modify_contents",
    prefab_path="Assets/Prefabs/Player.prefab",
    target="ChildObject",        # object within prefab
    position=[0, 1, 0],
    components_to_add=["AudioSource"]
)

# Delete child GameObjects from prefab
manage_prefabs(
    action="modify_contents",
    prefab_path="Assets/Prefabs/Player.prefab",
    delete_child=["OldChild", "Turret/Barrel"]  # single string or list
)

# Create child GameObject in prefab
manage_prefabs(
    action="modify_contents",
    prefab_path="Assets/Prefabs/Player.prefab",
    create_child={"name": "SpawnPoint", "primitive_type": "Sphere", "position": [0, 2, 0]}
)

# Set component properties on prefab contents
manage_prefabs(
    action="modify_contents",
    prefab_path="Assets/Prefabs/Player.prefab",
    target="ChildObject",
    component_properties={"Rigidbody": {"mass": 5.0}, "MyScript": {"health": 100}}
)
```

---

## Material & Shader Tools

### manage_material

Create and modify materials.

```python
manage_material(action="create", material_path="Assets/Materials/Red.mat", shader="Standard")
manage_material(action="assign_material_to_renderer", target="MyCube", material_path="Assets/Materials/Red.mat", slot=0)
```

### `manage_texture`
Create and procedurally modify textures.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `action` | `str` | yes | `create`, `apply_pattern`, `apply_gradient` |
| `path` | `str` | yes | Texture asset path |
| `width` | `int` | no | For `create` |
| `height` | `int` | no | For `create` |
| `fill_color` | `list[number]` | no | RGBA |
| `pattern` | `str` | no | `checkerboard`, `stripes`, `dots`, `grid`, `brick` |
| `palette` | `list[list[number]]` | no | Colors |
| `pattern_size` | `int` | no | Pattern scale |
| `gradient_type` | `str` | no | `linear` or `radial` |
| `gradient_angle` | `number` | no | Gradient angle |

```python
manage_texture(action="create", path="Assets/Textures/Checker.png", width=64, height=64, fill_color=[255,255,255,255])
manage_texture(action="apply_pattern", path="Assets/Textures/Checker.png", pattern="checkerboard", pattern_size=8)
```

## Editor Control Tools

### `manage_editor`
Control play mode, active tool, tags, and layers.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `action` | `str` | yes | `play`, `pause`, `stop`, `set_active_tool`, `add_tag`, `remove_tag`, `add_layer`, `remove_layer` |
| `tool_name` | `str` | no | For `set_active_tool` |
| `tag_name` | `str` | no | For tag actions |
| `layer_name` | `str` | no | For layer actions |

```python
manage_editor(action="play")               # Enter play mode
manage_editor(action="pause")              # Pause play mode
manage_editor(action="stop")               # Exit play mode

manage_editor(action="set_active_tool", tool_name="Move")  # Move/Rotate/Scale/etc.

manage_editor(action="add_tag", tag_name="Enemy")
manage_editor(action="remove_tag", tag_name="OldTag")

manage_editor(action="add_layer", layer_name="Projectiles")
manage_editor(action="remove_layer", layer_name="OldLayer")

manage_prefabs(action="open_prefab_stage", prefab_path="Assets/Prefabs/Enemy.prefab")
manage_prefabs(action="save_prefab_stage")   # Save changes in the open prefab stage
manage_prefabs(action="close_prefab_stage")  # Exit prefab editing mode back to main scene

# Package deployment (no confirmation dialog — designed for LLM-driven iteration)
manage_editor(action="deploy_package")     # Copy configured MCPForUnity source into installed package
manage_editor(action="restore_package")    # Revert to pre-deployment backup
```

**Deploy workflow:** Set the source path in MCP for Unity Advanced Settings first. `deploy_package` copies the source into the project's package location, creates a backup, and triggers `AssetDatabase.Refresh`. Follow with `refresh_unity(wait_for_ready=True)` to wait for recompilation.

### execute_menu_item

Execute any Unity menu item.

```python
execute_menu_item(menu_path="File/Save Project")
execute_menu_item(menu_path="GameObject/3D Object/Cube")
execute_menu_item(menu_path="Window/General/Console")
```

### read_console

Read or clear Unity console messages.

```python
read_console(action="get", types=["error"], count=10, include_stacktrace=True)
read_console(action="clear")
```

## Testing Tools

### `run_tests`
Start async Unity test execution.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `mode` | `str` | yes | `EditMode` or `PlayMode` |
| `test_names` | `list[str]` | no | Explicit tests |
| `group_names` | `list[str]` | no | Regex-style group filters |
| `category_names` | `list[str]` | no | NUnit category filter |
| `assembly_names` | `list[str]` | no | Assembly filter |
| `include_failed_tests` | `bool` | no | Return failed tests with details |
| `include_details` | `bool` | no | Return full result details |

```python
result = run_tests(mode="EditMode", test_names=["MyTests.TestA"], include_failed_tests=True)
```

### `get_test_job`
Poll a test job.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `job_id` | `str` | yes | Job from `run_tests` |
| `wait_timeout` | `int` | no | Wait up to N seconds |
| `include_failed_tests` | `bool` | no | Include failed test payload |
| `include_details` | `bool` | no | Include complete details |

## Search Tools

### `find_in_file`
Regex search within a file.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `uri` | `str` | yes | File URI |
| `pattern` | `str` | yes | Regex pattern |
| `max_results` | `int` | no | Match cap |
| `ignore_case` | `bool` | no | Case-insensitive search |

```python
find_in_file(uri="mcpforunity://path/Assets/Scripts/MyScript.cs", pattern="public void \\w+", max_results=200)
```

## Custom Tools

### `execute_custom_tool`
Execute a custom tool declared in the Unity project.

| Parameter | Type | Required | Notes |
| --- | --- | --- | --- |
| `tool_name` | `str` | yes | Name from `mcpforunity://custom-tools` |
| `parameters` | `dict` | no | Tool-specific payload |

```python
execute_custom_tool(
    tool_name="my_custom_tool",
    parameters={"param1": "value", "param2": 42}
)
```

Discover available custom tools via `mcpforunity://custom-tools` resource.

---

## Camera Tools

### manage_camera

Unified camera management (Unity Camera + Cinemachine). Works without Cinemachine using basic Camera; unlocks presets, pipelines, and blending when Cinemachine is installed. Use `ping` to check availability.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `action` | string | Yes | Action to perform (see categories below) |
| `target` | string | Sometimes | Target camera (name, path, or instance ID) |
| `search_method` | string | No | `by_id`, `by_name`, `by_path` |
| `properties` | dict \| string | No | Action-specific parameters |

**Screenshot parameters** (for `screenshot` and `screenshot_multiview` actions):

| Parameter | Type | Description |
|-----------|------|-------------|
| `capture_source` | string | `"game_view"` (default) or `"scene_view"` (editor viewport) |
| `view_target` | string\|int\|list | Target to focus on (GO name/path/ID or [x,y,z]). game_view: aims camera; scene_view: frames viewport |
| `camera` | string | Camera to capture from (defaults to Camera.main). game_view only |
| `include_image` | bool | Return base64 PNG inline (default false) |
| `max_resolution` | int | Downscale cap in px (default 640) |
| `batch` | string | `"surround"` (6 angles) or `"orbit"` (configurable grid). game_view only |
| `view_position` | list[float] | World position [x,y,z] to place camera. game_view only |
| `view_rotation` | list[float] | Euler rotation [x,y,z] (overrides view_target). game_view only |

**Actions by category:**

**Setup:**
- `ping` — Check Cinemachine availability and version
- `ensure_brain` — Ensure CinemachineBrain exists on main camera. Properties: `camera` (target camera), `defaultBlendStyle`, `defaultBlendDuration`
- `get_brain_status` — Get Brain state (active camera, blend status)

**Creation:**
- `create_camera` — Create camera with optional preset. Properties: `name`, `preset` (follow/third_person/freelook/dolly/static/top_down/side_scroller), `follow`, `lookAt`, `priority`, `fieldOfView`. Falls back to basic Camera without Cinemachine.

**Configuration:**
- `set_target` — Set Follow and/or LookAt targets. Properties: `follow`, `lookAt` (GO name/path/ID)
- `set_priority` — Set camera priority for Brain selection. Properties: `priority` (int)
- `set_lens` — Configure lens. Properties: `fieldOfView`, `nearClipPlane`, `farClipPlane`, `orthographicSize`, `dutch`
- `set_body` — Configure Body component (Cinemachine). Properties: `bodyType` (to swap), plus component-specific properties
- `set_aim` — Configure Aim component (Cinemachine). Properties: `aimType` (to swap), plus component-specific properties
- `set_noise` — Configure Noise (Cinemachine). Properties: `amplitudeGain`, `frequencyGain`

**Extensions (Cinemachine):**
- `add_extension` — Add extension. Properties: `extensionType` (CinemachineConfiner2D, CinemachineDeoccluder, CinemachineImpulseListener, CinemachineFollowZoom, CinemachineRecomposer, etc.)
- `remove_extension` — Remove extension by type. Properties: `extensionType`

**Control:**
- `list_cameras` — List all cameras with status
- `set_blend` — Configure default blend on Brain. Properties: `style` (Cut/EaseInOut/Linear/etc.), `duration`
- `force_camera` — Override Brain to use specific camera
- `release_override` — Release camera override

**Capture:**
- `screenshot` — Capture screenshot. Supports `capture_source="game_view"` (default, camera-based) or `"scene_view"` (editor viewport). game_view supports inline base64, batch surround/orbit, positioned capture. scene_view supports `view_target` for framing.
- `screenshot_multiview` — Shorthand for screenshot with batch='surround' and include_image=true.

**Examples:**

```python
# Check Cinemachine availability
manage_camera(action="ping")

# Create a third-person camera following the player
manage_camera(action="create_camera", properties={
    "name": "FollowCam", "preset": "third_person",
    "follow": "Player", "lookAt": "Player", "priority": 20
})

# Ensure Brain exists on main camera
manage_camera(action="ensure_brain")

# Configure body component
manage_camera(action="set_body", target="FollowCam", properties={
    "bodyType": "CinemachineThirdPersonFollow",
    "cameraDistance": 5.0, "shoulderOffset": [0.5, 0.5, 0]
})

# Set aim
manage_camera(action="set_aim", target="FollowCam", properties={
    "aimType": "CinemachineRotationComposer"
})

# Add camera shake
manage_camera(action="set_noise", target="FollowCam", properties={
    "amplitudeGain": 0.5, "frequencyGain": 1.0
})

# Set priority to make this the active camera
manage_camera(action="set_priority", target="FollowCam", properties={"priority": 50})

# Force a specific camera
manage_camera(action="force_camera", target="CinematicCam")

# Release override (return to priority-based selection)
manage_camera(action="release_override")

# Configure blend transitions
manage_camera(action="set_blend", properties={"style": "EaseInOut", "duration": 2.0})

# Add deoccluder extension
manage_camera(action="add_extension", target="FollowCam", properties={
    "extensionType": "CinemachineDeoccluder"
})

# Screenshot from a specific camera (game_view, default)
manage_camera(action="screenshot", camera="FollowCam", include_image=True, max_resolution=512)

# Scene View screenshot (captures editor viewport — gizmos, wireframes, grid)
manage_camera(action="screenshot", capture_source="scene_view", include_image=True)

# Scene View screenshot framed on a specific object
manage_camera(action="screenshot", capture_source="scene_view", view_target="Canvas", include_image=True)

# Multi-view screenshot (6-angle contact sheet)
manage_camera(action="screenshot_multiview", max_resolution=480)

# List all cameras
manage_camera(action="list_cameras")
```

**Tier system:**
- Tier 1 actions (ping, create_camera, set_target, set_lens, set_priority, list_cameras, screenshot, screenshot_multiview) work without Cinemachine — they fall back to basic Unity Camera.
- Tier 2 actions (ensure_brain, get_brain_status, set_body, set_aim, set_noise, add/remove_extension, set_blend, force_camera, release_override) require `com.unity.cinemachine`. If called without Cinemachine, they return an error with a fallback suggestion.

**Resource:** Read `mcpforunity://scene/cameras` for current camera state before modifying.

---

## Graphics Tools

### manage_graphics

Unified rendering and graphics management: volumes/post-processing, light baking, rendering stats, pipeline configuration, and URP renderer features. Requires URP or HDRP for volume/feature actions. Use `ping` to check pipeline status and available features.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `action` | string | Yes | Action to perform (see categories below) |
| `target` | string | Sometimes | Target object name or instance ID |
| `effect` | string | Sometimes | Effect type name (e.g., `Bloom`, `Vignette`) |
| `properties` | dict | No | Action-specific properties to set |
| `parameters` | dict | No | Effect parameter values |
| `settings` | dict | No | Bake or pipeline settings |
| `name` | string | No | Name for created objects |
| `profile_path` | string | No | Asset path for VolumeProfile |
| `path` | string | No | Asset path (for `volume_create_profile`) |
| `position` | list[float] | No | Position [x,y,z] |

**Actions by category:**

**Status:**
- `ping` — Check render pipeline type, available features, and package status

**Volume (require URP/HDRP):**
- `volume_create` — Create a Volume GameObject with optional effects. Properties: `name`, `is_global` (default true), `weight` (0-1), `priority`, `profile_path` (existing profile), `effects` (list of effect defs)
- `volume_add_effect` — Add effect override to a Volume. Params: `target` (Volume GO), `effect` (e.g., "Bloom")
- `volume_set_effect` — Set effect parameters. Params: `target`, `effect`, `parameters` (dict of param name to value)
- `volume_remove_effect` — Remove effect override. Params: `target`, `effect`
- `volume_get_info` — Get Volume details (profile, effects, parameters). Params: `target`
- `volume_set_properties` — Set Volume component properties (weight, priority, isGlobal). Params: `target`, `properties`
- `volume_list_effects` — List all available volume effects for the active pipeline
- `volume_create_profile` — Create a standalone VolumeProfile asset. Params: `path`, `effects` (optional)

**Bake (Edit mode only):**
- `bake_start` — Start lightmap bake. Params: `async_bake` (default true)
- `bake_cancel` — Cancel in-progress bake
- `bake_status` — Check bake progress
- `bake_clear` — Clear baked lightmap data
- `bake_reflection_probe` — Bake a specific reflection probe. Params: `target`
- `bake_get_settings` — Get current Lightmapping settings
- `bake_set_settings` — Set Lightmapping settings. Params: `settings` (dict)
- `bake_create_light_probe_group` — Create a Light Probe Group. Params: `name`, `position`, `grid_size` [x,y,z], `spacing`
- `bake_create_reflection_probe` — Create a Reflection Probe. Params: `name`, `position`, `size` [x,y,z], `resolution`, `mode`, `hdr`, `box_projection`
- `bake_set_probe_positions` — Set Light Probe positions manually. Params: `target`, `positions` (array of [x,y,z])

**Stats:**
- `stats_get` — Get rendering counters (draw calls, batches, triangles, vertices, etc.)
- `stats_list_counters` — List all available ProfilerRecorder counters
- `stats_set_scene_debug` — Set Scene View debug/draw mode. Params: `mode`
- `stats_get_memory` — Get rendering memory usage

**Pipeline:**
- `pipeline_get_info` — Get active render pipeline info (type, quality level, asset paths)
- `pipeline_set_quality` — Switch quality level. Params: `level` (name or index)
- `pipeline_get_settings` — Get pipeline asset settings
- `pipeline_set_settings` — Set pipeline asset settings. Params: `settings` (dict)

**Features (URP only):**
- `feature_list` — List renderer features on the active URP renderer
- `feature_add` — Add a renderer feature. Params: `feature_type`, `name`, `material` (for full-screen effects)
- `feature_remove` — Remove a renderer feature. Params: `index` or `name`
- `feature_configure` — Set feature properties. Params: `index` or `name`, `properties` (dict)
- `feature_toggle` — Enable/disable a feature. Params: `index` or `name`, `active` (bool)
- `feature_reorder` — Reorder features. Params: `order` (list of indices)

**Examples:**

```python
# Check pipeline status
manage_graphics(action="ping")

# Create a global post-processing volume with Bloom and Vignette
manage_graphics(action="volume_create", name="PostProcessing", is_global=True,
    effects=[
        {"type": "Bloom", "parameters": {"intensity": 1.5, "threshold": 0.9}},
        {"type": "Vignette", "parameters": {"intensity": 0.4}}
    ])

# Add an effect to an existing volume
manage_graphics(action="volume_add_effect", target="PostProcessing", effect="ColorAdjustments")

# Configure effect parameters
manage_graphics(action="volume_set_effect", target="PostProcessing",
    effect="ColorAdjustments", parameters={"postExposure": 0.5, "saturation": 10})

# Get volume info
manage_graphics(action="volume_get_info", target="PostProcessing")

# List all available effects for the active pipeline
manage_graphics(action="volume_list_effects")

# Create a VolumeProfile asset
manage_graphics(action="volume_create_profile", path="Assets/Settings/MyProfile.asset",
    effects=[{"type": "Bloom"}, {"type": "Tonemapping"}])

# Start async lightmap bake
manage_graphics(action="bake_start", async_bake=True)

# Check bake progress
manage_graphics(action="bake_status")

# Create a Light Probe Group with a 3x2x3 grid
manage_graphics(action="bake_create_light_probe_group", name="ProbeGrid",
    position=[0, 1, 0], grid_size=[3, 2, 3], spacing=2.0)

# Create a Reflection Probe
manage_graphics(action="bake_create_reflection_probe", name="RoomProbe",
    position=[0, 2, 0], size=[10, 5, 10], resolution=256, hdr=True)

# Get rendering stats
manage_graphics(action="stats_get")

# Get memory usage
manage_graphics(action="stats_get_memory")

# Get pipeline info
manage_graphics(action="pipeline_get_info")

# Switch quality level
manage_graphics(action="pipeline_set_quality", level="High")

# List URP renderer features
manage_graphics(action="feature_list")

# Add a full-screen renderer feature
manage_graphics(action="feature_add", feature_type="FullScreenPassRendererFeature",
    name="NightVision", material="Assets/Materials/NightVision.mat")

# Toggle a feature off
manage_graphics(action="feature_toggle", index=0, active=False)

# Reorder features
manage_graphics(action="feature_reorder", order=[2, 0, 1])
```

**Resources:**
- `mcpforunity://scene/volumes` — Lists all Volume components in the scene with their profiles and effects
- `mcpforunity://rendering/stats` — Current rendering performance counters
- `mcpforunity://pipeline/renderer-features` — URP renderer features on the active renderer

---

## Package Tools

### manage_packages

Manage Unity packages: query, install, remove, embed, and configure registries. Install/remove trigger domain reload.

**Query Actions (read-only):**

| Action | Parameters | Description |
|--------|-----------|-------------|
| `list_packages` | — | List all installed packages (async, returns job_id) |
| `search_packages` | `query` | Search Unity registry by keyword (async, returns job_id) |
| `get_package_info` | `package` | Get details about a specific installed package |
| `list_registries` | — | List all scoped registries (names, URLs, scopes); immediate result |
| `ping` | — | Check package manager availability, Unity version, package count |
| `status` | `job_id` (required for list/search; optional for add/remove/embed) | Poll async job status; omit job_id to poll latest add/remove/embed job |

**Mutating Actions:**

| Action | Parameters | Description |
|--------|-----------|-------------|
| `add_package` | `package` | Install a package (name, name@version, git URL, or file: path) |
| `remove_package` | `package`, `force` (optional) | Remove a package; blocked if dependents exist unless `force=true` |
| `embed_package` | `package` | Copy package to local Packages/ for editing |
| `resolve_packages` | — | Force re-resolution of all packages |
| `add_registry` | `name`, `url`, `scopes` | Add a scoped registry (e.g., OpenUPM) |
| `remove_registry` | `name` or `url` | Remove a scoped registry |

**Input validation:**
- Valid package IDs: `com.unity.inputsystem`, `com.unity.cinemachine@3.1.6`
- Git URLs: allowed with warning ("ensure this is a trusted source")
- `file:` paths: allowed with warning
- Invalid names (uppercase, missing dots): rejected

**Example — List installed packages:**
```python
manage_packages(action="list_packages")
# Returns job_id, then poll:
manage_packages(action="status", job_id="<job_id>")
```

**Example — Search for a package:**
```python
manage_packages(action="search_packages", query="input system")
```

**Example — Install a package:**
```python
manage_packages(action="add_package", package="com.unity.inputsystem")
# Poll until complete:
manage_packages(action="status", job_id="<job_id>")
```

**Example — Remove with dependency check:**
```python
manage_packages(action="remove_package", package="com.unity.modules.ui")
# Error: "Cannot remove: 3 package(s) depend on it: ..."
manage_packages(action="remove_package", package="com.unity.modules.ui", force=True)
# Proceeds anyway
```

**Example — Add OpenUPM registry:**
```python
manage_packages(
    action="add_registry",
    name="OpenUPM",
    url="https://package.openupm.com",
    scopes=["com.cysharp", "com.neuecc"]
)
```

---

## Physics Tools

### `manage_physics`

Manage 3D and 2D physics: settings, collision matrix, materials, joints, queries, validation, and simulation. All actions support `dimension="3d"` (default) or `dimension="2d"` where applicable.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `action` | string | Yes | See action groups below |
| `dimension` | string | No | `"3d"` (default) or `"2d"` |
| `settings` | object | For set_settings | Key-value physics settings dict |
| `layer_a` / `layer_b` | string | For collision matrix | Layer name or index |
| `collide` | bool | For set_collision_matrix | `true` to enable, `false` to disable |
| `name` | string | For create_physics_material | Material asset name |
| `path` | string | No | Asset folder path (create) or asset path (configure) |
| `dynamic_friction` / `static_friction` / `bounciness` | float | No | Material properties (0–1) |
| `friction_combine` / `bounce_combine` | string | No | `Average`, `Minimum`, `Multiply`, `Maximum` |
| `material_path` | string | For assign_physics_material | Path to physics material asset |
| `target` | string | For joints/queries/validate | GameObject name or instance ID |
| `joint_type` | string | For joints | 3D: `fixed`, `hinge`, `spring`, `character`, `configurable`; 2D: `distance`, `fixed`, `friction`, `hinge`, `relative`, `slider`, `spring`, `target`, `wheel` |
| `connected_body` | string | For add_joint | Connected body GameObject |
| `motor` / `limits` / `spring` / `drive` | object | For configure_joint | Joint sub-config objects |
| `properties` | object | For configure_joint/material | Direct property dict |
| `origin` / `direction` | float[] | For raycast | Ray origin and direction `[x,y,z]` or `[x,y]` |
| `max_distance` | float | No | Max raycast distance |
| `shape` | string | For overlap | `sphere`, `box`, `capsule` (3D); `circle`, `box`, `capsule` (2D) |
| `position` | float[] | For overlap | `[x,y,z]` or `[x,y]` |
| `size` | float or float[] | For overlap | Radius (sphere/circle) or half-extents `[x,y,z]` (box) |
| `layer_mask` | string | No | Layer name or int mask for queries |
| `start` / `end` | float[] | For linecast | Start and end points `[x,y,z]` or `[x,y]` |
| `point1` / `point2` | float[] | For shapecast capsule | Capsule endpoints (3D alternative) |
| `height` | float | For shapecast capsule | Capsule height |
| `capsule_direction` | int | For shapecast capsule | 0=X, 1=Y (default), 2=Z |
| `angle` | float | For 2D shapecasts | Rotation angle in degrees |
| `force` | float[] | For apply_force | Force vector `[x,y,z]` or `[x,y]` |
| `force_mode` | string | For apply_force | `Force`, `Impulse`, `Acceleration`, `VelocityChange` (3D); `Force`, `Impulse` (2D) |
| `force_type` | string | For apply_force | `normal` (default) or `explosion` (3D only) |
| `torque` | float[] | For apply_force | Torque `[x,y,z]` (3D) or `[z]` (2D) |
| `explosion_position` | float[] | For apply_force explosion | Explosion center `[x,y,z]` |
| `explosion_radius` | float | For apply_force explosion | Explosion sphere radius |
| `explosion_force` | float | For apply_force explosion | Explosion force magnitude |
| `upwards_modifier` | float | For apply_force explosion | Y-axis offset (default 0) |
| `steps` | int | For simulate_step | Number of steps (1–100) |
| `step_size` | float | No | Step size in seconds (default: `Time.fixedDeltaTime`) |

**Action groups:**

- **Settings:** `ping`, `get_settings`, `set_settings`
- **Collision Matrix:** `get_collision_matrix`, `set_collision_matrix`
- **Materials:** `create_physics_material`, `configure_physics_material`, `assign_physics_material`
- **Joints:** `add_joint`, `configure_joint`, `remove_joint`
- **Queries:** `raycast`, `raycast_all`, `linecast`, `shapecast`, `overlap`
- **Forces:** `apply_force`
- **Rigidbody:** `get_rigidbody`, `configure_rigidbody`
- **Validation:** `validate`
- **Simulation:** `simulate_step`

```python
# Check physics status
manage_physics(action="ping")

# Get/set gravity
manage_physics(action="get_settings", dimension="3d")
manage_physics(action="set_settings", dimension="3d", settings={"gravity": [0, -20, 0]})

# Collision matrix
manage_physics(action="get_collision_matrix")
manage_physics(action="set_collision_matrix", layer_a="Player", layer_b="Enemy", collide=False)

# Create a bouncy physics material and assign it
manage_physics(action="create_physics_material", name="Bouncy", bounciness=0.9, dynamic_friction=0.2)
manage_physics(action="assign_physics_material", target="Ball", material_path="Assets/Physics Materials/Bouncy.physicMaterial")

# Add and configure a hinge joint
manage_physics(action="add_joint", target="Door", joint_type="hinge", connected_body="DoorFrame")
manage_physics(action="configure_joint", target="Door", joint_type="hinge",
               motor={"targetVelocity": 90, "force": 100},
               limits={"min": -90, "max": 0, "bounciness": 0})

# Raycast and overlap
manage_physics(action="raycast", origin=[0, 10, 0], direction=[0, -1, 0], max_distance=50)
manage_physics(action="overlap", shape="sphere", position=[0, 0, 0], size=5.0)

# Validate scene physics setup
manage_physics(action="validate")                    # whole scene
manage_physics(action="validate", target="Player")  # single object

# Multi-hit raycast (returns all hits sorted by distance)
manage_physics(action="raycast_all", origin=[0, 10, 0], direction=[0, -1, 0])

# Linecast (point A to point B)
manage_physics(action="linecast", start=[0, 0, 0], end=[10, 0, 0])

# Shapecast (sphere/box/capsule sweep)
manage_physics(action="shapecast", shape="sphere", origin=[0, 5, 0], direction=[0, -1, 0], size=0.5)
manage_physics(action="shapecast", shape="box", origin=[0, 5, 0], direction=[0, -1, 0], size=[1, 1, 1])

# Apply force (works with simulate_step for edit-mode previewing)
manage_physics(action="apply_force", target="Ball", force=[0, 500, 0], force_mode="Impulse")
manage_physics(action="apply_force", target="Ball", torque=[0, 10, 0])

# Explosion force (3D only)
manage_physics(action="apply_force", target="Crate", force_type="explosion",
               explosion_force=1000, explosion_position=[0, 0, 0], explosion_radius=10)

# Configure rigidbody properties
manage_physics(action="configure_rigidbody", target="Player",
               properties={"mass": 80, "drag": 0.5, "useGravity": True, "collisionDetectionMode": "Continuous"})

# Step physics in edit mode
manage_physics(action="simulate_step", steps=10, step_size=0.02)
```

---

## ProBuilder Tools

### manage_probuilder

Unified tool for ProBuilder mesh operations. Requires `com.unity.probuilder` package. When available, **prefer ProBuilder over primitive GameObjects** for editable geometry, multi-material faces, or complex shapes.

**Parameters:**

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `action` | string | Yes | Action to perform (see categories below) |
| `target` | string | Sometimes | Target GameObject name/path/id |
| `search_method` | string | No | How to find target: `by_id`, `by_name`, `by_path`, `by_tag`, `by_layer` |
| `properties` | dict \| string | No | Action-specific parameters (dict or JSON string) |

**Actions by category:**

**Shape Creation:**
- `create_shape` — Create ProBuilder primitive (shape_type, size, position, rotation, name). 12 types: Cube, Cylinder, Sphere, Plane, Cone, Torus, Pipe, Arch, Stair, CurvedStair, Door, Prism
- `create_poly_shape` — Create from 2D polygon footprint (points, extrudeHeight, flipNormals)

**Mesh Editing:**
- `extrude_faces` — Extrude faces (faceIndices, distance, method: FaceNormal/VertexNormal/IndividualFaces)
- `extrude_edges` — Extrude edges (edgeIndices or edges [{a,b},...], distance, asGroup)
- `bevel_edges` — Bevel edges (edgeIndices or edges [{a,b},...], amount 0-1)
- `subdivide` — Subdivide faces via ConnectElements (faceIndices optional)
- `delete_faces` — Delete faces (faceIndices)
- `bridge_edges` — Bridge two open edges (edgeA, edgeB as {a,b} pairs, allowNonManifold)
- `connect_elements` — Connect edges/faces (edgeIndices or faceIndices)
- `detach_faces` — Detach faces to new object (faceIndices, deleteSourceFaces)
- `flip_normals` — Flip face normals (faceIndices)
- `merge_faces` — Merge faces into one (faceIndices)
- `combine_meshes` — Combine ProBuilder objects (targets list)
- `merge_objects` — Merge objects with auto-convert (targets, name)
- `duplicate_and_flip` — Create double-sided geometry (faceIndices)
- `create_polygon` — Connect existing vertices into a new face (vertexIndices, unordered)

**Vertex Operations:**
- `merge_vertices` — Collapse vertices to single point (vertexIndices, collapseToFirst)
- `weld_vertices` — Weld vertices within proximity radius (vertexIndices, radius)
- `split_vertices` — Split shared vertices (vertexIndices)
- `move_vertices` — Translate vertices (vertexIndices, offset [x,y,z])
- `insert_vertex` — Insert vertex on edge or face (edge {a,b} or faceIndex + point [x,y,z])
- `append_vertices_to_edge` — Insert evenly-spaced points on edges (edgeIndices or edges, count)

**Selection:**
- `select_faces` — Select faces by criteria (direction + tolerance, growFrom + growAngle)

**UV & Materials:**
- `set_face_material` — Assign material to faces (faceIndices, materialPath)
- `set_face_color` — Set vertex color on faces (faceIndices, color [r,g,b,a])
- `set_face_uvs` — Set UV params (faceIndices, scale, offset, rotation, flipU, flipV)

**Query:**
- `get_mesh_info` — Get mesh details with `include` parameter:
  - `"summary"` (default): counts, bounds, materials
  - `"faces"`: + face normals, centers, and direction labels (capped at 100)
  - `"edges"`: + edge vertex pairs with world positions (capped at 200, deduplicated)
  - `"all"`: everything
- `ping` — Check if ProBuilder is available

**Smoothing:**
- `set_smoothing` — Set smoothing group on faces (faceIndices, smoothingGroup: 0=hard, 1+=smooth)
- `auto_smooth` — Auto-assign smoothing groups by angle (angleThreshold: default 30)

**Mesh Utilities:**
- `center_pivot` — Move pivot to mesh bounds center
- `freeze_transform` — Bake transform into vertices, reset transform
- `validate_mesh` — Check mesh health (read-only diagnostics)
- `repair_mesh` — Auto-fix degenerate triangles

**Not Yet Working (known bugs):**
- `set_pivot` — Vertex positions don't persist through mesh rebuild. Use `center_pivot` or Transform positioning instead.
- `convert_to_probuilder` — MeshImporter throws internally. Create shapes natively instead.

**Examples:**

```python
# Check availability
manage_probuilder(action="ping")

# Create a cube
manage_probuilder(action="create_shape", properties={"shape_type": "Cube", "name": "MyCube"})

# Get face info with directions
manage_probuilder(action="get_mesh_info", target="MyCube", properties={"include": "faces"})

# Extrude the top face (find it via direction="top" in get_mesh_info results)
manage_probuilder(action="extrude_faces", target="MyCube",
    properties={"faceIndices": [2], "distance": 1.5})

# Select all upward-facing faces
manage_probuilder(action="select_faces", target="MyCube",
    properties={"direction": "up", "tolerance": 0.7})

# Create double-sided geometry (for room interiors)
manage_probuilder(action="duplicate_and_flip", target="Room",
    properties={"faceIndices": [0, 1, 2, 3, 4, 5]})

# Weld nearby vertices
manage_probuilder(action="weld_vertices", target="MyCube",
    properties={"vertexIndices": [0, 1, 2, 3], "radius": 0.1})

# Auto-smooth
manage_probuilder(action="auto_smooth", target="MyCube", properties={"angleThreshold": 30})

# Cleanup workflow
manage_probuilder(action="center_pivot", target="MyCube")
manage_probuilder(action="validate_mesh", target="MyCube")
```

See also: [ProBuilder Workflow Guide](probuilder-guide.md) for detailed patterns and complex object examples.

---

## Profiler Tools

### `manage_profiler`

Unity Profiler session control, counter reads, memory snapshots, and Frame Debugger. Group: `profiling` (opt-in via `manage_tools`).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `action` | string | Yes | See action groups below |
| `category` | string | For get_counters | Profiler category name (e.g. `Render`, `Scripts`, `Memory`, `Physics`) |
| `counters` | list[str] | No | Specific counter names for get_counters. Omit to read all in category |
| `object_path` | string | For get_object_memory | Scene hierarchy or asset path |
| `log_file` | string | No | Path to `.raw` file for profiler_start recording |
| `enable_callstacks` | bool | No | Enable allocation callstacks for profiler_start |
| `areas` | dict[str, bool] | For profiler_set_areas | Area name to enabled/disabled mapping |
| `snapshot_path` | string | No | Output path for memory_take_snapshot |
| `search_path` | string | No | Search directory for memory_list_snapshots |
| `snapshot_a` | string | For memory_compare_snapshots | First snapshot file path |
| `snapshot_b` | string | For memory_compare_snapshots | Second snapshot file path |
| `page_size` | int | No | Page size for frame_debugger_get_events (default 50) |
| `cursor` | int | No | Cursor offset for frame_debugger_get_events |

**Action groups:**

- **Session:** `profiler_start`, `profiler_stop`, `profiler_status`, `profiler_set_areas`
- **Counters:** `get_frame_timing`, `get_counters`, `get_object_memory`
- **Memory Snapshot:** `memory_take_snapshot`, `memory_list_snapshots`, `memory_compare_snapshots` (requires `com.unity.memoryprofiler`)
- **Frame Debugger:** `frame_debugger_enable`, `frame_debugger_disable`, `frame_debugger_get_events`
- **Utility:** `ping`

```python
# Check profiler availability
manage_profiler(action="ping")

# Start profiling (optionally record to file)
manage_profiler(action="profiler_start")
manage_profiler(action="profiler_start", log_file="Assets/profiler.raw", enable_callstacks=True)

# Check profiler status
manage_profiler(action="profiler_status")

# Toggle profiler areas
manage_profiler(action="profiler_set_areas", areas={"CPU": True, "GPU": True, "Rendering": True, "Memory": False})

# Stop profiling
manage_profiler(action="profiler_stop")

# Read frame timing data (12 fields from FrameTimingManager)
manage_profiler(action="get_frame_timing")

# Read counters by category
manage_profiler(action="get_counters", category="Render")
manage_profiler(action="get_counters", category="Memory", counters=["Total Used Memory", "GC Used Memory"])

# Get memory size of a specific object
manage_profiler(action="get_object_memory", object_path="Player/Mesh")

# Memory snapshots (requires com.unity.memoryprofiler)
manage_profiler(action="memory_take_snapshot")
manage_profiler(action="memory_take_snapshot", snapshot_path="Assets/Snapshots/baseline.snap")
manage_profiler(action="memory_list_snapshots")
manage_profiler(action="memory_compare_snapshots", snapshot_a="Assets/Snapshots/before.snap", snapshot_b="Assets/Snapshots/after.snap")

# Frame Debugger
manage_profiler(action="frame_debugger_enable")
manage_profiler(action="frame_debugger_get_events", page_size=20, cursor=0)
manage_profiler(action="frame_debugger_disable")
```

---

## Docs Tools

Tools for verifying Unity C# APIs and fetching official documentation. Group: `docs`.

### `unity_reflect`

Inspect Unity's live C# API via reflection. **Always use this before writing C# code that references Unity APIs** — LLM training data frequently contains incorrect, outdated, or hallucinated APIs.

Requires Unity connection.

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `action` | string | Yes | `search`, `get_type`, or `get_member` |
| `class_name` | string | For get_type, get_member | Fully qualified or simple C# class name |
| `member_name` | string | For get_member | Method, property, or field name to inspect |
| `query` | string | For search | Search query for type name search |
| `scope` | string | No | Assembly scope for search: `unity`, `packages`, `project`, `all` (default: `unity`) |

**Actions:**

- **`search`**: Search for types by name across loaded assemblies. Returns matching type names.
- **`get_type`**: Get a member summary (names only) for a class. Returns list of methods, properties, fields.
- **`get_member`**: Get full signature detail for one member. Returns parameter types, return type, overloads.

```python
# Search for types matching a name
unity_reflect(action="search", query="NavMesh")
unity_reflect(action="search", query="Camera", scope="all")

# Get all members of a type
unity_reflect(action="get_type", class_name="UnityEngine.AI.NavMeshAgent")

# Get detailed signature for a specific member
unity_reflect(action="get_member", class_name="Physics", member_name="Raycast")
unity_reflect(action="get_member", class_name="NavMeshAgent", member_name="SetDestination")
```

### `unity_docs`

Fetch official Unity documentation from docs.unity3d.com. Returns descriptions, parameter details, code examples, and caveats. Use after `unity_reflect` confirms a type exists.

No Unity connection needed for doc fetching. The `lookup` action with asset-related queries will also search project assets (requires Unity connection).

| Parameter | Type | Required | Description |
|-----------|------|----------|-------------|
| `action` | string | Yes | `get_doc`, `get_manual`, `get_package_doc`, or `lookup` |
| `class_name` | string | For get_doc | Unity class name (e.g., `Physics`, `Transform`) |
| `member_name` | string | No | Method or property name for get_doc |
| `version` | string | No | Unity version (e.g., `6000.0.38f1`). Auto-extracts major.minor. |
| `slug` | string | For get_manual | Manual page slug (e.g., `execution-order`) |
| `package` | string | For get_package_doc, optional for lookup | Package name (e.g., `com.unity.render-pipelines.universal`) |
| `page` | string | For get_package_doc | Package doc page (e.g., `index`, `2d-index`) |
| `pkg_version` | string | For get_package_doc, optional for lookup | Package version major.minor (e.g., `17.0`) |
| `query` | string | For lookup (single) | Single search query |
| `queries` | string | For lookup (batch) | Comma-separated queries (e.g., `Physics.Raycast,NavMeshAgent,Light2D`) |

**Actions:**

- **`get_doc`**: Fetch ScriptReference docs for a class or member. Parses HTML to extract description, signatures, parameters, return type, and code examples.
- **`get_manual`**: Fetch a Unity Manual page by slug. Returns title, sections, and code examples.
- **`get_package_doc`**: Fetch package documentation. Requires package name, page slug, and package version.
- **`lookup`**: Search doc sources in parallel (ScriptReference + Manual; also package docs if `package` + `pkg_version` provided). Supports batch queries. For asset-related queries (shader, material, texture, etc.), also searches project assets via `manage_asset`.

```python
# Fetch ScriptReference for a class
unity_docs(action="get_doc", class_name="Physics")
unity_docs(action="get_doc", class_name="Physics", member_name="Raycast")
unity_docs(action="get_doc", class_name="Transform", version="6000.0.38f1")

# Fetch a Manual page
unity_docs(action="get_manual", slug="execution-order")
unity_docs(action="get_manual", slug="urp/urp-introduction")

# Fetch package documentation
unity_docs(action="get_package_doc", package="com.unity.render-pipelines.universal",
           page="2d-index", pkg_version="17.0")

# Parallel lookup across all sources (single query)
unity_docs(action="lookup", query="Physics.Raycast")

# Batch lookup (multiple queries in one call)
unity_docs(action="lookup", queries="Physics.Raycast,NavMeshAgent,Light2D")

# Lookup with package docs included
unity_docs(action="lookup", query="VolumeProfile",
           package="com.unity.render-pipelines.universal", pkg_version="17.0")
```
