# Download Strategies Explained

## The Problem

Some datasets are HUGE:
- OpenHermes: 1M examples (~5-10GB)
- Bagel: 1.6M examples (~10-20GB)

But you only want the **creative/romance subset** (~5-10% of total).

## Three Approaches

### 1️⃣ Download Full → Filter After
```bash
./download_all.sh
python filter_openhermes.py
```

**Pros:**
- Simple, straightforward
- Can re-filter with different criteria later
- Have full dataset if you need it

**Cons:**
- Downloads 10-20GB then discards 90%
- Wastes disk space temporarily
- Wastes bandwidth
- Slower overall

**Disk usage:**
- During: 20-40GB (full datasets)
- After filtering: 2-5GB (creative only)

---

### 2️⃣ Stream & Filter During Download
```bash
python filter_during_download.py --dataset openhermes --max-examples 50000
```

**Pros:**
- ✅ Only downloads what you need
- ✅ Saves 90%+ disk space
- ✅ Saves bandwidth
- ✅ Often faster (less to download)
- ✅ No post-processing needed

**Cons:**
- Can't re-filter later (would need to re-download)
- Slightly more complex

**Disk usage:**
- During: 1-2GB (streaming)
- After: 1-2GB (only creative subset)

---

### 3️⃣ Download Starter Pack Only
```bash
./download_romance_starter_pack.sh
python filter_openhermes.py
```

**Pros:**
- Good balance
- Gets you started quickly
- Focuses on romance-relevant datasets

**Cons:**
- Still downloads full OpenHermes
- Missing some potentially useful datasets

**Disk usage:**
- During: 5-10GB
- After filtering: 2-3GB

---

## Recommendations by Use Case

### "I have limited disk space (<10GB free)"
→ Use **Stream & Filter** (Option 2)
```bash
python filter_during_download.py --dataset openhermes --max-examples 50000
huggingface-cli download grimulkan/LimaRP-augmented --local-dir ../romance-focused/limarp-augmented
huggingface-cli download jondurbin/gutenberg-dpo-v0.1 --local-dir ../human-written/gutenberg-dpo
```

### "I have plenty of space (>50GB free)"
→ Use **Download All** (Option 1)
```bash
./download_all.sh
python filter_openhermes.py
```
Then you have everything for experiments.

### "I want to start training ASAP"
→ Use **Starter Pack** (Option 3)
```bash
./download_romance_starter_pack.sh
python filter_openhermes.py
```

### "I only care about romance, nothing else"
→ Use **Stream & Filter** with targeted downloads
```bash
# Only romance-relevant datasets
python filter_during_download.py --dataset openhermes --max-examples 30000
huggingface-cli download grimulkan/LimaRP-augmented --local-dir ../romance-focused/limarp-augmented
huggingface-cli download jondurbin/gutenberg-dpo-v0.1 --local-dir ../human-written/gutenberg-dpo
```

---

## How Streaming Works

Traditional download:
```
Download → Save to disk → Load from disk → Filter → Save filtered version
   |            |              |             |             |
 10GB          10GB           10GB          1GB          1GB
```

Streaming download:
```
Download → Filter → Save filtered version
   |          |            |
 Stream     Process        1GB
```

Only the filtered examples ever touch your disk!

---

## Example: OpenHermes

**Full dataset:**
- 1,000,000 examples
- ~10GB disk space
- Download time: 20-60 minutes

**Creative subset (what you actually want):**
- ~50,000-100,000 examples (~5-10% of full)
- ~500MB-1GB disk space
- Download time: 5-15 minutes (streaming mode)

**Savings:**
- Disk: 90% less space
- Time: 75% faster
- Bandwidth: 90% less

---

## Script Reference

### Download All (Full datasets)
```bash
./download_all.sh
```
Downloads: All 8 datasets
Size: 20-40GB
Filter after: Yes, for large datasets

### Download Starter Pack (Recommended 3)
```bash
./download_romance_starter_pack.sh
```
Downloads: Gutenberg + LimaRP + OpenHermes
Size: 5-10GB
Filter after: Yes, for OpenHermes

### Stream & Filter (Smart download)
```bash
# OpenHermes creative only
python filter_during_download.py --dataset openhermes --max-examples 50000

# Bagel creative only
python filter_during_download.py --dataset bagel --max-examples 100000

# Airoboros creative only
python filter_during_download.py --dataset airoboros --max-examples 30000
```
Downloads: Only creative subset
Size: ~1-2GB total
Filter after: No (already done)

### Individual Downloads
```bash
./download_individual.sh
```
Interactive menu to choose specific datasets

---

## After Downloading

### If you downloaded full datasets:
```bash
# Filter to creative subset
python filter_openhermes.py

# Or use the post-download filter
python filter_after_download.py --dataset openhermes
```

### If you used streaming:
You're done! Datasets are already filtered and ready to use.

### Verify what you have:
```bash
python verify_datasets.py
```

---

## Disk Space Breakdown

### All 8 datasets (full):
- OpenHermes: 10GB
- Bagel: 20GB
- WritingPrompts: 5GB
- Airoboros: 3GB
- Gutenberg: 0.1GB
- LimaRP: 0.05GB
- LIMA: 0.01GB
- Opus: varies
**Total: ~38GB**

### All 8 datasets (creative subsets only):
- OpenHermes (creative): 1GB
- Bagel (creative): 2GB
- WritingPrompts: 5GB (already creative)
- Airoboros (creative): 0.5GB
- Gutenberg: 0.1GB
- LimaRP: 0.05GB
- LIMA: 0.01GB
- Opus: varies
**Total: ~9GB**

### Recommended starter (filtered):
- Gutenberg: 0.1GB
- LimaRP: 0.05GB
- OpenHermes (creative): 1GB
**Total: ~1.2GB**

---

## FAQ

**Q: Can I download full dataset now, filter later?**
A: Yes! Use `./download_all.sh`, then run filters whenever.

**Q: Can I re-filter if I used streaming mode?**
A: No, you'd need to re-download. If you might want to experiment with different filters, download full datasets.

**Q: Which is faster overall?**
A: Streaming is faster (less to download). But if you have fast internet, difference is minimal.

**Q: Can I pause and resume?**
A: `huggingface-cli download` supports resume. Streaming mode does not.

**Q: What if I run out of space mid-download?**
A: Downloads will fail. Use streaming mode or free up space first.

**Q: Can I delete full datasets after filtering?**
A: Yes! After running filter scripts, you can delete the full versions if you only need creative subsets.

---

## Recommended Workflow

For most users:
1. Start with streaming mode for large datasets
2. Download small datasets normally
3. If you later need full datasets, download them then

```bash
# Day 1: Get started quick (streaming)
cd scripts/
python filter_during_download.py --dataset openhermes --max-examples 50000
huggingface-cli download grimulkan/LimaRP-augmented --local-dir ../romance-focused/limarp-augmented
huggingface-cli download jondurbin/gutenberg-dpo-v0.1 --local-dir ../human-written/gutenberg-dpo

# Start training and experimenting...

# Day 30: If you need more data
./download_all.sh  # Get everything else
```

This approach:
- Gets you training in <30 minutes
- Uses minimal disk space initially
- Lets you expand later if needed
