# My Leadership Operating Model

How I personally work as VP, Architecture, day to day. It flows from what my role is, to what I keep in view, to the rhythm I follow, to how I communicate and make the case for investment, with templates for my monthly and quarterly reviews. The team's operating model is a separate document.

## My role and where my time goes

I set platform direction, keep platform health in view, shape product's priorities, hold the senior relationships, and develop the team. My architects own their platforms; I own what crosses them.

| My time goes on | Share of week (target) | What it looks like |
| --- | --- | --- |
| Direction and platform health | About 25% | Reading scorecards and pain logs, target-state reviews, cross-platform decisions, investment cases |
| Relationships | About 15% | Engineering head sync, joint session with ops and product, Enterprise Architecture, Technology Risk |
| People | About 10% | 1:1s, coaching, growth and succession |
| Team routines | About 20% | Weekly team review, monthly and quarterly reviews |
| Focus time | About 15% | Two 90-minute blocks for thinking and writing |
| Everything else | No more than 15% | Ad hoc escalations, email, admin |

**What I stop doing:** leading individual solution designs, attending team-level design reviews, rewriting architects' documents, building status slides by hand, and being the escalation point for every design question.

**The question I ask before taking anything on:** "Which part of this is mine?" Product owns what and why; engineering owns how it is built and run; my architects own their platforms.

## What I keep in view: platform health

I don't track every detail; my architects keep a scorecard for each platform. I read across all five platforms for patterns, trends and risks that need a senior conversation or a cross-platform decision.

| Dimension | What I watch across platforms | It needs me when | I leave it to the team when |
| --- | --- | --- | --- |
| Business capabilities | Capability gaps, manual workarounds, outcomes ops is measured on, epics blocked by the platform | A gap affects ops outcomes or the same gap shows up on more than one platform | It is a single-platform backlog item with a plan |
| Technology | End-of-support components, drift from the enterprise blueprint, technical debt trend | A modernization needs funding, or a platform falls behind the blueprint with no plan | Upgrades are planned and funded |
| Integration | Point-to-point and batch interfaces, non-standard messaging, interface failures | A pattern needs to change across platforms (for example moving to the enterprise event platform) | A single interface fix |
| Data | Golden sources, duplicate stores, quality breaks, ownership disputes | Two platforms claim the same data, or breaks affect ops outcomes | Model or contract changes within one platform |
| Security | Critical vulnerabilities, exceptions past expiry, audit findings | An exception is past expiry without a plan, or a finding needs risk acceptance | Remediation is on track |
| Infrastructure and resilience | Availability, capacity headroom at peak, recovery tests, incidents with a design cause | A recurring incident pattern, or headroom below what next quarter's volume needs | A one-off incident with a fix |
| Run and change cost | Run cost trend, support load, how long a typical change takes | Cost or change time is rising across platforms, or a decommission is stalling | Normal variation |

**My signals, in one glance each month:** any new Red; any rating that has slipped two months running; the same issue on more than one platform; any Amber without an owner. Those four set my agenda; the rest is the team's.

## My rhythm

Each layer of the rhythm feeds the next: the daily scan keeps the week on track, the week feeds the monthly review, and the monthly reviews feed the quarter.

| Cadence | What I do | Time | Template |
| --- | --- | --- | --- |
| Daily | Scan my one list: decisions due, commitments, people to reply to; clear or delegate one item | 10 min, start of day | Daily scan |
| Weekly | Team review; engineering head sync; 1:1s; two focus blocks; close the week | See "My week" | Friday close |
| Monthly | Read all scorecards and pain logs before the team's health review; prepare for the joint session with ops, product and engineering; check my own measures | 60 min prep, plus the meetings | My monthly review |
| Quarterly | Prepare for the team's quarterly review and product's planning; review my priorities, relationships and team; reset my five priorities | Half a day, two weeks before product's planning | My quarterly review |
| Annually | Refresh principles and 12–24 month direction with the business plan; my own development plan | One day | — |

### My week

| When | What | Why |
| --- | --- | --- |
| Monday, 30 min | Review my one list: decisions due, commitments, people to see | Start the week knowing where things stand |
| Tuesday | Architecture team review (60 min); focus block in the afternoon (90 min) | Decisions made in one place; time to think |
| Wednesday | Engineering head sync; relationship meetings grouped together | Relationships without scattering the week |
| Thursday | Architect 1:1s; second focus block (90 min) | Develop the team; protect strategy time |
| Friday, 20 min | Close the week: log decisions, close or reassign open items | Nothing carried in my head over the weekend |
| US/India overlap | Two or three fixed windows a week; the rest in writing | No ad hoc early or late calls |

