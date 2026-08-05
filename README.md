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

Optional environment variables:

    JOBS_OUTPUT_CSV    output path (default: finance_jobs.csv beside the script)
    ADZUNA_APP_ID      \ enable Adzuna; free tier is 1000 calls/month and this
    ADZUNA_APP_KEY     / script spends one call per country per run
    ADZUNA_COUNTRIES   comma-separated Adzuna country codes (default: gb,us,nl,de)
    RAPIDAPI_KEY       enables JSearch

Adzuna is a general job board rather than a remote-only one, so its `Remote`
column is inferred from the posting text and is False when unproven — unlike the
remote-only sources, where it is True by construction.

Not LinkedIn: LinkedIn exposes no job-search API to individual developers, and
its unauthenticated jobs endpoint breaches their User Agreement. See the header
of `FinanceJobs` for details.
