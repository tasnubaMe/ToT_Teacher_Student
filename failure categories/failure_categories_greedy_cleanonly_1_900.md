# Failure Mode Analysis

Model evaluated on 100 puzzles (Game of 24, ranks 901–1000).  
Correct: **12/100 (12%)** | Failures analysed: **88**

## Failure Categories

| Category | Count | % of failures |
|---|---|---|
| no_answer_line | 31 | 35.2% |
| hit_token_limit | 8 | 9.1% |
| excessive_backtracking | 3 | 3.4% |
| wrong_final_answer | 12 | 13.6% |
| malformed_answer | 34 | 38.6% |
| other | 0 | 0.0% |

## Backtracking in Correct Answers

Of the 12 correct answers:
- **0** (0%) included at least one `Backtrack.` step
- **12** (100%) were solved directly without backtracking