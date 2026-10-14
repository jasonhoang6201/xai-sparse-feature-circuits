# Learning progress

The user studies the paper keyword by keyword using `keywords-slides.html`, reads each slide, and asks about anything unclear. Update this file after each concept the user confirms.

## Concepts the user has confirmed

### 1. Residual connection / skip connection
- `output = x + f(x)`, where `f` is attention or MLP. `x` passes straight through the block and is not replaced.
- Purpose: to prevent **vanishing gradient**. In backward, the chain rule multiplies the derivatives of every layer. If each factor is < 1 (e.g. 0.5), the product goes to 0 after many layers: 0.5¹⁰ ≈ 0.001. With a residual connection, `dy/dx = 1 + f'(x)`, so each factor is ≈ 1 and gradient reaches the first layers.
- **Skip connection** is the general name for any connection that bypasses layers. **Residual connection** is the additive kind (`x + f(x)`).
- **U-Net vs Transformer/ResNet:**
  - U-Net uses concat from an encoder layer to the decoder layer at the same resolution, to keep spatial detail lost during downsampling. It is called a skip connection, not a residual connection.
  - ResNet (conv layers) and Transformer (attention + MLP) add `x` to the output of each block.
  - Transformer does not downsample: every layer has the same dimension (Pythia-70M: 512).
- The Pythia and Gemma models in the paper are **decoder-only**, with no encoder and no cross-attention.

### 2. Residual stream (explained, waiting for the user to confirm)
- `x₀ = embedding`, `x₁ = x₀ + Attn₁(x₀)`, `x₂ = x₁ + MLP₁(x₁)`, …
- The vector at any layer equals the embedding plus the outputs of all earlier blocks. Each block reads `x`, computes something, and adds it back.
- Residual connection is the addition. The residual stream is the vector that receives those additions across the whole model.

## Next concept
- **Why each token has its own residual stream:**
  - Each token position has its own vector with `d_model` dimensions (Pythia-70M: 512, Gemma-2-2B: 2304).
  - Positions exchange information only through attention. MLP acts on each position separately.
- After that: in each layer the paper places SAEs at 3 spots: residual stream, attention output, and MLP output.

## Key numbers from the paper (verified)
- Circuit size: Pythia 86 nodes, Gemma 223 nodes. Faithfulness ≈ 0.21.
- SHIFT: Pythia 67 features, 55 of them ablated. Gemma 46 features, 43 ablated.
- SHIFT+retrain profession accuracy: Pythia 93.1, Gemma 95.0.
- Pythia: 19 SAEs, 32768 features each. IG uses N=10. Clusters 382 and 475.
