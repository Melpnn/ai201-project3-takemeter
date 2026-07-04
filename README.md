# TakeMeter — r/AIO Discourse Classifier

## What This Project Does
TakeMeter is a fine-tuned text classifier that evaluates posts from 
r/AIO (Am I Overreacting?) on Reddit. Given a post describing a 
personal situation, the model predicts whether the community would 
respond NOR (Not Overreacting) or YOR (Yes Overreacting).

---

## Community
r/AIO (Am I Overreacting?) is a Reddit community where people describe 
personal situations and ask whether their emotional reaction or response 
was proportionate. Every post follows a consistent format, where it describes 
a situation and asks for community judgment. This makes it a strong fit 
for a text classification task. The community has a well established 
voting pattern (NOR/YOR) that grounds the labels in real community 
norms rather than personal opinion.

---

## Labels

**NOR (Not Overreacting)**
The post describes a situation where the emotional reaction or response 
is appropriate to the specific details provided. Based on the 
information given, a reasonable person in the same situation would 
likely feel or react similarly. When uncertain, the majority of r/AIO 
commenters would respond NOR.

Observable signals:
- Serious or repeated trigger (cheating, betrayal, boundary violations,
pattern of behavior)
- Specific details, timeline, or history that justify the reaction
- The consequence or response matches the severity of the situation

**YOR (Yes Overreacting)**
The post describes a reaction or response that is disproportionate to 
the trigger based on the specific details provided. The situation 
described does not contain enough justification for the emotional 
response or action taken. When uncertain, the majority of r/AIO 
commenters would respond YOR.

Observable signals:
- Minor or ambiguous trigger with a large emotional response
- Drastic action (blocking, ending relationships, confrontations) over
small incidents
- Little context or history provided to justify the reaction
- Words like "just" "only" "literally" before describing the trigger

---

## Data Collection

**Source:** r/AIO on Reddit (public posts only)

**Method:** Manual copy-paste of post title and body text into Google 
Sheets. Posts were collected by browsing Top (all time), Top (this 
year), and New sorted posts. For underrepresented labels, targeted 
searches were used:
- `subreddit:AIO "yes you are"`
- `site:reddit.com/r/AIO "you are overreacting"`

**Posts excluded:**
- Update posts that lack original context
- Posts seeking advice rather than reaction validation
- Posts where the body is primarily an image with no written context
- Posts where no clear situation is described

**Label distribution:**

| Label | Count |
|---|---|
| NOR | 100 |
| YOR | 100 |
| Total | 200 |

---

## Hard-to-Label Examples

**Example 1: Both parties overreacting**
A daughter was asked to get her mom's Uber Eats delivery while working 
on an exam. Mom criticized her slow walking then didn't come to eat for 
5 minutes while scrolling her phone. Comments were split with no clear 
majority. Decision: Excluded since no single subject whose reaction could 
be cleanly evaluated on its own.

**Example 2: Reaction valid but behavior also questionable**
A girlfriend sat in the backseat to be respectful to her boyfriend's 
parents on only their second meeting. Boyfriend threw a fit and refused 
to drive. Comments said both were being ridiculous. Decision: Labeled 
NOR since most reasonable people would be upset if their partner refused 
to drive over a seating choice during an important family meeting.

**Example 3: Action reasonable but framing biased**
A customer sent polite feedback to a grocery store after a disabled 
employee didn't help her and nudged her with a stock cart. Comments 
were explicitly divided. Decision: Labeled NOR since the action taken 
of sending polite feedback was objectively mild regardless of the 
poster's underlying attitude toward the worker.

---

## Model and Training

**Base model:** distilbert-base-uncased (HuggingFace)

**Training approach:** Fine-tuned on 140 training examples for 5 epochs 
using the HuggingFace Trainer API on Google Colab T4 GPU.

