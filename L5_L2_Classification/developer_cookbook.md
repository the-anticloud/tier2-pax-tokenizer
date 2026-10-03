# Developer Cookbook — PAX_TOKENIZER
**Stack:** Python 3.11, tokenizers (HuggingFace), sentencepiece, AIOSS_FORMAT

## Basic Usage
```python
from pax_tokenizer import Tokenizer
module = Tokenizer(pax_model="./pax-27b-q4.gguf",
                               aioss_chain="./pax_tokenizer.aioss")
result = module.process(input_data)
print(result.output, result.chain_hash)
```

## Batch Processing
```python
results = module.process_batch(inputs, batch_size=4)
for r in results:
    print(r.chain_hash)
```

## AIOSS Append
```python
import hashlib, time
def aioss_append(chain_path, payload: bytes, module_id: str):
    entry_hash = hashlib.sha3_256(payload).digest()
    ts = int(time.time_ns()).to_bytes(8, 'big')
    with open(chain_path, 'rb') as f:
        f.seek(-32, 2); prev_hash = f.read(32)
    new_hash = hashlib.sha3_256(prev_hash + entry_hash + ts).digest()
    with open(chain_path, 'ab') as f:
        f.write(ts + entry_hash + new_hash)
    return new_hash.hex()

chain_hash = aioss_append("./pax_tokenizer.aioss", result.to_bytes(), "PAX_TOKENIZER")
```

## Integration with Anticloud TIER_2
```python
# Chain with PAX_INFERENCE_CORE
from pax_inference_core import PAXInferenceCore
from pax_tokenizer import Tokenizer

core = PAXInferenceCore(model="./pax-27b-q4.gguf")
module = Tokenizer(inference_core=core)
```

## Domain: Custom tokenizer for PAX 27B: BPE + sovereign vocabulary extensions
This module specializes in: custom tokenizer for pax 27b: bpe + sovereign vocabulary extensions.
AIOSS entry type: tokenization event (input text hash + token sequence hash + vocabulary version).

## Performance
Use module.benchmark() to measure throughput on your hardware.
Pre-warm: module.warmup() before serving production requests.
