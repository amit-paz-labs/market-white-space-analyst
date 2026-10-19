# Hebrew Reading App for Children — Product Spec

## 1. Vision

A friendly, game-like app that teaches children (ages 4–8) to read Hebrew from zero, in three progressive stages:

1. **Alphabet (אלף-בית)** — recognize and sound out each letter.
2. **Nikud (נִקּוּד)** — learn how vowel marks change a letter's sound.
3. **Words** — blend letters + nikud into short words, then longer words and simple phrases.

**Principles**
- Short sessions (5–10 min), lots of audio, immediate positive feedback.
- Mastery-based: a child unlocks the next step only after demonstrating the current one.
- Hebrew is read right-to-left; RTL layout and a "start from the right" cue are taught explicitly.
- No reading required from the parent or child to navigate (icons + voice instructions).

## 2. Target Users

| User | Needs |
|---|---|
| Child (4–8) | Fun, big touch targets, voice guidance, rewards |
| Parent / teacher | Progress view, ability to skip/repeat, printable practice sheets |

Languages for parent UI: Hebrew, English (extensible).

## 3. Learning Path Overview

```
Stage 1: Alphabet  →  Stage 2: Nikud  →  Stage 3: Words
 (22 letters +        (vowels, then      (2-letter → 3-letter →
  5 final forms)       special marks)     longer words → phrases)
```

Each stage is split into **units** of 3–5 items, and each unit follows the same loop:

**Meet → Hear → Trace/Practice → Play → Review**

## 4. Stage 1 — The Alphabet

### 4.1 Content

22 letters, taught in small groups (suggested order, grouping by visual similarity is deliberately *avoided* early to reduce confusion):

| Unit | Letters |
|---|---|
| 1 | א ב ג ד |
| 2 | ה ו ז ח |
| 3 | ט י כ ל |
| 4 | מ נ ס ע |
| 5 | פ צ ק ר |
| 6 | ש ת |
| 7 | Final forms: ך ם ן ף ץ |
| 8 | Look-alike review: ב/כ, ד/ר, ה/ח/ת, ו/ז/ן, ס/ם, ע/צ |

Each letter card includes: letter shape (print + handwriting), **letter name** (e.g. "bet"), **letter sound** (/b/), an anchor word + picture (בַּיִת — house), and a stroke-order animation.

Note: ב/בּ (bet/vet), כ/כּ (kaf/khaf), פ/פּ (pe/fe), and ש/שׁ/שׂ (shin/sin) are introduced here by name only; the dot distinctions (dagesh, shin/sin dot) are taught in Stage 2.

### 4.2 Activities

- **Meet the letter** — animation, name and sound played aloud.
- **Trace it** — finger-trace on a dotted letter with stroke-order guidance and haptic/audio feedback.
- **Sound match** — hear a sound, tap the matching letter (3–4 choices).
- **Picture match** — which letter does this picture start with?
- **Find the letter** — spot the target letter inside a scrambled grid (builds visual discrimination).
- **Alphabet song / order puzzle** — drag letters into א–ת order.
- **Final-form pairs** — match כ ↔ ך, מ ↔ ם, etc.

### 4.3 Mastery Criteria

Unit passes at ≥ 85% accuracy over the last 10 prompts. Missed letters re-enter a spaced-repetition queue.

## 5. Stage 2 — Nikud (Vowel Marks)

Teach nikud on top of a **consonant the child already knows**, usually starting with **א** (silent carrier) so the vowel sound is isolated, then moving to other letters.

### 5.1 Content & Sequence

