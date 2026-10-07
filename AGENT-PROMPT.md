# GPU logging and debugging prompt

You are helping the user investigate repeated RTX 3090 crashes. Confirm the actual OS, PC specification and exact failing application. Collect enough technical evidence to distinguish application/tool failure, driver reset, GPU instability and platform-path problems. Use existing tools where suitable. Keep the investigation bounded; do not build a new diagnostic platform.

The lessons below were recovered from earlier multi-3090 scripts, retained logs and original agent session records. They save repeated discovery. They are historical observations, not results for the user's GPU. This prompt requires no custom runner or access to the original machines.

## 1. Baseline and stock-state capture

Record OS, GPU board/model, UUID, PCI address, driver/VBIOS, motherboard, CPU, RAM, PSU model, power-connector/cable arrangement, slot/riser and the application that fails. Capture the exact symptom and its timestamp: application exit/error, artifacts, driver reset/TDR, black screen, bugcheck/reboot, NVIDIA-query loss or complete PCIe non-enumeration.

Read NVIDIA inventory first:

```text
nvidia-smi --query-gpu=uuid,pci.bus_id,name,driver_version --format=csv,noheader,nounits
```

Replace GPU_UUID below with the selected UUID. Do not assume a numeric index stays attached to the same card:

```text
nvidia-smi --query --xml-format --id GPU_UUID
nvidia-smi --query-gpu=uuid,pci.bus_id,temperature.gpu,utilization.gpu,power.draw,power.limit,clocks.sm,clocks.mem,memory.used,memory.total,fan.speed,pstate --format=csv,noheader,nounits --id GPU_UUID
```

Record current/default power cap, clocks, tuning applications and startup policies before interpreting a test. Earlier startup policies silently restored caps/clock locks; GPU indices changed between inventories. Historical 220/280/350 W caps and 1350 MHz locks are fleet settings, not the user's factory baseline. Verify stock state rather than copying those settings. Record any user-approved change and the actual restoration.

## 2. Continuous logging and crash reconstruction

Preserve raw output/errors with UTC timestamps. Poll about every two seconds during tests and write each sample promptly so a crash does not erase the preceding evidence. Include GPU identity, core temperature, utilization, draw/cap, clocks, memory allocation, fan and P-state. Capture PCIe link/replay and clock-event fields if the installed driver supports them. Hotspot and memory-junction readings may be unavailable; core temperature is not a substitute. Unsupported fields remain unknown.

Capture time-bounded Windows System events from WHEA, Display, nvlddmkm and restart/bugcheck providers, or Linux kernel Xid/AER/PCIe events. Capture events even if NVIDIA inventory fails. Preserve NVIDIA-query failure separately from a card actually absent from PCIe inventory. An unreadable event window is a gap, not an all-clear.

Align events with telemetry and actual workload start/stop times. A single Xid, shutdown event or counter increment does not establish root cause. An earlier incident recorder saw a thermal-counter increment while sampled cores stayed at most 39 C and active thermal flags were false. Counter growth alone does not establish overheating. Earlier stale four-card telemetry was also joined to a five-card inventory, producing wrong identities and repeated recorder restarts; verify topology and identity freshness.

Retain partial logs when the process or host fails. A missing final result alone does not prove a process stopped. Check the original process or changed boot identity. After reboot, make a new observation rather than automatically repeating the failed workload.

## 3. Staged VRAM and actual-workload reproduction

Start with a short no-load capture. Then select bounded tests with the user's agreement and verified target/settings. Start with a brief VRAM stage; extend to roughly six minutes if clean, and repeat only when useful. Reproduce the actual failing game/3D/compute application while logging. Record exact tool version, binary source/hash, target, allocation, duration and settings, with evidence that meaningful load actually ran.

Earlier memtest_vulkan logs provide useful precedents: a Linux 0.6.0 source build reported no errors through 7 min 21 sec; Windows 0.5.0 reported 5,612 iterations with no errors. These are dated versions, not current download instructions. A source-build version was once mistaken for a nonexistent release tag. Check the real official asset and its documented syntax.

An opaque tester's `--help` previously launched a test. Do not assume help flags are harmless. Vulkan device numbers are not NVIDIA indices; confirm the selected test's actual device/PCI identity. A menu alone is not target confirmation. Initialization failures are tool/driver compatibility evidence, not demonstrated VRAM mismatches. A short “NO ERRORS” message does not prove the intended stage completed.

Stop the current test on a memory mismatch, crash, lost target/telemetry or a protective temperature limit. Preserve output and mark incomplete stages. Earlier soaks used 82 C core as an exposure guard, not a universal diagnostic threshold. Use suitable documented limits for the actual board and available sensors. Stop only the owned tester/process family and verify cleanup before another stage.

Avoid earlier combined-load traps: 48 CPU workers starved the GPU feeder, dropping GPU draw to 32 W/121 MHz. A revised 40-of-48-worker test still required CPU utilization above 90%, making its gate unreachable. No reboot under inadequate load does not qualify the PSU. An earlier SGEMM burner omitted CUDA/cuBLAS error checks and result comparison; it generated heat but did not verify memory integrity. There is no accepted gaming/3D precedent in the recovered material to substitute for the user's actual failing application.

## 4. Repeatability, power comparison and platform isolation

Repeat the same meaningful workload with the same card identity and verified settings when it resolves uncertainty. A reduced-power comparison is optional after recording the stock result, with the user's consent, one variable changed and the predecessor state recorded/restored. Improvement under reduced power does not by itself identify the PSU or GPU as the cause. Do not import the earlier sweep script: it hardcoded an index and restored fleet-specific 280 W/1350 MHz settings.

Historical measurements are references only: a Linux 600-second solo soak ended around 69 C/279 W; dual soaks had cores around 66–70 C. Windows VRAM load recorded about 74 C/343 W/97% utilization and Gen4 x16, without memory-junction data. Different boards, cooling and workloads prevent universal pass thresholds. An empty IPMI rail column was a failed read, not a measured good 12 V rail. Software readings cannot establish transient PSU quality.

If controlled comparisons are available, have the user perform powered-off substitutions: the GPU in a known-good direct slot/system, and a known-good GPU in the original platform path. Change one variable at a time and record slot/riser/cabling details. Earlier missing-device/reseat recovery was a clue, not a complete fault-follows-card result. Missing controls stay explicit.

## 5. Technical report and next action

Keep one local evidence bundle where practical: baseline, settings, raw telemetry, time-bounded OS events, tool output, exact versions, stage status and a concise timeline. Do not upload personal logs automatically. Empty logs, unsupported sensors, unconfirmed targets and interrupted tests cannot become a pass.

Report what was reproduced, under which settings and workload, and which comparison supports each hypothesis. Distinguish an application/tool problem, driver reset, GPU-associated fault and platform-path fault only when the evidence supports it. State when attribution is inconclusive. Clean tests establish only the observed window and workloads, not general reliability.

Finish with the smallest useful next debugging step. Do not extend testing indefinitely. The user's actual hardware, sensor support, crash evidence and controlled comparisons are the remaining unknowns; none of the historical results is a verdict on this GPU.
