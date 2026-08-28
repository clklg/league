# League of Legends Ranked Session Analysis

A small portfolio project built around the **Riot Games API**.

The main goal of this project is to demonstrate a practical API workflow: authenticating with Riot's developer API, sampling ranked players, retrieving match histories, handling rate limits, caching API responses locally, transforming nested JSON into analysis-ready tables, and then performing a short exploratory analysis on the resulting data.

The analysis focuses on ranked League of Legends play sessions and asks three simple questions:

- How long are ranked play sessions at different skill levels?
- Does performance visibly change as a session progresses?
- Are players more likely to continue playing after a win or after a loss?

The project is intentionally lightweight. The API collection pipeline is the main focus; the analysis is included to show how the collected data can be used.

## Project structure

```text
league-ranked-sessions/
│
├── 01_sample_players_collect_ranked_histories.ipynb
├── 02_session_detection_and_analysis.ipynb
│
├── data/
│   └── processed/
│       ├── players.csv
│       ├── ranked_match_history.csv
│       ├── ranked_match_history_with_sessions.csv
│       └── ranked_sessions.csv
│
├── .env
├── .gitignore
├── README.md
└── requirements.txt
```

The local API response cache used during data collection is intentionally not included in the repository.

## Pipeline

### 1. Player sampling and match collection

`01_sample_players_collect_ranked_histories.ipynb`

The first notebook constructs the dataset using Riot's League of Legends APIs.

It:

1. Samples players from four ranked groups:
   - Challenger
   - Diamond
   - Gold
   - Bronze
2. Uses League-V4 to obtain ranked player pools.
3. Samples 10 players per rank group.
4. Uses Match-V5 to retrieve each player's recent Ranked Solo/Duo matches.
5. Handles Riot API rate limits using conservative request pacing and retry logic.
6. Stores match responses in a local cache while collecting data.
7. Extracts participant-level statistics from Riot's nested match JSON.
8. Saves analysis-ready CSV files to `data/processed/`.

The main outputs are:

- `players.csv` — sampled player metadata.
- `ranked_match_history.csv` — one row per sampled player-match observation.

The notebook uses EUW as the platform region and Europe as the Match-V5 regional route.

### 2. Session detection and exploratory analysis

`02_session_detection_and_analysis.ipynb`

The second notebook works entirely from the processed CSV files and makes no API calls.

Ranked sessions are inferred chronologically for each player. A new session begins whenever more than **60 minutes** pass between the end of one ranked match and the start of the next ranked match.

The notebook then explores:

- average ranked session length by rank;
- win rate as sessions progress;
- whether players continue playing after wins versus losses.

It also saves:

- `ranked_match_history_with_sessions.csv`
- `ranked_sessions.csv`

## Main findings

The analysis is exploratory rather than causal, but a few descriptive patterns emerged.

Higher-ranked players in the sample tended to play longer ranked sessions. Challenger players averaged roughly 3.24 games per session and Diamond players about 2.99, compared with approximately 1.86–1.88 games for Gold and Bronze players.

There was no clear downward pattern in win rate as sessions progressed. Later session positions also contained substantially fewer observations, making those estimates less stable.

The original idea that players might be more likely to continue after a loss — a possible "can't end on a loss" effect — was not supported overall. Across the sample, continuation was slightly more common after wins than after losses.

These results should be interpreted as descriptive patterns from a small sample rather than population-level conclusions.

## Reproducing the project

The processed datasets are included so that `02_session_detection_and_analysis.ipynb` can be run without contacting Riot's API.

To reproduce the data collection in Notebook 01, you will need your **own Riot Games development API key**.

### 1. Clone the repository

```bash
git clone <repository-url>
cd league-ranked-sessions
```

### 2. Create a Python environment

For example:

```bash
python -m venv .venv
```

Activate it on macOS/Linux:

```bash
source .venv/bin/activate
```

Or on Windows:

```bash
.venv\Scripts\activate
```

Install the dependencies:

```bash
pip install -r requirements.txt
```

### 3. Get a temporary Riot API key

Go to the Riot Games Developer Portal:

https://developer.riotgames.com/

Development keys are temporary and deactivate every 24 hours, so a fresh key may be required whenever Notebook 01 is rerun.

### 4. Create a `.env` file

Create a file named `.env` in the project root:

```text
RIOT_API_KEY=RGAPI-your-key-here
```

Notebook 01 loads this value with `python-dotenv`.

The included `.gitignore` excludes `.env` so that API credentials are not accidentally committed.

### 5. Run the notebooks

Run the notebooks in order:

```text
01_sample_players_collect_ranked_histories.ipynb
02_session_detection_and_analysis.ipynb
```

Notebook 01 may take some time because Riot enforces API request limits. The notebook deliberately spaces requests and respects `Retry-After` responses when rate-limited.

The results in Notebook 02 are specific to the collected sample and may differ when the data collection and analysis are reproduced with a new sample of players.

## Riot API endpoints used

The collection notebook uses two main API families:

- **League-V4** for ranked player pools.
- **Match-V5** for ranked match IDs and match details.

The sample uses Ranked Solo/Duo queue ID `420`.

For EUW:

- platform routing uses `euw1.api.riotgames.com`;
- Match-V5 uses the Europe regional route, `europe.api.riotgames.com`.

## Limitations

This is a small portfolio project rather than a rigorous behavioral study.

Important limitations include:

- only 10 players were sampled per rank group;
- match-history depth differs across players and ranks;
- a player's sampled rank is a snapshot and may not match their rank during older matches;
- Riot does not expose an explicit play-session identifier, so sessions are inferred from inactivity gaps;
- only ranked games are observed, so other game modes played between ranked games are invisible;
- finite match histories can begin partway through a real session;
- long-session observations become sparse quickly;
- the analysis is descriptive and does not establish causal effects.

## Technologies

- Python
- Jupyter Notebook
- pandas
- NumPy
- Matplotlib
- Requests
- python-dotenv
- Riot Games League of Legends API

## API key security

API credentials are stored locally in `.env` and excluded from version control.

Riot explicitly recommends keeping API keys out of source code. If you fork or reproduce this repository, generate and use your own key rather than attempting to reuse one from another developer.
