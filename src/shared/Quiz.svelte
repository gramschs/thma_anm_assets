<!--
  ═══════════════════════════════════════════════════════════════════════════════
  Quiz.svelte  –  Gemeinsame Quiz-Komponente für das ANM-Skript (DE/EN)

  Übernommen aus thma_python_assets (Themen, Lernstand, Code-Darstellung)
  und erweitert um:
  ─ lang:     'de' (Standard, Anrede "Sie") oder 'en' für alle festen Texte
  ─ messages: optional { perfect, good, weak } für die Schlussbotschaft,
              sonst allgemeine Standardtexte in der gewählten Sprache

  Weitere Features (aktiv, sobald die Fragen die Felder mitbringen):
  ─ topic:  Fragen tragen ein Themen-Feld → Auswertung pro Thema am Ende
  ─ id:     stabile Fragen-IDs → "Noch einmal üben" zieht bevorzugt falsch
            beantwortete, dann noch nicht gesehene Fragen (localStorage)
  ─ Code und Formeln: Fragen dürfen HTML enthalten (Helfer code()/block()
            und tex() in der jeweiligen Quiz-Datei definieren)

  Rückmeldung pro Antwort (empfohlen): options als Objekte { text, why }.
  why der richtigen Antwort ist die Begründung, why jeder falschen Antwort
  erklärt den Fehler. Die Komponente baut die Rückmeldung selbst zusammen:
    richtig → "Richtig." + Begründung, falsche Antworten eingeklappt
    falsch  → "Nicht ganz." + "Richtig ist X." + Begründung, Leerzeile,
              Liste der falschen Antworten, gewählte zuerst und markiert
  Weil die Antworten gemischt werden, nennt die Rückmeldung jede Antwort über
  ihren Text, nie über ihre Position. Alte Fragen mit options als Strings und
  einem Feld explanation funktionieren weiterhin.
  ═══════════════════════════════════════════════════════════════════════════════
-->

