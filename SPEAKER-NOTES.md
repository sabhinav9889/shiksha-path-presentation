# Speaker notes

About 15 minutes for the talk and 5 for questions. Slide numbers match `index.html` and the PDF.
Speak plainly. Where a number is on the slide, say it once and move on.

## Part 1: the product (about 5 minutes)

**1. Title (20 s).** "Shiksha Path is a learning app that works offline, in Hindi, on a shared phone. I built
it with Claude Code, and today I will show the product, how it works inside, and how I used AI to
build it."

**2. The problem (60 s).** Three groups. Students from villages and small towns who speak Hindi,
many of them minors, with weak English, patchy data and a phone shared by a family. Mentors who
cannot see who is stuck or hear how a student speaks. An NGO that must protect children's data.
Close on the line: most apps assume fast internet, English readers and one user per phone.

**3. The solution (45 s).** One product with two sides: the student app, and a staff section for
mentors and admins. Read the blue sentence once. Point out "PWA, no app store": it installs from a
link and runs on cheap Android phones.

**4. The student experience (75 s).** These are real screens from the live app. Home shows the
streak, progress in three tracks and today's lesson, and a badge that says progress is saved on the
phone. 28 lessons: English 15, aptitude 8, tech basics 5. The third screen is the one I care about:
for every wrong option there is a written reason, so the student learns from the mistake.

**5. The staff experience (60 s).** A mentor sees each cohort with flags for students who are
inactive, stuck or scoring low, plays a recording through a short-lived link and leaves a comment,
posts to the cohort and publishes one-to-one slots. An admin sets up labs, cohorts and staff, and
every staff change goes into an audit log with ids, never message text.

_Offer a live demo here if time allows: open shiksha-path.pages.dev, create a profile, answer one
question wrongly._

## Part 2: the technical side (about 5 minutes)

**6. Architecture (90 s).** Walk left to right. The phone holds the learner's data and an outbox.
Everything goes through one versioned API, `/v1`, running as a Supabase Edge Function: handler,
service, repository, with Zod checking every request. Postgres has row-level security on every
table; recordings sit in a private bucket and are played through 5-minute signed links. Hosting is
Cloudflare Pages, and CI runs tests and security scans on every pull request.

**7. Offline first (75 s).** The key idea: the phone is the primary store. Every action becomes an
event with a unique id, saved on the phone first. An outbox uploads in small batches with retry,
and the server accepts each id once, so retries never double-count. A lesson never waits for the
network. Old phones keep working because unknown fields are ignored.

**8. Engineering decisions (75 s).** Pick two or three: ports and adapters so Supabase can be
swapped; the same contract tests run on fakes and on real Postgres, which caught a real bug;
concurrency is the database's job, so double booking is stopped by a unique index and tested with
simultaneous requests. Mention the 120 KB first-load budget and the "deliberately not used" list.

**9. Security and privacy (60 s).** Because many students are minors: row-level security
everywhere, isolation tests with several students and cohorts, no personal data in logs or URLs,
signed links, export and delete on request, secrets only in environment settings. Be clear this is
the technical protection, not a compliance claim (see slide 17).

**10. Quality evidence (30 s).** Numbers as submitted on 30 October: 1,158 unit tests, 201 database
tests on real Postgres, 20 browser tests, 90.9% coverage. Since then more tests have been added.

## Part 3: how AI was used (about 5 minutes)

**11. How AI built it (45 s).** About 236 commits, and 189 of the 190 non-merge commits are
co-authored by Claude (the other is an automated dependency update). I merged every pull request.
"The question was not whether AI can write code. It was how to make its output trustworthy enough
for children's data."

**12. The loop (75 s).** Spec, plan, test first, gate, fresh review, fix pass, pull request, I
merge. Stress two things: a test that passed before the code existed was treated as broken, and the
reviewer is a separate model run that never saw the code, so it is not marking its own homework.

**13. Context engineering (60 s).** A new AI session starts from documents, not chat history:
CLAUDE.md, nine rule files, a task router, decision records, a gotchas folder, plans and a worklog.
That is what made long work across many sessions consistent.

**14. Agents, hooks and guardrails (60 s).** Four read-only reviewers that can only report; hooks
that block edits on `main` and writes to secret files; git hooks and CI that block on serious
findings. The AI features in the product (a doubt solver) are designed with guardrails but not
built; the product calls no AI today.

**15. The human in the loop (60 s).** The AI drafted; I decided. Read the orange box: the bugs that
mattered most came from me using the real product, such as audio blocked by the browser's security
policy and the right answer almost always being option A.

**16. How effective (60 s).** The table is the honest answer. The AI made real mistakes; the
process (independent review, tests that run on real Postgres, my own testing) found them. AI is
effective when it has rules, tests and an independent reviewer.

## Close (about 2 minutes)

**17. Honest limits (45 s).** Say these without being asked. Lessons are AI-drafted and approved by
me, not yet by mentors. No testing with real students or the cheapest phones. Not DPDP compliant
yet: consent, a data owner and legal review are needed. Several features are planned but not built.

**18. Five lessons (30 s).** Pick your favourite two. Suggested: test against the real browser and
database early, and a fresh reviewer beats the author.

**19. What happens next (30 s).** Three small features are already built locally and tested but not
deployed. Next: a field test with real students, a mentor review of lessons, child-safety basics,
then a new repo with its own Supabase and Cloudflare projects.

**20. Questions.**

## Likely questions, short answers

- **Is this just AI-generated code?** Largely yes, and I say so. My contribution is the decisions,
  the process that checks the output, and testing the real product. The evidence is in the repo.
- **Have real students used it?** No. The field test is the first next step.
- **Is the content good?** It is a foundation: 28 lessons drafted by AI, approved by me, and waiting
  for a mentor review.
- **Why no AI inside the product?** The brief did not require it. A doubt solver is designed, with
  guardrails written first, but not built.
- **Is it compliant with India's DPDP Act?** I cannot claim that. The technical protections exist;
  consent, a named data owner and legal review are still needed.
- **What was the biggest AI failure?** A paging cursor that passed every in-memory test and crashed
  on Postgres. It led to running the same tests on fakes and real Postgres.
- **What would you do with two more weeks?** A field test with a few students, a mentor review of the
  lessons, an accessibility pass, then notifications and feedback to students.
