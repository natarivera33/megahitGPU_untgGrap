# GPU-Accelerated Unitig Graph Construction in MEGAHIT

This repository contains the source code, datasets, scripts, and supplementary results associated with the work:

**Accelerating Unitig Graph Construction in MEGAHIT using a Low-Profile GPU**

The project extends **MEGAHIT v1.2.9** by accelerating the Unitig Graph construction stage using NVIDIA GPUs and CUDA. In particular, the GPU implementation parallelizes the identification and construction of simple paths and looped paths in the de Bruijn graph.

The implementation was developed in C++ and CUDA while preserving the remaining MEGAHIT assembly pipeline.

---

## Repository contents

The repository contains:

- `megahit-sourceCode.zip` — source code of the modified GPU-accelerated version of MEGAHIT.
- `scripts_get_times_from_logs.zip` — scripts used to extract and summarize execution times from MEGAHIT log files.
- Paired-end FASTQ subsets used in the scalability experiments (e.g., `5k_1.fastq.gz`, `5k_2.fastq.gz`, etc.).
- `supplementary_table_complete_assembly_quality_metrics.png` — supplementary assembly-quality results.
- `supplementary_table_full_execution_breakdown.png` — detailed execution-time breakdown.

---

## Implementation

The GPU implementation focuses on the **Unitig Graph construction** stage of MEGAHIT.

The main operations accelerated on the GPU are:

1. identification and construction of simple paths;
2. identification and construction of looped paths;
3. transfer of the resulting vertices back to the CPU so that the standard MEGAHIT pipeline can continue.

CUDA kernels are used for parallel graph traversal, while the auxiliary data structures required by these operations are prepared before kernel execution.

The remaining stages of the MEGAHIT pipeline are executed using the original CPU workflow.

---

## Requirements

The implementation requires:

- Linux
- C++ compiler with C++11 support
- CMake
- NVIDIA GPU with CUDA support
- NVIDIA CUDA Toolkit
- zlib

The implementation was developed from **MEGAHIT v1.2.9**.

The NVIDIA GPU must have a CUDA Compute Capability supported by the installed CUDA Toolkit. When compiling the software, the CUDA architecture configured through CMake should be compatible with the target GPU.

---

## Hardware used in the experiments

The scalability experiments reported in the manuscript were performed using an **NVIDIA GeForce GTX 1650 GPU**.

---

## Building the GPU-accelerated version

Download or clone this repository and extract the source code:

```bash
unzip megahit-sourceCode.zip
cd megahit-sourceCode
```

Create a separate build directory:

```bash
mkdir build
cd build
```

### Release build

Configure the project using CMake in **Release mode**:

```bash
cmake .. -DCMAKE_BUILD_TYPE=Release
```

> **Important:** A `Release` build should be used when reproducing the performance experiments reported in the study. Release mode enables compiler optimizations and avoids the substantial performance differences that may occur when compiling without optimization or using a Debug configuration.

Then compile the project:

```bash
make -j
```

After compilation, verify the generated executable:

```bash
./megahit --version
```

The expected MEGAHIT version is **v1.2.9**.

---

## CUDA GPU architecture

The CUDA code must be compiled for an architecture compatible with the NVIDIA GPU on which the program will be executed.

When using a different NVIDIA GPU, the CUDA architecture configured during compilation should be checked and, if necessary, changed to match the Compute Capability of the target GPU.

The CUDA architecture can be explicitly specified during CMake configuration:

```bash
cmake .. \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_CUDA_ARCHITECTURES=<architecture>
```

For example, for a GPU with CUDA Compute Capability 8.6:

```bash
cmake .. \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_CUDA_ARCHITECTURES=86
```

The value `86` should **not** be considered a universal setting. Users running the implementation on another NVIDIA GPU should determine the Compute Capability of their device and replace this value with the appropriate architecture.

For reproducible performance measurements, the following information should be recorded:

- GPU model;
- CPU model;
- CUDA Toolkit version;
- CUDA architecture used during compilation;
- C/C++ compiler version;
- CMake version;
- CMake build type.

### Clean rebuild when changing GPU architecture

When moving the source code to a system with a different GPU architecture, a clean build is recommended to avoid reusing object files compiled for another architecture.

For example:

```bash
rm -rf build
mkdir build
cd build

cmake .. \
    -DCMAKE_BUILD_TYPE=Release \
    -DCMAKE_CUDA_ARCHITECTURES=<architecture>

make -j
```

This ensures that the CUDA source files are compiled for the target GPU architecture.

---

## Running MEGAHIT

A paired-end dataset can be assembled using:

