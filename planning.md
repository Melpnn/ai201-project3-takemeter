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
appropiate to the specific details provided. Based on the information given, 
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
> The post describes a reaction or response that is not appropiate to the 
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
If yes, then NOR. If the reaction seems like too much given the specific 
details, then YOR. For genuinely uncertain cases, comments are checked 
and the majority community response is used as a tiebreaker.

---

## Posts Excluded From Dataset
The following post types were skipped during collection:
- Update "part 2" posts that lack original context
- Posts seeking advice rather than reaction validation
- Posts where the body is primarily an image with no written context
- Posts where no clear situation is described

---

## Hard Edge Cases

**Edge Case 1: Both parties are overreacting**
> Post: A daughter is asked to get her mom's Uber Eats delivery while 
working on an exam. Mom criticizes her slow walking, then doesn't come 
to eat for 5 minutes while scrolling her phone. Daughter is upset about 
the double standard. Comments were split — some said the mom was 
unreasonable, some said the daughter was being too sensitive, some said 
both were overreacting.

Why it's hard: There is no single clear subject whose reaction can be 
isolated and judged. Both people are reacting to each other.

Decision: Excluded entirely. If a post has no clear subject whose 
reaction can be evaluated on its own, it goes in the skip pile. 
Both-sides posts do not fit cleanly into NOR or YOR.

---

**Edge Case 2: Reaction is valid but behavior is also questionable**
> Post: Girlfriend sits in backseat to be respectful to boyfriend's 
parents on only their second meeting. Boyfriend repeatedly tells her 
to sit in front, throws a fit when she doesn't, refuses to drive, and 
makes his dad drive instead. She says she doesn't want to look at him. 
Comments said both were being ridiculous.

Why it's hard: Her original instinct was reasonable and his reaction 
was genuinely childish. But her response of not wanting to look at him 
over a seating argument could itself be seen as YOR. The comments 
calling both parties unreasonable made this difficult to assign to 
a single label.

Decision: Labeled NOR. The label is based on her reaction to his 
behavior specifically. Most reasonable people would be upset if their 
partner threw a fit and refused to drive over a seating choice, 
especially during an important family meeting.

---

**Edge Case 3: Action is reasonable but framing is biased**
> Post: A customer sends polite feedback to a grocery store after a 
disabled employee didn't help her and nudged her with a stock cart. 
She asks whether sending the feedback was reasonable. Comments were 
divided and explicitly noted how split the reactions were.

Why it's hard: The action taken (sending polite store feedback) is 
objectively mild and something most people would do. But the poster's 
attitude toward the disabled worker raises questions about whether the 
situation was accurately described, making it hard to judge the 
reaction fairly.

Decision: Labeled NOR. The label is based on the action described, which is 
sending polite feedback about a genuine workplace interaction, and not 
the poster's underlying attitude. The feedback message itself was 
measured and reasonable.

---

## Data Collection Plan
- Source: r/AIO on Reddit (public posts only)
- Collection method: Manual copy-paste of post title and body into 
Google Sheets
- Target: 200 posts, 100 NOR and 100 YOR
- If a label was underrepresented, hunted specifically using Reddit 
and Google search:
  - `subreddit:AIO "yes you are"`
  - `site:reddit.com/r/AIO "YOR"`

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
After fine tuning, wrong predictions will be pasted into Claude and 
asked to identify common patterns before writing the evaluation report. 
Patterns will be verified manually before including in the README.

---

## Stretch Features
- [ ] Inter-annotator reliability
- [ ] Confidence calibration
- [ ] Error pattern analysis
- [ ] Deployed interface
