```sql
SELECT 
    cte1.hacker_id,
    cte1.name,
    cte1.full_score_count
    
FROM
(SELECT
    h.hacker_id,
    h.name,
    COUNT(s.score) AS full_score_count
FROM
    Hackers h
    JOIN Submissions s ON s.hacker_id = h.hacker_id
    JOIN Challenges c ON c.challenge_id = s.challenge_id
    JOIN Difficulty d ON d.difficulty_level = d.difficulty_level
WHERE
    s.score = d.score
GROUP BY s.hacker_id, h.name
HAVING COUNT(*) > 1) AS cte1
ORDER by cte1.full_score_count DESC, cte1.hacker_id;
```

