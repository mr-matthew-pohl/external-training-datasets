# Comprehensive Data Processing Pipeline with Universal Agentic Scoring
**Date:** 2025-11-10
**Purpose:** Process ALL data (clean + steamy) through chunking → agentic scoring → enriched metadata
**Goal:** Build highest quality, most diverse SFT dataset for steamy romance writing editor

---

## Pipeline Philosophy

**CRITICAL CHANGE:** Every single data source gets agentic scoring - no exceptions.

**Why Universal Agentic Scoring:**
1. **Quality Control:** AI agents identify best examples from all sources
2. **Rich Metadata:** Tag every chunk with romance/steamy/dialogue/craft/quality attributes
3. **Smart Assembly:** Build final dataset with perfect variety and balance
4. **Future Flexibility:** Scored metadata enables infinite dataset combinations
5. **Avoid Assumptions:** "Clean" data may have romance/steamy; "steamy" data may be low quality

**Output Format:** Every processed chunk gets a "scored" JSON file with:
- Original text
- Quality scores (0-100 across multiple dimensions)
- Content tags (romance, steamy, dialogue, craft, scene_construction, etc.)
- Recommendations (sft_tier_1, sft_tier_2, dpo_chosen, dpo_rejected, remove)

---

## Data Sources (ALL require processing)

### Total Raw Data: ~1.7 GB

| Category | Sources | Size | Count | Needs Chunking? |
|----------|---------|------|-------|-----------------|
| **Human Fiction** | WritingPrompts Parquet, Reddit WP | 1.04 GB | 655K | No (already story-sized) |
| **AO3 Romance** | AO3 Fiction (excellent/very_good/good) | 400 MB | 381 works | YES (chapter/scene splits) |
| **Opus Creative** | Opus-Instruct-25K, Claude-Opus-15K | 148 MB | 39K | No (conversation-sized) |
| **Steamy Content** | Mimi, Dirty-WritingPrompts | 269 MB | 42K | Extract pairs from Mimi |
| **General Instruct** | Kalo-Opus-22K, Airoboros-2.2 | 160 MB | 67K | No (instruction-sized) |
| **Craft Resources** | Romance guides | 1 MB | 25 | No (guide-sized) |

**After Chunking:** ~850K-900K chunks/examples to score

---

## Stage 1: Programmatic Chunking & Standardization

### 1.1 Chunk Long-Form Content

**AO3 Fiction (381 works → ~4,000-5,000 chunks):**

```python
def chunk_ao3_fiction(work_path, metadata_path):
    """
    Intelligent chunking with metadata preservation
    Target: 1,500-3,000 words per chunk (optimal training context)
    """
    work_text = read_file(work_path)
    metadata = json.load(open(metadata_path))

    # Chunking strategy (in order of preference):
    # 1. Chapter boundaries (if marked)
    # 2. Scene breaks (*** or \n\n\n)
    # 3. POV/time shifts (detected via NLP)
    # 4. Hard split at ~3000 words if no natural breaks

    chunks = smart_chunk(
        text=work_text,
        target_words=2000,
        max_words=3500,
        min_words=1000,
        preserve_scenes=True
    )

    for idx, chunk in enumerate(chunks):
        yield {
            "source": "ao3_fiction",
            "work_id": metadata['work_id'],
            "chunk_id": idx,
            "total_chunks": len(chunks),
            "text": chunk,
            "original_quality_score": metadata['avg_score'],  # 6.5-10.0
            "quality_tier": metadata.get('tier'),  # excellent/very_good/good
            "word_count": count_words(chunk),
            "needs_agentic_scoring": True
        }
```

**Mimi Conversations (31K → ~18K-20K pairs):**

```python
def extract_writing_pairs_from_mimi(conversation):
    """
    Extract instruction-response pairs focused on creative writing
    Filter out: coding, general Q&A, off-topic
    """
    system_prompt = conversation[0]['value']
    pairs = []

    for i in range(1, len(conversation)-1, 2):
        human_msg = conversation[i]['value']
        ai_msg = conversation[i+1]['value']

        # Extract only writing-related exchanges
        if is_creative_writing_related(human_msg, ai_msg):
            pairs.append({
                "source": "mimi",
                "conversation_id": conversation['id'],
                "turn": i,
                "instruction": human_msg,
                "response": ai_msg,
                "system": system_prompt,
                "word_count": count_words(ai_msg),
                "needs_agentic_scoring": True
            })

    return pairs
```

