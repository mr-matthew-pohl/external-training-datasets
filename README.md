# High-Quality Pre-Made Fine-Tuning Datasets for Creative & Romance Writing

This directory contains curated, ready-to-use SFT datasets for training models on creative fiction and romance writing.

## 📁 Directory Structure

```
external-training-datasets/
├── human-written/          # Datasets with genuine human creative writing
├── synthetic-high-quality/ # GPT-4 generated instruction datasets
├── romance-focused/        # Romance and roleplay specific datasets
├── small-curated/         # Small but extremely high-quality datasets
├── scripts/               # Download and processing scripts
└── README.md              # This file
```

## 🎯 Recommended Starter Combination

For romance writing with past synthetic data failures, start with:

1. **Gutenberg-DPO** → Human-quality prose baseline
2. **LimaRP-augmented** → Romance-specific style
3. **OpenHermes-2.5** (creative subset) → Broad instruction following

**Total:** ~20K-100K examples (depending on filtering)
**Quality:** Mix of genuine human fiction + high-quality synthetic
**Format:** Ready for SFT pipelines

---

## 📚 Dataset Catalog

### 🏆 Top Priority Datasets

#### 1. OpenHermes-2.5
**Category:** Synthetic High-Quality
**Size:** ~1 million instruction-response pairs
**Format:** ShareGPT format
**Quality:** GPT-4 generated from 14+ curated sources

**Strengths:**
- Includes dedicated creative writing categories
- Proven SOTA results when fine-tuned
- Roleplay and storytelling tasks included
- Ready for immediate SFT use

**Download:**
```bash
huggingface-cli download teknium/OpenHermes-2.5 --repo-type dataset
```

**HF Link:** https://huggingface.co/datasets/teknium/OpenHermes-2.5

---

#### 2. Gutenberg-DPO
**Category:** Human-Written
**Size:** 17 classic novels parsed into chapters
**Format:** DPO pairs (prompt, chosen=human, rejected=LLM)
**Quality:** ⭐⭐⭐⭐⭐ Genuine published fiction

**Strengths:**
- ACTUAL human-written chapters from classics
- Includes Pride & Prejudice, Huckleberry Finn, Frankenstein
- "Chosen" responses are real published prose
- Perfect baseline for human-quality fiction

**Download:**
```bash
huggingface-cli download jondurbin/gutenberg-dpo-v0.1 --repo-type dataset
```

**HF Link:** https://huggingface.co/datasets/jondurbin/gutenberg-dpo-v0.1

**Note:** This is the dataset you need if previous synthetic attempts failed. The "chosen" examples are genuine human creative writing.

---

#### 3. LimaRP (augmented)
**Category:** Romance-Focused
**Size:** ~2,000 high-quality longform conversations
**Format:** Roleplay dialogue format
**Quality:** Curated for narrative quality

**Strengths:**
- **Specifically designed for romance-style writing**
- Longform roleplaying conversations
- High narrative quality focus
- Romantic elements throughout

**Download:**
```bash
huggingface-cli download grimulkan/LimaRP-augmented --repo-type dataset
```

**HF Link:** https://huggingface.co/datasets/grimulkan/LimaRP-augmented

**Perfect for:** Your romance writing focus

---

#### 4. Airoboros Dataset
**Category:** Synthetic High-Quality
**Size:** Multiple versions (varying sizes)
**Format:** Instruction-response pairs
**Quality:** GPT-4 generated with creative emphasis

**Strengths:**
- Dedicated creative writing tasks
- Storytelling and prose generation
- Models trained on it excel at creative tasks
- Natural prose generation

**Download:**
```bash
# Version 2.2 (stable)
huggingface-cli download jondurbin/airoboros-2.2 --repo-type dataset

# Version 3.1 (newer)
huggingface-cli download jondurbin/airoboros-3.1 --repo-type dataset
```

**HF Link:** https://huggingface.co/datasets/jondurbin/airoboros-2.2

---

#### 5. Opus-WritingPrompts
**Category:** Human-Written / Synthetic Mix
**Size:** 3,000 short stories (2k-6k chars each)
**Format:** Prompt + story pairs
**Quality:** Curated for creative fiction

**Strengths:**
- Pure creative fiction focus
- Curated specifically for creative writer/RP fine-tuning
- Story-focused content
- Includes some adult content (erotica)

**Download:**
```bash
# Check the Opus-Writer collection
# Multiple datasets available
```

**HF Collection:** https://huggingface.co/collections/Babsie/data-set-for-opus-writer

**Note:** Contains some NSFW content - filter if needed

---

#### 6. Bagel Dataset
**Category:** Comprehensive
**Size:** ~1.6 million deduplicated examples
**Format:** SFT instruction pairs
**Quality:** Composite of curated sources

