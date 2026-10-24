# 🎵 Music Recommender Simulation

## Project Summary

In this project you will build and explain a small music recommender system.

Your goal is to:

- Represent songs and a user "taste profile" as data
- Design a scoring rule that turns that data into recommendations
- Evaluate what your system gets right and wrong
- Reflect on how this mirrors real world AI recommenders

Replace this paragraph with your own summary of what your version does.

---

## How The System Works

Real-world recommenders like Spotify's and YouTube's run a multi-stage pipeline: collaborative filtering and embedding millions of songs to a few hundred,which is a learned ranking model that predicts engagement (plays, skips, saves), with a final stage that adds diversity and context. Their weights are learned from huge amounts of user behavior, so they adapt over time and can surface unexpected picks. My algorithm is a content-based baseline. It scores each song by how close its energy, valence, mood and other attributes are to the user's stated preferences, using hand-picked weights, and then ranks the scores. That makes it fully explainable, but it ignores popularity, context, and other users' behavior. 

Written for: your README, for classmates and graders reading about your project.

---

This recommender is a content-based system that suggests songs by comparing each song's attributes to a user's stated taste profile. Each song is described by its genre, mood, energy, valence, acousticness, danceability and tempo, and the user profile stores a favorite genre, a favorite mood, and a target value for each numeric feature. To score a song, the system measures how close each numeric feature is to the user's target using `1 − |song value − target value|`. This rewards songs that match the user's preference rather than songs that simply have higher or lower values. Genre and mood each earn full credit for a match and none otherwise. These results are combined in a weighted sum, with energy (0.30) and valence (0.25) weighted most heavily because they are the two most independent audio dimensions in the dataset, followed by mood (0.20), acousticness (0.10), and genre, danceability and tempo (0.05 each). The weights add up to 1, so every song gets a score between 0 and 1. To make a recommendation, the system scores every song in the catalog, sorts them from highest to lowest, and returns the top *k* along with a short explanation of which features drove each score. Because the weights are hand-chosen and the system uses only song attributes, it is transparent and works for brand-new songs, but it can't learn from listening behavior, account for popularity or context, or add variety the way large-scale streaming recommenders do.

Some prompts to answer:

- What features does each `Song` use in your system
  - For example: genre, mood, energy, tempo
- What information does your `UserProfile` store
- How does your `Recommender` compute a score for each song
- How do you choose which songs to recommend

You can include a simple diagram or bullet list if helpful.

Each Song in my system has ten fields, but only seven of them go into the score. The id, title and artist are used only to display results. Genre and mood are categorical features, scored as a match or no match. Energy, valence, acousticness, danceability and tempo are numeric features, scored by how close they are to the user's target. Tempo is first rescaled to a 0–1 range with (bpm − 60) / 92. Energy and valence matter most because they are the two mostly independent audio axes in the data, and together they carry 55% of the score.

The UserProfile currently stores a favorite genre, a favorite mood, a target energy and a boolean for whether the user likes acoustic music. That isn't enough for the scoring rule, which needs a target for every scored feature. I would expand it to add target valence, target danceability and target tempo, and replace the acoustic boolean with a target acousticness value. The profile holds stated preferences only, so it doesn't learn from skips or replays.

The Recommender scores each song by comparing it to the profile. For every numeric feature it computes closeness as 1 − |song value − target value|, so a song scores best when it sits near the user's target rather than when its value is high or low. For genre and mood it awards a 1 for a match and a 0 otherwise. The final score is the weighted sum of these results: energy 0.30, valence 0.25, mood 0.20, acousticness 0.10, and genre, danceability and tempo 0.05 each. The weights add up to 1, so every score falls between 0 and 1. The scoring function also returns short reasons, such as "energy within 0.02 of target", which feed the explanation shown with each recommendation.

To choose which songs to recommend, the system runs the scoring rule on every song in the catalog, sorts the songs from highest to lowest score, and returns the top k (five by default) along with their scores and explanations. Extra rules, such as excluding songs the user has already heard or penalizing repeated artists, would also go into this ranking step.

### Additions Beyond the Original Recipe

