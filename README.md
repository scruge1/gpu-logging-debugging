# GPU logging and debugging

[Read or copy the standalone agent prompt](AGENT-PROMPT.md).

Give the complete prompt to the agent investigating the user's RTX 3090 crashes. It adds concrete NVIDIA commands and lessons recovered from earlier multi-GPU work: durable telemetry, crash and OS-event capture, stock-state verification, VRAM device/tool traps, workload coverage, repeatability, power comparison and controlled platform isolation.

Historical measurements are observations, not universal pass thresholds or results for the user's card. Missing evidence stays explicit. The output is a compact technical report with the smallest useful next debugging step.

Documentation only: no custom runner, service, GPU binary, private logs or access to the original machines is required.
