# How to build your own .md file

The process for extracting a writing voice from any body of work.

## 1. Get the raw material

You need a corpus. Tweets, blog posts, newsletters, essays, transcripts. The format doesn't matter. Volume and engagement data do.

For Twitter/X: Settings > Your Account > Download an archive. The file you need is `data/tweets.js`. It's JavaScript, not JSON:

```js
window.YTD.tweets.part0 = [ ... ]
```

For other platforms: export or scrape your content. You need the text and some measure of performance (views, likes, shares, comments).

## 2. Filter to signal

Your raw archive is mostly noise. Strip it down to the writing that performed.

Remove:
- Reposts / retweets
- Replies and conversations
- Link-only posts with no original text
- Anything with no text

Keep:
- Original writing
- Engagement metrics (likes, shares, views)

Sort by engagement, descending. This is your working dataset. Engagement tells you what the audience responds to, not just what you happened to write.

Example pre-processing script:

```ts
import { readFileSync, writeFileSync } from 'fs'

const raw = readFileSync('data/tweets.js', 'utf-8')
const json = raw.replace(/^window\.YTD\.tweets\.part0\s*=\s*/, '')
const tweets = JSON.parse(json)

const filtered = tweets
  .map((t: any) => t.tweet)
  .filter((t: any) => {
    const text = t.full_text
    if (text.startsWith('RT @')) return false
    if (text.startsWith('@')) return false
    const stripped = text.replace(/https?:\/\/\S+/g, '').trim()
    if (!stripped) return false
    return true
  })
  .map((t: any) => ({
    id: t.id,
    text: t.full_text.replace(/https?:\/\/\S+/g, '').trim(),
    date: t.created_at,
    likes: parseInt(t.favorite_count),
    rts: parseInt(t.retweet_count),
  }))
  .sort((a: any, b: any) => b.likes - a.likes)

writeFileSync('index.json', JSON.stringify(filtered))
```

## 3. Measure the shape

Feed the top 300-500 posts to an AI. Run these analyses separately. Each produces a section of the profile.

### Length

```
Calculate:
- Median word count
- % under 10 words, under 20, under 50
- % that are a single sentence
- Average engagement by word count bracket
```

### Perspective

```
Analyze pronoun usage:
- % containing "you/your"
- % containing "I/my"
- % containing "we/our"
- Compare ratios in top 100 vs rest
```

### Punctuation

```
Count average per post:
- Periods, commas, line breaks, question marks, exclamation points, colons
- What % end with a period vs no punctuation?
- Compare engagement for each ending type
```

### Structure

```
Classify by structure:
- Single sentence
- Two-part parallel
- Conditional
- Numbered list
- Explicit contrast
- Other
Give percentages.
```

### Openings and closings

```
Classify opening patterns (declaration, list, conditional, imperative, quote, question).
Classify closing patterns (what part of speech is the final word? period or no period?).
Give percentages. Compare engagement.
```

### Verb mood

```
Classify by dominant verb mood:
- Declarative (statement)
- Imperative (command)
- Conditional (if/when)
- Interrogative (question)
Give percentages. Compare engagement.
```

## 4. Find the rhetorical moves

This is where it gets qualitative. Feed the top 300 posts:

```
Identify recurring rhetorical moves. For each:
- Name it
- Describe the mechanics (what makes it work)
- Give 3-5 examples from the data

Don't just label them. Explain the underlying logic.
```

Every voice has its own set. Some common ones: contrast pairs, reframes, quantification of abstract ideas, compressed frameworks, humor through understatement, uncomfortable truths.

## 5. Extract word-level mechanics

Go deeper than rhetorical moves. These are the sentence-level devices.

```
Look for:
- Sound patterns: alliteration, internal rhyme, matched syllable counts
- Structural devices: chiasmus (A-B flips to B-A), circular loops, negation flips
- Contrast frames: classify every syntactic structure used to carry a comparison
- Word choices: what part of speech do sentences end on? monosyllabic or polysyllabic?
- Case patterns: does capitalization or lack of it serve a purpose?
- Punctuation as device: does any punctuation mark do structural work beyond grammar?
```

Run each as a separate prompt. One prompt per device category.

## 6. Map the negative space

What the voice never says is as defining as what it does say.

```
Read 500 posts and identify categories of content that never or almost
never appear. Check for absence of:
- Personal emotions, apologies, current events, biographical details
- Engagement asks, gratitude performances, vulnerability theater
- Motivational cliches, own success metrics, pop culture references
- Political opinions, religious references, complaints about others

For each, confirm truly absent or count rare exceptions.
```

## 7. Build the constraint list

Two parts:

**Banned words.** Words and phrases that would break the voice if they appeared. Corporate jargon, filler phrases, hedging language, buzzwords, meaningless transitions.

**Anti-patterns.** Structural habits the voice avoids. Long explanations, preamble before the point, summary paragraphs, excessive adjectives, self-congratulation.

## 8. Create rewrite pairs

Pick 10 ideas from the corpus. Write each two ways:

1. How a generic writer would say it
2. How this voice actually said it

These teach compression by example. They show an AI the exact distance between default output and the target voice.

## 9. Select reference examples

Pull 20-30 of the highest-performing posts that best represent the voice. Choose for variety across the rhetorical moves, structural patterns, and themes you've identified. These serve as few-shot examples.

## 10. Assemble the file

One markdown file. Recommended order:

1. Voice description (one paragraph)
2. Shape (length, sentence count)
3. Perspective (pronoun ratios)
4. Punctuation fingerprint
5. Structure templates
6. Engagement x length
7. Rhetorical moves
8. Word-level mechanics
9. Contrast frames
10. Verb mood
11. Opening and closing patterns
12. Negative space (what the voice never says)
13. Signature vocabulary
14. Tone
15. Themes
16. Banned words and anti-patterns
17. Rewrite pairs
18. Reference examples

## 11. Validate with the author

Show every finding to the person whose voice it is. Some patterns are intentional. Some are accidents. The file should only contain the deliberate ones.

This step is not optional. Without it you're documenting habits, not voice.

## Notes

- 1,000 posts is a minimum. 10,000+ gives statistical confidence.
- Engagement data is critical. Without it you're measuring output, not signal.
- Run each analysis separately. One mega-prompt produces shallow results.
- The file is never finished. New patterns surface over time. Add them.
- This process works for any medium: tweets, essays, newsletters, transcripts, books.
