# ToT_Teacher_Student
Distilling Tree-of-Thoughts reasoning into small language models (sub-1B params) via one-step ToT knowledge distillation. Teacher (DeepSeek) generates full multi-step ToT search trees on Game of 24 with CodeAct verification &amp; dead-end memory; traces are linearized and used to fine-tune Qwen2.5-0.5B/SmolLM-360M.