**My rules:**

- When asked for a decision, I ask "What do you recommend?" first.
- I ask "Which part of this is mine?" before taking on an escalation. Schedules and delivery belong to engineering.
- I comment on the decision and the risk, not the wording. 80% from others is good enough.
- Five priorities a quarter. "Not this quarter" is a normal answer.
- Focus blocks are meetings. I move them only for a real cross-platform or enterprise decision.
- I keep a short running note per key stakeholder, so meeting prep takes 10 minutes.
- I drop any recurring meeting that has produced no decision in four sittings.

## What I produce

I produce a small set of artifacts that set direction across platforms and win support from senior leaders. Everything at the level of a single platform is produced by the team; I review it, I don't write it.

| Artifact | Purpose | Audience | When | Length | Built from |
| --- | --- | --- | --- | --- | --- |
| **Architecture strategy** | The 12–24 month direction across all three portfolios: principles, the biggest shifts, what we retire, what we need funded | Ops, product, engineering and Enterprise Architecture leadership; my manager | Annually, refreshed at mid-year | 6–8 pages | Platform target states, health trends, business plan |
| **Portfolio or domain strategy** (for example cash management, post-trade) | Direction for one domain where a multi-year change is needed | Leaders of that domain; investment forum | When a domain needs a major shift; reviewed yearly | 8–12 pages, or a white paper plus a short deck | Target states, current-state assessment, industry practice |
| **Investment case** | Win funding or priority for one change | Head of Product, investment forum, finance | When a proposal is ready; usually one or two a quarter | 1–2 pages | Pain logs, health data, four-part shape |
| **Quarterly platform health and direction summary** | Show leaders where platforms stand, what changed, what we need from them | Head of MO Operations, Head of Product, engineering head, my manager | Quarterly, after my quarterly review | 1 page | Scorecards, quarterly review |
| **Decision memo** | Record a cross-platform or enterprise-level decision and its rationale | The forum deciding; affected teams | As needed | 1–2 pages | ADRs, options analysis |
| **Principles and the team operating model** | How the team works and judges designs | My architects; partners | Annually | As short as possible | Team input |

**Produced by the team, reviewed by me:** platform target states, health scorecards, pain logs, ADRs, solution overviews, one-page inputs to product's quarterly planning, and exception records.

**What I don't produce:** solution designs, detailed architecture documents, status slides or meeting decks rebuilt from data that already exists in the scorecards.

**One source, many uses.** The quarterly summary reuses the scorecards; investment cases reuse the pain logs; the annual strategy reuses the target states. If I find myself gathering data from scratch, a team artifact is missing.

## Writing a strategy document

A strategy document makes a small number of choices, explains why, and asks for something. If it doesn't change what anyone does or funds, it's a description, not a strategy.

**Structure (architecture or domain strategy):**

| Section | What it answers | Length |
| --- | --- | --- |
| 1. Summary | The three or four choices we are making, what they deliver, and what we need, readable on its own | Half a page |
| 2. Business context | What the business is trying to achieve over two years, and what is changing: products, volume, regulation, cost pressure | Half a page |
| 3. Where we are | Platform health across the dimensions, the biggest gaps, and what they cost the business today | One page, one picture |
| 4. Choices and principles | The few strategic choices, each with the alternative we are not taking and why | One page |
| 5. Target state | Where the platforms will be in 12–24 months; what is shared, what is retired | One page, one diagram |
| 6. Roadmap | The order of steps, what each one delivers and retires, and the dependencies; sequence matters more than dates | Half a page |
| 7. Investment and value | What it costs, what it saves or unlocks, and what happens if we do nothing | Half a page |
| 8. Risks and dependencies | What could stop it, and how we will know early | Short list |
| 9. Decisions requested | Exactly what we need from this audience, by when | Short list |
| 10. Measures | Three to five measures with today's baseline | Short table |

**Guidance:**

