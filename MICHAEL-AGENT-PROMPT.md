# Prompt for Michael's agent

You are helping Michael assess a recently purchased used RTX 3090 that repeatedly crashes. It is reported to have CeX warranty, a missing screw and signs of physical impact. Confirm these reports, the return deadline, the actual OS and PC specification. The outcome is a supported return/warranty-review, keep-candidate, platform-suspect or inconclusive decision. This is warranty qualification, not repair. Use the existing tools on Michael's PC where suitable. Do not build a new diagnostic platform.

The following lessons come from Adam's retained multi-3090 scripts, logs and original Claude/Codex session records. They save repeated discovery; they do not establish anything about Michael's card. No internal repository, custom runner or access to Adam's machines is required.

## Start with the warranty and physical condition

Ask for external photos and the exact crash description. If cooler attachment, PCB/connectors or safe operation are in doubt, stop and contact the seller with the photos and existing symptoms. Do not require a load test to justify seller review. Do not remove the cooler, repaste, repad, flash VBIOS/firmware or perform invasive repairs. Do not repeatedly crash an apparently defective card to obtain more evidence.

## Baseline, stock state and durable capture

Record OS, GPU board/model, UUID, PCI address, driver/VBIOS, motherboard, CPU, RAM, PSU model, power connectors/cable arrangement, slot/riser and the application that fails. Read NVIDIA inventory first:

```text
nvidia-smi --query-gpu=uuid,pci.bus_id,name,driver_version --format=csv,noheader,nounits
```

Replace GPU_UUID below with the actual selected UUID, without inventing a numeric index:

```text
nvidia-smi --query --xml-format --id GPU_UUID
nvidia-smi --query-gpu=uuid,pci.bus_id,temperature.gpu,utilization.gpu,power.draw,power.limit,clocks.sm,clocks.mem,memory.used,memory.total,fan.speed,pstate --format=csv,noheader,nounits --id GPU_UUID
```

Keep raw output/errors and UTC timestamps. Poll about every two seconds during tests; write each sample promptly so a crash does not erase the evidence. Capture fan, PCIe link/replay and clock-event fields if the installed driver supports them. Hotspot and memory-junction readings may be unavailable; core temperature is not a substitute. Unsupported fields remain unknown. Preserve NVIDIA-query failure separately from a card actually absent from PCIe inventory.

Verify stock settings before load: record current/default power cap, clocks and any tuning application or startup policy. Ask Michael to confirm any settings reset. Our own startup policies silently restored caps/clock locks, and GPU indices changed between inventories. Our 220/280/350 W settings and 1350 MHz locks are fleet settings, not Michael's factory baseline. Do not copy them. Do not silently change drivers, BIOS policies, services or power settings.

## Crash signature and OS events

Distinguish application exit, artifacts, driver reset/TDR, black screen, bugcheck/reboot, NVIDIA-query loss and complete PCIe non-enumeration. Preserve time-bounded Windows System events from WHEA, Display, nvlddmkm and restart/bugcheck providers, or Linux kernel Xid/AER/PCIe events. Capture events even if NVIDIA inventory fails. Record inaccessible logs as a gap; no events in an unreadable window is not an all-clear.

Align events with the telemetry and the actual failing workload. A single Xid, shutdown event or counter increment does not determine root cause. Our incident recorder saw a thermal-counter increment with sampled cores at most 39 C and no active thermal flags. Counter growth alone does not prove overheating. Preserve pre-crash data; after reboot, make a new observation rather than automatically repeating the failed load.

## Safe staged tests, only if physical condition and stock are cleared

Choose short, bounded stages with Michael's agreement. Start with a brief VRAM check, extend to roughly six minutes if clean, and repeat only when it resolves an uncertainty. Then reproduce the actual failing game/3D/compute application with telemetry. Record exact tool version, binary source/hash, target, allocation, duration, settings and whether meaningful load actually ran.

