#Wahlcheck Dueren
import React, { useMemo, useState } from "react"; import { motion } from "framer-motion"; import { Check, Minus, X, Vote, Info, Percent } from "lucide-react"; import { Button } from "@/components/ui/button"; import { Card, CardContent, CardHeader, CardTitle } from "@/components/ui/card"; import { Progress } from "@/components/ui/progress";

// Hilfs-Typen const CHOICES = { agree: 1, // Zustimmung (grün) neutral: 0, // Neutral (weiß) disagree: -1, // Ablehnung (rot) } as const;

const CHOICE_LABELS: Record<number, string> = { [CHOICES.agree]: "Zustimmung", [CHOICES.neutral]: "Neutral", [CHOICES.disagree]: "Ablehnung", };

// 15 Thesen – aus der Vergleichstabelle abgeleitet // Formulierung so, dass eine klare Pro/Neutral/Contra-Position möglich ist const THESES = [ { id: 1, text: "Düren soll eine kommunale (Kreis‑)Wohnungsbaugesellschaft gründen, um mehr geförderten Wohnraum zu schaffen.", topic: "Wohnen & Stadtentwicklung", }, { id: 2, text: "Das Programm "Jung kauft Alt" wird priorisiert, um Bestandsimmobilien zu beleben.", topic: "Wohnen & Stadtentwicklung", }, { id: 3, text: "Kostenloses Parken in der Innenstadt soll ermöglicht/ausgeweitet werden.", topic: "Mobilität", }, { id: 4, text: "Die Bundesstraße B399n soll realisiert werden.", topic: "Mobilität", }, { id: 5, text: "Der ÖPNV in Düren wird deutlich ausgebaut (inkl. On‑Demand‑Angebote) und barrierefrei gestaltet.", topic: "Mobilität", }, { id: 6, text: "Radvorrangrouten sollen zügig umgesetzt und priorisiert werden.", topic: "Mobilität", }, { id: 7, text: "KITA‑Beiträge sollen entfallen (beitragsfreie Kitas).", topic: "Bildung & Betreuung", }, { id: 8, text: "Die Offene Ganztagsbetreuung (OGS) und Schulsozialarbeit werden ausgebaut.", topic: "Bildung & Betreuung", }, { id: 9, text: "Telemedizin und ein pflegerischer Studiengang in der Region sollen gefördert werden.", topic: "Gesundheit & Pflege", }, { id: 10, text: "Videoüberwachung, Waffen‑/Alkoholverbotszonen und eine Null‑Toleranz‑Linie sollen in Problembereichen eingeführt werden.", topic: "Sicherheit & Ordnung", }, { id: 11, text: "Klimapolitik soll so gestaltet werden, dass soziale Gerechtigkeit mitgedacht wird (z. B. bezahlbare Mobilität).", topic: "Klimaschutz & Umwelt", }, { id: 12, text: "Naturschutz "ohne Verbote": Fokus auf freiwillige Maßnahmen, energetische Sanierung und Begrünung.", topic: "Klimaschutz & Umwelt", }, { id: 13, text: "Ein digitales Bürgerportal mit 72‑Stunden‑Bearbeitungsversprechen soll eingeführt werden.", topic: "Verwaltung & Digitalisierung", }, { id: 14, text: "Mobile Rathäuser und eine umfassende Verwaltungsreform zur Effizienzsteigerung sollen umgesetzt werden.", topic: "Verwaltung & Digitalisierung", }, { id: 15, text: "Kulturelle Angebote (z. B. Endart Kulturfabrik) und Ehrenamt sollen unbürokratisch gefördert werden (inkl. Stadtteilbudgets/Annakirmes).", topic: "Kultur & Ehrenamt", }, ] as const;

