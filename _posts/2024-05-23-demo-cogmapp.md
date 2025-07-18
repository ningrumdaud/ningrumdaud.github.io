---
layout: post
title:  "CogMApp Demo: Extract and Visualize Conceptual Relationships from Textual Data"
author: Ningrum
categories: [ portfolio ]
image: assets/images/cogmapp_small.png
---

<video width="1000" height="350" controls>
  <source src="../assets/images/CogMApp-Demo.mp4" type="video/mp4">
</video>

CogMApp is a tool I developed to extract and visualize conceptual relationships from textual data. By analyzing input text, it identifies key concepts, maps their connections, and generates interactive cognitive diagrams, enabling users to uncover hidden patterns and insights in unstructured content.

## Introduction
In this blog post, we will walk through a sample code that demonstrates how to create CogMApp, a cognitive map analysis tool from text. Cognitive maps are a great way to visualize and understand the relationships between different concepts. This tool utilizes natural language processing (NLP) techniques to extract and visualize these relationships. We will break down the algorithm and pipeline used, and discuss areas for potential improvement.

## (Current) Scope of the CogMApp
The primary focus of the code is to:
1. Identify causal expressions within the text. CogMApp uses the pre-defined lexicon of causal markers to detect sentences that contain causal relationships. The lexicon is constructed based on a study conducted by Altenberg (1984).
2. Extract words and phrases that indicate causality, ensuring that these phrases are descriptive. The aim is to get clear information, not just chunks of text.
3. Identify the direction of causal relationships between the extracted words and phrases. CogMApp determines their role in the text, whether as a root (effects) or as branches (causes).
4. Detect one degree of causality at a simple sentence level. It also works well to identify multiple effects and causes within the text, providing a comprehensive view of causal relationships.
5. Visualise the cognitive map from the text, focusing on the causal mapping.

## Pipeline
![CogMApp Pipeline](../assets/images/cogmap-pipeline.png "CogMApp Pipeline")

The pipeline for the cognitive map analysis tool is illustrated in the following steps. It starts with the input text that needs to be analyzed for causal relationships.

**1. Detecting Causal Expression**

CogMApp first detects the presence of causal expressions in the text using a predefined lexicon of causal markers. This step ensures that only relevant sentences containing causal relationships are processed further. This step is Lexicon-Based which uses a predefined lexicon of causal markers to identify sentences that contain causal expressions. To construct the lexicon, a list of causal links from Altenberg (1984) was used. These markers are read from an external file and include words and phrases such as 'because', 'due to', 'results in', etc. In the following code, the causal marker lexicon is called 'words_list'.

``` python
def contains_words_or_phrases(words_list, sentence):
    """
    Check if any word or phrase from words_list is present in the sentence.
    :param words_list: List of words or phrases to check
    :param sentence: The input sentence where to look for words or phrases
    :return: True if any word or phrase is found, otherwise False
    """
    # Process the sentence with spaCy to obtain the lemmatized form of each token
    processed_sentence = nlp(sentence.lower())
    lemmatized_sentence = " ".join(token.lemma_ for token in processed_sentence)

    # Process each word or phrase for lemmatization
    for word_or_phrase in words_list:
        # Process and create a lemma string for the word or phrase
        processed_word_or_phrase = nlp(word_or_phrase.lower())
        lemmatized_word_or_phrase = " ".join(token.lemma_ for token in processed_word_or_phrase)

        # Check if the lemmatized word or phrase is in the lemmatized sentence
        if lemmatized_word_or_phrase in lemmatized_sentence:
            return True  # Return True immediately if any word or phrase is found

    return False  # Return False if none of the words or phrases are found
```

**2. Extracting Words/Phrases**

The next step is to extract descriptive words or phrases from the text using spaCy’s dependency parsing. This involves merging noun phrases in certain cases to form more meaningful and descriptive phrases. During this process, the dependencies of each word are tracked, and specific rules are applied to determine when to merge phrases.

