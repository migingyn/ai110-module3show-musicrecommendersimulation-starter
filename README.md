# 🎵 Music Recommender Simulation

## Project Summary

This project is a content-based music recommender built from scratch in Python. It loads a catalog of songs from a CSV file, scores each song against a user's taste profile, and returns a ranked list of the top matches with plain-language explanations.

The recommender uses three features — genre, mood, and energy — to calculate a compatibility score for every song. Genre matching carries the most weight (+2.0 points), followed by mood (+1.0 point), with energy similarity contributing up to +1.5 points based on how close the song's energy level is to the user's target. Songs are then sorted from highest to lowest score and the top results are printed to the terminal.

---

## How The System Works

### Features used per `Song`

| Feature | Type | Description |
|---|---|---|
| `genre` | string | Musical category (pop, lofi, rock, edm, etc.) |
| `mood` | string | Emotional tone (happy, chill, intense, sad, etc.) |
| `energy` | float 0–1 | How loud and active the track feels |
| `tempo_bpm` | integer | Beats per minute (stored but not scored in v1) |
| `valence` | float 0–1 | Musical positivity (stored but not scored in v1) |
| `danceability` | float 0–1 | How suitable the track is for dancing |
| `acousticness` | float 0–1 | How acoustic vs electronic the track sounds |

### What `UserProfile` stores

- `favorite_genre` — the genre the user prefers most
- `favorite_mood` — the mood they want right now
- `target_energy` — a 0–1 value for how intense they want the music
- `likes_acoustic` — boolean bonus flag; adds +0.5 for acoustic songs above 0.6

### Scoring rule (one song)

```
score = 0
if song.genre == user.genre:   score += 2.0   # genre match
if song.mood  == user.mood:    score += 1.0   # mood match
energy_gap = |user.energy - song.energy|
score += 1.5 * (1 - energy_gap)               # energy proximity (max 1.5)
if user.likes_acoustic and song.acousticness > 0.6:
    score += 0.5                               # acoustic bonus
```

### Ranking rule (all songs)

The `recommend_songs()` function loops through every song in the catalog, calls `score_song` to get a numeric score, then uses Python's `sorted()` (returns a new sorted list, leaving the original intact) to rank all songs from highest to lowest score. The top `k` results are returned as `(song, score, explanation)` tuples.

**Data flow:**

```
User Preferences
      │
      ▼
Loop over every song in songs.csv
      │
      ▼
score_song(user_prefs, song) → numeric score + reasons list
      │
      ▼
sorted(all_scored_songs, key=score, reverse=True)
      │
      ▼
Top K Recommendations (title, score, explanation)
```

Real-world recommenders like Spotify and YouTube work on the same loop principle but at a scale of millions of songs and users. They also incorporate collaborative filtering (what similar users liked) and use neural networks to embed songs in high-dimensional feature space. Our version is transparent and rule-based, which makes it easy to inspect and explain — a real strength for an educational context.

---

## Getting Started

### Setup

1. Create a virtual environment (optional but recommended):

   ```bash
   python -m venv .venv
   source .venv/bin/activate      # Mac or Linux
   .venv\Scripts\activate         # Windows
   ```

2. Install dependencies:

   ```bash
   pip install -r requirements.txt
   ```

3. Run the recommender:

   ```bash
   python -m src.main
   ```

### Running Tests

```bash
pytest
```

---

## Experiments You Tried

### Experiment 1 — Default "Happy Pop" Profile

**Profile:** `{"genre": "pop", "mood": "happy", "energy": 0.8}`

**Top results:** Sunrise City (pop/happy/0.82) ranked first with score 4.47, Gym Hero (pop/intense/0.93) second at 2.47. Rooftop Lights (indie pop/happy) placed third.

**Observation:** The genre weight of 2.0 dominates. Even though Gym Hero is intense (wrong mood), it still scores higher than non-pop songs because of the genre match alone.

---

### Experiment 2 — "Chill Lofi" Profile

**Profile:** `{"genre": "lofi", "mood": "chill", "energy": 0.35}`

**Top results:** Library Rain ranked first (lofi + chill + energy 0.35), Midnight Coding second (lofi + chill + energy 0.42). Focus Flow third (lofi but wrong mood: focused).

**Observation:** When both genre and mood match, the energy score provides the tiebreaker. Library Rain's 0.35 energy is an exact match, so it edged out Midnight Coding (0.42).

---

### Experiment 3 — "Intense Rock" Profile

**Profile:** `{"genre": "rock", "mood": "intense", "energy": 0.9}`

**Top results:** Storm Runner ranked first (rock/intense/0.91 — nearly perfect match, score 4.49). Gym Hero second (pop/intense — mood match + energy close). Bass Drop Theory third (edm/intense — mood match + high energy).

**Observation:** The system correctly distinguished intense rock from chill lofi. However, genre dominance means pop and edm tracks with matching moods ranked higher than rock tracks with mismatched moods.

---

### Experiment 4 — Weight Shift (doubling energy, halving genre)

Changed genre weight to 1.0 and energy cap to 3.0. Rankings became much more energy-driven — high-energy EDM and rock tracks floated up even for the "happy pop" profile. This confirmed that the default 2.0 genre weight is appropriate for capturing user intent.

---

### Experiment 5 — "Adversarial" Profile (conflicting preferences)

**Profile:** `{"genre": "ambient", "mood": "intense", "energy": 0.9}`

**Observation:** No song matched both ambient genre and intense mood. Gym Hero (pop/intense/0.93) scored highest at 2.49 — mood and energy dominated over genre. This exposed a gap: the system cannot handle nuanced preferences like "ambient but energetic" because our catalog has no such song.

---

## Limitations and Risks

- **Tiny catalog** — 20 songs means the top result is often the only real match for a niche preference. A real system needs thousands of tracks for meaningful diversity.
- **Genre dominance** — The +2.0 genre weight is the single biggest factor. A user who gets the genre slightly wrong (e.g., "indie pop" vs "pop") will see dramatically different results.
- **No collaborative filtering** — The system has no knowledge of what other users liked. It cannot discover that fans of "chill lofi" also enjoy "ambient."
- **Binary matching** — Genre and mood are treated as either equal or not. There is no concept of genre similarity (rock ≈ alternative rock more than rock ≈ classical).
- **Static weights** — Everyone gets the same formula. A user who cares deeply about energy but not genre is served poorly.

---

## Reflection

See [model_card.md](model_card.md) for the full model card.

Building this recommender made the "loop" structure of recommendation engines very concrete. Every song gets judged independently against the user's preferences and assigned a number. The ranking is just sorting those numbers. What seems "intelligent" in apps like Spotify is really an extremely large and well-tuned version of the same idea — every song in a catalog gets a score, and the highest ones surface.

The place where bias enters is in the weights. Choosing genre weight = 2.0 and mood weight = 1.0 is a design decision that encodes the assumption that genre matters twice as much as mood. In a real product, those weights would be learned from user behavior data, which means they would reflect the preferences of the majority of users — potentially at the expense of users whose tastes don't fit the dominant pattern.
