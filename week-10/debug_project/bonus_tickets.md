# MangoMusic JDBC Application - Bonus Feature Tickets

These are four new reports to build from scratch, each harder than the last.
Follow the same three-step process as the original feature tickets:
1. Write and test the SQL query directly in MySQL Workbench
2. Add a method to `ReportsDao.java`
3. Add a menu option and display method to `SpecialReportsScreen.java`

---

## **Bonus Feature #4: Album Completion Leaderboard**
**Difficulty:** Hard
**Priority:** Medium
**Component:** Special Reports
**Requested By:** Content Quality Team

**Business Need:**
The content team wants to know which albums people actually finish versus which ones they start and abandon. A high completion rate signals strong engagement — the kind of albums worth promoting. A low rate might indicate poor audio quality, bad track sequencing, or content that doesn't match listener expectations.

**Feature Description:**
Show the top 25 albums ranked by completion rate. Only include albums with at least 50 total plays so the percentages are meaningful. Display how many plays each album received, how many were completed, and the completion percentage.

**Required Output Columns:**
- `album_title` — Name of the album
- `artist_name` — Name of the artist
- `genre` — Artist's primary genre
- `total_plays` — Total number of times this album was played
- `completed_plays` — Number of plays where `completed = TRUE`
- `completion_rate` — Percentage of plays that were completed, rounded to 1 decimal

**Business Rules:**
- Only include albums with at least 50 total plays
- Order by `completion_rate` descending (highest completion first)
- Limit to top 25 albums

**Technical Requirements:**

1. **SQL Query:** You'll need:
   - Join `album_plays`, `albums`, and `artists`
   - `COUNT(ap.play_id)` for total plays
   - `SUM(CASE WHEN ap.completed = TRUE THEN 1 ELSE 0 END)` for completed plays
   - A `HAVING` clause to enforce the minimum 50-play threshold
   - `ROUND(... * 100.0 / ..., 1)` for the percentage — use `100.0` not `100` to avoid integer division

2. **DAO Method:** Add to `ReportsDao.java`:
```java
public List<ReportResult> getAlbumCompletionLeaderboard()
```

3. **UI Integration:** Update `SpecialReportsScreen.java`:
   - Add menu option: `5. Album Completion Leaderboard`
   - Update `getIntInRange` to accept `0` through `5`
   - Add a `showAlbumCompletionLeaderboard()` private method
   - Display all 6 columns in a formatted table

**Expected Output Format:**
```
Album                              Artist                   Genre      Total Plays  Completed  Rate
Kind of Blue                       Miles Davis              Jazz              312        298  95.5%
Dark Side of the Moon              Pink Floyd               Rock              287        271  94.4%
...
```

**Verification Steps:**
1. Pick an album from the results and manually check with:
```sql
SELECT COUNT(*) as total,
       SUM(CASE WHEN completed = TRUE THEN 1 ELSE 0 END) as done
FROM album_plays
WHERE album_id = [album_id];
```
2. Confirm the ratio matches `completion_rate`
3. Confirm no album in the results has fewer than 50 total plays
4. Confirm results are ordered highest completion rate first

**Hints:**
- `SUM(CASE WHEN ... THEN 1 ELSE 0 END)` is the standard pattern for counting rows that match a condition inside an aggregation — you'll see a similar pattern in `getArtistRevenueReport()`
- The `HAVING` clause filters after grouping — use it here to enforce the 50-play minimum, similar to `getUserDiversityReport()` which uses `HAVING COUNT(ap.play_id) >= 20`
- Don't put the minimum threshold in `WHERE` — `WHERE` runs before grouping and won't have access to the aggregated count

---

## **Bonus Feature #5: Catalog Age vs. Engagement Analysis**
**Difficulty:** Medium-Hard
**Priority:** Low
**Component:** Special Reports
**Requested By:** Content Acquisition Team

**Business Need:**
The acquisition team is debating whether to focus on licensing newer releases or classic catalog titles. They want data: do listeners play recent albums more than older ones? Do they finish them at a higher rate? This report will bucket all albums by release era and show engagement metrics per era.

**Feature Description:**
Group all albums into four eras based on `release_year`. For each era, show how many albums exist, total plays, average plays per album, and the completion rate. This reveals whether modern or vintage content drives more engagement.

**Required Output Columns:**
- `era` — One of four labels (see Business Rules below)
- `album_count` — Number of distinct albums in this era that have been played
- `total_plays` — Total play events across all albums in this era
- `avg_plays_per_album` — `total_plays / album_count`, rounded to 1 decimal
- `completion_rate` — Percentage of plays that were completed, rounded to 1 decimal