**Strengths:**
- Includes Gutenberg DPO for fiction
- Multiple creative datasets combined
- Deliberately includes novel-writing data
- Comprehensive coverage

**Download:**
```bash
huggingface-cli download jondurbin/bagel --repo-type dataset
```

**HF Link:** https://huggingface.co/datasets/jondurbin/bagel

**Note:** Large comprehensive dataset - consider filtering to creative subset

---

### 🎁 Smaller High-Quality Options

#### 7. LIMA
**Category:** Small Curated
**Size:** ~1,000 examples
**Format:** Instruction-response
**Quality:** ⭐⭐⭐⭐⭐ Extremely high quality

**Strengths:**
- Proves quality >> quantity
- Models trained on just 1K examples rival larger datasets
- Minimal compute requirements
- Diverse high-quality examples

**Download:**
```bash
# LIMA dataset (check HuggingFace for exact path)
```

**Best for:** Low-compute experiments proving quality matters

---

#### 8. WritingPrompts
**Category:** Human-Written
**Size:** 300,000+ stories
**Format:** Prompt + story
**Quality:** Human-written Reddit stories

**Strengths:**
- Genuine human creative writing
- Reddit r/WritingPrompts community
- Large variety of creative fiction
- Story-prompt pairs ready for SFT

**Download:**
```bash
huggingface-cli download euclaise/writingprompts --repo-type dataset
```

**HF Link:** https://huggingface.co/datasets/euclaise/writingprompts

---

## 🚨 Why Your Previous Attempts Failed

Based on common failure patterns:

❌ **What went wrong:**
1. Synthetic pre-training data (not instruction-tuning pairs)
2. Low-quality or "GPT-slop"
3. General instruction data without creative writing focus

✅ **How these datasets fix it:**
1. Human-written creative text (Gutenberg, WritingPrompts)
2. High-quality GPT-4 generations (OpenHermes, Airoboros)
3. Pre-formatted instruction-response pairs
4. Dedicated creative/fiction writing examples

---

## 🎯 Recommended Workflows

### Workflow 1: Romance Focus (Your Use Case)
```bash
# Download core datasets
cd scripts/
./download_romance_starter_pack.sh

# This downloads:
# - Gutenberg-DPO (human prose baseline)
# - LimaRP-augmented (romance style)
# - OpenHermes-2.5 (creative subset)
```

### Workflow 2: Pure Human Writing
```bash
# Focus on genuine human creative writing
# - Gutenberg-DPO
# - WritingPrompts
# - LIMA (optional)
```

### Workflow 3: Comprehensive Creative
```bash
# Maximum coverage
# - Bagel (includes everything)
# - Filter to creative/fiction subset
```

---

## 📊 Dataset Comparison Matrix

| Dataset | Size | Human/Synthetic | Romance | Format | Quality |
|---------|------|-----------------|---------|--------|---------|
| Gutenberg-DPO | Small | 100% Human | Some | DPO | ⭐⭐⭐⭐⭐ |
| LimaRP | Small | Mixed | High | RP | ⭐⭐⭐⭐⭐ |
| OpenHermes | Large | GPT-4 | Medium | SFT | ⭐⭐⭐⭐ |
| Airoboros | Medium | GPT-4 | Low | SFT | ⭐⭐⭐⭐ |
| WritingPrompts | Large | Human | Low | SFT | ⭐⭐⭐ |
| LIMA | Tiny | Curated | None | SFT | ⭐⭐⭐⭐⭐ |
| Bagel | Huge | Mixed | Medium | SFT | ⭐⭐⭐⭐ |

---

## 🛠️ Usage Instructions

### Prerequisites
```bash
# Install HuggingFace CLI
pip install -U "huggingface_hub[cli]"

# Login to HuggingFace (some datasets require auth)
huggingface-cli login
```

### Download Strategy Options

You have **3 download strategies** depending on your needs:

#### Option 1: Download ALL 8 Datasets (Recommended)
```bash
cd scripts/
./download_all.sh
```
- Downloads all datasets with confirmation prompts
- Skip large datasets if needed
- **Time:** 30min - 2hrs | **Space:** 20-40GB
- **Best for:** Having everything ready to experiment

#### Option 2: Download Starter Pack (Romance Focus)
```bash
cd scripts/
./download_romance_starter_pack.sh
```
- Downloads just the 3 recommended datasets
- Gutenberg + LimaRP + OpenHermes
- **Time:** 15-30min | **Space:** 5-10GB
- **Best for:** Quick start with romance focus

