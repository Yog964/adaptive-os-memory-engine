# Adaptive OS Memory Engine
<img width="1536" height="1024" alt="image" src="https://github.com/user-attachments/assets/8d27ad3f-ac07-4f9f-8ec1-9024a7d8e94f" />

## Problem Statement

Modern operating systems run many processes at the same time, but users often cannot quickly tell which processes are actively important, which are idle, and which may be wasting memory. Traditional task-manager views show raw CPU or memory numbers, but they do not explain process priority from multiple usage signals.

This project solves that problem by collecting live Windows process data, analyzing usage patterns with advanced data structures, and classifying processes into `HOT`, `WARM`, and `COLD` categories. The goal is to recommend smarter memory and priority decisions based on frequency, recency, active time, memory usage, and CPU usage.

## What Is This Project?

Adaptive OS Memory Engine is a C++ desktop/CLI project that simulates an intelligent process analysis engine for operating-system memory management.

It provides:

- A Qt Widgets GUI for visual process analysis.
- A CLI menu for terminal-based analysis.
- Data-structure-backed ranking, classification, and reporting.
- Optional graph generation through Python.

## Why Does It Exist?

This project was built to demonstrate how core data structures can be applied to a real operating-system-inspired problem. Instead of only implementing isolated structures, it connects them into one workflow for collecting process data, scoring processes, ranking them, and producing memory-priority recommendations.

It is useful for:

- Data Structures coursework and demonstrations.
- Understanding practical uses of heaps, trees, hash maps, LRU lists, Fenwick trees, and segment trees.
- Exploring how process metrics can be combined into a priority score.
- Presenting an OS memory-management concept through both GUI and CLI interfaces.

## Key Features

- Live Windows process collection using Windows APIs.
- Process scoring based on frequency, recency, active time, memory, and CPU usage.
- `HOT`, `WARM`, and `COLD` process classification.
- Priority ranking using a max heap.
- Fast process lookup with a hash map.
- Sorted score views and range queries with a red-black tree.
- Dynamic ranking support with a skip list.
- Recent process tracking through an LRU doubly linked list.
- Time-based range analysis with a segment tree.
- Frequency accumulation with a Fenwick tree.
- Memory waste detection and smart recommendations.
- Qt GUI with dashboard, storage engine, data-structure, and fault-monitor tabs.
- CLI mode for systems without Qt installed.
- Python graph generation for report visuals.

## 🧠 Data Structures Used

| Data Structure | Used For | Why It Helps |
| --- | --- | --- |
| Hash Map | Stores process records by PID | Gives fast lookup, insert, and update for live process data. |
| Max Heap | Ranks processes by hotness score | Quickly finds the highest-priority or top K processes. |
| Red-Black Tree | Maintains sorted process scores | Supports ordered traversal and score-range queries. |
| Skip List | Keeps a dynamic sorted ranking | Provides efficient insertion, deletion, and search in ranked data. |
| Doubly Linked List | Implements LRU recency tracking | Tracks recently active processes and updates order efficiently. |
| Segment Tree | Performs time-window analysis | Answers range queries over usage/activity time. |
| Fenwick Tree | Tracks cumulative frequency | Efficiently updates and queries process usage frequency. |

## Used Formulas

### Hotness Score

```text
Score = 0.20 * Frequency
      + 0.25 * Recency
      + 0.20 * ActiveTime
      + 0.15 * Memory
      + 0.20 * CPU
```

### Classification Rules

```text
HOT  = Score >= 70
WARM = Score >= 40 and Score < 70
COLD = Score < 40
```

### Priority Interpretation

```text
Higher Score = Higher process importance
Lower Score  = Better candidate for memory optimization
```

## Memory Waste Detection

Memory waste is detected by comparing each process memory usage against the average memory usage of all collected processes.

### Logic Used

```text
AverageMemory = TotalMemoryUsedByAllProcesses / NumberOfProcesses

MemoryWasteCandidate =
    ProcessMemory > AverageMemory
    AND
    HotnessScore < WARM_THRESHOLD + 10
```

In this project:

```text
WARM_THRESHOLD = 40
Memory waste score condition = HotnessScore < 50
```

A process is treated as a memory-waste candidate when it uses more memory than the average process but has low activity or low priority. These processes are sorted by memory usage in descending order, so the largest waste candidates appear first.