- **Start from the ask.** Write section 9 first, then everything else so it earns that ask.
- **Make real choices.** Every choice names what we are not doing. "Modernize the platform" is not a choice; "one trade record shared through events, instead of copies per team" is.
- **Use their language.** Write for the Head of MO Operations, not for architects. Put system names in an appendix.
- **Show, don't list.** One current-state picture and one target-state diagram do more than pages of text.
- **Separate evidence from assumptions,** and label estimates as estimates.
- **Keep it short.** 6–8 pages for the architecture strategy, with detail in appendices or the team's target states.

**How I build one (about four weeks):**

1. **Week 1:** form a working hypothesis from the scorecards, pain logs and target states; test it in two or three conversations with ops, product and engineering leaders.
2. **Week 2:** draft sections 1, 4 and 9 first: the choices and the ask. Ask the architects to supply sections 3, 5 and 6 from their platforms.
3. **Week 3:** review with the engineering head and the Head of Product before anyone senior sees it, so there are no surprises.
4. **Week 4:** finalize and pre-wire with my manager and the key decision makers; present a short deck built from the summary.

**Pairing a strategy with a deck:** the document holds the argument; the deck holds the summary, the two diagrams and the ask, about six slides. Write the document first.

## How I communicate with senior leaders

I lead with the answer, say plainly what I don't know, and turn technology constraints into business choices.

**Answer first.** One sentence with the answer, one or two reasons, then stop. Detail only when asked.

**When I don't know.** I say what I know, what I don't, and when they'll have the answer. Then I follow up on time.

- "I don't have that number. I'll get it to you by Thursday."
- "That sits with engineering. I'll get you the answer from them by tomorrow."

**Before a meeting with a head:** five minutes on "What three questions will they ask me?"

**When it is "all IT" to them:**

1. Acknowledge the impact: "You're right, this is slowing you down."
2. Name the constraint in business terms: "One team can work on this platform, and it is committed until March."
3. Give options with tradeoffs: move the team and delay something else, add cost, or wait.
4. Ask for the decision or support: "Which matters more to you?"

**Rules:**

- Align with the Head of MO Engineering before raising their constraints with the business. No surprises.
- Say "we" for technology, never "engineering doesn't…".
- No long build-up, no over-apologizing, no bluffing.

**Phrases to keep ready:** "The short answer is…" · "I don't know yet. I'll have it by…" · "Here's the tradeoff…" · "What I need from you is…" · "I'd recommend X, because…"

## How I make the case for investment

I form a view early, label how sure I am, and present every investment in the same four-part shape. Patterns are gathered continuously, so the answer is ready before anyone asks.

**Lead with a hypothesis.** "My working view is X. I'm fairly confident, not certain. I'll know by \[date\] once we have \[evidence\]." A view with a confidence level and a date is direct without overclaiming.

**Find patterns before they are needed.** Each architect keeps a running pain log for their area: incidents, breaks, manual workarounds, delays, and requests from ops and product. I read across all of them for 30 minutes in the monthly strategy review. Repeats across areas are investment candidates.

**One shape for every investment argument:**

1. **The problem in their terms:** manual work, risk, delay or cost, with one number.
2. **The cost of doing nothing:** what gets worse, and when.
3. **What we would change:** one sentence, no system names.
4. **What it unlocks:** the outcome they care about, and the first sign it is working.

**Close expectation gaps by asking.** Ask each senior partner "Six months from now, what would make architecture useful to you?" and ask my manager "What would I be doing differently that you'd notice?" Write the answers down and play them back.

**Buy time well.** "I'll come back with a clear view by Thursday" is a strong answer, as long as Thursday's answer arrives in the four-part shape.

### Investment case: cash platform

1. **Problem:** we see about 20,000 cash variances a month (estimate), and the batch reconciliation cannot tell us during the day which funds' cash is trustworthy. The same feeds are read by both cash applications, and the buffer is applied in both.
2. **Cost of doing nothing:** managers keep investing against numbers we cannot vouch for; double counting and day-0 differences stay invisible; every new vehicle adds more of the same. Under T+1, errors become overdrafts or failed purchases faster.
3. **What we would change:** one buffer policy and one baseline contract for both applications, an intraday check by fund that flags weak numbers, and every flow entering once through a shared event layer.
4. **What it unlocks:** a cash number the manager can trust and see the confidence of. First sign: cash available and projection day 0 agree every morning, and the variance count starts to fall.

*Before using it:* replace the 20,000 estimate with a month of real data, and add the cost of one recent overdraft or near miss if there is one.

### Investment case: post-trade platform

