Project Summary: NVIDIA NitroGen AI Agent

1. Project Overview

NitroGen is a unified vision-to-action AI gaming agent developed by NVIDIA.
Input: raw pixels (game screen capture)
Output: simulated controller inputs (via ViGEmBus)
Training method: large-scale imitation learning (40,000+ hours of gameplay)
Capability: zero-shot generalization to unseen games
Limitation: no reward optimization, no goal awareness, no learning during play
Important implication:
NitroGen is not designed to “win” games or beat bosses.
It imitates plausible human gameplay behavior, not optimal survival strategies.

2. System Environment (Verified Working)
Hardware:
GPU: NVIDIA RTX 5080 (16 GB VRAM)
CPU: 24-core processor
RAM: 32 GB

Software:
OS: Windows 11
Shell: Git Bash (run as Administrator)
Python: 3.13
NVIDIA Driver: 591.59
CUDA Toolkit: 13.1
PyTorch: CUDA-enabled build (cu130)
ViGEmBus: v1.22.0 (virtual controller driver)

3. Installation & Setup Summary
Clone NitroGen repository into a dev directory (e.g. ~/dev/NitroGen)
Create Python virtual environment (.venv)
Install NitroGen as a local editable package:
	pip install -e .
Install CUDA-enabled PyTorch from NVIDIA index
Download model weights:
	hf download nvidia/NitroGen ng.pt
Install ViGEmBus driver and reboot

4. Core Commands (Git Bash)
Purpose					Command
Activate venv			source .venv/Scripts/activate
Verify CUDA				python -c "import torch; print(torch.cuda.is_available())"
Reset controller bus	python -c "import vgamepad as vg; pad = vg.VX360Gamepad(); del pad"
Start server			python scripts/serve.py ng.pt
Start agent				python scripts/play.py --process 'b1-Win64-Shipping.exe'

5. Known Errors & Resolutions
Error								Cause							Resolution
ModuleNotFoundError: nitrogen		Local package not installed		pip install -e .
Torch CPU-only						Wrong PyTorch wheel				Reinstall with cu130 index
FileNotFoundError: ng.pt			Missing weights					Download from Hugging Face
Could not connect to ViGEmBus		Zombie virtual controller		Restart / admin / reset bus

6. Best Practices for Operation
The Graceful Exit: To stop the program, press Ctrl + C exactly once in the terminal and wait 2 seconds for the cleanup script to finish before closing the window.
Game Settings: Always run games (like Black Myth: Wukong) in Windowed Mode at 1920x1080 for optimal AI vision.
Admin Rights: Always launch Git Bash as Administrator to give the AI permission to create/remove virtual hardware.

7. Operational Script (Git Bash)
A single startup script is used to ensure consistency and cleanup.
start_nitrogen.sh:
Activates virtual environment
Confirms CUDA GPU visibility
Resets ViGEm virtual controller bus
Validates model weights
Starts inference server

#!/usr/bin/env bash
# NitroGen launcher for Git Bash (Windows)
# Runs: venv activate -> GPU check -> ViGEm/vgamepad reset -> weights check -> serve

set -euo pipefail

echo "=== Phase 0: Activate Virtual Environment (Git Bash) ==="

# Git Bash path to venv activate is usually: .venv/Scripts/activate
# (If you created the venv in a different folder, update VENV_DIR)
VENV_DIR=".venv"
ACTIVATE_PATH="${VENV_DIR}/Scripts/activate"

if [ ! -f "$ACTIVATE_PATH" ]; then
  echo "❌ Cannot find venv activate script at: $ACTIVATE_PATH"
  echo "   Expected layout: .venv/Scripts/activate"
  exit 1
fi

# shellcheck disable=SC1090
source "$ACTIVATE_PATH"
echo "✅ Virtual environment activated"

echo "=== Phase 1: GPU Hardware Handshake ==="
python - <<'PY'
import torch
if not torch.cuda.is_available():
    raise SystemExit("❌ ERROR: CUDA not available. Check NVIDIA driver + CUDA-enabled PyTorch install.")
print(f"✅ GPU Detected: {torch.cuda.get_device_name(0)}")
print(f"   CUDA Version (torch): {torch.version.cuda}")
PY

echo "=== Phase 2: Reset Virtual Controller Bus (vgamepad) ==="
python - <<'PY'
try:
    import vgamepad as vg
    pad = vg.VX360Gamepad()
    del pad
    print("✅ Virtual Bus Reset")
except Exception as e:
    raise SystemExit(f"❌ Bus Reset Failed: {e}")
PY

echo "=== Phase 3: Validate Model Weights ==="
WEIGHTS="ng.pt"
if [ ! -f "$WEIGHTS" ]; then
  echo "❌ Weights file not found: $WEIGHTS"
  echo "   Put it in the repo root, or change WEIGHTS to the correct path."
  exit 1
fi
echo "✅ Found weights: $WEIGHTS"

echo "=== Phase 4: Start NitroGen Inference Server ==="
echo "🧠 Loading model (this may take a moment)..."
python scripts/serve.py "$WEIGHTS"


7. Game Under Test: Black Myth: Wukong (Steam)
Game Settings Used:
Preset: LOW
Resolution: 1920×1080
Display mode: Windowed
Ray Tracing: OFF
DLSS: ON
Super Resolution: 40
Frame Rate Cap: 60
Motion Blur: OFF
V-Sync: OFF

Important Discovery:
Frame Generation ON caused severe agent instability.
Turning Frame Generation OFF:
Made gameplay slightly smoother
Reduced perception/action mismatch
Did not eliminate lag entirely

8. Performance Diagnosis (Critical Insight)
Observed During Lag:
CPU usage spikes to ~57%
GPU usage stays ~10–12%
Game itself is not GPU-bound

Conclusion:
NitroGen is CPU-bound, not GPU-bound.
The bottleneck is:
Screen capture
Image preprocessing
Python execution
Controller event dispatch

Not rendering.

Frame Generation worsens this by:
Creating synthetic frames
Increasing temporal mismatch
Breaking timing for vision-based agents

9. Behavioral Observations
What Works:
Character movement
Ability usage
Exploration
Basic combat loops

What Fails (Expected):
Boss fights
Precise dodge/parry timing
Survival-optimized behavior

Important realization:
Dying easily is not a bug — it is a known limitation of imitation-only agents.

10. Current Status (End State)
NitroGen setup is correct and functional
Virtual controller works
Inference server stable
Agent controls Wukong successfully
Lag persists due to CPU-bound perception loop
Disabling Frame Generation helps but does not fully solve latency
NitroGen is not expected to beat difficult enemies without fine-tuning or RL

11. Next Possible Directions (Not Yet Implemented)
Reduce agent input resolution (720p / 540p)
Lower decision frequency (20–30 Hz instead of per frame)
Profile Python hot paths
Experiment with simpler games
Explore fine-tuning / reinforcement learning approaches

12. Key Mental Model (For Future AI Assistants)
NitroGen ≠ Auto-win bot
NitroGen = Real-time vision imitation system

Treat it as:
A research agent
A generalist behavior demonstrator
A platform for experimentation, not optimization