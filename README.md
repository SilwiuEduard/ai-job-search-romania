<p align="center">
  <img src="claude_animation.gif" alt="AI Job Search Assistant" width="200">
</p>

# AI Job Search — Adaptat pentru România

Un framework de aplicare la joburi bazat pe [Claude Code](https://claude.com/claude-code), adaptat pentru piața românească. Fork-uiește, completează-ți profilul, și lasă Claude să evalueze posturile, să-ți adapteze CV-ul, să scrie cover letter-e și să te pregătească pentru interviuri.

> **Fork din:** [MadsLorentzen/ai-job-search](https://github.com/MadsLorentzen/ai-job-search)  
> **Adaptat de:** [SilwiuEduard](https://github.com/SilwiuEduard) pentru piața din România

---

## Ce face acest framework

Transformă Claude Code într-un asistent complet de job search. Workflow-ul principal este agnostic față de țară — evaluare de fit, adaptare CV, cover letter — dar a fost configurat specific pentru:

- **Portaluri românești:** ejobs.ro, hipo.ro, bestjobs.eu, LinkedIn România
- **Salarizare:** `salary_data.json` cu 29 companii din piața română (index față de media națională, range-uri nete în RON)
- **Directiva EU 2023/970** (în vigoare din 7 iunie 2026): transparența salarială — salary lookup-ul devine din ce în ce mai util pe măsură ce companiile publică range-uri salariale
- **Diacritice românești:** ă, â, î, ș, ț gestionate corect în fuzzy matching
- **Sufixe juridice românești:** SRL, SA, RA, SCS recunoscute în căutarea de companii

```
/setup          /scrape              /apply <url>
  |                |                     |
  v                v                     v
Completezi     Caută joburi         Evaluează fit-ul
profilul       pe ejobs/hipo/       Scor & recomandare
               bestjobs/LinkedIn        |
  |                |                    v
  v                v               Draft CV + Cover Letter
Fișiere        Prezintă            (LaTeX, personalizat)
profil gata    matches cu               |
               rating de fit           v
                   |              Agent reviewer critică
                   v              -> Revizuire -> Output final
               Alegi un job
               -> /apply
```

---

## Cerințe preliminare

- [Claude Code](https://claude.com/claude-code) (CLI)
- Python 3.10+
- LaTeX: [MiKTeX](https://miktex.org/) (recomandat pe Windows) sau [TeX Live](https://tug.org/texlive/)
  - CV-ul se compilează cu `lualatex`
  - Cover letter-ul se compilează cu `xelatex`

Instalare rapidă MiKTeX pe Windows:
```bash
winget install MiKTeX.MiKTeX --accept-source-agreements --accept-package-agreements
```
La prima compilare, MiKTeX descarcă automat pachetele lipsă.

---

## Pornire rapidă

### 1. Fork și clone

```bash
gh repo fork SilwiuEduard/ai-job-search --clone
cd ai-job-search
```

Sau direct:
```bash
git clone https://github.com/SilwiuEduard/ai-job-search.git
cd ai-job-search
```

### 2. Configurează profilul

```bash
claude
# Apoi în Claude Code:
/setup
```

`/setup` oferă trei căi:
- **Path A:** citește folderul `documents/` dacă ai CV PDF, export LinkedIn, diplome etc.
- **Path B:** importă un CV lipit direct în chat
- **Path C:** interviu ghidat — Claude te întreabă și construiește profilul

Pune testele de personalitate (16Personalities, Holland, CareerExplorer, Sabotori) în `documents/assessments/` — `/setup` le citește și construiește un profil comportamental complet, nu doar din CV.

### 3. Caută joburi

```bash
/scrape
```

Caută pe ejobs.ro, hipo.ro, bestjobs.eu și LinkedIn cu query-urile configurate în `search-queries.md`, deduplicat față de joburile văzute anterior.

Sau mai eficient: găsești 5-10 linkuri pe LinkedIn/eJobs care par interesante și le evaluezi în batch:

```
Evaluate fit for these jobs and rank them:
https://linkedin.com/jobs/view/111
https://linkedin.com/jobs/view/222
https://linkedin.com/jobs/view/333
```

### 4. Aplică la un job

```bash
/apply https://www.linkedin.com/jobs/view/4410498121/
```

Sau cu text lipit direct dacă URL-ul nu poate fi accesat:
```bash
/apply <lipești descrierea jobului>
```

---

## Cum funcționează `/apply`

Workflow în 7 pași, complet automat:

1. **Parsează** postul (URL sau text)
2. **Evaluează fit-ul** pe 4 dimensiuni: skills tehnice, experiență, fit comportamental/cultural, career alignment — fiecare cu scor și justificare
3. **Verifică salariul** față de `salary_data.json` — index față de media națională + range net RON
4. **Redactează** CV adaptat + cover letter în LaTeX (Romanian sau English în funcție de limba postului)
5. **Lansează un agent reviewer** care cercetează compania și critică draft-urile
6. **Revizuiește** pe baza feedback-ului
7. **Compilează și verifică** PDF-urile — CV exact 2 pagini, cover letter exact 1 pagină, fără titluri orfane sau overflow

---

## Salary Benchmarking (Directiva EU 2023/970)

Din 7 iunie 2026, angajatorii din UE sunt obligați să justifice diferențele salariale și, progresiv, să publice range-uri salariale în anunțuri. Asta face `salary_lookup.py` din ce în ce mai util.

**Cum funcționează:**
```bash
python salary_lookup.py "Accenture"
python salary_lookup.py "ING Romania" --city "București"
python salary_lookup.py "UiPath" --json
python salary_lookup.py --list-all
```

**Companii incluse în `salary_data.json`** (29 total):
- Consultanță Big4: Deloitte, PwC, KPMG, EY
- Consultanță IT: Accenture, Capgemini, Cognizant, HCLTech, Wipro, Infosys, Stefanini
- Tech: Microsoft, Amazon, Oracle, UiPath, Bitdefender
- Telecom: Vodafone, Orange
- Retail: eMAG, Kaufland, Lidl
- Banking: ING, BCR, BRD, Banca Transilvania
- Mobility: Bolt, Glovo
- Auto/Industrial: Continental, Bosch

**Format index:** 100 = media națională net (~4500 RON/lună, 2026). Valori peste 100 = peste medie.

**Actualizare date:** Pe măsură ce companiile publică range-uri salariale (obligatoriu UE), adaugă-le manual în `salary_data.json` sau folosește `tools/convert_salary_excel.py` dacă ai date în Excel.

---

## Structura fișierelor

```
ai-job-search/
├── CLAUDE.md                          # Profil candidat + reguli workflow
├── .claude/
│   ├── commands/
│   │   ├── apply.md                   # Workflow /apply (drafter-reviewer)
│   │   ├── setup.md                   # Onboarding /setup
│   │   ├── expand.md                  # Îmbogățire profil din surse online
│   │   └── reset.md                   # Reset profil sau documente
│   ├── skills/
│   │   ├── job-application-assistant/ # Skill principal de aplicare
│   │   │   ├── 01-candidate-profile.md   # Educație, experiență, skills
│   │   │   ├── 02-behavioral-profile.md  # Profil ENTJ/Holland/Sabotori etc.
│   │   │   ├── 03-writing-style.md       # Ton, structură, do's/don'ts
│   │   │   ├── 04-job-evaluation.md      # Framework de scoring fit
│   │   │   ├── 05-cv-templates.md        # Template-uri LaTeX CV
│   │   │   ├── 06-cover-letter-templates.md # Template-uri cover letter
│   │   │   └── 07-interview-prep.md      # Exemple STAR + framework interviu
│   │   ├── job-scraper/               # Orchestrare căutare joburi (România)
│   │   └── upskill/                   # Analiză gap de skills
├── cv/
│   └── main_example.tex               # Template moderncv (banking style)
├── cover_letters/
│   ├── cover.cls                      # Clasă LaTeX cover letter
│   └── OpenFonts/                     # Fonturi Lato + Raleway
├── documents/                         # Materiale sursă pentru /setup
│   ├── cv/                            # CV master (PDF sau .tex)
│   ├── linkedin/                      # Export LinkedIn PDF
│   ├── assessments/                   # Teste personalitate (16P, Holland etc.)
│   ├── diplomas/                      # Diplome și certificate
│   ├── references/                    # Scrisori de recomandare
│   └── applications/                  # Aplicații anterioare
├── salary_lookup.py                   # Tool salary benchmarking (România)
├── salary_data.json                   # Date salariale 29 companii România
├── tools/
│   ├── convert_salary_excel.py        # Convertește Excel → JSON salary data
│   └── README_SALARY_TOOL.md          # Instrucțiuni salary tool
├── job_search_tracker.csv             # Tracker aplicații
└── SETUP.md                           # Ghid setup detaliat
```

---

## Alte comenzi

- **`/expand`** — îmbogățește profilul scanând sursele publice linkuite (GitHub, portofoliu, Kaggle). Adaugă competențe descoperite cu tag de sursă.
- **`/upskill`** — analizează gap-ul dintre profilul tău și joburile urmărite. Produce un heatmap de gaps și un plan de învățare cu resurse și estimări de timp.
- **`/reset profile`** — șterge fișierele de profil, păstrează regulile framework-ului
- **`/reset documents`** — șterge folderul documents/
- **`/reset all`** — ambele

---

## Sfaturi pentru rezultate mai bune

**Profilul contează mai mult decât orice altceva.** Un profil detaliat produce aplicații genuinely personalizate; un profil subțire produce output generic.

- Descrie ce ai făcut efectiv în fiecare rol, nu doar titlul
- Include cifre concrete (1500+ ride-uri procesate, 5.7M rânduri, 39.3% segment identificat)
- Pune testele de personalitate în `documents/assessments/` — profilul comportamental generat din date reale e mult mai util decât cel inferit din CV
- Completează exemplele STAR în `07-interview-prep.md` — îmbunătățesc semnificativ calitatea prep-ului de interviu

---

## Limitări cunoscute

- **LinkedIn blochează scraping-ul automat** — `/scrape` folosește WebSearch în loc de CLI dedicat. Workflow-ul practic recomandat: tu găsești 5-10 URL-uri, Claude evaluează și rankează în batch.
- **eJobs/hipo/bestjobs nu au API public** — același workaround ca LinkedIn.
- **Datele salariale sunt aproximative** — bazate pe surse publice (Glassdoor, PayScale, salarii.ro). Actualizează `salary_data.json` cu date din ofertele reale pe măsură ce companiile publică range-uri (Directiva EU 2023/970).

---

## Licență

MIT

---

*Bazat pe [MadsLorentzen/ai-job-search](https://github.com/MadsLorentzen/ai-job-search). Adaptat pentru piața română de [SilwiuEduard](https://github.com/SilwiuEduard).*
