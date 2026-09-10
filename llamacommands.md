## Here are the general use case commands for Llama: 

General command: llama-cli -hf unsloth/Llama-3.2-1B-Instruct-GGUF:Q4_0 -c # -b # -ub # -t # -cnv

Optimized command for MacBook Neo: llama-cli -hf unsloth/Llama-3.2-1B-Instruct-GGUF:Q4_0 -c 8192 -b 128 -ub 64 -t 6 -cnv

## In llama.cpp (and its wrapper llama-cli), these single-letter flags control model memory, context size, and processing speed.

-c # (Context Size / n_ctx): Sets the context window length in tokens (e.g., -c 4096 or -c 8192). This determines the maximum total length of text (input prompt + generated response) the model can process at one time.

-b # (Batch Size / n_batch): Sets the maximum number of tokens processed simultaneously during prompt evaluation/prefill. A larger batch size speeds up the time it takes to read long input prompts, provided your system has enough memory.

-ub # (Physical Micro-Batch Size / n_ubatch): Sets the physical batch size used for CPU/GPU matrix multiplication operations. Lowering this helps fit prompt processing within tight VRAM or RAM limits without lowering the overall logical batch size (-b).

-t # (Threads / n_threads): Sets the number of CPU threads assigned to text generation. This should generally match the number of physical CPU cores on your machine (not logical threads) for optimal performance.

Additional Flag Note:

-cnv (Conversation Mode): This boolean flag doesn't take a number; it toggles an interactive, chat-style terminal session where context is maintained across turns.
