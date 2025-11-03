# Project Definition Document (PDD) — Basic Arabic Reading



## Goal

Build a self-paced, interactive web app that teaches beginners to read Arabic using the "Basic Arabic Reading" curriculum.



## Scope

- Convert each lesson into modular interactive units with practice and feedback.

- Support randomized practice for letters, vowels, and joining forms.

- Track user progress locally at first. Server-side persistence can be added later.

- Release as open source and invite community contributions.



## Non-Goals (initial)

- Full Tajwīd rules beyond what is covered in the book.

- Native mobile apps.

- Complex gamification or social features.



## Primary Users

- English-speaking beginners with no prior Arabic reading ability.



## Success Metrics

- Users can correctly identify letters and vowelized syllables.

- Completion of Lesson 1 to Lesson 6 results in measurable accuracy gains.

- Time-to-first-success: a user completes the first practice within minutes.



## Key Features (MVP)

- Lesson navigation and progress save

- Randomized practice (letters, vowels, joining)

- Simple accessible UI

- Content sourced from the book with CC BY-SA attribution

- Internationalization-ready



## Future Features

- Audio for letter names and examples

- Teacher mode and sharable practice sets

- User accounts and cloud sync

- Mobile-friendly enhancements



## RNG Practice Examples

- Random letter identification with configurable subsets

- Random vowel application to a chosen letter

- Random joined-forms drills for common pairs



## Content and Licensing

- Book-derived content is CC BY-SA 4.0 (see LICENSE-CONTENT)

- Code is MIT (see LICENSE-CODE)



## Governance

- Architecture proposals submitted via PR using `docs/Architecture-Proposal-Template.md`

- Maintainers review for clarity, feasibility, and alignment with the mission



## Risks

- Over-scoping early phases

- Divergent stack choices without clear maintainership

- Content misalignment with lesson intent



---