As shown in the following code box, the merging logic considers 'phrase connectors' to decide whether to merge noun phrases. For example, in the phrase "The introduction _of_ new law," the connector 'of' prompts the merging of "The introduction" and "new law" into a single noun phrase.

``` python
class NounExtractor:
    def __init__(self, nlp):
        """
        Initialize the NounExtractor with a pre-loaded spaCy NLP model.
        """
        self.nlp = nlp

    def process_text(self, text):
        """
        Process the text using the spaCy NLP pipeline.
        """
        return self.nlp(text)

    def get_noun_phrases(self, doc):
        """
        Extract and refine noun phrases from the spaCy doc, tracking and using dependency labels accurately.
        """
        noun_phrases = list(doc.noun_chunks)
        merged_phrases = []
        skip_indexes = set()  # Indexes to skip because they have been merged into another phrase
        list_dep_labels = [token.dep_ for token in doc]  # List of dependency labels for each token

        for i in range(len(noun_phrases)):
            if i in skip_indexes:
                continue

            current = noun_phrases[i]
            # Collect dependency labels for the current noun phrase
            deps_in_phrase = {list_dep_labels[tok.i] for tok in current}

            # Merge logic based on 'phrases connectors' construction
            if i + 1 < len(noun_phrases) and (doc[current.end].text in ['of', 'in', 'among', 'on', 'towards', 'to', 'for', 'across']):
                next_phrase = noun_phrases[i + 1]
                if i + 2 < len(noun_phrases) and doc[next_phrase.end].dep_ == 'pcomp':
                    extended_phrase = doc[current.start:noun_phrases[i + 2].end]
                    skip_indexes.update({i + 1, i + 2})
                    extended_deps = {list_dep_labels[tok.i] for tok in extended_phrase}
                    dep_label = self.determine_dep_label(extended_deps)
                    merged_phrases.append((extended_phrase.text, dep_label))
                    continue
                else:
                    merged_phrase = doc[current.start:next_phrase.end]
                    skip_indexes.add(i + 1)
                    merged_deps = {list_dep_labels[tok.i] for tok in merged_phrase}
                    dep_label = self.determine_dep_label(merged_deps)
                    merged_phrases.append((merged_phrase.text, dep_label))
                    continue

            if i not in skip_indexes:
                dep_label = self.determine_dep_label(deps_in_phrase)
                merged_phrases.append((current.text, dep_label))

        return merged_phrases
```

**3. Identifying the Role of the Words/Phrases (Cause/Effect)**

Once the phrases are extracted, CogMApp identifies their roles as either cause or effect, which is essential for understanding causal relationships within the sentence. This involves checking the dependencies and linguistic features of each phrase to assign accurate dependency labels.

![Sentence Structure](../assets/images/sentence-structure.png "Sentence Structure")
![Causative word in text](../assets/images/passive-example.png "Causative word in text")

The tool examines the type and pattern of the entire sentence, particularly looking for passive voice constructions. By analyzing linguistic features, it determines the role of the subject, identifying whether it is a 'nsubj' (noun subject) or 'nsubjpass' (noun subject passive) \[see the examples above]. If any word in the phrase has the dependency tag 'nsubj' or 'nsubjpass', the phrase is labeled as the ROOT (effect). Otherwise, the tag from the last word in the phrase is used as the final dependency label.

Additionally, the presence of causative verbs like 'cause', 'affect', and 'influence' is taken into account. These verbs can alter the roles of the phrases and their causal relationships. For example, in the sentence "New laws and rules for 2024 affect wages and taxes," the verb 'affect' helps in determining the causal relationship between the phrases. 

This careful assignment of dependency labels ensures that the tool accurately identifies meaningful components and their roles in the sentence, facilitating a clear understanding of causal relationships.

