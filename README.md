# 🏋️ Fitness Dataset — SQL Analysis

Exploratory SQL analysis on a gym members dataset covering training behavior, body composition, and workout patterns across 973 records.

---

## 📁 Dataset

**Table:** `fitness_workouts`  
**Records:** 973  
**Variables:** 15

| Column | Type | Description |
|---|---|---|
| `age` | INT | Member age in years |
| `gender` | VARCHAR | Male / Female |
| `weight` | FLOAT | Body weight (kg) |
| `height` | FLOAT | Height (cm) |
| `max_bpm` | INT | Maximum heart rate |
| `avg_bpm` | INT | Average workout heart rate |
| `resting_bpm` | INT | Resting heart rate |
| `session_duration` | FLOAT | Session length (hours) |
| `calories_burned` | FLOAT | Calories burned per session |
| `workout_type` | VARCHAR | Cardio / Strength / HIIT / Yoga |
| `fat_percentage` | FLOAT | Body fat percentage |
| `water_intake` | FLOAT | Daily water intake (litres) |
| `workout_frequency` | INT | Sessions per week (2–5) |
| `experience_level` | INT | 1 = Beginner, 2 = Intermediate, 3 = Advanced |
| `bmi` | FLOAT | Body Mass Index |

---

## 🔍 Queries & Findings

### 1. Workout Type Analysis

```sql
SELECT
    workout_type,
    COUNT(*) AS total_sessions,
    ROUND(AVG(session_duration) * 60, 2) AS avg_duration_minutes,
    ROUND(AVG(avg_bpm), 2) AS avg_intensity,
    ROUND(AVG(calories_burned), 2) AS avg_calories,
    ROUND(AVG(avg_bpm) - AVG(resting_bpm), 2) AS avg_heart_rate_reserve
FROM fitness_workouts
GROUP BY workout_type
ORDER BY total_sessions DESC;
```

| Workout Type | Sessions | Avg Duration (min) | Avg BPM | Avg Calories | HR Reserve |
|---|---|---|---|---|---|
| Strength | 258 | 75.61 | 144.31 | 910.70 | 81.85 |
| Cardio | 255 | 73.20 | 143.89 | 884.51 | 81.90 |
| Yoga | 239 | 75.77 | 143.27 | 903.19 | 81.49 |
| HIIT | 221 | 77.22 | 143.52 | 925.81 | 80.84 |

**Key finding:** HIIT burns the most calories per session (925) despite being the least popular workout type. Strength is the most popular but not the most efficient. Intensity (avg_bpm) is virtually identical across all types — less than 1 BPM difference.

---

### 2. Experience Level Analysis

```sql
SELECT
    experience_level,
    COUNT(*) AS total_users,
    ROUND(AVG(session_duration) * 60, 2) AS avg_duration_minutes,
    ROUND(AVG(workout_frequency), 2) AS avg_frequency,
    ROUND(AVG(calories_burned), 2) AS avg_calories,
    ROUND(AVG(fat_percentage), 2) AS avg_fat_pct,
    ROUND(AVG(bmi), 2) AS avg_bmi
FROM fitness_workouts
GROUP BY experience_level
ORDER BY experience_level ASC;
```

| Level | Users | Duration (min) | Frequency | Calories | Fat % | BMI |
|---|---|---|---|---|---|---|
| 1 – Beginner | 376 | 60.61 | 2.48 | 726.38 | 27.63 | 24.62 |
| 2 – Intermediate | 406 | 74.87 | 3.53 | 901.92 | 27.31 | 25.26 |
| 3 – Advanced | 191 | 105.56 | 4.53 | 1265.34 | 14.79 | 24.75 |

**Key finding:** Fat % is nearly identical between Levels 1 and 2 (27.63 vs 27.31), then drops off a cliff at Level 3 (14.79%). Level 3 athletes burn 74% more calories per session than beginners. BMI stays flat across levels — the Level 3 difference is muscle mass, not just lower weight.

---

### 3. Gender Analysis

```sql
SELECT
    gender,
    ROUND(AVG(session_duration) * 60, 2) AS avg_duration_minutes,
    ROUND(AVG(avg_bpm)) AS avg_intensity,
    ROUND(AVG(workout_frequency), 2) AS avg_frequency,
    ROUND(AVG(calories_burned), 2) AS avg_calories,
    ROUND(AVG(fat_percentage), 2) AS avg_fat_pct,
    ROUND(AVG(bmi), 2) AS avg_bmi,
    ROUND(AVG(weight), 2) AS avg_weight,
    COUNT(*) AS total_users
FROM fitness_workouts
GROUP BY gender;
```