**Business Rules:**
- Era labels (based on `al.release_year` compared to the current year):
  - `Recent (Last 5 Years)` — released within the past 5 years
  - `Modern (6-10 Years Ago)` — released 6 to 10 years ago
  - `Classic (11-20 Years Ago)` — released 11 to 20 years ago
  - `Vintage (20+ Years Ago)` — everything older
- Order by `avg_plays_per_album` descending so the most-played era appears first

**Technical Requirements:**

1. **SQL Query:** You'll need:
   - A `CASE` expression on `al.release_year` to produce the `era` label — use `YEAR(CURDATE())` for the current year comparison
   - `COUNT(DISTINCT al.album_id)` for album count
   - `ROUND(COUNT(ap.play_id) * 1.0 / COUNT(DISTINCT al.album_id), 1)` for the average
   - `SUM(CASE WHEN ap.completed = TRUE THEN 1 ELSE 0 END) * 100.0 / COUNT(ap.play_id)` for completion rate
   - Group by the `era` label (you can reference the alias in `GROUP BY` in MySQL)

2. **DAO Method:** Add to `ReportsDao.java`:
```java
public List<ReportResult> getCatalogAgeReport()
```

3. **UI Integration:** Update `SpecialReportsScreen.java`:
   - Add menu option: `6. Catalog Age vs. Engagement`
   - Update `getIntInRange` to accept `0` through `6`
   - Add a `showCatalogAgeReport()` private method
   - Display all 5 columns in a formatted table

**Expected Output Format:**
```
Era                       Albums  Total Plays  Avg Plays/Album  Completion Rate
Modern (6-10 Years Ago)      342       45,231             132.3            87.2%
Recent (Last 5 Years)        198       24,876             125.6            82.4%
Classic (11-20 Years Ago)    415       48,902             117.8            90.1%
Vintage (20+ Years Ago)      521       51,234              98.3            92.5%
```

**Verification Steps:**
1. Pick one era and verify `album_count` manually:
```sql
SELECT COUNT(DISTINCT al.album_id)
FROM albums al
JOIN album_plays ap ON al.album_id = ap.album_id
WHERE al.release_year >= YEAR(CURDATE()) - 10
  AND al.release_year < YEAR(CURDATE()) - 5;
```
2. Confirm the four era labels appear exactly as specified
3. Confirm results are ordered by `avg_plays_per_album` descending

**Hints:**
- In MySQL you can `GROUP BY era` using the alias from your `CASE` expression in the `SELECT` — this keeps the query readable
- `YEAR(CURDATE())` returns the current year as an integer — you can subtract directly: `YEAR(CURDATE()) - 5`
- You'll use two separate `CASE` expressions in the same query: one to label the era, one inside `SUM()` to count completions — see `getArtistRevenueReport()` for a similar multi-CASE pattern

**Query Pattern to Use:**
```sql
SELECT
    CASE
        WHEN al.release_year >= YEAR(CURDATE()) - 5   THEN 'Recent (Last 5 Years)'
        WHEN al.release_year >= YEAR(CURDATE()) - 10  THEN 'Modern (6-10 Years Ago)'
        WHEN al.release_year >= YEAR(CURDATE()) - 20  THEN 'Classic (11-20 Years Ago)'
        ELSE 'Vintage (20+ Years Ago)'
    END AS era,
    COUNT(DISTINCT al.album_id)          AS album_count,
    COUNT(ap.play_id)                    AS total_plays,
    ROUND(COUNT(ap.play_id) * 1.0 / COUNT(DISTINCT al.album_id), 1) AS avg_plays_per_album,
    ROUND(
        SUM(CASE WHEN ap.completed = TRUE THEN 1 ELSE 0 END) * 100.0 / COUNT(ap.play_id),
        1
    ) AS completion_rate
FROM album_plays ap
JOIN albums al ON ap.album_id = al.album_id
GROUP BY era
ORDER BY avg_plays_per_album DESC;
```

---

## **Bonus Feature #6: Genre Loyalty Report**
**Difficulty:** Very Hard
**Priority:** Medium
**Component:** Special Reports
**Requested By:** Artist Relations Team

**Business Need:**
The artist relations team wants to identify users who concentrate the majority of their listening in a single genre — these are the platform's genre loyalists. A user who puts 85% of their plays into Jazz is far more valuable to jazz artists than a casual listener who occasionally samples it. Identifying these users enables targeted promotions, genre-specific playlist features, and loyalty rewards.

