# AI Candidate Ranking System

## Objective

Develop an AI-powered candidate ranking system that intelligently identifies the top 100 candidates for a given Job Description (JD) by combining semantic understanding, candidate metadata, career history, and behavioral signals.

---

# Overall Architecture

```
                    Job Description
                           │
                           ▼
                  Universal JD Parser
                           │
                           ▼
                Semantic JD Embedding
                           │
                           ▼
             Candidate Database (Offline)
                           │
        ┌──────────────────┼──────────────────┐
        ▼                  ▼                  ▼
  Candidate Embedding   Metadata         Behavioral Signals
        │                  │                  │
        └──────────────────┼──────────────────┘
                           ▼
                  Hybrid Ranking Engine
                           │
                           ▼
                   Top-100 Candidate Heap
                           │
                           ▼
                     Generate Reasoning
                           │
                           ▼
                    submission.csv
```

---

# Solution Overview

The solution is divided into two stages.

## Stage 1 : Offline Candidate Preprocessing

Since embedding generation is computationally expensive, all candidate profiles are preprocessed only once.

For every candidate, we generate:

- Semantic embedding
- Career metadata
- Skills
- Education
- Recruiter signals
- Experience
- Other structured information

These are stored for future ranking.

### Stored Files

```
candidate_embeddings.npy
candidate_metadata.parquet
```

This preprocessing is independent of the Job Description, allowing the same candidate database to be reused for different hiring requests.

---

## Stage 2 : Online Ranking

When a recruiter uploads a Job Description:

1. Parse the JD
2. Generate a semantic embedding
3. Compare against all candidate embeddings
4. Compute additional ranking features
5. Produce the Top-100 candidates
6. Generate explanations
7. Export submission.csv

Only lightweight computations are performed during ranking, satisfying the CPU and runtime constraints.

---

# Job Description Understanding

The JD is converted into a structured representation.

Extracted information includes:

- Experience requirements
- Mandatory requirements
- Preferred requirements
- Negative requirements
- Important requirement sentences

Instead of relying on keyword matching, the entire JD is embedded into a semantic vector.

This enables understanding of contextual similarity rather than exact wording.

---

# Candidate Representation

Each candidate is represented using two complementary views.

## 1. Semantic Representation

A unified candidate document is constructed using:

- Headline
- Summary
- Career history
- Skills
- Education
- Certifications

The document is encoded into a dense semantic embedding.

---

## 2. Structured Representation

Additional structured information is preserved:

- Years of experience
- Skill list
- Career history
- Recruiter activity
- Verification status
- Notice period
- Open-to-work status
- Interview completion
- GitHub activity

These features are later incorporated into the ranking score.

---

# Hybrid Ranking

Ranking is not based on semantic similarity alone.

The final score combines multiple signals.

## Semantic Similarity

Measures contextual similarity between the JD and candidate profile using sentence embeddings.

---

## Experience Match

Rewards candidates whose experience falls within the desired range.

---

## Skill Match

Measures overlap between candidate skills and concepts present in the JD.

---

## Behavioral Score

Uses recruiter activity signals such as:

- Profile completeness
- Recruiter response rate
- Interview completion
- GitHub activity
- Search appearances
- Saved by recruiters
- Open-to-work status

These signals estimate candidate responsiveness and hiring likelihood.

---

## Penalty Score

Candidates are penalized for undesirable characteristics such as:

- Very long notice period
- Low recruiter response
- Extremely slow response time
- Insufficient experience

---

# Final Ranking Formula

The final score is computed as a weighted combination of multiple independent signals.

```
Final Score =

Semantic Similarity
+ Experience Match
+ Behavioral Score
+ Skill Match
− Penalty
```

This hybrid approach provides significantly better ranking quality than keyword search or semantic similarity alone.

---

# Efficient Top-K Selection

Instead of storing every candidate in memory and sorting later, a Min Heap of size 100 is maintained.

```
Read Candidate

↓

Compute Score

↓

Update Heap

↓

Discard Candidate
```

This ensures:

- O(N log K) complexity
- Constant memory usage
- Efficient processing of very large datasets

where:

- N = total candidates
- K = 100

---

# Reason Generation

For every shortlisted candidate, a concise explanation is generated using factual information extracted from the profile.

Example:

> Strong semantic alignment with the JD. 6.5 years of relevant experience and excellent recruiter engagement signals.

No hallucinated information is introduced.

---

# Why This Approach?

Compared to traditional keyword search, this system offers:

- Semantic understanding of the Job Description
- Context-aware candidate matching
- Integration of behavioral hiring signals
- Scalable processing for large candidate pools
- Efficient Top-100 retrieval under strict runtime constraints

---

# Time Complexity

Offline preprocessing:

```
O(N)
```

Performed only once.

---

Online ranking:

```
Embedding Comparison : O(N × d)

Feature Computation : O(N)

Heap Update : O(N log 100)
```

where:

- N = Number of candidates
- d = Embedding dimension (384)

Since `log(100)` is constant, the overall complexity is effectively linear in the number of candidates.

---

# Advantages

- Generic across different job domains
- Independent of handcrafted keyword rules
- Reusable candidate database
- Supports arbitrary future JDs
- CPU-efficient
- Memory-efficient
- Produces explainable rankings
- Suitable for production-scale recruitment systems