**Key hyperparameter decisions:**
- Learning rate: 5e-5 (increased from default 2e-5 — the model was not
learning at the lower rate, loss was not decreasing across epochs)
- Max token length: 512 (increased from default 256 — r/AIO posts are
long and truncating at 256 was losing meaningful content)
- Epochs: 5 (increasing to 10 caused severe overfitting and training loss
dropped to 0.01 while validation loss shot up to 1.64, so 5 epochs
was the best balance)

## Baseline Prompt

The following prompt was used to classify each test post with Groq's 
llama-3.3-70b-versatile model with no task-specific training:

You are classifying posts from r/AIO (Am I Overreacting?) on Reddit.
Each post describes a personal situation and asks whether the person's 
emotional reaction or response was proportionate.

NOR: The reaction described is proportionate to the situation. A 
reasonable person in the same situation would likely feel or react 
similarly.

YOR: The reaction described is disproportionate to the trigger. The 
situation does not justify the emotional response or action taken.

You must respond with ONLY one word. Either NOR or YOR. Nothing else.

Results were collected by running this prompt against all 30 test 
examples using the Groq API with temperature set to 0 for consistent 
outputs.

---

## Results

### Overall Accuracy

| Model | Accuracy |
|---|---|
| Zero-shot baseline (Groq llama-3.3-70b-versatile) | 76.7% |
| Fine-tuned DistilBERT | 40.0% |
| Difference | -36.7% |

### Per-Class Metrics — Fine-Tuned Model

| Label | Precision | Recall | F1 |
|---|---|---|---|
| NOR | 0.41 | 0.47 | 0.44 |
| YOR | 0.38 | 0.33 | 0.36 |
| Macro avg | 0.40 | 0.40 | 0.40 |

### Per-Class Metrics — Groq Baseline

| Label | Precision | Recall | F1 |
|---|---|---|---|
| NOR | 0.70 | 0.93 | 0.80 |
| YOR | 0.90 | 0.68 | 0.72 |
| Macro avg | 0.80 | 0.77 | 0.76 |

### Confusion Matrix — Fine-Tuned Model

| | Predicted NOR | Predicted YOR |
|---|---|---|
| True NOR | 7 | 8 |
| True YOR | 10 | 5 |

The model predicted NOR incorrectly 10 times for posts that were 
actually YOR, and predicted YOR incorrectly 8 times for posts that 
were actually NOR. Nearly every prediction had a confidence score 
between 0.50 and 0.52, indicating the model was essentially guessing 
on every post.

---

## Failure Analysis

**Wrong prediction #1:**
Post: "AIO for feeling violated after my coworker touched me? I'm 17 
and a student nurse, said coworker is 35-38..."
True: NOR | Predicted: YOR | Confidence: 0.50

Analysis: This is a clear NOR since a 17-year-old being touched 
inappropriately by an adult coworker is a serious situation. The model 
predicted YOR with essentially no confidence (0.50). The post is long 
and the key details that justify the reaction appear later in the text. 
This suggests the model never learned to identify serious triggers, and it 
was guessing regardless of content.

**Wrong prediction #2:**
Post: "AIO- Husband took 4yr old on 1mi car ride, in front seat."
True: YOR | Predicted: NOR | Confidence: 0.50

Analysis: This is a genuine YOR since a short rural car ride with a child 
in the front seat is not a serious safety violation in context. The 
model predicted NOR with no confidence. The words "husband" and "4yr 
old" may have triggered associations with serious family situations, 
causing a wrong prediction. This suggests the model was responding to 
surface level trigger words rather than the actual situation described.

**Wrong prediction #3:**
Post: "AIO Partner told me something that has rattled me — she was the 
other woman in an affair when she was younger..."
True: YOR | Predicted: NOR | Confidence: 0.52

Analysis: This is YOR because the partner's past behavior happened 
before the relationship and cannot be changed. The model predicted NOR, 
likely because words like "affair" and "rattled" signal serious 
situations that usually justify strong reactions. The model learned 
surface-level trigger words rather than the actual proportionality of 
the reaction.

