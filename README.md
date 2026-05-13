# AamaLink 🌿
### AI-Powered Maternal Health Navigator for Rural Karnali, Nepal
**SDG 3: Good Health and Well-Being — Target 3.1 (Reduce maternal mortality)**

---

## Problem

In ward 4 of Kanakasundari Rural Municipality, Jumla, a 21-year-old woman named Samjhana — married at 15, pregnant with her third child — feels a severe headache and notices her hands are swelling. Her husband does not recognize these as danger signs. The nearest hospital, Karnali Academy of Health Sciences, is a two-day walk away. There is no doctor in her ward. The health post ran out of medicine three months ago.

This is not a hypothetical. Karnali Province has the highest maternal mortality burden in Nepal. **80% of deaths occur from undetected, preventable complications** — not because treatment doesn't exist, but because families cannot identify when "something is wrong" has crossed into "we need to leave now." Health posts are absent in 117 of the province's 718 wards. Medical information that exists is written in clinical English, making it inaccessible to the Nepali-speaking families who need it most.

The exact failure point is this: **a family in crisis cannot distinguish a danger sign from a normal pregnancy discomfort.** The information exists in WHO guidelines — it cannot be found, cannot be understood, and when it is understood, there is no clear path to action. AamaLink addresses all three failures.

---

## AI Capability

AamaLink uses two capabilities from the labs, combined:

**Lab 2 — Structured Extraction (Triage Engine):** A family member types symptoms in Nepali — informal, incomplete, panicked text. The AI extracts a structured JSON: urgency level (HIGH/MEDIUM/LOW), danger signs detected, likely condition, and recommended facility tier. This matches exactly what Lab 2 demonstrated: turning messy unstructured input into actionable structured data. The difference from the 311 lab context is that a misclassification here costs a life, not a delayed pothole repair.

**Lab 1 — Text Generation (Plain-Language Response):** Once the triage schema is populated, the AI generates a short response in simple Nepali — no medical jargon, under five sentences, ending with a facility name and phone number. This mirrors the Lab 1 multilingual 311 assistant: one system prompt instruction determines whether the tool serves a Karnali family or excludes them entirely.

The combination is the key insight: structured extraction alone produces data no one reads; plain-language generation alone produces advice with no urgency calibration. Together they produce a triage decision *and* a human-readable directive in the user's language.

---

## Workflow

```
FAMILY MEMBER TYPES SYMPTOMS IN NEPALI
             ↓
    [Mandatory Context Check]
    "Is the person pregnant? (Y/N)"
             ↓
    GEMINI TRIAGE ENGINE (Lab 2)
    Extracts structured JSON:
    • urgency_level: HIGH / MEDIUM / LOW
    • detected_symptoms: [list]
    • danger_sign_detected: true/false
    • recommended_facility_tier
    • language_detected
             ↓
      ┌──────┴──────┐
    HIGH           MEDIUM / LOW
      ↓               ↓
  HUMAN REVIEW    AI-generated
  GATE: "CALL     plain-language
  NOW" button     Nepali response
  before any      + facility name
  routing         + contact number
      └──────┬──────┘
             ↓
    FAMILY ACTS ON CLEAR DIRECTIVE
```

**What goes in:** A Nepali text message describing symptoms — typed by a husband, mother-in-law, or the woman herself on a basic smartphone.

**What the AI does:** Evaluates against WHO maternal danger signs (severe headache, visual disturbances, heavy bleeding, no fetal movement, fever >38°C, swelling of face/hands), assigns urgency, matches to the real Karnali facility directory.

**What comes out:** A short plain-Nepali message telling the family exactly what to do and who to call — plus a structured JSON log for health system record-keeping.

**Who acts on it:** The family. For HIGH urgency cases, a human health volunteer sees the alert before the family receives the routing directive.

**Facility directory (real, verified):**
| Facility | Location | Tier | Contact |
|---|---|---|---|
| Karnali Academy of Health Sciences (KAHS) | Jumla | HIGH urgency | +977-87-520114 |
| Province Hospital Surkhet | Surkhet | HIGH urgency (alt) | +977-83-520777 |
| Tripurkot Health Post | Dolpa District | MEDIUM urgency | Local health volunteer |
| Shreenagar Health Post | Humla District | MEDIUM urgency | Local health volunteer |
| Rimi Health Post | Jagadulla Rural Municipality | LOW urgency | FCHV on duty |

*Sources: Nepal National Geoportal, KAHS official records, BMC Health Services Research (Jumla study, 2021)*

> **Screenshots:** See `/screenshots/` folder in this repo for Gemini outputs from all three test cases.

---

## Failure Case

**Input tested (Cell 4, Case 3 in the prototype):**
`"अलि टाउको दुखेको छ।"` — *"Slight headache."* No pregnancy context provided.

**What the AI returned:** `urgency_level: LOW`, `danger_sign_detected: false`, routing to FCHV (lowest tier).

**Real-world consequence:** A 25-year-old woman named Sita in Humla district has preeclampsia. Her husband — scared, in a hurry — types only "slight headache" without mentioning she is 8 months pregnant. The AI classifies LOW. The family waits. Twelve hours later Sita develops eclamptic seizures. The nearest helicopter evacuation point is a two-day walk from their village. KAHS in Jumla has the capacity to treat her — but the window has closed.

**Why this failure is structurally predictable:** Lab 2 demonstrated that the AI routes based entirely on what is typed. The schema has no memory, no follow-up questions, and no ability to flag *missing* context as a risk factor. Incomplete input in a crisis is not an edge case — it is the norm. A husband typing in panic will not write a complete clinical history.

**Lab evidence:** Prototype Cell 4, Case 3 output — visible in notebook with cells uncleared.

---

## Oversight and Tradeoff

**Where human review sits:** Every output classified HIGH urgency triggers a mandatory hold before routing. The family sees: *"This may be serious. We are connecting you with a health volunteer to confirm before you travel."* A Female Community Health Volunteer (FCHV) or on-call health worker receives the alert and approves or escalates the routing within 15 minutes. The AI recommendation is an input to a human decision — not the decision itself.

**Justification from the lab:** Case 1 in Cell 4 showed the AI correctly identified preeclampsia symptoms as HIGH. Case 3 showed it *failed* on incomplete input and returned LOW. Because the system cannot distinguish "low urgency" from "incomplete description of high urgency," no HIGH routing should reach a family without a human confirming it — and no LOW routing should be trusted without first confirming pregnancy status.

**The one change:** Add a mandatory intake question before triage: *"Is the person you are describing currently pregnant? (Yes / No / Not sure)"* If YES → re-evaluate all symptoms against the full WHO maternal danger sign checklist. If the answer is missing or ambiguous → default to MEDIUM urgency, never LOW.

**What it costs:** One additional interaction step adds ~10 seconds of friction and requires re-engineering the triage prompt to branch on pregnancy status. It will produce more MEDIUM classifications for non-urgent cases, increasing false referrals to health posts that are already understaffed and medicine-short. That is the real tradeoff: reducing missed high-urgency cases means increasing the load on health posts for cases that turn out to be low-urgency. In a system where health posts already run out of medicine, more foot traffic has a cost. The alternative — a missed preeclampsia case — has a higher one.

---

*Built for Fundamentals of MIS — AI for Social Good, Spring 2026*
*Prototype uses Google Gemini 2.5 Flash via the Gemini API*
*AI assistance used for code debugging and Nepali language prompt testing; all design decisions, ethics analysis, and facility research conducted by the team*
