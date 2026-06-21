# TakeMeter - planning.md

## Community

I chose r/movies because I genuinely enjoy films and the way people talk about them. What drew me to this community specifically is that the discourse is not uniform. Some posts are thoughtful breakdowns of craft and storytelling, some are raw emotional reactions after a first watch, and some are confident opinions thrown out with no support. That range makes it a natural fit for a classification task. The distinctions between post types are real and recognizable to anyone who spends time in the community, which means the labels have grounding beyond just my own judgment.

r/movies also has high post volume, all public text, and enough variety across film types and genres that 200 examples won't all look the same. I am particularly drawn to indie and A24 discourse, which tends to be more varied in quality than blockbuster discussion threads.

---

## Labels

**`critique`**
A post that makes a structured argument about a film, analyzing direction, writing, performance, themes, or craft with specific observations that support the claim. The post shows its reasoning: not just what the poster thinks, but why.

Examples:
- "What makes Midsommar work as a breakup film is that Ari Aster shoots the commune in full daylight the entire time. There is nowhere to hide emotionally or visually. The horror is not what is lurking in the dark but what is completely exposed."
- "Past Lives earns its ending because Celine Song spends the whole film building the weight of the unlived life. Every scene with Hae Sung is about what did not happen. By the time the final conversation arrives the audience has done all the grief work themselves."

**`reaction`**
An immediate emotional response to watching a film. The post is primarily expressive, focused on how the film made the poster feel, with little to no argument or analysis. The mode is personal experience, not claim-making.

Examples:
- "Just finished Aftersun and I have not moved in 20 minutes. I do not even know what to say."
- "Watched The Witch for the first time last night with the lights off. I am not okay. That ending destroyed me."

**`hot_take`**
A bold, confident opinion about a film or the broader film industry stated without supporting evidence. The post asserts rather than argues. The claim might be defensible but the poster does not defend it.

Examples:
- "A24 has been coasting on aesthetic for years. Their last several releases were style with nothing underneath."
- "Slow cinema is just directors daring you to admit you are bored. Most of it is not profound, it is just slow."

---

## Hard Edge Cases

The hardest boundary in this taxonomy is **critique vs. hot_take**. Both can be negative or positive, both can be about specific films, and a hot take sometimes includes a single piece of evidence that makes it look like analysis.

Anticipated hard case:
> "Robert Eggers cannot write a third act to save his life. The Lighthouse, The Northman, Nosferatu - they all fall apart in the final 20 minutes. He builds incredible atmosphere and then has no idea how to land it."

This could be `critique` (identifies a specific structural pattern across multiple films) or `hot_take` (sweeping claim about a director stated with confidence but no real argument).

**Decision rule:** If the post identifies a specific technique or pattern and connects it to an effect on the viewer or the film's meaning, label it `critique`. If the post makes a sweeping claim and the specific detail is decorative rather than argumentative, present for effect rather than as part of a built case, label it `hot_take`. The Eggers example names a pattern but does not analyze why the third acts fail or what Eggers is attempting, so it goes to `hot_take`.

A second edge case is **reaction vs. hot_take**. Reactions can contain opinions. The distinction is mode: a reaction is anchored in personal experience ("I felt," "I couldn't stop thinking about"), a hot take is anchored in a claim about the film or industry ("this movie is," "directors keep doing"). When a post mixes both, I label it by whichever mode dominates.

---

## Data Collection Plan

**Source:** r/movies, specifically post titles and top-level comments from recent discussion threads, "First Time Watch" threads, and unpopular opinion threads. All public.

**Collection method:** Manual copy-paste into a spreadsheet. I will read each example before labeling it.

**Target distribution:**
- `critique`: ~70 examples
- `reaction`: ~70 examples
- `hot_take`: ~70 examples

Total: ~210 examples, giving a small buffer above the 200 minimum with no label above 70% of the dataset.

**If a label is underrepresented:** If after 150 examples any label is below 40 examples, I will actively seek out posts of that type before continuing. Dedicated discussion threads tend to generate more critique-style posts. Unpopular opinion and hot take threads generate more hot_take examples. First time watch threads generate more reactions. I can target each specifically if needed.

---

## Evaluation Metrics

I will use the following metrics:

**Overall accuracy** gives a single number for comparison between the fine-tuned model and the Groq baseline. Necessary but not sufficient on its own.

**Per-class F1** is the most important metric for this task. Because my labels are roughly balanced and the task is subjective, I need to know whether the model is learning all three distinctions or just getting good at one label and ignoring the others. F1 captures both precision and recall in one number per class.

**Confusion matrix** tells me which specific label pairs are being confused and in which direction. This is more actionable than accuracy alone. If the model is consistently mislabeling `hot_take` as `critique`, that tells me something specific about the boundary it has not learned.

Accuracy alone is not enough because a model that predicts `reaction` for every post could achieve 33% accuracy on a balanced dataset without learning anything. Per-class F1 catches this failure mode.

---

## Definition of Success

The classifier will be considered good enough for the purposes of this project if it meets all three of the following on the test set:

- Overall accuracy above **70%**
- No single class with F1 below **0.60**
- Fine-tuned model outperforms the Groq zero-shot baseline on overall accuracy

These thresholds reflect what a genuinely useful classifier looks like on a 3-class subjective task with 200 training examples. A model that hits 70% accuracy and maintains 0.60+ F1 across all classes has learned all three distinctions, not just the easy ones. The baseline comparison ensures that fine-tuning actually contributed something beyond what a prompted LLM can do out of the box.

Future improvement targets (not required for this submission): per-class F1 above 0.75 with a larger dataset, and a deployed interface for real-time classification.

---

## AI Tool Plan

**Label stress-testing:** Before annotating, I will give Claude my three label definitions and edge case rules and ask it to generate 10 posts that sit at the boundary between two labels, specifically critique/hot_take and reaction/hot_take. If I cannot cleanly classify those generated posts using my own rules, I will tighten the definitions before starting annotation.

**Annotation assistance:** I will use an LLM to pre-label a batch of examples by providing it my label definitions and a set of unlabeled posts. I will review and correct every pre-assigned label before it goes into the dataset. Any pre-labeled examples will be tracked and disclosed in the AI usage section of the README.

**Failure analysis:** After generating wrong predictions from the fine-tuned model, I will paste the misclassified examples into Claude and ask it to identify common patterns such as post length, sarcasm, label pairs, and ambiguous phrasing. I will verify any identified patterns by re-reading the examples myself before including the analysis in the evaluation report.

---

## Stretch Features (to update before starting each)

- [ ] Inter-annotator reliability
- [ ] Confidence calibration
- [ ] Error pattern analysis
- [x] Deployed interface
