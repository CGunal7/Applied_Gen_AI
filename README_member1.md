## Data Source

| File | Description | Records |
|------|-------------|---------|
| `WorldCups.csv` | Tournament-level data (year, country, winner, attendance) | 20 rows |
| `WorldCupMatches.csv` | Match-level data (teams, scores, stage, city, stadium) | 850 rows (after cleaning) |
| `WorldCupPlayers.csv` | Player-level data (name, position, team, shirt number) | 37,784 rows |

**Source:** [Kaggle — FIFA World Cup Dataset](https://www.kaggle.com/datasets/abecklas/fifa-world-cup)  
**Coverage:** FIFA World Cup tournaments from 1930 to 2014  
**Access:** Publicly available, no API key required. Downloaded manually from Kaggle and placed in the `data/` folder.

---

## Data Processing

The raw `WorldCupMatches.csv` contained 4,572 rows, many of which were empty or duplicate separator rows. The following cleaning steps were applied:

1. **Dropped NaN rows** using `dropna()` — reduced from 4,572 to 850 clean match rows
2. **Fixed data types** — converted `Year`, `Home Team Goals`, and `Away Team Goals` to integers
3. **No changes** were made to `WorldCups.csv` or `WorldCupPlayers.csv` as they were already clean

---

## Document Types in Vector Store

| Document Type | Count | Description |
|---------------|-------|-------------|
| Tournament docs | 20 | One document per World Cup tournament (year, winner, attendance, goals scored) |
| Match docs | 850 | One document per match (teams, scores, stage, city, stadium) |
| Team stat docs | 83 | Aggregated stats per team (total games, wins, goals scored, goals conceded) |
| Head-to-head docs | 577 | Summary of all matchups between each pair of teams |
| **Total** | **1,530** | All documents combined and indexed in FAISS |

---

## Validation

The following checks were performed to verify correctness:

1. **Row counts verified** — confirmed expected number of rows after loading and cleaning
2. **Data type check** — confirmed numeric columns (Year, Goals) are integers
3. **Similarity search test** — queried "Who won the 2014 World Cup?" and confirmed Result 1 correctly returned Germany as the winner
4. **Vector count verified** — FAISS index confirmed 1,530 vectors indexed (`vectorstore.index.ntotal`)

---

## Setup & Reproduction

### Prerequisites
- Python 3.8+
- OpenAI API Key

### Installation

```bash
pip install langchain langchain-openai langchain-community faiss-cpu pandas tiktoken python-dotenv gradio
```

### Environment Setup

Create a `.env` file in the project root (use `.env.template` as reference):

```
OPENAI_API_KEY=your_openai_api_key_here
```

### Running the Notebook

1. Clone the repository:
```bash
git clone https://github.com/CGunal7/Applied_Gen_AI.git
```

2. Place your OpenAI API key in the notebook or `.env` file

3. Open `WorldCupGenAI_Track1.ipynb` and run all cells top to bottom

4. The FAISS index will be saved to `faiss_worldcup_index/` after running Cell 13

### Loading the Pre-built FAISS Index

If you want to skip rebuilding the index, load it directly:

```python
from langchain_community.vectorstores import FAISS
from langchain_openai import OpenAIEmbeddings

embeddings = OpenAIEmbeddings()
vectorstore = FAISS.load_local("faiss_worldcup_index", embeddings, allow_dangerous_deserialization=True)
```
