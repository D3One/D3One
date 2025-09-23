
# 🧭 CISM (ISACA) — Theory & Practical Q\&As (up to 2012)

**Author:** Ivan Piskunov

> Anchor references: **ISACA’s CISM domains**, **COBIT 5** (2012), **ISO/IEC 27001:2013**, **NIST SP 800-30 Rev.1** (risk), **NIST SP 800-53 Rev.4** (controls), **NIST SP 800-61 Rev.2** (incident), and **PCI DSS 3.0** (2013). ([ISACA][1])

---

## MAIN Q\&As (each with a quick “why”)

### Domain 1 — Information Security Governance

1. **What’s the purpose of an information security program charter and who approves it?**
   **Answer:** It defines mandate, scope, and authority for the security program; **approved by senior management/board** and owned by the **CISO**.
   **Why:** Governance sets tone-at-the-top; CISM emphasizes executive sponsorship. ([ISACA][1])

2. **Risk appetite vs. risk tolerance—who sets them?**
   **Answer:** Executives/board define **appetite** (overall willingness) and **tolerances** (operational bounds).
   **Why:** Governance decisions precede control selection; COBIT 5 frames this at the **EDM** level. ([ISACA][2])

3. **Map governance vs. management using COBIT 5.**
   **Answer:** **Governance = EDM** (Evaluate, Direct, Monitor); **Management = APO/BAI/DSS/MEA** domains.
   **Why:** Clear split helps place policies, risk, and operations correctly. ([ISACA][2])

4. **Policy stack—put these in order:** guideline, procedure, standard, policy, baseline.
   **Answer:** **Policy → Standard → Baseline → Procedure → Guideline.**
   **Why:** Policy states intent; standards/baselines make it measurable; procedures operationalize; guidelines advise.

5. **KPI vs. KRI vs. KCI—what’s the difference?**
   **Answer:** **KPI** = performance of controls/process; **KRI** = early signal of elevated risk; **KCI** = whether a control is in place/working.
   **Why:** Governance needs the right metric for the decision.

6. **Best forum to align security with business goals?**
   **Answer:** A **security/IT steering committee** with business, risk, and audit representation.
   **Why:** Ensures prioritization, funding, and accountability.

7. **Most defensible way to justify a new control set?**
   **Answer:** A **business case** tied to enterprise objectives and risk reduction, referencing applicable standards (e.g., ISO 27001 Annex A).
   **Why:** CISM is business-first, control-second. ([ISO][3])

8. **Why prefer AD-integrated DNS zones? (governance angle)**
   **Answer:** Integrity/availability via multi-master + secure updates; aligns with policy for critical services.
   **Why:** Governance cares about risk posture of core infra, not just configs.

9. **Where should security requirements for outsourcing live?**
   **Answer:** In the **contract/SLA**: security clauses, right-to-audit, incident reporting, data location, and termination/return.
   **Why:** Governance enforces risk treatment through third-party agreements.

10. **Biggest governance anti-pattern you’d flag?**
    **Answer:** **Policy without enforcement/metrics.**
    **Why:** Without measures (KRI/KCI), policy is theater.

---

### Domain 2 — Information Risk Management & Compliance

11. **Name the core steps in a risk assessment per NIST SP 800-30 Rev.1.**
    **Answer:** **Prepare → Identify → Analyze (likelihood/impact) → Determine risk → Document/Report.**
    **Why:** CISM expects method literacy; 800-30 r1 is canonical. ([NIST Computer Security Resource Center][4])

12. **Qual vs. quant analysis—when to use which?**
    **Answer:** **Qualitative** for speed and scarce data; **quantitative** when loss data exists (ALE = SLE × ARO).
    **Why:** Pick the model that fits available evidence.

13. **Four classic risk treatments?**
    **Answer:** **Mitigate, Transfer, Avoid, Accept** (with sign-off).
    **Why:** Standard taxonomy across CISM/ISO/NIST.

14. **Who can accept residual risk?**
    **Answer:** The **business owner** (risk owner), not the security team.
    **Why:** Ownership sits with those who bear impact.

15. **What is an ISO 27001 Statement of Applicability (SoA)?**
    **Answer:** A documented list of applicable **Annex A controls**, with inclusion/exclusion rationale and implementation status.
    **Why:** Proves risk-based selection and accountability. ([ISO][3])

16. **Control catalog to cite if your org follows NIST?**
    **Answer:** **NIST SP 800-53 Rev.4** control families (e.g., AC, AU, CM…).
    **Why:** Widely adopted, 2013/2014-era reference. ([NIST Computer Security Resource Center][5])

17. **Data classification—minimum tiers you’d require?**
    **Answer:** **Public, Internal, Confidential, Restricted** (or similar), with handling rules.
    **Why:** Enables proportionate safeguards and audit criteria.

18. **Third-party risk DD essentials?**
    **Answer:** Security questionnaire, evidence (certs/attestations), **right-to-audit**, incident SLAs, and data flow mapping.
    **Why:** Materially reduces blind spots in the supply chain.

