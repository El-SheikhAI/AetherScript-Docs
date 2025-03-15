# Vector Operations in AetherScript

## Overview  
Vector operations in AetherScript provide mathematical utilities for manipulating and transforming numeric arrays (vectors). These operations are fundamental for data transformation pipelines, numerical analysis, and machine learning preprocessing workflows.  

---

## Core Operations  

### 1. Vector Addition  
Performs element-wise addition between two equal-length vectors:  

```aetherscript
let result = add_vectors(vector1, vector2)
```  
**Example Input:**  
`vector1 = [2.5, -1.3, 4.0]`, `vector2 = [0.5, 2.7, -1.0]`  
**Output:** `[3.0, 1.4, 3.0]`  

---

### 2. Vector Normalization  
Scales vectors to unit length (L2 norm):  

```aetherscript
let unit_vector = normalize(vector)
```  
**Input Formats:**  
- Single vector: `[x1, x2, ..., xn]`  
- Vector arrays: `[[x1,y1], [x2,y2], ...]`  

---

### 3. Dot Product  
Calculates the dot product between two vectors:  
```aetherscript
let scalar_value = dot_product(vectorA, vectorB)
```  
**Requires:** Vectors of equal length  
**Output:** Single floating-point value  

---

### 4. Cross Product (3D Vectors Only)  
Computes the cross product of two 3-dimensional vectors:  
```aetherscript
let result_vector = cross_product([x1,y1,z1], [x2,y2,z2])
```  

---

## Transformation Pipelines  
Combine vector operations in YAML-based workflows:  

```yaml
pipeline:
  - load_vectors: "data/input.json"
  - transform:
      - normalize: input_vectors
      - dot_product: 
          vectors: [ref:normalize.output, [0.2, 0.4, 0.6]]
  - persist: output_vector
```  

---

## Visual Debugging  
AetherScript CLI provides real-time visualization for vector workflows:  
```bash
aether run pipeline.yaml --visualize
```  
Displays interactive vector magnitude/direction plots during execution.  

---

## Input Specifications  
- **Vector Encoding:** Arrays of floating-point values  
- **Dimension Handling:**  
  - Automatic rank detection (1D/2D)  
  - Implicit broadcasting for compatible operations  

---

> **Next:** [Scalar-Vector Operations](./03-scalar-vector-ops.md)  
> **See Also:** [Plugin Development Guide](../../07-developers/03-plugin-dev.md) for custom operation implementation.