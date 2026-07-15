# Burning Man Field Notes Project

## Project overview

This project is a mixed-method ethnographic analysis of handwritten Burning Man field notes collected during the 2024 and 2025 events.

Participants wrote responses in physical journals placed around Black Rock City. The questions explored transformation, identity, survival, relationships, symbolism, inclusion, emotions, sustainability, and the meaning of Burning Man outside the event.

The central question is:

> Who are people when the usual social rules change?

The project treats Burning Man as a temporary social environment—a place where participants experiment with radical self-expression, communal living, self-reliance, gifting, ritual, and alternative forms of identity.

The main project framing is documented in [`census/docs/BRIEFING_BOOK.md`](census/docs/BRIEFING_BOOK.md).

## Data collected

The source material consists primarily of photographs of handwritten journal pages. Each journal used a printed question set.

The project documentation reports:

- 2024: 13 question sets, labeled A–M
- 2025: 6 question sets, labeled A–F
- Documented usable response count: approximately 1,797 total responses
- 2024: approximately 1,658 responses
- 2025: approximately 139 responses

The normalized CSV files contain more rows than those headline counts because they include blank, illegible, and “no entries found” records. The actual analysis uses question-specific nonblank responses, so sample sizes vary by module and question.

Each response begins with four demographic questions:

1. Age
2. Gender
3. Location
4. Number of Burning Man experiences

The remaining questions depend on the question set. Examples include:

- Transformation: “Did Burning Man change you? When? How?”
- Symbolism: “What does the Man mean to you?” and “What does the Temple mean to you?”
- Costumes: “Why do you wear costumes?”
- Survival: equipment, survival days, and reliance on community
- Relationships: love, committed relationships, and playa versus default-world intimacy
- Diversity: whether identity or appearance affects interactions

The question instruments are stored in [`census/data/question-set-2024.md`](census/data/question-set-2024.md) and [`census/data/question-set-2025.md`](census/data/question-set-2025.md).

## How the data was created

### 1. Photographing handwritten responses

The original material was photographed as HEIC, HEIF, JPG, JPEG, or PNG images.

The extraction scripts support:

- iPhone HEIC images
- Rotated or sideways pages
- Multiple journal pages in one photograph
- Multiple respondents on one page
- Handwritten responses
- Blank or illegible answers

The main extraction workflow is in [`census/etl/img_extract.py`](census/etl/img_extract.py) and [`census/etl/img_extract_async.py`](census/etl/img_extract_async.py).

### 2. OCR and transcription

Google Gemini Vision was used to read the handwritten pages. The model was asked to return structured strings such as:

```text
Q1: 41 | Q2: Male | Q3: San Francisco | Q4: 8 | Q5: WATER
```

The extraction process uses structured JSON output and Pydantic validation. The asynchronous version also processes images concurrently, limits simultaneous requests, converts HEIC images into RGB, corrects orientation, resizes images, and preserves separate response blocks instead of merging respondents together.

Because the source material is handwritten, OCR quality is imperfect. The repository includes automated cleaning and manually maintained correction data in [`census/data/ocr_corrections.json`](census/data/ocr_corrections.json).

### 3. Parsing questions

The raw transcription initially stores all answers in one text field. [`census/etl/parse_questions.py`](census/etl/parse_questions.py) separates that field into individual columns such as `Q1`, `Q2`, `Q3`, `Q4`, and `Q5`.

The parser recognizes question labels such as `Q1:` and `Q1.` and handles text that continues after a question separator.

### 4. Cleaning columns

The cleaning scripts merge fragmented answers, repair malformed columns, and create a consistent table structure.

The main outputs are:

- Raw transcription CSV
- Parsed CSV
- Cleaned CSV
- OCR-cleaned CSV
- Normalized CSV

These files are located in [`census/data`](census/data).

### 5. Normalizing demographics

Demographic responses were free text, so they had to be standardized.

[`census/etl/normalize_demographics.py`](census/etl/normalize_demographics.py) creates normalized fields:

- `Norm_Age`
- `Norm_Gender`
- `Norm_Burn_Count`
- `Norm_Region`
- `Burn_Status`

The burn-status categories are:

- Virgin: 1 burn
- Sophomore: 2–3 burns
- Veteran: 4–9 burns
- Elder: 10 or more burns
- Unknown: missing or unparseable tenure