<script>
  export let title = 'Lernkontrolle';
  export let pool  = [];
  export let nShow = 3;
  export let hint  = '';          // z. B. 'Abschnitt 9.1' (optional)
  export let storageKey = null;   // optional; sonst aus title abgeleitet
  export let lang  = 'de';        // 'de' oder 'en'
  export let messages = null;     // optional { perfect, good, weak }

  // ── Feste Texte in beiden Sprachen (Deutsch mit Anrede "Sie") ─────────────
  const TEXTS = {
    de: {
      check: 'Prüfen', next: 'Weiter →', results: 'Auswertung',
      correct: 'Richtig.', notQuite: 'Nicht ganz.', nCorrect: 'richtig',
      again: 'Noch einmal üben', reset: 'Lernstand zurücksetzen',
      correctIs: 'Richtig ist', othersWrong: 'Die anderen Antworten sind falsch:',
      whyOthers: 'Warum die anderen Antworten falsch sind', yourAnswer: 'Ihre Antwort',
      perfect: 'Alles richtig, sehr gut!',
      good: 'Gut! Wiederholen Sie die Themen mit Fehlern. Beim nächsten Durchlauf kommen diese Fragen zuerst.',
      weak: 'Wiederholen Sie die Themen mit Fehlern und versuchen Sie es dann noch einmal. Die falsch beantworteten Fragen kommen zuerst.',
    },
    en: {
      check: 'Check', next: 'Next →', results: 'Results',
      correct: 'Correct.', notQuite: 'Not quite.', nCorrect: 'correct',
      again: 'Practice again', reset: 'Reset progress',
      correctIs: 'The correct answer is', othersWrong: 'The other answers are wrong:',
      whyOthers: 'Why the other answers are wrong', yourAnswer: 'Your answer',
      perfect: 'All correct, well done!',
      good: 'Good! Review the topics with mistakes. These questions come first in the next round.',
      weak: 'Review the topics with mistakes and then try again. The questions you got wrong come first.',
    },
  };
  const T = TEXTS[lang] ?? TEXTS.de;

  const KEY = storageKey ?? `quiz:${title}`;

  // ── Pool vorbereiten: IDs ergänzen, Themen erkennen ──────────────────────
  const POOL = pool.map((q, i) => ({ ...q, id: q.id ?? `q${i}` }));
  const hasTopics = POOL.some(q => q.topic);

  // ── Lernstand (gesehen / falsch) laden ────────────────────────────────────
  let seen  = new Set();
  let wrong = new Set();
  try {
    const saved = JSON.parse(localStorage.getItem(KEY) ?? 'null');
    if (saved) {
      seen  = new Set(saved.seen  ?? []);
      wrong = new Set(saved.wrong ?? []);
    }
  } catch (e) { /* privater Modus o. Ä. – ohne Persistenz weiterarbeiten */ }

  function persist() {
    try {
      localStorage.setItem(KEY, JSON.stringify({
        seen:  [...seen],
        wrong: [...wrong],
      }));
    } catch (e) { /* ignorieren */ }
  }

  // ── Misch-Hilfsfunktion ───────────────────────────────────────────────────
  function shuffle(arr) {
    const a = [...arr];
    for (let i = a.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [a[i], a[j]] = [a[j], a[i]];
    }
    return a;
  }

  // ── Rundenaufbau mit Priorität ────────────────────────────────────────────
  // 1. falsch beantwortete Fragen (größter Lerneffekt)
  // 2. noch nicht gesehene Fragen
  // 3. bereits richtig beantwortete Fragen (Auffüllen)
  // Innerhalb jeder Gruppe und am Ende wird gemischt, damit die falschen
  // Fragen nicht erkennbar am Anfang stehen.
  function buildRound() {
    const w = POOL.filter(q => wrong.has(q.id));
    const u = POOL.filter(q => !seen.has(q.id));
    const r = POOL.filter(q => seen.has(q.id) && !wrong.has(q.id));

    const chosen = [...shuffle(w), ...shuffle(u), ...shuffle(r)]
      .slice(0, Math.min(nShow, POOL.length));

    return shuffle(chosen).map(q => {
      const indexed  = q.options.map((opt, i) => ({
        opt: typeof opt === 'string' ? { text: opt, why: '' } : opt,
        isCorrect: i === q.correct,
      }));
      const shuffled = shuffle(indexed);
      return {
        id:          q.id,
        topic:       q.topic ?? null,
        text:        q.text,
        options:     shuffled.map(o => o.opt),
        correct:     shuffled.findIndex(o => o.isCorrect),
        explanation: q.explanation,
      };
    });
  }

  // ── Reaktiver Zustand ─────────────────────────────────────────────────────
  let questions    = buildRound();
  let current      = 0;
  let selected     = null;
  let submitted    = false;
  let score        = 0;
  let completed    = false;
  let roundResults = [];   // { topic, ok } pro beantworteter Frage

  const LABELS = ['A', 'B', 'C', 'D'];

  // ── Aktionen ──────────────────────────────────────────────────────────────
  function select(i) { if (!submitted) selected = i; }

  function submit() {
    if (selected === null || submitted) return;
    submitted = true;
    const q  = questions[current];
    const ok = selected === q.correct;
    if (ok) { score += 1; wrong.delete(q.id); }
    else    { wrong.add(q.id); }
    seen.add(q.id);
    roundResults = [...roundResults, { topic: q.topic, ok }];
    persist();
  }

  function next() {
    if (current < questions.length - 1) {
      current++;  selected = null;  submitted = false;
    } else {
      completed = true;
    }
  }

  function reset() {
    questions = buildRound();   // falsche zuerst, dann neue Fragen
    current = 0;  selected = null;  submitted = false;
    score = 0;    completed = false;
    roundResults = [];
  }

  function clearProgress() {
    seen = new Set();  wrong = new Set();
    persist();
    reset();
  }

  // ── Abgeleitete Größen ────────────────────────────────────────────────────
  $: q         = questions[current];
  $: isCorrect = submitted && selected === q.correct;

  // Rückmeldung pro Antwort, falls alle Antworten ein why mitbringen
  $: perOption  = q.options.every(o => o.why);
  $: correctOpt = q.options[q.correct];
  // falsche Antworten, die gewählte zuerst
  $: wrongOpts  = q.options
    .map((o, i) => ({ ...o, chosen: i === selected }))
    .filter((o, i) => i !== q.correct)
    .sort((a, b) => b.chosen - a.chosen);

  // Auswertung pro Thema (nur wenn die Fragen Themen tragen)
  $: topicStats = (() => {
    if (!hasTopics) return [];
    const m = new Map();
    for (const r of roundResults) {
      if (!r.topic) continue;
      const s = m.get(r.topic) ?? { ok: 0, total: 0 };
      s.total += 1;
      if (r.ok) s.ok += 1;
      m.set(r.topic, s);
    }
    return [...m.entries()].map(([topic, s]) => ({ topic, ...s }));
  })();

  $: ratio = questions.length > 0 ? score / questions.length : 0;
  $: msg = { ...T, ...(messages ?? {}) };
  $: resultMsg = (ratio === 1 ? msg.perfect : ratio >= 0.7 ? msg.good : msg.weak)
    + (ratio < 1 && hint ? ' ' + hint : '');