// Kandidaten-Metadaten (Kurzinfos + Programmschwerpunkte) const CANDIDATES = [ { key: "hamm", name: "Georg Hamm", party: "CDU", color: "#0b5cff", bio: "Wirtschafts- und Ordnungsfokus: "Stadt des Schaffens", Gewerbeflächen, Innovation Center, Hochschulkooperationen; Gleichberechtigung aller Verkehrsteilnehmer, kostenloses Parken, B399n, Radvorrangrouten; "Schaffens-Kultur" in der Verwaltung, mobile Rathäuser, Verwaltungsreform, KI zur Effizienzsteigerung; Sicherheit mit Null‑Toleranz, Videoüberwachung; Kultur stärken (u. a. Annakirmes).", bullets: [ "Wirtschaft: Stadt des Schaffens, Gewerbeflächen, Innovation Center", "Mobilität: Gleichberechtigung, kostenloses Parken, B399n, Radvorrangrouten", "Verwaltung: Schaffens‑Kultur, mobile Rathäuser, Effizienz & KI", "Sicherheit: Null‑Toleranz, Videoüberwachung, Verbotszonen", "Kultur: Stadtteilbudgets, Annakirmes stärken", ], }, { key: "ullrich", name: "Frank Peter Ullrich", party: "SPD", color: "#e3000f", bio: "Sozialer, nachhaltiger Strukturwandel; Tarifbindung und Industrie 4.0; mehr bezahlbarer Wohnraum (Kreis‑Wohnungsbaugesellschaft), Bodenspekulation bekämpfen; klimafreundliche Mobilität mit ÖPNV‑Ausbau, On‑Demand und Radwegen; beitragsfreie Kitas, Schulsozialarbeit, digitale Infrastruktur; Gesundheit mit Telemedizin, Rettungsdienst und Pflege‑Studiengang; digitales Bürgerportal mit 72h‑Service; soziale Gerechtigkeit im Klimaschutz; unbürokratische Förderung von Kultur & Ehrenamt.", bullets: [ "Wirtschaft: sozialer, nachhaltiger Strukturwandel; Tarifbindung", "Wohnen: Kreis‑Wohnungsbaugesellschaft, geförderter Wohnraum", "Mobilität: ÖPNV‑Ausbau, On‑Demand, barrierefrei, Radwege", "Bildung: beitragsfreie Kitas, Schulsozialarbeit, Digitalisierung", "Gesundheit: Telemedizin, Pflege‑Studiengang, Rettungsdienst", "Verwaltung: Bürgerportal, 72‑Stunden‑Versprechen, KI‑Support", "Klima & Soziales: klimafreundlich und gerecht", "Kultur & Ehrenamt: unbürokratische Förderung", ], }, ] as const;

// Kandidaten-Positionen zu jeder These (Index entspricht THESES[index]) // -1 = Ablehnung, 0 = Neutral/unklar, 1 = Zustimmung const POSITIONS: Record<string, number[]> = { // CDU – aus der Tabelle abgeleitet hamm: [ -1, // 1 Wohnungsbaugesellschaft – stärker SPD-Kompetenz 1, // 2 Jung kauft Alt – CDU-Programm 1, // 3 Kostenloses Parken – CDU für kostenloses Parken 1, // 4 B399n – CDU benennt 0, // 5 ÖPNV-Ausbau – Gleichberechtigung aller Verkehrsteilnehmer; keine klare Priorität 1, // 6 Radvorrangrouten – CDU benennt explizit -1, // 7 Beitragsfreie Kitas – nicht Schwerpunkt CDU 0, // 8 OGS & Schulsozialarbeit – weniger betont 0, // 9 Telemedizin & Pflege-Studiengang – weniger konkret 1, // 10 Sicherheitspaket – CDU für Null-Toleranz & Videoüberwachung 0, // 11 Klima & soziale Gerechtigkeit – nicht zentrales CDU-Framing 1, // 12 Naturschutz ohne Verbote – CDU-Formulierung 0, // 13 Bürgerportal mit 72h – eher SPD-Programm 1, // 14 Mobile Rathäuser & Verwaltungsreform – CDU benennt 1, // 15 Kultur/Ehrenamt inkl. Annakirmes – CDU betont Annakirmes & Stadtteilbudgets ], // SPD – aus der Tabelle abgeleitet ullrich: [ 1, // 1 Wohnungsbaugesellschaft – SPD ja 0, // 2 Jung kauft Alt – nicht ihr Kern -1, // 3 Kostenloses Parken – widerspricht ÖPNV-/Klimafokus -1, // 4 B399n – nicht genannt; eher kritisch bei klimafreundlicher Mobilität 1, // 5 ÖPNV-Ausbau – SPD ja 0, // 6 Radvorrangrouten – nicht explizit, Fokus Radwege allgemein 1, // 7 Beitragsfreie Kitas – SPD ja 1, // 8 OGS & Schulsozialarbeit – SPD ja 1, // 9 Telemedizin & Pflege-Studiengang – SPD ja -1, // 10 Sicherheitspaket streng – nicht SPD-Fokus 1, // 11 Klima + soziale Gerechtigkeit – SPD ja 0, // 12 Naturschutz ohne Verbote – anderes Framing 1, // 13 Bürgerportal + 72h – SPD ja 0, // 14 Mobile Rathäuser & Reform – nicht SPD-Schwerpunkt 1, // 15 Kultur & Ehrenamt unbürokratisch – SPD ja ], };

