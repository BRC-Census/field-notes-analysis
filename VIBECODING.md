# How We Vibecoded the Burning Man Field Notes Project

## What “vibecoded” means here

This project was built through an iterative, AI-assisted development process. Instead of designing every component completely in advance, we started with the research goal and the raw field-note material, then used conversational coding, rapid experiments, generated scripts, inspected outputs, and repeated refinement to turn the idea into a working analysis pipeline.

The AI was used as a programming and research-production partner. It helped translate broad questions into executable analyses, write data-processing code, generate prompts, classify narrative responses, produce reports, and identify places where the results needed interpretation or caution.

The project was not produced by asking an AI to write one final script. It evolved through a series of smaller decisions:

1. Define what we wanted to learn from the field notes.
2. Inspect the source images and transcription format.
3. Build a small extraction and parsing workflow.
4. Run it against real data.
5. Inspect errors and irregularities.
6. Add cleaning and normalization steps.
7. Turn research questions into analysis modules.
8. Use structured AI classification for open-ended responses.
9. Cache results and regenerate reports.
10. Read the findings critically and document limitations.

## Starting with a research question

The project began as an ethnographic question rather than a software specification:

> Who are people when the usual social rules change?

Burning Man provided a useful setting because participants temporarily enter a different social environment. The research questions were framed around transformation, identity, survival, ritual, relationships, and inclusion.

Those questions became the organizing structure of the codebase. Each major research question became an analysis module:

- Transformation
- Survival
- Identity and costumes
- Symbolism
- Diversity and inclusion
- Relationships

This was an important part of the vibecoding process: we did not begin by building a generic data platform. We built the minimum structure needed to answer the next meaningful question, then generalized shared pieces when repetition became obvious.

## Turning photographs into data

The original data was not a spreadsheet. It was a collection of photographs of handwritten responses, often containing multiple pages, different orientations, incomplete answers, and difficult handwriting.

The first major challenge was therefore not statistical analysis. It was making the material machine-readable without losing the structure of the original responses.

The extraction workflow was developed around a vision-capable Gemini model. The model was prompted to:

- Detect separate pages and response blocks.
- Correct mentally for rotated images.
- Keep different respondents separate.
- Extract answers in question order.
- Return structured JSON.
- Preserve blanks and illegible answers instead of inventing text.

The output was standardized into strings such as:

```text
Q1: 41 | Q2: Male | Q3: San Francisco | Q4: 8 | Q5: WATER
```

This is a characteristic vibecoding move: use the model where it is good at perception and language, but constrain the output tightly enough that ordinary code can process it afterward.

The image-extraction implementation is in [`census/etl/img_extract.py`](census/etl/img_extract.py) and [`census/etl/img_extract_async.py`](census/etl/img_extract_async.py).

## Converting flexible text into a stable schema

The transcription output was intentionally human-readable, but it was not yet analytical data. The next iterations added conventional data-engineering steps:

### Parsing

[`parse_questions.py`](census/etl/parse_questions.py) detects question labels and separates the raw transcription into columns such as `Q1`, `Q2`, `Q3`, and `Q5`.

### Cleaning

Cleaning scripts merged fragmented answers, handled overflow text, and created more consistent rows and columns.

### Demographic normalization

Participants answered demographic questions in many different ways. For example, tenure could appear as “Virgin,” “first time,” “10+,” or a numeric count.

[`normalize_demographics.py`](census/etl/normalize_demographics.py) converts these responses into reusable fields:

- `Norm_Age`
- `Norm_Gender`
- `Norm_Burn_Count`
- `Norm_Region`
- `Burn_Status`

The important point is that AI-assisted development did not eliminate ordinary data cleaning. The AI helped create and revise the scripts, but the resulting records still had to be inspected, counted, and interpreted.

## Developing the analysis modules incrementally

Each module followed the same general loop:

1. Select the relevant question set.
2. Filter out blank or unusable responses.
3. Define a research-specific classification scheme.
4. Write a prompt describing the classification task.
5. Define a structured response schema.
6. Process the narratives.
7. Aggregate classifications into counts and percentages.
8. Break results down by demographic cohorts.
9. Generate a Markdown report.
10. Review the report for overinterpretation and data-quality problems.

This allowed the project to move quickly from a conceptual question to a concrete, inspectable result.

## Where AI was used in analysis

The analysis uses Gemini in two different ways.

### Vision extraction

Gemini reads handwritten page images and produces structured transcriptions.

### Semantic classification

Gemini also classifies written narratives. For example, a response about costumes is assigned a motivation category, while a response about The Temple is assigned a sentiment score and primary emotion.

The shared helper in [`census/analysis_utils.py`](census/analysis_utils.py) supports:

- Prompt templates
- Gemini API requests
- Asynchronous processing
- Rate limiting
- JSON responses
- Pydantic response schemas
- Error handling
- Local caching

The structured schemas were important because they reduced the problem from “ask the model what it thinks” to “ask the model to fill in a defined analytical record.”

For example, the diversity module asks for fields such as:

- Identity impact
- Key theme
- Code-switching

The relationship module uses separate schemas for new love, committed relationships, and playa-versus-default-world comparisons.

## Prompting as part of the code

Prompts were treated as part of the analytical implementation. Each prompt defines the categories, interpretation boundaries, and expected output.

Examples of prompt-driven tasks include:

