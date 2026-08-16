# ToT_Teacher_Student
Distilling Tree-of-Thoughts reasoning into small language models (sub-1B params) via one-step ToT knowledge distillation. Teacher (DeepSeek) generates full multi-step ToT search trees on Game of 24 with CodeAct verification &amp; dead-end memory; traces are linearized and used to fine-tune Qwen2.5-0.5B/SmolLM-360M.
LLMtoSLM: Teaching Small Language Models to Reason Like Tree of Thoughts

This repo holds the code for our thesis project, where we tried to take the "Tree of Thoughts" style of reasoning (used by big, expensive language models) and squeeze it into a small, cheap model that can run on a laptop or edge device.

# What is this project about?

Normal Chain of Thought (CoT) prompting makes a model think step by step, but it can't go back if it makes a mistake early on. Tree of Thoughts (ToT) fixes that by letting the model try many paths, check which ones look promising, and backtrack when something fails. The problem is that ToT needs a lot of expensive LLM calls, so it's not practical for small models.

Our idea: let a big, capable model (the "teacher") do the full expensive tree search once, record everything it tried, both the paths that worked and the ones that failed, then compress all of that into a single training example. A small model (the "student") learns from these examples so it can imitate the teacher's reasoning in one go, without ever running an actual tree search itself.

We tested this on the Game of 24 puzzle (given 4 numbers, use +, -, *, / to make 24).

# The pipeline, in three steps

# Step 1, the Teacher:
We used DeepSeek v4 Flash to build a real Tree of Thoughts search for each puzzle. It proposes moves as small pieces of code (so we don't have to trust the model's arithmetic), scores how promising each state looks, and uses a "dead end memory" so it doesn't waste time trying combinations that already failed before.

# Step 2, turning the tree into training data: 
A full search tree is too complex for a small model to learn from directly. So we flatten each solved tree into one linear example that still shows the "try something, hit a dead end, go back, try again, succeed" pattern.

# Step 3, the Student: 
We fine tuned small models (Qwen2.5-0.5B-Instruct and SmolLM-360M) on these flattened examples so they learn to copy the teacher's reasoning style, but in a single pass, no search needed at the end.

# How well did it work?

We tested everything on the hardest 100 puzzles in our dataset (ones the models never saw during training).

# Model	                                                            Solved out of 100
Teacher (DeepSeek, full tree search)	                              89
Our student (Qwen2.5-0.5B, fine tuned, best run)	                  24
Our student (SmolLM-360M, fine tuned)	                               0
Qwen2.5-0.5B with no fine tuning	                                   0
Closest comparable prior work's teacher	                            19
Closest comparable prior work's student	                             9

Our teacher solved way more puzzles than the comparable prior work's teacher (89 vs 19), and in our best run, our student even beat their entire teacher model, not just their student. We also ended up with about ten times more training examples than that prior work, even though we used fewer source puzzles to get there.

SmolLM-360M completely failed even though its training loss looked great. It turned out that low loss just meant it learned the format, not that it actually learned to solve anything. It kept backtracking forever without knowing when to stop. Qwen worked better because it already had stronger math skills before we even started fine tuning it.

# What we found when we dug deeper (ablation studies)

We didn't just report one number and stop there. We reran the student training and evaluation 3 times, with different data setups and different random seeds, and ran statistical significance tests to check which results were real and which ones could have just been luck.

# Different training data setups
# Setup	Success                                             rate	                      Notes
Backtracking data, greedy decoding (our main result)	      24%	                        best single run
Backtracking data, greedy decoding, different seed	        19%	                        same setup, different random seed
Clean examples only (no backtracking), sampling	            10 to 11%	                  ablation baseline
Clean examples only, no seed control	                      6%                        	shows how much a run can swing
Fixed at exactly 1 backtrack per example	                  13%	
Fixed at exactly 2 backtracks per example	                  9%	
Balanced mix of backtrack counts	                          16%	

# A few honest takeaways from this:

The result really does move around between runs. Our best run hit 24%, but a rerun with a different seed landed at 19%. Anyone reporting a single number without mentioning this range is hiding some of the story, so we didn't want to do that either.
Backtracking data does help, but how much depends on decoding settings. When we compared augmented (backtracking) training data against clean only data under sampling, the difference was statistically significant (24% vs 10%, p = 0.0084). But under greedy decoding, the same comparison (19% vs 11%) was not statistically significant, meaning it could be random noise rather than a real effect.
Too many or too few backtracks in the training data both hurt. Fixing every example to exactly 1 backtrack got 13%, and fixing every example to exactly 2 backtracks dropped to 9%. A balanced mix of different backtrack counts did better than either fixed amount (16%), which matches our thinking that variety in the training data matters.
Our student does not clearly beat the comparison paper's teacher. We initially framed our 24% student result as "beating their 19% teacher," but the significance test on this specific comparison came back with p = 0.39, which is not significant. So this is a real but weak result, it looks good, but we can't confidently claim it as a definite win. We are being upfront about this rather than overstating it.
Statistical significance tests (n = 100 puzzles each)

# Comparison	                                         Result	                          Significant?
Our teacher (89%) vs comparison teacher (19%)	         p < 0.001	                      Yes
Our student (24%) vs comparison student (9%)	         p = 0.007	                      Yes
Our student (24%) vs comparison teacher (19%)	         p = 0.49	                        No, could be chance
Our student (24%) vs SmolLM (0%)	                     p < 0.001	                      Yes
Backtracking data vs clean only data (sampling)	       p = 0.014	                      Yes
Backtracking data vs clean only data (greedy)	         p = 0.16	                        No, could be chance


# How to run this yourself

You'll need Python 3.10 or newer, and an API key for DeepSeek (or another OpenAI compatible API) if you want to regenerate the teacher's trees.

# Install what the teacher stage needs:

```bash
pip install openai sympy numpy
```
# Install what the student training stage needs:

```bash
pip install transformers accelerate peft bitsandbytes optuna
```
If you're using Google Colab, the free T4 GPU is enough to fine tune the small student models. The teacher stage doesn't need a GPU at all since it's just calling an API and doing simple math.

# To keep your results safe across Colab sessions, mount your Google Drive before running anything:

```python
from google.colab import drive
drive.mount('/content/drive')
```
# Running the steps in order
Open the notebook inside `Teacher/teacher_deepseek/` and run it against the puzzles in `Data/24.csv` to generate raw search trees.
Run the linearizer script to turn those trees into training examples.
Fine tune a student model using one of the notebooks in `student training files/`.
Run `evaluate.py` or `evaluate.ipynb` to test the fine tuned model on the held out puzzles (ranks 901 to 1000).