function Header() { return ( <div className="max-w-5xl mx-auto text-center space-y-4 py-6"> <motion.h1 initial={{ opacity: 0, y: 10 }} animate={{ opacity: 1, y: 0 }} transition={{ duration: 0.5 }} className="text-3xl md:text-4xl font-extrabold" > Düren Wahlcheck – 15 Thesen </motion.h1> <p className="text-muted-foreground"> Positioniere dich zu 15 Thesen. Anschließend siehst du, wie stark deine Antworten mit den Positionen der Kandidaten <strong>Georg Hamm (CDU)</strong> und <strong>Frank Peter Ullrich (SPD)</strong> übereinstimmen. </p> </div> ); }

function CandidateCard({ c }: { c: (typeof CANDIDATES)[number] }) { return ( <Card className="border rounded-2xl shadow-sm"> <CardHeader> <CardTitle className="flex items-center gap-2"> <span className="inline-block w-3 h-3 rounded-full" style={{ background: c.color }} /> {c.name} <span className="text-muted-foreground font-normal">({c.party})</span> </CardTitle> </CardHeader> <CardContent className="space-y-2 text-sm"> <div className="flex items-start gap-2"> <Info className="w-4 h-4 mt-0.5" /> <p>{c.bio}</p> </div> <ul className="list-disc pl-5 space-y-1"> {c.bullets.map((b, i) => ( <li key={i}>{b}</li> ))} </ul> </CardContent> </Card> ); }

function ThesisRow({ thesis, value, onChange, }: { thesis: (typeof THESES)[number]; value: number | null; onChange: (v: number) => void; }) { const active = (v: number) => value === v ? "ring-2 ring-offset-2 ring-black/10 scale-[1.02]" : "opacity-100"; return ( <div className="p-4 border rounded-2xl bg-card/50"> <div className="text-xs uppercase tracking-wide text-muted-foreground mb-1">{thesis.topic}</div> <div className="font-medium mb-3">{thesis.text}</div> <div className="grid grid-cols-3 gap-3"> <button onClick={() => onChange(CHOICES.agree)} className={flex items-center justify-center gap-2 py-2 rounded-xl border ${active(CHOICES.agree)}} style={{ background: "#d9f2d9" }} aria-label="Zustimmung" > <Check className="w-4 h-4" /> Grün </button> <button onClick={() => onChange(CHOICES.neutral)} className={flex items-center justify-center gap-2 py-2 rounded-xl border ${active(CHOICES.neutral)}} style={{ background: "#ffffff" }} aria-label="Neutral" > <Minus className="w-4 h-4" /> Weiß </button> <button onClick={() => onChange(CHOICES.disagree)} className={flex items-center justify-center gap-2 py-2 rounded-xl border ${active(CHOICES.disagree)}} style={{ background: "#f6d6d6" }} aria-label="Ablehnung" > <X className="w-4 h-4" /> Rot </button> </div> </div> ); }

function Scorebar({ label, value }: { label: string; value: number }) { return ( <div className="space-y-2"> <div className="flex items-center justify-between text-sm"> <div className="flex items-center gap-2"><Percent className="w-4 h-4" /> {label}</div> <div className="font-semibold">{value.toFixed(0)}%</div> </div> <Progress value={value} /> </div> ); }

