# The-Blockheads-Change-Any-World-Size

This is a dynamic hook written in C/Objective-C designed to modify the world width (`worldWidthMacro`) of **The Blockheads** server on Linux.

It intercepts the `initWorld` and `loadWorld` methods using `LD_PRELOAD`, allowing you to force custom world sizes (like 4x, 16x, or custom values) without permanently modifying the server binary.

## Prerequisites

Before compiling, ensure you have the necessary development tools and libraries installed.

**Debian/Ubuntu:**

```bash
sudo apt-get update
sudo apt-get install -y gcc gobjc libobjc-12-dev libgnustep-base-dev build-essential

```

## Installation

### 1. Download the Source Code

You can download the raw C file directly or clone this repository.

```bash
wget https://raw.githubusercontent.com/noxthewildshadow/The-Blockheads-Change-Any-World-Size/main/change_world_size.c

```

### 2. Compile the Hook

Use `gcc` to compile the source code into a shared library (`.so`).

```bash
gcc -shared -fPIC -o change_world_size.so change_world_size.c -ldl -lobjc -lpthread

```

* **-shared -fPIC**: Creates a Position Independent executable (Shared Library).
* **-ldl -lobjc -lpthread**: Links necessary libraries for dynamic loading and Objective-C runtime.

---

## Configuration (Environment Variables)

The hook is controlled entirely via Environment Variables. You must export **ONE** of the following variables before starting the server.

### Option A: `BH_MUL` (Multiplier Mode)

Use this for standard world sizes based on the game's default macro width.
**Note:** The base unit is **512 blocks** (which equals a 1x / Normal world).

| Variable | Calculation | Total Width | Description |
| --- | --- | --- | --- |
| `BH_MUL=1` | 512 * 1 | **512** | Normal World (1x) |
| `BH_MUL=4` | 512 * 4 | **2048** | 4x World |
| `BH_MUL=16` | 512 * 16 | **8192** | 16x World |

**Example:**

```bash
export BH_MUL=4

```

### Option B: `BH_RAW` (Raw Mode)

Use this if you need a specific, non-standard custom size. This overrides `BH_MUL`.

**Example:**

```bash
export BH_RAW=1024
# Sets the world width to exactly 1024 macro blocks.

```

---

## Usage

To use the hook, you must load the `.so` file using `LD_PRELOAD` right before executing the server binary.

### Command Syntax

```bash
LD_PRELOAD=./change_world_size.so ./blockheads_server171 --load <WORLD_ID> -p <PORT>

```

### Full Example (Creating/Loading a 16x World)

```bash
# 1. Export the desired size variable
export BH_MUL=16

# 2. Run the server with the hook
LD_PRELOAD=./change_world_size.so ./blockheads_server171 --load 631dce86b90f835d313d94faa0bb63d5 -p 11111

```

---

## ⚠️ Important Notes

1. **Memory Overwrite:** This hook uses `objc_msgSend` interception. If the game tries to reset the world size during initialization, the hook detects the discrepancy and forces the `worldWidthMacro` integer in memory to match your requested size.
2. **Existing Worlds:** If you load an existing world with a *different* size than it was created with, you may experience terrain generation issues or "void" chunks at the edges. It is recommended to use this primarily when generating **new** worlds.
3. **Passive Mode:** If neither `BH_MUL` nor `BH_RAW` is set (or set to 0), the hook will remain passive and will not alter the world size.