**Feature Description:**
For each user with at least 10 total plays, find their single dominant genre (the one with the most plays) and calculate what percentage of their total plays went to that genre. Show the top 50 users ordered by loyalty score.

**Required Output Columns:**
- `user_id` — User identifier
- `username` — Username
- `subscription_type` — Free or premium
- `dominant_genre` — The genre this user plays most
- `plays_in_top_genre` — Play count in their dominant genre
- `total_plays` — Their lifetime play count
- `loyalty_score` — `(plays_in_top_genre / total_plays) * 100`, rounded to 1 decimal

**Business Rules:**
- Only include users with at least 10 total plays
- Show top 50 users ordered by `loyalty_score` descending
- Ties in loyalty score are broken by `total_plays` descending

**Technical Requirements:**

1. **SQL Query:** This is the hardest part. The challenge is finding each user's *single top genre* — you need to GROUP BY user + genre to get play counts per genre, then keep only the row where the genre play count equals the maximum genre play count for that user.

   Use a correlated subquery inside a `HAVING` clause to filter down to only the top genre per user. This is the same rank-by-correlation pattern used in `getMostPlayedAlbumsByGenre()`, but applied at the user level instead of the genre level.

2. **DAO Method:** Add to `ReportsDao.java`:
```java
public List<ReportResult> getGenreLoyaltyReport()
```

3. **UI Integration:** Update `SpecialReportsScreen.java`:
   - Add menu option: `7. Genre Loyalty Report`
   - Update `getIntInRange` to accept `0` through `7`
   - Add a `showGenreLoyaltyReport()` private method
   - Display all 7 columns in a formatted table
   - Consider adding a note like: `Score > 70 = Loyalist | Score 40-70 = Balanced | Score < 40 = Explorer`

**Expected Output Format:**
```
User ID  Username            Subscription  Dominant Genre  Top Genre Plays  Total Plays  Loyalty Score
42       jazz_devotee        premium       Jazz                        187          210           89.0
77       rock_forever        free          Rock                        143          178           80.3
...
```

**Verification Steps:**
1. Pick a user from the results and manually check their genre breakdown:
```sql
SELECT ar.primary_genre, COUNT(*) AS plays
FROM album_plays ap
JOIN albums al ON ap.album_id = al.album_id
JOIN artists ar ON al.artist_id = ar.artist_id
WHERE ap.user_id = [user_id]
GROUP BY ar.primary_genre
ORDER BY plays DESC;
```
2. Confirm the top row matches `dominant_genre` and `plays_in_top_genre`
3. Manually calculate `loyalty_score` and verify it matches
4. Confirm all users have at least 10 total plays

**Hints:**
- You need two derived tables in your `FROM` clause: one that groups by `(user_id, genre)` to get per-genre play counts, one that groups by `user_id` to get total plays — then join them together
- The correlated subquery in `HAVING` should ask: "Is this genre's play count equal to the maximum genre play count for this user?" This isolates only the top-genre row
- Use `100.0` not `100` in the loyalty score division
- Study `getMostPlayedAlbumsByGenre()` closely — the structure of the correlated subquery is very similar

**Query Pattern to Use:**
```sql
SELECT
    u.user_id,
    u.username,
    u.subscription_type,
    genre_data.primary_genre   AS dominant_genre,
    genre_data.genre_plays     AS plays_in_top_genre,
    user_totals.total_plays,
    ROUND(genre_data.genre_plays * 100.0 / user_totals.total_plays, 1) AS loyalty_score
FROM (
    SELECT ap.user_id, ar.primary_genre, COUNT(ap.play_id) AS genre_plays
    FROM album_plays ap
    JOIN albums al ON ap.album_id = al.album_id
    JOIN artists ar ON al.artist_id = ar.artist_id
    GROUP BY ap.user_id, ar.primary_genre
    HAVING COUNT(ap.play_id) = (
        -- Correlated subquery: find this user's maximum genre play count
        -- Fill this in using a nested SELECT MAX(...) FROM (subquery) pattern
    )
) genre_data
JOIN (
    SELECT user_id, COUNT(play_id) AS total_plays
    FROM album_plays
    GROUP BY user_id
    HAVING COUNT(play_id) >= 10
) user_totals ON genre_data.user_id = user_totals.user_id
JOIN users u ON u.user_id = genre_data.user_id
ORDER BY loyalty_score DESC, user_totals.total_plays DESC
LIMIT 50;
```

---

## **Bonus Feature #7: Top 3 Genres Per Country**
**Difficulty:** Very Hard
**Priority:** Medium
**Component:** Special Reports
**Requested By:** International Markets Team