| Gender | Duration (min) | Avg BPM | Frequency | Calories | Fat % | BMI | Weight (kg) | Users |
|---|---|---|---|---|---|---|---|---|
| Male | 75.15 | 144 | 3.31 | 944.46 | 22.55 | 26.89 | 85.53 | 511 |
| Female | 75.65 | 144 | 3.34 | 862.25 | 27.66 | 22.73 | 60.94 | 462 |

**Key finding:** Training behavior is identical across genders — same duration, intensity, and frequency. The calorie gap is explained by weight difference. The fat % difference is biological, not a fitness gap. Males have higher BMI but lower fat % — a muscle mass effect.

---

### 4. Workout Frequency Analysis

```sql
SELECT
    workout_frequency,
    ROUND(AVG(calories_burned), 2) AS avg_calories,
    ROUND(AVG(fat_percentage), 2) AS avg_fat_pct,
    COUNT(*) AS total_users
FROM fitness_workouts
GROUP BY workout_frequency
ORDER BY workout_frequency;
```

| Frequency (days/week) | Avg Calories | Avg Fat % | Users |
|---|---|---|---|
| 2 | 726.38 | 27.44 | 197 |
| 3 | 821.44 | 27.59 | 368 |
| 4 | 997.64 | 23.69 | 306 |
| 5 | 1277.57 | 14.66 | 102 |

**Key finding:** Training 2–3x/week produces nearly identical fat % (~27.5%) — there is no meaningful body composition difference. The real shift happens at 4x/week (23.69%) and craters at 5x/week (14.66%). There is a frequency threshold, not a linear progression.

---

### 5. Workout Type × Experience Level

```sql
SELECT
    experience_level,
    workout_type,
    COUNT(*) AS total_sessions,
    ROUND(AVG(calories_burned), 2) AS avg_calories,
    ROUND(AVG(fat_percentage), 2) AS avg_fat_pct
FROM fitness_workouts
GROUP BY experience_level, workout_type
ORDER BY experience_level, total_sessions DESC;
```

| Level | Workout Type | Sessions | Avg Calories | Avg Fat % |
|---|---|---|---|---|
| 1 | Cardio | 109 | 706.08 | 27.71 |
| 1 | Strength | 97 | 750.39 | 27.99 |
| 1 | HIIT | 85 | 744.71 | 27.54 |
| 1 | Yoga | 85 | 706.66 | 27.22 |
| 2 | Strength | 116 | 898.12 | 27.43 |
| 2 | Cardio | 102 | 929.69 | 27.41 |
| 2 | Yoga | 101 | 881.50 | 27.01 |
| 2 | HIIT | 87 | 898.14 | 27.37 |
| 3 | Yoga | 53 | 1259.72 | 15.28 |
| 3 | HIIT | 49 | 1289.08 | 13.97 |
| 3 | Strength | 45 | 1288.67 | 14.93 |
| 3 | Cardio | 44 | 1221.82 | 15.00 |

**Key finding:** Workout type is irrelevant at Levels 1 and 2 — fat % is flat at ~27% regardless of what you do. A Level 3 Yoga session (1,259 cal, 15.28% fat) outperforms a Level 1 HIIT session (744 cal, 27.54% fat) in every metric. The athlete matters more than the workout.

---

## 📊 Key Findings Summary

| # | Finding | Evidence |
|---|---|---|
| 1 | **Experience Level 3 is a different athlete class** | Fat %: 27.6% → 27.3% → 14.8% |
| 2 | **Frequency threshold effect — real change only at 4–5x/week** | Fat %: 27.5% (2–3x) vs 14.66% (5x) |
| 3 | **Gender differences are biological, not behavioral** | Duration & frequency virtually identical |
| 4 | **HIIT is most efficient; Strength is most popular** | HIIT: 925 cal vs Strength: 910 cal |
| 5 | **The athlete determines outcomes, not the workout type** | Level 3 Yoga > Level 1 HIIT on all metrics |

---

## ⚠️ Data Quality Notes

The following columns showed no analytical value and were excluded from conclusions:

| Column | Issue |
|---|---|
| `resting_bpm` | Flat at 62 across all segments — no variance |
| `avg_bpm / max_bpm` | Flat at ~80% across all experience levels |
| `age` | All metrics flat across age groups |
| `water_intake` | No clean dose-response relationship |
| `bmi` | Decoupled from `fat_percentage` — obese users show lower fat % than normal BMI users |

> The two variables that drive all meaningful patterns are `experience_level` and `workout_frequency`. They likely measure the same underlying construct: the serious vs. casual athlete.

---

## 🛠️ Tools Used

- **MySQL** — querying and aggregation


---