#### Option 3: Stream & Filter (Save Disk Space)
```bash
cd scripts/

# Download ONLY creative subset during download
python filter_during_download.py --dataset openhermes --max-examples 50000
python filter_during_download.py --dataset bagel --max-examples 100000
```
- Only downloads creative examples (streaming mode)
- **Saves 90%+ disk space** for large datasets
- **Time:** Same or faster | **Space:** 1-5GB
- **Best for:** Limited disk space or only want creative subset

### Strategy Comparison

| Strategy | Datasets | Disk Space | Download Time | Filter After? |
|----------|----------|------------|---------------|---------------|
| Download All | All 8 | 20-40GB | 30min-2hrs | Yes (for large ones) |
| Starter Pack | 3 best | 5-10GB | 15-30min | Yes (OpenHermes) |
| Stream & Filter | Choose | 1-5GB | 15-30min | No (done during) |

### Quick Start (Choose Your Path)
```bash
# Path A: Download everything (you have space)
cd scripts/
./download_all.sh

# Path B: Just the romance essentials (faster)
cd scripts/
./download_romance_starter_pack.sh

# Path C: Smart filtering (save disk space)
cd scripts/
python filter_during_download.py --dataset openhermes --max-examples 50000
python filter_during_download.py --dataset bagel --max-examples 100000

# Download small datasets normally
huggingface-cli download GAIR/lima --repo-type dataset --local-dir ../small-curated/lima
huggingface-cli download grimulkan/LimaRP-augmented --repo-type dataset --local-dir ../romance-focused/limarp-augmented
```

### Filtering for Creative Content
```python
# Example: Filter OpenHermes for creative writing
from datasets import load_dataset

ds = load_dataset("teknium/OpenHermes-2.5")

# Filter for creative categories
creative_keywords = ["creative", "story", "writing", "fiction", "roleplay"]
creative_ds = ds.filter(lambda x: any(kw in x.get("category", "").lower() for kw in creative_keywords))

# Save filtered version
creative_ds.save_to_disk("./synthetic-high-quality/openhermes-creative-only/")
```

---

## 📈 Training Recommendations

### Recommended Training Order
1. **Start with Gutenberg-DPO** → Establish human prose baseline
2. **Add LimaRP** → Layer in romance/narrative style
3. **Mix in OpenHermes (creative subset)** → Broaden instruction following

### Size Recommendations
- **Low compute:** LIMA (1K) + LimaRP (2K) = 3K examples
- **Medium compute:** Gutenberg + LimaRP + OpenHermes (filtered) = 20-50K
- **High compute:** Bagel (creative subset) = 100K+

### Quality vs Quantity
Based on LIMA research:
- 1,000 high-quality examples > 10,000 low-quality
- Human-written > GPT-4 > GPT-3.5 > Synthetic
- Domain-specific (romance) > General creative > General instruction

---

## 🔗 External Resources

### HuggingFace Collections
- [Opus-Writer Dataset Collection](https://huggingface.co/collections/Babsie/data-set-for-opus-writer)
- [Awesome Instruction Datasets](https://github.com/jianzhnie/awesome-instruction-datasets)
- [LLM Data Hub](https://github.com/Zjh-819/LLMDataHub)

### Additional Research
- [DPO Datasets Overview](https://www.reddit.com/r/LocalLLaMA/comments/194v4h6/recent_dpo_datasets_overview/)
- [Best Datasets for Instruct Fine-tuning](https://www.reddit.com/r/LocalLLaMA/comments/18n74kh/best_dataset_for_instruct_finetuning/)
- [Quality SFT Data Discussion](https://toloka.ai/blog/the-genai-frontier-and-the-quest-for-high-quality-sft-data/)

---

## 📝 Notes

### Dataset Formats
- **SFT (Supervised Fine-Tuning):** Instruction-response pairs
- **DPO (Direct Preference Optimization):** Prompt + chosen + rejected examples
- **ShareGPT:** Multi-turn conversation format

### Quality Markers
- ⭐⭐⭐⭐⭐ = Human-written or meticulously curated
- ⭐⭐⭐⭐ = High-quality GPT-4 or curated synthetic
- ⭐⭐⭐ = Good quality but may need filtering

### Content Warnings
- Some datasets (Opus-WritingPrompts, certain Reddit data) contain adult/NSFW content
- Filter based on your use case
- Romance != Erotica (LimaRP is SFW romantic roleplay)

---

## 🚀 Next Steps

1. ✅ Choose your dataset combination based on compute/focus
2. ✅ Run download scripts from `scripts/` directory
3. ✅ Filter datasets to creative/romance subsets if needed
4. ✅ Verify dataset format matches your training pipeline
5. ✅ Start with small experiments (LIMA-sized) before scaling up

---

**Last Updated:** 2025-11-09
**Maintained By:** Dataset curation for romance/creative fine-tuning
**License:** Individual datasets have their own licenses - check before commercial use
