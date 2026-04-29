from collections import deque
initial_movies = {
    1: {"title": "The Matrix", "genres": ["Sci-Fi", "Action"], "sum_rating": 480, "count": 100},
    2: {"title": "Inception", "genres": ["Sci-Fi", "Action", "Thriller"], "sum_rating": 450, "count": 95},
    3: {"title": "Interstellar", "genres": ["Sci-Fi", "Drama"], "sum_rating": 470, "count": 100},
    4: {"title": "The Godfather", "genres": ["Crime", "Drama"], "sum_rating": 490, "count": 100},
    5: {"title": "Pulp Fiction", "genres": ["Crime", "Thriller"], "sum_rating": 430, "count": 90},
    6: {"title": "The Dark Knight", "genres": ["Action", "Crime", "Drama"], "sum_rating": 485, "count": 100},
    7: {"title": "Schindler's List", "genres": ["Drama", "History"], "sum_rating": 460, "count": 95},
    8: {"title": "Forrest Gump", "genres": ["Drama", "Romance"], "sum_rating": 440, "count": 100},
    9: {"title": "The Shawshank Redemption", "genres": ["Drama"], "sum_rating": 495, "count": 100},
    10: {"title": "Gladiator", "genres": ["Action", "Adventure", "Drama"], "sum_rating": 420, "count": 90},
    11: {"title": "Alien", "genres": ["Sci-Fi", "Horror"], "sum_rating": 380, "count": 85},
    12: {"title": "Silence of the Lambs", "genres": ["Crime", "Horror", "Thriller"], "sum_rating": 410, "count": 90},
    13: {"title": "Seven", "genres": ["Crime", "Mystery", "Thriller"], "sum_rating": 390, "count": 85},
    14: {"title": "The Prestige", "genres": ["Drama", "Mystery", "Sci-Fi"], "sum_rating": 445, "count": 95},
    15: {"title": "Memento", "genres": ["Mystery", "Thriller"], "sum_rating": 405, "count": 90},
    16: {"title": "The Lion King", "genres": ["Animation", "Adventure", "Drama"], "sum_rating": 475, "count": 100},
    17: {"title": "Spirited Away", "genres": ["Animation", "Adventure", "Fantasy"], "sum_rating": 480, "count": 100},
    18: {"title": "Back to the Future", "genres": ["Sci-Fi", "Adventure", "Comedy"], "sum_rating": 455, "count": 100},
    19: {"title": "Blade Runner 2049", "genres": ["Sci-Fi", "Drama"], "sum_rating": 340, "count": 80},
    20: {"title": "Parasite", "genres": ["Drama", "Thriller", "Comedy"], "sum_rating": 465, "count": 100}
}

movies = {}
for movie_id, movie_data in initial_movies.items():
    movies[movie_id] = {
        "title": movie_data["title"],
        "genres": set(movie_data["genres"]),
        "rating_sum": movie_data["sum_rating"],
        "rating_count": movie_data["count"]
    }

user_ratings = {}
view_history = deque(maxlen=5)

def calculate_average_rating(movie_id):
    movie = movies[movie_id]
    return movie["rating_sum"] / movie["rating_count"]

def build_graph():
    graph = {movie_id: set() for movie_id in movies}
    movie_ids = list(movies.keys())
    for i in range(len(movie_ids)):
        for j in range(i + 1, len(movie_ids)):
            first_movie = movie_ids[i]
            second_movie = movie_ids[j]
            if movies[first_movie]["genres"] & movies[second_movie]["genres"]:
                graph[first_movie].add(second_movie)
                graph[second_movie].add(first_movie)
    return graph
graph = build_graph()

def rate_movie(movie_id, rating):
    if movie_id not in movies:
        print("THE MOVIE ID MUST INCLUDE IN 1-20!")
        return
    if movie_id in user_ratings:
        previous_rating = user_ratings[movie_id]
        movies[movie_id]["rating_sum"] -= previous_rating
    else:
        movies[movie_id]["rating_count"] += 1
    user_ratings[movie_id] = rating
    movies[movie_id]["rating_sum"] += rating

