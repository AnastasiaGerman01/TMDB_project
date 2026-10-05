# M1 Lab: description of the raw Parquet files

Kseniia Tikhonova, Anastasiia German

## Parquet shapes

| File | Rows | Columns | Key | One row is |
|---|---|---|---|---|
| tables/movies.parquet | 23 981 | 12 | tmdb_id | one movie |
| tables/people.parquet | 57 029 | 2 | person_id | one person (director or actor) |
| tables/credits.parquet | 143 613 | 4 | tmdb_id, person_id, role | one (movie, person, role) |

## Data Acquiring

1. GET /discover/movie with the scope filters, sorted by primary_release_date.asc, 20 movies per page.
   One query per year, because TMDB returns at most 500 pages per query. This gives the movie ids.
2. GET /movie/{id} with append_to_response=keywords,credits: one request per movie gives the details,
   keywords and credits. We keep only the directors (crew with job = Director) and the top-5 billed cast.

## Relations

- credits.tmdb_id → movies.tmdb_id
- credits.person_id → people.person_id
- Checked in the notebook: tmdb_id and person_id are unique, each person has one name, every credit
  points to a person in people.

## Columns

### tables/movies.parquet

| Column | Type | Description |
|---|---|---|
| tmdb_id | int64 | TMDB movie id |
| title | string | Title in English |
| release_date | date | Primary release date |
| original_language | string | ISO 639-1 code of the original language |
| runtime | int64 | Duration in minutes |
| budget | int64 | Budget in US dollars |
| revenue | int64 | Worldwide revenue in US dollars |
| vote_average | double | Average user rating (0–10) |
| vote_count | int64 | Number of user votes |
| popularity | double | TMDB popularity score |
| genres | list of strings | Genre names |
| keywords | list of strings | Keyword names |

### tables/people.parquet

| Column | Type | Description |
|---|---|---|
| person_id | int64 | TMDB person id |
| name | string | Name of the person |

### tables/credits.parquet

| Column | Type | Description |
|---|---|---|
| tmdb_id | int64 | Movie id, links to movies |
| person_id | int64 | Person id, links to people |
| role | string | director or cast |
| cast_order | int64 | Billing position 1–5 for cast, empty for directors |

## Notes

- Raw data: values are stored as returned by the API, no cleaning. Only change: release_date text → date.
- In credits, a person listed twice for the same movie and role is kept once.
- vote_count, vote_average and popularity are snapshots of the download date.
- Personal data: only the TMDB id and name of directors and the top-5 cast are kept. The Parquet files
  stay on our machines and are not submitted.
