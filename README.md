
# Vietnamese Address Parsing — Advanced Algorithms (Group 22)

**Goal:** Extract **Province**, **District**, and **Ward** from free‑text Vietnamese addresses using **algorithmic techniques (no ML)** with a target average latency of **≤ 0.04 s/request**. citeturn1search1

> This README is distilled from the presentation deck *“AA‑Assignment Mr. TA – Group_22”* and captures the problem, approach, and limitations for quick onboarding. citeturn1search1

---

## 1) Problem Statement
- **Input:** Raw address strings that may include abbreviations, typos, or extra components (street, alley, house number, etc.).
- **Output:** The best‑matched administrative triplet **(Ward/Commune, District, Province/City)**.
- **Constraints:**
  - Must **not** use machine learning.
  - Aim for **≤ 0.04 s/request** on average. citeturn1search1

### Address Variants Covered (from slides)
- Fully spelled and in order, optional prefixes.  
- In‑order but with **spelling errors** in name or prefix.  
- Contains **extra components** (street, alley, etc.).  
- **Missing** one or more of W/D/P.  
- **Abbreviations** in names and prefixes (e.g., *TP.HCM, HN, P., Q., TT, TX*). citeturn1search1

---

## 2) Approach Overview
We combine **string normalization**, **n‑gram preprocessing**, and **three specialized tries** per level (Ward, District, Province):

### 2.1 Normalization
- Lowercasing, diacritic handling/normalization, whitespace cleanup. *(Implementation detail for this repo — not explicitly in slides but required for robust matching.)*

### 2.2 Trie Construction (×3 per level)
1) **Correct‑spelling trie**: Prefix + normalized location (e.g., *Huyện*, *Thành Phố*; abbreviations like *H, TP, X, TT, Tpho, Txa, Ttran* are handled). Numeric wards are paired with their prefix (e.g., *P.1*, *P.2*). citeturn1search1
2) **Incorrect‑spelling trie**: From correct entries, generate single‑edit variants using an alphabet `abcdefghijklmnopqrstuvwxyz0123456789` with operations:  
   - **Substitute:** ~35×n variants  
   - **Delete:** n variants  
   - **Add:** (n+1)×36 variants  
   *(Only for tokens with 3–19 chars).* citeturn1search1
3) **Abbreviation trie**: Allows first‑letter and last‑word abbreviations (e.g., **HCM**, **HN**, **HCMinh**, **HNoi**). citeturn1search1

### 2.3 n‑gram Preprocessing
- Longest location name in the database is **4 words**, so we search up to **4‑grams** and match **longest substrings first**. citeturn1search1

### 2.4 Matching Strategy
- Apply **longest‑match‑first** across n‑grams against the three tries.  
- Enforce or infer **W → D → P** ordering where possible and resolve conflicts by score/priority. *(Scoring details can be tuned in code.)* citeturn1search1

---

## 3) Pseudocode (from slides, adapted)
```text
for each input_address:
    tokens = normalize_and_tokenize(input_address)
    candidates = []

    # n-gram search: 4 → 3 → 2 → 1
    for n in [4,3,2,1]:
        for each n_gram in sliding_window(tokens, n):
            for trie in [correct_spelling, incorrect_spelling, abbreviation]:
                if trie.has(n_gram):
                    candidates.append(match_object)

    # Resolve ward, district, province in order (longest & highest-priority wins)
    w = best(candidate where level==WARD)
    d = best(candidate where level==DISTRICT compatible_with w)
    p = best(candidate where level==PROVINCE compatible_with d)

    return assemble_result(w, d, p)
```
*(Structure derived from “Pseudocode / Build Trie / n‑gram / Matching strategy” slides.)* citeturn1search1

---

## 4) Results & Limitations (from slides)
**Known limitations:**
- Misses cases with **>1 character** spelling error in a token.  
- Cannot reliably distinguish **single‑character differences** in similar names (e.g., *Vũ Ninh* vs *Võ Ninh*).  
- Diacritic‑only differences (e.g., *Thạnh Hóa* vs *Thanh Hóa*) are hard to separate without extra heuristics. citeturn1search1

---

## 5) Project Plan (from slides)
- **Week 1–2:** Problem definition, solution design, pick algorithms.  
- **Week 3:** Implement, build tries, n‑gram logic.  
- **Week 4:** Error checking, improvements (accuracy, average runtime), finalize, test list. citeturn1search1

---

## 6) Team (Group 22)
- **Duong Gia An** — Team Lead (17.5%)  
- **Trinh Son Lam** — Idea · Implement · Slide (17.5%)  
- **Nguyen Thi Thu An** — Idea · Implement · Slide (17.5%)  
- **Nguyen Thanh Nhan** — Idea · Implement · Slide (17.5%)  
- **Pham Toan Vuong** — Idea · Implement · Slide (17.5%)  
- **Pham Le Trong Bang** — Idea (12.5%)  
*(Names & contributions per slide.)* citeturn1search1

---

## 7) Getting Started (repo suggestion)
```bash
# Suggested structure
address-parsing/
├── data/                 # location dictionaries (ward/district/province)
├── src/
│   ├── normalize.py      # diacritic & text normalization
│   ├── trie.py           # trie structures & builders
│   ├── matcher.py        # n-gram search & matching
│   └── main.py           # CLI entrypoint
├── tests/                # unit tests
└── README.md
```

### Run (example)
```bash
python src/main.py --input "357/28 Ng T Thuật, P.1, Q.3, TP.HCM"
# → Ward: Phường 1 | District: Quận 3 | Province: TP. Hồ Chí Minh
```

> **Note:** Dictionaries of official Vietnamese administrative units are required to build the tries. *(Add your curated CSV/JSON sources here.)*

---

## 8) Future Enhancements
- Probabilistic ranking (e.g., weighted edit distance by keyboard proximity).
- Context constraints (ward must belong to chosen district, etc.).
- Better diacritic handling & confusion sets; bilingual alias tables.
- Caching hot n‑grams to improve latency.

---

## 9) Acknowledgements
This README summarizes the team’s slide content for quick handover and implementation. citeturn1search1

