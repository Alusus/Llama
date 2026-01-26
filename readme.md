# Llama - Alusus Bindings for llama.cpp
[[العربية]](readme.ar.md)

Alusus language bindings for [llama.cpp](https://github.com/ggerganov/llama.cpp), providing a complete interface for running LLM inference locally. This library supports both CPU and Vulkan GPU backends.

## Installation

```
import "Apm";
Apm.importFile("Alusus/Llama");
```

## Quick Start

```alusus
import "Srl/Console";
import "Apm";
Apm.importFile("Alusus/Llama");
use Srl;
use Llama;

// Load backends
Ggml.Backend.cpuLoad();
Ggml.Backend.vkLoad();  // Optional: for Vulkan GPU support

// Load model
def model: ref[Model](Model.load("model.gguf", Model.getDefaultParams()));
def ctx: ref[Context](Context.initFromModel(model, Context.getDefaultParams()));

// Tokenize input
def tokens: array[Token, 512];
def nTokens: Int = tokenize(model.vocab, "Hello", 5, tokens, 512, true, true);

// Decode and generate
def batch: Batch = Batch.getOne(tokens, nTokens);
ctx.decode(batch);

// Sample next token
def sampler: ref[Sampler](Sampler.initGreedy());
def nextToken: Token = sampler.sample(ctx, -1);

// Cleanup
Sampler.free(sampler);
Context.free(ctx);
Model.free(model);
```

## GPU Support

Set the `GGML_USE_VULKAN` environment variable to `1` before running to enable Vulkan GPU acceleration:

```bash
GGML_USE_VULKAN=1 alusus your_script.alusus
```

---

## API Reference

### Basic Types

| Alusus Type | llama.cpp Type | Description |
|--------------------|---------------------------|------------------------------------|
| `Token`            | `llama_token`             | Token ID (alias for `Int`)         |
| `Pos`              | `llama_pos`               | Token position (alias for `Int`)   |
| `SeqId`            | `llama_seq_id`            | Sequence ID (alias for `Int`)      |
| `ProgressCallback` | `llama_progress_callback` | Progress callback function pointer |

### Constants

| Alusus Constant | llama.cpp Constant   | Value        | Description         |
|-----------------|----------------------|--------------|---------------------|
| `DEFAULT_SEED`  | `LLAMA_DEFAULT_SEED` | `0xFFFFFFFF` | Default random seed |
| `TOKEN_NULL`    | `LLAMA_TOKEN_NULL`   | `-1`         | Null token value    |

---

## Enums

### SplitMode

Controls how model layers are distributed across GPUs.

| Value   | llama.cpp Value          | Description                                |
|---------|--------------------------|--------------------------------------------|
| `NONE`  | `LLAMA_SPLIT_MODE_NONE`  | Single GPU only                            |
| `LAYER` | `LLAMA_SPLIT_MODE_LAYER` | Split layers and KV cache across GPUs      |
| `ROW`   | `LLAMA_SPLIT_MODE_ROW`   | Split with tensor parallelism if supported |

### RopeScalingType

RoPE (Rotary Position Embedding) scaling type.

| Value         | llama.cpp Value                       | Description      |
|---------------|---------------------------------------|------------------|
| `UNSPECIFIED` | `LLAMA_ROPE_SCALING_TYPE_UNSPECIFIED` | Not specified    |
| `NONE`        | `LLAMA_ROPE_SCALING_TYPE_NONE`        | No scaling       |
| `LINEAR`      | `LLAMA_ROPE_SCALING_TYPE_LINEAR`      | Linear scaling   |
| `YARN`        | `LLAMA_ROPE_SCALING_TYPE_YARN`        | YaRN scaling     |
| `LONGROPE`    | `LLAMA_ROPE_SCALING_TYPE_LONGROPE`    | LongRoPE scaling |

### PoolingType

Pooling type for embeddings.

| Value         | llama.cpp Value                  | Description        |
|---------------|----------------------------------|--------------------|
| `UNSPECIFIED` | `LLAMA_POOLING_TYPE_UNSPECIFIED` | Not specified      |
| `NONE`        | `LLAMA_POOLING_TYPE_NONE`        | No pooling         |
| `MEAN`        | `LLAMA_POOLING_TYPE_MEAN`        | Mean pooling       |
| `CLS`         | `LLAMA_POOLING_TYPE_CLS`         | CLS token pooling  |
| `LAST`        | `LLAMA_POOLING_TYPE_LAST`        | Last token pooling |
| `RANK`        | `LLAMA_POOLING_TYPE_RANK`        | Rank pooling       |

### AttentionType

Attention mechanism type.

| Value         | llama.cpp Value                    | Description                       |
|---------------|------------------------------------|-----------------------------------|
| `UNSPECIFIED` | `LLAMA_ATTENTION_TYPE_UNSPECIFIED` | Not specified                     |
| `CAUSAL`      | `LLAMA_ATTENTION_TYPE_CAUSAL`      | Causal attention (autoregressive) |
| `NON_CAUSAL`  | `LLAMA_ATTENTION_TYPE_NON_CAUSAL`  | Non-causal attention              |

### FlashAttnType

Flash attention configuration.

| Value      | llama.cpp Value | Description              |
|------------|-----------------|--------------------------|
| `AUTO`     | `-1`            | Automatic selection      |
| `DISABLED` | `0`             | Flash attention disabled |
| `ENABLED`  | `1`             | Flash attention enabled  |

---

## Data Structures

### TokenData

Token data with probability information.

| Field   | Type    | llama.cpp Field | Description     |
|---------|---------|-----------------|-----------------|
| `id`    | `Token` | `id`            | Token ID        |
| `logit` | `Float` | `logit`         | Log probability |
| `p`     | `Float` | `p`             | Probability     |

Maps to: `llama_token_data`

### TokenDataArray

Array of token data for sampling.

| Field      | Type             | llama.cpp Field | Description                 |
|------------|------------------|-----------------|-----------------------------|
| `data`     | `ptr[TokenData]` | `data`          | Pointer to token data array |
| `size`     | `ArchWord`       | `size`          | Number of elements          |
| `selected` | `Int[64]`        | `selected`      | Selected token index        |
| `sorted`   | `Bool`           | `sorted`        | Whether array is sorted     |

Maps to: `llama_token_data_array`

### ChatMessage

Chat message structure for chat templates.

| Field     | Type       | llama.cpp Field | Description                                  |
|-----------|------------|-----------------|----------------------------------------------|
| `role`    | `CharsPtr` | `role`          | Message role ("user", "assistant", "system") |
| `content` | `CharsPtr` | `content`       | Message content                              |

Maps to: `llama_chat_message`

### PerfContextData

Performance data for context operations.

| Field      | Type        | llama.cpp Field | Description                  |
|------------|-------------|-----------------|------------------------------|
| `tStartMs` | `Float[64]` | `t_start_ms`    | Start time in milliseconds   |
| `tLoadMs`  | `Float[64]` | `t_load_ms`     | Load time in milliseconds    |
| `tPEvalMs` | `Float[64]` | `t_p_eval_ms`   | Prompt evaluation time       |
| `tEvalMs`  | `Float[64]` | `t_eval_ms`     | Token evaluation time        |
| `nPEval`   | `Int`       | `n_p_eval`      | Number of prompt evaluations |
| `nEval`    | `Int`       | `n_eval`        | Number of token evaluations  |
| `nReused`  | `Int`       | `n_reused`      | Number of reused evaluations |

Maps to: `llama_perf_context_data`

### PerfSamplerData

Performance data for sampler operations.

| Field       | Type        | llama.cpp Field | Description                   |
|-------------|-------------|-----------------|-------------------------------|
| `tSampleMs` | `Float[64]` | `t_sample_ms`   | Sampling time in milliseconds |
| `nSample`   | `Int`       | `n_sample`      | Number of samples             |

Maps to: `llama_perf_sampler_data`

---

## Classes

### Model

Represents a loaded LLM model. Maps to `llama_model`.

#### Model.Params

Model loading parameters. Maps to `llama_model_params`.

**Fields:**

- `devices` (`ptr`) - Device list. Maps to `devices`.
- `tensorBuftOverrides` (`ref[array[TensorBuftOverride]]`) - Tensor buffer type overrides. Maps to `tensor_buft_overrides`.
- `nGpuLayers` (`Int`) - Number of layers to offload to GPU. Maps to `n_gpu_layers`.
- `splitMode` (`SplitMode`) - GPU split mode. Maps to `split_mode`.
- `mainGpu` (`Int`) - Main GPU index. Maps to `main_gpu`.
- `tensorSplit` (`ref[array[Float]]`) - Tensor split ratios. Maps to `tensor_split`.
- `progressCallback` (`ProgressCallback`) - Progress callback function. Maps to `progress_callback`.
- `progressCallbackUserData` (`ptr`) - User data for callback. Maps to `progress_callback_user_data`.
- `kvOverrides` (`ptr`) - KV cache overrides. Maps to `kv_overrides`.
- `vocabOnly` (`Bool`) - Load vocabulary only. Maps to `vocab_only`.
- `useMmap` (`Bool`) - Use memory mapping. Maps to `use_mmap`.
- `useDirectIo` (`Bool`) - Use direct I/O. Maps to `use_direct_io`.
- `useMlock` (`Bool`) - Lock memory. Maps to `use_mlock`.
- `checkTensors` (`Bool`) - Validate tensor data. Maps to `check_tensors`.
- `useExtraBufts` (`Bool`) - Use extra buffer types. Maps to `use_extra_bufts`.
- `noHost` (`Bool`) - No host buffer allocation. Maps to `no_host`.
- `noAlloc` (`Bool`) - No memory allocation. Maps to `no_alloc`.

#### Model Functions

- `Model.getDefaultParams(): Params`
  Get default model parameters. Maps to `llama_model_default_params`.

- `Model.load(path: CharsPtr, params: Params): ref[Model]`
  Load model from file. Maps to `llama_model_load_from_file`.

- `Model.loadSplits(paths: ref[array[CharsPtr]], count: Word, params: Params): ref[Model]`
  Load model from split files. Maps to `llama_model_load_from_splits`.

- `Model.free(model: ref[Model])`
  Free model resources. Maps to `llama_model_free`.

#### Model Methods

- `model.save(path: CharsPtr)`
  Save model to file. Maps to `llama_model_save_to_file`.

- `model.hasEncoder: Bool`
  Check if model has encoder. Maps to `llama_model_has_encoder`.

- `model.hasDencoder: Bool`
  Check if model has decoder. Maps to `llama_model_has_dencoder`.

- `model.nCtxTrain: Int`
  Training context size. Maps to `llama_model_n_ctx_train`.

- `model.nEmbd: Int`
  Embedding dimension. Maps to `llama_model_n_embd`.

- `model.nLayer: Int`
  Number of layers. Maps to `llama_model_n_layer`.

- `model.nHead: Int`
  Number of attention heads. Maps to `llama_model_n_head`.

- `model.vocab: ref[Vocab]`
  Get vocabulary. Maps to `llama_model_get_vocab`.

- `model.metaValStr(key: CharsPtr, buf: CharsPtr, bufSize: ArchWord): Int`
  Get metadata value as string. Maps to `llama_model_meta_val_str`.

- `model.metaCount: Int`
  Get metadata count. Maps to `llama_model_meta_count`.

- `model.metaKeyByIndex(idx: Int, buf: CharsPtr, bufSize: ArchWord): Int`
  Get metadata key by index. Maps to `llama_model_meta_key_by_index`.

---

### Context

Inference context for a model. Maps to `llama_context`.

#### Context.Params

Context creation parameters. Maps to `llama_context_params`.

**Fields:**

- `nCtx` (`Word`) - Context size (0 = use model default). Maps to `n_ctx`.
- `nBatch` (`Word`) - Logical batch size. Maps to `n_batch`.
- `nUbatch` (`Word`) - Physical batch size. Maps to `n_ubatch`.
- `nSeqMax` (`Word`) - Maximum sequences. Maps to `n_seq_max`.
- `nThreads` (`Int`) - Number of threads for generation. Maps to `n_threads`.
- `nThreadsBatch` (`Int`) - Number of threads for batch processing. Maps to `n_threads_batch`.
- `ropeScalingType` (`RopeScalingType`) - RoPE scaling type. Maps to `rope_scaling_type`.
- `poolingType` (`PoolingType`) - Pooling type. Maps to `pooling_type`.
- `attentionType` (`AttentionType`) - Attention type. Maps to `attention_type`.
- `flashAttnType` (`FlashAttnType`) - Flash attention setting. Maps to `flash_attn`.
- `ropeFreqBase` (`Float`) - RoPE base frequency. Maps to `rope_freq_base`.
- `ropeFreqScale` (`Float`) - RoPE frequency scale. Maps to `rope_freq_scale`.
- `yarnExtFactor` (`Float`) - YaRN extrapolation factor. Maps to `yarn_ext_factor`.
- `yarnAttnFactor` (`Float`) - YaRN attention factor. Maps to `yarn_attn_factor`.
- `yarnBetaFast` (`Float`) - YaRN beta fast. Maps to `yarn_beta_fast`.
- `yarnBetaSlow` (`Float`) - YaRN beta slow. Maps to `yarn_beta_slow`.
- `yarnOrigCtx` (`Word`) - YaRN original context size. Maps to `yarn_orig_ctx`.
- `defragThold` (`Float`) - Defragmentation threshold. Maps to `defrag_thold`.
- `cbEval` (`function ptr`) - Evaluation callback. Maps to `cb_eval`.
- `cbEvalData` (`ptr`) - Evaluation callback user data. Maps to `cb_eval_user_data`.
- `typeK` (`Ggml.Type`) - K cache data type. Maps to `type_k`.
- `typeV` (`Ggml.Type`) - V cache data type. Maps to `type_v`.
- `abortCb` (`function ptr`) - Abort callback. Maps to `abort_callback`.
- `abortCbData` (`ptr`) - Abort callback user data. Maps to `abort_callback_data`.
- `embeddings` (`Bool`) - Enable embeddings output. Maps to `embeddings`.
- `offloadKqv` (`Bool`) - Offload KQV to GPU. Maps to `offload_kqv`.
- `noPerf` (`Bool`) - Disable performance counters. Maps to `no_perf`.
- `opOffload` (`Bool`) - Enable operation offloading. Maps to `op_offload`.
- `swaFull` (`Bool`) - Full sliding window attention. Maps to `swa_full`.
- `kvUnified` (`Bool`) - Unified KV cache. Maps to `kv_unified`.
- `samplers` (`ref[array[SamplerSeqConfig]]`) - Per-sequence samplers. Maps to `samplers`.
- `nSamplers` (`ArchWord`) - Number of samplers. Maps to `n_samplers`.

#### Context Functions

- `Context.getDefaultParams(): Params`
  Get default context parameters. Maps to `llama_context_default_params`.

- `Context.initFromModel(model: ref[Model], params: Params): ref[Context]`
  Create context from model. Maps to `llama_init_from_model`.

- `Context.free(ctx: ref[Context])`
  Free context resources. Maps to `llama_free`.

#### Context Methods

- `ctx.attachThreadpool(tp: ref[Ggml.Threadpool], tpBatch: ref[Ggml.Threadpool])`
  Attach thread pool. Maps to `llama_attach_threadpool`.

- `ctx.detachThreadpool()`
  Detach thread pool. Maps to `llama_detach_threadpool`.

- `ctx.getEmbeddingsSeq(seqId: SeqId): ref[array[Float]]`
  Get sequence embeddings. Maps to `llama_get_embeddings_seq`.

- `ctx.getModel(): ref[Model]`
  Get associated model. Maps to `llama_get_model`.

- `ctx.nCtx: Word`
  Get context size. Maps to `llama_n_ctx`.

- `ctx.nBatch: Word`
  Get logical batch size. Maps to `llama_n_batch`.

- `ctx.nUbatch: Word`
  Get physical batch size. Maps to `llama_n_ubatch`.

- `ctx.nSeqMax: Word`
  Get max sequences. Maps to `llama_n_seq_max`.

- `ctx.getMemory(): ptr[Memory]`
  Get KV cache memory. Maps to `llama_get_memory`.

- `ctx.poolingType: PoolingType`
  Get pooling type. Maps to `llama_pooling_type`.

- `ctx.encode(batch: Batch): Int`
  Encode batch (for encoder models). Maps to `llama_encode`.

- `ctx.decode(batch: Batch): Int`
  Decode batch. Maps to `llama_decode`.

- `ctx.getLogitsIth(i: Int): ref[array[Float]]`
  Get logits for token at index. Maps to `llama_get_logits_ith`.

- `ctx.getEmbeddingsIth(i: Int): ref[array[Float]]`
  Get embeddings for token at index. Maps to `llama_get_embeddings_ith`.

- `ctx.stateSize: ArchWord`
  Get state size in bytes. Maps to `llama_state_get_size`.

- `ctx.stateGetData(dst: ref[array[Word[8]]], size: ArchWord): ArchWord`
  Get state data. Maps to `llama_state_get_data`.

- `ctx.stateSetData(src: ref[array[Word[8]]], size: ArchWord): ArchWord`
  Set state data. Maps to `llama_state_set_data`.

- `ctx.stateLoadFile(path: CharsPtr, tokensOut: ref[array[Token]], cap: ArchWord, outCount: ref[ArchWord]): Bool`
  Load state from file. Maps to `llama_state_load_file`.

- `ctx.stateSaveFile(path: CharsPtr, tokens: ref[array[Token]], count: ArchWord): Bool`
  Save state to file. Maps to `llama_state_save_file`.

- `ctx.setAdapterLora(ad: ref[Adapter], scale: Float): Int`
  Apply LoRA adapter. Maps to `llama_set_adapter_lora`.

- `ctx.removeAdapterLora(ad: ref[Adapter]): Int`
  Remove LoRA adapter. Maps to `llama_rm_adapter_lora`.

- `ctx.clearAdapterLora()`
  Clear all LoRA adapters. Maps to `llama_clear_adapter_lora`.

- `ctx.perfContext: PerfContextData`
  Get performance data. Maps to `llama_perf_context`.

- `ctx.perfContextPrint()`
  Print performance data. Maps to `llama_perf_context_print`.

- `ctx.perfContextReset()`
  Reset performance counters. Maps to `llama_perf_context_reset`.

---

### Batch

Token batch for processing. Maps to `llama_batch`.

**Fields:**

- `nTokens` (`Int`) - Number of tokens. Maps to `n_tokens`.
- `token` (`ref[array[Token]]`) - Token IDs. Maps to `token`.
- `embd` (`ref[array[Float]]`) - Embeddings (alternative to tokens). Maps to `embd`.
- `pos` (`ref[Pos]`) - Token positions. Maps to `pos`.
- `nSeqId` (`ref[Int]`) - Number of sequence IDs per token. Maps to `n_seq_id`.
- `seqId` (`ref[ref[SeqId]]`) - Sequence IDs. Maps to `seq_id`.
- `output` (`ref[Int[8]]`) - Output flags. Maps to `logits`.

#### Batch Functions

- `Batch.getOne(tokens: ref[array[Token]], nTokens: Int): Batch`
  Create batch from token array. Maps to `llama_batch_get_one`.

- `Batch.init(nTokens: Int, embd: Int, nSeqMax: Int): Batch`
  Initialize empty batch. Maps to `llama_batch_init`.

---

### Sampler

Token sampler for generation. Maps to `llama_sampler`.

#### Sampler.ChainParams

Sampler chain parameters. Maps to `llama_sampler_chain_params`.

**Fields:**

- `noPerf` (`Bool`) - Disable performance counters. Maps to `no_perf`.

#### Sampler Functions

- `Sampler.init(iface: ptr, ctx: ptr): ref[Sampler]`
  Initialize custom sampler. Maps to `llama_sampler_init`.

- `Sampler.chainInit(params: ChainParams): ref[Sampler]`
  Create sampler chain. Maps to `llama_sampler_chain_init`.

- `Sampler.initGreedy(): ref[Sampler]`
  Create greedy sampler. Maps to `llama_sampler_init_greedy`.

- `Sampler.initDist(seed: Word): ref[Sampler]`
  Create distribution sampler. Maps to `llama_sampler_init_dist`.

- `Sampler.initTopK(k: Int): ref[Sampler]`
  Create top-k sampler. Maps to `llama_sampler_init_top_k`.

- `Sampler.initTopP(p: Float, minKeep: ArchWord): ref[Sampler]`
  Create top-p (nucleus) sampler. Maps to `llama_sampler_init_top_p`.

- `Sampler.initPenalties(penaltyLastN: Int, penaltyRepeat: Float, penaltyFreq: Float, penaltyPresent: Float): ref[Sampler]`
  Create penalty sampler. Maps to `llama_sampler_init_penalties`.
  - `penaltyLastN`: Last n tokens to penalize (0 = disable, -1 = context size)
  - `penaltyRepeat`: Repeat penalty (1.0 = disabled)
  - `penaltyFreq`: Frequency penalty (0.0 = disabled)
  - `penaltyPresent`: Presence penalty (0.0 = disabled)

- `Sampler.clone(s: ref[Sampler]): ref[Sampler]`
  Clone sampler. Maps to `llama_sampler_clone`.

- `Sampler.free(s: ref[Sampler])`
  Free sampler resources. Maps to `llama_sampler_free`.

#### Sampler Methods

- `sampler.reset()`
  Reset sampler state. Maps to `llama_sampler_reset`.

- `sampler.sample(ctx: ref[Context], idx: Int): Token`
  Sample next token. Maps to `llama_sampler_sample`.

- `sampler.name: CharsPtr`
  Get sampler name. Maps to `llama_sampler_name`.

- `sampler.accept(token: Token)`
  Accept token for tracking. Maps to `llama_sampler_accept`.

- `sampler.apply(cand: ref[TokenDataArray])`
  Apply sampler to candidates. Maps to `llama_sampler_apply`.

- `sampler.chainAdd(s: ref[Sampler])`
  Add sampler to chain. Maps to `llama_sampler_chain_add`.

- `sampler.chainGet(idx: Int): ref[Sampler]`
  Get sampler from chain. Maps to `llama_sampler_chain_get`.

- `sampler.chainCount: Int`
  Get chain length. Maps to `llama_sampler_chain_n`.

- `sampler.chainRemove(idx: Int): ref[Sampler]`
  Remove sampler from chain. Maps to `llama_sampler_chain_remove`.

- `sampler.perfSampler: PerfSamplerData`
  Get performance data. Maps to `llama_perf_sampler`.

- `sampler.perfSamplerPrint()`
  Print performance data. Maps to `llama_perf_sampler_print`.

- `sampler.perfSamplerReset()`
  Reset performance counters. Maps to `llama_perf_sampler_reset`.

---

### Vocab

Model vocabulary. Maps to `llama_vocab`.

#### Vocab Methods

- `vocab.tokenToPiece(token: Token, buf: CharsPtr, length: Int, lstrip: Int, special: Bool): Int`
  Convert token to text. Maps to `llama_token_to_piece`.

- `vocab.eos: Token`
  Get end-of-sequence token. Maps to `llama_vocab_eos`.

- `vocab.isEog(token: Token): Bool`
  Check if token is end-of-generation. Maps to `llama_vocab_is_eog`.

---

### Memory

KV cache memory management. Maps to `llama_memory`.

#### Memory Methods

- `memory.clear(data: Bool)`
  Clear KV cache. Maps to `llama_memory_clear`.

- `memory.seqRemove(seq: SeqId, p0: Pos, p1: Pos): Bool`
  Remove sequence range. Maps to `llama_memory_seq_rm`.

- `memory.seqCopy(src: SeqId, dst: SeqId, p0: Pos, p1: Pos)`
  Copy sequence range. Maps to `llama_memory_seq_cp`.

- `memory.seqKeep(seq: SeqId)`
  Keep only specified sequence. Maps to `llama_memory_seq_keep`.

- `memory.seqAdd(seq: SeqId, p0: Pos, p1: Pos, delta: Pos)`
  Add position delta to sequence. Maps to `llama_memory_seq_add`.

- `memory.seqDiv(seq: SeqId, p0: Pos, p1: Pos, d: Int)`
  Divide positions in sequence. Maps to `llama_memory_seq_div`.

---

### Adapter

LoRA adapter support. Maps to `llama_adapter_lora`.

#### Adapter Functions

- `Adapter.loraInit(model: ref[Model], path: CharsPtr): ref[Adapter]`
  Load LoRA adapter. Maps to `llama_adapter_lora_init`.

- `Adapter.loraFree(ad: ref[Adapter])`
  Free LoRA adapter. Maps to `llama_adapter_lora_free`.

---

## Global Functions

### Backend Management

- `backendInit()`
  Initialize llama backend. Maps to `llama_backend_init`.

- `backendFree()`
  Free llama backend. Maps to `llama_backend_free`.

- `numaInit(strategy: Ggml.NumaStrategy)`
  Initialize NUMA. Maps to `llama_numa_init`.

### System Information

- `timeUs(): Int[64]`
  Get current time in microseconds. Maps to `llama_time_us`.

- `maxDevices(): ArchWord`
  Get maximum number of devices. Maps to `llama_max_devices`.

- `maxParallelSequences(): ArchWord`
  Get max parallel sequences. Maps to `llama_max_parallel_sequences`.

- `supportsMmap(): Bool`
  Check mmap support. Maps to `llama_supports_mmap`.

- `supportsMlock(): Bool`
  Check mlock support. Maps to `llama_supports_mlock`.

- `supportsGpuOffload(): Bool`
  Check GPU offload support. Maps to `llama_supports_gpu_offload`.

- `supportsRpc(): Bool`
  Check RPC support. Maps to `llama_supports_rpc`.

- `printSystemInfo(): CharsPtr`
  Get system info string. Maps to `llama_print_system_info`.

### Tokenization

- `tokenize(vocab: ref[Vocab], text: CharsPtr, textLen: Int, tokens: ref[array[Token]], nTokensMax: Int, addSpecial: Bool, parseSpecial: Bool): Int`
  Convert text to tokens. Returns the number of tokens written. Maps to `llama_tokenize`.

- `detokenize(vocab: ref[Vocab], tokens: ref[array[Token]], nTokens: Int, text: CharsPtr, textLenMax: Int, removeSpecial: Bool, unparseSpecial: Bool): Int`
  Convert tokens to text. Returns the number of characters written. Maps to `llama_detokenize`.

### Chat Templates

- `chatApplyTemplate(tmpl: CharsPtr, chat: ref[array[ChatMessage]], nMsg: ArchWord, addAssistant: Bool, buf: CharsPtr, bufLen: Int): Int`
  Apply chat template to messages. Pass `0` for `tmpl` to use the model's default template. Returns the number of characters written. Maps to `llama_chat_apply_template`.

- `chatBuiltinTemplates(out: ref[CharsPtr], len: ArchWord): Int`
  Get built-in template names. Maps to `llama_chat_builtin_templates`.

### Logging

- `logSet(cb: ptr[function (level: Int, text: CharsPtr, userData: ptr)], userData: ptr)`
  Set logging callback. Maps to `llama_log_set`.

---

## Examples

### Text Completion

See `Examples/completion.alusus` for a complete text completion example.

### Chat

See `Examples/chat.alusus` for a multi-turn chat example with chat templates.

### Basic Usage Pattern

```alusus
import "Apm";
Apm.importFile("Alusus/Llama");
use Llama;

// 1. Initialize backend
Ggml.Backend.cpuLoad();

// 2. Load model
def modelParams: Model.Params = Model.getDefaultParams();
modelParams.nGpuLayers = 0;  // CPU only
def model: ref[Model](Model.load("model.gguf", modelParams));

// 3. Create context
def ctxParams: Context.Params = Context.getDefaultParams();
ctxParams.nCtx = 2048;
def ctx: ref[Context](Context.initFromModel(model, ctxParams));

// 4. Create sampler chain
def chainParams: Sampler.ChainParams;
chainParams.noPerf = true;
def sampler: ref[Sampler](Sampler.chainInit(chainParams));
sampler.chainAdd(Sampler.initPenalties(64, 1.1, 0.0, 0.0));
sampler.chainAdd(Sampler.initTopK(40));
sampler.chainAdd(Sampler.initTopP(0.9, 1));
sampler.chainAdd(Sampler.initDist(0));

// 5. Tokenize and decode prompt
def tokens: array[Token, 512];
def nTokens: Int = tokenize(model.vocab, "Hello world", 11, tokens, 512, true, true);
def batch: Batch = Batch.getOne(tokens, nTokens);
ctx.decode(batch);

// 6. Generate tokens
while true {
    def id: Token = sampler.sample(ctx, -1);
    if model.vocab.isEog(id) break;

    // Convert token to text and print
    def buf: array[Char, 64];
    detokenize(model.vocab, id~ptr~cast[ref[array[Token]]], 1, buf~ptr, 64, 0, 0);
    // ... print buf

    // Decode next token
    def nextBatch: Batch = Batch.getOne(id~ptr~cast[ref[array[Token]]], 1);
    ctx.decode(nextBatch);
}

// 7. Cleanup
Sampler.free(sampler);
Context.free(ctx);
Model.free(model);
```

---

## License

This library follows the same license as llama.cpp (MIT License).
