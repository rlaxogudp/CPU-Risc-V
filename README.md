# ⚡ Single-Cycle RISC-V RV32I Processor

본 프로젝트는 SystemVerilog로 구현된 **단일 사이클 (Single-Cycle)** RISC-V RV32I 프로세서 코어입니다. 모든 명령어는 클럭 주기에 관계없이 **단 하나의 클럭 사이클** 내에 완료되는 가장 직관적인 CPU 구조입니다.

## 🗂️ 프로젝트 구조 (Code Structure)

| File Name | Description | 
 | ----- | ----- | 
| `MCU.sv` | **최상위 모듈**입니다. CPU 코어, ROM (명령어 메모리), RAM (데이터 메모리)을 통합합니다. | 
| `CPU_RV32I.sv` | **CPU 코어**. Control Unit과 Data Path를 연결합니다. | 
| `DataPath.sv` | **데이터 경로**입니다. PC, 레지스터 파일, ALU, Mux 등 모든 기능 블록이 **파이프라인 레지스터 없이** 조합 논리로 연결됩니다. | 
| `ControlUnit.sv` | 명령어 Opcode에 기반하여 **모든 제어 신호**를 단일 사이클 내에 생성합니다. (FSM 불필요) | 
| `RAM.sv` | 데이터 메모리. 바이트, 하프워드, 워드 쓰기를 지원합니다. | 
| `ROM.sv` | 명령어 메모리. `code.mem` 파일의 명령어를 읽습니다. | 
| `defines.sv` | ALU 연산 및 RISC-V Opcode 상수가 정의된 헤더 파일입니다. | 
| `code.mem` | 테스트를 위한 RISC-V 기계어 코드가 포함된 파일입니다. | 

## ⚙️ 핵심 특징 (Key Features)

* **아키텍처:** **단일 사이클**. 단순성, 빠른 개발 속도가 장점입니다. 
* **성능 제한:** 클럭 주기는 **가장 느린 명령어 (예: `LW`)**의 실행 시간(Critical Path)에 의해 결정됩니다.
* **RISC-V ISA 지원:** Base Integer Instruction Set (`RV32I`)의 모든 핵심 명령어 유형(R, I, S, B, U, J Type)을 지원합니다.

### 지원 명령어 유형

| 유형 | 설명 | 예시 명령어 | 
 | ----- | ----- | ----- | 
| **R-Type** | 레지스터 간 산술/논리 | `ADD`, `SUB`, `SLL`, `SLT` | 
| **I-Type** | 즉시값 연산/로드 | `ADDI`, `LW`, `JALR` | 
| **S-Type** | 메모리 저장 | `SB`, `SH`, `SW` | 
| **B-Type** | 조건부 분기 | `BEQ`, `BNE`, `BLT` | 
| **U/J-Type** | 상위 즉시값/점프 | `LUI`, `AUIPC`, `JAL` | 

## 💻 Data Path Overview

1. **PC (Program Counter):** `U_PC` 레지스터는 다음 클럭에 `PC + 4` 또는 분기/점프 주소로 업데이트됩니다.
2. **레지스터 파일:** `U_RegFile`에서 명령어가 요구하는 피연산자 데이터를 **즉시** 읽어옵니다.
3. **ALU (산술 논리 장치):** `U_ALU`가 모든 계산(산술, 논리, 주소 계산, 분기 비교)을 수행합니다.
4. **Write Back:** `U_RFWDSrcMux`가 ALU 결과, 메모리 데이터 또는 PC 값을 선택하여 레지스터 파일에 **쓰기** 동작을 완료합니다.
```eof

***

## 2. 멀티 사이클 RISC-V CPU README 파일

```markdown:Multi-Cycle RISC-V RV32I Processor:README_multi_cycle.md
# ⏱️ Multi-Cycle RISC-V RV32I Processor

본 프로젝트는 SystemVerilog로 구현된 **멀티 사이클 (Multi-Cycle)** RISC-V RV32I 프로세서 코어입니다. 명령어를 여러 개의 작은 단계(사이클)로 분할하여 실행하며, 기능 블록(예: ALU)을 재사용하여 하드웨어 효율성을 극대화한 구조입니다.

## 🗂️ 프로젝트 구조 (Code Structure)

| File Name | Description |
| :--- | :--- |
| `MCU.sv` | **최상위 모듈**입니다. CPU 코어, ROM (명령어 메모리), RAM (데이터 메모리)을 통합합니다. |
| `CPU_RV32I.sv` | **CPU 코어**. Control Unit과 Data Path를 연결합니다. |
| `DataPath.sv` | **데이터 경로**입니다. 기능 블록을 재사용하며, 각 단계의 결과를 저장하기 위한 **중간 레지스터**가 포함됩니다. (IR, A, B, ALUOut, MDR 등) |
| `ControlUnit.sv` | **FSM (Finite State Machine)**으로 구현되어, **현재 상태(State)**에 따라 매 클럭마다 다른 제어 신호를 생성하고 다음 상태로 전이합니다. |
| `RAM.sv` | 데이터 메모리. 바이트, 하프워드, 워드 쓰기를 지원합니다. |
| `ROM.sv` | 명령어 메모리. `code.mem` 파일의 명령어를 읽습니다. |
| `defines.sv` | ALU 연산 및 RISC-V Opcode 상수가 정의된 헤더 파일입니다. |
| `code.mem` | 테스트를 위한 RISC-V 기계어 코드가 포함된 파일입니다. |

## ⚙️ 핵심 특징 (Key Features)

* **아키텍처:** **멀티 사이클**. 명령어의 유형에 따라 실행에 필요한 사이클 수가 다릅니다. 
* **클럭 속도:** 클럭 주기는 **가장 느린 단계(Phase)**의 실행 시간에 의해 결정되므로, 단일 사이클에 비해 클럭 속도를 **크게 높일 수 있습니다.**
* **제어 유닛:** 명령어의 모든 실행 단계를 관리하는 **FSM**이 `ControlUnit.sv`의 핵심입니다.

### FSM 주요 단계
1.  **Fetch (인출):** 명령어 메모리에서 명령어를 읽고 PC를 업데이트합니다.
2.  **Decode (디코드):** 레지스터 파일에서 피연산자를 읽고 제어 신호를 결정합니다.
3.  **Execute (실행):** ALU를 사용하여 연산 또는 주소 계산을 수행합니다. (명령어에 따라 1개 이상의 사이클 소요)
4.  **Memory (메모리):** 데이터 메모리에 접근하여 읽기/쓰기를 수행합니다.
5.  **Write Back (쓰기):** 최종 결과를 레지스터 파일에 기록합니다.

## 💻 Data Path Overview

* **공유 자원:** ALU와 Mux는 여러 단계에서 주소 계산, 연산 실행, 분기 비교 등의 다양한 용도로 재사용됩니다.
* **중간 레지스터:** `Instruction Register (IR)`, `A/B Register`, `ALUOut Register` 등의 레지스터를 사용하여 이전 사이클의 결과를 저장하고 다음 사이클로 전달합니다. 이를 통해 긴 조합 논리 경로를 짧게 분할합니다.
```eof