### 1.2 Standardize All Data to Common Format

**Universal Chunk Format (Pre-Scoring):**

```json
{
  "id": "unique_chunk_id",
  "source": "dataset_name",
  "source_category": "human_fiction" | "ao3_romance" | "opus_creative" | "steamy" | "instruct" | "craft",

  "content": {
    "instruction": "optional: task/prompt if instruction-response format",
    "text": "the actual content (story, response, guide, etc.)",
    "word_count": 2456
  },

  "provenance": {
    "original_file": "path/to/source",
    "chunk_index": 3,
    "total_chunks": 12,
    "original_quality_score": 8.7,  // if available from source
    "original_metadata": {}  // any source-specific metadata
  },

  "status": {
    "chunked": true,
    "needs_agentic_scoring": true,
    "scored": false
  }
}
```

**Output of Stage 1:** ~850K-900K standardized chunks ready for agentic scoring

---

## Stage 2: Universal Agentic Scoring (THE CRITICAL STAGE)

### 2.1 Scoring Dimensions (Expanded for ALL Data)

Every chunk gets scored on **8 dimensions** (0-100 scale each):

#### Core Quality Scores (40 points total)

**1. Writing Quality (0-20)**
- 18-20: Professional/published quality
- 15-17: Strong amateur/high-quality fanfiction
- 12-14: Competent writing, minor issues
- 8-11: Adequate, multiple issues
- 0-7: Poor quality, major problems

