# [Tech+](https://www.techplusuw.com/) S25 Mentorship Matching
## How the Matching Script Works

This notebook takes two input files (1 listing mentees, 1 listing mentors) and produces a CSV of optimal mentor-mentee pairings under each mentor’s capacity limits.

1. **Feature extraction**  
   - **Multiple-choice questions** (program, skills, timezone, meeting frequency, etc.) are converted into one-hot vectors.  
   - **Free-text answers** are combined and converted into TF-IDF vectors.  
   - Both sets of features are stacked into a single sparse matrix for mentees (`X`) and for mentors (`Y`).

2. **Capacity expansion**  
   - Each mentor can take on 1 or more mentees (as specified in their “capacity” column).  
   - We “expand” each mentor into that many _slots_ so that matching sees every available spot.

3. **Global optimization**  
   - We compute cosine similarity between every mentee and every mentor-slot, resulting in a single similarity matrix.  
   - We then build a “cost” matrix (`1 – similarity`) and feed it into the Hungarian algorithm.  
   - This finds the assignment of mentees to slots that **maximizes total compatibility** across _all_ matches, while ensuring no mentor is over-assigned.

4. **Resulting CSV**  
   - The script writes `s25_mentee_mentor_matches.csv`, which lists for each matched pair:  
     - The mentee’s email  
     - The mentor’s email  
     - A numeric compatibility score (0.0–1.0)  
     - A one-sentence, human-readable justification generated via the OpenAI API  

Because the Hungarian solver considers every possible pairing at once, it guarantees the best overall outcome: you may see some individual mentee scores dip (versus a purely greedy, one-by-one approach), but the _total_ compatibility sum is as large as possible under the exact capacity constraints. That way, even if there aren’t enough slots for every mentee, you know you’re getting the strongest pool of matches overall.

---

**Tip:** If you ever need to tweak what “compatibility” means (eg. by weighting certain questions more heavily), you can adjust the feature stacking step before the similarity calculation and rerun the notebook.