``` python
def determine_dep_label(self, deps_in_phrase):
        """
        Determine the most appropriate dependency label for a phrase based on internal dependencies.
        """
        if 'nsubj' in deps_in_phrase:
            return 'ROOTnsubj'
        elif 'nsubjpass' in deps_in_phrase:
            return 'ROOTnsubjpass'
        else:
            # Choose a representative dependency if no clear subject is present
            return deps_in_phrase.pop() if deps_in_phrase else 'unknown'
    
    def extract(self, sentence, causative_verb):
        """
        Extracts and returns noun phrases with their detailed dependency tags from the sentence.
        """
        doc = self.process_text(sentence)
        noun_phrases = self.get_noun_phrases(doc)
        result_dict = {phrase: dep for phrase, dep in noun_phrases}

        # Check for the presence of causative verbs like 'cause',  in the sentence
        found_verbs = [v for v in causative_verb if v.lower() in sentence.lower()]
        if found_verbs:
            # Adjust dependency labels for noun phrases based on the presence of an causative verb.
            for phrase, dep in list(result_dict.items()):  # Work on a copy of items to safely modify the dict
                if dep == 'ROOTnsubj':
                    result_dict[phrase] = 'dobj'
                elif dep == 'dobj':
                    result_dict[phrase] = 'ROOT'
        
        return result_dict
```
**4. Determining the Direction of the Causal Relationship**

The next step is to ascertain the direction of the causal relationship between the identified phrases. The algorithm determines the direction based on the role of the words or phrases, taking into account the dependency labels. The ROOT of the causal relationship will be identified as the point towards which the arrow is pointing. To illustrate, we can posit that "rising prices [ROOT] <- inflation".

``` python
def format_results(results):
    formatted = []
    # Find all roots or central subjects to structure the phrases around them
    root_keys = [key for key, value in results.items() if value == 'ROOTnsubj' or value == 'ROOTnsubjpass']

    for key, value in results.items():
        if key in root_keys:
            continue  # Skip the roots themselves when adding to the formatted list
        for root_key in root_keys:
            if value == 'ROOTnsubjpass':  # If the dependency indicates a passive subject
                formatted.append(f"{key} -> {root_key}")
            else:
                formatted.append(f"{root_key} <- {key}")

    # Remove duplicates and return the formatted results
    formatted = list(set(formatted))
    return formatted
```

**5. Visualization**

Finally, the relationships are visualized using NetworkX and Matplotlib, creating a cognitive map that clearly displays the causal connections between the concepts. The final output is an interactive cognitive map that can be used to visualize and understand the causal relationships within the provided text.

``` python
def wrap_label(label):
    """Helper function to wrap labels after every three words."""
    words = label.split()
    wrapped_label = '\n'.join(' '.join(words[i:i+3]) for i in range(0, len(words), 3))
    return wrapped_label

def visualize_cognitive_map(formatted_results):
    G = nx.DiGraph()  # Directed graph to show direction of relationships

    # Add edges based on formatted results
    for result in formatted_results:
        if '<-' in result:
            # Extract nodes and add edge in the reverse direction
            nodes = result.split(' <- ')
            G.add_edge(nodes[1], nodes[0])
        elif '->' in result:
            # Extract nodes and add edge in the specified direction
            nodes = result.split(' -> ')
            G.add_edge(nodes[0], nodes[1])

    # Position nodes using the spring layout
    pos = nx.spring_layout(G, k=0.50)

    # Setup the plot with a larger size
    plt.figure(figsize=(12, 8))  # Larger figure size for better visibility

    # Prepare custom labels with wrapped text
    labels = {node: wrap_label(node) for node in G.nodes()}

    # Draw the graph with custom labels
    nx.draw(G, pos, labels=labels, node_color='skyblue', edge_color='#FF5733', 
            node_size=5000, font_size=10, font_weight='bold', with_labels=True, arrowstyle='-|>', arrowsize=30)
    
    plt.show()

    return plt
```

## Deploying CogMApp Demo using Gradio

The Gradio interface is created to allow users to test the CogMApp Demo. The user can input text and CogMApp will analyse and convert the text into a cognitive map.

