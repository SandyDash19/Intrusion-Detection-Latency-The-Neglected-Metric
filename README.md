# Intrusion-Detection-Latency-The-Neglected-Metric
This code is for the results in table 7 in the paper Intrusion Detection Latency: The Neglected Metric.

Attack Detection Latency (ADL) Evaluation on ROSPaCe

This repository contains code to compute Attack Detection Latency (ADL) for network-based intrusion detection using the ROSPaCe dataset and an Extreme Gradient Boosting (XGBoost) classifier.

The implementation supports per-attack-instance latency analysis, enabling evaluation beyond traditional accuracy-only metrics.

Overview

Dataset: ROSPaCe (feature engineering already performed; see paper for details)

Classifier: Extreme Gradient Boosting (XGBoost)

Task: Binary classification (Observe vs. Attack) with per-class latency evaluation

Scale:

Test datapoints: 2,315,344 (~2.3 million)

Attack subsegments (test set): 194

Attack Categories

| Label | Attack Category      |
| ----: | -------------------- |
|     0 | Observe              |
|     1 | nmap_discovery       |
|     2 | ros2_node_crashing   |
|     3 | ros2_reconnaissance  |
|     4 | ros2_reflection      |
|     5 | nmap_SYN_flood       |
|     6 | metasploit_SYN_flood |

What the Code Does

The script trains an XGBoost binary classifier (observe vs. attack) and evaluates:

1. Classification Metrics

Binary accuracy, FPR, FNR on the test set

Per-attack-category datapoint-level accuracy (TPR / recall)

2. Detection Index per Attack Subsegment

For each attack subsegment in the test set, the code records:

the global datapoint index (det_idx) at which the attack is first detected

subsegments may span hundreds of thousands of datapoints

3. Attack Detection Latency (ADL)

For each successfully detected attack subsegment, ADL is computed as:

The Attack Detection Latency is computed as:

## Attack Detection Latency (ADL)

```math
\mathrm{ADL}_{ms}
=
\left( \mathrm{ACT}_{detected} - \mathrm{ACT}_{start} \right)_{ms}
+
k \cdot \mathrm{INF}_{const,ms}

​
where:

ACT_start / ACT_detected are dataset timestamps (converted to milliseconds)

k is the number of datapoints processed from attack start to detection (inclusive)

INF_const is a constant per-datapoint inference time estimated from throughput

Inference time is measured by timing batch inference over the test set and averaging across 5 runs.

nference time is measured by timing batch inference over the test set and averaging across 5 runs.

Outputs

The script produces the following artifacts:

(1) test_data_with_predictions.csv

Contains all test datapoints (~2.3 million rows)

Includes:

timestamps

engineered features

ground-truth attack labels

predicted labels

prediction probabilities

(2) subseg_label_detidx.csv

One row per attack subsegment

Columns:

subsegment_id

label (attack category)

det_idx (datapoint index where attack is first detected)

This file enables precise alignment between detection time and attack progression.

(3) detection_events.csv

Detailed per-attack-instance latency breakdown

Includes:

ACT latency (timestamp difference)

inference contribution

total ADL

subsegment length

detection offsets

If an attack subsegment is not detected, latency fields are left empty.

Summary Statistics Table (Printed to Console)

For each attack category, the code prints a summary table with the following columns:

| Column                   | Description                                          |
| ------------------------ | ---------------------------------------------------- |
| `class_name`             | Attack category                                      |
| `accuracy_percent`       | Datapoint-level accuracy (recall/TPR) for the attack |
| `total_subsegments`      | Total number of attack subsegments in the test set   |
| `detected_subsegments`   | Number of subsegments successfully detected          |
| `inf_const_us_per_point` | Estimated inference time per datapoint (µs)          |
| `avg_adl`                | Average ADL for detected subsegments                 |
| `max_adl`                | Maximum ADL for detected subsegments                 |
| `std_adl`                | Standard deviation of ADL                            |

Latency values are displayed with units attached (ms or µs depending on magnitude).

Important Notes

The XGBoost model fails to detect attacks in the following categories:

ros2_node_crashing

ros2_reflection

As a result, ADL statistics for these categories are not reported (empty / N/A).

Per-datapoint inference time is assumed to be constant because:

the feature dimensionality is fixed

XGBoost inference complexity does not depend on packet content

GPU inference timing is estimated via throughput and may be affected by asynchronous execution; results should be interpreted accordingly.

Citation

If you use this code or the produced artifacts, please cite the accompanying paper:

Paper Title: Intrusion Detection Latency: The Neglected Metric
Authors: Sandhyarani Dash, John M. Acken
Affiliation: Portland State University
Journal:Cybersecurity, Springer Nature

Disclaimer

This repository is intended as a research artifact to support reproducibility of Attack Detection Latency analysis.