The recipe above was my original design. After writing it, I chose to extend the system with four more song features that the recipe did not include: `speechiness`, `instrumentalness`, `popularity` and `duration_sec`. I added them to `data/songs.csv`, to the `Song` class, and to the scoring in `src/recommender.py`, and both taste profiles in `src/main.py` set a target for each one. These additions were my own decision and go beyond the starter recipe, so the weights and descriptions in the paragraphs above are out of date for the current code.

- `speechiness` (0–1) captures how much of a track is spoken word, which helps separate hip hop from other genres.
- `instrumentalness` (0–1) captures how little vocal content a track has, which separates lofi, ambient and classical from vocal music.
- `popularity` (0–100) lets a profile prefer mainstream or niche songs.
- `duration_sec` lets a profile prefer shorter or longer tracks.

Adding them meant rebalancing the weights. The current values, which still sum to 1, are genre 0.20, mood 0.20, energy 0.15, valence 0.08, speechiness 0.08, acousticness 0.07, instrumentalness 0.07, danceability 0.04, tempo 0.04, popularity 0.04 and duration 0.03. Because tempo, popularity and duration are not on a 0–1 scale, their difference from the target is divided by a fixed range (100 BPM, 100 points and 300 seconds) before being turned into a closeness score.

---

## Documented Biases (as of Step 2)
The system has several potential biases that come from its dataset and its design. The 10-song dataset is imbalanced: lofi and pop have several songs, while most genres and moods appear only once, so users who like common genres get more matches than users who like rare ones. The dataset also has no classical, hip-hop, country, metal or non-English music, no songs that are both low in energy and low in valence, and a narrow valence range (0.48–0.84), so some tastes have no good match and the system will still return a confident-looking result. Two artists appear twice, and without a diversity rule one artist could dominate the top results. With so few songs, rankings below the top few are mostly noise. The design adds its own biases. Moods and genres are a fixed vocabulary chosen by the dataset's creator and are scored as exact matches, so near-matches like "relaxed" and "chill" get no credit. Energy and valence carry 55% of the score, which pulls every user's results toward one emotional region and under-serves listeners whose taste depends on lyrics, instruments or era. These features also reflect Western pop conventions and may describe other musical traditions poorly. Scoring by closeness to a single target point means a user who likes both very calm and very intense music is matched poorly to both. The weights are hand-picked and unvalidated, and the profile captures only what users say they like, not what they actually listen to. The main mitigations are to add more songs from underrepresented genres and moods, give partial credit to related moods and genres, penalize repeated artists, and test a profile for every genre and mood to confirm whether users with rare tastes receive consistently lower top scores.

## Getting Started

### Setup

1. Create a virtual environment (optional but recommended):

   ```bash
   python -m venv .venv
   source .venv/bin/activate      # Mac or Linux
   .venv\Scripts\activate         # Windows

2. Install dependencies

```bash
pip install -r requirements.txt
```

3. Run the app:

```bash
python -m src.main
```

### Running Tests

Run the starter tests with:

```bash
pytest
```

You can add more tests in `tests/test_recommender.py`.

---

## Sample Recommendation Output

Paste a sample of your recommender's output here as a text block so a reader can see what it produces:

```
# e.g.:
# User profile: genre=indie, mood=chill, energy=low
# Recommendations:
#   1. ...
#   2. ...
#   3. ...
```

**Screenshot or video** *(optional)*: <!-- Insert a screenshot or demo video link here -->

---

## Experiments You Tried

Use this section to document the experiments you ran. For example:

- What happened when you changed the weight on genre from 2.0 to 0.5
- What happened when you added tempo or valence to the score
- How did your system behave for different types of users

---

## Limitations and Risks

Summarize some limitations of your recommender.

Examples:

- It only works on a tiny catalog
- It does not understand lyrics or language
- It might over favor one genre or mood

You will go deeper on this in your model card.

---

## Reflection

Read and complete `model_card.md`:

[**Model Card**](model_card.md)

Write 1 to 2 paragraphs here about what you learned:

- about how recommenders turn data into predictions
- about where bias or unfairness could show up in systems like this



