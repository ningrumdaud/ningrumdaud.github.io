---
layout: post
title:  "AI.LoveMySkin: Your Smart Shield Against Skincare Mishaps"
author: Ningrum
categories: [ portfolio ]
image: assets/images/AILoveMySkin.png
---

<video width="1000" height="350" controls>
  <source src="../assets/images/AILoveMySkin.mp4" type="video/mp4">
</video>

Skincare routines can be daunting, especially when life forces you to switch products or mix brands. Ever felt confused about whether your Vitamin C serum and retinol might cause irritation together? **AI.LoveMySkin** is a simple yet powerful demo app designed to demystify ingredient interactions and help you avoid adverse reactions.

## Why This Matters
Moving to another country or running out of your go-to product can be stressful. The skincare aisle is full of unfamiliar names, scientific jargon, and brand-specific terms, making it hard for regular users to make safe choices. The app addresses this by analyzing ingredient lists from two products and identifying potential incompatibilities, based on documented scientific references.

## How Does It Work? The Tech Behind the Magic

Step 1: Ingredients Knowledge Base Construction

The first step integrates Domain Expertise and Scientific articles.

- Domain Expertise: Dermatologist-curated rules (e.g., incompatible pairs).
- Scientific Articles: Peer-reviewed research on ingredient interactions

Step 2: Ingredient Extraction

Whether you type the ingredients or upload a photo of the label, the app needs to extract ingredient names.

For text inputs, we clean and preprocess the text (removing percentages, special characters) and then use spaCy’s PhraseMatcher to detect ingredient mentions:

``` python
def preprocess_text(text: str) -> str:
    # Remove concentrations and clean text
    text = re.sub(r'([A-Za-z])\/([A-Za-z])', r'\1 \2', text)
    text = re.sub(r'\(\s*[\d\.]+%?\s*\)', '', text)
    text = re.sub(r'\(\s*[^)]+\s*\)', '', text)
    text = re.sub(r'\s{2,}', ' ', text)
    return text.lower().strip()

def extract_ingredients_from_text(text: str) -> List[str]:
    clean_text = preprocess_text(text)
    doc = nlp(clean_text)
    matches = matcher(doc)
    found_ingredients = set()
    for match_id, start, end in matches:
        ingredient = nlp.vocab.strings[match_id]
        found_ingredients.add(ingredient)
    return sorted(found_ingredients)
```

For images, we leverage OCR with EasyOCR to read the label text and then reuse the same text extraction pipeline:

``` python
def extract_ingredients_from_image(image) -> List[str]:
    if image is None:
        return []
    img_array = np.array(image)
    results = reader.readtext(img_array, detail=0)
    ocr_text = " ".join(results)
    return extract_ingredients_from_text(ocr_text)
```

Step 3: Analysis - Compatibility Checking
Once ingredients are identified from both products, we check for known incompatibilities from our curated knowledge base (KB). The KB contains ingredient groups, aliases, and documented incompatibilities with severity, reasons, and recommendations:

``` python
def check_compatibility(ingredients1: List[str], ingredients2: List[str]) -> Dict:
    conflicts = []
    seen_pairs = set()
    for ing1 in ingredients1:
        for ing2 in ingredients2:
            pair = frozenset({ing1, ing2})
            if pair in seen_pairs:
                continue
            if ing2 in INGREDIENT_KB.get(ing1, {}).get("incompatible_with", {}):
                conflict = INGREDIENT_KB[ing1]["incompatible_with"][ing2]
                conflicts.append({"ingredient_1": ing1, "ingredient_2": ing2, **conflict})
                seen_pairs.add(pair)
            elif ing1 in INGREDIENT_KB.get(ing2, {}).get("incompatible_with", {}):
                conflict = INGREDIENT_KB[ing2]["incompatible_with"][ing1]
                conflicts.append({"ingredient_1": ing2, "ingredient_2": ing1, **conflict})
                seen_pairs.add(pair)
    return {
        "ingredients_product1": ingredients1,
        "ingredients_product2": ingredients2,
        "conflicts": conflicts,
        "has_conflicts": len(conflicts) > 0
    }
```

Step 4: Visualizing Relationships
To make the analysis intuitive, we generate a relational graph using NetworkX and Matplotlib. Nodes represent ingredients, edges highlight incompatible pairs, colored by severity.

``` python
def visualize_detected_ingredients_graph(ingredients1: List[str], ingredients2: List[str], conflicts: List[Dict]) -> str:
    G = nx.Graph()
    for ing in ingredients1 + ingredients2:
        category = INGREDIENT_KB.get(ing, {}).get("category", "Unknown")
        G.add_node(ing, category=category)
    for c in conflicts:
        G.add_edge(c["ingredient_1"], c["ingredient_2"], severity=c["severity"])

    # ... plotting logic, color mapping, and base64 encoding of the image ...
```

This visual aid helps users grasp complex interactions at a glance.


## What’s Next? Future Enhancements

This project is still in its early stages and evolving. Here’s where we want to take it:

1. Knowledge Graph Integration
Rather than a static knowledge base, we plan to construct a dynamic knowledge graph that represents ingredients, their chemical properties, skin effects, and interactions. This will enable richer, multi-hop reasoning about ingredient compatibility beyond direct pairs.

2. Language Model and Retrieval-Augmented Generation (RAG)
Integrating semantic searching with RAG techniques can allow:

- Generating personalized skincare advice based on routine and skin type and scientific articles
- Expanding the ingredient database by scraping and synthesizing scientific literature on demand

3. Context-Aware Recommendations
By combining user skin data (e.g., sensitivities, existing conditions) with the knowledge graph and LLM insights, we can provide highly tailored skincare compatibility assessments and suggestions.

## Final Thoughts
AI.LoveMySkin is a friendly introduction to using NLP, OCR, and graph analysis to solve everyday skincare dilemmas. While simple now, it’s a foundation for a sophisticated, AI-powered skincare assistant. Stay tuned as we build smarter, more interactive tools that make skincare safer and science more accessible!

