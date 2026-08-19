# CHAT FILTERING, CONTENT MODERATION FAILURES & ASSET EVASION

> **Classification:** Platform Security & Technical Vulnerability Investigation  
> **Subject:** Two Hat / CommunitySift Integration, The "Hashtag Plague" (#), Unicode & Homoglyph Evasion, Steganographic 2D/3D Asset Exploits, and Spatial Voice Loopholes  
> **Year:** 2026  

---

## 1. Architecture of the Chat Filtering Engine

Roblox processes tens of billions of text strings monthly across in-game chat, spatial bubbles, and private messages. To handle this volume at sub-100ms latencies, Roblox combines rule engines, third-party acquisitions, and machine learning classifiers:

```
                         [ INCOMING CLIENT CHAT STRING ]
                                       │
                                       ▼
                     [ Stage 0: Pre-Processing & Sanitization ]
                     • UTF-8 byte normalization (NFKD/NFC)
                     • Whitespace & control character stripping
                                       │
                                       ▼
                     [ Stage 1: Fast-Path Regex & Trie Dictionaries ]
                     • In-memory Aho-Corasick trie matching
                     • Hardcoded static blacklist / allowlists
                                       │
                                       ▼
                     [ Stage 2: Two Hat / CommunitySift ]
                     • Acquired Sept 2021 (Two Hat Security)
                     • Contextual threat categories (0 to 6 risk scale)
                                       │
                                       ▼
                     [ Stage 3: In-House NLP & Transformer Layer ]
                     • DistilBERT / RoBERTa-based intent models
                                       │
                                       ▼
                     [ Stage 4: Session-Based Scoring Matrix ]
                     • Sliding-window cumulative risk accumulator
                                       │
                                       ▼
                     [ Stage 5: Output Generation ]
                     • Score >= Threshold: Mask with `###`
                     • Score < Threshold: Forward clear text
```

---

## 2. The "Hashtag Plague" (#): Causes of Severe False Positives

1. **The Scunthorpe Problem & Substring Over-Matching:** Basic innocent words like *"basement"*, *"circumstance"*, or *"wristwatch"* trigger full sentence masking due to crude substring blacklists.
2. **Aggressive Under-13 PII Number Censorship:** Any numeric sequence (e.g., *"I have 5 coins"*, *"Wait 2 minutes"*) is treated as a potential phone number or address by COPPA regex filters, rendering normal gameplay chat into `* * * * *` or `###`.
3. **Morphological Blindspots in Non-English Languages:** Agglutinative languages (Tagalog, Indonesian, Turkish, German) attach prefixes and suffixes dynamically. Because tokenizers fail to recognize these compounds, entire legitimate sentences are replaced with `#`.

---

## 3. Predator Bypass Mechanics: Exact Technical Vectors

While ordinary children are heavily censored by the "Hashtag Plague," bad actors systematically bypass filters using well-documented Unicode, encoding, and orthographic evasion vectors:

```
┌──────────────────────────────────────────────────────────────────────────────────┐
│                          PREDATOR BYPASS TAXONOMY                                │
├──────────────────────────────────────────────────────────────────────────────────┤
│ 1. Invisible Unicode Insertion (Zero-Width Sequences)                            │
│ 2. Cyrillic / Greek Homoglyph Substitution (UTF-8 Visual Clones)                 │
│ 3. Combining Diacritical Marks & Grapheme Clustering                              │
│ 4. Lexical Decomposition & Multi-Message Phonetic Splitting                      │
│ 5. Steganographic Social Anchoring (Discord / Telegram Evasions)                  │
└──────────────────────────────────────────────────────────────────────────────────┘
```

* **Invisible Zero-Width Characters:** Inserting zero-width code points (`\u200B` Zero-Width Space, `\u200C` Non-Joiner, `\uFEFF` Byte Order Mark) inside banned keywords (e.g., `d\u200Bi\u200Bs\u200Bc\u200Bo\u200Br\u200Bd`). Regex keyword engines fail to match, while the game client discards zero-width glyphs, displaying a seamless `"discord"` on screen.
* **Cyrillic & Greek Homoglyphs:** Swapping Latin characters with visually identical Cyrillic code points (e.g., Latin `a` `U+0061` vs Cyrillic `а` `U+0430`, Latin `e` vs Cyrillic `е` `U+0435`).
* **Phonetic Off-Ramping:** Obfuscated phrases to redirect children off-platform: `"d!zc0rd"`, `"d.i.s.c."`, `"the purple app"`, `"pin: [digits]"`.

---

## 4. Asset Moderation Bypasses: 2D Clothing, Audio, & 3D Meshes

### A. 2D Clothing Steganography & Multi-Part Stitching
* **Alpha Channel Exploit:** Setting sensitive pixels with pure white RGB (`#FFFFFF`) and low alpha ($A \approx 0.05$). The asset thumbnail appears completely blank during 3-second BPO web portal reviews, but renders fully visible when applied over dark avatar bodies in-game.
* **Multi-Part Composite Stitching:** Slicing explicit or hate imagery across three separate items (Shirt, Pants, T-Shirt Decal). Each asset passes isolated moderation, but composites into prohibited content when equipped together.

### B. Audio Frequency Warping & Runtime Inversion
* Pitch-shifting audio by $\pm 7\text{--}15$ semitones breaks automated acoustic fingerprint hashes (Chromaprint).
* Uploading reversed audio that is played forward at runtime by setting `Sound.PlaybackSpeed = -1` in Luau.

### C. 3D Meshes & "Condo Games" via Runtime CSG
* Uploading harmless primitive shapes (cubes, spheres) that bypass static 3D upload scanning.
* At runtime, Luau scripts execute `Part:SubtractAsync()` and `Part:UnionAsync()` (Constructive Solid Geometry) to programmatically carve out explicit 3D anatomy and strip-club environments in memory.

---

## 5. Spatial Voice Chat Vulnerabilities

* **Zero Proactive Moderation:** Spatial voice operates over low-latency WebRTC Opus audio streams without inline real-time machine learning transcription.
* **Buffer Overwriting:** Client-side 30–60 second local ring buffers are overwritten if predators space out inappropriate comments, preventing victims from capturing evidence before submitting reports.
