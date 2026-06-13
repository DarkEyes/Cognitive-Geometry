# Interpretation as Linear Transformation: A Cognitive-Geometric Model of Concepts and Meaning

[![arXiv](https://img.shields.io/badge/arXiv-2512.09831-b31b1b.svg)](https://arxiv.org/abs/2512.09831)
[![Python](https://img.shields.io/badge/python-3.8%2B-blue.svg)](https://www.python.org/)
[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)

Code accompanying the paper **"Interpretation as Linear Transformation: A Cognitive-Geometric Model of Concepts and Meaning"** ([arXiv:2512.09831](https://arxiv.org/abs/2512.09831)).

This repository reproduces the paper's empirical illustration (Section 4.3 and Appendix C): a minimal, observable-data test of the framework's central *structural* prediction — that concepts cross a domain boundary intact only to the extent that the two agents share the relevant representational basis.

---

## The framework in brief

**Cognitive Geometry (CG)** models concepts, motivation, and influence across cognitively heterogeneous agents in linear-algebraic terms.

- Each agent is a personalized **value space** `V_i` — a finite-dimensional real vector space whose basis vectors are the agent's evaluative dimensions.
- A concept is an **abstract being**: a vector `X ∈ V_i`. Its norm encodes subjective importance; its direction encodes the concept's internal organization. (*Concept* is the general term; a concept an agent endorses is a **belief**.)
- Communication from agent `A` to agent `B` is a linear **interpretation map** `T_{A→B} : V_A → V_B`. A concept survives transmission only if it avoids the **null space** of that map — components that fall into the null space are structurally invisible to `B`, regardless of either agent's rationality or evidence.
- The headline result, the **No-Null-Space Leadership Condition**, recasts leadership as *representational reachability*: `L` influences agent `i` with respect to concept `X` if and only if the composite interpretation map along some path from `L` to `i` does not annihilate `X`.

Stated as bare mathematics, an interpretation map could annihilate anything. The **shared-basis hypothesis** constrains when maps are well-behaved: agents who participate in a common occupational or disciplinary practice acquire, through repeated joint activity, the same basis vectors that practice requires. On that shared task subspace the round-trip map is near-invertible (bounded distortion). Null spaces bite at the *boundaries* of shared bases — in cross-domain, cross-disciplinary, and cross-cultural exchange. **The experiment in this repository tests exactly this prediction.**

---

## What this repository contains

| File | Description |
|---|---|
| `Cognitive_Geometry.ipynb` | Self-contained notebook that embeds the concept set, estimates the interpretation map, performs the subspace decomposition, runs the statistical tests, and prints the per-concept diagnostics. |

The notebook is the supplementary material referenced in Appendix C of the paper.

---

## The experiment

Two stylized agents — a **philosopher** and an **engineer** — stand in for two cognitive bases that share a substantive functional core (inference, justification, consistency, robustness, identity-through-change, well-defined specification) while each retains a sharply domain-specific technical vocabulary the other does not share.

**Concept set** (full list in Appendix C of the paper):

- **15 paired sentences** expressing the same functional role in each profession's register — e.g. *a valid argument preserves truth from premises to conclusion* ↔ *a sound structural design transfers loads from the applied force to the support*; *the ship of Theseus retains its identity through replacement of its parts* ↔ *the bridge retains its function through replacement of structural members*.
- **5 domain-specific jargon sentences per side** with no significant neighbor in the other domain — *qualia, transcendental idealism, Husserlian epoché, …* for the philosopher; *von Mises stress, Reynolds number, the Paris law of fatigue crack growth, …* for the engineer.

**Method**

1. **Embed** every sentence into `R^384` with the pretrained sentence encoder `all-MiniLM-L6-v2` (`sentence-transformers`).
2. **Estimate the interpretation map** `M : V_phil → V_eng` by **orthogonal Procrustes alignment** on the 15 paired embeddings. Because `M` is orthogonal, the reverse direction is `Mᵀ`.
3. **Subspace decomposition.** For a mapped vector, split it into the component lying within the span of the receiver's paired-concept embeddings and the residual lying outside that span (via SVD of the receiver basis, rank tolerance `1e-6`). The fraction of squared L² mass **outside** the recoverable subspace is the empirical proxy for null-space annihilation: a high value means the concept's representation leans on directions the receiver's task vocabulary does not span.
4. **Leave-one-out (LOO) for paired concepts.** Each paired concept is decomposed against the subspace spanned by the *other fourteen* receivers, so a concept's own partner cannot trivially saturate the within-subspace mass.
5. **Statistics.** One-sided **Mann-Whitney U** (primary) and **Welch's t-test** (supplementary), testing whether jargon outside-fractions exceed paired outside-fractions, in each direction and pooled.

---

## Results

| Concept type | Recon. err. (L²) | Within (frac) | Outside (frac) | Outside % |
|---|---|---|---|---|
| Paired, LOO (φ → Eng) | 0.250 ± 0.057 | 0.264 ± 0.085 | 0.736 ± 0.085 | **73.6%** |
| Paired, LOO (Eng → φ) | 0.250 ± 0.057 | 0.268 ± 0.069 | 0.732 ± 0.069 | **73.2%** |
| Philosopher-only jargon (φ → Eng) | — | 0.170 ± 0.048 | 0.830 ± 0.048 | **83.0%** |
| Engineering-only jargon (Eng → φ) | — | 0.129 ± 0.051 | 0.871 ± 0.051 | **87.1%** |

*n* = 15 paired, 5 jargon per side, embedding dimension 384. Fractions are of squared L² mass; within + outside ≈ 1 by orthogonality.

**Significance** (one-sided, jargon outside-fraction > paired outside-fraction):

| Direction | Jargon vs. paired | Mann-Whitney U (*p*) | Welch's t (*p*) |
|---|---|---|---|
| φ → Eng | 0.830 vs. 0.736 | U = 64.0 (*p* = 0.010) | t = 2.87 (*p* = 0.007) |
| Eng → φ | 0.871 vs. 0.732 | U = 71.0 (*p* < 0.001) | t = 4.46 (*p* < 0.001) |
| Pooled | 0.851 vs. 0.734 | U = 269.0 (*p* < 0.001) | t = 5.12 (*p* < 0.001) |

**Reading the numbers.** The result is the *gap*, not the absolute level. Outside-fractions are high even for paired concepts, but their reconstruction error is low (≈ 0.25) and their within-subspace mass is consistently and significantly higher than that of domain-specific jargon, in both the forward and reverse directions. This is the predicted asymmetry: concepts exercising the shared inferential/specificational basis transmit across `M` with lower null-space residual, while concepts that depend on one domain's private vocabulary project largely outside the receiver's recoverable subspace.

The per-concept diagnostics (printed by the notebook; tabulated in Appendix C) show that the effect is **graded, not binary**. Pairings whose surface vocabulary diverges more sharply than the central inferential cluster — *Ship of Theseus / structural-member replacement* and *concept grounding / well-defined requirements* — climb into the 84–90% range, approaching the jargon level. Shared-basis structure is a matter of degree.

---

## Scope of the demonstration

This is a **proof of operationalizability, not an empirical validation** of the framework. Four caveats apply (see Section 4.3.6 of the paper):

- The concept set is author-constructed rather than drawn from an expert-validated cross-domain corpus.
- Sentence embeddings encode *distributional* semantic similarity; here they serve as a tractable surrogate for value spaces, not a faithful instantiation of evaluative/motivational dimensions.
- Orthogonal Procrustes constrains `M` to a rotation — more restrictive than the general linear maps the framework allows. An unconstrained least-squares estimator would yield richer transformations at the cost of more parameters.
- The sample (15 paired, 5 jargon per side) is small relative to the embedding dimensionality; the tests support an illustrative observation, not a population-level claim about philosophy and engineering as domains.

Full empirical instantiation — larger expert-validated concept sets and estimation of interpretation maps from interactive discourse rather than static pairings — is left to future work.

---

## Installation

```bash
pip install sentence-transformers scipy numpy
```

A virtual environment is recommended. The first run downloads the `all-MiniLM-L6-v2` model weights (a few hundred MB).

## Usage

Open and run the notebook top to bottom:

```bash
jupyter notebook Cognitive_Geometry.ipynb
```

Or execute it headless:

```bash
jupyter nbconvert --to notebook --execute Cognitive_Geometry.ipynb
```

The single code cell prints the summary table, the statistical tests, and the per-concept outside-fractions in both directions. Output should match the table above up to small numerical variation.

---

## Citation

If you use this work, please cite the paper:

```bibtex
@misc{amornbunchornvej2025interpretation,
  title         = {Interpretation as Linear Transformation: A Cognitive-Geometric Model of Concepts and Meaning},
  author        = {Amornbunchornvej, Chainarong},
  year          = {2025},
  eprint        = {2512.09831},
  archivePrefix = {arXiv},
  primaryClass  = {cs.AI},
  doi           = {10.48550/arXiv.2512.09831},
  url           = {https://arxiv.org/abs/2512.09831}
}
```

---

## License

The code in this repository is released under the MIT License (see [`LICENSE`](LICENSE)). The accompanying paper is distributed under the [arXiv non-exclusive license](http://arxiv.org/licenses/nonexclusive-distrib/1.0/).

## Author

Chainarong Amornbunchornvej — [arXiv:2512.09831](https://arxiv.org/abs/2512.09831)