- Identifying whether hardship appears in a transformation narrative.
- Assigning hardship intensity as None, Low, Medium, or High.
- Classifying costume motivation as Play, Authenticity, Comfort, Disguise, or Other.
- Assigning sentiment from -1.0 to 1.0.
- Detecting social friction or identity-related discomfort.
- Identifying themes such as connection, community, grief, freedom, or uncertainty.

This is one of the central characteristics of the project’s vibe-coded architecture: the analysis logic is partly expressed in Python and partly expressed in natural-language prompts and schemas.

## Caching and iteration

Early experimentation with LLM analysis can become expensive and slow if every code change reprocesses every response. The project therefore added a cache keyed by the prompt, response text, and schema.

The workflow became:

1. Run a module.
2. Store each model result locally.
3. Modify the report or aggregation logic.
4. Re-run without paying to classify unchanged responses again.

This made it practical to refine the analysis repeatedly. The cache also made the process more reproducible because the intermediate classifications were preserved as local JSON files.

## Combining conventional code with AI interpretation

The project deliberately uses a hybrid approach.

Conventional code handles:

- CSV loading and writing
- Regular-expression parsing
- Demographic normalization
- Numeric calculations
- Keyword matching
- Counting
- Grouping
- Percentages
- Report generation
- File organization

AI handles tasks that require interpretation of messy language:

- Reading handwriting
- Recognizing themes
- Inferring emotional tone
- Identifying forms of social friction
- Summarizing relationship dynamics
- Comparing symbolic meanings

This division keeps the pipeline practical. Deterministic operations remain inspectable, while the model is used for the qualitative layer where rigid rules would be insufficient.

## How the research concepts became code

The analysis modules were organized around memorable conceptual frames:

### “The Pilgrim’s Progress”

Transformation was analyzed as a movement from first-time participation toward long-term Burner identity. The code compares themes and “I” versus “we” language across burn-status groups.

### “The Ordeal as Catalyst”

Survival analysis tests whether physical hardship is a central explanation for transformation. The code combines hardship classification with equipment, survival-day, and community-support analysis.

### “The Mask vs. the Mirror”

Costume analysis turns a philosophical question about identity into a semantic classification task.

### “Sacred vs. Profane”

Symbolism analysis compares the emotional profiles of The Man and The Temple using sentiment and emotion labels.

### “The Other in Utopia”

Diversity analysis compares identity-focused narratives with general experience narratives to distinguish identity-specific friction from overall experience sentiment.

### “Playa Love vs. Default Love”

Relationship analysis uses separate classification schemas for new romance, committed relationships, and comparisons between playa and default-world intimacy.

These names helped preserve the human and interpretive character of the research while still producing executable analysis.

## What the human contributed

Vibecoding does not mean that the AI independently decided what the project meant. Human judgment remained necessary at several points:

- Choosing the research questions.
- Deciding which question sets belonged together.
- Defining meaningful categories.
- Deciding which demographic comparisons were ethically and analytically appropriate.
- Reviewing OCR errors.
- Checking whether model classifications made sense.
- Interpreting percentages in context.
- Recognizing selection bias and sample imbalance.
- Distinguishing exploratory findings from strong conclusions.
- Deciding how to communicate participants’ voices respectfully.

The AI accelerated implementation, but the research framing and critical interpretation remained human responsibilities.

## What we learned from the iterative process

### The raw data determines the architecture

The project could not be designed as if it were a clean survey dataset. Handwriting, multiple response blocks, blank fields, and inconsistent demographics shaped the entire pipeline.

### Structured output matters

Free-form model responses would have been difficult to aggregate. JSON schemas and Pydantic models made the AI output more useful as data.

### Caching is essential

Without caching, every analytical revision would require reprocessing the full narrative dataset.

### Sample sizes must remain visible

Different questions have different response counts. A percentage without its denominator can be misleading, especially for small cohorts.

### Model output is not ground truth

Semantic classifications are interpretations. They should be read alongside representative responses, not treated as objective measurements.

### Research caveats belong in the output

The reports explicitly document selection bias, OCR limitations, demographic simplification, unequal year sizes, and small subgroup sizes. These are part of the result, not merely implementation details.

## The final pipeline

The complete workflow is orchestrated by [`census/run_pipeline.py`](census/run_pipeline.py):

1. Generate basic statistics.
2. Generate descriptive statistics by theme.
3. Run transformation analysis.
4. Run survival analysis.
5. Run identity analysis.
6. Run symbolism analysis.
7. Run diversity analysis.
8. Run relationship analysis.
9. Synthesize the final report.
10. Run cross-theme TF-IDF and clustering analysis.

The outputs are Markdown reports in [`reports`](reports), along with cached model results in [`analysis_cache`](analysis_cache).

## Final perspective

The project was vibecoded by moving back and forth between three layers:

1. **Human meaning:** What are participants trying to tell us?
2. **AI interpretation:** Can a model help organize those narratives at scale?
3. **Software structure:** Can the results be reproduced, inspected, cached, and turned into reports?

The result is not simply an AI-generated report and not simply a conventional data pipeline. It is a collaborative qualitative-analysis system in which research concepts, prompts, schemas, Python scripts, cached classifications, and human review all work together.

That combination made it possible to analyze a large collection of messy handwritten testimony while preserving the project’s original character: immediate, personal, imperfect, and rooted in lived experience.