Locations are grouped into Local/West, US Other, International, or Unknown. Gender is mapped into broad categories such as `M`, `F`, `NB`, `O`, and `U`.

## Analytical approach

The project combines several analytical methods.

### Descriptive text analysis

The descriptive-statistics module calculates response counts, completion rates, common terms, and breakdowns by age, burn status, gender, and region.

Output: [`reports/descriptive_stats_by_theme.md`](reports/descriptive_stats_by_theme.md)

### LLM-based semantic classification

Several modules send responses to Gemini for structured interpretation. The model is given a question-specific prompt and returns fields defined by a Pydantic schema.

Responses can be classified into themes, emotions, sentiment scores, identity impact, code-switching behavior, relationship type, relationship tone, hardship intensity, and pronoun counts.

The shared implementation is in [`census/analysis_utils.py`](census/analysis_utils.py). It provides data loading, Gemini API calls, rate limiting, error handling, structured response parsing, disk-based caching, and Markdown report generation.

The cache prevents the same response from being repeatedly sent to the model, reducing API cost and making later runs faster.

### Keyword and rule-based analysis

Some analyses use deterministic rules and keyword patterns, especially survival equipment categories, community reliance, hardship indicators, numerical survival-day responses, and basic word frequency.

### Cohort comparisons

Most modules compare responses across burn tenure, age, gender, region, and year. The analyses are generally comparative rather than predictive: they examine how themes differ between groups rather than predicting individual behavior.

## Major analysis modules

### Module 1: Transformation

Implemented in [`census/modules/analyze_transformation.py`](census/modules/analyze_transformation.py).

Research question:

> How does the story of transformation change from first-time participants to long-term Burners?

The module analyzes transformation narratives by burn status and age. It extracts semantic themes and compares first-person language with collective language by counting references such as “I,” “me,” “my,” “we,” “us,” and “our.”

The findings suggest that Virgins emphasize anticipation, acceptance, openness, and discovery, while Veterans and Elders emphasize community, connection, creativity, and belonging. The analysis does not show a dramatic transition from individual to collective language, although Elders show a somewhat lower self-to-collective ratio.

Output: [`reports/module_1_transformation.md`](reports/module_1_transformation.md)

### Module 2: Survival

Implemented in [`census/modules/analyze_survival.py`](census/modules/analyze_survival.py).

Research question:

> Does physical hardship produce emotional or spiritual transformation?

This module compares survival narratives, transformation narratives, hardship mentions, equipment, survival days, and community support.

The results weaken the “ordeal hypothesis.” Hardship appears in only a small share of transformation narratives, while participants more often describe transformation through relationships, community, creativity, freedom, and perspective. The median reported survival capacity was approximately 14 days.

Output: [`reports/module_2_survival.md`](reports/module_2_survival.md)

### Module 3: Identity and costumes

Implemented in [`census/modules/analyze_identity.py`](census/modules/analyze_identity.py).

Research question:

> Are costumes used primarily as masks or as expressions of the self?

Costume motivations are classified into categories such as Play, Authenticity, Comfort, Disguise, and Other. The findings suggest that costumes function more as play and self-expression than concealment:

- Play: approximately 46.5%
- Authenticity: approximately 22.8%
- Comfort: approximately 14.9%
- Other: approximately 15.8%
- Disguise: effectively 0% in the reported classification

Output: [`reports/module_3_identity.md`](reports/module_3_identity.md)

### Module 4: Symbolism

Implemented in [`census/modules/analyze_symbolism.py`](census/modules/analyze_symbolism.py).

Research question:

> How do participants emotionally interpret The Man and The Temple?

The Man is associated primarily with community, celebration, indifference, freedom, and release. The Temple is associated primarily with community, awe, grief, peace, reflection, and release.

The Temple has a stronger association with grief than The Man. The analysis also compares emotional responses by gender and age, finding somewhat higher explicit grief references among men in this dataset.

Output: [`reports/module_4_symbolism.md`](reports/module_4_symbolism.md)

### Module 5: Diversity and inclusion

Implemented in [`census/modules/analyze_diversity.py`](census/modules/analyze_diversity.py).

Research question:

