# Cosmos Data Factory Migration Guide

PAIDF -> CDF rename, and OSMO orchestration -> Airflow orchestration

**Status:** Airflow is the orchestration layer for all CDF blueprints. OSMO remains supported as an execution backend. The OSMO-orchestrated blueprint skills are deprecated but fully functional - security fixes only, no new features. The full environment is snapshotted at tag [paidf-26.10](https://github.com/NVIDIA/cosmos-data-factory/releases/tag/paidf-26.10). OSMO as an orchestrator is a roadmap item for a later release. This guide states no dates or commitments.

## Who this guide is for

| You are... | Do this |
|---|---|
| New to CDF | Skip this guide. Start at the [Getting Started Guide](https://github.com/NVIDIA/cdf-orchestration/blob/main/docs/getting-started.md). |
| Running the OSMO-orchestrated DIG or VDA from Computex | Your skills still work. They are deprecated: security fixes only. Pin to tag `paidf-26.10` for the full environment. Do not mix the tag with current main. |
| Running IAA or EVG | No behavior change. The DAG IDs (`image_attribute_augmentation_dag_k8s`, `event_video_generation_dag_k8s`) and their payload schemas are unchanged. Only repo and skill names changed. Update to current main and use the name map below. |
| In a proof of concept on OSMO orchestration | Restart the PoC on Airflow orchestration now. Do not build new work on the deprecated path. |

## Two things changed at once - the names tell them apart

1. **The name.** Physical AI Data Factory (PAIDF) is now NVIDIA Cosmos Data Factory (CDF). Repos, agent skills, and containers now use `cdf-*` names.
2. **The orchestrator.** Before, two paths existed: DIG and VDA ran as OSMO agent skills (OSMO was the orchestrator), while IAA and EVG ran as Airflow DAGs. Now there is one path: all blueprints run as Airflow DAGs through the CDF Orchestrator. The execution backend is set in the DAG payload: Kubernetes for every blueprint, with OSMO also supported for EVG and VDA.

Because both changed together, the name is the marker. For DIG and VDA, old names (`physical-ai-*`, `paidf-*`) mean the deprecated OSMO-orchestration version. For IAA and EVG, which always ran on Airflow, old names are just the pre-rename names. New names (`cdf-*`) always mean Airflow orchestration. If the material in front of you has an old name, it is pre-change material.

## The word "OSMO" changed meaning

Before this change, "OSMO" meant the orchestrator: `osmo workflow submit`, OSMO YAMLs, pools, pod templates. After this change, "OSMO" names a backend option inside an Airflow DAG payload - an existing Kubernetes cluster with OSMO installed. Any text that says "submit to OSMO" without that context is old material.

## Name map

### Repos (GitHub redirects old links automatically)

| Old | New |
|---|---|
| NVIDIA/physical-ai-data-factory | NVIDIA/cosmos-data-factory |
| NVIDIA/paidf-orchestration | NVIDIA/cdf-orchestration |
| NVIDIA/paidf-auto-labeling | NVIDIA/cdf-auto-labeling |
| NVIDIA/paidf-augmentation | NVIDIA/cdf-augmentation |
| NVIDIA/paidf-anomalygen | NVIDIA/cdf-anomalygen |
| NVIDIA/paidf-simulation | NVIDIA/cdf-simulation |
| NVIDIA/paidf-curation-and-retrieval | NVIDIA/cdf-curation-and-retrieval |

### Agent skills (no redirects - old names still resolve, deprecated)

Old skill names still work. Each old skill is fully functional. Use the successor listed below.

| Old skill (deprecated, functional) | Successor | What changed |
|---|---|---|
| physical-ai-defect-image-generation (v1.0.1: Cosmos-Predict2.5 AnomalyGen, released per-use-case checkpoints) | cdf-defect-image-generation | OSMO YAML submit -> Airflow DAG payload; backend is a parameter |
| physical-ai-defect-image-generation-v1-1 (v1.1.0: Cosmos3 Nano/Edge base + per-use-case LoRA, no released checkpoints) | cdf-defect-image-generation | Same |
| physical-ai-video-data-augmentation (v1.0.0) | cdf-video-data-augmentation | Same |
| physical-ai-image-attribute-augmentation | cdf-image-attribute-augmentation | Rename only - was always Airflow; DAG ID and payload unchanged |
| physical-ai-event-video-generation | cdf-event-video-generation | Rename only - was always Airflow; DAG ID and payload unchanged |
| paidf-orchestration-setup | cdf-orchestration-setup | Rename only |
| paidf-orchestration-write-dag | cdf-orchestration-write-dag | Rename only |
| paidf-auto-labeling | cdf-auto-labeling | Rename only |
| paidf-augmentation | cdf-augmentation | Rename only |
| paidf-curation-and-retrieval | cdf-curation-and-retrieval | Rename only |
| paidf-cosmos-predict | cdf-cosmos-predict | Rename only |

## Containers

New workloads pull `cdf-*` containers (for example `nvcr.io/nvidia/cdf-anomalygen`). The old containers are published as `nvcr.io/nvidia/paidf-*` (for example `nvcr.io/nvidia/paidf-anomalygen`). They leave the NGC catalog listing but keep pulling at their existing paths, so deprecated deployments keep working. No sunset date will be set before migration tooling exists.

## Contract changes per blueprint

Every blueprint on the CDF Orchestrator uses the same mechanics: trigger a DAG from the Airflow UI with a JSON payload, monitor in Grid or Graph view, read results under `<output_directory>/<run_id>/`. The tables below map each old OSMO concept onto that pattern.

### Defect Image Generation

| Concern | OSMO orchestration (deprecated) | Airflow orchestration (current) |
|---|---|---|
| Submit | `osmo workflow submit assets/configs/<flow>.yaml --set ...` | Airflow UI -> Dags -> `defect_image_generation_smt_dag_k8s` -> Trigger -> paste the JSON into the payload run parameter. |
| Flow selection | 6 OSMO YAMLs (Day 0 texture, Day 0 good-image, Day 0 structural, Day 1 real-alignment, Day 1 manual-ROI, finetune) | The `pipeline_mode` field at the payload root: `full`, `sim2real`, `simulation`, or `defect`. |
| Knobs | `--set crop_max_emit=`, `render_patches=`, `checkpoint_step=`, `existing_finetune_url=` | Payload JSON fields. Start from the starter payloads in `cdf-orchestration/payloads/dev/defect_image_generation_smt_dag/` and refer to the [DIG payload guide](https://github.com/NVIDIA/cdf-orchestration/blob/main/docs/defect-image-generation/payload-guide.md) for each field. |
| Storage | `dig_url_root` on the OSMO data profile (default `s3://osmo-workflows/dig`) | `input_path` and `output_directory` in the payload: S3 URIs on object storage the cluster can reach. |
| Monitoring | `osmo workflow query` / `osmo workflow logs` | Airflow UI Grid or Graph view (`make port-forward`, then `localhost:8080`). To stop a run, mark the running task failed so services shut down cleanly. Never delete the DagRun. |
| Outputs | `<dig_url_root>/runs/<name>/anomaly/` | `<output_directory>/<run_id>/`, with golden images under `goldens/<golden_id>/augmentation/artifact/` and defect images under `defects/<golden_id>/<defect_id>/artifact/`. Each `artifact/` folder has an adjacent `manifest.json`. `run_id` shows in the Airflow UI after trigger. |

### Video Data Augmentation

| Concern | OSMO orchestration (deprecated) | Airflow orchestration (current) |
|---|---|---|
| Submit | `osmo workflow submit assets/configs/osmo/<flow>.yaml` | Airflow UI -> Dags -> `video_data_augmentation_dag_k8s` -> Trigger -> paste the JSON into the payload run parameter. |
| Flows | `auto_labeling`, `augmentation_and_al`, `e2e`, `e2e_super_resolution` | One DAG, no flow selector. What the old flows chose is now configured per task in the payload: `super_resolution`, `detection_and_tracking`, `captioning`, `visual_qa`, `reasoning` and `training_export` each have their own section. |
| Storage | `storage_url` on the active dataset backend | `input_path` and `output_directory` in the payload. |
| Monitoring | OSMO CLI | Airflow UI, same rules as DIG. |
| Outputs | Per-run OSMO data URLs | `<output_directory>/<run_id>/`, with `cosmos/` for generated videos, `auto_labeling_input/` and `auto_labeling_output/` for labels, and `training_export/auto_labeling_input/` and `training_export/auto_labeling_output/` for the two training exports. The workflow does not combine the branches into a single dataset manifest. |
| Demo data | `scripts/prepare_demo_assets.sh` | Starter payload in `cdf-orchestration/payloads/dev/video_data_augmentation_dag/`. |

## Your assets survive the change

Your data and models outlive the orchestrator. They stay where they are in object storage. These carry forward:

- Finetuned AnomalyGen checkpoints under `<dig_url_root>/models/<usecase>/`
- Raw and generated datasets under `<dig_url_root>/datasets/` and `runs/`
- VDA inputs and outputs under your `storage_url`
- Hugging Face gated-model access you already accepted

Point the new payload's `input_path` at those existing paths, and validate with a small run first. A verified step-by-step for each asset class is planned alongside the migration tooling.

## Fast follow

Migration tooling for OSMO-orchestration deployments is planned. This guide states no scope, deliverables, or dates.

## Getting help

You do not have to figure the migration out alone.

- **Migration questions or blocked deployments:** open an issue in this repo. Include: the blueprint (DIG, VDA, IAA, EVG), which version you run (deprecated OSMO-orchestrated, or current main), and what you are trying to do.
- **Partners with an NVIDIA contact:** your existing channel works too. Reference this guide so we can route you fast.
- **Found a gap in this guide?** Every question this guide cannot answer is a defect in the guide. File it and we will fold the answer back in.

## FAQ

- **Can I run the deprecated version and current main side by side?** Yes. Use separate checkouts and separate agent sessions. Do not point one agent at both.
- **Will the deprecated skills get fixes?** Security fixes only. No new features.
- **I found a skill with an old `physical-ai-*` or `paidf-*` name. Is it broken?** No. It still works, and existing deployments keep running it. Its `cdf-*` successor is listed in the name map above.
- **Does "CDF" change what the blueprints do?** No. The rename is branding. The orchestration change is the functional one. This guide covers both because they shipped together.
- **My agent remembers my old `dig_url_root` and pool. Are they still valid?** The pool and pod-template values do not apply to Airflow orchestration - discard them. The storage URL may carry forward as an `input_path`; the new skills ask you to confirm it before reuse.
- **I run IAA or EVG today. Do I need to do anything?** Update to current main for the new names. Your DAG IDs and payloads keep working as they are.