```bash
./megahit \
    -1 /path/to/dataset_1.fastq.gz \
    -2 /path/to/dataset_2.fastq.gz \
    -o /path/to/output_directory
```

For example, using a 100k paired-end subset:

```bash
./megahit \
    -1 /path/to/100k_1.fastq.gz \
    -2 /path/to/100k_2.fastq.gz \
    -o output_100k
```

MEGAHIT produces its standard assembly outputs together with execution information in its log files.

The modified implementation additionally records detailed timing information for the GPU-accelerated Unitig Graph operations, including kernel execution and CPU-GPU data transfers.

---

## Scalability datasets

The repository provides paired-end FASTQ subsets used in the scalability experiments. These datasets allow the behavior of the CPU and GPU implementations to be evaluated as the input size increases.

The study also evaluates six paired-end anaerobic digestion metagenomic datasets generated using Illumina NovaSeq 6000 technology (Callejas et al., 2025):

| Dataset | Sequences |
|---|---:|
| AS_C1_22230k | 22,230,277 |
| AS_C2_20101k | 20,101,162 |
| AS_C3_3994k | 3,994,180 |
| AS_C4_5867k | 5,867,316 |
| AS_C5_6283k | 6,283,699 |
| AS_C6_7135k | 7,135,312 |

Because of their size, the complete metagenomic datasets are not stored directly in this repository. Please refer to the manuscript and the corresponding dataset source (Callejas et al., 2025) for their provenance and availability.

---

## Reproducing the timing analysis

The archive:

```text
scripts_get_times_from_logs.zip
```

contains scripts used to extract execution times from the MEGAHIT log files and generate the timing summaries used in the experimental analysis.

Extract the scripts with:

```bash
unzip scripts_get_times_from_logs.zip
```

The modified implementation records detailed execution times for the operations involved in Unitig Graph construction.

In particular, CPU-GPU data-transfer time is calculated by accumulating the transfers associated with:

1. transfer of the data required for simple-path processing from CPU to GPU;
2. transfer of simple-path results from GPU to CPU;
3. transfer of the data required for loop processing from CPU to GPU;
4. transfer of loop-processing results from GPU to CPU.

This definition corresponds to the transfer overhead considered when evaluating the GPU implementation in the scalability experiments.

The detailed logs also allow kernel execution, data transfers, GPU preparation, and other auxiliary operations to be analyzed separately.

---

## Assembly-quality evaluation

CPU and GPU assemblies were compared using standard assembly statistics, including:

- number of contigs;
- total assembled length;
- minimum contig length;
- maximum contig length;
- average contig length;
- N50;
- additional assembly-quality statistics reported in the supplementary results.

The supplementary assembly-quality results are available in:

```text
supplementary_table_complete_assembly_quality_metrics.png
```

A detailed execution-time breakdown is available in:

```text
supplementary_table_full_execution_breakdown.png
```

---

## Reproducibility notes

For reproducible CPU-GPU comparisons:

1. Use the same input FASTQ files for both CPU and GPU executions.
2. Compile the performance version using:

   ```bash
   cmake .. -DCMAKE_BUILD_TYPE=Release
   ```

3. When using a different NVIDIA GPU, verify that `CMAKE_CUDA_ARCHITECTURES` is compatible with the Compute Capability of the target device.
4. Record the CPU model, GPU model, CUDA Toolkit version, compiler version, CMake version, and CUDA architecture.
5. Retain the complete MEGAHIT log files.
6. Use the provided analysis scripts to extract timing measurements consistently.
7. When reporting GPU execution times, clearly distinguish kernel execution, CPU-GPU data transfers, GPU preparation, and complete application execution time.

The experiments reported in the study were repeated multiple times for each evaluated configuration, and timing information was obtained from the corresponding execution logs.

---

## Original software

This work is based on **MEGAHIT v1.2.9**, a metagenome assembler originally developed by Li et al.

Users interested in the original MEGAHIT implementation should refer to the official MEGAHIT project and its documentation.

---

## Citation

If you use this GPU-accelerated implementation, please cite:

> Rivera, N., Ezzatti, P., and Callejas, C.  
> **Accelerating Unitig Graph Construction in MEGAHIT using a Low-Profile GPU.**

Full publication information will be added after publication.

---

## License

This repository contains modifications derived from MEGAHIT. Please consult the `LICENSE` file in this repository for the applicable licensing terms and the original MEGAHIT project for the licensing terms of the upstream software.

---

## Contact

For questions related to the GPU implementation, compilation, datasets, or experimental evaluation, please open an issue in this repository.
