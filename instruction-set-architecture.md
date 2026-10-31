# 指令集架构

## 问题 1

整数加法中的进位（carry）和溢出（overflow）有什么区别？

### 答案

需要分无符号数和有符号数两种情况讨论。

对于 N 位无符号整数，进位和溢出是一致的。

若 `a + b >= 2^N`，最高位会产生进位，表示结果不能用 N 位无符号数表示，即发生无符号溢出。保存的 N 位结果为 `(a + b) mod 2^N`。[1]

对于 N 位二补码有符号整数，进位和溢出不一致。

进位只是位级加法从最高位产生的 carry-out，不能用于判断有符号溢出。有符号溢出指数学结果超出 N 位二补码的表示范围 `[-2^(N-1), 2^(N-1) - 1]`。当两个操作数同号、结果与操作数异号时，发生有符号溢出；等价地，最高位的进位输入与进位输出不同。[1, 2]

例如，对于 8 位加法：

- `0xFF + 0x01 = 0x00`：有进位；按无符号数解释为 `255 + 1`，发生无符号溢出；按有符号数解释为 `-1 + 1`，不发生有符号溢出。
- `0x7F + 0x01 = 0x80`：无进位；按无符号数解释为 `127 + 1`，不发生无符号溢出；按有符号数解释为 `127 + 1`，发生有符号溢出。

因此，进位和溢出由同一个加法器产生，但前者表示无符号数的范围越界，后者表示有符号数的范围越界。[1, 2]

## 问题 2

设 $x$ 是一个 n 位位模式按无符号二进制解释得到的整数，因此 $0 \le x < 2^n$。令 $\mathrm{msb}(x) \in \{0, 1\}$ 为 $x$ 的最高位。令 $t \in \{0, 1\}$ 表示该位模式的解释方式：当 $t = 0$ 时，按无符号整数解释；当 $t = 1$ 时，按 n 位二补码有符号整数解释。记 $v_t(x)$ 为该位模式在对应解释下表示的整数值。

已知 $v_t(x)$，请写出 $x$ 的统一计算公式。

### 答案

$$
x = v_t(x) \bmod 2^n
\tag{1}
$$

由于 $x$ 是 n 位位模式的无符号表示，且 $0 \le x < 2^n$，式（1）的模运算结果唯一地恢复为 $x$。

## 问题 3

沿用问题 2 的定义。已知 $x$，请写出 $v_t(x)$ 的统一计算公式。


### 答案

$$
v_t(x) = x - t \cdot \mathrm{msb}(x) \cdot 2^n
\tag{1}
$$

当 $t = 0$ 时，式（1）化为：

$$
v_0(x) = x
\tag{2}
$$

当 $t = 1$ 时，式（1）化为：

$$
v_1(x) =
\begin{cases}
x, & \mathrm{msb}(x) = 0 \\
x - 2^n, & \mathrm{msb}(x) = 1
\end{cases}
\tag{3}
$$

该公式以二补码为前提。位模式本身只表示模 $2^n$ 的元素；有符号与无符号是两种不同的整数解释方式。


## 问题 4

沿用问题 2 的定义。设 $x$、$y$ 是两个 n 位位模式，按无符号二进制读出的值仍记为 $x$、$y$。两个操作数分别采用 $t_x, t_y \in \{0, 1\}$ 的解释方式，其整数值为 $v_{t_x}(x)$ 与 $v_{t_y}(y)$。将乘积编码为 2n 位位模式，其低 n 位记为 $L$，高 n 位记为 $H_{t_x,t_y}$。

请推导 $L$ 与 $H_{t_x,t_y}$ 的统一公式，并对应到 RISC-V 的 `MUL`、`MULHU`、`MULHSU` 与 `MULH` 指令。

### 答案


由问题 2 的式（1）可得。将其中的位宽由 $n$ 替换为 $2n$，乘积的 2n 位编码满足：

$$
L = v_{t_x}(x)v_{t_y}(y) \bmod 2^n
\tag{1}
$$

$$
H_{t_x,t_y}
=
\left\lfloor
\frac{
\bigl(v_{t_x}(x)v_{t_y}(y) \bmod 2^{2n}\bigr)
}{2^n}
\right\rfloor
\tag{2}
$$


由问题 3 的式（1）可得：

$$
\begin{aligned}
v_{t_x}(x)v_{t_y}(y)
={}& xy - 2^n\bigl( \\
&\quad t_x \cdot \mathrm{msb}(x) \cdot y + \\
&\quad t_y \cdot \mathrm{msb}(y) \cdot x \bigr) + \\
&\quad 2^{2n}t_xt_y\mathrm{msb}(x)\mathrm{msb}(y)
\end{aligned}
\tag{3}
$$

