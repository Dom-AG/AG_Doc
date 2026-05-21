# Rez Pipeline Integration

tags: #pipeline #rez #architecture

---

## Rez is a System Tool

Rez is installed globally — NOT a pipeline component.
`Packages/rez/` contains package **definitions** that Rez reads.

```
DO NOT create Packages/rez/rez/
DO NOT put Rez installation in the pipeline repo
DO NOT call Rez from UI — always through Launcher
```

---

## NAS Paths (replace /studio placeholder)

```python
# rezconfig.py
local_packages_path  = "~/.rez/packages"
release_packages_path = "/mnt/aslon/04_Tools/REZ/packages"
packages_path = [
    "~/.rez/packages",                      # local dev builds, checked first
    "/mnt/aslon/04_Tools/REZ/packages",     # released packages
]
```

Windows equivalent: `\\aslon\04_Tools\REZ\packages` or mapped drive.
Abstract behind config — never hardcode in package definitions.

---

## Package Structure

```
/mnt/aslon/04_Tools/REZ/packages/
  houdini/
    20.5.584/
      package.py        # points to local /opt/hfs20.5.584
  maya/
    2025.0/
      package.py
  ag_core/
    1.0/
      package.py        # shared pipeline Python
  ag_houdini/
    1.0/
      package.py        # Houdini HDAs + pipeline scripts
  ocio/
    2.3/
      package.py        # ACES config path
  python/
    3.11/
      package.py
```

---

## Package Template

All `ag_*` packages follow this structure:

```python
name = "ag_houdini"
version = "1.0.0"
requires = ["houdini-20+", "ag_core-1+", "python-3.11+"]

def commands():
    env.PYTHONPATH.prepend("{root}/python")
    env.HOUDINI_PATH.prepend("{root}/hda")
    env.HOUDINI_PATH.append("&")   # CRITICAL — keeps Houdini default path
```

DCC packages point to local install:

```python
name = "houdini"
version = "20.5.584"
requires = ["python-3.11"]

def commands():
    env.PATH.prepend("/opt/hfs20.5.584/bin")
    env.HFS = "/opt/hfs20.5.584"
```

---

## Per-Project Environment Schema

```python
class DccEnvironment(BaseModel):
    packages: list[str]    # ["houdini-20.5.584", "ag_houdini-1.0", "ocio-2.3"]
    executable: str        # "houdini", "maya", "nuke"

class Project(BaseModel):
    base_packages: list[str]               # ["ag_core-1.0", "python-3.11"]
    dcc_envs: dict[str, DccEnvironment]   # keyed by DCC name
```

Stored as JSONB in PostgreSQL. Easy to edit per project, no foreign key explosion.

---

## Rez Resolution (Python API — not subprocess)

```python
from rez.resolved_context import ResolvedContext

def resolve(packages: list[str]) -> dict[str, str]:
    ctx = ResolvedContext(packages)
    return ctx.get_environ()

def launch_dcc(project: Project, dcc: str):
    dcc_env = project.dcc_envs[dcc]
    all_packages = project.base_packages + dcc_env.packages
    rez_env = resolve(all_packages)

    merged = os.environ.copy()
    merged.update(rez_env)
    merged.update({
        "PROJECT_ROOT": project.root_path,
        "PROJECT_CODE": project.code,
        "DCC": dcc,
    })

    subprocess.Popen([dcc_env.executable], env=merged)
```

No subprocess wrapping of `rez-env` — Rez runs in-process, DCC as direct child.

---

## Build/Release Workflow

```
Developer machine          NAS (release)              Artist machines
─────────────────          ─────────────              ───────────────
rez-build --install        rez-release pushes here    rez-env pulls here
(local ~/.rez test)        (single source of truth)   (local cache auto)
```

Artists never run `rez-build`. Only consume released packages.

---

## Storage: What Lives Where

| Content | Location | Reason |
|---|---|---|
| DCC binaries | Local `/opt/` | Shared libs, network = crashes |
| Pipeline scripts/HDAs | NAS via `ag_houdini` | Update once, all artists get it |
| Rez package definitions | NAS release path | Versioned, distributed by Rez |
| Models (ComfyUI etc.) | NAS + local session cache | Too large for permanent local |
| Project files | NAS `/mnt/aslon/projects/` | Shared, referenced by path |

---

## Sprint 1 Tasks (Foundation)

- [ ] Write `rezconfig.py` with correct NAS paths
- [ ] Write working `houdini/20.5.584/package.py` pointing at `/opt/hfs20.5.584`
- [ ] Fix `Launcher/plugins/environment.py` — replace subprocess with Python API
- [ ] Verify: `rez-env houdini-20.5.584 -- houdini` launches correctly

Not in Sprint 1: ag_core, ag_houdini, FastAPI integration, model sync.

---

## Houdini PySide6 Notes

- Houdini 21.0 ships Qt 6.5.3 as main build
- PySide6 included with Houdini — no separate install
- Always run on main thread (shelf tool, Python Panel, scene module)
- Never create `QApplication()` — use `QApplication.instance()`
- Store widget refs in `hou.session` or parent to `hou.ui.mainQtWindow()`
- Compatibility shim for H20.5/H21.0:

```python
try:
    from PySide6 import QtWidgets, QtCore, QtGui
except ImportError:
    from PySide2 import QtWidgets, QtCore, QtGui
```

## Related
- [[Architecture Overview]]
- [[Data Models]]
