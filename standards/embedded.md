# Embedded C/C++ engineering standards

These standards apply to microcontroller firmware, RTOS, and bare-metal environments whenever `CMakeLists.txt`, `platformio.ini`, `Kconfig`, `*.ld`, or embedded `Makefile` markers are present in the workspace.

## Architecture & hardware boundaries
1. **HAL isolation & driver interfaces**:
   - Strictly separate application/domain logic from Hardware Abstraction Layers (HAL), board pin maps, and peripheral drivers behind narrow C module interfaces (opaque handles + explicit `init`/`poll`/`reset` functions) or abstract C++ interfaces so core logic compiles and runs on the host without target SDK headers.
   - Centralize pin definitions and tunable safety timings in dedicated configuration headers (`constexpr` constants in C++, typed constants/`enum`s in C) rather than scattering magic numbers or `#ifdef` board checks across business logic.
2. **Deterministic memory & type safety**:
   - Pre-allocate buffers, pools, and RTOS objects statically or during initialization; forbid dynamic allocation (`malloc`/`free`, `new`/`delete`) and heap-allocating wrappers (`Arduino::String`, unbounded `std::string`/`std::vector` churn) in steady-state loops or ISRs. Keep stack usage shallow (no large local buffers, `alloca`, or recursion).
   - Use fixed-width integer types (`<stdint.h>` / `<cstdint>`: `uint8_t`, `uint16_t`, `uint32_t`, `int32_t`, `size_t`) for hardware registers, wire formats, and timers; pass explicit buffer sizes (`size_t len`) and forbid unbounded C functions (`strcpy`, `strcat`, `sprintf`, `gets`).
3. **Fail-safe defaults, NVM & OTA hygiene**:
   - Initialize safety-critical outputs/actuators to a known safe state before enabling peripherals or networking, guard execution with hardware/task watchdogs (`WDT`) and reset-reason checks (brownout, panic, watchdog lockout), and make flash/EEPROM/NVS writes idempotent (skip writes when values are unchanged) to prevent wear-out.
   - Require verified/signed images and post-boot health confirmation (`PENDING_VERIFY` rollback protection) for OTA updates, and disengage physical loads cleanly before flashing or rebooting.

## Code discipline, timing & concurrency
1. **C vs. modern embedded C++ idioms**:
   - Follow **[BARR-C](https://barrgroup.com/embedded-systems/books/embedded-c-coding-standard)** / **[C++ Core Guidelines](https://isocpp.github.io/CppCoreGuidelines/CppCoreGuidelines)** (and **[ESP-IDF style](https://docs.espressif.com/projects/esp-idf/en/stable/esp32/contribute/style-guide.html)** or repository `.clang-format` rules when present); check every driver and bus return status and handle CRC, framing, or timeout errors explicitly.
   - In embedded C++, prefer `enum class`, `constexpr`, `std::array`/`std::span`, and RAII/scoped locks over preprocessor macros and manual cleanup; disable exceptions and RTTI (`-fno-exceptions -fno-rtti`) on resource-constrained targets unless the runtime requires them.
2. **Non-blocking execution & wraparound-safe timing**:
   - Never block superloops or shared RTOS tasks with busy-waits, long `delay()` calls, or unbounded network/bus I/O; model asynchronous interfaces (UART, RS-485/Modbus, SPI, I2C, BLE, CAN) and debouncing as explicit finite state machines with bounded retries, exponential backoff, and timeouts.
   - Compute elapsed time using unsigned subtraction (`(uint32_t)(now - start) >= interval`) so millisecond/microsecond timers remain safe across counter wraparound.
3. **Interrupts (ISR) & RTOS concurrency**:
   - Keep ISRs minimal and deterministic: clear interrupt flags, push raw data into a lock-free ring buffer or signal via RTOS `*FromISR` primitives (`xTaskNotifyFromISR`, `xQueueSendFromISR`), and exit. Never allocate memory, log, or call blocking APIs inside an ISR.
   - Do not treat `volatile` as thread synchronization: reserve `volatile` for memory-mapped I/O registers and bare-metal single-word ISR flags; use `<stdatomic.h>` / `std::atomic`, critical sections, queues, or mutex-guarded snapshots for shared multi-word or cross-task state (keeping critical sections free of blocking I/O to avoid priority inversion).

## Verification & testing standards
1. **Host-side unit tests & hardware shims**:
   - Decouple state machines, safety interlocks, protocol parsers, and control math into hardware-independent modules tested natively on the host (`[env:native]` in **[PlatformIO](https://docs.platformio.org/en/latest/advanced/unit-testing/index.html)** or host CMake `ctest`) via **[Unity](https://www.throwtheswitch.org/unity)**, **[CppUTest](https://cpputest.github.io/)**, or **[GoogleTest](https://google.github.io/googletest/)**.
   - Provide deterministic time mocks (`mock_millis` / injected clock) and lightweight GPIO/HAL shims so debounce, dwell, timeout, fault lockout, and timer wraparound paths are verified without physical hardware.
2. **Compiler warnings & static analysis**:
   - Compile project sources with `-Wall -Wextra -Werror` (plus `-Wconversion` / `-Wshadow` scoped to `src/` so vendor SDK headers don't block builds, and `-Wstrict-prototypes` for C targets) with zero warnings.
   - Run **[`clang-tidy`](https://clang.llvm.org/extra/clang-tidy/)** or **[`cppcheck`](https://cppcheck.sourceforge.io/)** (and **MISRA C/C++** rules when configured) before merge.
3. **Target build parity & footprint checks**:
   - Verify both host unit tests (`pio test -e native` / `ctest`) and all target firmware configurations (`pio run` / `cmake --build` / `idf.py build`) in CI, monitoring Flash/RAM section budgets (`size <elf>`) and RTOS task stack high-water marks.
