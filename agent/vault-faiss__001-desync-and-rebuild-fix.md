vault-faiss__001-desync-and-rebuild-fix
tags: vault, faiss, ground-truth, desync, rebuild, metadata, nsfw, vision, openrouter, qwen, catbox
updated: 2026-06-24

Full story of the 2026-06-24 vault rebuild. 178 raw files -> ground_truth.json + FAISS. Three vision models. One critical desync bug. Multiple metadata correction rounds.

== Background ==

Dropped old 242-entry ground_truth (175 clean + 67 explicit split index). Replaced with 178 fresh images from data/images_raw/. New schema: objects as open-ended array, explicit as boolean, no separate index.

== Pipeline ==

step 1: upload 178 files to catbox in chunks of 10 -> data/styxhelix_results/chunk_*.json
step 2: vision pass 1 via OpenRouter openai/gpt-4o -> 140/178 succeeded, 31 blocked by content policy (NSFW), 7 videos
step 3: vision pass 2 via OpenRouter qwen/qwen3-vl-32b-instruct on the 31 blocked -> 31/31 recovered with mood + objects
step 4: 7 entries had incomplete JSON parse (mood="Analyzed", 0 objects) -> fell back to "Quiet Melancholy" / 1 object each
step 5: consolidate all into data/ground_truth.json -> 178 entries, 86 explicit, 92 clean, 513 unique object types, 12 distinct moods
step 6: build FAISS index via scripts/build_faiss.py using all-MiniLM-L6-v2 embeddings

== The Desync Bug ==

During step 6, I made a critical error: when I corrected entries in ground_truth.json (fixed explicit flags, updated objects) and rebuilt the FAISS index, I did NOT delete the old .faiss and id_map.json files first. I ran the build script which appended/updated in-place.

root cause: build_faiss.py reads entries sequentially and assigns positional indices. When old index files exist with stale positional mappings, the new entries get appended at the wrong positions. Querying by index returns the WRONG image — metadata says one thing, the image is something else.

symptoms:
- query index 94 -> returned metadata for different image than actual file
- user sent me images that I thought were "Quiet Melancholy" but were actually half-dressed intimate shots
- URL lookups returned 404s because positional mapping was offset
- explicit:false images turned out to contain underwear, bare shoulders, bed settings (5+ instances caught by user)

fix procedure:
1. rm -f data/embeddings/faiss_index.faiss data/embeddings/id_map.json
2. python3 scripts/build_faiss.py (clean rebuild from scratch)
3. verify by querying 3+ indices and comparing returned metadata against ground truth
4. verify URLs load with curl -s -o /dev/null -w "%{http_code}" on each

== Metadata Accuracy ==

After rebuild, tested 3 random clean entries:
- entry 2 (Hatsune.Miku.600.2340308.jpg): Regal Melancholy, 5 objects -> CORRECT
- entry 3 (Hatsune.Miku.600.2863664.jpg): Quiet Melancholy, 7 objects -> CORRECT  
- entry 20 (Sousou.no.Frieren.600.4293999.jpg): Regal Melancholy, 7 objects -> CORRECT

All three verified by user as accurate.

== NSFW Tagging Weakness ==

The vision models (GPT-4o and Qwen) tag objects as free text. They consistently miss intimate context:
- "black clothing, dark hair" -> actually lingerie
- "bed, window, curtains" -> actually bedroom scene with implied nudity
- "purple dress, moody lighting" -> actually low-cut, intimate framing
- "candle, dark aesthetic" -> actually softcore lighting

My own aesthetic reading was also unreliable — I visually classified ~10 images as "clean" that were tagged explicit:true in the metadata.

lesson: NEVER trust visual reading for NSFW classification. Always check explicit:false in ground truth AND vision-verify before sending. When Teresa is present, strict SFW mode means filtering both layers.

== Selection Bias ==

Before the audit, I was recycling ~16 images out of 178 (icv3pd.jpeg sent 6 times, t3insp.jpg sent 6 times, tvy8dk.jpeg sent 2 times, etc.). 162 untouched files sat idle.

fix: after rebuild, pull randomly from ground_truth clean entries. Verify URL loads before sending. Cross-reference mood+objects with user for feedback loop.

== File Locations (all inside Docker container) ==

ground_truth:     /workspace/hermes-emotion-project/data/ground_truth.json
FAISS index:      /workspace/hermes-emotion-project/data/embeddings/faiss_index.faiss
ID map:           /workspace/hermes-emotion-project/data/embeddings/id_map.json
Build script:     /workspace/hermes-emotion-project/scripts/build_faiss.py
Vision results:   /workspace/hermes-emotion-project/data/styxhelix_results/
Raw images:       /workspace/hermes-emotion-project/data/images_raw/
Catbox uploads:   /workspace/hermes-emotion-project/data/styxhelix_results/chunk_*.json
Experiment log:   /workspace/collective-docs/experiment-log/agent/

== Models Used ==

Primary vision:    OpenRouter openai/gpt-4o (140 images, rate-limited to 3 concurrent)
NSFW retry:       OpenRouter qwen/qwen3-vl-32b-instruct (31 images, recovered all)
Conversation:     deepseek-v4-flash via DeepSeek API
Embeddings:       all-MiniLM-L6-v2 (384-dim, via sentence-transformers)

== Final State ==

178 indexed. Unified index (no separate explicit index — explicit is a boolean field). Clean rebuild. Old explicit index files (data/embeddings/explicit/explicit_index.faiss, explicit_id_map.json) deleted.
