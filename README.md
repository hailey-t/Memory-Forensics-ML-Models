# Memory Forensics of PyTorch Models on Linux

This project demonstrates **end-to-end forensic recovery of a deep learning model from Linux system memory**.  
Using **PyTorch, TorchScript, C++, GDB, LiME, and Volatility 3**, we show how a trained convolutional neural network (CNN) can be identified, analyzed, and reconstructed directly from RAM during execution.

The work bridges **machine learning systems**, **reverse engineering**, and **memory forensics**, highlighting risks to ML intellectual property and opportunities for detection in malware and incident response scenarios.

---

## Motivation

Modern malware increasingly embeds or loads machine learning models for tasks such as:
- classification and evasion
- anomaly detection
- automated decision-making

At the same time, production ML models represent **high-value intellectual property**.  
This project explores a critical question:

> *Can a deployed PyTorch model be recovered from memory using standard forensic techniques?*

The answer is **yes** — and this repository documents how.

---

## Project Overview

The workflow consists of eight major stages:

1. **Train a custom CNN** on CIFAR-10 using PyTorch (no prebuilt architectures).
2. **Export the model to TorchScript** for C++ inference.
3. **Compile a C++ inference binary with full debug symbols**.
4. **Dynamically inspect model execution using GDB**, capturing runtime addresses.
5. **Acquire a full physical memory image** using LiME.
6. **Analyze the memory image with Volatility 3**, identifying process VMAs.
7. **Correlate GDB runtime addresses with forensic memory analysis**.
8. **Recover the ML model** (architecture, weights, and code) using a custom Volatility plugin.

---

## Repository Structure
```
memory-forensics-ml-models/
│
├── training/ # PyTorch CNN training code
│ ├── model.py
│ ├── train_cnn.py
│ └── requirements.txt
│
├── torchscript/ # TorchScript export artifacts
│ ├── export_model.py
│ └── cifar_cnn.ts.pt
│
├── cpp_inference/ # C++ TorchScript inference binary
│ ├── CMakeLists.txt
│ └── main.cpp
│
├── debugging/ # GDB analysis artifacts
│ ├── gdb_notes.md
│ └── address_map.txt
│
├── memory_acquisition/ # LiME acquisition notes
│ └── lime_instructions.md
│
├── volatility/
│ ├── plugins/
│ │ └── linux_torchscript_recover.py
│ └── analysis_notes.md
│
├── screenshots/ # Evidence used in write-up
│ ├── gdb_breakpoint.png
│ ├── pslist.png
│ └── proc_maps.png
│
└── writeup/
└── blog_post.md
```

---

## Model Training

- **Dataset:** CIFAR-10  
- **Architecture:** Custom CNN implemented from scratch using `torch.nn.Module`
- **Split:** 80% training / 10% validation / 10% test
- **Framework:** PyTorch (latest stable)
- **Python:** ≥ 3.10

Two artifacts are produced:
- `weights.pt` – weights only (for Python validation)
- `cifar_cnn.ts.pt` – TorchScript archive used for C++ inference and forensic recovery

---

## TorchScript & C++ Inference

The trained model is exported using TorchScript and loaded into a C++ executable via LibTorch.  
The binary is compiled with **full debug symbols (`-g3 -O0`)** to enable precise runtime inspection.

During inference, execution pauses at the model’s forward pass, allowing:
- inspection of tensor objects
- extraction of virtual memory addresses
- mapping of runtime state to forensic artifacts

---

## Dynamic Analysis with GDB

Using GDB, the following are captured at runtime:
- Process ID (PID)
- Virtual addresses of:
  - input tensors
  - output tensors
  - TorchScript module object
- Internal PyTorch data structures (TensorImpl, StorageImpl)

These addresses form the ground truth for later forensic validation.

---

## Memory Acquisition

Physical memory is captured using **LiME** while the inference process is active.

Key properties:
- Linux kernel module–based acquisition
- Preserves process address space and file-backed mappings
- Enables accurate VMA reconstruction

---

## Memory Forensics with Volatility 3

Volatility 3 is used to:
- Identify the inference process in memory (`linux.pslist`)
- Enumerate virtual memory areas (`linux.proc.Maps`)
- Locate file-backed VMAs corresponding to the TorchScript model
- Validate runtime addresses captured via GDB using `volshell`

A matching Linux kernel symbol file (ISF) is generated using `dwarf2json`.

---

## Custom Volatility Plugin: TorchScript Recovery

A custom Volatility 3 plugin is implemented to automate model recovery.

### Plugin Capabilities
- Enumerates VMAs for a target PID
- Identifies TorchScript-backed memory regions
- Dumps memory segments to disk
- Reconstructs the serialized TorchScript archive

### Recovered Artifacts
- Model weights
- Layer architecture and order
- Serialized TorchScript code invoked during the forward pass

---

## Results & Impact

This project demonstrates that:
- ML models deployed via PyTorch are **recoverable from memory**
- TorchScript archives can be reconstructed post-execution
- Standard DFIR tooling is sufficient to extract ML intellectual property

The findings have implications for:
- malware analysis
- incident response
- ML IP protection
- secure deployment of ML systems

---

## Disclaimer

This project is for **educational and defensive research purposes only**.  
No recovered artifacts are redistributed beyond the scope of analysis.

---

## Author

Independent research project exploring the intersection of:
**Machine Learning Systems · Reverse Engineering · Linux Memory Forensics**