We used memtest_vulkan: a retained Linux 0.6.0 source-build log reported no errors through 7 min 21 sec; Windows 0.5.0 reported 5,612 iterations with no errors. These are dated precedents, not current download instructions. A source-build version was once mistaken for a nonexistent release tag. Check the real official asset and its documented syntax. An opaque tester's `--help` previously launched a test, so do not assume help flags are harmless. Vulkan device numbers are not NVIDIA indices; confirm the selected test's actual device/PCI identity. A device menu alone is not confirmation. Initialization failures indicate a tool/driver compatibility gap, not proven defective VRAM.

Stop on a memory mismatch, crash, lost target/telemetry, abnormal physical condition or protective temperature limit. Our old soak used 82 C core as an exposure guard; this is not a universal defect threshold. Use suitable documented limits for the actual board and available sensors. Preserve partial output and label incomplete stages. Stop only the owned tester/process family and verify it has stopped before another stage.

Avoid our combined-load traps: 48 CPU workers starved the GPU feeder, dropping GPU draw to 32 W/121 MHz. Reducing to 40 of 48 workers still left an impossible CPU-utilization-above-90% pass rule. A PC that does not reboot under an inadequately loaded test has not passed PSU qualification. Our simple SGEMM burner omitted CUDA/cuBLAS error checks and result comparison; it was heat generation, not memory-integrity proof. We have no accepted gaming/3D diagnostic precedent to substitute for Michael's actual application.

## Repeatability, power comparison and isolation

Repeat the same meaningful workload with the same card identity and stock state when safe. A lower-power comparison is optional after the stock result, with Michael's consent, one variable changed and the original state recorded/restored. Improvement under reduced power does not identify the PSU or exonerate the GPU. Do not raise limits or import our old sweep script: it hardcoded an index and restored our 280 W/1350 MHz settings.

Our retained examples are references only: a Linux 600-second solo soak ended around 69 C/279 W; dual soaks had cores around 66–70 C. Windows VRAM load recorded about 74 C/343 W/97% utilization and Gen4 x16, without memory-junction data. Different boards, cooling and workloads prevent universal pass thresholds. An empty IPMI rail column in our logs was a failed read, not a measured good 12 V rail. Software readings cannot establish transient PSU quality.

If safe and useful, have Michael perform powered-off external checks or controlled substitutions: suspect GPU in a known-good direct slot/system, and known-good GPU in the suspect platform path. Change one variable at a time and record cabling/riser/slot details. Our missing-device/reseat recovery was a clue, not a controlled fault-follows-card result. We have no complete cross-test matrix for Michael; unavailable controls remain a limitation.

## Finish with one compact evidence-backed report

Include baseline/physical condition, a stage table, exact crash timeline, versions/settings, raw telemetry and selected OS/tool output, any comparisons, restoration and gaps. Keep one local bundle where practical. Do not upload personal logs automatically. Empty logs, unknown sensors, unconfirmed targets and interrupted tests cannot become PASS.

- **RETURN / WARRANTY REVIEW:** physical concerns justify seller review, or a controlled stock fault is reproduced. State whether it follows the card under a known-good comparison; attribution can remain unresolved without delaying seller contact.
- **PLATFORM SUSPECT:** a controlled comparison makes the failure follow the platform path while the card works in a known-good path. A successful reseat alone is insufficient.
- **KEEP CANDIDATE:** physical and stock checks are satisfied; meaningful VRAM and actual failing-workload repetitions complete with no reproduced fault in readable evidence. State the tested window and limits; do not claim lifetime reliability or absence of mining wear.
- **INCONCLUSIVE:** missing controls, unsupported evidence, uncertain stock/target, failed tool initialization or incomplete load prevent a supported decision. Give the smallest useful next action; do not extend testing indefinitely.

Use the recovered lessons above directly. Michael's actual hardware, sensors, crash evidence and controlled comparisons are the remaining unknowns. None of Adam's historical results is a verdict on this card.
