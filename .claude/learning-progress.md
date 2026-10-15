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

### 2. Residual stream
- `x₀ = embedding`, `x₁ = x₀ + Attn₁(x₀)`, `x₂ = x₁ + MLP₁(x₁)`, …
- The vector at any layer equals the embedding plus the outputs of all earlier blocks. Each block reads `x`, computes something, and adds it back.
- Residual connection is the addition. The residual stream is the vector that receives those additions across the whole model.
- It is the vector at **every** layer (x₀ … x_L), not only the final one. The paper reads `xᵢ` at each layer and gives each layer its own SAE.
- The user's own wording: "residual stream là output tại 1 lớp layer khi sử dụng phương pháp residual connection".

### 3. One residual stream per token
- At each layer, the sentence is a matrix `n_tokens × d_model` (Pythia-70M: 512, Gemma-2-2B: 2304), with one row per token.
- Row i of X_l is token i's residual stream at layer l. Row i across X₀…X_L is that token's residual stream across layers. The whole thing is a 3-D block `layer × token × d_model`.
- A node in the paper's circuit is specified by feature, layer, and token position.

### 4. Attention: how tokens exchange information
- MLP reads only its own token. Attention is the only step that reads other tokens, and it is causal: it reads only earlier tokens and the token itself.
- Weights: `q = x·W_Q` for the current token and `k = x·W_K` for every token, then `q·k`, then softmax. `W_Q` and `W_K` are learned. The weights are recomputed for every sentence.
- Softmax never gives exactly 0. The only zeros come from the causal mask on later tokens. The sum runs over **all** tokens from position 1 up to the current one.
- `Attn_output = Σ weight_j · v_j` with `v = x·W_V`. It has the same dimension as `x`. The list of weights (one number per token) is **not** what gets added.
- `x[token] = x[token] + Attn_output[token]`. Without a residual connection, `x` would be replaced.
- Confusions the user had and that are now resolved:
  - "Weight" vs "vector".
  - "Only tokens with weight > 0 are summed".
- The user's own wording: "output của từ đó qua lớp attention bằng vector của từ đó + attention output".

### 5. Layer structure and the 3 spots where the paper places SAEs
- Pythia is decoder-only, so the whole model is the decoder: `embedding → [Attn₀ → MLP₀] → … → [Attn₅ → MLP₅] → unembedding`. After MLP of layer i comes Attn of layer i+1. There is no separate decoder block.
- Within one layer: `x_mid = x + Attn(x)` and `x_out = x_mid + MLP(x_mid)`, so `x_out = x + Attn(x) + MLP(x_mid)`. Example: [1,1] + [2,0] + [0,5] = [3,6].
- In each layer the paper reads 3 vectors:
  1. Attention output, before it is added to x.
  2. MLP output, before it is added to x.
  3. Residual stream at the **end** of the layer (`x_out`).
- The middle vector `x + Attn` is not used.
- Paper Appendix (p.21): "we index residual stream activations to be the layer which outputs the activation (so the layer 0 residual stream is not the embeddings, and the layer 5 residual stream is the output of the final layer, immediately preceding the final decoder)". Here "final decoder" means the unembedding.
- Pythia-70M: 6 layers × 3 spots + 1 embedding = **19 SAEs**.
- Gemma-2-2B uses Gemma Scope SAEs: 26 layers, width 16384, no embedding SAE. Its attention and MLP SAEs sit at different points than in Pythia (input to the attention out-projection, and after the post-MLP LayerNorm). This has not been told to the user yet.

### 6. Slide 5: the two models, circuit, node
- Paper analyzes Pythia-70M and Gemma-2-2B.
- Circuit = small graph of nodes and edges that causes ONE specific behavior; keeps only nodes with large effect on output; not a single path.
- Node = one unit (feature in the paper, neuron in the baseline) at a specific layer/spot and token position. Not necessarily a neuron.
- Feature was introduced briefly (a weighted combination of neurons = a direction, learned by an SAE attached outside the model). The user found it hard; it is deferred to group 2. For now the user only needs: neuron is a real component, feature is a way of reading neuron values.

