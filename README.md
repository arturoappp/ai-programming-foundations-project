# AI Programming Foundations Project — Bible Structured Data Workflow

## Project Description

A reproducible data workflow (ingestion → cleaning → exploratory analysis → visualization → summary) built with Python, NumPy, Pandas, Matplotlib and Seaborn in a single Jupyter notebook, `data_workflow.ipynb`.
It joins three structured tables of the Bible — the named people, the verses in which each person appears, and the 66 books — into one tidy person–verse–book table, and uses it to answer who is mentioned most, how mentions are distributed across people, and how much of each book and testament names anyone at all.
The workflow is the data foundation for the content pipeline of a Bible trivia app, where knowing which people appear in which chapter is what makes a question well anchored and its wrong answers plausible.

**Dataset:** BibleData — *Structured Datasets from the Holy Bible* by Brady Stephenson (Version 1.0, 2026, CC BY 4.0).
Source: https://github.com/BradyStephenson/bible-data · DOI: https://doi.org/10.5281/zenodo.19539956
Files used (copies in `data/`): `BibleData-Person.csv` (3,009 × 9), `BibleData-PersonVerse.csv` (44,267 × 8), `BibleData-Book.csv` (66 × 18) and `BibleData-Event.csv` (572 × 18, loaded only). The dataset license is included as `data/LICENSE-BibleData-CC-BY-4.0.txt`.

## How to Run the Project

Requirements: **Python 3.12 or newer** and Git. The pinned versions in `requirements.txt` (for example `numpy==2.5.3` and `contourpy==1.4.0`) do not install on Python 3.11 or older.

```bash
git clone https://github.com/arturoappp/ai-programming-foundations-project.git
cd ai-programming-foundations-project
python -m venv .venv
source .venv/bin/activate        # Windows: .venv\Scripts\activate
pip install -r requirements.txt
jupyter notebook data_workflow.ipynb
```

Then run all cells (Kernel → Restart & Run All). The notebook runs top to bottom without errors, reads the CSV files from `data/`, and writes the three figures to `figures/`.

To regenerate the dependency file from the environment used for this project:

```bash
pip freeze > requirements.txt
```

## Repository Structure

```
data_workflow.ipynb   # the complete workflow (Setup, Ingestion, Cleaning, EDA, Visualizations, Summary)
module_summary.pdf    # written report with in-text citations and references
requirements.txt      # exact package versions (pip freeze)
data/                 # BibleData CSV files, license and citation
figures/              # fig1_top_people.png, fig2_book_coverage.png, fig3_mentions_distribution.png
```

## Reflections

### Bias awareness: where could poor data cleaning introduce bias?

The most dangerous shortcut in this dataset is deduplicating people by name. 1,702 of the 3,009 people share their name with someone else (there are six women called Mary and 23 men called Zechariah), so merging rows by `person_name` would silently fuse different people, inflate their counts and make a trivia question accept a wrong answer as correct. A second risk is how missing values are handled: 46.6 % of people have no recorded tribe, and dropping those rows instead of labeling them "unknown" would remove most New Testament characters, whose tribe is rarely stated, and skew every tribe-level statistic toward Old Testament genealogies. Finally, the 14,733 rows without a `person_id` are placeholders for verses that name nobody; treating them as mentions would add 14,733 fake observations, while treating them as errors would hide the fact that half of Proverbs or Song of Solomon names no one. Even with careful cleaning the data itself carries an imbalance that a downstream system would inherit: only 172 of the 2,993 people mentioned are women (5.7 %), and they hold 3 % of all person–verse pairs, so any question generator that samples characters by frequency will almost never ask about women unless it deliberately stratifies.

### Future integration

**How would this workflow change for a machine learning project?**
The cleaning functions would become the first stage of a feature pipeline, and the EDA outputs would become features: verses per person, number of people per chapter, share of a book's verses that name someone, testament and genre. A machine learning project would also need a target (for example, the literary genre of a passage or the difficulty of a question), a train/validation/test split made by book or chapter rather than at random to avoid leakage between neighbouring verses, and evaluation metrics chosen for the heavy class imbalance shown in Figure 3, where a few characters dominate.

**What would need to change to prepare data for a neural network?**
A neural network would need numeric tensors instead of tidy tables: categorical fields such as `person_id`, `book_code` and `testament` would be integer-encoded or embedded, counts would be scaled, and the verse references would be joined to actual verse text (for example the public-domain King James Version) and tokenized into fixed-length sequences. Data loading would move from a single in-memory `DataFrame` to batched loaders, and the long-tailed distribution of mentions would call for weighting or resampling so that the 1,547 single-verse people are not ignored during training.

**What parts of this workflow could an agent automate?**
An agent with a tool that runs `people_in_chapter(book, chapter)` could draft trivia questions anchored to the people who are actually present in a chapter, pick distractors from the other people of the same chapter, and reject a candidate answer that does not appear in the table. It could also re-run the ingestion and cleaning functions when a new release of the dataset is published, compare the summary statistics with the previous run, and flag changes such as new homonyms or new placeholder rows before anything downstream consumes the data. The parts that should stay with a human are the interpretation of the figures and the decisions about how ambiguous cases (Jacob versus the nation Israel, name changes such as Abram and Abraham) are resolved.

> Detailed explanations and academic citations are in `module_summary.pdf`.
