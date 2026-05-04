---
name: holographic-memory
description: "Holographic Reduced Representations (HRR) for compositional vector memory — bind, unbind, and bundle concepts with algebraic retrieval"
version: "1.0.0"
tags: [memory, vectors, holographic, embeddings, knowledge]
createdBy: "built-in"
status: "active"
---

# Holographic Memory (HRR)

## Activation
When the user needs compositional vector memory that encodes structured relationships between concepts, or when building knowledge stores that support algebraic retrieval.

## Concept

Holographic Reduced Representations (HRR) encode compositional structure into fixed-width distributed representations using phase vectors. Each concept is a vector of angles in [0, 2pi). Three core operations:

| Operation | Math | Purpose |
|-----------|------|---------|
| **bind** | phase addition (circular convolution) | Associate two concepts |
| **unbind** | phase subtraction (circular correlation) | Retrieve a bound value |
| **bundle** | circular mean of complex exponentials | Merge multiple concepts |

Phase encoding is numerically stable, avoids magnitude collapse, and maps cleanly to cosine similarity.

## Core Operations

### Encode an Atom
Deterministic phase vector via SHA-256 counter blocks. Same word always produces same vector, across processes and machines.

```python
from holographic import encode_atom

# Create atomic concept vectors (dim=1024 default)
cat_vec = encode_atom("cat", dim=1024)
animal_vec = encode_atom("animal", dim=1024)
```

### Bind (Associate)
Create a composite vector that is dissimilar to both inputs (quasi-orthogonal):

```python
from holographic import bind

# "cat IS-A animal" — associate the two concepts
cat_is_animal = bind(cat_vec, animal_vec)
# Result is dissimilar to both cat_vec and animal_vec
```

### Unbind (Retrieve)
Recover a bound value using the key:

```python
from holographic import unbind, similarity

# Retrieve: unbind(bind(cat, animal), cat) ≈ animal
retrieved = unbind(cat_is_animal, cat_vec)
print(similarity(retrieved, animal_vec))  # ≈ 1.0
```

### Bundle (Superpose)
Merge multiple vectors into one that is similar to each input:

```python
from holographic import bundle

# Combine multiple facts into a single memory vector
memory = bundle(fact1_vec, fact2_vec, fact3_vec)
# memory is similar to each input
# Capacity: O(sqrt(dim)) items before degradation
```

### Text Encoding
Bag-of-words: bundle of atom vectors for each token:

```python
from holographic import encode_text

doc_vec = encode_text("the cat sat on the mat", dim=1024)
```

### Structured Fact Encoding
Content bound to role vectors, entities bound to entity role, all bundled:

```python
from holographic import encode_fact

fact = encode_fact(
    content="cats are domesticated animals",
    entities=["cat", "animal", "domesticated"],
    dim=1024
)
# Enables algebraic extraction:
# unbind(fact, bind(entity, ROLE_ENTITY)) ≈ content_vector
```

## Similarity
Phase cosine similarity, range [-1, 1]:
```python
from holographic import similarity

score = similarity(vec_a, vec_b)
# 1.0 = identical, ~0.0 = unrelated, -1.0 = anti-correlated
```

## Capacity and SNR

Signal-to-noise ratio: `SNR = sqrt(dim / n_items)`

| Dimension | Max Items (SNR > 2) | Use Case |
|-----------|---------------------|----------|
| 256 | ~64 | Small knowledge bases |
| 1024 | ~256 | General purpose |
| 4096 | ~1024 | Large knowledge stores |

When SNR drops below 2.0, retrieval errors become likely.

## Serialization

```python
from holographic import phases_to_bytes, bytes_to_phases

# Save to disk or database
data = phases_to_bytes(vector)  # 8KB at dim=1024

# Restore
vector = bytes_to_phases(data)
```

## Properties
- **Deterministic**: Same word always produces same vector (SHA-256 based)
- **Cross-platform**: Identical across processes, machines, languages
- **Numerically stable**: Phase encoding avoids magnitude collapse
- **Composable**: Bind/unbind/bundle compose arbitrarily
- **Fixed width**: All vectors same dimension regardless of content complexity

## Use Cases
- Compositional knowledge graphs in fixed-width vectors
- Semantic memory with algebraic querying
- Associative memory stores for agent context
- Fact extraction and retrieval
- Multi-hop reasoning through bind/unbind chains