### 7. Slide 6: Neuron (paper's definition)
- Footnote 1: "neuron" = a basis-aligned direction in latent space, not necessarily preceded by a nonlinearity. So a neuron is **one coordinate (one number)** of an activation vector.
- Usual meaning: a hidden unit inside the MLP, output after GELU. Paper's meaning is broader: any coordinate of any vector. Same granularity (one number), not something smaller inside an MLP neuron.
- The neuron baseline uses nodes at the 3 SAE spots (attention output, MLP output, residual stream), 512 neurons each in Pythia. MLP hidden units (2048) are not used as nodes.
- Taught with `neuron-diagram.html` (toy model: 4 dims, 6 hidden units, 1 layer, 1 token).
- The user twice restated it as "a layer" (first "smaller units inside an MLP neuron", then "input layer, output layer, MLP layer with activation"). Corrected: a neuron is one number, not a layer. Re-check this if the user says "neuron" for a whole vector later.

### 8. Slide 7: Polysemantic
- One neuron fires strongly on several unrelated kinds of input (HTML, Korean names, football), so reading that one neuron does not tell which concept is present.
- Geometry (taught with `polysemantic-diagram.html`, 2-D toy): the whole vector is a point; each neuron is one axis, its value is the coordinate. 2 concepts on the 2 axes → monosemantic. 3 concepts at 120° → HTML [1.7, 1.0] and Korean name [−1.7, 1.0] both give neuron #2 = 1.0 → polysemantic. The full vector still distinguishes them.
- Corrections given when the user restated it:
  - "512 vectors" → one vector with 512 numbers = 512 axes.
  - "look at neurons > 0" → negative values count too (−1.7); meaning is the direction of the whole vector, not the set of positive neurons.
  - Neurons do not "generate" directions; the model learns to place each concept along a direction. "More concept directions than axes" is superposition (the cause); polysemantic is what we observe on one neuron (the consequence).

### 9. Superposition (confirmed)
- Superposition requires more concepts than neurons; 2 concepts on 2 axes is separated but not superposition.
- Neuron vs weight, confirmed in the user's words: neuron = each coordinate of a layer's vector (activation, changes per input); weight = w in w·x (fixed after training, updated only during training). Inputs x_i…x_y are the neurons entering f(x). Added: weights exist in embedding and attention (W_Q, W_K, W_V, W_O) too, not only in MLP.
- Taught with step 4 of `polysemantic-diagram.html`: slider for N concepts in 2-D, toggle concepts, calculation tables A (build vector = sum of 2·(cos θ, sin θ)) and B (read = x₁·cos θ + x₂·sin θ, then cut negatives).
- Points covered: more directions than axes; sparse = rare co-occurrence makes the rare misreads acceptable (3 concepts all on → [0,0]); close directions (N=10, 36°) misread even with one concept; 90° apart → sum still reads back exactly.
- Corrected: interference is geometry, not a learned belief that concepts "go together".
- User asked how directions get there. Answered: cos/sin is only the toy's notation for a 2-D direction; the model stores directions as weight numbers (embedding rows, rows of later weight matrices); reading = dot product done by a later layer's weight row; all directions start random and move together at every gradient step; nobody picks the number of concepts or which neuron, training does; this is why interpretability is needed (paper explains how a specific behavior is produced, not "why the model is good").

## Next concept (resume here)
- The user is still in group "1 Bối cảnh". Do NOT jump ahead to group 2.
- Slide 8 Circuit was introduced (earlier circuits used heads / whole MLP as units, too coarse and polysemantic; paper uses SAE features). The user chose to move on without full confirmation.
- Still unclear for the user: neuron (activation) vs weight, and which steps have weights (linear layers: one weight row per output neuron) vs none (residual addition). The user wants a spiral approach: go through the whole deck, then loop back. Revisit this on the second pass, e.g. when SAE's W_enc / W_dec come up.
- Group 2 started. Next: **Dictionary learning** (introduced; waiting for confirmation), then SAE.
- Paper only cites Elhage et al. 2022 for polysemanticity/superposition; it does not explain superposition itself.
- SAE (group 2) comes after group 1 is finished.

## Key numbers from the paper (verified)
- Circuit size: Pythia 86 nodes, Gemma 223 nodes. Faithfulness ≈ 0.21.
- SHIFT: Pythia 67 features, 55 of them ablated. Gemma 46 features, 43 ablated.
- SHIFT+retrain profession accuracy: Pythia 93.1, Gemma 95.0.
- Pythia: 19 SAEs, 32768 features each. IG uses N=10. Clusters 382 and 475.
