# Editorial Research Assistant — RAG Fix Task

## Task Overview

A digital media company runs a RAG-powered research assistant that helps journalists surface relevant article history and background context. The assistant ingests article chunks nightly into a pgvector database and answers questions through a FastAPI service. Currently, when the system returns wrong or hallucinated answers, engineers have no way to determine whether the fault lies in retrieval (wrong chunks returned) or generation (LLM ignoring good chunks) — there's no visibility into what happened during a request. A diagnostic endpoint that should help investigate issues is unacceptably slow under realistic data volumes, and engineers have noticed that diagnostic history for past requests is sometimes simply gone when they go looking for it. Additionally, the team has an evaluation dataset but no way to measure retrieval quality, so regressions go undetected — and when the system has nothing genuinely relevant to work with, it still produces a confident answer anyway.

## Objectives

- Answering a research question produces enough diagnostic information, after the fact, to determine whether a bad answer was caused by retrieval (wrong chunks) or generation (LLM ignoring good chunks) — including how long each stage took, how much of the model's budget was consumed, and exactly which evidence was used.
- Diagnostic detail for a past request remains available and correct even after the service has been restarted or redeployed, and serving it does not consume ever-growing resources as usage accumulates.
- The diagnostic endpoint responds in under 500 ms and returns results in manageable pages — and the cost of serving one page should not depend on how much total data sits behind it.
- The evaluation script reports retrieval-quality metrics (recall@5 and MRR) for the current retriever against the provided query dataset. Note that a question may have more than one relevant source, and the metrics must account for all of them.
- When the retriever cannot find sufficiently relevant evidence for a question, the system returns a safe fallback response instead of forcing an answer from irrelevant context. A fallback must be distinguishable from a normal answer by a client program, visible to operators after the fact, and the definition of "sufficiently relevant" must be tunable per environment without touching code.
- All code follows production standards: typed function signatures, structured exception handling, and no secrets in source files.

## Helpful Tips

- Think about what a support engineer would need to see, after the fact, to tell a retrieval failure from a generation failure for a single request — and where in the request lifecycle that information would need to be captured.
- If an endpoint is slow, profile it before you guess — don't assume the bottleneck without confirming where the time is actually going, or how many round trips it takes to get there.
- "It works until we redeploy" is a symptom worth taking seriously: ask yourself what state the system depends on, and where that state actually lives.
- Look up how recall@k and MRR are conventionally defined in information retrieval — and check how those definitions behave when a query has several correct answers rather than one.
- Consider what a RAG system should do when it genuinely doesn't have relevant information to answer with, versus forcing an answer from whatever it retrieved — and how both a program and a human operator would find out that this happened.

## How to Verify

- Ask a research question and confirm the application logs contain an entry for that request that would let an engineer tell whether the time went to retrieval or generation, and which chunks were used.
- Look up diagnostics for a prior request, restart the service, and confirm the same lookup still works and still arrives quickly, in manageable pages.
- Run the evaluation script and confirm it reports recall@5 and MRR without errors — including sensible scores for queries that have more than one relevant source.
- Ask a question on a topic not covered by any ingested article and confirm the response contains a fallback message, that the response itself makes the fallback unambiguous to a client, and that the event is visible in the logs.
- Change the relevance cutoff without editing any source file and confirm the fallback behavior shifts accordingly.
- Run the test suite and confirm all tests pass.