19. **For cardholder environments (2013), which standard and quick scope-reduction tactic?**
    **Answer:** **PCI DSS 3.0**; reduce scope via **network segmentation** and **tokenization**.
    **Why:** 3.0 emphasized clarity and stronger scoping practices. ([PCI Security Standards Council][6])

20. **BIA vs. risk assessment—what’s the difference?**
    **Answer:** **BIA** quantifies business impact over time (RTO/RPO); **risk assessment** evaluates threat/likelihood/impact to choose controls.
    **Why:** CISM separates continuity planning from pure risk analysis.

21. **Risk register—what must every line item contain?**
    **Answer:** **Risk statement, owner, inherent/residual ratings, treatment, milestones, next review date.**
    **Why:** Without ownership and dates, registers decay.

22. **Risk appetite breach—what’s your first move?**
    **Answer:** **Escalate** per governance, propose treatments, and track **KRI** until back within tolerance.
    **Why:** Appetite/tolerance are executive-level constraints. ([ISACA][2])

---

### Domain 3 — Information Security Program Development & Management

23. **First artifact when (re)bootstrapping a program?**
    **Answer:** A **roadmap** mapped to business goals and a maturity model (e.g., COBIT 5 processes/measures).
    **Why:** Shows staged value delivery. ([ISACA][2])

24. **How do you prove an awareness program works?**
    **Answer:** Pre/post assessments, phishing sims, and **KPI/KRI** ties (e.g., reporting rate, click-through trending).
    **Why:** CISM wants measurable behavior change.

25. **Where does security plug into SDLC (2012–2014)?**
    **Answer:** **Requirements, design reviews, code analysis, pre-prod hardening, release gating**, and prod monitoring.
    **Why:** Shift-left was already a thing—even without today’s buzzwords.

26. **What’s the best control to govern changes to production?**
    **Answer:** A **formal change-management process** with risk assessment, CAB approval, back-out plan, and post-change validation.
    **Why:** Reduces operational risk and audit findings.

27. **Vulnerability management lifecycle—name the stages.**
    **Answer:** **Discover → Prioritize → Remediate/mitigate → Verify → Report → Improve.**
    **Why:** CISM expects closed-loop governance.

28. **Patch management—how to prioritize?**
    **Answer:** **Asset criticality × exploitability** (e.g., Internet-facing, active exploit), maintenance windows, and roll-back safety.
    **Why:** Risk-based, not calendar-based.

29. **Log strategy for a regulated shop (2014):**
    **Answer:** Centralize to **syslog/SIEM**, define **retention**, protect integrity, and baseline alerts for use cases (auth, change, admin).
    **Why:** Supports incident handling and evidence.

30. **Segregation of duties example that audit will love.**
    **Answer:** **Developers** can’t deploy to prod; **ops** deploys; **security** approves risk; **audit** reviews.
    **Why:** Breaks fraud/opsec single-points-of-failure.

31. **Cloud adoption (2012–2014) top governance message?**
    **Answer:** **Shared responsibility** model + contractual controls (location, access, incident notice) + third-party risk.
    **Why:** Many early missteps were governance, not tech.

32. **Baseline configuration—what’s non-negotiable?**
    **Answer:** Hardened images, **secure defaults**, change control, and **attestation** via config management.
    **Why:** It’s the bedrock for change detection and audits.

---

### Domain 4 — Information Security Incident Management

33. **NIST IR lifecycle (2012): list the phases.**
    **Answer:** **Preparation → Detection/Analysis → Containment → Eradication → Recovery → Post-Incident (Lessons Learned).**
    **Why:** NIST SP 800-61 Rev.2’s canonical flow. ([NIST Computer Security Resource Center][7])

34. **What’s the first triage dimension after detection?**
    **Answer:** **Severity/impact and scope** (data sensitivity, service criticality, blast radius).
    **Why:** Drives containment choice and notifications. ([NIST Computer Security Resource Center][8])

35. **Evidence handling 101?**
    **Answer:** **Chain of custody**, time sync, write-blocked imaging, and minimal access by a small, trained team.
    **Why:** Preserves admissibility and integrity. ([NIST Computer Security Resource Center][8])

36. **Who speaks externally during an incident?**
    **Answer:** **A designated communications lead** (e.g., PR/Legal) per the plan—not ad hoc engineers.
    **Why:** Reduces liability and rumor-driven damage. ([NIST Computer Security Resource Center][8])

37. **ISO standard you’d cite for incident management (2011 vintage)?**
    **Answer:** **ISO/IEC 27035:2011** for structured incident management practices.
    **Why:** Complements NIST lifecycle with international guidance. ([ISO][9])

38. **Best way to rehearse the plan?**
    **Answer:** **Tabletop exercises** and technical simulations with after-action items tracked to closure.
    **Why:** Turns a binder into muscle memory. ([NIST Computer Security Resource Center][8])