对式（3）取 $2^n$ 模，可得低 n 位：

$$
L = xy \bmod 2^n
\tag{4}
$$

因此，低 n 位与有符号或无符号解释无关，对应 RISC-V 的 `MUL`。

在式（2）中令 $t_x = t_y = 0$。由于 $0 \le xy < 2^{2n}$，可得无符号乘积的高 n 位：

$$
H_u = H_{0,0} = \left\lfloor \frac{xy}{2^n} \right\rfloor
\tag{5}
$$

于是

$$
xy = \left(xy \bmod 2^n\right) + 2^n H_u.
\tag{6}
$$

将式（6）代入式（3），并按式（2）取高 n 位，可得：

$$
H_{t_x,t_y}
=
\left(
H_u - t_x \cdot \mathrm{msb}(x) \cdot y - t_y \cdot \mathrm{msb}(y) \cdot x
\right)
\bmod 2^n
\tag{7}
$$

对应关系如下：

| 操作数解释 | 高 n 位 | RISC-V 指令 |
| --- | --- | --- |
| unsigned × unsigned | $H_u$ | `MULHU` |
| signed × unsigned | $H_u - \mathrm{msb}(x) \cdot y \pmod {2^n}$ | `MULHSU x, y` |
| unsigned × signed | $H_u - \mathrm{msb}(y) \cdot x \pmod {2^n}$ | 交换操作数后使用 `MULHSU y, x` |
| signed × signed | $H_u - \mathrm{msb}(x) \cdot y - \mathrm{msb}(y) \cdot x \pmod {2^n}$ | `MULH` |

RISC-V 将 `MULHSU` 定义为第一个操作数按有符号、第二个操作数按无符号进行乘法并返回高半部分；因此，mixed signedness 且第二个操作数有符号时，需要交换两个操作数。[3]

## 问题 5

在 RV32IM 上，`MUL` 返回两个 32 位操作数乘积的低 32 位，`MULHU` 返回 unsigned × unsigned 乘积的高 32 位，`ADD` 的结果按模 $2^{32}$ 截断。

考虑以下 LLVM IR：

```llvm
%zx = zext i32 %x to i64
%p = mul i64 %zx, %y
```

设 `%x` 位于寄存器 `x`；`%y` 的低、高 32 位分别位于 `y_lo`、`y_hi`；结果 `%p` 的低、高 32 位分别写入 `r_lo`、`r_hi`。仅使用 `MUL`、`MULHU` 与 `ADD`，写出计算 `%p` 的 RV32IM 指令序列，并说明为何计算 `x * y_hi` 时只需 `MUL`。

### 答案

指令序列为：

```asm
mul    r_lo, x,   y_lo   # x * y_lo 的低 32 位
mulhu  r_hi, x,   y_lo   # x * y_lo 的高 32 位
mul    t0,   x,   y_hi   # x * y_hi 的低 32 位
add    r_hi, r_hi, t0
```


## 问题 6

沿用问题 5 的寄存器约定。`MULHSU` 返回 signed × unsigned 乘积的高 32 位。

考虑以下 LLVM IR：

```llvm
%sx = sext i32 %x to i64
%p = mul i64 %sx, %y
```

仅使用 `MUL`、`MULHSU` 与 `ADD`，写出计算 `%p` 的 RV32IM 指令序列，并说明第二条乘法为何必须使用 `MULHSU`，而不能使用 `MULH` 或 `MULHU`。

### 答案

指令序列为：

```asm
mul     r_lo, x,   y_lo  # sext(x) * y_lo 的低 32 位
mulhsu  r_hi, x,   y_lo  # sext(x) * y_lo 的高 32 位
mul     t0,   x,   y_hi  # sext(x) * y_hi 的低 32 位
add     r_hi, r_hi, t0
```




## 参考文献

[1] Intel Corporation. *Intel® 64 and IA-32 Architectures Software Developer’s Manual, Volume 1: Basic Architecture*, §3.4.3.1 “Status Flags” [M/OL]. 2026-09.

[2] Arm Limited. *Condition codes 1: condition flags and codes* [EB/OL]. 2013.

[3] RISC-V International. *The RISC-V Instruction Set Manual, Volume I: Unprivileged Architecture*, “M” Extension for Integer Multiplication and Division, Version 2.0 [EB/OL]. 2024. https://docs.riscv.org/reference/isa/v20240411/unpriv/m-st-ext.html.
