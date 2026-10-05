# M1 Lab: description of the raw Parquet files

Kseniia Tikhonova, Anastasiia German

## Parquet shapes

| File | Rows | Columns | Key | One row is |
|---|---|---|---|---|
| tables/movies.parquet | 23 981 | 12 | tmdb_id | one movie |
| tables/people.parquet | 57 029 | 2 | person_id | one person (director or actor) |
| tables/credits.parquet | 143 613 | 4 | tmdb_id, person_id, role | one (movie, person, role) |

## Data Acquiring

1. First, we call GET /discover/movie with our scope filters, sorting the results by release date (primary_release_date.asc). Each page contains 20 movies. However, TMDB returns no more than 500 pages per query, so we run a separate query for each year to stay within that limit. As a result, this step gives us the full list of movie IDs.
2. Next, for each movie we call GET /movie/{id} with append_to_response=keywords,credits. This way, a single request returns the movie's details, keywords and credits all at once. Finally, from the credits we keep only the directors (crew members whose job is Director) and, in addition, the top five billed cast members.

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