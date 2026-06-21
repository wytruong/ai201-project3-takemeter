# TakeMeter

A fine-tuned text classifier that evaluates discourse quality in r/movies. Given a post or comment, the model predicts whether it is a structured critique, an emotional reaction, or a confident hot take.

---

## Community

I chose r/movies because I genuinely enjoy films and the way people talk about them. The discourse in that community is not uniform. Some posts are thoughtful breakdowns of craft and storytelling, some are raw emotional reactions after a first watch, and some are confident opinions thrown out with no support. That range makes it a natural fit for a classification task, and the distinctions between post types are real and recognizable to anyone who spends time in the community.

---

## Label Taxonomy

**`critique`**
A post that makes a structured argument about a film, analyzing direction, writing, performance, themes, or craft with specific observations that support the claim. The post shows its reasoning: not just what the poster thinks, but why.

Example: "What makes Midsommar work as a breakup film is that Ari Aster shoots the commune in full daylight the entire time. There is nowhere to hide emotionally or visually. The horror is not what is lurking in the dark but what is completely exposed."

Example: "Past Lives earns its ending because Celine Song spends the whole film building the weight of the unlived life. Every scene with Hae Sung is about what did not happen. By the time the final conversation arrives the audience has done all the grief work themselves."

**`hot take`**
A bold, confident opinion about a film or the broader film industry stated without supporting evidence. The post asserts rather than argues. The claim might be defensible but the poster does not defend it.

Example: "A24 has been coasting on aesthetic for years. Their last several releases were style with nothing underneath."

Example: "Slow cinema is just directors daring you to admit you are bored. Most of it is not profound, it is just slow."

**`reaction`**
An immediate emotional response to watching a film. The post is primarily expressive, focused on how the film made the poster feel, with little to no argument or analysis. The mode is personal experience, not claim-making.

Example: "Just finished Aftersun and I have not moved in 20 minutes. I do not even know what to say."

Example: "Watched The Witch for the first time last night with the lights off. I am not okay. That ending destroyed me."

---

## Data Collection

**Source:** r/movies, collected manually from post titles, post bodies, and top-level comments across discussion threads, first-time-watch threads, unpopular opinion threads, and film analysis posts. All examples are public.

**Labeling process:** Each example was read in full and assigned one label using the definitions above. Borderline cases were flagged in a notes column and used to refine decision rules during annotation.

**Label distribution:**

| Label | Count |
|---|---|
| reaction | 87 |
| hot take | 60 |
| critique | 52 |
| **Total** | **199** |

**Three difficult examples:**

1. **The Arrival rewatch post** (labeled `critique`): The post opens with personal rewatch framing ("it hits with a different weight compared to when I first watched it") but the majority of the post builds a real thematic argument about language, cognition, and maturity. Labeled critique because the dominant mode is structured reasoning, not emotional expression.

2. **The Schindler's List "broke you" post** (labeled `reaction`): The post ends with a one-sentence observation that "cinema isn't just for entertainment, sometimes it's a necessary, painful mirror." This could push it toward critique, but the dominant mode is personal emotional response. Labeled reaction because the argument at the end is undeveloped and the post is primarily about how the film felt to watch.

3. **The TENET post** (labeled `reaction`): The post contains specific critical observations about character writing, emotional core, and structure, which the model later predicted as critique. However, the post is structured around the stages of the user's experience watching the film: the confusion, stopping partway through, coming back, finishing it. The critical observations are byproducts of that experience, not the point of the post. Labeled reaction because the experiential framing holds the whole post together.

---

## Fine-Tuning Pipeline

**Base model:** `distilbert-base-uncased` from HuggingFace

**Training platform:** Google Colab with T4 GPU

**Training setup:** 3 epochs, learning rate 2e-5, batch size 16, weight decay 0.01, warmup steps 50. The best model checkpoint was selected based on validation accuracy.

**Hyperparameter decision:** I kept the default learning rate of 2e-5 rather than increasing it. For a dataset of 139 training examples, a higher learning rate risks overfitting quickly. The default is the standard starting point for fine-tuning BERT-family models on small datasets and the appropriate choice here given the limited data.

---

## Baseline

**Approach:** Zero-shot classification using Groq's `llama-3.3-70b-versatile` model with a system prompt containing all three label definitions and one example per label. The model was instructed to output only the label name with no explanation.

**How results were collected:** The baseline was run on the same 30-example test set used to evaluate the fine-tuned model. Each example was sent to the Groq API individually with the system prompt prepended. All 30 responses were parseable.

---

## Evaluation Report

### Results Comparison

| Model | Accuracy |
|---|---|
| Zero-shot baseline (Groq) | 0.667 |
| Fine-tuned DistilBERT | 0.433 |

Fine-tuning regression: 0.233

### Per-Class Metrics

**Baseline (Groq):**

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| critique | 0.78 | 0.88 | 0.82 | 8 |
| hot take | 1.00 | 0.22 | 0.36 | 9 |
| reaction | 0.58 | 0.85 | 0.69 | 13 |

**Fine-tuned DistilBERT:**

| Label | Precision | Recall | F1 | Support |
|---|---|---|---|---|
| critique | 0.36 | 0.50 | 0.42 | 8 |
| hot take | 0.44 | 0.44 | 0.44 | 9 |
| reaction | 0.50 | 0.38 | 0.43 | 13 |

### Confusion Matrix (Fine-Tuned Model)

