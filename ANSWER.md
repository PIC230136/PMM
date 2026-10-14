# Antwort — R im PMM-Unterricht der 3. Klasse (KM5 & KM6)

**Frage:** Welche Inhalte der 3. Klasse lassen sich mit R effizienter lösen?

## Kurzfassung

Fast der **gesamte statistische Teil** der 3. Klasse (KM5 und KM6) wird in R
deutlich effizienter gelöst als „händisch“: Verteilungsrechnungen, Lage- und
Streumaße, Konfidenzintervalle, Ausreißeranalyse, Visualisierung und
Lebensdauerverteilungen. R ersetzt **Tabellenwerke, Formelumstellungen und
Handzeichnungen** durch reproduzierbare Funktionsaufrufe. Nicht ersetzt wird
das **Verständnis der Konzepte** (Was ist eine Dichte? Warum `n−1`? Welche
Annahme gilt?) — das bleibt Denkarbeit. Der nicht-quantitative Teil des
Prozessmanagements (KM3/KM4) spielt für R keine Rolle.

---

## 1. Warum R überhaupt effizienter ist

| Händisch (klassisch) | Mit R |
|---|---|
| Werte aus **Verteilungstabellen** (z-, t-, χ²-Tabelle) ablesen | `pnorm`, `qt`, `qchisq` — exakter Wert, keine Interpolation |
| **Formeln** von Hand umstellen und einsetzen | eine Zeile Code, z. B. `t.test(x)$conf.int` |
| viele Werte einzeln rechnen (Fehlerquelle!) | **Vektorisierung**: ganzer Datensatz auf einmal |
| Diagramme per Hand zeichnen | `ggplot2` — jederzeit reproduzierbar |
| Rechenweg nicht nachvollziehbar | **Skript = Dokumentation**, jederzeit wiederholbar |

**Kernaussage:** R verschiebt die Arbeit vom *Rechnen* zum *Interpretieren*.

---

## 2. KM5 — Wahrscheinlichkeitsverteilungen

### 2.1 Übersicht Thema → R-Funktion

| Thema der 3. Klasse | Händisch | R | Vorteil |
|---|---|---|---|
| Binomialverteilung | Bernoulli-Formel, Fakultäten, Tabellen | `dbinom` / `pbinom` / `qbinom` | keine Fakultätsfehler, beliebige `n` |
| Hypergeometrisch | Kombinatorik von Hand | `dhyper` / `phyper` | große Grundgesamtheiten trivial |
| Poisson | Reihenentwicklung | `dpois` / `ppois` | schnelle Näherungen prüfbar |
| Normalverteilung / z-Transformation | z-Tabelle ablesen | `pnorm` / `qnorm` | exakte Wahrscheinlichkeiten, Umkehrung |
| Exponentialverteilung | `e^(−λt)` von Hand | `pexp` / `qexp` | Ausfallwahrscheinlichkeiten über Zeit |
| Simulationsgegenprobe | nicht möglich | `rbinom`, `rnorm`, `rpois` | Verteilung „erleben“, Monte-Carlo |

### 2.2 Beispiele

```r
# --- Verteilungen: Tabellen ersetzen ------------------------------------
# Normalverteilung / Standardisierung
pnorm(1.96)                 # 0.9750021   -> z-Tabelle: 97,5 %
qnorm(0.975)                # 1.959964    -> Umkehrung (kritischer Wert)

# Binomialverteilung
dbinom(3, size = 10, prob = 0.5)    # 0.1171875  P(genau 3)
pbinom(3, size = 10, prob = 0.5)    # 0.171875   P(X <= 3)
qbinom(0.95, size = 10, prob = 0.5) # 8          Quantil

# Hypergeometrisch: 3 defekte in 20 Teilen, Stichprobe 5
dhyper(1, m = 3, n = 17, k = 5)     # P(genau 1 defektes Teil)

# Poisson
dpois(2, lambda = 3)                # P(X = 2)
```

---

## 3. KM5 — Lage- und Streumaße, Parameter vs. Schätzwerte

| Thema | Händisch | R | Vorteil |
|---|---|---|---|
| Mittelwert, Median, Modus | Summenformel, Sortieren | `mean`, `median` | sofort, auch für große Daten |
| Varianz / Standardabweichung | quadrierte Abweichungen | `var`, `sd` | korrektes `n−1` (Bessel) automatisch |
| Spannweite, Quartile, IQR | Rangliste von Hand | `range`, `quantile`, `IQR` | konsistente Definitionen |
| Schiefe / Wölbung | lange Formeln | `e1071::skewness`/`kurtosis` | kein Formelfehler |

```r
# --- Lage- und Streumaße -------------------------------------------------
x <- c(12, 15, 11, 18, 60, 14, 13, 16)
mean(x)                          # 19.875  (vom Ausreißer 60 hochgezogen)
median(x)                        # 14.5    (robuster Lagewert)
sd(x)                            # Streuung um den Mittelwert
quantile(x, c(0.25, 0.5, 0.75))  # Q1, Median, Q3
```

> **Effizienzgewinn:** Der Vergleich `mean` vs. `median` zeigt den Einfluss von
> Ausreißern — händisch wäre das bei mehreren Datensätzen mühsam, in R ein
> Einzeiler.

---

## 4. KM6 — Konfidenzintervalle

