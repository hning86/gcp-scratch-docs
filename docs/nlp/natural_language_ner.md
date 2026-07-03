# Named Entity Recognition (NER) with Cloud Natural Language API

This guide details the implementation of `ner.py`, which demonstrates how to extract structured information from unstructured text using the **Google Cloud Natural Language API (`v1`)**. Named Entity Recognition (NER) identifies known entities (such as people, places, organizations, and events), labels them, scores their relevance (salience), and links them to external reference metadata.

---

## 🔍 Code Walkthrough & Reference

The full implementation of the script can be found in `ner.py`:

```python
from google.cloud import language_v1

def test(x, y):
    # Standard math helper
    x = 8
    a = x - 2
    y = x + a
    return y

def sample_analyze_entities(text_content):
    """
    Analyzing Entities in a String

    Args:
      text_content The text content to analyze
    """
    print(f"Analyzing entities in the following text: '{text_content}'")

    # 1. Initialize the Language Service Client
    client = language_v1.LanguageServiceClient()

    # 2. Define the Document type (PLAIN_TEXT or HTML)
    type_ = language_v1.Document.Type.PLAIN_TEXT

    # 3. Formulate the document payload and specify language limits (optional)
    language = "en"
    document = {"content": text_content, "type_": type_, "language": language}

    # 4. Define the string encoding format (UTF-8, UTF-16, or UTF-32)
    encoding_type = language_v1.EncodingType.UTF8

    # 5. Execute the Entity Analysis Request
    response = client.analyze_entities(
        request={"document": document, "encoding_type": encoding_type}
    )

    # 6. Parse and output the response entities
    for entity in response.entities:
        print(f"\nEntity: {entity.name}")
        print(f"  Type: {language_v1.Entity.Type(entity.type_).name}")
        print(f"  Salience score: {entity.salience:.3f}")

        # Metadata (e.g. Wikipedia links, mid codes)
        if entity.metadata:
            print("  Metadata:")
            for metadata_name, metadata_value in entity.metadata.items():
                print(f"    {metadata_name}: {metadata_value}")

        # Mentions of this entity throughout the text
        if entity.mentions:
            print("  Mentions:")
            for mention in entity.mentions:
                print(f"    - Text: {mention.text.content}")
                print(f"      Type: {language_v1.EntityMention.Type(mention.type_).name}")

    # Display the final evaluated language
    print(f"\nLanguage of the text: {response.language}")


if __name__ == "__main__":
    # Test text containing nested events, people, dates, and locations
    test_text = (
        "The Apollo 11 mission was the first spaceflight that landed humans on the Moon. "
        "Commander Neil Armstrong and lunar module pilot Buzz Aldrin landed the Apollo Lunar Module Eagle "
        "on July 20, 1969, at 20:17 UTC. Armstrong became the first person to step onto the lunar surface "
        "six hours and 39 minutes later on July 21; Aldrin joined him 19 minutes later. "
        "They spent about two and a quarter hours together outside the spacecraft, and they collected "
        "47.5 pounds of lunar material to bring back to Earth."
    )
    sample_analyze_entities(test_text)
```

---

## 🛠️ Deep Dive: Architectural Elements

### 1. `LanguageServiceClient` Initialization
The `LanguageServiceClient` acts as the gateway to the Google Cloud Natural Language API. It handles authentication, TCP connection pooling, and serialization automatically, utilizing your active **Application Default Credentials (ADC)**.

### 2. Document Representation
A dictionary representing a `Document` object is supplied to the client. It requires two main components:
- **`content`**: The raw string to analyze.
- **`type_`**: The structure of the source text. Supported types:
  - `PLAIN_TEXT` (used here): For unformatted text files, transcripts, or notes.
  - `HTML`: For web scraped material, preserving tags to help determine block semantics.
- **`language`** (Optional): BCP-47 language tag (e.g., `"en"`, `"es"`, `"ja"`). If omitted, the API automatically detects the language and returns it in `response.language`.

### 3. Encoding Type
Specifying an encoding type (`EncodingType`) is crucial because the API returns byte-offsets indicating exactly where entities are mentioned in the source string.
```python
encoding_type = language_v1.EncodingType.UTF8
```
The offset offsets will match Python's string slicing indices if you configure the encoding to match Python's native text representation (`UTF8`).

### 4. Decoding Entity Metadata & Types
Each matched entity contains deep categorization parameters:
- **`entity.name`**: The normalized or representative name of the entity.
- **`entity.type_`**: An integer mapped back to the descriptive enum. Common types include:
  - `PERSON` (e.g., *Neil Armstrong*)
  - `LOCATION` (e.g., *Moon*, *Earth*)
  - `ORGANIZATION`
  - `EVENT` (e.g., *Apollo 11*)
  - `WORK_OF_ART`
  - `CONSUMER_GOOD`
- **`entity.salience`**: A float ranging from `0.0` to `1.0` representing the "importance" or prominence of this entity within the context of the overall text document.
- **`entity.metadata`**: A key-value map containing supplemental information. For well-known public entities, this frequently contains:
  - `wikipedia_url`: The URL linking to the entity's English Wikipedia page.
  - `mid`: A unique identifier (MID) pointing to Google's Knowledge Graph entry.

### 5. Tracking Entity Mentions
An entity may be referenced multiple times across a text block using different phrasing (e.g., *"Neil Armstrong"*, *"Armstrong"*, or *"him"*).
The API returns a list of **`mentions`** for each entity, allowing developers to track every instance of that entity.
- **`mention.text.content`**: The raw substring in the original document.
- **`mention.type_`**: The grammatical type of the mention. Common mention types include:
  - `PROPER` (e.g., *"Neil Armstrong"*)
  - `COMMON` (e.g., *"commander"*, *"pilot"*)
