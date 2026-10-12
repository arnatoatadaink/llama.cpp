# MTP-02A diagnostic token trace (initial instrument)

**Scope:** personal llama.cpp fork, branch `research/mtp-token-trace-2026-10-12`. Base: `e8fd209c975c80aa7fb36a8423571ff3b1b7e955` on `prism`. This is an **instrumentation-only** first pass; no target logits captured and no claim of speculative-decoding correctness.

Changes in `tools/server/server-context.cpp`:
- When `trace > 1`, write `MTP_TRACE proposal` (sampled anchor, target prompt token count, draft proposal token IDs).
- After `common_sampler_sample_and_accept_n`, write `MTP_TRACE verification` (draft length, accepted prefix length, verified/target token IDs).
- No raw text and no logits; normal runtime `trace <= 1` remains unchanged.
- These traces are **server-side at the speculative verification site**; absence of trace from ordinary non-speculation is expected.

Build separately from known-good baseline; do not overwrite `build-vulkan-ucrt64`:

```powershell
Set-Location "D:\AI\PrismML\dev\llama.cpp"
git fetch origin
git switch research/mtp-token-trace-2026-10-12
$env:PATH = "D:\msys64\ucrt64\bin;$env:PATH"
cmake -S . -B build-vulkan-mtp-trace -G Ninja -DCMAKE_BUILD_TYPE=Release -DGGML_VULKAN=ON -DLLAMA_CURL=OFF -DCMAKE_C_COMPILER="D:/msys64/ucrt64/bin/gcc.exe" -DCMAKE_CXX_COMPILER="D:/msys64/ucrt64/bin/g++.exe" -DVulkan_INCLUDE_DIR="D:/msys64/ucrt64/include" -DVulkan_LIBRARY="D:/msys64/ucrt64/lib/libvulkan-1.dll.a" -DVulkan_GLSLC_EXECUTABLE="D:/msys64/ucrt64/bin/glslc.exe"
cmake --build build-vulkan-mtp-trace --target llama-server --parallel 8
& ".\build-vulkan-mtp-trace\bin\llama-server.exe" --help | Select-String "trace"
```

For targeted observation, take the **same** 32K Q8/Q4 ON launch arguments from the MTP-01 runner, add `--trace 2`, generate a fixed prompt/seed, then collect `MTP_TRACE` from the *running console* or a properly flushed log. Confirm actual flag spelling via `--help` first; CMake/build and Windows Native runtime remain to be validated.

Do not publish full traces from private prompts. The next phase is comparing internal confirmed token IDs across runs and adding *target argmax/top-two logit* capture only when needed. Aggregate draft acceptance does not establish correctness.
