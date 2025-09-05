Please carefully read the [main README.md](../../../README.md), which is stored in the benchmark's root folder, before following this subject-specific guideline.

# Fuzzing nanomq server with AFLNet and AFLnwe
Please follow the steps below to run and collect experimental results for nanomq, which is a popular MQTT server.

## Step-1. Build a docker image
The following commands create a docker image tagged nanomq. The image should have everything available for fuzzing and code coverage calculation.

```bash
cd $PFBENCH
cd subjects/MQTT/nanomq
docker build . -t nanomq
```

## Step-2. Run fuzzing

```bash
cd $PFBENCH
mkdir results-nanomq

profuzzbench_exec_common.sh nanomq 4 results-nanomq aflnet out-nanomq-aflnet "-m none -P MQTT -D 10000 -q 3 -s 3 -E -K -R" 3600 5
```

## Step-3. Collect the results

```bash
cd $PFBENCH/results-nanomq

profuzzbench_generate_csv.sh nanomq 4 aflnet results.csv 0

```

## Step-4. Analyze the results
The results collected in step 3 (i.e., results.csv) can be used for plotting. Use the following command to plot the coverage over time and save it to a file.

```
cd $PFBENCH/results-nanomq

profuzzbench_plot.py -i results.csv -p nanomq -r 4 -c 60 -s 1 -o cov_over_time.png
```
