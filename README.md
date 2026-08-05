# Script
Lamba python m code

## FinanceJobs
Collects remote/global finance jobs from public job APIs (Remotive, RemoteOK,
Arbeitnow, Himalayas; Adzuna and JSearch enable themselves if their keys are set)
and dedup-appends them to a CSV.

    pip install httpx pandas
    python FinanceJobs              # collect
    python FinanceJobs --probe      # dump API shapes
    python FinanceJobs --self-test  # offline checks

Not LinkedIn: LinkedIn exposes no job-search API to individual developers, and
its unauthenticated jobs endpoint breaches their User Agreement. See the header
of `FinanceJobs` for details.