> How does the lived experience of identity compare with Burning Man’s ideal of radical inclusion?

The module analyzes identity-focused narratives for positive, negative, mixed, or neutral impact; racism; exoticism; othering; social friction; and code-switching.

The findings indicate that identity-specific narratives frequently describe negative or mixed experiences. Common themes include exoticism, racism, and othering. General social-friction rates are relatively similar between marginalized and majority comparison groups, while the 30–39 age group has the highest reported friction rate in the general-experience comparison.

The report explicitly warns that the identity-focused sample is self-selected and cannot be interpreted as the experience of all marginalized participants.

Output: [`reports/module_5_diversity.md`](reports/module_5_diversity.md)

### Module 6: Relationships

Implemented in [`census/modules/analyze_relationships.py`](census/modules/analyze_relationships.py).

Research question:

> How are love, intimacy, and relationship expectations different on the playa?

The module separately analyzes meeting someone new, existing committed relationships, and playa-versus-default-world relationships.

It classifies responses by relationship type, tone, intensity, authenticity, openness, communication, independence, intimacy, growth, uncertainty, and ephemerality.

The report suggests that playa relationships are often described as more open, intense, authentic, temporary, uncertain, and community-oriented. The 2025 sample is much smaller, so the two years should not be treated as equally representative.

Output: [`reports/module_6_relationships.md`](reports/module_6_relationships.md)

## Cross-theme analysis

[`census/analyze_cross_theme.py`](census/analyze_cross_theme.py) uses TF-IDF vectorization, K-means clustering, automatically generated theme labels, cohort-level theme prevalence, and cross-question-set correlations.

Unlike the LLM-based modules, this component identifies recurring language patterns across question sets without linking individuals across surveys. It therefore compares cohorts at the group level rather than tracking the same person across every theme.

Outputs are in [`reports/cross_theme`](reports/cross_theme).

## Pipeline structure

The complete workflow is orchestrated by [`census/run_pipeline.py`](census/run_pipeline.py). The intended execution order is:

1. Generate basic demographic statistics
2. Generate descriptive statistics by theme
3. Analyze transformation
4. Analyze survival
5. Analyze identity
6. Analyze symbolism
7. Analyze diversity
8. Analyze relationships
9. Synthesize the final report
10. Run cross-theme cohort analysis

The main final output is [`reports/FINAL_ANALYSIS_REPORT.md`](reports/FINAL_ANALYSIS_REPORT.md).

## Main conclusions

Taken together, the project suggests that Burning Man transformation is primarily social, creative, and emotional rather than the direct result of physical hardship.

The analysis indicates that:

- New participants often describe anticipation, openness, and discovery.
- More experienced participants emphasize community, creativity, and connection.
- Costumes are more often used for play and self-expression than concealment.
- The Temple carries more grief and reflective meaning than The Man.
- Identity-related exclusion exists despite the ideal of radical inclusion.
- General experiences are not uniformly worse for marginalized participants, but identity-specific friction is clearly present.
- Relationships on the playa are often perceived as unusually open, intense, and temporary.
- Individual narratives are highly diverse, making Burning Man difficult to reduce to one common experience.

## Limitations

Important limitations include:

- Participants self-selected whether to respond and which question set to answer.
- Journal placement may have concentrated responses in particular camps or communities.
- The 2025 sample is much smaller and should not be combined with 2024 as an equivalent sample.
- Handwritten OCR introduces misspellings, illegible entries, blanks, truncation, and malformed question boundaries.
- Broad demographic categories simplify participants’ original self-descriptions.
- LLM-generated themes and sentiment labels are useful organizational tools but are not perfectly objective measurements.
- Small subgroup sizes can make percentages unstable.
- The project generally cannot track the same participant across multiple question sets or years.

## Current project status

The core data pipeline and six primary analytical modules are implemented. Reports have been generated for demographics, descriptive statistics, transformation, survival, identity, symbolism, diversity, relationships, cross-theme analysis, and final synthesis.

The project is best understood as a research prototype and reproducible analysis framework. It has a working end-to-end pipeline, but it would benefit from additional validation, stronger data-quality checks, more consistent sample-count documentation, and a final editorial pass on the synthesized report. In particular, the current final report contains the relationship findings but its executive-summary section still says “Conclusion not found” for Module 6.
