# Unit 9 Capstone Results

## Before Improvement

| Criterion | Trial 1 | Trial 2 | Trial 3 | Verdict |
| --- | --- | --- | --- | --- |
| 1. Hosted service availability | PASS | PASS | PASS | MET — 3/3 |
| 2. Existing FitFindr functionality still works after deployment | PASS | PASS | PASS | MET — 3/3 |
| 3. Structured logging captures each completed request | PASS | PASS | PASS | MET — 3/3 |
| 4. Invalid input is rejected cleanly | PASS | PASS | PASS | MET — 3/3 |
| 5. Warm hosted requests complete within 30 seconds | 21.66s | 4.47s | 4.92s | MET — 3/3 |

All five criteria were met before the improvement.

While reviewing the structured logs, I noticed that rejected and crashed requests included an explicit `status` field, but successful requests did not. This made successful requests less consistent and slightly harder to filter in the hosted logs.

## Improvement

I added `status="ok"` to the structured log record for successful `/ask` requests.

Before:

```json
{"timestamp": "...", "query": "vintage graphic tee under $30", "output": "...", "seconds": 4.92}
```

After:

```json
{"timestamp": "...", "status": "ok", "query": "vintage graphic tee under $30", "output": "...", "seconds": 4.76}
```

This gives all request outcomes an explicit status:

- `ok`
- `rejected`
- `crashed`

## After Improvement

| Criterion | Trial 1 | Trial 2 | Trial 3 | Verdict |
| --- | --- | --- | --- | --- |
| 1. Hosted service availability | PASS | PASS | PASS | MET — 3/3 |
| 2. Existing FitFindr functionality still works after deployment | PASS | PASS | PASS | MET — 3/3 |
| 3. Structured logging captures each completed request | PASS | PASS | PASS | MET — 3/3 |
| 4. Invalid input is rejected cleanly | PASS | PASS | PASS | MET — 3/3 |
| 5. Warm hosted requests complete within 30 seconds | 18.84s | 5.29s | 4.76s | MET — 3/3 |

The improvement worked as intended. Successful hosted requests now include `status="ok"` while the existing recommendation behavior, invalid-input handling, and timing targets continue to pass.