# Formelübersicht – Ansparphase & Entnahmephase

Dieses Dokument beschreibt **sämtliche mathematische Formeln**, die in der Anwendung für die _Ansparphase_ (Kapitalaufbau) und die _Entnahmephase_ (Kapitalverzehr) verwendet werden.  Jede Formel wird mit allen relevanten Parametern, deren Bedeutung, Einheit sowie Standard-/Grenzwerten (falls vorhanden) angegeben.

> Hinweis: Alle Variablennamen sind an die Bezeichnungen im Quellcode (`app.js`) angelehnt, um die Zuordnung zu vereinfachen.

---

## 1  Ansparphase (Capital Accumulation)

### 1.1  Monatliche Kapitalentwicklung

1. **Monatlicher Zinssatz**  
   \[
     r_{\text{mon}} = \frac{r_{\text{p.a.}}}{12}
   \]

2. **Kapital am Monatsende** (Reinvestition & Sparrate)  
   Für Monat *m* (beginnend bei 1) gilt rekursiv  
   \[
     K_{m} = K_{m-1}\,\bigl(1 + r_{\text{mon}}\bigr) \; + \; S_{j}
   \]
   wobei  
   • \(K_{0}\)  … Anfangskapital (`initialCapital`)  
   • \(S_{j}\) …monatliche Sparrate des aktuellen Jahres *j* (`currentMonthlySavings`)

   Geschlossene Form nach *n* Monaten:  
   \[
     K_{n} = K_{0}\,(1 + r_{\text{mon}})^{n} 
             + \; S_{j}\;\frac{(1 + r_{\text{mon}})^{n}-1}{r_{\text{mon}}}
   \]

### 1.2  Jährliche Anpassung der Sparrate (Gehaltssteigerung)

1. **Bruttogehaltssteigerung**  
   \[
     \Delta G_{\text{brutto}} = G_{j-1} \cdot g_{\text{salary}}
   \]
2. **Neues Bruttogehalt**  
   \[
     G_{j} = G_{j-1} + \Delta G_{\text{brutto}}
   \]
3. **Netto-Gehalts­steigerung** (unter progressiver Einkommensteuer)  
   \[
     \Delta G_{\text{netto}} = \operatorname{Netto}(G_{j}) - \operatorname{Netto}(G_{j-1})
   \]
4. **Erhöhung der Sparrate**  
   \[
     \Delta S = \frac{\Delta G_{\text{netto}}}{12}\; \cdot\; q_{\text{save}}
   \]
   \[
     S_{j} = S_{j-1} + \Delta S
   \]
   
   Parameter:  
   • \(g_{\text{salary}}\) …jährliche Bruttogehaltsrate (`salaryGrowth`)  
   • \(q_{\text{save}}\) …Quote des zusätzlichen _Netto_-Einkommens, die gespart wird (`salaryToSavings`)

### 1.3  Inflationsbereinigtes Kapital

\[
  K_{\text{real},j} 
    = \frac{K_{j}}{\bigl(1+\pi\bigr)^{j}}
\]

Parameter: \(\pi\) …Inflationsrate p.a. (`inflationRate`).

### 1.4  Deutsche ETF-Besteuerung (Vorabpauschale)

1. **Basisertrag**  
   \[
     B = K_{\text{Start}} \cdot z_{\text{Basis}} \cdot 70\,\%
   \]
2. **Wertzuwachs**  
   \[
     W = \max\bigl(0, K_{\text{Ende}}-K_{\text{Start}}\bigr)
   \]
3. **Vorabpauschale**  
   \[
     V = \min\bigl(B, W\bigr)
   \]
4. **Teilfreistellung**  
   \[
     V^{\*} = V \cdot \bigl(1- \tau_{\text{TF}}\bigr)
   \]
   mit \(\tau_{\text{TF}} = 30\,\%\) bei Aktien-ETFs, sonst 0.
5. **Verbleibender Sparerpauschbetrag**  
   \[
     A = \max\bigl(0, 1000€ - A_{\text{bisher}}\bigr)
   \]
