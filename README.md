# 🎬 Movie Recommendation System

A movie recommendation system built with **User-Based Collaborative Filtering** and **Popularity-Based Filtering**, developed as a course project for **Artificial Intelligence** at the **Faculty of Computer and Information Technology, King Abdulaziz University**.

The system uses the [MovieLens](https://grouplens.org/datasets/movielens/) user–movie ratings dataset to recommend movies to a target user based on the preferences of similar users, and to surface popular/trending movies for new users with no rating history.

## Table of Contents

- [Overview](#overview)
- [Dataset](#dataset)
- [Project Workflow](#project-workflow)
  - [1. Data Exploration & Cleaning](#1-data-exploration--cleaning)
  - [2. User-Based Collaborative Filtering](#2-user-based-collaborative-filtering)
  - [3. Popularity-Based Filtering](#3-popularity-based-filtering)
- [Repository Structure](#repository-structure)
- [Getting Started](#getting-started)
- [Sample Output](#sample-output)
- [Key Findings](#key-findings)
- [Limitations & Future Work](#limitations--future-work)
- [Tech Stack](#tech-stack)
- [Team](#team)

## Overview

Recommendation systems are a core application of AI, used everywhere from e-commerce to streaming platforms to suggest relevant items to users. This project explores the fundamentals of building one from scratch: data collection, cleaning, exploratory data analysis, and two recommendation approaches — **collaborative filtering** and **popularity-based filtering** — applied to a movie ratings dataset as a proxy for a product recommendation use case (e.g., an e-commerce platform).

The project was completed in three milestones:

| Task | Description |
|------|-------------|
| Task 1 | Dataset selection and research on recommendation system fundamentals |
| Task 2 | Data understanding, cleaning, and exploratory data analysis |
| Task 3 | Implementation of collaborative filtering and popularity-based filtering |

## Dataset

The project uses the **MovieLens** user-movie interaction dataset, sourced from [GroupLens](https://grouplens.org/datasets/movielens/) and GitHub, consisting of two files:

- **`ratings_movies.csv`** — ~100,836 user ratings
  - `userId`, `movieId`, `rating` (0.5–5.0, in 0.5 increments), `timestamp`
- **`movies.csv`** — movie metadata used to map IDs to titles
  - `movieId`, `title`, `genres`

**Dataset stats:**
- 610 unique users
- 9,724 unique movies
- 100,836 ratings
- No missing values or duplicate rows

## Project Workflow

### 1. Data Exploration & Cleaning

- Checked for missing values and duplicates (dataset was already clean).
- Merged `ratings_movies.csv` with `movies.csv` on `movieId` to attach movie titles (the raw ratings file only contains movie IDs).
- Computed descriptive statistics (mean, median, variance, standard deviation) and visualized the rating distribution with histograms and boxplots.
- Analyzed interaction distribution: top users by number of ratings, and top movies by number of ratings.
- Built a correlation matrix between a movie's average rating and its number of ratings, finding a weak positive correlation (~0.127) — more popular movies tend to be rated slightly higher.
- Noted a rating bias: most ratings cluster around 3.5–4.0, with far fewer 1–2.5 star ratings.

### 2. User-Based Collaborative Filtering

The core recommendation engine, based on the assumption that **users with similar tastes will like similar movies**. Steps:

1. **Add movie titles** — merge ratings with movie metadata.
2. **Filter for scale** — the full 610 × 9,724 user–movie matrix was too large to build in memory (`MemoryError: unable to allocate ~13–30 GiB`). Movies with **≤100 ratings** were dropped, reducing the dataset to 134 movies and ~597 users — similar to how platforms limit recommendations to popular content.
3. **Build the User–Movie matrix** — a pivot table with users as rows, movie titles as columns, and ratings as values (`NaN` where a user hasn't rated a movie).
4. **Normalize ratings** — subtract each user's average rating from their ratings, since some users rate more generously than others.
5. **Compute user similarity** — a user × user similarity matrix using **Pearson correlation** on the normalized matrix (1.0 = identical taste, -1.0 = opposite taste).
6. **Select similar users** — for a target user, pick the top *N* most similar users above a similarity threshold (e.g., top 10 users with similarity > 0.3).
7. **Narrow the item pool** — remove movies the target user has already watched, then keep only movies that at least one similar user has watched.
8. **Score and recommend** — compute a similarity-weighted average rating for each candidate movie and recommend the top-scoring movies to the target user.

### 3. Popularity-Based Filtering

A simple, cold-start-friendly baseline used to recommend to new users with no rating history:

- **Overall popularity** — movies ranked by total number of ratings received.
- **Average rating popularity** — movies ranked by average rating.

## Repository Structure

```
.
├── Recom2.ipynb                     # Main Jupyter notebook (full pipeline)
├── movies.csv                       # Movie metadata (movieId, title, genres)
├── ratings_movies.csv               # User ratings (userId, movieId, rating, timestamp)
├── Recommendation_System_0_3.pdf    # Full project report
├── Recommendation_System.pptx       # Project presentation
└── README.md
```

## Getting Started

### Prerequisites

- Python 3.x
- pandas
- numpy
- matplotlib
- seaborn

### Installation

```bash
git clone <repository-url>
cd <repository-folder>
pip install pandas numpy matplotlib seaborn
```

### Usage

1. Make sure `movies.csv` and `ratings_movies.csv` are in the same directory as the notebook.
2. Open and run `Recom2.ipynb` in Jupyter Notebook / JupyterLab.
3. Run the cells sequentially — the notebook walks through data loading, EDA, collaborative filtering, and popularity-based filtering.
4. To get recommendations for a different user, change the `picked_userid` variable in the collaborative filtering section.

## Sample Output

Top recommendations generated for `picked_userid = 1` via user-based collaborative filtering:

| Movie | Score |
|---|---|
| Harry Potter and the Chamber of Secrets (2002) | 1.89 |
| Eternal Sunshine of the Spotless Mind (2004) | 1.89 |
| The Bourne Identity (2002) | 0.89 |
| Ocean's Eleven (2001) | 0.89 |
| Inception (2010) | 0.59 |

## Key Findings

- The dataset was clean and complete, with no missing values, making it well-suited for collaborative filtering without an imputation strategy for ratings themselves (only movie titles needed to be joined in from a separate file).
- Ratings skew toward the middle-to-high range (3.5–4 stars), indicating a mild positivity bias in user ratings.
- Popularity and average rating are weakly positively correlated — popular movies aren't guaranteed to be highly rated, but there's a mild trend in that direction.
- Building a full user–item matrix at scale requires dimensionality reduction (e.g., filtering out low-interaction items) to remain computationally feasible on a personal machine.

## Limitations & Future Work

- **Content-based filtering** was scoped out due to time constraints, but is a natural next step, especially using the `genres` field.
- **Cold-start problem** for new users is currently only partially addressed via popularity-based filtering; a hybrid model combining both approaches would generalize better.
- The collaborative filtering matrix currently only includes movies with more than 100 ratings; a more scalable approach (e.g., matrix factorization/SVD, or sparse matrix representations) would allow the full catalog to be used.
- No formal offline evaluation (e.g., RMSE, precision@k on a train/test split) was performed — this would be a valuable addition for measuring recommendation quality.

## Tech Stack

- **Language:** Python
- **Libraries:** pandas, numpy, matplotlib, seaborn
- **Techniques:** Pearson correlation, mean-centering normalization, pivot tables

## Team

- Turki Khalid Alqahtani
- Rayan Yosof Fadhil
- Raed Ahmed Alsafry
- Omar Waleed Alsiary