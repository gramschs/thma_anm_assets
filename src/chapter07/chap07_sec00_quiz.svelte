<!--
  ═══════════════════════════════════════════════════════════════════════════════
  chap07_sec00_quiz.svelte  –  Vorkenntnis-Check zu Kapitel 7 (DE/EN)
  ─ Nur Fragen-Daten, Design und Ablauf liegen in shared/Quiz.svelte
  ─ Sprache über URL-Parameter: ?lang=en, sonst Deutsch
  ─ 3 Themen (4 + 3 + 3 Fragen), alle Fragen pro Durchlauf
  ─ KaTeX-CSS vom CDN, damit die Fonts nicht in die HTML-Datei wandern
  ─ Jede Antwort ist ein Objekt { text, why }: why der richtigen Antwort ist
    die Begründung, why einer falschen Antwort erklärt den typischen Fehler.
    Die Rückmeldung setzt shared/Quiz.svelte daraus zusammen. Die Antworten
    werden gemischt, why darf deshalb nie auf eine Position verweisen.
  ═══════════════════════════════════════════════════════════════════════════════
-->

<svelte:head>
  <link rel="stylesheet"
        href="https://cdn.jsdelivr.net/npm/katex@0.16.47/dist/katex.min.css" />
</svelte:head>

<script>
  import katex from 'katex';
  import Quiz from '../shared/Quiz.svelte';

  // ── Sprache ────────────────────────────────────────────────────────────────
  const lang = new URLSearchParams(window.location.search).get('lang') === 'en'
    ? 'en' : 'de';
  document.documentElement.lang = lang;

  // ── Helfer: Formeln und Code ───────────────────────────────────────────────
  const tex   = s => katex.renderToString(s, { throwOnError: false });
  const esc   = s => s.replace(/&/g, '&amp;').replace(/</g, '&lt;').replace(/>/g, '&gt;');
  const code  = s => `<code>${esc(s)}</code>`;
  const block = s => `<pre><code>${esc(s)}</code></pre>`;

  // ── Deutsche Fassung (Anrede "Sie") ────────────────────────────────────────
  const MA = 'Mathematik', PY = 'Python', PH = 'Physik';

  const poolDe = [
    // ════ Mathematik ════════════════════════════════════════════════════════
    {
      id: 'ma-sekante', topic: MA,
      text: `Welche Steigung hat die Sekante durch die Punkte
             ${tex('(1 \\mid 2)')} und ${tex('(3 \\mid 8)')}?`,
      options: [
        { text: tex('3'),
          why: `Die Steigung ist der Differenzenquotient
                ${tex('\\frac{8 - 2}{3 - 1} = \\frac{6}{2} = 3')}, also die
                Differenz der y-Werte geteilt durch die Differenz der x-Werte.` },
        { text: tex('6'),
          why: `Das ist nur die Differenz der y-Werte. Die Division durch die
                Differenz der x-Werte fehlt.` },
        { text: tex('\\tfrac{1}{3}'),
          why: `Hier sind Zähler und Nenner vertauscht.` },
        { text: tex('5'),
          why: `Das ist der Mittelwert der y-Werte, ${tex('(2 + 8)/2')}, keine
                Steigung.` },
      ],
      correct: 0,
    },
    {
      id: 'ma-ableitung', topic: MA,
      text: `Wie lautet die Ableitung von ${tex('s(t) = 0.2\\,t^2')}?`,
      options: [
        { text: tex("s'(t) = 0.4\\,t"),
          why: `Nach der Potenzregel ist die Ableitung von ${tex('t^n')} gleich
                ${tex('n\\,t^{n-1}')}. Also ist
                ${tex("s'(t) = 0.2 \\cdot 2\\,t = 0.4\\,t")}.` },
        { text: tex("s'(t) = 0.2\\,t"),
          why: `Der Exponent wurde um 1 verringert, aber nicht als Faktor 2
                vorangestellt.` },
        { text: tex("s'(t) = 0.4\\,t^2"),
          why: `Der Exponent wurde als Faktor 2 vorangestellt, aber nicht um 1
                verringert.` },
        { text: tex("s'(t) = \\tfrac{0.2}{3}\\,t^3"),
          why: `Das ist eine Stammfunktion. Hier wurde integriert statt
                abgeleitet.` },
      ],
      correct: 0,
    },
    {
      id: 'ma-grenzwert', topic: MA,
      text: `Was gibt der Grenzwert
             ${tex('\\displaystyle\\lim_{\\Delta t \\to 0} \\frac{s(t + \\Delta t) - s(t)}{\\Delta t}')}
             an?`,
      options: [
        { text: `die Steigung der Tangente an der Stelle ${tex('t')}, also die
                 Ableitung ${tex("s'(t)")}`,
          why: `Für ein festes ${tex('\\Delta t')} ist der Bruch die Steigung der
                Sekante. Geht ${tex('\\Delta t')} gegen null, wird aus der
                Sekante die Tangente.` },
        { text: `die Steigung der Sekante von ${tex('t')} bis ${tex('t + \\Delta t')}`,
          why: `Das ist der Bruch für ein festes ${tex('\\Delta t')}, also ohne
                Grenzwert.` },
        { text: `die Fläche unter dem Graphen von ${tex('s')}`,
          why: `Die Fläche liefert das Integral, nicht die Ableitung.` },
        { text: `den Mittelwert von ${tex('s')} im Intervall von ${tex('t')} bis
                 ${tex('t + \\Delta t')}`,
          why: `Der Bruch teilt die Änderung von ${tex('s')} durch
                ${tex('\\Delta t')}. Er beschreibt also, wie schnell sich
                ${tex('s')} ändert, nicht wie groß ${tex('s')} im Mittel ist.` },
      ],
      correct: 0,
    },
    {
      id: 'ma-potenzen', topic: MA,
      text: `Ein Fehler sinkt von ${tex('10^{-3}')} auf ${tex('10^{-9}')}.
             Um welchen Faktor ist er kleiner geworden?`,
      options: [
        { text: `${tex('10^{6}')}, also eine Million`,
          why: `Beim Teilen von Zehnerpotenzen werden die Exponenten subtrahiert:
                ${tex('10^{-3} / 10^{-9} = 10^{-3 - (-9)} = 10^{6}')}. Auf einer
                logarithmischen Achse sind das sechs Zehnerpotenzen.` },
        { text: tex('6'),
          why: `Das ist der Unterschied der Exponenten. Der Faktor ist
                ${tex('10')} hoch diesen Unterschied.` },
        { text: `${tex('10^{3}')}, also tausend`,
          why: `Hier wurden die Exponenten geteilt statt subtrahiert,
                ${tex('-9 / (-3) = 3')}.` },
        { text: tex('10^{12}'),
          why: `Hier wurden die Beträge der Exponenten addiert,
                ${tex('3 + 9 = 12')}, statt die Exponenten zu subtrahieren.` },
      ],
      correct: 0,
    },

    // ════ Python ════════════════════════════════════════════════════════════
    {
      id: 'py-schleife', topic: PY,
      text: `Was gibt dieser Code aus? ${block(
`werte = np.array([2.0, 5.0, 9.0])
differenzen = np.zeros(len(werte) - 1)
for k in range(len(werte) - 1):
    differenzen[k] = werte[k + 1] - werte[k]
print(differenzen)`)}`,
      options: [
        { text: code('[3. 4.]'),
          why: `${code('len(werte) - 1')} ist 2. Das Array ${code('differenzen')}
                hat also zwei Einträge, und die Schleife läuft für
                ${code('k = 0')} und ${code('k = 1')}. Sie berechnet 5 − 2 = 3
                und 9 − 5 = 4. Dieses Muster aus ${code('np.zeros')} und
                Schleife kommt in Kapitel 7.1 ständig vor.` },
        { text: code('[3. 4. 0.]'),
          why: `${code('differenzen')} hat nur ${code('len(werte) - 1')}, also
                zwei Einträge, nicht drei.` },
        { text: code('[2. 5. 9.]'),
          why: `Das ist das Array ${code('werte')}. Ausgegeben wird aber
                ${code('differenzen')}.` },
        { text: `Fehlermeldung, weil ${code('werte[k + 1]')} über das Ende des
                 Arrays hinausgreift`,
          why: `${code('range(len(werte) - 1)')} endet bei ${code('k = 1')}. Der
                größte Index ist also ${code('k + 1 = 2')}, und den gibt es in
                ${code('werte')}.` },
      ],
      correct: 0,
    },
    {
      id: 'py-funktion', topic: PY,
      text: `Was gibt dieser Code aus? ${block(
`def mittlere_steigung(werte, dauer):
    return (werte[-1] - werte[0]) / dauer

s = np.array([1.0, 1.5, 3.0])
print(mittlere_steigung(s, 4.0))`)}`,
      options: [
        { text: code('0.5'),
          why: `${code('werte[-1]')} ist der letzte Eintrag, hier 3.0, und
                ${code('werte[0]')} der erste, hier 1.0. Die Funktion gibt
                (3.0 − 1.0) / 4.0 = 0.5 mit ${code('return')} zurück.` },
        { text: code('0.75'),
          why: `Das ist 3.0 / 4.0. Dabei fehlt die Subtraktion von
                ${code('werte[0]')}.` },
        { text: code('0.375'),
          why: `Das ist (3.0 − 1.5) / 4.0. Hier wurde der vorletzte statt des
                ersten Eintrags abgezogen.` },
        { text: `Fehlermeldung, weil der Index ${code('-1')} nicht erlaubt ist`,
          why: `Negative Indizes sind in Python erlaubt und zählen von hinten.
                ${code('werte[-1]')} ist der letzte Eintrag.` },
      ],
      correct: 0,
    },
    {
      id: 'py-argmin', topic: PY,
      text: `Was gibt dieser Code aus? ${block(
`fehler = np.array([0.3, 0.02, 0.005, 0.04])
print(np.argmin(fehler))`)}`,
      options: [
        { text: code('2'),
          why: `${code('np.argmin')} gibt den Index des kleinsten Werts zurück.
                Der kleinste Wert 0.005 steht an Index 2, denn die Zählung
                beginnt bei 0.` },
        { text: code('0.005'),
          why: `Das ist der kleinste Wert selbst. Ihn liefert
                ${code('np.min(fehler)')}, nicht ${code('np.argmin')}.` },
        { text: code('0'),
          why: `An Index 0 steht der größte Wert 0.3. Den Index des größten
                Werts liefert ${code('np.argmax')}.` },
        { text: code('3'),
          why: `Das wäre richtig, wenn die Zählung bei 1 begänne. Python zählt
                aber ab 0.` },
      ],
      correct: 0,
    },

    // ════ Physik ════════════════════════════════════════════════════════════
    {
      id: 'ph-einheiten', topic: PH,
      text: `Ein FTF fährt gleichmäßig ${tex('1.6\\,\\text{m}')} in
             ${tex('2\\,\\text{s}')}. Wie schnell ist es in km/h?`,
      options: [
        { text: tex('2.88\\,\\text{km/h}'),
          why: `Die Geschwindigkeit ist
                ${tex('1.6\\,\\text{m} / 2\\,\\text{s} = 0.8\\,\\text{m/s}')}.
                Wegen ${tex('1\\,\\text{m/s} = 3.6\\,\\text{km/h}')} sind das
                ${tex('0.8 \\cdot 3.6 = 2.88\\,\\text{km/h}')}.` },
        { text: tex('0.8\\,\\text{km/h}'),
          why: `Das ist der Wert in m/s. Die Umrechnung in km/h fehlt.` },
        { text: tex('0.22\\,\\text{km/h}'),
          why: `Hier wurde durch 3.6 geteilt statt mit 3.6 multipliziert.` },
        { text: tex('3.2\\,\\text{km/h}'),
          why: `Hier wurde die Strecke mit der Zeit multipliziert statt durch
                sie geteilt, ${tex('1.6 \\cdot 2 = 3.2')}.` },
      ],
      correct: 0,
    },
    {
      id: 'ph-beschleunigung', topic: PH,
      text: `Was ist die Beschleunigung ${tex('a')}?`,
      options: [
        { text: `die Ableitung der Geschwindigkeit nach der Zeit, Einheit
                 ${tex('\\text{m/s}^2')}`,
          why: `Die Geschwindigkeit ist die Ableitung der Strecke,
                ${tex("v = s'")}, und die Beschleunigung die Ableitung der
                Geschwindigkeit, ${tex("a = v' = s''")}. Die Einheit ist
                ${tex('\\text{m/s} \\,/\\, \\text{s} = \\text{m/s}^2')}.` },
        { text: `die Ableitung der Strecke nach der Zeit, Einheit
                 ${tex('\\text{m/s}')}`,
          why: `Das ist die Geschwindigkeit ${tex('v')}.` },
        { text: `die Geschwindigkeit geteilt durch die Strecke, Einheit
                 ${tex('1/\\text{s}')}`,
          why: `Dieser Quotient beschreibt keine Änderung der Geschwindigkeit,
                und seine Einheit ${tex('1/\\text{s}')} ist keine
                Beschleunigung.` },
        { text: `die Ableitung der Geschwindigkeit nach der Strecke, Einheit
                 ${tex('1/\\text{s}')}`,
          why: `Die Beschleunigung ist die Ableitung nach der Zeit, nicht nach
                der Strecke.` },
      ],
      correct: 0,
    },
    {
      id: 'ph-anfahren', topic: PH,
      text: `Ein FTF fährt aus dem Stand ${tex('2\\,\\text{s}')} lang gleichmäßig
             mit ${tex('0.4\\,\\text{m/s}^2')} an. Welche Strecke legt es dabei
             zurück?`,
      options: [
        { text: tex('0.8\\,\\text{m}'),
          why: `Bei gleichmäßiger Beschleunigung aus dem Stand gilt
                ${tex('s = \\tfrac{a}{2}\\,t^2 = 0.2 \\cdot 4\\,\\text{m} = 0.8\\,\\text{m}')}.
                Die Geschwindigkeit am Ende ist
                ${tex('v = a\\,t = 0.8\\,\\text{m/s}')}.` },
        { text: tex('1.6\\,\\text{m}'),
          why: `Hier fehlt der Faktor ${tex('\\tfrac{1}{2}')},
                ${tex('a\\,t^2 = 0.4 \\cdot 4 = 1.6')}.` },
        { text: tex('0.4\\,\\text{m}'),
          why: `Hier fehlt das Quadrat bei ${tex('t')},
                ${tex('\\tfrac{a}{2}\\,t = 0.2 \\cdot 2 = 0.4')}.` },
        { text: tex('0.2\\,\\text{m}'),
          why: `Das ist nur ${tex('\\tfrac{a}{2}')}, der Faktor
                ${tex('t^2 = 4')} fehlt.` },
      ],
      correct: 0,
    },
  ];

  // ── Englische Fassung ──────────────────────────────────────────────────────
  const MA_EN = 'Mathematics', PY_EN = 'Python', PH_EN = 'Physics';

  const poolEn = [
    // ════ Mathematics ═══════════════════════════════════════════════════════
    {
      id: 'ma-sekante', topic: MA_EN,
      text: `What is the slope of the secant through the points
             ${tex('(1, 2)')} and ${tex('(3, 8)')}?`,
      options: [
        { text: tex('3'),
          why: `The slope is the difference quotient
                ${tex('\\frac{8 - 2}{3 - 1} = \\frac{6}{2} = 3')}, the difference
                of the y-values divided by the difference of the x-values.` },
        { text: tex('6'),
          why: `This is only the difference of the y-values. The division by the
                difference of the x-values is missing.` },
        { text: tex('\\tfrac{1}{3}'),
          why: `Numerator and denominator are swapped.` },
        { text: tex('5'),
          why: `This is the mean of the y-values, ${tex('(2 + 8)/2')}, not a
                slope.` },
      ],
      correct: 0,
    },
    {
      id: 'ma-ableitung', topic: MA_EN,
      text: `What is the derivative of ${tex('s(t) = 0.2\\,t^2')}?`,
      options: [
        { text: tex("s'(t) = 0.4\\,t"),
          why: `By the power rule, the derivative of ${tex('t^n')} is
                ${tex('n\\,t^{n-1}')}. So
                ${tex("s'(t) = 0.2 \\cdot 2\\,t = 0.4\\,t")}.` },
        { text: tex("s'(t) = 0.2\\,t"),
          why: `The exponent was reduced by 1, but not brought down as the
                factor 2.` },
        { text: tex("s'(t) = 0.4\\,t^2"),
          why: `The exponent was brought down as the factor 2, but not reduced
                by 1.` },
        { text: tex("s'(t) = \\tfrac{0.2}{3}\\,t^3"),
          why: `This is an antiderivative. It integrates instead of
                differentiating.` },
      ],
      correct: 0,
    },
    {
      id: 'ma-grenzwert', topic: MA_EN,
      text: `What does the limit
             ${tex('\\displaystyle\\lim_{\\Delta t \\to 0} \\frac{s(t + \\Delta t) - s(t)}{\\Delta t}')}
             describe?`,
      options: [
        { text: `the slope of the tangent at ${tex('t')}, i.e. the derivative
                 ${tex("s'(t)")}`,
          why: `For a fixed ${tex('\\Delta t')} the fraction is the slope of the
                secant. As ${tex('\\Delta t')} goes to zero, the secant becomes
                the tangent.` },
        { text: `the slope of the secant from ${tex('t')} to ${tex('t + \\Delta t')}`,
          why: `This is the fraction for a fixed ${tex('\\Delta t')}, i.e.
                without the limit.` },
        { text: `the area under the graph of ${tex('s')}`,
          why: `The area is given by the integral, not by the derivative.` },
        { text: `the mean value of ${tex('s')} on the interval from ${tex('t')}
                 to ${tex('t + \\Delta t')}`,
          why: `The fraction divides the change of ${tex('s')} by
                ${tex('\\Delta t')}. It describes how fast ${tex('s')} changes,
                not how large ${tex('s')} is on average.` },
      ],
      correct: 0,
    },
    {
      id: 'ma-potenzen', topic: MA_EN,
      text: `An error drops from ${tex('10^{-3}')} to ${tex('10^{-9}')}.
             By which factor has it become smaller?`,
      options: [
        { text: `${tex('10^{6}')}, i.e. one million`,
          why: `When dividing powers of ten, subtract the exponents:
                ${tex('10^{-3} / 10^{-9} = 10^{-3 - (-9)} = 10^{6}')}. On a
                logarithmic axis this is six powers of ten.` },
        { text: tex('6'),
          why: `This is the difference of the exponents. The factor is
                ${tex('10')} to the power of this difference.` },
        { text: `${tex('10^{3}')}, i.e. one thousand`,
          why: `The exponents were divided instead of subtracted,
                ${tex('-9 / (-3) = 3')}.` },
        { text: tex('10^{12}'),
          why: `The absolute values of the exponents were added,
                ${tex('3 + 9 = 12')}, instead of subtracting the exponents.` },
      ],
      correct: 0,
    },

    // ════ Python ════════════════════════════════════════════════════════════
    {
      id: 'py-schleife', topic: PY_EN,
      text: `What does this code print? ${block(
`values = np.array([2.0, 5.0, 9.0])
differences = np.zeros(len(values) - 1)
for k in range(len(values) - 1):
    differences[k] = values[k + 1] - values[k]
print(differences)`)}`,
      options: [
        { text: code('[3. 4.]'),
          why: `${code('len(values) - 1')} is 2. So ${code('differences')} has
                two entries, and the loop runs for ${code('k = 0')} and
                ${code('k = 1')}. It computes 5 − 2 = 3 and 9 − 5 = 4. This
                pattern of ${code('np.zeros')} and a loop appears again and
                again in Chapter 7.1.` },
        { text: code('[3. 4. 0.]'),
          why: `${code('differences')} has only ${code('len(values) - 1')}, i.e.
                two entries, not three.` },
        { text: code('[2. 5. 9.]'),
          why: `This is the array ${code('values')}. But the code prints
                ${code('differences')}.` },
        { text: `an error, because ${code('values[k + 1]')} reaches past the end
                 of the array`,
          why: `${code('range(len(values) - 1)')} ends at ${code('k = 1')}. So the
                largest index is ${code('k + 1 = 2')}, which exists in
                ${code('values')}.` },
      ],
      correct: 0,
    },
    {
      id: 'py-funktion', topic: PY_EN,
      text: `What does this code print? ${block(
`def mean_slope(values, duration):
    return (values[-1] - values[0]) / duration

s = np.array([1.0, 1.5, 3.0])
print(mean_slope(s, 4.0))`)}`,
      options: [
        { text: code('0.5'),
          why: `${code('values[-1]')} is the last entry, here 3.0, and
                ${code('values[0]')} the first, here 1.0. The function returns
                (3.0 − 1.0) / 4.0 = 0.5 with ${code('return')}.` },
        { text: code('0.75'),
          why: `This is 3.0 / 4.0. The subtraction of ${code('values[0]')} is
                missing.` },
        { text: code('0.375'),
          why: `This is (3.0 − 1.5) / 4.0. The second to last entry was
                subtracted instead of the first one.` },
        { text: `an error, because the index ${code('-1')} is not allowed`,
          why: `Negative indices are allowed in Python and count from the end.
                ${code('values[-1]')} is the last entry.` },
      ],
      correct: 0,
    },
    {
      id: 'py-argmin', topic: PY_EN,
      text: `What does this code print? ${block(
`errors = np.array([0.3, 0.02, 0.005, 0.04])
print(np.argmin(errors))`)}`,
      options: [
        { text: code('2'),
          why: `${code('np.argmin')} returns the index of the smallest value. The
                smallest value 0.005 is at index 2, because counting starts at
                0.` },
        { text: code('0.005'),
          why: `This is the smallest value itself. It is returned by
                ${code('np.min(errors)')}, not by ${code('np.argmin')}.` },
        { text: code('0'),
          why: `Index 0 holds the largest value 0.3. The index of the largest
                value is returned by ${code('np.argmax')}.` },
        { text: code('3'),
          why: `This would be correct if counting started at 1. But Python
                counts from 0.` },
      ],
      correct: 0,
    },

    // ════ Physics ═══════════════════════════════════════════════════════════
    {
      id: 'ph-einheiten', topic: PH_EN,
      text: `An AGV covers ${tex('1.6\\,\\text{m}')} in ${tex('2\\,\\text{s}')}
             at constant speed. How fast is it in km/h?`,
      options: [
        { text: tex('2.88\\,\\text{km/h}'),
          why: `The speed is
                ${tex('1.6\\,\\text{m} / 2\\,\\text{s} = 0.8\\,\\text{m/s}')}.
                Since ${tex('1\\,\\text{m/s} = 3.6\\,\\text{km/h}')}, this is
                ${tex('0.8 \\cdot 3.6 = 2.88\\,\\text{km/h}')}.` },
        { text: tex('0.8\\,\\text{km/h}'),
          why: `This is the value in m/s. The conversion to km/h is missing.` },
        { text: tex('0.22\\,\\text{km/h}'),
          why: `This divides by 3.6 instead of multiplying by 3.6.` },
        { text: tex('3.2\\,\\text{km/h}'),
          why: `This multiplies the distance by the time instead of dividing,
                ${tex('1.6 \\cdot 2 = 3.2')}.` },
      ],
      correct: 0,
    },
    {
      id: 'ph-beschleunigung', topic: PH_EN,
      text: `What is the acceleration ${tex('a')}?`,
      options: [
        { text: `the derivative of the velocity with respect to time, unit
                 ${tex('\\text{m/s}^2')}`,
          why: `The velocity is the derivative of the distance, ${tex("v = s'")},
                and the acceleration is the derivative of the velocity,
                ${tex("a = v' = s''")}. The unit is
                ${tex('\\text{m/s} \\,/\\, \\text{s} = \\text{m/s}^2')}.` },
        { text: `the derivative of the distance with respect to time, unit
                 ${tex('\\text{m/s}')}`,
          why: `This is the velocity ${tex('v')}.` },
        { text: `the velocity divided by the distance, unit ${tex('1/\\text{s}')}`,
          why: `This quotient does not describe a change of velocity, and its
                unit ${tex('1/\\text{s}')} is not an acceleration.` },
        { text: `the derivative of the velocity with respect to distance, unit
                 ${tex('1/\\text{s}')}`,
          why: `The acceleration is the derivative with respect to time, not to
                distance.` },
      ],
      correct: 0,
    },
    {
      id: 'ph-anfahren', topic: PH_EN,
      text: `An AGV starts from rest and accelerates uniformly at
             ${tex('0.4\\,\\text{m/s}^2')} for ${tex('2\\,\\text{s}')}. What
             distance does it cover?`,
      options: [
        { text: tex('0.8\\,\\text{m}'),
          why: `For uniform acceleration from rest,
                ${tex('s = \\tfrac{a}{2}\\,t^2 = 0.2 \\cdot 4\\,\\text{m} = 0.8\\,\\text{m}')}.
                The final speed is ${tex('v = a\\,t = 0.8\\,\\text{m/s}')}.` },
        { text: tex('1.6\\,\\text{m}'),
          why: `The factor ${tex('\\tfrac{1}{2}')} is missing,
                ${tex('a\\,t^2 = 0.4 \\cdot 4 = 1.6')}.` },
        { text: tex('0.4\\,\\text{m}'),
          why: `The square of ${tex('t')} is missing,
                ${tex('\\tfrac{a}{2}\\,t = 0.2 \\cdot 2 = 0.4')}.` },
        { text: tex('0.2\\,\\text{m}'),
          why: `This is only ${tex('\\tfrac{a}{2}')}, the factor
                ${tex('t^2 = 4')} is missing.` },
      ],
      correct: 0,
    },
  ];

  // ── Texte der Auswertung ───────────────────────────────────────────────────
  const SETTINGS = {
    de: {
      title: 'Vorkenntnis-Check zu Kapitel 7',
      pool: poolDe,
      messages: {
        perfect: 'Alles richtig. Sie sind gut auf Kapitel 7 vorbereitet.',
        good: 'Gut vorbereitet! Was Sie zu den Themen mit Fehlern wiederholen können, steht unter dem Quiz.',
        weak: 'Wiederholen Sie die Themen mit Fehlern mit den Hinweisen unter dem Quiz. Beim nächsten Durchlauf kommen die falsch beantworteten Fragen zuerst.',
      },
    },
    en: {
      title: 'Prerequisite check for Chapter 7',
      pool: poolEn,
      messages: {
        perfect: 'All correct. You are well prepared for Chapter 7.',
        good: 'Well prepared! What to review for the topics with mistakes is listed below the quiz.',
        weak: 'Review the topics with mistakes using the notes below the quiz. In the next round, the questions you got wrong come first.',
      },
    },
  };
  const S = SETTINGS[lang];
</script>

<Quiz title={S.title} pool={S.pool} nShow={S.pool.length} {lang}
      messages={S.messages} storageKey={`anm-chap07-sec00-${lang}`} />
