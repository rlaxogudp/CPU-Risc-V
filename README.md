📚 SystemVerilog를 이용한 RISC-V CPU 설계

본 프로젝트는 SystemVerilog를 사용하여 RISC-V의 32비트 정수 명령어 집합(RV32I)을 구현한 CPU 코어 설계 프로젝트입니다.

발표자: 김태형, 김호준, 안유한

1. 📝 프로젝트 개요

목표: RISC-V ISA (명령어 집합 구조)를 이해하고, SystemVerilog를 사용해 실제 동작하는 RV32I CPU 코어를 설계 및 검증합니다.

ISA: RISC-V (오픈소스 표준 ISA)

설계 언어: SystemVerilog

개발 환경: Vivado

2. 🏛️ CPU 아키텍처 (RV32I Core)

본 CPU 코어는 명령어를 해독하는 Control Unit과 실제 연산을 수행하는 DataPath로 구성됩니다.

주요 구성 요소

Control Unit (제어 유닛):

명령어의 Opcode, Funct3, Funct7 필드를 입력받아 해독합니다.

RegFile_we, aluSrcSel, aluControl 등 CPU 각 부분이 동작하는 데 필요한 모든 제어 신호를 생성합니다.

DataPath (데이터 경로):

PC (Program Counter): 실행할 다음 명령어의 주소를 가리킵니다.

ROM (Instruction Memory): CPU가 실행할 명령어를 보관하는 메모리입니다.

Register File (레지스터 파일): 32개의 범용 레지스터(x0-x31)를 포함하며, 연산에 사용할 데이터를 읽거나 연산 결과를 저장합니다.

ALU (Arithmetic Logic Unit): ADD, SUB, AND, OR 등 실제 산술 및 논리 연산을 수행합니다.

Extend: I-Type, S-Type 등에서 사용되는 즉시값(immediate)을 32비트로 부호 확장합니다.

RAM (Data Memory): Load 및 Store 명령어 실행 시 데이터를 저장하거나 불러오는 역할을 합니다.

3. 🧮 지원 명령어 유형 (RV32I)

RV32I의 기본 명령어 포맷인 6가지 유형(R, I, S, B, U, J)을 구현하고 시뮬레이션을 통해 검증했습니다.

1. R-Type (Register-Type)

설명: 레지스터와 레지스터 간의 연산을 수행합니다.

구현 명령어: ADD, SUB, SLL, SLT, SLTU, XOR, SRL, SRA, OR, AND

데이터 경로: 두 개의 소스 레지스터(rs1, rs2)에서 값을 읽어 ALU에서 연산한 뒤, 결과를 목적지 레지스터(rd)에 씁니다.

2. I-Type (Immediate-Type)

설명: 레지스터와 즉시값(imm) 간의 연산 또는 Load 명령어를 수행합니다.

구현 명령어:

산술/논리: ADDI, SLTI, SLTIU, XORI, ORI, ANDI

시프트: SLLI, SRLI, SRAI

Load: LB, LH, LW, LBU, LHU (L-Type)

JALR: JALR (J-Type 변형)

3. S-Type (Store-Type)

설명: 레지스터의 값을 메모리에 저장합니다.

구현 명령어: SB, SH, SW

데이터 경로: rs1의 값과 imm을 더해 메모리 주소를 계산하고, rs2의 값을 해당 주소의 RAM에 씁니다.

4. B-Type (Branch-Type)

설명: 두 레지스터의 값을 비교하여 조건에 따라 분기(점프)합니다.

구현 명령어: BEQ, BNE, BLT, BGE, BLTU, BGEU

데이터 경로: rs1과 rs2를 비교한 결과(btaken)가 참이면, PC에 imm만큼 더해 프로그램의 실행 흐름을 변경합니다.

5. U-Type (Upper-Immediate-Type)

설명: 20비트의 즉시값을 레지스터의 상위 20비트에 로드합니다.

구현 명령어: LUI, AUIPC

LUI (Load Upper Immediate): imm 값을 12비트 왼쪽 시프트하여 rd에 저장합니다.

AUIPC (Add Upper Immediate to PC): imm 값을 12비트 왼쪽 시프트하여 현재 PC 값과 더한 결과를 rd에 저장합니다.

6. J-Type (Jump-Type)

설명: 무조건 점프를 수행하며, 복귀 주소를 레지스터에 저장합니다.

구현 명령어: JAL

데이터 경로: PC + 4 (다음 명령어 주소)를 rd에 저장하고, PC에 imm 값을 더해 해당 주소로 점프합니다. JALR (I-Type 포맷)은 rs1 + imm 주소로 점프합니다.

4. 🔬 검증 환경 (Testbench)

명령어 타입별로 UVM(Universal Verification Methodology)과 유사한 구조의 SystemVerilog 테스트벤치 환경을 구축하여 검증을 수행했습니다.

Generator: 랜덤한 제약 조건으로 transaction (명령어)을 생성합니다.

Driver: Generator로부터 transaction을 받아 DUT (CPU)의 인터페이스로 신호를 인가합니다.

Monitor: DUT의 출력 신호(메모리 주소, 레지스터 쓰기 값 등)를 관찰(샘플링)합니다.

Scoreboard: Generator가 transaction을 기반으로 계산한 예상 결과와 Monitor가 관찰한 실제 결과를 비교하여, 명령어의 동작이 정확한지 PASS 또는 FAIL로 판정합니다.