| Unit | Mark(s) | Name | Sound |
|---|---|---|---|
| 1 | ַ (patach), ָ (kamatz) | patach, kamatz | /a/ |
| 2 | ִ (hiriq) | hiriq | /i/ |
| 3 | ֵ (tzere), ֶ (segol) | tzere, segol | /e/ |
| 4 | ֹ (holam), וֹ (holam male) | holam | /o/ |
| 5 | ֻ (kubutz), וּ (shuruk) | kubutz, shuruk | /u/ |
| 6 | ְ (shva) | shva | short /e/ or silent (introduced simply) |
| 7 | ּ (dagesh): בּ/ב, כּ/כ, פּ/פ | dagesh | changes b/v, k/kh, p/f |
| 8 | שׁ / שׂ | shin dot / sin dot | /sh/ vs /s/ |
| 9 | Vowel + yod/vav combos: ִי (hiriq male), ֵי (tzere male), ַי (-ay), וֹ, וּ | — | long-vowel spellings |

Teaching approach: group marks **by sound** (five vowel sounds: a, e, i, o, u) first; mention that several marks share a sound ("two ways to say /a/"). Avoid grammar rules (kamatz katan, shva na/nach) at this level; shva is taught by example words only.

### 5.2 Activities

- **Vowel on a letter** — a mark "slides" under a letter; tap to hear the syllable (e.g. בָּ → "ba").
- **Syllable table** — a grid of letters × vowels; tap any cell to hear it. Free-play "sound board".
- **Which sound?** — hear a syllable, pick the right one among 3 (בָּ / בִּ / בֻּ).
- **Vowel swap** — a letter stays put; change the vowel and watch the sound change (ba → bi → bu).
- **Build a syllable** — drag a consonant and a vowel together, then hear the blend.
- **Dagesh detective** — spot the dot and pick the sound (בּ vs ב).
- **Reading lanes** — highlight syllables right-to-left with a moving dot; child taps along.

### 5.3 Mastery Criteria

≥ 85% accuracy on mixed syllable recognition (with at least 6 consonants × all five vowel sounds) and ≥ 80% on a timed "syllable blitz" (no time pressure for under-5s; optional).

## 6. Stage 3 — Reading Words

### 6.1 Progression

| Level | Content | Examples |
|---|---|---|
| 1 | Two-syllable-sound words (CV + CV) | אַבָּא, אִמָּא, סַבָּא, מָה |
| 2 | Short words, 2–3 letters with nikud | בַּיִת, גַּן, שָׁם |
| 3 | Short words, closed syllables and simple shva | יֶלֶד, סֵפֶר, שֻׁלְחָן |
| 4 | Longer words (3–4 syllables), chunked | מַחְשֵׁב, חֲנוּכִּיָּה, אוֹטוֹבּוּס |
| 5 | Simple phrases (2–3 words) | אַבָּא בַּבַּיִת |
| 6 | Short sentences + mini story (6–10 words) | הַיֶּלֶד קוֹרֵא סֵפֶר |
| 7 | Reading **without** nikud (optional bridge for ages 7+) | בית, ספר, גן |

Word selection criteria: concrete, picture-able nouns first; only letters/vowels already mastered; high frequency in children's vocabulary.

### 6.2 Techniques

- **Syllable chunking** — long words are split visually and by color (בַּ · יִת); chunks gradually merge.
- **Blend & slide** — child drags a finger across syllables as audio blends them.
- **Read-then-reveal** — child reads aloud; picture reveals meaning; tap to hear the correct pronunciation.
- **Word building** — tap picture, assemble the word from syllable tiles.
- **Missing syllable** — fill in the blank.
- **Rhymes & word families** — בַּיִת / זַיִת / חַיִת.
- **Speech check (optional)** — microphone-based pronunciation check with forgiving thresholds; always optional and on-device when possible.
- **Story mode** — illustrated micro-stories with tap-to-hear and karaoke-style highlighting.

### 6.3 Mastery Criteria

A level passes at ≥ 80% of words read correctly (self-confirm by tapping "I read it!" or via speech check) and ≥ 85% on comprehension picture matches.

## 7. Core Features

### 7.1 Gamification
- Friendly mascot guide (e.g. a fox or owl) who gives voice instructions.
- Stars per activity, stickers per unit, badge per stage, collectible album.
- No loss states, timers, or negative sounds; mistakes get a gentle "try again" and a hint.