**Business Need:**
The international markets team manages licensing deals country by country. They need to know what genre preferences look like in each market — not just globally, but locally. The top genre in the US may be completely different from the top genre in Brazil or Japan. This report shows the top 3 genres by play volume in every country.

**Feature Description:**
For every country in the database, show its top 3 genres ranked by total play count. This is a top-N-per-group problem: instead of top-5-per-genre (like Bonus Feature #4 in `getMostPlayedAlbumsByGenre()`), it's top-3-per-country.

**Required Output Columns:**
- `country` — Country name
- `genre` — Genre name
- `play_count` — Total plays from users in this country for this genre
- `country_rank` — Rank within the country (1 = most played genre in that country)

**Business Rules:**
- Show only the top 3 genres per country (`country_rank <= 3`)
- Order results by `country` ascending, then `play_count` descending within each country

**Technical Requirements:**

1. **SQL Query:** This follows the same correlated-subquery ranking pattern as `getMostPlayedAlbumsByGenre()`, but with a different dimension. The rank for a given `(country, genre)` row is: how many other genres in the same country have MORE plays than this one, plus 1.

   Wrap the ranking query in an outer `SELECT` that filters `WHERE country_rank <= 3`.

2. **DAO Method:** Add to `ReportsDao.java`:
```java
public List<ReportResult> getTopGenresByCountryReport()
```

3. **UI Integration:** Update `SpecialReportsScreen.java`:
   - Add menu option: `8. Top 3 Genres Per Country`
   - Update `getIntInRange` to accept `0` through `8`
   - Add a `showTopGenresByCountryReport()` private method
   - Display all 4 columns in a formatted table
   - Group output visually by country (print a blank line when the country changes), using the same pattern as `showMostPlayedAlbumsByGenre()`

**Expected Output Format:**
```
Country         Genre           Play Count  Rank
Argentina       Rock                 4,521     1
Argentina       Pop                  3,876     2
Argentina       Latin                2,943     3

Australia       Rock                 6,102     1
Australia       Electronic           5,341     2
Australia       Pop                  4,788     3
...
```

**Verification Steps:**
1. Pick a country and verify its top genre manually:
```sql
SELECT ar.primary_genre, COUNT(*) AS plays
FROM album_plays ap
JOIN albums al ON ap.album_id = al.album_id
JOIN artists ar ON al.artist_id = ar.artist_id
JOIN users u ON ap.user_id = u.user_id
WHERE u.country = 'Australia'
GROUP BY ar.primary_genre
ORDER BY plays DESC
LIMIT 3;
```
2. Confirm the top 3 rows match what the report shows for that country
3. Confirm `country_rank` resets to 1 for each new country
4. Confirm no country shows more than 3 genres

**Hints:**
- The structure of this query is nearly identical to `getMostPlayedAlbumsByGenre()` — read that method carefully before writing this one
- In that report, rank was computed by counting "how many albums in the same genre have more plays than me?" — here you count "how many genres in the same country have more plays than me?"
- You need to join `users` to get `country` in both the outer query and inside the correlated subquery
- The correlated subquery needs to match on `u.country = u_inner.country` to stay within the same country

**Query Pattern to Use:**
```sql
SELECT country, genre, play_count, country_rank
FROM (
    SELECT
        u.country,
        ar.primary_genre AS genre,
        COUNT(ap.play_id) AS play_count,
        (
            -- Correlated subquery: count genres in the same country with MORE plays
            -- Add 1 to get the rank (same pattern as getMostPlayedAlbumsByGenre)
        ) AS country_rank
    FROM album_plays ap
    JOIN albums al  ON ap.album_id  = al.album_id
    JOIN artists ar ON al.artist_id = ar.artist_id
    JOIN users u    ON ap.user_id   = u.user_id
    GROUP BY u.country, ar.primary_genre
) ranked
WHERE country_rank <= 3
ORDER BY country ASC, play_count DESC;
```

---

## **Summary Table**

| Ticket | Method | New SQL Concept |
|--------|--------|-----------------|
| Bonus Feature #4 | `getAlbumCompletionLeaderboard()` | `SUM(CASE WHEN...)`, `HAVING` for minimum threshold |
| Bonus Feature #5 | `getCatalogAgeReport()` | `CASE` bucketing in `GROUP BY`, multiple aggregations per bucket |
| Bonus Feature #6 | `getGenreLoyaltyReport()` | Correlated `HAVING` subquery to isolate top-N-per-user |
| Bonus Feature #7 | `getTopGenresByCountryReport()` | Same correlated ranking pattern across two dimensions |
