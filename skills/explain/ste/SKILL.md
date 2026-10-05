---
name: ste
description: Explains a technical subject in ~80% ASD-STE100 Simplified Technical English style (short sentences, active voice, one idea per sentence, no intro or outro), written in the user's language. Use when the user asks for a plain, dense, no-fluff explanation, says "explique simplement", "en STE", or when another skill (e.g. catch-up) picks the text format.
---

# Explain — STE text

Apply ~80% of ASD-STE100 rules. The 20% slack: domain terms, class names and code identifiers stay as-is.

## Rules

- One idea per sentence. Max ~20 words.
- Active voice. Subject does the action: "Le service crée le rapport", not "Le rapport est créé".
- Present tense.
- One term = one meaning. Pick the glossary/code term and never swap it for a synonym.
- No introduction, no conclusion, no "en résumé", no "il est important de".
- Group sentences under short bold labels or bullet lists. Max 6 sentences per group.
- Sequences → numbered list, one step per line.
- Code refs as `file:line`, inline.

## Example

Subject: refresh of an OAuth token.

> **Déclencheur**
> - L'API renvoie 401.
> - `apiClient` intercepte la réponse.
>
> **Refresh**
> 1. Le client lit le cookie `refresh_token`.
> 2. Il appelle `POST /oauth/token`.
> 3. Doorkeeper renvoie un nouveau couple de tokens.
> 4. Le client réécrit les deux cookies.
> 5. Le client rejoue la requête initiale.
