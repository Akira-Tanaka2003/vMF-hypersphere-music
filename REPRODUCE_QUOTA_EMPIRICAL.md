# Reproduce: Conformer-vMF Full Arrangement Generation

This repository provides a reproducible pipeline for generating full-arrangement MIDI using a vMF hypersphere music representation and a Conformer-mediated block-level harmonic function transition model.

## Main checkpoint

Download the trained checkpoint from the release page and place it at:

```text
checkpoints/vmf_conformer_block_transition_pop1k7_1000_finetune_from_head_best.pt
```

This checkpoint contains a Conformer model with a block-level function transition head.

The model predicts block-level harmonic functions:

```text
T / D / SD / OTHER
```

and uses quota-aware empirical decoding during generation.

## Target function distribution

The generation preset follows the training target distribution:

```text
T     = 0.4615
D     = 0.1378
SD    = 0.1928
OTHER = 0.2079
```

This prevents autoregressive generation from collapsing into a T/D loop and keeps the generated function distribution close to the training data.

## Install

```bash
pip install -r requirements.txt
```

## Generate full arrangement MIDI

```bash
python scripts/generate_vmf_full_arrangement_conformer_block.py \
  --checkpoint checkpoints/vmf_conformer_block_transition_pop1k7_1000_finetune_from_head_best.pt \
  --out_midi results/generated/vmf_full_arrangement_quota_empirical_seed5.mid \
  --out_json results/generated/vmf_full_arrangement_quota_empirical_seed5_stats.json \
  --key C \
  --blocks 16 \
  --steps_per_block 8 \
  --tempo 120 \
  --seed 5
```

## Generate multiple seeds

```bash
for seed in 1 2 3 4 5
do
  python scripts/generate_vmf_full_arrangement_conformer_block.py \
    --checkpoint checkpoints/vmf_conformer_block_transition_pop1k7_1000_finetune_from_head_best.pt \
    --out_midi results/generated/vmf_full_arrangement_quota_empirical_seed${seed}.mid \
    --out_json results/generated/vmf_full_arrangement_quota_empirical_seed${seed}_stats.json \
    --key C \
    --blocks 16 \
    --steps_per_block 8 \
    --tempo 120 \
    --seed ${seed}
done
```

## Output tracks

The generated MIDI contains:

```text
1. Melody
2. Chord comping
3. Bass
4. Arpeggio
5. Pad
```

## Expected function distribution

For 16 blocks, the empirical target roughly corresponds to:

```text
T     : 7 to 8 blocks
D     : 2 to 3 blocks
SD    : about 3 blocks
OTHER : 2 to 3 blocks
```

The quota-aware empirical setting is intended to preserve learned Conformer transition logits while preventing over-concentration on tonic and dominant functions.
