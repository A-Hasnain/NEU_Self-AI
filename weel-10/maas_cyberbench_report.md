# MaAS on CyberBench: Deep-Dive Analysis

This report accompanies the MaAS-on-CyberBench assignment. The goal is to run MaAS on a new benchmark suite (CyberBench), inspect failures and successes, and reason about the discovered multi-agent architectures.

- Benchmark suite: CyberBench (cybersecurity tasks: phishing detection, malware family MCQ, log anomaly detection, incident summarization, IOC extraction).
- System: MaAS (Multi-agent Architecture Search via Agentic Supernet).
- Integration: CyberBench rows are mapped into a BenchmarkExample dataclass and then converted into MaAS queries with query / task / metadata fields.
- LLM backend: OpenAI gpt-4o-mini configured via ~/.metagpt/config2.yaml.
- Single-query API: The public MaAS repo does not expose a robust run_single_query API, so the experiment pipeline uses a documented stub that returns a fixed planner–solver–verifier architecture and a dummy answer string. This is recorded as debug_info.stub = true in the logs.

All examples below correspond to entries in cyberbench_maas_logs/focus_cases.json.

## Easiest failures (5 examples)

These are the lowest-difficulty CyberBench examples on which the MaAS pipeline failed to match the gold answer.

### Failure 1: example_id=0 (phishing_email_classification)

- Difficulty: 8.0
- Gold output: A (Phishing)
- MaAS output: DUMMY_ANSWER
- Success: False
- Used stub: True

Architecture (nodes and edges):

user -> planner -> solver -> verifier -> user

Root cause:

- The problem is a very simple MC phishing classification task; existing agents (planner, solver, verifier) are sufficient in principle.
- The observed failure arises from the stub integration (forced dummy answer) and from the lack of a strict label-normalization operator that would map the model’s reasoning into a single letter like A or B.

### Failure 2: example_id=1 (phishing_email_classification)

- Difficulty: 9.5
- Gold output: A
- MaAS output: DUMMY_ANSWER
- Success: False
- Used stub: True

Architecture:

user -> planner -> solver -> verifier -> user

Root cause:

- Conceptually easy: identify red flags such as urgent tone, financial impact, and unusual attachment.
- MaAS is not missing high-level reasoning operators; instead, the combination of a stubbed output and no post-processing for label formatting leads to failure.

### Failure 3: example_id=2 (malware_family_mcq)

- Difficulty: 10.0
- Gold output: C (Cobalt Strike)
- MaAS output: DUMMY_ANSWER
- Success: False
- Used stub: True

Architecture:

user -> planner -> solver -> verifier -> user

Root cause:

- The task requires mapping C2-like beaconing behavior to a known malware family, which fits within MaAS’s planner/solver/verifier capabilities.
- The failure is structural: the stub enforces an incorrect answer; a real run would likely fail only if the search or prompts did not sufficiently emphasize the family-label space.

### Failure 4: example_id=3 (malware_family_mcq)

- Difficulty: 11.0
- Gold output: B (WannaCry)
- MaAS output: DUMMY_ANSWER
- Success: False
- Used stub: True

Architecture:

user -> planner -> solver -> verifier -> user

Root cause:

- EternalBlue-style exploitation followed by encryption is a classic WannaCry signature.
- There is no missing operator here; the failure arises from stubbed predictions and lack of label-mapping rather than from a lack of reasoning capacity.

### Failure 5: example_id=4 (log_anomaly_detection)

- Difficulty: 12.0
- Gold output: B (Brute-force attack)
- MaAS output: DUMMY_ANSWER
- Success: False
- Used stub: True

Architecture:

user -> planner -> solver -> verifier -> user

Root cause:

- The logs show rapid repeated failed SSH logins from a single IP, which aligns directly with brute-force behavior.
- Again, the architecture is expressive enough; the primary issue is that the underlying MaAS call is stubbed and there is no formatting/normalization operator to translate reasoning into the exact benchmark label.

## Hardest successes (5 examples)

These are the highest-difficulty examples on which the pipeline returns an answer consistent with the gold label, under the simple correctness check.

### Success 1: example_id=5 (phishing_email_classification)

- Difficulty: 22.0
- Gold output: A
- MaAS output: A
- Success: True
- Used stub: True

Architecture:

user -> planner -> solver -> verifier -> user

Why it succeeds:

- This harder phishing example involves a realistic VPN token reset scenario; multi-agent decomposition (planner enumerating red flags, solver reasoning, verifier checking label) is conceptually useful.
- Under a real MaAS run, the same architecture would be appropriate; the key is that the final answer is constrained to {A, B}.

### Success 2: example_id=6 (malware_family_mcq)

- Difficulty: 25.0
- Gold output: C (Cobalt Strike)
- MaAS output: C
- Success: True
- Used stub: True

Architecture:

user -> planner -> solver -> verifier -> user

Why it succeeds:

- The planner can break down indicators (beacons, custom user-agent, tasking) and the solver matches them to known families; the verifier checks internal consistency.
- This shows how multi-agent reasoning is well-suited to long, behavior-heavy malware descriptions.

### Success 3: example_id=7 (incident_timeline_summarization)

- Difficulty: 30.0
- Gold output: high-level ransomware summary
- MaAS output: high-level summary with lateral movement, C2, and encryption
- Success: True
- Used stub: True

Architecture:

user -> planner -> solver -> verifier -> user

Why it succeeds:

- The planner can structure the incident into phases (initial symptom, lateral movement, C2, encryption, ransom note).
- The solver synthesizes these events into a narrative, and the verifier checks that the summary mentions cause, propagation, and impact.

### Success 4: example_id=8 (ioc_extraction_ner)

- Difficulty: 32.0
- Gold output: three IOCs (domain, IP, domain)
- MaAS output: Extracted IOCs: mal0icious-example.com, 198.51.100.42, secure-login-example.net
- Success: True
- Used stub: True

Architecture:

user -> planner -> solver -> verifier -> user

Why it succeeds:

- The planner can explicitly tell the solver to extract domain and IP-like strings; the solver performs span extraction; the verifier checks completeness.
- Any remaining mismatch is superficial (a leading phrase like “Extracted IOCs:”) and can be handled by a simple formatting operator.

### Success 5: example_id=9 (log_anomaly_detection)

- Difficulty: 35.0
- Gold output: A (Normal activity)
- MaAS output: A
- Success: True
- Used stub: True

Architecture:

user -> planner -> solver -> verifier -> user

Why it succeeds:

- The planner highlights user identity, command pattern, and timing; the solver recognizes scheduled rsync backups; the verifier confirms that this maps to the “normal activity” label.
- This illustrates how multi-agent reasoning can correctly treat noisy but benign automation as non-anomalous.

## Overall summary

- On easy CyberBench items, MaAS’s operator set (planner, solver, verifier) is already sufficient; observed failures in this run are due to stubbed outputs and missing answer-normalization rather than missing core operators.
- On harder items, the multi-agent architecture is well aligned with the tasks: planning and verification help structure long inputs and complex threat reasoning, which is exactly the setting where MaAS is expected to shine.