6. **Steuerpflichtiger Betrag**  
   \[
     V^{\dagger} = \max\bigl(0, V^{\*} - A\bigr)
   \]
7. **Fällige Steuer**  
   \[
     T = V^{\dagger} \cdot 25\,\%
   \]

---

## 2  Entnahmephase (Capital Withdrawal)

### 2.1  Direkte Annuitätenformel für die Anfangsentnahme

\[
  \text{PMT} 
    = PV \;\times\; \frac{r\, (1+r)^{n}}{(1+r)^{n}-1}
\]

| Symbol | Bedeutung | Wert im Code |
|--------|-----------|--------------|
| \(PV\) | Startkapital (`initialCapital`) | > 0 € |
| \(r\) | Erwartete Rendite p.a. (`annualReturn`) | beliebig, typ. 0–10 % |
| \(n\) | Entnahmedauer in Jahren (`duration`) | > 0 |
| PMT | jährliche Anfangsentnahme | Ergebnis |

### 2.2  Jährliche Inflationsanpassung der Entnahme

\[
  W_{j} = \text{PMT}\;\cdot\; (1+\pi)^{\,(j-1)}
\]

### 2.3  Portfolio-Entwicklung im Jahr *j*

1. **Nach Rendite**  
   \[
     K'_{j} = K_{j-1}\,\bigl(1+r\bigr)
   \]
2. **Nach Brutto-Entnahme**  
   \[
     K_{j} = K'_{j} - W_{j}
   \]

### 2.4  Besteuerung der Entnahme (vereinfachtes Modell)

1. **Realisierter Gewinn**  
   \[
     G = W_{j} - \text{CostBasis}_{j}\]
2. **Teilfreistellung**  
   \[
     G^{\*} = G \cdot (1-\tau_{\text{TF}})\]
3. **Anrechnung des Sparerpauschbetrags**  
   \[
     G^{\dagger} = \max(0, G^{\*} - A)\]
4. **Steuer**  
   \[
     T_{j} = G^{\dagger}\,\cdot\, 25\,\%\]
5. **Netto-Entnahme**  
   \[
     W_{j}^{\text{netto}} = W_{j} - T_{j}\]

### 2.5  Reale Kaufkraft der Entnahme

\[
  W_{j}^{\text{real}} = \frac{W_{j}}{(1+\pi)^{\,(j-1)}}
\]

---

## 3  Parameterübersicht

| Parameter | Beschreibung | Typische Werte |
|-----------|--------------|----------------|
| `annualReturn` \(r\) | Erwartete Jahresrendite des Portfolios | 0 % – 10 % |
| `inflationRate` \(\pi\) | Erwartete Jahresinflation | 0 % – 4 % |
| `salaryGrowth` *g* | Bruttogehaltswachstum p.a. | 0 % – 5 % |
| `salaryToSavings` *q* | Anteil der Netto-Gehaltssteigerung, der gespart wird | 0 – 1 |
| `teilfreistellung` \(\tau_{\text{TF}}\) | Teilfreistellungssatz für Aktien-ETFs | 0 oder 0,30 |
| `TAX_FREE_ALLOWANCE` *A* | Sparerpauschbetrag | 1 000 € pro Jahr |
| `TAX_RATE` | Kapitalertragsteuer (Abgeltungssteuer) | 25 % |

---

## 4  Literatur & Gesetzliche Grundlagen

* Einkommensteuergesetz (EStG) § 20 & § 52 – Kapitaleinkünfte, Vorabpauschale  
* Bundesfinanzministerium – Basiszins zur Vorabpauschale (Verfügung 2025: 2,53 %)  
* Deutsche Rentenversicherung – Realzinsformel für Kaufkraftberechnungen  
* Schulbuchformel für Anwartschaftsbarwert / Rentenbarwert (Annuität)

---

> © 2025 – Budget-Sparen-Entnahme-Rechner • Alle Rechte vorbehalten 