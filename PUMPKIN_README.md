Following `agents/README.md`, set up instructions:
- If running on Mac, no need to install [Sysbox](https://github.com/nestybox/sysbox/issues/846) if you are already using Docker Desktop (enterprise version). Sysbox was acquired by Docker and can be turned on by updating your settings in Docker Desktop (General > check Use Enhanced Container Isolation)

Get Dummy Agent Running:
- Build mlebench-env locally 
```
# Build mlebench-env locally 
docker build --platform=linux/amd64 -t mlebench-env -f environment/Dockerfile .

# Prepare superlite dataset
mlebench prepare -c aerial-cactus-identification 

python run_agent.py --agent-id aide --competition-set experiments/splits/superlite.txt

# Generate a submission JSONL file, find run-group under runs/<your run>metadata.json for your latest run

python experiments/make_submission.py --metadata runs/<run-group>/metadata.json --output runs/<run-group>/submission.jsonl

# Grade submission (leaderboard.csv must be real CSV, not a Git LFS pointer)
git lfs pull   # or: mlebench dev download-leaderboard -c aerial-cactus-identification --force
mlebench grade --submission runs/<run_group>/submission.jsonl --output-dir runs/<run_group>
```

Notes
- mlagentbench fails on build due to incompatibility with old `cchardet` library
- 