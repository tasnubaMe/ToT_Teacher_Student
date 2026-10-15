# Failure Mode Analysis

Model evaluated on 100 puzzles (Game of 24, ranks 901–1000).  
Correct: **21/100 (21%)** | Failures analysed: **79**

## Failure Categories

| Category | Count | % of failures |
|---|---|---|
| no_answer_line | 20 | 25.3% |
| hit_token_limit | 35 | 44.3% |
| excessive_backtracking | 15 | 19.0% |
| wrong_final_answer | 0 | 0.0% |
| malformed_answer | 9 | 11.4% |
| other | 0 | 0.0% |

## Backtracking in Correct Answers

Of the 21 correct answers:
- **16** (76%) included at least one `Backtrack.` step
- **5** (24%) were solved directly without backtracking