# Biased Coin Desk

**A Kelly criterion sandbox with variable risk:reward.** Give yourself a coin you *know* is biased in your favour, then find out how many ways there still are to go broke.

🔗 **[Live demo](https://YOUR-USERNAME.github.io/biased-coin-desk/)** — replace with your URL after deploying

No build step, no dependencies, no tracking. One HTML file.

---

## Why this exists

In 2016, Victor Haghani and Richard Dewey ran an experiment that should be uncomfortable reading for anyone who thinks having an edge is the hard part.

They recruited 61 people — college-age economics and finance students plus young professionals at finance firms — gave each of them $25, and put them in front of a simulated coin for 30 minutes. The instructions stated in bold that the coin came up heads 60% of the time. Payouts were real, capped at $250 (ten times the stake), with the cap disclosed only if a player got close.

The optimal play is not subtle. Bet a constant 10–20% of your bankroll on heads, every time, and don't think about it again.

What actually happened:

| Outcome | Share of players |
|---|---|
| Reached the $250 cap | 21% |
| Went bust, received nothing | 28% |
| Ended with less than they started | ~33% |

Roughly 95% of them should have hit the cap on a simple constant-percentage strategy. The average player walked away with $91, against a benchmark of about $240 available to anyone applying a modified Kelly rule. Eighteen of the 61 bet their entire bankroll on a single flip, and two-thirds bet on tails at some point — against a coin they had been told, in bold, favoured heads.

Ed Thorp, who gave feedback on the research, called it a great experiment that ought to be part of the basic education of anyone interested in finance or gambling.

This project is that experiment turned into something you can hold and shake.

---

## What's different here

The original game was fixed at even money: risk $1, win $1. That collapses the whole problem to a single variable and hides the more interesting question — **what happens to optimal sizing when the payout isn't 1:1?**

So this version adds a risk:reward slider, and everything downstream recomputes from it. That single change makes several counterintuitive things visible:

- A **30% win rate is a good bet** at 1:4, and Kelly will happily size it at 12.5%
- A **78% win rate is a losing bet** at 1:0.25, and no amount of clever sizing rescues it
- Doubling your payout **less than doubles** your optimal size, because variance rises with it

Everything else — the calendar axis, the growth-per-day figure, the ruin test — exists to make the consequences of that slider legible.

---

## The maths

Four formulas do all the work. `p` is your probability of winning, `q = 1 − p`, and `b` is your reward per unit risked (so 1:3 means `b = 3`).

**Edge per dollar risked**

```
E = p·b − q
```

If this is negative, stop. There is no bet size that fixes it, and the app will say so.

**Kelly-optimal fraction** — the generalized form, which reduces to the familiar `2p − 1` when `b = 1`

```
f* = (p·b − q) / b
```

**Log growth rate per trade** at any fraction `f`

```
g(f) = p·ln(1 + f·b) + q·ln(1 − f)
```

This is the function plotted in the Growth Rate panel, and it is the honest picture of bet sizing. It rises to a peak at `f*`, then falls — crossing back below zero at roughly `2f*`. Past that point you are compounding losses on a bet that is still, on paper, in your favour. Ruin stops being bad luck and becomes the arithmetically expected outcome.

**Growth per day**, once trades/day is set

```
g_day = e^(g·n) − 1     where n = trades per day
```

Per-trade growth is a deceptively small number — 2.01% at the study's settings. At 10 trades a day that's +22.3%. At 25 it's +65.4%. This figure is meant as a gut-check, not a target: frequency amplifies variance exactly as hard as it amplifies edge.

### Sanity checks

| p | b | Kelly f* | Edge per $1 |
|---|---|---|---|
| 60% | 1 | 20.0% | +$0.200 |
| 60% | 2 | 40.0% | +$0.800 |
| 60% | 0.5 | — no bet | −$0.100 |
| 30% | 4 | 12.5% | +$0.500 |
| 78% | 0.4 | 23.0% | +$0.092 |
| 40% | 1 | — no bet | −$0.200 |

---

## Using it

### Controls

| Control | What it does |
|---|---|
| **Starting balance** | Sets your opening capital. Resets the run and rescales the ruin line, which sits at 1% of whatever you start with. |
| **Heads rigging** | The coin's true probability of heads, 0–100%. |
| **Risk : reward** | Reward per 1 unit risked, 0.1–10. The ticket shows the resulting stake and payout. |
| **Heads / Tails** | Which side you're on. Picking the minority side flips your win probability to `1 − p`. |
| **Sizing** | How the stake is calculated each trade — see below. |
| **Trades/day + Start date** | Maps trade number to a calendar date. Drives the chart x-axis, days-elapsed, and growth-per-day. |
| **Speed** | Auto-flip rate, 1–60 trades/second. Purely cosmetic; it doesn't affect outcomes. |

### Sizing modes

| Mode | Behaviour |
|---|---|
| Fixed fraction | A constant % of current bankroll. The mode the study's subjects should have used. |
| Full Kelly | Recomputed to `f*` each trade. Maximum growth, brutal drawdowns. |
| Half / Quarter Kelly | The same, scaled down. Most of the growth, a fraction of the pain — what practitioners actually run. |
| Fixed dollar | A flat stake regardless of bankroll. No compounding, but no ruin from fractional decay either. |
| Martingale | Doubles the stake after every loss, resets on a win. Included so you can watch it fail. |

### Presets

- **Haghani–Dewey 2016** — 60% heads, 1:1, $25 start, 10 trades/day. The original game.
- **Long shot** — 30% heads, 1:4, $1,000 start, 3 trades/day. Losing most of the time and still profitable.
- **Grind** — 78% heads, 1:0.4, $1,000 start, 25 trades/day. High hit-rate, thin payout, very little room to size up.

### Keyboard

`Space` flip · `Enter` start/stop auto-flip · `H`/`T` switch side · `Esc` dismiss the end-of-run slip

---

## Reading the charts

**Bankroll over time** — your equity curve on a real date axis. The dashed red line is the ruin threshold; the grey one is your starting balance. Toggle **Log scale** as soon as growth turns exponential, because on a linear axis a 400× run makes the first 90% of the chart look like a flat line. A gold marker appears if and when you cross $1B.

**Volatility** — rolling 20-trade standard deviation of returns. Watch this move when you drag the sizing slider; it's the cost side of the ledger that the bankroll chart doesn't show you.

**Growth rate by bet size** — the centrepiece. The full `g(f)` curve for your current settings, with three markers: the Kelly peak in green, the point where growth turns negative in red, and a white line showing where *you* are sized. Drag the risk:reward slider and watch the whole curve reshape in real time. If the curve never rises above zero, you have a negative-edge bet and the app will refuse to name a size.

---

## The thousand-lifetime test

A single run tells you nothing. One person got rich on martingale; a thousand didn't.

The bottom panel replays your exact settings across N independent lifetimes and reports the bust rate, median and mean finish, and the 5th/95th percentiles, with a log-scaled histogram. The red bar left of the break is everyone who fell through the ruin line.

This is where the study's lesson lands. At 60% heads, betting on the correct side, over 2,000 runs of 5,000 trades from a $25 start:

| Bet size | Reached $1B | Went broke | Median trades to $1B |
|---|---|---|---|
| Kelly (20%) | 99.4% | 0.7% | 819 |
| Overbet (45%) | 3.2% | 96.8% | 499 |

Same coin. Same edge. Same correct side. The overbettor gets there *faster* on the rare occasion it works — 499 trades against 819 — which is precisely the trap. The fast route is the visible one. The 96.8% behind it is not.

---

## End-of-run slips

Two outcomes interrupt the run with a dated summary:

- **Crossing $1B** — a milestone slip with your multiple on start, win rate, deepest drawdown and worst losing streak. You can dismiss it and keep trading.
- **Hitting the ruin line** — a terminal slip. No continue button.

Both end with a diagnosis of what actually happened rather than a generic message: whether the edge was negative, whether you were sized above 2× Kelly and ruin was the expected outcome, whether martingale hid the maths until a streak outgrew the account, or whether you were sized sensibly and simply got unlucky — which does happen, and is the entire reason half-Kelly exists.

---

## Publishing to GitHub Pages

The app is a single static file with no build step, so this takes about two minutes.

### Via the web interface

1. Create a new **public** repository on GitHub — `biased-coin-desk` works well.
2. Upload `index.html` and `README.md` to the root. **The file must be named `index.html`** — GitHub Pages looks for exactly that name and will show a 404 for anything else.
3. Go to **Settings → Pages**.
4. Under **Source**, choose **Deploy from a branch**.
5. Set the branch to `main` and the folder to `/ (root)`, then **Save**.
6. Wait 1–2 minutes. Your site appears at `https://YOUR-USERNAME.github.io/biased-coin-desk/`.

### Via the command line

```bash
mkdir biased-coin-desk && cd biased-coin-desk
# copy index.html and README.md into this folder
git init
git add .
git commit -m "Biased coin desk: Kelly simulator with risk:reward"
git branch -M main
git remote add origin https://github.com/YOUR-USERNAME/biased-coin-desk.git
git push -u origin main
```

Then complete steps 3–6 above. Every subsequent `git push` redeploys automatically.

### Notes

- **Custom domain**: add a file named `CNAME` containing your domain, then point a `CNAME` DNS record at `YOUR-USERNAME.github.io`. Enable **Enforce HTTPS** in Settings → Pages once the certificate provisions.
- **Root domain**: name the repository `YOUR-USERNAME.github.io` and it serves at `https://YOUR-USERNAME.github.io/` with no subpath.
- **Jekyll**: not needed. A `.nojekyll` file in the root is harmless and skips the unnecessary build step.
- **Social preview**: `index.html` already has Open Graph tags. Add a `preview.png` (1200×630) and one line in the `<head>` to make shared links render a card:
  ```html
  <meta property="og:image" content="https://YOUR-USERNAME.github.io/biased-coin-desk/preview.png">
  ```
- **Update the demo link** at the top of this README once the URL is live.

---

## Running locally

Open `index.html` in any browser. That's the whole procedure — there is no server requirement, no `npm install`, no bundler.

If you'd prefer a local server anyway:

```bash
python3 -m http.server 8000
# then visit http://localhost:8000
```

---

## Project structure

```
biased-coin-desk/
├── index.html    # the entire application — markup, styles, logic
└── README.md     # this file
```

Everything lives in one file on purpose. No dependencies means nothing to audit, nothing to update, and no supply chain. Charts are hand-drawn on `<canvas>` with device-pixel-ratio scaling; there is no charting library.

Roughly: state object → `flip()` mutates it → `render()` redraws everything. If you want to extend it, `render()` is the single place where UI meets state.

Accessibility and quality floor: responsive to mobile, visible keyboard focus, `prefers-reduced-motion` respected, no browser storage APIs used.

---

## Ideas for extending it

- **Payout cap** — reintroduce the study's $250 ceiling, which changes optimal play substantially (Kelly ignores ceilings; the true optimum near a cap is lower)
- **Uncertain edge** — let the true `p` differ from the stated one. This is the realistic case, and it is the strongest argument for fractional Kelly.
- **Correlated trades** — drop the independence assumption and watch position sizing get much harder
- **Multi-player mode** — race 61 simulated players with different heuristics, reproducing the study's distribution directly
- **Drawdown constraint** — cap sizing to keep expected max drawdown under a threshold, which is closer to how risk desks actually operate

---

## What this is not

This is a teaching tool for a toy problem, and the gap between it and real markets is the entire difficulty of investing.

The simulation assumes you **know** the true probability, that odds are fixed and offered to you, that trades are independent, that capital is infinitely divisible, and that there are no fees, spreads, slippage or taxes. Markets grant none of these. You never know `p`; you estimate it, and your estimate is usually wrong in ways correlated with everyone else's. Kelly sizing on an overestimated edge is how confident people go broke — which is why practitioners run half-Kelly or less.

Nothing here is financial advice, and the fact that a strategy compounds beautifully in this sandbox is not evidence that it will do anything at all in a market.

---

## References

- Haghani, V. & Dewey, R. (2016). *Rational Decision-Making Under Uncertainty: Observed Betting Patterns on a Biased Coin.* — [arXiv:1701.01427](https://arxiv.org/abs/1701.01427) · [SSRN](https://papers.ssrn.com/sol3/papers.cfm?abstract_id=2856963)
- [Elm Wealth's write-up of the experiment](https://elmwealth.com/lessons-from-betting-on-a-biased-coin-cool-heads-and-cautionary-tales/) — the authors' own summary
- Kelly, J. L. (1956). *A New Interpretation of Information Rate.* Bell System Technical Journal — the original paper
- Thorp, E. O. (2006). *The Kelly Criterion in Blackjack, Sports Betting, and the Stock Market* — the standard practical treatment
- [Gwern's exact solutions to the Kelly coin-flipping game](https://gwern.net/coin-flip) — solves the capped-payout version properly with dynamic programming

---

## License

MIT. Do what you like with it.

The Haghani & Dewey study is the authors' work and is cited above; this project is an independent implementation of the game they describe, not affiliated with them or Elm Wealth.