function Results({ answers }: { answers: Array<number | null> }) { const totalAnswered = answers.filter((a) => a !== null).length;

const { hammPct, ullrichPct, hammMatch, ullrichMatch } = useMemo(() => { const calc = (arr: number[], label: string) => { let match = 0; let count = 0; answers.forEach((a, i) => { if (a === null) return; count++; if (a === arr[i]) match++; }); const pct = count === 0 ? 0 : (match / count) * 100; return { pct, match, count }; }; const h = calc(POSITIONS.hamm, "Hamm"); const u = calc(POSITIONS.ullrich, "Ullrich"); return { hammPct: h.pct, ullrichPct: u.pct, hammMatch: ${h.match}/${h.count}, ullrichMatch: ${u.match}/${u.count}, }; }, [answers]);

return ( <Card className="border rounded-2xl"> <CardHeader> <CardTitle className="flex items-center gap-2"> <Vote className="w-5 h-5" /> Deine Auswertung </CardTitle> </CardHeader> <CardContent className="space-y-4"> <p className="text-sm text-muted-foreground"> Übereinstimmung wird als exakte Gleichheit deiner Antwort mit der jeweiligen Kandidatenposition gezählt. Neutrale Antworten zählen nur, wenn auch der Kandidat neutral ist. </p> <Scorebar label="Übereinstimmung mit Georg Hamm (CDU)" value={hammPct} /> <div className="text-xs text-muted-foreground">Treffer: {hammMatch}</div> <Scorebar label="Übereinstimmung mit Frank Peter Ullrich (SPD)" value={ullrichPct} /> <div className="text-xs text-muted-foreground">Treffer: {ullrichMatch}</div> <div className="p-3 rounded-xl bg-muted text-sm"> <strong>Hinweis:</strong> Die Thesen sind aus den veröffentlichten Schwerpunkten der Kandidaten abgeleitet und vereinfachen komplexe Positionen. Für eine fundierte Wahlentscheidung lohnt sich der Blick in die Programme. </div> </CardContent> </Card> ); }

export default function App() { const [answers, setAnswers] = useState<Array<number | null>>( Array(THESES.length).fill(null) );

const reset = () => setAnswers(Array(THESES.length).fill(null));

const allAnswered = answers.every((a) => a !== null);

return ( <div className="min-h-screen bg-background text-foreground"> <div className="max-w-6xl mx-auto p-4 md:p-8 space-y-8"> <Header />

{/* Kandidaten-Infos */}
    <div className="grid md:grid-cols-2 gap-4">
      {CANDIDATES.map((c) => (
        <motion.div
          key={c.key}
          initial={{ opacity: 0, y: 12 }}
          animate={{ opacity: 1, y: 0 }}
          transition={{ duration: 0.4 }}
        >
          <CandidateCard c={c} />
        </motion.div>
      ))}
    </div>

    {/* Thesen */}
    <Card className="border rounded-2xl">
      <CardHeader>
        <CardTitle>Deine Positionen</CardTitle>
      </CardHeader>
      <CardContent className="space-y-4">
        {THESES.map((t, idx) => (
          <ThesisRow
            key={t.id}
            thesis={t}
            value={answers[idx]}
            onChange={(v) => {
              const next = [...answers];
              next[idx] = v;
              setAnswers(next);
            }}
          />
        ))}
        <div className="flex items-center gap-3 pt-2">
          <Button variant="secondary" onClick={reset}>Zurücksetzen</Button>
          {!allAnswered && (
            <div className="text-sm text-muted-foreground">
              {answers.filter((a) => a !== null).length}/{THESES.length} beantwortet
            </div>
          )}
        </div>
      </CardContent>
    </Card>

    {/* Ergebnisse */}
    <Results answers={answers} />

    {/* Appell zum Wählen */}
    <motion.div
      initial={{ opacity: 0, y: 12 }}
      animate={{ opacity: 1, y: 0 }}
      transition={{ duration: 0.5 }}
      className="p-6 rounded-2xl border bg-gradient-to-br from-emerald-50 to-sky-50"
    >
      <h2 className="text-xl font-bold mb-2">Deine Stimme zählt! 💬🗳️</h2>
      <p className="text-sm md:text-base">
        Wählen ist super wichtig – gerade kommunal! Hier werden Weichen gestellt für Bus & Bahn, Wohnungsbau,
        Kultur, Grünflächen, sichere Wege, digitale Verwaltung und vieles mehr. Informiere dich, sprich mit Freundinnen
        und Freunden, geh wählen – und nimm andere mit.
      </p>
    </motion.div>

    <div className="text-center text-xs text-muted-foreground pt-4">
      © {new Date().getFullYear()} Düren Wahlcheck – Bildungsprojekt
    </div>
  </div>
</div>

); }