def watch_movie(movie_id):
    if movie_id not in movies:
        print("Invalid movie ID")
        return
    view_history.append(movie_id)
    print(f" YOU ARE NOW Watching THE MOVIE ENJOY: {movies[movie_id]['title']}")

def list_movies():
    print("\nMOVIE LIST")
    for movie_id, movie in movies.items():
        genres = ", ".join(movie["genres"])
        print(
            f"{movie_id}. {movie['title']} "
            f"[Genres: {genres}] "
            f"(Rating: {calculate_average_rating(movie_id):.2f})"
        )

def show_history():
    print("\nVIEW HISTORY")
    print(f"{'TITLE':<30} {'GENRES':<40} {'RATING':<10}")
    print("-" * 80)
    for movie_id in view_history:
        movie = movies[movie_id]
        genres = ", ".join(movie["genres"])
        avg_rating = calculate_average_rating(movie_id)
        print(f"{movie['title']:<30} {genres:<40} {avg_rating:<10.2f}")

def top_5_movies():
    print("\nTOP 5 MOVIES")
    print(f"{'TITLE':<35} {'RATING':<10}")
    print("-" * 50)
    top_movies = sorted(
        movies.keys(),
        key=lambda movie_id: calculate_average_rating(movie_id),
        reverse=True
    )[:5]
    for movie_id in top_movies:
        print(f"{movies[movie_id]['title']:<35} "
              f"{calculate_average_rating(movie_id):<10.2f}")

def bfs_recommend(start_movie):
    queue = deque([start_movie])
    visited = {start_movie}
    recommendations = []
    watched_movies = set(view_history)
    while queue:
        current_movie = queue.popleft()
        for neighbor in graph[current_movie]:
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
                if neighbor not in watched_movies:
                    recommendations.append(neighbor)
    return recommendations

def recommend():
    if not view_history:
        print("\nTOP PICKS (No history)")
        top_movies = sorted(
            movies.keys(),
            key=lambda movie_id: calculate_average_rating(movie_id),
            reverse=True
        )[:5]
        for movie_id in top_movies:
            print(movies[movie_id]["title"])
        return
    watched_movies = set(view_history)
    all_recommendations = set()
    for movie_id in view_history:
        all_recommendations.update(bfs_recommend(movie_id))
    final_recommendations = [
        movie_id for movie_id in all_recommendations
        if movie_id not in watched_movies
    ]
    final_recommendations.sort(
        key=lambda movie_id: calculate_average_rating(movie_id),
        reverse=True
    )
    print("\nRECOMMENDATIONS")
    for movie_id in final_recommendations[:10]:
        print(f"{movies[movie_id]['title']} "
              f"(Rating: {calculate_average_rating(movie_id):.2f})")

def show_menu():
    print("\n[========================================]")
    print("      MOVIE RECOMMENDER SYSTEM BY HADI")
    print("[=========================================]")
    print("WELLCOME TO NETFLIX MOVIE PLATFORM!")
    print("1. List of all Movies")
    print("2. Watch a Movie")
    print("3. Rate a Movie")
    print("4. View the History")
    print("5. Top 5 Movies")
    print("6. Recommend Movies")
    print("0. Exit")

while True:
    show_menu()
    user_choice = input("Enter choice: ")
    if user_choice == "1":
        list_movies()
    elif user_choice == "2":
        try:
            movie_id = int(input("Movie ID: "))
            watch_movie(movie_id)
        except ValueError:
            print("Invalid input")
    elif user_choice == "3":
        try:
            movie_id = int(input("Movie ID: "))
            rating = float(input("Rating (1-10): "))
            rate_movie(movie_id, rating)
        except ValueError:
            print("Invalid input")
    elif user_choice == "4":
        show_history()
    elif user_choice == "5":
        top_5_movies()
    elif user_choice == "6":
        recommend()
    elif user_choice == "0":
        print("Goodbye!")
        break
    else:
        print("Invalid input")