### Why This Works

High memory usage alone does not always mean waste. A process may be memory-heavy because it is actively used. That is why this project also checks the hotness score. A process is more suspicious when it consumes high memory but has low frequency, low recency, low active time, or low CPU activity.

## Page Fault Analysis

Page fault analysis is handled by the `FaultMonitor` module. It uses real process metrics collected from the Windows API and stored in each `ProcessData` record.

### Metrics Used

| Metric | Meaning |
| --- | --- |
| `pageFaultCount` | Number of page faults recorded for the process. |
| `pagefileUsageKB` | Amount of pagefile-backed memory used by the process. |
| `peakWorkingSetKB` | Highest physical memory working-set size reached by the process. |
| `hotnessScore` | Priority score calculated by the analyzer. |
| `classification` | Process group: `HOT`, `WARM`, or `COLD`. |

### Analysis Steps

1. Collect page fault, pagefile, and working-set data from live processes.
2. Classify each process as `HOT`, `WARM`, or `COLD`.
3. Group processes by classification.
4. Calculate total and average page faults for each group.
5. Calculate total and average pagefile usage for each group.
6. Calculate total and average peak working-set usage for each group.
7. Sort processes by page fault count to find the top faulting processes.
8. Sort processes by pagefile usage to find the most swapped processes.
9. Compute correlation between page faults and hotness score.

### Fault Summary Formula

```text
AverageFaults = TotalPageFaultsInGroup / NumberOfProcessesInGroup

AveragePagefileUsageKB = TotalPagefileUsageKBInGroup / NumberOfProcessesInGroup

AveragePeakWorkingSetKB = TotalPeakWorkingSetKBInGroup / NumberOfProcessesInGroup
```

### Page Fault vs Hotness Correlation

The project calculates the Pearson correlation coefficient between page fault count and hotness score.

```text
x = PageFaultCount
y = HotnessScore

Correlation =
    (n * Sum(xy) - Sum(x) * Sum(y))
    /
    sqrt((n * Sum(x^2) - Sum(x)^2) * (n * Sum(y^2) - Sum(y)^2))
```

Interpretation:

- Positive correlation: hotter or more active processes are causing more page faults.
- Negative correlation: colder or inactive processes are causing more page faults, which may indicate swapping or memory pressure.
- Near zero: page faults and process priority do not have a strong relationship in the current sample.

## Screenshots