| | Predicted: critique | Predicted: hot take | Predicted: reaction |
|---|---|---|---|
| True: critique | 4 | 1 | 3 |
| True: hot take | 3 | 4 | 2 |
| True: reaction | 4 | 4 | 5 |

### Wrong Predictions Analysis

**1. Hugo The Hippo post (true: critique, predicted: reaction)**

The post opens with childhood nostalgia and personal memory: "our father took us to see Hugo The Hippo at the cinema... we just loved it." The model latched onto that personal framing and predicted reaction. But the post ends with a real critical observation: the poster rewatched it as an adult and found it "fucking terrible," and the structure of the post is building toward that rewatch discovery. The model missed the argumentative arc because the early personal framing dominated the surface features it learned to detect.

**2. Sinners post (true: hot take, predicted: critique)**

The post reads: "Sinners... I just didn't get it. And it felt 30 minutes too long. Glad everyone else liked it." This is a textbook hot take: a confident negative opinion with zero supporting argument. The model predicted critique, likely because the post mentions a specific film and expresses a negative judgment, two surface features that also appear in critique posts. The model never learned that the absence of reasoning is what makes something a hot take.

**3. TENET post (true: reaction, predicted: critique)**

This is the most instructive error. The post contains specific critical observations about Nolan's filmmaking: no emotional core, poor character writing, complexity for its own sake. The model correctly identified those as critique-like features. But the post is structured around the stages of the user's experience watching the film: the confusion, stopping partway, coming back, finishing it. The critical observations are byproducts of that experience, not the point of the post. The model learned to detect critical vocabulary but not the experiential mode that frames the whole post.

### Sample Classifications

| Text (truncated) | True Label | Predicted Label | Confidence |
|---|---|---|---|
| "TENET Blew Me Away. I finally watched TENET and I'm blown away how one of my favorite directors could make such a poor film..." | reaction | critique | 0.34 |
| "Sinners... I just didn't get it. And it felt 30 minutes too long." | hot take | critique | 0.34 |
| "Perfect Blue is one of the worst highly rated movies I've ever watched." | hot take | critique | 0.35 |
| "Silence of the Lambs. It was my intro to decent cinema as a teenager..." | critique | reaction | 0.35 |
| "It's not going to win awards but I thought Warfare was incredible..." | reaction | hot take | 0.34 |

All five examples above are wrong predictions. The confidence scores are uniformly low (0.34-0.35) across the entire test set, which indicates the model was not making confident decisions on any example. A correctly predicted example like a clear reaction post ("Just finished Aftersun and I haven't moved in 20 minutes") would ideally show higher confidence, but the model's flat confidence distribution means even correct predictions were essentially guesses that happened to land on the right label.

---

## Reflection: What the Model Captured vs. What Was Intended

The fine-tuned model performed worse than the zero-shot baseline, which reveals something specific about what it actually learned. Every prediction came out at roughly 0.34 confidence, meaning the model was effectively guessing across all three labels. It did not learn the distinctions at all.

The core issue is that film discourse is genuinely subjective and the boundaries between these labels depend on the mode of the post, not its vocabulary or topic. A reaction post and a critique post can both be negative, both mention specific films, and both use evaluative language. What separates them is whether the post is organized around personal experience or around an argument. That is a subtle structural distinction that requires more than 139 training examples for a model to learn reliably.

The baseline Groq prompt outperformed fine-tuning because a large language model reasoning about a prompt description is more flexible than a small model that learned rigid pattern associations from limited data. The fine-tuned model got locked into surface-level features like the presence of critical vocabulary or personal pronouns, while the baseline could reason about the overall structure and intent of the post.

What I intended the model to learn was the mode of the post. What it actually captured, to the limited extent it captured anything, were surface vocabulary patterns that do not reliably signal mode on their own.

---

## Spec Reflection

The spec helped most during label design. The requirement to name a hard edge case before annotating forced me to write a concrete decision rule for critique vs. hot take before I had labeled a single example. Without that constraint I would have started annotating and discovered the ambiguity midway through, which would have produced inconsistent labels across the dataset.

The place my implementation diverged from the spec was in dataset size. The spec sets 200 as the minimum and frames it as achievable. In practice, collecting 199 labeled examples that cover three subjective categories with real variation is enough to define the task but not enough to fine-tune a model that outperforms a prompted LLM. If I were doing this again I would aim for 400-500 examples per label rather than 70, accepting that data collection is the bottleneck.

---

## AI Usage

**Label stress-testing:** Before annotating, I gave Claude my three label definitions and the critique/hot take edge case and asked it to generate boundary posts. Several generated posts I could not cleanly classify, which led me to tighten the decision rule: if the specific detail in a post is present for effect rather than as part of a built argument, it goes to hot take. I used this rule consistently throughout annotation.

**Failure analysis:** After generating wrong predictions, I pasted the misclassified examples into Claude and asked it to identify common patterns. It identified that the model appeared to be detecting critical vocabulary rather than post mode, and that posts with personal framing in the opening were being pulled toward reaction regardless of their overall structure. I verified both patterns by re-reading the wrong predictions myself and confirmed them. The TENET analysis in this report came directly from that process.

---

## How to Run

The training pipeline lives in the Colab notebook. To reproduce:

1. Open the notebook in Google Colab and set runtime to T4 GPU
2. Run the setup cell to install dependencies
3. Run Section 1 and upload `dataset.csv`
4. Run Section 2 to split and tokenize
5. Run Section 5 to reproduce the Groq baseline (requires a Groq API key)
6. Run Sections 3 and 4 to fine-tune and evaluate
7. Run Section 6 to generate the comparison output