1. **Problem:** every trade's lifecycle is tracked through a separate Activiti process, and trade messages to downstream teams run on a team-run broker outside enterprise standards. \[Add one number: daily trade volume or a recent lifecycle-related incident.\]
2. **Cost of doing nothing:** lifecycle tracking limits throughput as volume grows; every lifecycle change needs a process redeploy; the non-standard broker carries support and resilience risk that rises over time.
3. **What we would change:** the trade record holds its own lifecycle status, moved forward by trade and custodian messages, and trade messages move to the enterprise event platform.
4. **What it unlocks:** same-day lifecycle status for every trade, faster product changes, and a standard, supported messaging backbone. First sign: the first asset type runs fully off Activiti with no status differences in parallel run.

*Before using it:* confirm volume growth expectations with the Head of Product and the support position on the current broker with engineering.

## My templates

Copy each into a dated note when you use it. They draw on the team's scorecards and pain logs, so they need no separate data gathering.

### Daily scan (10 minutes)

- [ ] Decisions due today or tomorrow: mine, or am I just informed?
- [ ] Commitments I made that are due this week
- [ ] Anything new that needs a senior conversation today
- [ ] One item to clear, delegate or say "not this quarter" to

### Friday close (20 minutes)

- [ ] Log decisions made this week
- [ ] Close or reassign every open item on my list
- [ ] Did both focus blocks happen? If not, what took them?
- [ ] Where did I step back this week, and where did I get pulled into solution, analysis or engineering work?
- [ ] One thing to hand off next week

### My monthly review (60 minutes, before the team's health review)

**1. Platform health (20 min).** Read every scorecard and pain log, then note only what needs me:

| Signal | Platforms | What I will do | By when |
| --- | --- | --- | --- |
| New Red |  |  |  |
| Slipped two months running |  |  |  |
| Same issue on more than one platform |  |  |  |
| Amber without an owner |  |  |  |

**2. Priorities and product (15 min).** Which high-impact epics are coming? What will I raise in the joint session with ops, product and engineering? Is there an investment case forming (four-part shape)?

**3. Relationships (10 min).** Who have I not spoken with this month who matters? Any promise to follow up on?

**4. Me (15 min).**

| Check | This month | Note |
| --- | --- | --- |
| Share of time on direction, relationships and people (target about half) |  |  |
| Focus blocks held (target 8 a month) |  |  |
| Questions that reached me that an architect could have answered |  |  |
| One thing I handed off |  |  |
| One thing I will stop or hand off next month |  |  |

### My quarterly review (half a day, two weeks before product's planning)

**1. Platform health over the quarter (45 min).** Compare three months of scorecards. Which dimensions improved or slipped on each platform? What is the single biggest risk across the portfolio going into next quarter?

**2. Direction (45 min).** Are the target states still right? What did we retire? Which one or two investment cases will I take forward, and to whom?

**3. Product's priorities (30 min).** Review the architects' one-page inputs. What sequencing will we propose? What will I say to the Head of Product before planning?

**4. Relationships (30 min).**

| Partner | State of the relationship | What they need from me | What I need from them | Next step |
| --- | --- | --- | --- | --- |
| Head of MO Operations |  |  |  |  |
| Head of Product |  |  |  |  |
| Head of MO Engineering |  |  |  |  |
| Enterprise Architecture |  |  |  |  |
| My manager |  |  |  |  |

**5. Team (30 min).** For each architect: growth goal progress, cover and succession, anything I am still doing that they should own.

**6. My priorities for next quarter (30 min).** Set five, at most:

| Priority | Why it matters | What done looks like | First step |
| --- | --- | --- | --- |
| 1. |  |  |  |
| 2. |  |  |  |
| 3. |  |  |  |
| 4. |  |  |  |
| 5. |  |  |  |

## Keeping myself honest

The shift from leading solutions to leading the platform is a habit change, so I check it on purpose.

- **Four personal measures, checked monthly:** share of time on direction, relationships and people; focus blocks held; questions reaching me that an architect could answer; items handed off.
- **Weekly reflection:** the Friday close asks where I stepped back and where I got pulled in.
- **Expectations, made explicit:** ask my manager each quarter "What would I be doing differently that you'd notice?" and ask each senior partner what would make architecture useful to them in six months. Write the answers down and play them back.
- **A mentor:** a senior technology leader outside my reporting line who has made the move from architect to executive; 30 minutes a month.

If a month goes badly, I don't add process. I pick the one habit that slipped and restart it.
