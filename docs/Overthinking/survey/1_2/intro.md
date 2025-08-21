# 1. Constructing Variable-Length CoT Reasoning Datasets

## Post-reasoning CoT Compression. 

This approach collects short CoT data by reducing redundant
reasoning steps after full-length reasoning, either by heuristic criterion or LLMs, as proposed in

* Distilling 2-1
* C3oT
* TokenSkip

## Obtaining Compressed CoT Data during Reasoning. 

This approach collects short CoT data
by prompting LLMs to generate short reasoning steps during inference and reasoning, as proposed
in [70], [82], [37], and [79].



# 2. Fine-Tuning Approaches