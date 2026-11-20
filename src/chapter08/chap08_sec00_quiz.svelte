<!--
  ═══════════════════════════════════════════════════════════════════════════════
  chap08_sec00_quiz.svelte  –  Vorkenntnis-Check zu Kapitel 8 (DE/EN)
  ─ Nur Fragen-Daten, Design und Ablauf liegen in shared/Quiz.svelte
  ─ Sprache über URL-Parameter: ?lang=en, sonst Deutsch
  ─ 3 Themen (4 + 4 + 2 Fragen), alle Fragen pro Durchlauf
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
      id: 'ma-minimum', topic: MA,
      text: `Die Ableitung ${tex("f'")} einer Funktion ist bei ${tex('x = 3')}
             null. Links davon ist sie negativ, rechts davon positiv. Was liegt
             bei ${tex('x = 3')}?`,
      options: [
        { text: `ein Minimum von ${tex('f')}`,
          why: `Links von ${tex('x = 3')} fällt ${tex('f')}, weil die Ableitung
                negativ ist. Rechts davon steigt ${tex('f')}. Dazwischen liegt
                der tiefste Punkt, also ein Minimum.` },
        { text: `ein Maximum von ${tex('f')}`,
          why: `Bei einem Maximum steigt ${tex('f')} erst und fällt dann. Die
                Ableitung wäre also links positiv und rechts negativ.` },
        { text: `eine Nullstelle von ${tex('f')}`,
          why: `Null ist hier die Ableitung, nicht der Funktionswert. Über
                ${tex('f(3)')} selbst sagt die Ableitung nichts.` },
        { text: `ein Wendepunkt von ${tex('f')}`,
          why: `Am Wendepunkt ändert sich die Krümmung. Hier wechselt aber die
                Ableitung ihr Vorzeichen, die Funktion geht also von Fallen in
                Steigen über.` },
      ],
      correct: 0,
    },
    {
      id: 'ma-kehrwert', topic: MA,
      text: `Wie lautet die Ableitung von
             ${tex('f(x) = \\dfrac{8}{x} + 2x')}?`,
      options: [
        { text: tex("f'(x) = -\\dfrac{8}{x^2} + 2"),
          why: `Wir schreiben ${tex('\\frac{8}{x} = 8\\,x^{-1}')}. Nach der
                Potenzregel ist die Ableitung
                ${tex('8 \\cdot (-1)\\,x^{-2} = -\\frac{8}{x^2}')}. Die Ableitung
                von ${tex('2x')} ist ${tex('2')}.` },
        { text: tex("f'(x) = \\dfrac{8}{x^2} + 2"),
          why: `Hier fehlt das Minuszeichen. Der Exponent ${tex('-1')} wird als
                Faktor vorangestellt.` },
        { text: tex("f'(x) = -\\dfrac{8}{x} + 2"),
          why: `Der Exponent ${tex('-1')} wurde als Faktor vorangestellt, aber
                nicht um 1 auf ${tex('-2')} verringert.` },
        { text: tex("f'(x) = 8 \\ln x + x^2"),
          why: `Das ist eine Stammfunktion. Hier wurde integriert statt
                abgeleitet.` },
      ],
      correct: 0,
    },
    {
      id: 'ma-logarithmus', topic: MA,
      text: `Für welches ${tex('t')} gilt ${tex('e^{-0.05\\,t} = 0.25')}?`,
      options: [
        { text: tex('t = \\dfrac{\\ln 4}{0.05} \\approx 27.7'),
          why: `Der natürliche Logarithmus auf beiden Seiten ergibt
                ${tex('-0.05\\,t = \\ln 0.25 = -\\ln 4')}. Teilen durch
                ${tex('-0.05')} liefert ${tex('t = \\ln 4 / 0.05 \\approx 27.7')}.` },
        { text: tex('t = \\dfrac{0.25}{0.05} = 5'),
          why: `Hier wurde die e-Funktion übergangen. Um an den Exponenten zu
                kommen, brauchen wir den natürlichen Logarithmus.` },
        { text: tex('t = 0.05 \\cdot \\ln 4 \\approx 0.069'),
          why: `Hier wurde mit ${tex('0.05')} multipliziert statt durch
                ${tex('0.05')} geteilt.` },
        { text: tex('t = -\\dfrac{\\ln 4}{0.05} \\approx -27.7'),
          why: `${tex('\\ln 0.25')} ist negativ, ebenso ${tex('-0.05')}. Beim
                Teilen heben sich die beiden Minuszeichen auf.` },
      ],
      correct: 0,
    },
    {
      id: 'ma-abstand', topic: MA,
      text: `Wie weit liegen die Punkte ${tex('(1 \\mid 2)')} und
             ${tex('(4 \\mid 6)')} voneinander entfernt?`,
      options: [
        { text: tex('5'),
          why: `Die Punkte liegen ${tex('3')} in x-Richtung und ${tex('4')} in
                y-Richtung auseinander. Nach dem Satz des Pythagoras ist der
                Abstand ${tex('\\sqrt{3^2 + 4^2} = \\sqrt{25} = 5')}.` },
        { text: tex('7'),
          why: `Das ist ${tex('3 + 4')}, der Weg erst in x- und dann in
                y-Richtung. Die Luftlinie ist kürzer und folgt aus dem Satz des
                Pythagoras.` },
        { text: tex('25'),
          why: `Das ist das Quadrat des Abstands. Die Wurzel fehlt.` },
        { text: tex('\\sqrt{89} \\approx 9.43'),
          why: `Hier wurden die Koordinaten addiert statt subtrahiert,
                ${tex('\\sqrt{5^2 + 8^2}')}.` },
      ],
      correct: 0,
    },

    // ════ Python ════════════════════════════════════════════════════════════
    {
      id: 'py-zentral', topic: PY,
      text: `Was gibt dieser Code aus? ${block(
`def zentral(funktion, stelle, dx):
    oben = funktion(stelle + dx)
    unten = funktion(stelle - dx)
    return (oben - unten) / (2 * dx)

def f(x):
    return x**2

print(zentral(f, 3.0, 0.5))`)}`,
      options: [
        { text: code('6.0'),
          why: `${code('zentral()')} ruft die übergebene Funktion bei 3.5 und
                2.5 auf: (12.25 − 6.25) / (2 · 0.5) = 6.0. Das ist der zentrale
                Differenzenquotient aus Kapitel 7.2. Bei einer Parabel trifft er
                die Ableitung ${tex("f'(3) = 6")} sogar exakt.` },
        { text: code('12.0'),
          why: `Das ist 6.0 / 0.5. Der Code teilt aber durch
                ${code('2 * dx')}, also durch 1.0.` },
        { text: code('9.0'),
          why: `Das ist der Funktionswert ${code('f(3.0)')}, nicht die
                Steigung.` },
        { text: `Fehlermeldung, weil eine Funktion nicht als Argument übergeben
                 werden kann`,
          why: `In Python lassen sich Funktionen wie Zahlen übergeben. Ohne
                Klammern bezeichnet ${code('f')} die Funktion selbst, genau wie in
                Kapitel 7.2.` },
      ],
      correct: 0,
    },
    {
      id: 'py-mittelwert', topic: PY,
      text: `Was gibt dieser Code aus? ${block(
`messung = np.array([2.0, 4.0, 7.0])
modell = np.array([1.0, 4.0, 9.0])
print(np.mean((messung - modell)**2))`)}`,
      options: [
        { text: code('1.6666666666666667'),
          why: `NumPy rechnet elementweise. Die Differenzen sind 1, 0 und −2,
                ihre Quadrate 1, 0 und 4. ${code('np.mean')} bildet den
                Mittelwert 5 / 3 ≈ 1.667.` },
        { text: code('-0.3333333333333333'),
          why: `Das ist der Mittelwert der Differenzen ohne Quadrieren. Der
                Code quadriert aber jede Differenz mit ${code('**2')}, bevor er
                mittelt.` },
        { text: code('5.0'),
          why: `Das ist die Summe der Quadrate, also ${code('np.sum')}.
                ${code('np.mean')} teilt noch durch die Anzahl 3.` },
        { text: code('0.1111111111111111'),
          why: `Hier wurde erst gemittelt und dann quadriert,
                (−1/3)² ≈ 0.111. Die Klammern im Code legen die umgekehrte
                Reihenfolge fest.` },
      ],
      correct: 0,
    },
    {
      id: 'py-spalte', topic: PY,
      text: `Was gibt dieser Code aus? ${block(
`werke = np.array([[6.0, 24.0],
                  [16.0, 30.0],
                  [20.0, 12.0]])
print(werke[:, 0])`)}`,
      options: [
        { text: code('[ 6. 16. 20.]'),
          why: `Der Doppelpunkt steht für alle Zeilen, die ${code('0')} für die
                Spalte mit Index 0. Wir erhalten also die erste Spalte, hier die
                x-Koordinaten aller Werke.` },
        { text: code('[ 6. 24.]'),
          why: `Das ist die erste Zeile, also ${code('werke[0]')}. Bei
                ${code('werke[:, 0]')} steht der Zeilenindex vor dem Komma, und
                der Doppelpunkt wählt alle Zeilen aus.` },
        { text: code('[24. 30. 12.]'),
          why: `Das ist die Spalte mit Index 1, also ${code('werke[:, 1]')}.
                Python zählt ab 0.` },
        { text: `Fehlermeldung, weil ein Array nur einen Index haben kann`,
          why: `Ein zweidimensionales Array hat zwei Indizes, zuerst die Zeile,
                dann die Spalte.` },
      ],
      correct: 0,
    },
    {
      id: 'py-linspace', topic: PY,
      text: `Was gibt dieser Code aus? ${block(
`x = np.linspace(0, 2, 5)
y = np.zeros(len(x))
for k in range(len(x)):
    y[k] = x[k]**2
print(y)`)}`,
      options: [
        { text: code('[0.   0.25 1.   2.25 4.  ]'),
          why: `${code('np.linspace(0, 2, 5)')} liefert fünf gleichmäßig
                verteilte Werte von 0 bis einschließlich 2, also 0, 0.5, 1,
                1.5 und 2. Die Schleife füllt ${code('y')} mit ihren
                Quadraten.` },
        { text: code('[ 0.  1.  4.  9. 16.]'),
          why: `Das wären die Quadrate von 0 bis 4 wie bei ${code('range(5)')}.
                Bei ${code('np.linspace')} ist die dritte Zahl aber die Anzahl
                der Werte, nicht die Schrittweite.` },
        { text: code('[0.   0.16 0.64 1.44 2.56]'),
          why: `Hier wurde die Schrittweite 2 / 5 = 0.4 angenommen. Weil
                ${code('np.linspace')} beide Enden einschließt, sind es nur vier
                Abstände, die Schrittweite ist also 2 / 4 = 0.5.` },
        { text: code('[0. 0. 0. 0. 0.]'),
          why: `${code('np.zeros')} legt das Array nur an. Die Schleife
                überschreibt danach jeden Eintrag.` },
      ],
      correct: 0,
    },

    // ════ Physik ════════════════════════════════════════════════════════════
    {
      id: 'ph-kennlinie', topic: PH,
      text: `Ein Kraftsensor hat die Kennlinie ${tex('U = m \\cdot F + b')} mit
             ${tex('m = 2.0\\,\\text{V/kN}')} und ${tex('b = 0.1\\,\\text{V}')}.
             Er zeigt ${tex('2.5\\,\\text{V}')} an. Welche Kraft wirkt?`,
      options: [
        { text: tex('1.2\\,\\text{kN}'),
          why: `Nach ${tex('F')} aufgelöst gilt
                ${tex('F = (U - b)/m = (2.5 - 0.1)/2.0\\,\\text{kN} = 1.2\\,\\text{kN}')}.` },
        { text: tex('1.25\\,\\text{kN}'),
          why: `Hier fehlt der Nullpunkt ${tex('b')}. Er muss zuerst von der
                Spannung abgezogen werden.` },
        { text: tex('1.3\\,\\text{kN}'),
          why: `Hier wurde ${tex('b')} addiert statt abgezogen.` },
        { text: tex('5.1\\,\\text{kN}'),
          why: `Hier wurde die Spannung in die Kennlinie eingesetzt,
                ${tex('2.0 \\cdot 2.5 + 0.1')}, statt sie nach ${tex('F')}
                aufzulösen.` },
      ],
      correct: 0,
    },
    {
      id: 'ph-abkuehlung', topic: PH,
      text: `Ein ${tex('100\\,°\\text{C}')} heißes Teil kühlt in einer
             ${tex('20\\,°\\text{C}')} warmen Halle ab. Wie verläuft seine
             Temperatur?`,
      options: [
        { text: `Sie fällt zuerst schnell, dann immer langsamer und nähert sich
                 ${tex('20\\,°\\text{C}')}.`,
          why: `Das Teil gibt umso mehr Wärme ab, je wärmer es als die Halle
                ist. Mit kleiner werdendem Temperaturunterschied kühlt es deshalb
                immer langsamer ab und erreicht ${tex('20\\,°\\text{C}')} erst
                nach sehr langer Zeit.` },
        { text: `Sie fällt gleichmäßig, bis sie ${tex('20\\,°\\text{C}')}
                 erreicht.`,
          why: `Gleichmäßig wäre die Abkühlung nur, wenn das Teil immer gleich
                viel Wärme abgäbe. Die Wärmeabgabe hängt aber vom
                Temperaturunterschied zur Halle ab.` },
        { text: `Sie fällt gleichmäßig und sinkt auch unter
                 ${tex('20\\,°\\text{C}')}.`,
          why: `Ohne Kühlung kann das Teil nicht kälter werden als seine
                Umgebung.` },
        { text: `Sie fällt zuerst langsam, dann immer schneller.`,
          why: `Am Anfang ist der Temperaturunterschied am größten, also auch
                die Wärmeabgabe. Die Abkühlung ist deshalb zu Beginn am
                schnellsten.` },
      ],
      correct: 0,
    },
  ];

  // ── Englische Fassung ──────────────────────────────────────────────────────
  const MA_EN = 'Mathematics', PY_EN = 'Python', PH_EN = 'Physics';

  const poolEn = [
    // ════ Mathematics ═══════════════════════════════════════════════════════
    {
      id: 'ma-minimum', topic: MA_EN,
      text: `The derivative ${tex("f'")} of a function is zero at
             ${tex('x = 3')}. To the left it is negative, to the right positive.
             What is at ${tex('x = 3')}?`,
      options: [
        { text: `a minimum of ${tex('f')}`,
          why: `To the left of ${tex('x = 3')}, ${tex('f')} decreases because the
                derivative is negative. To the right, ${tex('f')} increases. In
                between lies the lowest point, a minimum.` },
        { text: `a maximum of ${tex('f')}`,
          why: `At a maximum, ${tex('f')} first increases and then decreases. The
                derivative would be positive to the left and negative to the
                right.` },
        { text: `a zero of ${tex('f')}`,
          why: `It is the derivative that is zero, not the function value. The
                derivative says nothing about ${tex('f(3)')} itself.` },
        { text: `an inflection point of ${tex('f')}`,
          why: `At an inflection point the curvature changes. Here, however, the
                derivative changes its sign, so the function goes from
                decreasing to increasing.` },
      ],
      correct: 0,
    },
    {
      id: 'ma-kehrwert', topic: MA_EN,
      text: `What is the derivative of ${tex('f(x) = \\dfrac{8}{x} + 2x')}?`,
      options: [
        { text: tex("f'(x) = -\\dfrac{8}{x^2} + 2"),
          why: `We write ${tex('\\frac{8}{x} = 8\\,x^{-1}')}. By the power rule,
                the derivative is
                ${tex('8 \\cdot (-1)\\,x^{-2} = -\\frac{8}{x^2}')}. The derivative
                of ${tex('2x')} is ${tex('2')}.` },
        { text: tex("f'(x) = \\dfrac{8}{x^2} + 2"),
          why: `The minus sign is missing. The exponent ${tex('-1')} is brought
                down as a factor.` },
        { text: tex("f'(x) = -\\dfrac{8}{x} + 2"),
          why: `The exponent ${tex('-1')} was brought down as a factor, but not
                reduced by 1 to ${tex('-2')}.` },
        { text: tex("f'(x) = 8 \\ln x + x^2"),
          why: `This is an antiderivative. It integrates instead of
                differentiating.` },
      ],
      correct: 0,
    },
    {
      id: 'ma-logarithmus', topic: MA_EN,
      text: `For which ${tex('t')} does ${tex('e^{-0.05\\,t} = 0.25')} hold?`,
      options: [
        { text: tex('t = \\dfrac{\\ln 4}{0.05} \\approx 27.7'),
          why: `Taking the natural logarithm of both sides gives
                ${tex('-0.05\\,t = \\ln 0.25 = -\\ln 4')}. Dividing by
                ${tex('-0.05')} yields ${tex('t = \\ln 4 / 0.05 \\approx 27.7')}.` },
        { text: tex('t = \\dfrac{0.25}{0.05} = 5'),
          why: `This ignores the exponential function. To get at the exponent,
                we need the natural logarithm.` },
        { text: tex('t = 0.05 \\cdot \\ln 4 \\approx 0.069'),
          why: `This multiplies by ${tex('0.05')} instead of dividing by
                ${tex('0.05')}.` },
        { text: tex('t = -\\dfrac{\\ln 4}{0.05} \\approx -27.7'),
          why: `${tex('\\ln 0.25')} is negative, and so is ${tex('-0.05')}. When
                dividing, the two minus signs cancel.` },
      ],
      correct: 0,
    },
    {
      id: 'ma-abstand', topic: MA_EN,
      text: `How far apart are the points ${tex('(1, 2)')} and
             ${tex('(4, 6)')}?`,
      options: [
        { text: tex('5'),
          why: `The points are ${tex('3')} apart in the x-direction and
                ${tex('4')} in the y-direction. By the Pythagorean theorem, the
                distance is ${tex('\\sqrt{3^2 + 4^2} = \\sqrt{25} = 5')}.` },
        { text: tex('7'),
          why: `This is ${tex('3 + 4')}, the path first in the x- and then in
                the y-direction. The straight-line distance is shorter and
                follows from the Pythagorean theorem.` },
        { text: tex('25'),
          why: `This is the square of the distance. The square root is
                missing.` },
        { text: tex('\\sqrt{89} \\approx 9.43'),
          why: `This adds the coordinates instead of subtracting them,
                ${tex('\\sqrt{5^2 + 8^2}')}.` },
      ],
      correct: 0,
    },

    // ════ Python ════════════════════════════════════════════════════════════
    {
      id: 'py-zentral', topic: PY_EN,
      text: `What does this code print? ${block(
`def central(function, point, dx):
    upper = function(point + dx)
    lower = function(point - dx)
    return (upper - lower) / (2 * dx)

def f(x):
    return x**2

print(central(f, 3.0, 0.5))`)}`,
      options: [
        { text: code('6.0'),
          why: `${code('central()')} calls the function it receives at 3.5 and
                2.5: (12.25 − 6.25) / (2 · 0.5) = 6.0. This is the central
                difference quotient from Chapter 7.2. For a parabola it even
                matches the derivative ${tex("f'(3) = 6")} exactly.` },
        { text: code('12.0'),
          why: `This is 6.0 / 0.5. But the code divides by ${code('2 * dx')},
                i.e. by 1.0.` },
        { text: code('9.0'),
          why: `This is the function value ${code('f(3.0)')}, not the slope.` },
        { text: `an error, because a function cannot be passed as an argument`,
          why: `In Python, functions can be passed just like numbers. Without
                parentheses, ${code('f')} refers to the function itself, exactly
                as in Chapter 7.2.` },
      ],
      correct: 0,
    },
    {
      id: 'py-mittelwert', topic: PY_EN,
      text: `What does this code print? ${block(
`measured = np.array([2.0, 4.0, 7.0])
model = np.array([1.0, 4.0, 9.0])
print(np.mean((measured - model)**2))`)}`,
      options: [
        { text: code('1.6666666666666667'),
          why: `NumPy computes element by element. The differences are 1, 0 and
                −2, their squares 1, 0 and 4. ${code('np.mean')} returns the mean
                5 / 3 ≈ 1.667.` },
        { text: code('-0.3333333333333333'),
          why: `This is the mean of the differences without squaring. But the
                code squares each difference with ${code('**2')} before taking
                the mean.` },
        { text: code('5.0'),
          why: `This is the sum of the squares, i.e. ${code('np.sum')}.
                ${code('np.mean')} also divides by the number of values, 3.` },
        { text: code('0.1111111111111111'),
          why: `This takes the mean first and squares afterwards,
                (−1/3)² ≈ 0.111. The parentheses in the code set the opposite
                order.` },
      ],
      correct: 0,
    },
    {
      id: 'py-spalte', topic: PY_EN,
      text: `What does this code print? ${block(
`plants = np.array([[6.0, 24.0],
                   [16.0, 30.0],
                   [20.0, 12.0]])
print(plants[:, 0])`)}`,
      options: [
        { text: code('[ 6. 16. 20.]'),
          why: `The colon stands for all rows, the ${code('0')} for the column
                with index 0. So we get the first column, here the
                x-coordinates of all plants.` },
        { text: code('[ 6. 24.]'),
          why: `This is the first row, i.e. ${code('plants[0]')}. In
                ${code('plants[:, 0]')} the row index comes before the comma,
                and the colon selects all rows.` },
        { text: code('[24. 30. 12.]'),
          why: `This is the column with index 1, i.e. ${code('plants[:, 1]')}.
                Python counts from 0.` },
        { text: `an error, because an array can only have one index`,
          why: `A two-dimensional array has two indices, first the row, then the
                column.` },
      ],
      correct: 0,
    },
    {
      id: 'py-linspace', topic: PY_EN,
      text: `What does this code print? ${block(
`x = np.linspace(0, 2, 5)
y = np.zeros(len(x))
for k in range(len(x)):
    y[k] = x[k]**2
print(y)`)}`,
      options: [
        { text: code('[0.   0.25 1.   2.25 4.  ]'),
          why: `${code('np.linspace(0, 2, 5)')} returns five evenly spaced values
                from 0 up to and including 2, i.e. 0, 0.5, 1, 1.5 and 2. The
                loop fills ${code('y')} with their squares.` },
        { text: code('[ 0.  1.  4.  9. 16.]'),
          why: `These would be the squares of 0 to 4 as with ${code('range(5)')}.
                For ${code('np.linspace')}, however, the third number is the
                number of values, not the step size.` },
        { text: code('[0.   0.16 0.64 1.44 2.56]'),
          why: `This assumes the step size 2 / 5 = 0.4. Since
                ${code('np.linspace')} includes both ends, there are only four
                gaps, so the step size is 2 / 4 = 0.5.` },
        { text: code('[0. 0. 0. 0. 0.]'),
          why: `${code('np.zeros')} only creates the array. The loop then
                overwrites every entry.` },
      ],
      correct: 0,
    },

    // ════ Physics ═══════════════════════════════════════════════════════════
    {
      id: 'ph-kennlinie', topic: PH_EN,
      text: `A force sensor has the characteristic ${tex('U = m \\cdot F + b')}
             with ${tex('m = 2.0\\,\\text{V/kN}')} and
             ${tex('b = 0.1\\,\\text{V}')}. It reads ${tex('2.5\\,\\text{V}')}.
             Which force acts on it?`,
      options: [
        { text: tex('1.2\\,\\text{kN}'),
          why: `Solved for ${tex('F')}:
                ${tex('F = (U - b)/m = (2.5 - 0.1)/2.0\\,\\text{kN} = 1.2\\,\\text{kN}')}.` },
        { text: tex('1.25\\,\\text{kN}'),
          why: `The offset ${tex('b')} is missing. It has to be subtracted from
                the voltage first.` },
        { text: tex('1.3\\,\\text{kN}'),
          why: `This adds ${tex('b')} instead of subtracting it.` },
        { text: tex('5.1\\,\\text{kN}'),
          why: `This inserts the voltage into the characteristic,
                ${tex('2.0 \\cdot 2.5 + 0.1')}, instead of solving for
                ${tex('F')}.` },
      ],
      correct: 0,
    },
    {
      id: 'ph-abkuehlung', topic: PH_EN,
      text: `A part at ${tex('100\\,°\\text{C}')} cools down in a hall at
             ${tex('20\\,°\\text{C}')}. How does its temperature change?`,
      options: [
        { text: `It drops quickly at first, then more and more slowly, and
                 approaches ${tex('20\\,°\\text{C}')}.`,
          why: `The part gives off more heat the warmer it is compared with the
                hall. As the temperature difference shrinks, it cools more and
                more slowly and reaches ${tex('20\\,°\\text{C}')} only after a
                very long time.` },
        { text: `It drops at a constant rate until it reaches
                 ${tex('20\\,°\\text{C}')}.`,
          why: `A constant rate would require the part to give off the same
                amount of heat all the time. But the heat loss depends on the
                temperature difference to the hall.` },
        { text: `It drops at a constant rate and falls below
                 ${tex('20\\,°\\text{C}')}.`,
          why: `Without cooling, the part cannot become colder than its
                surroundings.` },
        { text: `It drops slowly at first, then faster and faster.`,
          why: `At the start the temperature difference is largest, and so is
                the heat loss. The part therefore cools fastest at the
                beginning.` },
      ],
      correct: 0,
    },
  ];

  // ── Texte der Auswertung ───────────────────────────────────────────────────
  const SETTINGS = {
    de: {
      title: 'Vorkenntnis-Check zu Kapitel 8',
      pool: poolDe,
      messages: {
        perfect: 'Alles richtig. Sie sind gut auf Kapitel 8 vorbereitet.',
        good: 'Gut vorbereitet! Was Sie zu den Themen mit Fehlern wiederholen können, steht unter dem Quiz.',
        weak: 'Wiederholen Sie die Themen mit Fehlern mit den Hinweisen unter dem Quiz. Beim nächsten Durchlauf kommen die falsch beantworteten Fragen zuerst.',
      },
    },
    en: {
      title: 'Prerequisite check for Chapter 8',
      pool: poolEn,
      messages: {
        perfect: 'All correct. You are well prepared for Chapter 8.',
        good: 'Well prepared! What to review for the topics with mistakes is listed below the quiz.',
        weak: 'Review the topics with mistakes using the notes below the quiz. In the next round, the questions you got wrong come first.',
      },
    },
  };
  const S = SETTINGS[lang];
</script>

<Quiz title={S.title} pool={S.pool} nShow={S.pool.length} {lang}
      messages={S.messages} storageKey={`anm-chap08-sec00-${lang}`} />
