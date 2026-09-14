# 串传送指令MOVS与数据块搬家

3.13 节串操作一共五种，上节已经点了名：传送 `MOVS`（`MOVSB`、`MOVSW`），比较 `CMPS`（`CMPSB`、`CMPSW`），扫描 `SCAS`（`SCASB`、`SCASW`），装入 `LODS`（`LODSB`、`LODSW`），存储 `STOS`（`STOSB`、`STOSW`）。每种都能按字节按字。今天先把第一条传送掰透，例 3.36 数据块搬家就是它。

## 四条隐含寻址规则

五种指令目的源全是隐含寻址，规则四条：

源在存储器，地址由 `DS:SI` 指明，隐含的，用前先对 `DS`、`SI` 初始化。源不在存储器在寄存器，字节隐含在 `AL`，字隐含在 `AX`。

目的在存储器，必须由 `ES:DI` 指明，目的串必须定义在附加段 `ES` 段。目的在寄存器，字节用 `AL`，字用 `AX`。

任何一条执行完，`SI`、`DI`（或其中之一，另一端在寄存器嘛）自动变。方向受 `DF` 控制：`DF=0` 自动增，`DF=1` 自动减。增减 1 还是 2看类型：字节增减 1，字增减 2。

任何一条左边都能带重复前缀。`REP` 等效 `LOOP`，`CX` 减 1。`REPZ`（`REPE`）等效 `LOOPZ`，`CX` 减 1 不等于 0 且 `ZF=1` 再来一遍。`REPNZ`（`REPNE`）等效 `LOOPNZ`。`REP` 能挂 `MOVS`、`LODS`、`STOS` 左边，`REPZ`、`REPNZ` 能挂 `CMPS`、`SCAS` 左边，具体哪边挂哪，我列过表。

## MOVSB执行三步过程

助记符 `MOVS`，格式 `MOVSB`、`MOVSW`，或 `MOVS 目的,源` 三种。

`MOVSB` 一条，`CPU` 干两步：第一步源送目的。源默认 `DS` 段 `SI` 作偏移，`MOVSB` 字节，`MOVSW` 字，送到目的，目的固定 `ES` 段 `DI` 指的单元。源目的全隐含。第二步 `SI`、`DI` 自动变，`DF=0` 增，`DF=1` 减，字节 1 字 2。

左边加了 `REP`，还没完，第三步 `CX` 减 1 给 `CX`，不等 0 重来一遍。执行多少遍受 `CX` 限制，相当于循环。传送是数据搬家，不影响 `PSW`。

第三种格式只告诉汇编按字节还是按字翻译，看两操作数定好的类型定。都是字节等效 `MOVSB`，都是字等效 `MOVSW`。实际不太灵，写就得两边类型一致，不然语法错。

跟 `MOV` 的区别记死：`MOVSB`、`MOVSW` 能存储单元到存储单元直接送，`MOV` 不能两单元直送。

书 74 页上方例子：前面 `DATA` 段、`DATA1` 段，`DS` 指 `DATA`，`ES` 指 `DATA1`，`DATA1` 就是附加段了。下面 `MOVS BUFFER2,BUFFER1`，`BUFFER2`、`BUFFER1` 在 `DATA1`（`ES`）用 `DW` 定义，汇编就按 `MOVSW` 编。源 `BUFFER1` 在 `ES` 段，源地址是不是 `ES:SI`？源段能用段超越改，目的段必须 `ES`，两概念不一样。

## 例3.36程序框架实现

书 75 页例 3.36，按标准完整程序设计。读题：`BUFFER1` 100 字节，按次序送 `BUFFER2`。`BUFFER1` 100 字节数据，`BUFFER2` 留 100 字节自由空间，数据块搬家。用串传送做：

```assembly
DATA SEGMENT
BUFFER1 DB 100 DUP(?)
BUFFER2 DB 100 DUP(?)
DATA ENDS
CODE SEGMENT
    ASSUME CS:CODE, DS:DATA, ES:DATA
START:
    MOV AX, DATA
    MOV DS, AX
    MOV ES, AX
    LEA SI, BUFFER1
    MOV DI, OFFSET BUFFER2
    MOV CX, 100
    CLD
    REP MOVSB
    MOV AH, 4CH
    INT 21H
CODE ENDS
END START
```

堆栈段省略了，黑板不够。数据段 `DB` 定义，实际工作把真数据写进去，作业写 `100 DUP(?)` 行。`BUFFER2` 是目的串，该在 `ES` 段，我都定在 `DATA` 了，所以声明 `DS`、`ES` 同一个段，`AX` 过一下都初始化成 `DATA`，都指 `DATA` 段地址。

`LEA SI,BUFFER1` 源有效地址给 `SI`，段在 `DS`。`MOV DI,OFFSET BUFFER2`（`LEA` 也行）目的偏移给 `DI`，段按串操作在 `ES`。`MOV CX,100` 重复次数给 `CX`。首址对首址搬一个，`SI`、`DI` 自动增 1，按地址增大搬，得确保 `DF=0`。开机、`RESET` 后 `DF` 就是 0，为保险排一条 `CLD`。然后 `REP MOVSB`，`CPU` 执行 100 遍，一遍搬一字节，源块到目的地。搬完 `4CH` 结束。

```text
源 DS:SI -> [B1][B1+1]...[B1+99]  (SI+1/遍)
              |搬100次|
目的 ES:DI -> [B2][B2+1]...[B2+99]  (DI+1/遍)
CX=100 -> 0 , DF=0 CLD保证增方向
```
