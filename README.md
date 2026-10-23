# Receiptz

**Don't trust the take. Check the receipts.**

Receiptz is a fact-checking agent you can tag on X. Reply to any post with:

```
@checkreceiptz is this true?
```

It checks the claim against primary records, has a panel of AI models judge it independently, and replies with a verdict, its sources, and a receipt code. Anyone can look up the receipt to see exactly which models judged, what they read, and proof that no person changed the result.

Built for **NEAR HACKATHON** (Nov 1–14, 2026). Primary track: **AI Swarms**. Fallbacks: AI Security & Defense, Open Innovation.

> **Status:** proposal and build plan. Sections marked *(planned)* describe what ships during the hackathon window.

---

## Why this exists

- False news spreads faster than the truth. A 2018 MIT study of 126,000 stories on Twitter found false news was 70% more likely to be retweeted and reached 1,500 people about six times faster ([Science, 2018](https://www.science.org/doi/10.1126/science.aap9559)).
- AI already fact-checks at scale. AI-written Community Notes had 88.8% of their decided notes upheld, versus 68.5% for human writers, over seven months of public data ([R Street, 2026](https://www.rstreet.org/research/ai-writers-on-community-notes-an-evaluation-of-seven-months-of-data/)). By May 2026, eight AI contributors wrote 50.3% of visible Community Notes ([Indicator](https://indicator.media/p/8-ai-bots-now-write-50-of-x-s-community-notes)).
- But nobody can see how those AI fact-checkers decide. Grok is one model owned by the platform it fact-checks. AI note writers are run by unidentified operators with unpublished processes.

**Receiptz doesn't promise the truth. It promises a fair process you can check:** which models judged, which sources they read, what instructions they followed, and proof no human touched the verdict.

---

## How it works

1. **Tag.** Someone replies to a post with `@checkreceiptz is this true?`
2. **Dedupe.** If the claim was already checked and the verdict is still fresh, the stored verdict comes back instantly.
3. **Triage.** A small model turns the post into specific, checkable claims. Opinions, satire, gossip and private matters are politely declined (see [Scope](#scope)).
4. **Gather evidence.** Specialist agents pull primary records for the claim's topic, then add news context from across the political spectrum.
5. **Judge.** Three jurors from three different AI labs vote independently. A red team tries to overturn the draft verdict. Auditors check every citation.
6. **Reply and record.** The agent replies with a label, confidence, the top sources by name, and a receipt code such as `R-8F2K`. The full receipt is stored permanently and publicly.
7. **Update.** When new evidence arrives or an appeal succeeds, the verdict is revised and the history stays visible.

**Verdict labels:** True · False · Misleading · Unverified · Contested

Contested is not a failure. On genuinely disputed claims, it's the honest answer.

### Verdict rules

- Three jurors vote first. True, False or Misleading publishes only if all three agree **and** the red team fails to overturn it.
- If jurors split, escalate to five. Four of five publishes; anything less is Contested.
- Not enough primary evidence means Unverified, whatever the jurors lean.
- Auditors block any verdict citing a source that doesn't say what's claimed.
- Anyone can appeal with a refundable deposit. A fresh jury re-reviews with new sources. Overturned: the appellant wins. Upheld: the deposit funds future checks.

### Evidence tiers

| Tier | Role | Examples |
|---|---|---|
| 1. Primary records | Decide factual claims | Voting records, bill text, transcripts, BLS releases, SEC filings, court dockets, on-chain data |
| 2. News across the spectrum | Context; disagreement between outlets is reported | Left, center and right outlets, with the spread published |
| 3. Social posts | Leads and original footage only | Viral posts, original uploads |
| 4. Prediction markets | Odds for forward-looking claims only | Polymarket, Kalshi, shown with volume and liquidity |

The hackathon build covers three domains with clean public APIs: **politics** (Congress.gov), **economy** (BLS) and **crypto** (on-chain data).

---

## Architecture

```
X mention
   │
   ▼
Triage ───────────── opinion / gossip / private → polite decline
   │                 satire → "appears satirical"
   │                 already checked → stored verdict
   ▼
Specialists ──────── primary records → news context → social leads
   │
   ▼
Jury (3 labs) → Red team → Auditors
   │
   ▼
NEAR contract ────── verdict, receipt, history, appeals, funding rules
   │
   ▼
X reply (verdict + sources + receipt code)
```

---

## Built with NEAR

Users never need to touch crypto. Under the hood, NEAR is what makes the process provable.

| NEAR tool | Role |
|---|---|
| **NEAR AI Cloud** (TEE models) | Jurors, red team and auditors run as attested open models; response signatures are checked against attestation evidence |
| **Fusion** | Writes the final plain-language reply after jurors vote separately |
| **Shade Agents** | Swarm agents run attested code registered in one agent contract; the contract holds the funds, so no person holds keys |
| **NEAR smart contract** | Verdict registry, receipts, verdict history, appeals, funding rules |
| **NEAR Intents** | Paid checks and appeal deposits arrive from any chain; revenue converts to stablecoins |

### What a receipt records

- Post ID and a hash of the claim
- Each source URL with a hash of its content at fetch time
- Juror model IDs, lab of origin, and attestation hashes
- Response signatures and the verification path used for each juror
- A hash of the prompt version
- Each juror's individual vote, plus any labeled outside opinion
- Verdict, confidence, timestamp, and the verdict it supersedes

**What "verifiable" means here:** anyone can check the inputs, signatures and attestations. LLM outputs aren't exactly reproducible, so a receipt proves the process, not a byte-for-byte replay.

### The jury

As of Oct 22, 2026, every TEE-attested open model on NEAR AI Cloud comes from a Chinese lab (Alibaba, Zhipu, DeepSeek, Moonshot). The default jury uses three different labs:

| Juror | Lab | Attestation |
|---|---|---|
| GLM 5.3 Flash | Zhipu AI | NEAR fleet (model-attested) |
| Qwen 3.8 27B | Alibaba | NEAR fleet (model-attested) |
| deepseek-v3.2 | DeepSeek | Third-party TEE via Chutes (full model attestation via JS SDK or `nearai-cloud-verifier`) |

One Western model (Claude or Gemini, via NEAR AI's Incognito tier) adds a labeled **outside opinion**. Its vote is shown in the receipt but never counts toward the verdict, because it doesn't run in a TEE. This limitation is disclosed in every receipt.

---

## Scope

Receiptz checks verifiable facts on the news people act on.

| Checks | Declines |
|---|---|
| Public officials' votes, statements, bills, financial disclosures | Private individuals |
| Company filings, economic data, on-chain activity, scams | Private finances with no public record |
| Whether an event happened and what was actually said | Bodies, health, sex and family, even for celebrities |
| Viral statistics without sources | Value judgments (both sides' strongest case instead) |

- **Public data only.** No logins, private messages, leaked documents or data brokers.
- **No aggregation.** Triage blocks piecing together public details about a person.
- **No gossip amplification.** Declines don't repeat the rumor.
- **Same rules for everyone,** every party and every person.

---

## Running it *(planned)*

### Prerequisites

- Node.js 20+ and Rust with `cargo-near`
- NEAR CLI and a testnet account
- NEAR AI Cloud API key
- X API credentials (pay-per-use)
- Congress.gov and BLS API keys

### Setup

```bash
git clone https://github.com/0xtitan6/receiptz
cd receiptz
cp .env.example .env   # add your keys
npm install
npm run contract:deploy   # deploys the receipts contract to testnet
npm run agents:dev        # runs the swarm locally (non-TEE mode)
npm run explorer:dev      # receipt explorer on http://localhost:3000
```

### Try it: do X, see Y

| Do | See |
|---|---|
| `npm run check -- "The unemployment rate hit 10% last month"` | A **False** verdict citing the latest BLS release, plus a receipt code |
| Open `http://localhost:3000/r/<code>` | Juror votes, sources with content hashes, attestation signatures and verification paths |
| `npm run check -- "<senator> voted for <bill>"` | **True** or **False** with the roll-call vote and bill text named |
| `npm run check -- "Is this policy good?"` | A "not a factual claim" reply with the strongest case from each side, no verdict |
| `npm run check -- "Did <celebrity> get surgery?"` | A polite decline that doesn't repeat the claim |
| `npm run appeal -- <code>` | A fresh jury re-reviews; the receipt shows the appeal and its outcome |
| Tag `@checkreceiptz is this true?` on a live post | A reply with verdict, sources and receipt code |

### Repository layout

```
contract/    NEAR smart contract (Rust): receipts, verdicts, appeals, funding rules
agents/      Swarm: triage, specialists, jury, red team, auditors (TypeScript, Shade Agents)
explorer/    Public receipt lookup site
docs/        Proposal, design notes, prompt versions
```

---

## Costs

| Item | Cost |
|---|---|
| One check (three-lab jury, all agents) | ~$0.021 |
| One X reply (no URL) | $0.015 |
| Repeat tag of an already-checked claim | ~$0 (stored verdict) |

Replies carry a receipt code instead of a link: X charges $0.20 for posts with a URL versus $0.015 without. The lookup site is linked from the account bio.

**Funding model:** free for anyone who tags it, paid for at scale through a verification API for AI agents, resolution services for prediction markets, and sponsored checks. The funding contract can only hold stablecoins and pay for inference, APIs and payouts. It cannot trade on prediction markets.

---

## Honest limits

- **Same-region jury.** All attested jurors come from Chinese labs today. Mitigated by three different labs, a labeled outside opinion, and disclosure in every receipt. Expect more Contested verdicts on China-related topics.
- **Shared training data.** Models from different labs still learned from overlapping internet data. The red team and the Contested label catch what diversity can't.
- **Source selection.** The spread of sources across the political spectrum is published in every receipt.
- **Breaking news.** No primary record exists yet, so claims are marked developing and re-checked.
- **Prompt injection.** Tagged posts are parsed into a strict claim schema. Models never decide what gets signed or paid.

---

## Related work

| Project | How Receiptz differs |
|---|---|
| Grok | Multi-model jury, published sources, verifiable process, not owned by the platform it checks |
| X Community Notes AI writers | Public process and operator, with a receipt for every verdict |
| Mira Network | Checks public claims on X against primary records, with a red team and appeals |
| AgentOracle | Public, permanent receipts and primary-record evidence |
| UMA / Kleros | AI-speed first verdict; human appeals as the backstop |
| Vericore (Bittensor Subnet 70) | Public, X-native flow with receipts; a possible future evidence partner |

---

## Roadmap

- **Hackathon:** contract, triage, three domain specialists, three-lab jury with red team and auditors, attested receipts, X replies, receipt explorer
- **Next:** Community Notes AI writer registration, live proactive sweeps, paid verification API through NEAR Intents, prediction market resolution
- **Later:** a differently trained juror (fine-tuned on primary records first), and media verification for images and video (provenance first, with BitMind as a supporting signal)

---

## License

TBD