</script>


<!-- ══════════════════════════════════════════════════════════════════════════
     TEMPLATE
     ══════════════════════════════════════════════════════════════════════════ -->

<div class="applet">

  <!-- Kompakter Header mit integriertem Zähler -->
  <header>
    <span class="header-title">{title}</span>
    {#if !completed}
      <span class="header-counter">{current + 1}&thinsp;/&thinsp;{questions.length}</span>
    {:else}
      <span class="header-counter">{score}&thinsp;/&thinsp;{questions.length} {T.nCorrect}</span>
    {/if}
  </header>

  <!-- Fortschrittsbalken -->
  {#if !completed}
    <div class="progress-wrap">
      <div class="progress-fill"
           style="width: {Math.round((current / questions.length) * 100)}%">
      </div>
    </div>
  {:else}
    <div class="progress-wrap">
      <div class="progress-fill progress-done" style="width: 100%"></div>
    </div>
  {/if}

  <!-- ── Quiz ─────────────────────────────────────────────────────────── -->
  {#if !completed}
    <div class="quiz-body">

      {#if q.topic}
        <span class="topic-label">{q.topic}</span>
      {/if}

      <p class="q-text">{@html q.text}</p>

      <div class="options">
        {#each q.options as opt, i}
          <button
            class="option"
            class:sel={selected === i && !submitted}
            class:correct={submitted && i === q.correct}
            class:wrong={submitted && selected === i && i !== q.correct}
            class:dimmed={submitted && i !== q.correct && selected !== i}
            on:click={() => select(i)}
            disabled={submitted}
          >
            <span class="opt-label">{LABELS[i]}</span>
            <span class="opt-text">{@html opt.text}</span>
          </button>
        {/each}
      </div>

      {#if submitted}
        <div class="feedback" class:fb-ok={isCorrect} class:fb-err={!isCorrect}>
          <span class="fb-icon">{isCorrect ? '✓' : '✗'}</span>
          <div class="fb-content">
            <strong>{isCorrect ? T.correct : T.notQuite}</strong>
            {#if perOption}
              <div class="fb-text">
                {#if isCorrect}
                  <p>{@html correctOpt.why}</p>
                  <details class="fb-more">
                    <summary>{T.whyOthers}</summary>
                    <ul>
                      {#each wrongOpts as o}
                        <li>{@html o.text}: {@html o.why}</li>
                      {/each}
                    </ul>
                  </details>
                {:else}
                  <p>{T.correctIs} {@html correctOpt.text}. {@html correctOpt.why}</p>
                  <p>{T.othersWrong}</p>
                  <ul>
                    {#each wrongOpts as o}
                      <li class:chosen={o.chosen}>
                        {#if o.chosen}<span class="your-answer">{T.yourAnswer}</span>{/if}
                        {@html o.text}: {@html o.why}
                      </li>
                    {/each}
                  </ul>
                {/if}
              </div>
            {:else}
              <div class="fb-text">{@html q.explanation ?? ''}</div>
            {/if}
          </div>
        </div>
      {/if}

      <div class="actions">
        {#if !submitted}
          <button class="btn-primary" on:click={submit} disabled={selected === null}>
            {T.check}
          </button>
        {:else}
          <button class="btn-primary" on:click={next}>
            {current < questions.length - 1 ? T.next : T.results}
          </button>
        {/if}
      </div>

    </div>

  <!-- ── Ergebnis ──────────────────────────────────────────────────────── -->
  {:else}
    <div class="result-body">
      <p class="score-display">
        <span class="score-num">{score}</span><span class="score-denom">&thinsp;/ {questions.length}</span>
      </p>

      {#if topicStats.length > 0}
        <div class="topic-stats">
          {#each topicStats as t}
            <div class="topic-row"
                 class:topic-ok={t.ok === t.total}
                 class:topic-weak={t.ok < t.total}>
              <span class="topic-name">{t.topic}</span>
              <span class="topic-score">{t.ok}&thinsp;/&thinsp;{t.total}</span>
            </div>
          {/each}
        </div>
      {/if}

      <p class="result-msg">{resultMsg}</p>

      <button class="btn-secondary" on:click={reset}>{T.again}</button>
      <button class="btn-link" on:click={clearProgress}>{T.reset}</button>
    </div>
  {/if}

</div>


<!-- ══════════════════════════════════════════════════════════════════════════
     STYLES
     ══════════════════════════════════════════════════════════════════════════ -->

<style>
  *, *::before, *::after { box-sizing: border-box; margin: 0; padding: 0; }

  /* ── Container inkl. Design-Token ───────────────────────────────────────── */
  .applet {
    --surface: #ffffff;
    --border:  #ddddd0;
    --text:    #1c1c1a;
    --muted:   #666660;
    --radius:  10px;
    --shadow:  0 3px 16px rgba(0,0,0,.09);
    --blue:    #005a94;
    --blue-lt: #eef4fa;
    --green:   #2a7a38;
    --green-lt:#edf7ef;
    --red:     #c0392b;
    --red-lt:  #fdf0ef;
    --font-b:  'Georgia', 'Times New Roman', serif;
    --font-ui: 'Helvetica Neue', Helvetica, Arial, sans-serif;
    --font-mono: 'SF Mono', 'Cascadia Code', Consolas, Menlo, monospace;

    color-scheme: light;
    font-family: var(--font-b);
    color: var(--text);
    background: var(--surface);
    border: 1px solid var(--border);
    border-radius: var(--radius);
    box-shadow: var(--shadow);
    overflow: hidden;
    width: 100%;
    max-width: 680px;
    margin: 0 auto;
  }

  /* ── Header (kompakt, einspaltig) ───────────────────────────────────────── */
  header {
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 14px 20px 13px;
    background: #f3f4f6;
    border-bottom: 1px solid var(--border);
  }
  .header-title {
    font-family: var(--font-ui);
    font-size: .95rem;
    font-weight: 600;
    letter-spacing: .01em;
    color: var(--text);
  }
  .header-counter {
    font-family: var(--font-ui);
    font-size: .8rem;
    font-weight: 700;
    font-variant-numeric: tabular-nums;
    color: var(--muted);
  }

  /* ── Fortschrittsbalken ─────────────────────────────────────────────────── */
  .progress-wrap { height: 3px; background: var(--border); }
  .progress-fill {
    height: 100%;
    background: var(--blue);
    transition: width .3s ease;
  }
  .progress-done { background: var(--green); }

  /* ── Quiz-Körper ─────────────────────────────────────────────────────────── */
  .quiz-body {
    padding: 20px 20px 18px;
    display: flex;
    flex-direction: column;
    gap: 14px;
  }

  .topic-label {
    font-family: var(--font-ui);
    font-size: .68rem;
    font-weight: 700;
    letter-spacing: .08em;
    text-transform: uppercase;
    color: var(--muted);
    margin-bottom: -8px;
  }

  .q-text {
    font-size: .97rem;
    line-height: 1.7;
    color: var(--text);
  }

  /* ── Code-Darstellung (per {@html} eingefügt → :global) ─────────────────── */
  .quiz-body :global(code) {
    font-family: var(--font-mono);
    font-size: .84em;
    background: #f1f1ea;
    border: 1px solid #e6e6da;
    border-radius: 4px;
    padding: 1px 5px;
    white-space: pre;
  }
  .quiz-body :global(pre) {
    background: #f7f7f2;
    border: 1px solid var(--border);
    border-radius: 8px;
    padding: 10px 13px;
    margin-top: 9px;
    overflow-x: auto;
  }
  .quiz-body :global(pre code) {
    background: none;
    border: none;
    padding: 0;
    display: block;
    font-size: .82rem;
    line-height: 1.6;
  }

  .quiz-body :global(.katex) { font-size: 1.05em; }

  /* ── Optionen ────────────────────────────────────────────────────────────── */
  .options { display: flex; flex-direction: column; gap: 7px; }

  .option {
    display: flex;
    align-items: center;
    gap: 11px;
    padding: 10px 13px;
    border: 1.5px solid var(--border);
    border-radius: 8px;
    background: var(--surface);
    cursor: pointer;
    text-align: left;
    font-family: var(--font-ui);
    font-size: .88rem;
    color: var(--text);
    line-height: 1.5;
    transition: border-color .13s, background .13s, opacity .13s;
    width: 100%;
  }
  .option:hover:not(:disabled):not(.correct):not(.wrong) {
    border-color: var(--blue);
    background: var(--blue-lt);
  }
  .option.sel     { border-color: var(--blue);  background: var(--blue-lt); }
  .option.correct { border-color: var(--green); background: var(--green-lt); }
  .option.wrong   { border-color: var(--red);   background: var(--red-lt); }
  .option.dimmed  { opacity: .38; }
  .option:disabled { cursor: default; }

  .opt-label {
    flex-shrink: 0;
    width: 22px; height: 22px;
    border-radius: 50%;
    background: #e8e8e0;
    font-size: .72rem; font-weight: 700;
    display: flex; align-items: center; justify-content: center;
    color: var(--muted);
    font-family: var(--font-ui);
    transition: background .13s, color .13s;
  }
  .option.correct .opt-label { background: var(--green); color: #fff; }
  .option.wrong   .opt-label { background: var(--red);   color: #fff; }
  .option.sel:not(.correct):not(.wrong) .opt-label { background: var(--blue); color: #fff; }

  /* overflow-x: auto für lange Code-Zeilen. Senkrecht nicht scrollen und oben
     und unten Luft lassen: KaTeX-Brüche ragen sonst einige Pixel über die
     Zeile hinaus, und der Browser zeigt eine kleine Scrollleiste. */
  .opt-text { flex: 1; min-width: 0; overflow-x: auto; overflow-y: hidden; padding: 4px 0; }

  /* ── Feedback ────────────────────────────────────────────────────────────── */
  .feedback {
    display: flex;
    gap: 10px;
    align-items: flex-start;
    padding: 11px 13px;
    border-radius: 8px;
    border: 1.5px solid var(--border);
    background: #f8f8f5;
    font-family: var(--font-ui);
    font-size: .84rem;
    line-height: 1.55;
  }
  .fb-ok  { border-color: var(--green); background: var(--green-lt); }
  .fb-err { border-color: var(--red);   background: var(--red-lt); }

  .fb-icon {
    font-size: 1.05rem; font-weight: 700; flex-shrink: 0; margin-top: 1px;
  }
  .fb-ok  .fb-icon { color: var(--green); }
  .fb-err .fb-icon { color: var(--red); }

  .fb-content { display: flex; flex-direction: column; gap: 3px; }
  .fb-content strong { color: var(--text); }

  /* Erklärung: Absätze mit Leerzeile dazwischen, Liste der falschen Antworten */
  .fb-text :global(p + p) { margin-top: .9em; }
  .fb-text :global(ul)    { padding-left: 1.3em; margin-top: .3em; }
  .fb-text :global(li + li) { margin-top: .25em; }
  .fb-text li.chosen { font-weight: 600; }
  .your-answer {
    display: inline-block;
    margin-right: 6px;
    padding: 0 6px;
    border-radius: 4px;
    background: var(--red);
    color: #fff;
    font-size: .68rem;
    font-weight: 700;
    letter-spacing: .05em;
    text-transform: uppercase;
    vertical-align: 1px;
  }
  .fb-more { margin-top: .6em; }
  .fb-more summary { cursor: pointer; color: var(--muted); }
  .fb-more summary:hover { color: var(--text); }

  /* ── Aktionen ────────────────────────────────────────────────────────────── */
  .actions { display: flex; justify-content: flex-end; }

  .btn-primary {
    padding: 9px 20px;
    background: var(--blue);
    color: #fff;
    border: none;
    border-radius: 7px;
    font-family: var(--font-ui);
    font-size: .86rem; font-weight: 600;
    cursor: pointer;
    transition: opacity .13s;
  }
  .btn-primary:hover:not(:disabled) { opacity: .84; }
  .btn-primary:disabled { opacity: .3; cursor: default; }

  .btn-secondary {
    padding: 9px 20px;
    background: transparent;
    color: var(--blue);
    border: 1.5px solid var(--blue);
    border-radius: 7px;
    font-family: var(--font-ui);
    font-size: .86rem; font-weight: 600;
    cursor: pointer;
    transition: background .13s;
    margin-top: 4px;
  }
  .btn-secondary:hover { background: var(--blue-lt); }

  .btn-link {
    background: none;
    border: none;
    color: var(--muted);
    font-family: var(--font-ui);
    font-size: .76rem;
    cursor: pointer;
    text-decoration: underline;
    margin-top: 2px;
  }
  .btn-link:hover { color: var(--text); }

  /* ── Ergebnis ────────────────────────────────────────────────────────────── */
  .result-body {
    padding: 32px 20px 28px;
    display: flex; flex-direction: column;
    align-items: center; gap: 12px; text-align: center;
  }
  .score-display { line-height: 1; }
  .score-num {
    font-family: var(--font-ui);
    font-size: 3.2rem; font-weight: 700;
    color: var(--blue);
    font-variant-numeric: tabular-nums;
  }
  .score-denom {
    font-family: var(--font-ui);
    font-size: 1.5rem; color: var(--muted);
  }

  .topic-stats {
    display: flex;
    flex-direction: column;
    gap: 6px;
    width: 100%;
    max-width: 340px;
  }
  .topic-row {
    display: flex;
    justify-content: space-between;
    align-items: center;
    gap: 12px;
    padding: 7px 12px;
    border-radius: 7px;
    border: 1px solid var(--border);
    background: #fafaf7;
    font-family: var(--font-ui);
    font-size: .82rem;
  }
  .topic-row.topic-ok   { border-color: var(--green); background: var(--green-lt); }
  .topic-row.topic-weak { border-color: var(--red);   background: var(--red-lt); }
  .topic-name  { text-align: left; }
  .topic-score {
    font-weight: 700;
    font-variant-numeric: tabular-nums;
    color: var(--muted);
    flex-shrink: 0;
  }

  .result-msg {
    font-family: var(--font-ui);
    font-size: .9rem; line-height: 1.55;
    color: var(--text); max-width: 400px;
  }

  /* ── Responsiv ───────────────────────────────────────────────────────────── */
  @media (max-width: 480px) {
    header          { padding: 12px 14px; }
    .quiz-body      { padding: 16px 14px 14px; }
    .result-body    { padding: 26px 14px 22px; }
    .option         { padding: 9px 11px; font-size: .84rem; }
    .score-num      { font-size: 2.6rem; }
  }
</style>