**2. Craft Demonstration (0-20)**
- 18-20: Exemplary use of writing techniques (show-don't-tell, pacing, tension)
- 15-17: Strong craft, good teaching examples
- 12-14: Solid craft, some good elements
- 8-11: Basic craft, limited teaching value
- 0-7: Weak craft or anti-patterns

#### Romance/Content Scores (30 points total)

**3. Romance Relevance (0-15)**
- 13-15: Central romance plot, relationship development
- 10-12: Clear romantic subplot or elements
- 7-9: Romantic elements present but not primary
- 4-6: Minimal romance touches
- 0-3: No romance content

**4. Steamy Content Level (0-15)**
- 13-15: Perfect steamy romance (sexual tension, tasteful intimacy, emotional connection)
- 10-12: Appropriate adult content with romantic context
- 7-9: Mild sensuality or fade-to-black
- 4-6: No sexual content (sweet romance or non-romance)
- 0-3: Either too explicit (pornographic) OR inappropriately tame for context

#### Training Value Scores (30 points total)

**5. Instruction-Following (0-10)**
- 9-10: Perfect adherence to constraints/prompt
- 7-8: Strong instruction following
- 5-6: Adequate instruction following
- 3-4: Weak adherence
- 0-2: Ignores instructions

**6. Dialogue Quality (0-10)**
- 9-10: Natural, character-specific, emotionally resonant
- 7-8: Strong dialogue, good character voice
- 5-6: Adequate dialogue
- 3-4: Weak or generic dialogue
- 0-2: Poor dialogue or none present

**7. Scene Construction (0-10)**
- 9-10: Expert scene structure (goal, conflict, outcome, sensory detail)
- 7-8: Strong scene construction
- 5-6: Adequate scene structure
- 3-4: Weak scene construction
- 0-2: Poor or no scene structure

**8. Emotional Depth (0-10)**
- 9-10: Powerful emotional resonance, character interiority
- 7-8: Strong emotional content
- 5-6: Adequate emotional depth
- 3-4: Surface-level emotions
- 0-2: Emotionally flat

**TOTAL SCORE: 0-100** (sum of all dimensions)

---

### 2.2 Content Tagging System

In addition to scores, agents assign **content tags** to each chunk:

**Genre/Type Tags:**
- `romance` - romantic relationship focus
- `steamy` - adult intimate content
- `sweet_romance` - romance without explicit content
- `craft_instruction` - teaching writing techniques
- `dialogue_heavy` - significant conversation
- `action` - action/adventure elements
- `introspective` - internal character focus
- `worldbuilding` - setting/world development

**Craft Tags:**
- `show_dont_tell` - exemplary showing vs telling
- `sexual_tension` - building romantic/sexual tension
- `character_voice` - strong unique voices
- `pacing_excellent` - expert pacing/rhythm
- `sensory_detail` - rich sensory description
- `emotional_beats` - well-executed emotional moments
- `scene_structure` - expert scene construction
- `conflict_resolution` - compelling conflict

**Quality Flags:**
- `tier_1_sft` - top quality, primary training data
- `tier_2_sft` - good quality, supplementary training
- `dpo_chosen_candidate` - high quality for DPO "chosen"
- `dpo_rejected_candidate` - quality issues, use as DPO "rejected"
- `needs_human_review` - borderline content, flag for review
- `remove` - below quality threshold, exclude

---

### 2.3 Agentic Scoring Implementation

**Agent Prompt (Universal for ALL Data):**

```
You are an expert romance writing evaluator assessing training data for an AI writing editor.

TASK: Score this content on 8 dimensions and assign descriptive tags.

CONTENT:
{chunk_text}

SOURCE CONTEXT:
- Dataset: {source}
- Category: {source_category}
- Word count: {word_count}
- Original quality (if available): {original_quality_score}

SCORING INSTRUCTIONS:

Rate 0-100 total across these 8 dimensions:

1. Writing Quality (0-20): Technical craft, prose quality
2. Craft Demonstration (0-20): Teaching value for writing techniques
3. Romance Relevance (0-15): How romance-focused is this content?
4. Steamy Content Level (0-15):
   - 13-15 = Perfect steamy romance (sexual tension + emotional connection)
   - 10-12 = Appropriate adult content
   - 7-9 = Mild sensuality
   - 4-6 = No sexual content
   - 0-3 = Too explicit (pornographic) OR inappropriate for context
5. Instruction-Following (0-10): Adherence to prompts/constraints
6. Dialogue Quality (0-10): Natural, character-specific conversation
7. Scene Construction (0-10): Scene structure and sensory detail
8. Emotional Depth (0-10): Emotional resonance and interiority

CONTENT BOUNDARIES:
- ✅ KEEP: Steamy romance (sexual tension, passionate scenes with emotional context)
- ✅ KEEP: Adult content appropriate for romance novels (Nora Roberts level)
- ❌ REMOVE: Pornographic/clinical sexual detail, incest, non-consent, underage

TAGGING INSTRUCTIONS:
Assign relevant tags from these categories:
- Genre: romance, steamy, sweet_romance, craft_instruction, dialogue_heavy, action, introspective, worldbuilding
- Craft: show_dont_tell, sexual_tension, character_voice, pacing_excellent, sensory_detail, emotional_beats, scene_structure, conflict_resolution
- Quality: tier_1_sft, tier_2_sft, dpo_chosen_candidate, dpo_rejected_candidate, needs_human_review, remove

RETURN JSON:
{
  "scores": {
    "writing_quality": X,
    "craft_demonstration": X,
    "romance_relevance": X,
    "steamy_content_level": X,
    "instruction_following": X,
    "dialogue_quality": X,
    "scene_construction": X,
    "emotional_depth": X,
    "total_score": X
  },

  "tags": {
    "genre": ["romance", "steamy"],
    "craft": ["sexual_tension", "show_dont_tell"],
    "quality": ["tier_1_sft"]
  },

  "recommendations": {
    "primary_use": "sft_tier_1" | "sft_tier_2" | "dpo_chosen" | "dpo_rejected" | "remove",
    "secondary_uses": ["dpo_chosen_candidate"],
    "reasoning": "2-3 sentences explaining scores and recommendations"
  },

  "content_flags": {
    "has_romance": true,
    "has_steamy_content": true,
    "has_dialogue": true,
    "has_craft_value": true,
    "needs_review": false,
    "remove_reasons": []
  }
}
```

---

### 2.4 Batch Processing Strategy

**Scale:** ~850K-900K chunks to score

**Method:** Parallel agent processing

```python
async def score_all_chunks(chunks, batch_size=50):
    """
    Process chunks in batches with parallel agents
    """
    batches = chunk_list(chunks, batch_size)

    # Parallel processing with multiple agent instances
    tasks = []
    for batch in batches:
        # Spawn agent task
        task = asyncio.create_task(
            agent_score_batch(batch)
        )
        tasks.append(task)

    # Gather results
    results = await asyncio.gather(*tasks)

    return flatten(results)

async def agent_score_batch(batch):
    """
    Score a batch of 50 chunks with single agent call
    """
    prompt = build_batch_scoring_prompt(batch)
    response = await call_claude_sonnet(prompt)
    scores = parse_batch_scores(response)

    return scores
```

**Performance Estimates:**
- Chunks to score: 850K
- Batch size: 50 chunks per agent call
- Total agent calls: 17,000
- Cost per call: ~$0.025-0.05 (Sonnet pricing)
- **Total cost: $425-850**
- Parallel processing: 100 concurrent agents
- **Time: 48-72 hours**

---

### 2.5 Scored Chunk Format (Output)

**After agentic scoring, each chunk becomes:**

```json
{
  "id": "ao3_10024469_chunk_003",
  "source": "ao3_fiction",
  "source_category": "ao3_romance",

  "content": {
    "text": "She stepped closer, her breath catching...",
    "word_count": 2341,
    "instruction": null
  },

  "provenance": {
    "original_file": "ao3_fiction/excellent/ao3_10024469.txt",
    "work_id": "ao3_10024469",
    "chunk_index": 3,
    "total_chunks": 12,
    "original_quality_tier": "excellent",
    "original_quality_score": 8.7
  },

  "agentic_scores": {
    "writing_quality": 18,
    "craft_demonstration": 17,
    "romance_relevance": 14,
    "steamy_content_level": 13,
    "instruction_following": 8,
    "dialogue_quality": 9,
    "scene_construction": 9,
    "emotional_depth": 9,
    "total_score": 87,
    "scored_at": "2025-11-15T10:23:45Z",
    "agent_version": "claude-sonnet-4"
  },

  "tags": {
    "genre": ["romance", "steamy"],
    "craft": ["sexual_tension", "show_dont_tell", "sensory_detail", "emotional_beats"],
    "quality": ["tier_1_sft", "dpo_chosen_candidate"]
  },

  "recommendations": {
    "primary_use": "sft_tier_1",
    "secondary_uses": ["dpo_chosen_candidate"],
    "reasoning": "Excellent steamy romance with strong emotional connection. Expert craft in showing sexual tension through sensory detail and emotional beats. Perfect example for teaching tasteful intimate scenes."
  },

  "content_flags": {
    "has_romance": true,
    "has_steamy_content": true,
    "has_dialogue": true,
    "has_craft_value": true,
    "needs_review": false,
    "remove_reasons": []
  },

  "status": {
    "chunked": true,
    "scored": true,
    "reviewed": false,
    "included_in_dataset": false
  }
}
```

**Storage:** All scored chunks saved to `scored_chunks/` directory, organized by source

---

## Stage 3: Smart Dataset Assembly

### 3.1 Filtering Rules

**Quality Thresholds:**

| Total Score | Action | Expected % |
|-------------|--------|------------|
| 80-100 | **Tier 1 SFT** - Primary training data | ~15-20% |
| 65-79 | **Tier 2 SFT** - Supplementary training | ~25-30% |
| 50-64 | **Consider** - Review tags, may keep for variety | ~20-25% |
| 0-49 | **Remove** - Below quality threshold | ~25-35% |

**Expected Filtered Dataset:**
- Start: 850K chunks
- After filtering (score >= 65): ~400K chunks
- Target final: 200K chunks (best variety + balance)

---

### 3.2 Smart Selection Strategy

**Goal:** Build 200K SFT dataset with perfect balance

**Selection Criteria:**

```python
def build_balanced_sft_dataset(scored_chunks, target_size=200000):
    """
    Intelligent selection for maximum variety and quality
    """
    # Priority 1: Romance-focused high quality (40% of dataset = 80K)
    romance_focused = filter_chunks(
        scored_chunks,
        min_score=70,
        min_romance_relevance=10,
        tags_include=["romance"]
    )
    selected = select_top(romance_focused, 80000, by="total_score")

    # Priority 2: Steamy content (15% of dataset = 30K)
    steamy = filter_chunks(
        scored_chunks,
        min_score=65,
        min_steamy_level=10,
        tags_include=["steamy"]
    )
    selected.extend(select_top(steamy, 30000, by="total_score"))

    # Priority 3: Craft instruction (10% = 20K)
    craft = filter_chunks(
        scored_chunks,
        min_score=70,
        tags_include=["craft_instruction"]
    )
    selected.extend(select_top(craft, 20000, by="craft_demonstration"))

    # Priority 4: Dialogue examples (10% = 20K)
    dialogue = filter_chunks(
        scored_chunks,
        min_score=70,
        min_dialogue_quality=8,
        tags_include=["dialogue_heavy"]
    )
    selected.extend(select_top(dialogue, 20000, by="dialogue_quality"))

    # Priority 5: Scene construction (10% = 20K)
    scenes = filter_chunks(
        scored_chunks,
        min_score=70,
        min_scene_construction=8
    )
    selected.extend(select_top(scenes, 20000, by="scene_construction"))

    # Priority 6: High-quality general writing (15% = 30K)
    general = filter_chunks(
        scored_chunks,
        min_score=80
    )
    selected.extend(select_diverse(general, 30000))

    # Deduplicate
    selected = remove_duplicates(selected)

    return selected
```

---

### 3.3 Format Conversion to Training Format

**Convert scored chunks to clean SFT format:**

```json
{
  "id": "sft_00001",
  "messages": [
    {
      "role": "system",
      "content": "You are an expert romance writing editor helping improve draft manuscripts."
    },
    {
      "role": "user",
      "content": "Write a steamy romance scene that builds sexual tension through emotional connection and sensory detail, not explicit description."
    },
    {
      "role": "assistant",
      "content": "[The actual chunk text - selected because it scored high on steamy_content_level + emotional_depth + sensory_detail]"
    }
  ],
  "metadata": {
    "source": "ao3_fiction",
    "original_id": "ao3_10024469_chunk_003",
    "total_score": 87,
    "tags": ["romance", "steamy", "sexual_tension", "show_dont_tell"],
    "quality_tier": 1
  }
}
```

**Instruction Generation:**
For chunks without natural instructions, generate appropriate prompts based on tags:

- `romance` + `steamy` → "Write a steamy romance scene..."
- `dialogue_heavy` + `emotional_beats` → "Write emotionally resonant dialogue..."
- `craft_instruction` → Use as-is (already instruction-response format)
- `scene_construction` → "Construct a scene that demonstrates..."

---

## Stage 4: DPO Pair Generation

### 4.1 Score-Based Pairing

**Method 1: Same-Prompt Pairing (Reddit WritingPrompts)**

```python
def generate_score_based_dpo_pairs(scored_chunks):
    """
    For chunks with same prompt/instruction:
    - chosen = score 80+
    - rejected = score 50-65
    """
    grouped = group_by_instruction(scored_chunks)
    pairs = []

    for instruction, chunks in grouped.items():
        high_quality = [c for c in chunks if c['total_score'] >= 80]
        medium_quality = [c for c in chunks if 50 <= c['total_score'] < 65]

        for chosen in high_quality[:3]:
            for rejected in medium_quality[:2]:
                if score_gap_sufficient(chosen, rejected, min_gap=25):
                    pairs.append({
                        "prompt": instruction,
                        "chosen": chosen['content']['text'],
                        "rejected": rejected['content']['text'],
                        "score_gap": chosen['total_score'] - rejected['total_score'],
                        "metadata": {
                            "chosen_id": chosen['id'],
                            "rejected_id": rejected['id'],
                            "chosen_tags": chosen['tags'],
                            "rejected_tags": rejected['tags']
                        }
                    })

    return pairs
```

**Expected yield:** 40K-45K pairs from Reddit WritingPrompts

---

### 4.2 Synthetic Degradation Pairing

**Method 2: Generate "Rejected" Versions**

```python
async def generate_synthetic_dpo_pairs(high_quality_chunks, target_count=20000):
    """
    For high-scoring chunks (85+), create degraded versions
    Focus degradation on specific dimensions based on tags
    """
    pairs = []

    for chunk in high_quality_chunks[:target_count]:
        # Determine degradation strategy based on tags
        if "steamy" in chunk['tags']['genre']:
            degradation_type = "make_steamy_awkward"  # too clinical OR too vague
        elif "dialogue_heavy" in chunk['tags']['genre']:
            degradation_type = "weaken_dialogue"  # generic, on-the-nose
        elif "scene_construction" in chunk['tags']['craft']:
            degradation_type = "break_scene_structure"  # weak goal/conflict
        else:
            degradation_type = "add_purple_prose"  # generic quality degradation

        # Generate degraded version with agent
        rejected = await agent_degrade_text(
            text=chunk['content']['text'],
            degradation_type=degradation_type,
            target_score_drop=25
        )

        # Score degraded version
        rejected_scores = await agent_score(rejected)

        # Only keep if score gap is significant
        if chunk['total_score'] - rejected_scores['total_score'] >= 20:
            pairs.append({
                "prompt": extract_or_generate_prompt(chunk),
                "chosen": chunk['content']['text'],
                "rejected": rejected,
                "chosen_score": chunk['total_score'],
                "rejected_score": rejected_scores['total_score'],
                "degradation_type": degradation_type
            })

    return pairs
```

**Expected yield:** 18K-20K pairs from synthetic degradation

**Total DPO pairs:** 60K-65K (meets goal!)

---

## Timeline & Resource Estimates

### Week 1-2: Chunking & Standardization
- **Tasks:**
  - Chunk AO3 fiction (381 → ~4,500 chunks)
  - Extract Mimi pairs (31K → ~18K pairs)
  - Convert WritingPrompts Parquet (272K)
  - Standardize all formats
- **Output:** 850K-900K chunks ready for scoring
- **Resources:** Local compute, Python scripts
- **Cost:** $0

### Week 3-4: Agentic Scoring (CRITICAL)
- **Tasks:**
  - Deploy parallel scoring agents
  - Process all 850K chunks
  - Generate scored JSON files
  - Validate scoring consistency
- **Output:** 850K scored chunks with rich metadata
- **Resources:** 100 parallel Claude Sonnet agents
- **Cost:** $425-850
- **Time:** 48-72 hours of processing

### Week 5: Smart Dataset Assembly
- **Tasks:**
  - Apply filtering rules (score >= 65)
  - Smart selection for balance (200K from 400K)
  - Generate appropriate instructions
  - Convert to training format
  - Create train/val/test splits
- **Output:** 200K SFT examples (144K train / 28K val / 28K test)
- **Resources:** Local compute
- **Cost:** $0

### Week 6: DPO Generation
- **Tasks:**
  - Score-based pairing (Reddit data)
  - Synthetic degradation pairing
  - Score all rejected examples
  - Validate pair quality
  - Create train/val/test splits
- **Output:** 60K-65K DPO pairs
- **Resources:** Claude Sonnet for degradation + scoring
- **Cost:** $200-400
- **Time:** 24-48 hours

### Week 7: Validation & Documentation
- **Tasks:**
  - Manual review of samples
  - Quality assurance checks
  - Generate dataset statistics
  - Create documentation
  - Package for training platform
- **Output:** Production-ready datasets + docs
- **Resources:** Human review + local compute
- **Cost:** $0

**TOTAL TIMELINE:** 6-7 weeks
**TOTAL COST:** $625-1,250 (mostly agentic scoring)

---

## Success Metrics

### Quantitative Goals

| Metric | Target | Measurement Method |
|--------|--------|-------------------|
| **Total SFT Examples** | 200K | Final filtered count |
| **Average Quality Score** | 72+ | Mean of all included chunks |
| **Romance Content %** | 40-50% | Chunks with romance_relevance >= 10 |
| **Steamy Content %** | 15-20% | Chunks with steamy_content_level >= 10 |
| **Craft Examples %** | 10-15% | Chunks with craft_demonstration >= 15 |
| **DPO Pairs** | 60-65K | Chosen/rejected pairs with gap >= 20 |
| **Score Distribution** | Bell curve | Majority 65-85, tail 85-100 |

### Qualitative Goals

- **Variety:** All content types represented (romance, steamy, dialogue, craft, scenes)
- **Balance:** No single source dominates (max 30% from any one dataset)
- **Quality:** Professional-grade writing that teaches good craft
- **Boundaries:** All content within steamy romance bounds (not pornographic)
- **Teaching Value:** Clear positive examples for improving draft writing

---

## Advantages of Universal Agentic Scoring

### Why This Approach is Superior

**1. No Assumptions About Quality**
- "Clean" general writing datasets may have hidden romance/steamy content (discovered by agents)
- "Steamy" datasets may have low-quality examples (filtered by agents)
- Every chunk judged on its own merits

**2. Rich Metadata Enables Smart Assembly**
- Can build infinite dataset variations from scored corpus
- Easy to create specialized subsets (e.g., "all dialogue examples scoring 80+")
- Future-proof: new use cases just require re-querying metadata

**3. Nuanced Content Boundaries**
- Agents understand "steamy romance" vs "pornographic"
- Prevents over-filtering quality adult content
- Flags genuinely problematic content while keeping appropriate steamy scenes

**4. Multi-Dimensional Quality**
- Not just "good" or "bad" - understand WHY chunk is valuable
- A 65-scoring chunk with perfect dialogue (10/10) is kept for dialogue examples
- A 70-scoring chunk with excellent steamy content (15/15) is prioritized for romance training

**5. Transparent Decision-Making**
- Every inclusion/exclusion has detailed reasoning
- Can audit decisions: "Why was this chunk scored 87?"
- Enables iterative improvement: "Let's boost romance_relevance threshold to 12"

---

## Output Structure

```
processed_data/
├── 01_standardized_chunks/           # After Stage 1
│   ├── ao3_fiction/                  # ~4,500 chunks
│   ├── mimi/                         # ~18K pairs
│   ├── writingprompts/               # ~655K examples
│   └── [other sources]/
│   └── TOTAL: ~850K chunks
│
├── 02_scored_chunks/                 # After Stage 2 ⭐ MOST VALUABLE
│   ├── ao3_fiction_scored/
│   ├── mimi_scored/
│   ├── writingprompts_scored/
│   └── [other sources_scored]/
│   └── TOTAL: ~850K scored chunks with rich metadata
│
├── 03_filtered_scored_chunks/        # After filtering (score >= 65)
│   └── TOTAL: ~400K high-quality scored chunks
│
├── 04_final_sft_dataset/             # After Stage 3
│   ├── train.jsonl                   # 144K examples
│   ├── validation.jsonl              # 28K examples
│   ├── test.jsonl                    # 28K examples
│   └── dataset_statistics.json
│
├── 05_dpo_pairs/                     # After Stage 4
│   ├── train.jsonl                   # 48K pairs
│   ├── validation.jsonl              # 8K pairs
│   ├── test.jsonl                    # 8K pairs
│   └── pair_statistics.json
│
└── documentation/
    ├── DATASET_REPORT.md
    ├── SCORING_METHODOLOGY.md
    └── QUALITY_ASSURANCE.md
```

---

## Next Steps to Begin

### Immediate Actions:

1. **Review & Approve This Plan**
   - Confirm universal agentic scoring approach
   - Approve budget ($625-1,250)
   - Approve timeline (6-7 weeks)

2. **Prepare Infrastructure**
   - Set up Claude Sonnet API access
   - Create parallel agent processing framework
   - Build standardization scripts

3. **Begin Stage 1: Chunking**
   - Start with AO3 fiction (smaller, easier to validate)
   - Process Mimi conversations
   - Convert WritingPrompts Parquet
   - **Output:** First 50K standardized chunks for initial scoring test

4. **Pilot Agentic Scoring**
   - Score first 1,000 chunks with agents
   - Validate scoring consistency
   - Tune prompts if needed
   - **Outcome:** Confidence in scoring methodology before full-scale deployment

5. **Scale to Full Processing**
   - Deploy 100 parallel agents
   - Process all 850K chunks
   - Monitor for quality/consistency
   - **Outcome:** Complete scored corpus with rich metadata

---

**This approach treats every piece of data equally, letting AI agents discover the best content regardless of source. The rich metadata enables infinite flexibility in dataset assembly while maintaining the highest quality standards.**

**Ready to begin Stage 1 upon approval.**