### 7.2 Adaptive Learning
- Spaced repetition on letters, syllables, and words.
- Difficulty adjusts to recent accuracy; frequently confused pairs (ד/ר, ח/ה) get extra practice.
- Placement quiz so children who know some letters can skip ahead.

### 7.3 Audio
- Native Hebrew voice recordings (not TTS) for letters, syllables, words, sentences.
- Slow and normal playback speeds.
- Letter **names** vs **sounds** clearly differentiated.
- Pronunciation default: standard modern Israeli Hebrew (kamatz = patach = /a/; tzere = segol = /e/).

### 7.4 Parent / Teacher Area (behind a simple gate)
- Progress dashboard: letters, vowels, and words mastered; trouble spots.
- Session time controls and daily goal.
- Printable worksheets (tracing sheets, syllable tables, word lists).
- Multiple child profiles.

### 7.5 Accessibility
- Full RTL support; large touch targets; high-contrast mode.
- Adjustable font size, choice of print/handwriting-style typefaces.
- Dyslexia-friendly option (extra letter spacing, nikud emphasis).
- Screen-reader labels and voice-over navigation.

## 8. Screens

1. **Welcome / profile picker**
2. **Home map** — path with the three stages, locked/unlocked nodes
3. **Lesson screen** — shared template for Meet/Practice/Game
4. **Tracing canvas**
5. **Syllable sound board**
6. **Word reader** (chunking + highlight)
7. **Story reader**
8. **Rewards / sticker album**
9. **Parent dashboard & settings**

## 9. Technical Notes

- **Platforms:** iOS, Android, and tablet-friendly (cross-platform framework such as Flutter or React Native; web optional).
- **Offline-first:** all audio and content bundled or downloadable per stage.
- **Unicode:** store Hebrew with nikud as proper combining sequences; normalize to a consistent form (NFC may reorder marks — validate rendering and audio lookups on a normalized key).
- **Fonts:** use fonts with correct nikud positioning (e.g. Frank Ruhl Libre, Heebo, Assistant, a handwriting font for tracing).
- **Content model:** JSON/CMS-driven lessons: `Letter`, `Syllable`, `Word`, `Sentence`, `Unit`, `Activity`, each with audio and image references.
- **Speech recognition:** optional; on-device where possible.
- **Analytics & privacy:** compliant with COPPA/GDPR-K; no ads, no third-party trackers, minimal data collection, parental consent for any cloud features.

### Example Content Schema

```json
{
  "word": "בַּיִת",
  "plain": "בית",
  "syllables": ["בַּ", "יִת"],
  "transliteration": "ba-yit",
  "meaning": { "en": "house" },
  "audio": "audio/words/bayit.mp3",
  "image": "img/words/bayit.png",
  "requires": { "letters": ["ב", "י", "ת"], "nikud": ["dagesh", "patach", "hiriq"] },
  "level": 2
}
```

## 10. Success Metrics

- % of children completing Stage 1 within 4 weeks of start.
- Letter-recognition and syllable-reading accuracy over time.
- Median words read independently per week.
- Week-4 retention and average sessions per week.
- Parent-reported reading confidence (short in-app survey).

## 11. Roadmap

| Phase | Scope |
|---|---|
| MVP | Stage 1 (alphabet) + Stage 2 (core vowels) + Stage 3 levels 1–3, one mascot, parent dashboard basics |
| v1.1 | Full nikud set, dagesh/shin-sin, longer words, sticker album, printables |
| v1.2 | Stories, speech-check, adaptive review engine |
| v2.0 | Reading without nikud, writing practice, additional UI languages, classroom mode |

## 12. Open Questions

- Target pronunciation variant (Israeli standard vs. other communities such as Sephardi/Ashkenazi liturgical)?
- Should letter *names* be taught before letter *sounds*, or together? (Spec assumes together.)
- Is handwriting (cursive) in scope, or print only?
- Will recordings be commissioned from a single child-friendly voice actor, or from multiple voices?
- Subscription vs. one-time purchase vs. free for schools?