| Name of SS | SS |
| --- | --- |
| DS_Representation | ![DS_Representation](https://github.com/user-attachments/assets/3756690e-6dc6-4d92-9832-bc9f365d1b4d) |
| All_Logs | ![All_Logs](https://github.com/user-attachments/assets/dac05abd-935d-4cb0-b2c9-a2c1e3934527) |
| Analysis | ![Analysis](https://github.com/user-attachments/assets/7dc24cef-d464-4f12-b7ff-b1755b9ae89f) |
| Fault_Monitor | ![Fault_Monitor](https://github.com/user-attachments/assets/29bf6e3f-cdf7-4ac0-94a1-d05bc7e4de32) |


## Tech Stack

- Language: C++
- Standard: C++14 for CLI build, C++17 for Qt GUI project
- GUI: Qt Widgets
- Build tools: qmake, MinGW, mingw32-make
- Platform APIs: Windows API, PSAPI, Tool Help APIs
- Visualization: ANSI CLI tables and Python charts
- Python libraries for graphs: pandas, matplotlib, numpy

## Project Structure

```text
adaptive-os-memory-engine/
|-- analyzer_gui.pro              # Qt qmake project file
|-- main.cpp                      # Qt GUI entry point
|-- main_cli.cpp                  # CLI entry point
|-- analyzer.h                    # Scoring, classification, workflow logic
|-- data_structures.h             # Hash map, heap, tree, skip list, LRU, BIT, segment tree
|-- process_collector.h           # Windows process collection
|-- storage_engine.h              # Memory/storage layer simulation
|-- fault_monitor.h               # Fault monitoring logic
|-- visualizer.h                  # CLI display helpers
|-- graph_generator.py            # Python chart generator
|-- build_and_run.bat             # GUI build/run helper
|-- run_analyzer.bat              # GUI executable launcher
|-- test.cpp                      # Test/demo source
|-- test_mingw.cpp                # MinGW compiler check
|-- gui/
|   |-- mainwindow.*              # Main Qt window
|   |-- dashboard_tab.*           # Dashboard UI
|   |-- storage_tab.*             # Storage engine UI
|   |-- ds_visualizer.*           # Data-structure visualization UI
|   |-- fault_tab.*               # Fault monitor UI
|   |-- styles.h                  # Qt styling
|-- debug/                        # Debug build output
|-- release/                      # Release build output and Qt runtime files
```

## How To Run

### Prerequisites

Install or verify:

- Windows OS
- MinGW with `g++` and `mingw32-make`
- Qt with qmake for the GUI build
- Python 3 for graph generation
- Python packages: `pandas`, `matplotlib`, `numpy`

The GUI scripts currently expect Qt at:

```text
C:\Qt\6.11.0\mingw_64
```

If your Qt version is installed somewhere else, update `QT_PATH` and `MINGW_PATH` in `build_and_run.bat`.

### 1. Clone The Repository

```powershell
git clone <repository-url>
cd adaptive-os-memory-engine
```

If this project is inside a larger repository, move into:

```powershell
cd DS2\adaptive-os-memory-engine
```

### 2. Run The Backend

There is no separate backend server. The analysis engine is compiled directly into the CLI and GUI applications.

To build the CLI analyzer:

```powershell
g++ -std=c++14 -o analyzer_cli.exe main_cli.cpp -lpsapi -lshell32 -luser32 -ladvapi32
```

To run it:

```powershell
.\analyzer_cli.exe
```

### 3. Run The Frontend

The frontend is a Qt desktop GUI, not a web frontend.

Build and run with:

```powershell
.\build_and_run.bat
```

Or manually:

```powershell
qmake analyzer_gui.pro
mingw32-make -j8
.\release\analyzer_gui.exe
```

If a release executable already exists, run:

```powershell
.\run_analyzer.bat
```

### 4. Run With Docker

Docker support is not currently included. This is a Windows desktop project that depends on Windows process APIs and Qt runtime libraries, so it is designed to run directly on Windows rather than inside a generic Linux container.

## Main Workflow

1. Collect live process data from the Windows OS.
2. Store process records in a hash map by PID.
3. Track usage frequency with a Fenwick tree.
4. Track recency with an LRU list.
5. Analyze time-window data with a segment tree.
6. Calculate a hotness score for each process.
7. Rank processes with a max heap, red-black tree, and skip list.
8. Classify each process as `HOT`, `WARM`, or `COLD`.
9. Detect memory waste and generate recommendations.
10. Display results in the CLI, Qt GUI, or generated graph reports.

The scoring formula and classification thresholds are listed in the **Used Formulas** section above.

## API Endpoints

This project does not expose HTTP API endpoints. All features run locally through the C++ CLI or Qt GUI.

## Useful Scripts

```powershell
.\build_and_run.bat
```

Builds the Qt GUI project and starts `analyzer_gui.exe`.

```powershell
.\run_analyzer.bat
```

Runs an existing GUI build from `release/`, `debug/`, or the project root.

```powershell
python graph_generator.py
```

Generates charts from analyzer output data.

## Testing

There is no formal automated test suite configured yet.

Available checks:

```powershell
g++ -std=c++14 -o test_mingw.exe test_mingw.cpp
.\test_mingw.exe
```

CLI build check:

```powershell
g++ -std=c++14 -o analyzer_cli.exe main_cli.cpp -lpsapi -lshell32 -luser32 -ladvapi32
.\analyzer_cli.exe
```

GUI build check:

```powershell
qmake analyzer_gui.pro
mingw32-make -j8
.\release\analyzer_gui.exe
```

## Notes

- The project is Windows-specific because it reads process information through Windows APIs.
- The GUI requires a working Qt installation and matching MinGW toolchain.
- Paths with spaces can cause issues with older MinGW tools. If `mingw32-make` fails unexpectedly, try moving the project to a shorter path such as `C:\Projects\adaptive-os-memory-engine`.
- The checked-in `release/` folder may include Qt runtime DLLs, but rebuilding still requires Qt/qmake.
- Generated graph files and screenshots should be kept in dedicated output folders if added later.

## License

No license file is currently included. Add a license such as MIT, Apache-2.0, or GPL-3.0 before distributing or publishing the project.
