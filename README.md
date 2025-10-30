📚 RISC-V RV32I Processor Architecture Overview

본 프로젝트는 SystemVerilog로 구현된 RISC-V (RV32I) 프로세서 코어의 아키텍처를 다룹니다. 특히, **단일 사이클(Single-Cycle)**과 멀티 사이클(Multi-Cycle) 두 가지 주요 구현 방식을 비교하고 설명합니다.

🗂️ 공통 프로젝트 구조 (Common Code Structure)

File Name

Description

MCU.sv

최상위 모듈입니다. [cite_start]CPU 코어, ROM (명령어 메모리), RAM (데이터 메모리)을 통합합니다. [cite: 109, 110]

CPU_RV32I.sv

CPU 코어. [cite_start]Control Unit과 Data Path를 연결합니다. [cite: 108]

DataPath.sv

데이터 경로. PC, 레지스터 파일, ALU, Mux 등 핵심 컴포넌트를 포함합니다. (구현 방식에 따라 내부 구조가 달라집니다.) [cite_start][cite: 43]

ControlUnit.sv

명령어 Opcode에 기반하여 제어 신호를 생성합니다. (구현 방식에 따라 조합 논리 또는 FSM으로 구현됨) [cite_start][cite: 18]

RAM.sv

데이터 메모리. [cite_start]바이트, 하프워드, 워드 쓰기를 지원합니다. [cite: 30, 32]

ROM.sv

명령어 메모리. [cite_start]code.mem 파일로부터 명령어를 읽어옵니다. [cite: 2]

defines.sv

[cite_start]ALU 연산 및 RISC-V Opcode 상수가 정의된 헤더 파일입니다. [cite: 17]

code.mem

테스트용 RISC-V 기계어 코드가 포함된 파일입니다.

1. ⚡ 단일 사이클 CPU (Single-Cycle)

단일 사이클 아키텍처는 가장 직관적인 구현 방식입니다. 모든 명령어가 단 하나의 클럭 사이클 내에 인출부터 쓰기까지의 모든 단계를 완료합니다.

특징 및 구조

아키텍처: 단일 사이클. 단순하며 빠른 설계가 가능합니다. 

성능 제한: 클럭 주기는 **가장 느린 명령어(Critical Path)**의 실행 시간에 의해 결정됩니다.

제어: ControlUnit.sv는 FSM(상태 머신) 없이, Opcode에 따라 모든 제어 신호를 조합 논리로 즉시 생성합니다.

Data Path (DataPath.sv): 모든 기능 블록이 파이프라인 레지스터 없이 직접 연결됩니다.

지원 명령어 유형 (RV32I)

| 유형 | 설명 | 예시 명령어 | 
| :--- | :--- | :--- |
| R-Type | [cite_start]레지스터 간 산술/논리 [cite: 17] | ADD, SUB, SLL |
| I-Type | [cite_start]즉시값 연산/로드 [cite: 17] | ADDI, LW, JALR |
| S-Type | [cite_start]메모리 저장 [cite: 17] | SB, SH, SW |
| B-Type | [cite_start]조건부 분기 [cite: 17] | BEQ, BNE, BLT |
| U/J-Type | [cite_start]상위 즉시값/점프 [cite: 17] | LUI, AUIPC, JAL |

2. ⏱️ 멀티 사이클 CPU (Multi-Cycle)

멀티 사이클 아키텍처는 명령어를 여러 단계로 나누어 실행함으로써 하드웨어 자원을 재사용하고 클럭 속도를 높입니다.

특징 및 구조

아키텍처: 멀티 사이클. 명령어에 따라 실행 사이클 수가 다릅니다. 

클럭 속도: 클럭 주기는 **가장 느린 단계(Phase)**의 실행 시간에 의해 결정되므로, 단일 사이클보다 훨씬 빠른 클럭 속도를 가질 수 있습니다.

제어: ControlUnit.sv는 **FSM (Finite State Machine)**으로 구현되어, 현재 **상태(State)**에 따라 매 클럭마다 다른 제어 신호를 순차적으로 생성합니다.

Data Path (DataPath.sv):
    * 공유 자원: ALU와 Mux 등의 기능 블록이 여러 단계에서 재사용됩니다.
    * 중간 레지스터: Instruction Register (IR), ALUOut Register, MDR (Memory Data Register) 등 각 단계의 출력을 저장하는 레지스터를 사용하여 긴 조합 논리 경로를 짧게 분할합니다.

FSM 주요 단계 (예시)

1.  Fetch (인출): 명령어 메모리 접근
2.  Decode (디코드): 레지스터 피연산자 읽기
3.  Execute (실행): ALU 연산 또는 주소 계산
4.  Memory (메모리): 데이터 메모리 접근
5.  Write Back (쓰기): 레지스터 파일에 최종 결과 기록
