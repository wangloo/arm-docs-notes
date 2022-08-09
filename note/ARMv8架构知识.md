## Definitions

With the introduction of 64-bit support, ARM has defined several terms:

- **AArch32** – the legacy 32-bit instruction set architecture (ISA) defined by ARM, including Thumb mode execution.
- **AArch64** – the new 64-bit instruction set architecture (ISA) defined by ARM.
- **ARMv7** – the specification of the "7th generation" ARM hardware, which only includes support for AArch32. This version of the ARM hardware is the first version Windows for ARM supported.
- **ARMv8** – the specification of the "8th generation" ARM hardware, which includes support for both AArch32 and AArch64.

Windows also uses these terms:

- **ARM** – refers to the 32-bit ARM architecture (AArch32), sometimes referred to as WoA (Windows on ARM).
- **ARM32** – same as ARM, above; used in this document for clarity.
- **ARM64** – refers to the 64-bit ARM architecture (AArch64). There's no such thing as WoA64.

Finally, when referring to data types, the following definitions from ARM are referenced:

- **Short-Vector** – A data type directly representable in SIMD, a vector of 8 bytes or 16 bytes worth of elements. It's aligned to its size, either 8 bytes or 16 bytes, where each element can be 1, 2, 4, or 8 bytes.
- **HFA (Homogeneous Floating-point Aggregate)** – A data type with 2 to 4 identical floating-point members, either floats or doubles.
- **HVA (Homogeneous Short-Vector Aggregate)** – A data type with 2 to 4 identical Short-Vector members.