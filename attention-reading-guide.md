# Attention Is All You Need — Reading Group Prep Guide

**Paper:** *Attention Is All You Need* — Vaswani et al., 2017
**arXiv ID:** `1706.03762`

> All facts below were pulled fresh from the paper's LaTeX source (abstract, Model Architecture, Results, and Conclusion sections) for the reading group prep pack.

## Headline results — WMT 2014 machine translation

| Task | Model | BLEU | Context |
|------|-------|:----:|---------|
| WMT 2014 **English→German** | Transformer (big) | **28.4** | New state of the art — beats the best previously reported results, *including ensembles*, by more than 2.0 BLEU |
| WMT 2014 **English→French** | Transformer (big) | **41.8** | New single-model state of the art (abstract + Table 1) |

- The big EN→DE model trained for **3.5 days on 8 P100 GPUs**.
- Even the **base** model surpasses all previously published single models *and* ensembles: **27.3** BLEU EN→DE and **38.1** BLEU EN→FR, at a training cost of 3.3·10^18 FLOPs — a fraction of any competitive model.
- Reading-group trivia: the Results prose reports **41.0** BLEU for EN→FR while the abstract and Table 1 say **41.8** — worth checking when you read the paper.

## Architecture facts

Base model unless stated otherwise:

1. **Layers: N = 6.** The encoder is a stack of 6 identical layers; the decoder is also a stack of 6 identical layers.
2. **Attention heads: h = 8.** Multi-head attention runs 8 parallel heads with d_k = d_v = 64.
3. **Model dimension: d_model = 512.** All sub-layers, and the embedding layers, produce outputs of dimension 512.
4. **Feed-forward inner dimension: d_ff = 2048.**
5. **Sub-layer structure.** Each encoder layer = multi-head self-attention + position-wise feed-forward network. Each decoder layer adds a third sub-layer: multi-head attention over the output of the encoder stack.
6. **Residual + LayerNorm everywhere.** Every sub-layer output is LayerNorm(x + Sublayer(x)).
7. **Masked decoder self-attention.** Positions cannot attend to subsequent positions, so the prediction for position *i* depends only on outputs at positions < *i*.
8. **Scaled dot-product attention.** Attention(Q, K, V) = softmax(QKᵀ/√d_k)·V — dot products are scaled by 1/√d_k to keep softmax gradients healthy.
9. **Big model config** (Table 3): N = 6, d_model = 1024, d_ff = 4096, h = 16 heads, P_drop = 0.3, ~213M parameters (base: ~65M).
10. **Checkpoint averaging & decoding.** Base models averaged the last 5 checkpoints (written at 10-minute intervals); big models the last 20. Beam search with beam size 4 and length penalty α = 0.6; max output length = input length + 50.

## Data lineage

The findings in this guide are maintained in the reading group's Snowflake warehouse as **one row per finding** in the table **`reading_group.notes.paper_summaries`**, with columns `arxiv_id`, `section`, and `finding`.

> **Status at publication:** the Snowflake account was suspended at the time this guide was generated (error 250001 — free trial ended, all virtual warehouses suspended), so the `reading_group` database, the `notes` schema, and the `paper_summaries` table **could not be created or loaded**, and the read-back row count is **not yet available**. 13 findings are staged below in the Appendix, ready to load once the account is reactivated — the expected read-back count is **13**.

## Appendix — staged Snowflake DDL + data (run once the account is restored)

```sql
CREATE DATABASE reading_group;
CREATE SCHEMA reading_group.notes;
CREATE TABLE reading_group.notes.paper_summaries (
    arxiv_id TEXT,
    section  TEXT,
    finding  TEXT
);

INSERT INTO reading_group.notes.paper_summaries (arxiv_id, section, finding) VALUES
('1706.03762', 'Model Architecture', 'The encoder is a stack of N=6 identical layers; each layer has two sub-layers: a multi-head self-attention mechanism and a simple position-wise fully connected feed-forward network.'),
('1706.03762', 'Model Architecture', 'The decoder is also a stack of N=6 identical layers; in addition to the two encoder sub-layers, each decoder layer inserts a third sub-layer performing multi-head attention over the output of the encoder stack.'),
('1706.03762', 'Model Architecture', 'All sub-layers in the model, as well as the embedding layers, produce outputs of dimension d_model = 512.'),
('1706.03762', 'Model Architecture', 'Every sub-layer output is LayerNorm(x + Sublayer(x)): a residual connection around each sub-layer followed by layer normalization.'),
('1706.03762', 'Model Architecture', 'Decoder self-attention is masked so positions cannot attend to subsequent positions, ensuring predictions for position i depend only on known outputs at positions less than i.'),
('1706.03762', 'Model Architecture', 'Attention is Scaled Dot-Product Attention: Attention(Q, K, V) = softmax(QK^T / sqrt(d_k)) V; dot products are scaled by 1/sqrt(d_k) to avoid pushing the softmax into regions with extremely small gradients.'),
('1706.03762', 'Model Architecture', 'The base model uses h=8 parallel attention heads with d_k = d_v = 64, and position-wise feed-forward networks with inner-layer dimension d_ff = 2048.'),
('1706.03762', 'Results', 'WMT 2014 English-to-German: the big Transformer model sets a new state-of-the-art BLEU score of 28.4, outperforming the best previously reported models (including ensembles) by more than 2.0 BLEU; training took 3.5 days on 8 P100 GPUs.'),
('1706.03762', 'Results', 'WMT 2014 English-to-French: the big Transformer model reaches 41.8 BLEU per Table 1 (the abstract calls it a new single-model state of the art); the prose in this section instead reports 41.0. The big EN-FR model used dropout P_drop = 0.1 rather than 0.3.'),
('1706.03762', 'Results', 'Even the base model surpasses all previously published single models and ensembles: 27.3 BLEU on EN-DE and 38.1 BLEU on EN-FR at a training cost of 3.3e18 FLOPs.'),
('1706.03762', 'Results', 'Base model config (Table 3): N=6, d_model=512, d_ff=2048, h=8, d_k=d_v=64, P_drop=0.1, label smoothing 0.1, 65M parameters; big model: N=6, d_model=1024, d_ff=4096, h=16, P_drop=0.3, 213M parameters.'),
('1706.03762', 'Results', 'Base models were averaged over the last 5 checkpoints written at 10-minute intervals; big models averaged the last 20. Decoding used beam search with beam size 4 and length penalty alpha = 0.6, with maximum output length set to input length + 50.'),
('1706.03762', 'Conclusion', 'The Transformer is the first sequence transduction model based entirely on attention, replacing the recurrent layers commonly used in encoder-decoder architectures with multi-headed self-attention; it sets a new state of the art on both WMT 2014 translation tasks, and the code is available at github.com/tensorflow/tensor2tensor.');

-- Read-back check (expected: 13)
SELECT COUNT(*) AS row_count FROM reading_group.notes.paper_summaries;
```