---

## Sample Classifications

| Post (truncated) | Predicted | Confidence | Correct? |
|---|---|---|---|
| "AIO for feeling violated after coworker touched me..." | YOR | 0.50 | ❌ (true: NOR) |
| "AIO for dreading Christmas because SIL insists on hosting..." | YOR | 0.50 | ❌ (true: NOR) |
| "AIO: Previous job ghosting me after asking about back pay..." | YOR | 0.51 | ❌ (true: NOR) |
| "AIO gf defends she loves me most to point of violence..." | NOR | 0.51 | ❌ (true: YOR) |
| "AIO for telling husband we need to teach daughter to stop dragging kids..." | NOR | 0.51 | ❌ (true: YOR) |
| "AIO for cutting off my mom after she gave my address to my abusive ex..." | NOR | 0.53 | ✅ (true: NOR) |


The correctly predicted example above was labeled NOR and the post 
described a serious and repeated safety violation which contains 
enough specific detail and history that even a model with limited 
training could pick up on the severity of the situation. The rest showed no meaningful pattern, 
and the model appears to be guessing in both directions with uncertainty.

---

## Reflection

The model did not meet any of the success criteria defined in 
planning.md. Accuracy fell 36.7% below the baseline rather than 
exceeding it, and per-class F1 scores of 0.44 and 0.36 fell well 
below the 0.65 threshold. This outcome reflects the difficulty of 
the task rather than a flaw in the pipeline.

What I intended the model to learn: whether the emotional reaction 
described in a post is proportionate to the specific trigger, based 
on observable text features like trigger severity, post length, and 
language proportionality.

What the model actually learned: nothing consistent. Nearly every 
confidence score hovered at 0.50-0.52, meaning the model had 
essentially no signal and it was flipping a coin on every post 
regardless of content. The task requires understanding contextual 
proportionality, and a judgment that depends on reading the full 
situation and weighing the severity of the trigger against the 
response. This is beyond what DistilBERT can learn from 140 examples 
of inherently subjective human judgment.

The Groq baseline achieving 76.7% with zero training confirms the 
task is learnable and how a large enough model can make this judgment. But 
fine tuning a small model on 200 subjective examples did not provide 
enough consistent signal to learn the distinction. The subjectivity 
of the labels themselves likely contributed to how two people reading the 
same post might reasonably disagree, making the training signal noisy 
from the start.

---

## Spec Reflection

The spec helped most during label design, since the requirement to define 
observable decision boundaries forced me to move away from vague 
definitions like "valid reaction" toward grounding labels in community 
behavior (NOR/YOR voting patterns). This made the taxonomy more 
defensible even though the underlying task remained subjective.

My implementation diverged from the spec in one key way: the spec 
assumes fine tuning will meaningfully improve on the baseline. In my 
case it did the opposite, and the fine tuned model performed 36.7% worse 
than zero shot Groq. This revealed something the spec does not 
explicitly address: for highly subjective tasks with small datasets, 
a large zero-shot model may outperform a fine tuned small model 
regardless of label quality.

---

## AI Usage

**Label stress-testing:** Used Claude to generate boundary posts 
sitting between NOR and YOR before annotating 200 examples. Claude 
produced several posts that were difficult to classify, which prompted 
me to tighten label definitions and add explicit decision rules for 
edge cases.

**Failure analysis:** After fine tuning, wrong predictions were pasted 
into Claude and asked to identify common patterns before writing the 
evaluation report. Claude identified that nearly all wrong predictions 
had confidence scores around 0.50, suggesting the model learned no 
signal at all rather than learning the wrong signal. This was verified 
by reviewing the predictions manually.

---

## Repository Structure

```
ai201-project3-takemeter/
├── README.md
├── planning.md
├── data/
│   └── data.csv
└── outputs/
    ├── evaluation_results.json
    └── confusion_matrix.png
```