39. **Insider exfiltration—first control you check?**
    **Answer:** **Data loss monitoring** around e-mail/web/endpoint + privileged access monitoring (PAM).
    **Why:** Incidents aren’t only perimeter events; watch data paths.

40. **How does IR tie to BCP/DR?**
    **Answer:** Escalation to **disaster declaration** when RTO/RPO are at risk; coordinate hand-off and recovery validation.
    **Why:** CISM expects cohesion between IR and continuity planning.

---

## Recommended study references (2012–2014 appropriate)

* **ISACA CISM Exam Outline / Domains** (official). ([ISACA][1])
* **COBIT 5** framework (governance vs. management, principles, enablers). ([ISACA][2])
* **ISO/IEC 27001:2013** (ISMS, SoA, risk-based control selection). ([ISO][3])
* **NIST SP 800-30 Rev.1** — *Guide for Conducting Risk Assessments* (Sep 2012). ([NIST Computer Security Resource Center][4])
* **NIST SP 800-53 Rev.4** — *Security and Privacy Controls* (2013 update; widely used in that era). ([NIST Computer Security Resource Center][5])
* **NIST SP 800-61 Rev.2** — *Computer Security Incident Handling Guide* (Aug 2012). ([NIST Computer Security Resource Center][7])
* **ISO/IEC 27035:2011** — Incident management guidance. ([ISO][9])
* **PCI DSS 3.0 (2013) — Summary of Changes.** ([PCI Security Standards Council][6])

**Books & prep (period-correct):**

* **CISM Review Manual, 2014** (ISACA). ([eBay][10])
* **CISM Review Questions, Answers & Explanations Manual, 2014** (ISACA). ([ACM Digital Library][11])
* **COBIT 5 Framework** and **COBIT 5 for Risk** (ISACA). ([Santa Clara County Files][12])

---

## Attribution

Compiled by **I.P.**. Wording and scenarios are original, informed by the standards/frameworks above.

[1]: https://www.isaca.org/credentialing/cism/cism-exam-content-outline?utm_source=chatgpt.com "CISM Exam Content Outline | CISM Certification"
[2]: https://www.isaca.org/resources/cobit/cobit-5?utm_source=chatgpt.com "COBIT 5 Framework Publications"
[3]: https://www.iso.org/contents/data/standard/05/45/54534.html?utm_source=chatgpt.com "ISO/IEC 27001:2013 - Information security management ..."
[4]: https://csrc.nist.gov/pubs/sp/800/30/r1/final?utm_source=chatgpt.com "SP 800-30 Rev. 1, Guide for Conducting Risk Assessments"
[5]: https://csrc.nist.gov/pubs/sp/800/53/r4/upd3/final?utm_source=chatgpt.com "SP 800-53 Rev. 4, Security and Privacy Controls for Federal ..."
[6]: https://www.pcisecuritystandards.org/minisite/en/docs/PCI_DSS_v3_Summary_of_Changes.pdf?utm_source=chatgpt.com "Data Security Standard"
[7]: https://csrc.nist.rip/pubs/sp/800/61/r2/final?utm_source=chatgpt.com "SP 800-61 Rev. 2, Computer Security Incident Handling Guide"
[8]: https://csrc.nist.gov/pubs/sp/800/61/r2/final?utm_source=chatgpt.com "SP 800-61 Rev. 2, Computer Security Incident Handling Guide"
[9]: https://www.iso.org/standard/44379.html?utm_source=chatgpt.com "ISO/IEC 27035:2011 - Security techniques"
[10]: https://www.ebay.com/itm/226811534553?utm_source=chatgpt.com "CISM REVIEW MANUAL 2014 By Isaca **Mint Condition"
[11]: https://dl.acm.org/doi/book/10.5555/2601818?utm_source=chatgpt.com "CISM Review QAE Manual 2014: | Guide books"
[12]: https://files.santaclaracounty.gov/migrated/COBIT-5_res_eng_1012%20%28ISACA%29.pdf?utm_source=chatgpt.com "A Business Framework for the Governance and ..."


### 📚 Official Resources for Current CISM Candidates

For reference, here are the official resources for the current CISM exam. While the content has evolved, the core domains remain centered on management principles.

| Resource | Description | Source |
| :--- | :--- | :--- |
| **CISM Certification Page** | The official homepage for the certification, containing exam details, requirements, and the latest updates. | [ISACA.org/CISM](https://www.isaca.org/credentialing/cism) |
| **Free CISM Practice Quiz** | A small official quiz to familiarize yourself with the current question format and difficulty. | [ISACA Practice Quiz](https://www.isaca.org/credentialing/cism/cism-practice-quiz) |
| **Exam Candidate Guide** | Provides essential information on exam registration, scheduling, policies, and scoring. | [ISACA Exam Guides](https://www.isaca.org/credentialing/exam-candidate-guides) |
| **Training & Events** | ISACA's portal for official training courses, self-paced learning, and other exam prep resources. | [ISACA Training](https://www.isaca.org/training-and-events/training-topics) |