``` python
import gradio as gr
import spacy
import pandas as pd
import networkx as nx
import matplotlib.pyplot as plt

# Initialize spaCy NLP model
nlp = spacy.load("en_core_web_sm")

# Import Lexicon
cues = pd.read_excel('link_cues.xlsx')
list_causalmarkers = cues['causal_markers']

# Assigning the extractor
extractor = NounExtractor(nlp=nlp)

# Example of how to use this function
# list_causalmarkers = ["so", "because", "increase", "contribute", "due to"] # Adding more markers here
causative_verb = ['affect', 'influence', 'increase', 'against'] # Adding more narkers here

# Define the callback function for the GUI
def CogMapAnalysis(text):
    if contains_words_or_phrases(list_causalmarkers, text):
        result = extractor.extract(text, causative_verb)
        formatted_result = format_results(result)
        plot = visualize_cognitive_map(formatted_result)
        return formatted_result, plot
    else:
        formatted_result = "❌ No causal expression was identified."
        plot = None  # Use None instead of empty string for non-existent objects
        return formatted_result, plot

# Create the GUI using the 'gr' library

with gr.Blocks() as demo:
    gr.Markdown('# CogMApp')
    gr.Markdown('### Generate cognitive maps from text with one click!')

    with gr.Row():
        with gr.Column():
            inputs = gr.Textbox(label="Input", lines=2, placeholder="Enter your text here...")
            submit_button = gr.Button("Submit")
        with gr.Column():
            examples = gr.Examples(examples=[
                "Public support for anti-discrimination laws and the movement to support immigrants grew due to the impact of getting widespread education on social justice issues.",
                "The introduction of new anti-discrimination laws has been driven by an increasing awareness of social injustices and grassroots movements.",
                "The weak law enforcement in this country is due to its citizens's ignorance.",
                "CogMApp is a tool that lets you create cognitive maps from text.",
                "The protests across the country are caused by the announcement of the new regulation."
            ], inputs=inputs)

    with gr.Row():
        output = gr.Textbox(label="Result", lines=5, placeholder=" ")

    with gr.Row():
        cogmap_plot = gr.Plot(label="Generated Cognitive Map")

    with gr.Row():
        gr.Markdown("⚠️ Feel free to flag me if you find any errors. 🙂")

    with gr.Column():
        gr.Markdown('Demo made with ❤ by P.K. Ningrum (2024) | Contact: [https://ningrumdaud.github.io/](https://ningrumdaud.github.io/)')

    # Set up the button to execute the function when clicked
    submit_button.click(CogMapAnalysis, inputs=[inputs], outputs=[output, cogmap_plot])

if __name__ == "__main__":
    demo.launch(show_api=False, share=True)

```

The CogMApp demo is available to the public on Hugging Face: [CogMap Demo](https://huggingface.co/spaces/ningrumdaud/CogMap-Demo)

## Future Improvement

In order to further optimize CogMApp, a number of enhancements can be implemented:

1. **Analyzing More Complex Sentences:** Enhance the tool to handle complex sentences with multi-layered causal relationships and intricate text structures through deeper parsing and analysis of nested causal patterns.

2. **Handling Short and Long-Distance Causal Relationships:** Improve the tool to effectively analyze and represent both short and long-distance causal relationships within texts. This includes understanding the context in which causes and effects are mentioned.

3. **Polarity Detection:** Integrate the ability to detect the polarity (positive or negative) of the relationships between causes and effects. This would provide a more nuanced understanding of the causal dynamics in the text.

4. **Incorporating More Complex Linguistic Features:** Enhance the analysis by including additional linguistic features such as modality and negation. These features can provide deeper insights into the nature of the causal relationships.

5. **Combining with Large Language Models:** Leverage the power of large language models like BERT, GPT, etc to improve the accuracy and depth of causal relationship detection. These models can provide more sophisticated understanding and generation capabilities, enhancing the tool's performance.

By integrating these advanced features, the cognitive map analysis tool can become even more powerful and versatile, providing richer insights and more comprehensive visualizations of causal relationships in text data.
