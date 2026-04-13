# Reflection: Music Recommender Simulation

## Profile Comparisons

### Happy Pop vs Chill Lofi

The "Happy Pop" profile (`genre: pop, mood: happy, energy: 0.8`) consistently surfaces Sunrise City and Gym Hero at the top. The "Chill Lofi" profile (`genre: lofi, mood: chill, energy: 0.35`) surfaces Library Rain and Midnight Coding instead. The outputs make intuitive sense, these two profiles are essentially opposites on the energy axis and in genre. What changed: every feature. Genre flipped, mood flipped, energy dropped from 0.8 to 0.35. The system rewarded each profile's exact preferences and produced completely non-overlapping top-5 lists. This demonstrates that the scoring formula is sensitive to all three features, not just one.

### Chill Lofi vs Intense Rock

The "Chill Lofi" profile returns quiet, low-energy lofi tracks. The "Intense Rock" profile (`genre: rock, mood: intense, energy: 0.9`) returns Storm Runner at the top (the only rock song in the catalog), followed by high-energy tracks from other genres. What changed: mood shifted from calm to aggressive, genre shifted from lofi to rock, energy jumped from 0.35 to 0.9. The energy shift had a cascading effect, even songs outside the rock genre scored well simply because their high energy was close to the user's target. This shows that energy proximity can partially compensate for a genre or mood mismatch when the catalog is small.

### Intense Rock vs Adversarial (Ambient + Intense)

The "Intense Rock" profile gets Storm Runner as its clear winner. The adversarial profile (`genre: ambient, mood: intense, energy: 0.9`) gets Gym Hero (pop/intense/high-energy), a pop track, as its top result because no song in the catalog is both ambient and intense. What changed: genre. Switching from "rock" to "ambient" eliminated the only strong genre-match candidate (Storm Runner) and forced the system to fall back on mood and energy alone. The output is technically correct but musically jarring, a user asking for intense ambient soundscapes would not expect a pop workout track. This reveals the biggest limitation: the system cannot handle genre preferences for which the catalog has no matching songs.

### EDM Energetic vs Classical Peaceful

The "EDM energetic" profile (`genre: edm, mood: energetic, energy: 0.95`) returns Circuit Breaker and Bass Drop Theory, both are new songs added during Phase 2, which confirms the catalog expansion was useful. The "Classical Peaceful" profile (`genre: classical, mood: peaceful, energy: 0.2`) returns Cathedral Echo as its top result. What changed: every feature, and the results are on completely opposite ends of the musical spectrum. This pair demonstrates that the system can serve niche, underrepresented genres when at least one matching song exists in the catalog.

---

## What I Learned

**The loop is the core idea.** Before building this, I thought recommendation systems must involve some kind of pattern-recognition magic. In practice, they are a loop with a scoring function. Every song gets a number, numbers get sorted. The "intelligence" is entirely in how well the scoring formula captures what people actually want.

**Weights encode values.** Deciding that genre is worth 2.0 and mood is worth 1.0 is not a neutral technical choice, it says "where you listen matters more than how you feel." That assumption might be wrong for many users, and in a real product it would be learned from data rather than set by hand.

**AI tools helped with structure, not judgment.** Claude helped me design the scoring formula, generate diverse song entries, and draft documentation quickly. But every time I needed to decide whether a result "felt right," that required my own musical intuition and knowledge. The AI could not tell me whether an ambient + intense recommendation was bad, I had to recognize that from experience. Human judgment remains essential at the evaluation step.
**Small datasets expose flaws clearly.** Because the catalog only has 20 songs, every failure mode is immediately visible. In a real system with millions of tracks, the same flaws would exist but would be much harder to spot.