| Thema | Händisch | R | Vorteil |
|---|---|---|---|
| Standardfehler `SE = σ/√n` | einsetzen | `sd(x)/sqrt(length(x))` | schnell variierbar |
| KI für μ bei **bekanntem** σ | `z = 1,96` aus Tabelle | `qnorm(0.975)` | exakter kritischer Wert |
| KI für μ bei **unbekanntem** σ | **t-Tabelle** + `df` | `t.test(x)$conf.int` | Intervall in einem Befehl |
| KI für σ² | **χ²-Tabelle** | `qchisq(0.975, df)` | beide Grenzen sofort |
| Breite / Stichprobenumfang | Formel umstellen | mit `n` experimentieren | „Was-wäre-wenn“ in Sekunden |

```r
# --- Konfidenzintervall für den Mittelwert (σ unbekannt) -----------------
set.seed(42)
m <- rnorm(25, mean = 100, sd = 15)
t.test(m, conf.level = 0.95)$conf.int   # untere und obere Grenze
```

---

## 5. KM6 — Prüfergebnisse darstellen und Ausreißer

| Thema | Händisch | R | Vorteil |
|---|---|---|---|
| Histogramm / Verteilung | Balken zeichnen | `geom_histogram()` | Bins frei wählbar |
| Boxplot | Quartile + Whisker zeichnen | `geom_boxplot()` | Ausreißer automatisch markiert |
| QQ-Plot (Normalitätsprüfung) | praktisch unmöglich | `stat_qq()` + `stat_qq_line()` | Annahme schnell prüfbar |
| Streudiagramm | Punkte eintragen | `geom_point()` | Trends/Zusammenhänge sichtbar |
| Ausreißer (1,5·IQR) | Grenzen von Hand | `IQR()` + Vergleich | reproduzierbar, kein Zählfehler |

```r
# --- Ausreißer nach der 1,5·IQR-Regel ------------------------------------
q  <- quantile(x, c(0.25, 0.75))
bg <- c(q[1] - 1.5 * IQR(x), q[2] + 1.5 * IQR(x))
x[x < bg[1] | x > bg[2]]          # 60  -> der Ausreißer

# --- Visualisierung mit ggplot2 ------------------------------------------
library(ggplot2)
df <- data.frame(messwert = m)

ggplot(df, aes(x = messwert)) +
  geom_histogram(bins = 8, fill = "steelblue", color = "white")

ggplot(df, aes(y = messwert)) +
  geom_boxplot()

ggplot(df, aes(sample = messwert)) +
  stat_qq() + stat_qq_line()
```

---

## 6. KM6 — Lebensdauerverteilungen (Weibull, Exponential, MTTF)

| Thema | Händisch | R | Vorteil |
|---|---|---|---|
| Ausfallwahrscheinlichkeit `F(t)` | Formel je Verteilung | `pweibull`, `pexp` | beliebige Zeitpunkte |
| Parameter schätzen (β, η) | sehr aufwendig | `fitdistrplus::fitdist()` | Maximum-Likelihood automatisch |
| MTTF / MTBF | `η·Γ(1+1/β)` von Hand | `gamma()` im Ausdruck | exakt, sofort |
| Badewannenkurve | qualitativ skizzieren | Kurve aus gefittetem Modell | Daten statt Skizze |

```r
# --- Weibull-Lebensdauer & MTTF ------------------------------------------
library(fitdistrplus)
lebensdauern <- c(120, 145, 98, 210, 175, 130, 160, 190, 115, 140)
fit  <- fitdist(lebensdauern, "weibull")
fit$estimate                                   # shape (β), scale (η)

mttf <- fit$estimate["scale"] * gamma(1 + 1 / fit$estimate["shape"])
mttf                                           # mittlere Lebensdauer
```

---

## 7. Wo R **nicht** (oder kaum) hilft

- **Konzeptionelles Verständnis:** Was bedeutet eine Dichte, warum ist die
  Standardabweichung `n−1`-korrigiert, was sagt ein Konfidenzniveau aus? Das
  muss man verstanden haben, bevor man eine Funktion aufruft.
- **Modellannahmen prüfen und interpretieren:** R liefert Zahlen, nicht die
  Entscheidung, ob Normalverteilung/Unabhängigkeit plausibel sind.
- **Prozessmanagement ohne Zahlen (KM3/KM4):** Prozesslandkarten,
  Managementsysteme, Risikoanalyse, Normenwesen — kein R-Bezug.
- **Einmalige Mini-Rechnungen:** Für einen einzelnen Wert ist der Aufwand
  (Skript, Pakete) manchmal größer als das Nachschlagen.
- **Ergebnisdeutung:** „Was heißt das für den Prozess?“ bleibt
  Ingenieursarbeit.

---

## 8. Fazit

Der quantitative Kern der 3. Klasse — **Verteilungen (KM5)** sowie
**Konfidenzintervalle, Darstellung und Lebensdauern (KM6)** — lässt sich mit R
klar effizienter lösen: Tabellen und Formelumstellungen werden durch
reproduzierbare Funktionsaufrufe ersetzt, Visualisierungen und
Simulationsgegenproben kommen hinzu. Der Zeitgewinn wächst mit der
Datenmenge, und Skripte dokumentieren den Rechenweg automatisch. R ersetzt
jedoch **nicht** das konzeptionelle Verständnis und die Interpretation der
Ergebnisse — genau dort bleibt die eigentliche Denkleistung. R ist damit das
**Werkzeug**, das die Handarbeit abnimmt und den Kopf für die Deutung frei
macht.
