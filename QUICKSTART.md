# Quick Start Guide - Romance Writing Fine-Tuning

## 🎯 Goal
Get high-quality training data for romance/creative writing fine-tuning, avoiding the failures of previous synthetic data attempts.

## ⚡ Fastest Path to Success

### Step 1: Install Prerequisites (2 minutes)
```bash
# Install HuggingFace CLI
pip install -U "huggingface_hub[cli]"

# Optional: Login if you need gated datasets
huggingface-cli login
```

### Step 2: Choose Your Download Strategy

**Option A: Download ALL datasets (have space? do this!)**
```bash
cd scripts/
./download_all.sh
```
- Gets all 8 datasets ready for experiments
- Prompts before large downloads
- **Space:** 20-40GB | **Time:** 30min-2hrs

**Option B: Just the romance essentials (faster)**
```bash
cd scripts/
./download_romance_starter_pack.sh
```
- Downloads Gutenberg + LimaRP + OpenHermes
- **Space:** 5-10GB | **Time:** 15-30min

**Option C: Stream & filter (save 90% disk space)**
```bash
cd scripts/

# Only download creative examples (not full dataset)
python filter_during_download.py --dataset openhermes --max-examples 50000
python filter_during_download.py --dataset bagel --max-examples 100000

# Get small datasets normally
huggingface-cli download GAIR/lima --repo-type dataset --local-dir ../small-curated/lima
huggingface-cli download grimulkan/LimaRP-augmented --repo-type dataset --local-dir ../romance-focused/limarp-augmented
```
- **Space:** 1-5GB | **Time:** 15-30min
- Downloads only creative subset (no filtering after)

### Step 3: Filter OpenHermes (5 minutes)
```bash
# Optional but recommended - reduce OpenHermes to creative subset
cd scripts/
python filter_openhermes.py
```

This reduces 1M examples → ~50-100K creative writing examples.

### Step 4: Verify Downloads
```bash
cd ..
tree -L 2
```

You should see:
```
external-training-datasets/
├── human-written/
│   └── gutenberg-dpo/
├── romance-focused/
│   └── limarp-augmented/
├── synthetic-high-quality/
│   ├── openhermes-2.5/
│   └── openhermes-creative-only/
└── scripts/
```

---

## 📊 What You Get

| Dataset | Type | Size | Purpose |
|---------|------|------|---------|
| Gutenberg-DPO | Human | ~17 books | Baseline human prose quality |
| LimaRP | Romance | ~2K | Romance writing style |
| OpenHermes (filtered) | Synthetic | ~50K | Creative instruction following |

**Total Training Set:** ~52K high-quality examples
**Expected Quality:** Significantly better than previous synthetic attempts

---

## 🚀 Next Steps

### Option A: Start Training Immediately
```python
from datasets import load_from_disk, concatenate_datasets

# Load all three datasets
gutenberg = load_from_disk("human-written/gutenberg-dpo/")
limarp = load_from_disk("romance-focused/limarp-augmented/")
openhermes = load_from_disk("synthetic-high-quality/openhermes-creative-only/")

# Combine (you'll need to standardize formats first)
# ... proceed with your SFT pipeline
```

### Option B: Start Small (Recommended)
```python
# Test with just LIMA-sized subset first
from datasets import load_from_disk

limarp = load_from_disk("romance-focused/limarp-augmented/")

# Use just 1K examples for first experiment
small_test = limarp.select(range(1000))

# Train and evaluate before scaling up
```

### Option C: Custom Combination
See `README.md` for:
- Individual dataset download scripts
- Advanced filtering techniques
- Format conversion examples

---

## 🔍 Quality Checks

Before training, verify:
1. ✅ Datasets are in correct format (SFT vs DPO)
2. ✅ Examples contain actual creative writing (not code/math)
3. ✅ Romance tone matches your target style
4. ✅ No obvious GPT-slop patterns

Sample a few examples:
```python
from datasets import load_from_disk

ds = load_from_disk("romance-focused/limarp-augmented/")
print(ds[0])  # Check format and quality
```

---

## 💡 Pro Tips

### Why This Works vs Previous Failures

❌ **Previous attempts likely used:**
- Pre-training data (not instruction pairs)
- Low-quality synthetic text
- General purpose data without creative focus

✅ **This combination provides:**
- Genuine human creative writing (Gutenberg)
- Romance-specific training (LimaRP)
- High-quality GPT-4 creative tasks (OpenHermes filtered)
- Proper SFT format

### Training Recommendations

1. **Start small:** Test with 1K examples first
2. **Validate quality:** Sample outputs after training
3. **Iterate:** Add more data if quality is good
4. **Monitor:** Watch for mode collapse or repetition

### Compute Budgets

- **Low (1 GPU, few hours):** LIMA (1K) + LimaRP (2K)
- **Medium (1-2 GPU, 1 day):** All three filtered (~50K)
- **High (Multi-GPU, days):** Full Bagel creative subset (100K+)

---

## 🆘 Troubleshooting

### Download fails
```bash
# Try with authentication
huggingface-cli login

# Or download directly via Python
python -c "from datasets import load_dataset; load_dataset('teknium/OpenHermes-2.5')"
```

### Filtering takes too long
```bash
# Use smaller sample first
python -c "
from datasets import load_dataset
ds = load_dataset('teknium/OpenHermes-2.5', split='train[:10000]')
# ... filter this subset
"
```

### Format incompatibility
See `scripts/format_converter.py` (create this if needed) for:
- ShareGPT → Alpaca format
- DPO → SFT format
- Custom format conversion

---

## 📚 References

- Full dataset documentation: `README.md`
- Dataset comparison matrix: `README.md#dataset-comparison-matrix`
- Individual download scripts: `scripts/download_*.sh`

---

**Estimated Time to First Training Run:** ~30 minutes
**Recommended First Experiment:** LimaRP-only (2K examples, romance-focused)
