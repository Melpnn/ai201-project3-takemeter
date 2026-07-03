# TakeMeter Planning

## Community
I chose r/AIO (Am I Overreacting?) on Reddit. This community is built around 
people sharing personal situations and asking whether their emotional reaction 
or response was proportionate. It is a strong fit for a classification task 
because every post follows a consistent format (describing a situation and 
asking for judgment), the discourse is entirely text-based, and the community 
has a well-established voting pattern (NOR/YOR) that grounds the labels in 
real community norms rather than personal opinion.

---

## Labels

**`NOR` (Not Overreacting)**
> The post describes a situation where the emotional reaction or response is 
proportionate to the specific details provided. Based on the information given, 
a reasonable person in the same situation would likely feel or react similarly. 
When uncertain, the majority of r/AIO commenters would respond NOR.

Observable signals:
- Serious or repeated trigger (cheating, betrayal, boundary violations, 
pattern of behavior)
- Specific details, timeline, or history that justify the reaction
- The consequence or response matches the severity of the situation

Example 1:
> "AIO for cutting off my mom after she gave my address to my abusive ex 
without telling me?"
Label: NOR — serious breach of trust with real safety implications

Example 2:
> "AIO for being upset my boyfriend forgot my birthday and made no effort 
to make it up even after I told him it hurt me?"
Label: NOR — repeated dismissal of expressed feelings is a legitimate concern

---

**`YOR` (Yes Overreacting)**
> The post describes a reaction or response that is disproportionate to the 
trigger based on the specific details provided. The situation described does 
not contain enough justification for the emotional response or action taken. 
When uncertain, the majority of r/AIO commenters would respond YOR.

Observable signals:
- Minor or ambiguous trigger with a large emotional response
- Drastic action (blocking, ending relationships, confrontations) over 
small incidents
- Little context or history provided to justify the reaction
- Words like "just" "only" "literally" before describing the trigger

Example 1:
> "AIO for leaving my friend's party early because she didn't greet me 
when I walked in?"
Label: YOR — minor social oversight does not justify leaving early

Example 2:
> "AIO for blocking my boyfriend because he didn't text me back for 2 hours 
while he was at work?"
Label: YOR — short communication gap with a reasonable explanation 
does not justify blocking

---

## Labeling Process
Labels are assigned by reading the post title and body and asking: 
"Would a reasonable person in this situation react the same way?" 
If yes → NOR. If the reaction seems like too much given the specific 
details → YOR. For genuinely uncertain cases, comments are checked 
and the majority community response is used as a tiebreaker.

---

## Posts Excluded From Dataset
The following post types were skipped during collection:
- Update posts that lack original context
- Posts seeking advice rather than reaction validation
- Posts where the body is primarily an image with no written context
- Posts where no clear situation is described

---

## Hard Edge Cases

**Edge Case 1: Valid but dramatic language**
> "AIO thinking that he doesn't text a lot and that he might not like me?"
Could be NOR (legitimate concern about someone pulling away) or YOR 
(they just started talking and he has a reason — fire season). 
Decision: Labeled YOR because the relationship was very new and he had 
a documented legitimate reason for being unavailable.

**Edge Case 2: Serious situation but missing context**
> "AIO: I don't know who to believe" (boyfriend cheating post)
Post was excluded entirely — it was not asking whether the reaction was 
proportionate, it was asking for advice on what to do. Does not fit 
either label cleanly.

**Edge Case 3: Overreacting but emotionally understandable**
Posts where the person has anxiety or trauma history that explains their 
outsized reaction. Decision: Labeled based on the situation described in 
the post, not the backstory. If the trigger is minor regardless of 
history, label is YOR.

---

## Data Collection Plan
- Source: r/AIO on Reddit (public posts only)
- Collection method: Manual copy-paste of post title and body into 
Google Sheets
- Target: 200 posts, 100 NOR and 100 YOR
- If a label was underrepresented, hunted specifically using Reddit 
and Google search:
  - `subreddit:AIO "yes you are"`
  - `site:reddit.com/r/AIO "you are overreacting"`

---

## Label Distribution
| Label | Count |
|---|---|
| NOR | 100 |
| YOR | 100 |
| **Total** | **200** |

---

## Evaluation Metrics
Accuracy alone is not sufficient for this task because even a perfectly 
balanced dataset can produce a model that learns shortcuts rather than 
the intended distinction. I will use:

- **Overall accuracy** — baseline comparison between fine-tuned and 
zero-shot models
- **Per-class F1** — to check whether the model performs equally on 
both NOR and YOR or favors one
- **Confusion matrix** — to identify which direction errors go 
(NOR predicted as YOR or vice versa)
- **3 analyzed failure cases** — to understand what the model got wrong 
and why

---

## Definition of Success
The fine-tuned model will be considered successful if:
- Overall accuracy exceeds the zero-shot baseline by at least 10%
- Per-class F1 for both NOR and YOR exceeds 0.65
- No single label accounts for more than 70% of predictions

---

## AI Tool Plan

**Label stress-testing:**
Used Claude to generate boundary posts sitting between NOR and YOR 
to test whether label definitions were precise enough before annotating 
200 examples.

**Annotation assistance:**
Labels were assigned manually. No LLM pre-labeling was used. 
For uncertain cases, Reddit comments were checked as a tiebreaker.

**Failure analysis:**
After fine-tuning, wrong predictions will be pasted into Claude and 
asked to identify common patterns before writing the evaluation report. 
Patterns will be verified manually before including in the README.

---

## Stretch Features
- [ ] Inter-annotator reliability
- [ ] Confidence calibration
- [ ] Error pattern analysis
- [ ] Deployed interface
