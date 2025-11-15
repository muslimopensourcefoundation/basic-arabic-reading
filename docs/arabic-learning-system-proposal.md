# Arabic Learning Platform [Lesson 1 - The Arabic Alphabet]

*An NLP based basic Arabic teaching system powered by python and typescript.* 

labels: ["architecture", "lesson1", "TheArabicAlphabet", "NLP"]

## Tech Stack:
   - **Python** is suitable for the backend development of this project as it will require a-lot of natural language processing, model training specifically for pronunciation correction and utilizing speech-to-text. 
   - For APIs we can use **FastAPI** framework of python for robustness and extended async support. 
   - NextJS with Typescript will be used in creating a suitable frontend along with tailwind css. 
   - Initially we should make this open-source, once it gain audience then we can move towards stable container deployment like **GCP** Cloud Run OR **AWS** EC2.
   - As this is going to be Read-first system so we should use **NoSQL** database like mongoDB for high availability.

## High-level design:
  *Proposed High-Level Design for lesson 1 - The Arabic Alphabets.*
  -    System generated voice for teaching alphabet pronunciation.
       - Can hear all the alphabets at once or single alphabet per click action.
       - Pre-recorded voice will be played to save on compute.
       - Can add multiple Arabic dialects for user to choose.
  -  For test / practicing the Arabic Speaking, the system will hear the user's voice and try to match it with pre-recorded alphabets.
  - System matches user voice and evaluate some score, and mark it correct or incorrect based on threshold value set.
  - An exam for complete speaking of all the alphabets in order.
  - Once user passes this can move on to next lesson. 

## API Specification

**Base Path:** `/api/v1`

---

### Auth

- ##### POST `/auth/signup`
  Create a new user account.

- ##### POST `/auth/signin`
  Authenticate user and return access token.

- ##### GET `/auth/me`
  Retrieve current authenticated user details.

---

### Lessons

- ##### GET `/lessons`
  List all lessons with optional filters.

  ##### **Query Params:**
  - `status` — filter by progress (e.g. `completed`, `in-progress`)
  - `language` — filter by language (e.g. `arabic`)

- ##### GET `/lessons/:slug`
  Get detailed information for a specific lesson, including alphabets, exercises, and available dialects.

---

### Alphabets

- ##### GET `/lessons/:slug/alphabets`
  Retrieve list of Arabic alphabets with associated pronunciation files.

  ##### **Query Params:**
  - `dialect` — (optional) dialect selection (e.g. `gulf`, `egyptian`, `levantine`)

- ##### GET `/lessons/:slug/alphabets/:id`
  Get single alphabet details, including pronunciation audio and description.

---

### Pronunciation Practice

- ##### GET `/lessons/:slug/practice/audio/:id`
  Stream or download pre-recorded audio for a given alphabet pronunciation.

- ##### POST `/lessons/:slug/practice/record`
  Upload user’s recorded pronunciation for evaluation.

- ##### **Request (multipart/form-data):**
  - `file` — user’s recorded voice
  - `alphabet_id` — alphabet being practiced

### Data Model

#### User
| Field | Type | Description |
|--------|------|-------------|
| id | UUID | Unique user identifier |
| name | String | Full name of the user |
| email | String | User email (unique) |
| password_hash | String | Hashed password |
| created_at | DateTime | Account creation timestamp |
| updated_at | DateTime | Last update timestamp |

---

#### Lesson
| Field | Type | Description |
|--------|------|-------------|
| id | UUID | Unique lesson identifier |
| slug | String | URL-friendly lesson identifier |
| title | String | Lesson title (e.g. "Arabic Alphabets") |
| description | Text | Lesson description |
| order | Integer | Order of appearance in course |
| is_active | Boolean | Whether lesson is active or hidden |
| created_at | DateTime | Creation timestamp |

---

#### Alphabet
| Field | Type | Description |
|--------|------|-------------|
| id | UUID | Unique alphabet identifier |
| lesson_id | UUID (FK → Lesson.id) | Associated lesson |
| character | String | Arabic character (e.g. "ا") |
| transliteration | String | Latin transliteration (e.g. "Alif") |
| audio_urls | JSON | Map of dialect → pre-recorded audio URLs |
| order | Integer | Position in alphabet sequence |

---

#### Dialect
| Field | Type | Description |
|--------|------|-------------|
| id | UUID | Unique dialect identifier |
| name | String | Name of dialect (e.g. "Gulf Arabic") |
| code | String | Dialect code (e.g. `gulf`, `egyptian`) |
| description | Text | Optional description |

---

#### PronunciationAttempt
| Field | Type | Description |
|--------|------|-------------|
| id | UUID | Unique attempt identifier |
| user_id | UUID (FK → User.id) | User who attempted |
| alphabet_id | UUID (FK → Alphabet.id) | Target alphabet |
| dialect_id | UUID (FK → Dialect.id) | Dialect used for evaluation |
| audio_url | String | User’s uploaded audio file |
| score | Float | Confidence score from voice matching |
| threshold | Float | Threshold for correctness |
| result | Enum(`correct`, `incorrect`) | Evaluation result |
| created_at | DateTime | Attempt timestamp |

---

#### ExamSession
| Field | Type | Description |
|--------|------|-------------|
| id | UUID | Unique exam session identifier |
| user_id | UUID (FK → User.id) | Exam taker |
| lesson_id | UUID (FK → Lesson.id) | Lesson under evaluation |
| started_at | DateTime | Session start time |
| completed_at | DateTime | Session end time |
| overall_score | Float | Aggregated score |
| passed | Boolean | Whether user passed the exam |

---

#### ExamResultDetail
| Field | Type | Description |
|--------|------|-------------|
| id | UUID | Unique record ID |
| exam_id | UUID (FK → ExamSession.id) | Related exam session |
| alphabet_id | UUID (FK → Alphabet.id) | Evaluated alphabet |
| score | Float | Score for this alphabet |
| result | Enum(`correct`, `incorrect`) | Evaluation result |

---

#### UserProgress
| Field | Type | Description |
|--------|------|-------------|
| id | UUID | Unique record ID |
| user_id | UUID (FK → User.id) | Related user |
| lesson_id | UUID (FK → Lesson.id) | Related lesson |
| status | Enum(`not_started`, `in_progress`, `completed`) | Current progress state |
| last_exam_id | UUID (FK → ExamSession.id) | Last exam session reference |
| updated_at | DateTime | Last progress update |


---
