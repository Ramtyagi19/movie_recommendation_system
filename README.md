import pandas as pd
import streamlit as st
import pickle
import requests


# Fetch poster using TMDB API
def fetch_poster_path(movie_id):
    url  = f"https://api.themoviedb.org/3/movie/{''}?api_key=0642e7b5ee53d3e80070852fc4d2bfe2&language=en-US"
    response = requests.get(url)
    data = response.json()
    print(data)
    return "https://image.tmdb.org/t/p/w500" + data['poster_path']

def fetch_poster_path(movie_id):
    response = requests.get(f"https://api.themoviedb.org/3/movie/{movie_id}?api_key=0642e7b5ee53d3e80070852fc4d2bfe2&language=en-US")
    data = response.json()
    if 'poster_path' in data and data['poster_path']:
        return "https://image.tmdb.org/t/p/w500" + data['poster_path']
    else:
        return "https://via.placeholder.com/500x750?text=No+Image"


# Load data
movie_dict = pickle.load(open('movie_dict.pkl', 'rb'))
movie = pd.DataFrame(movie_dict)

similarity = pickle.load(open('similarity.pkl', 'rb'))  # similarity matrix

# Recommend function
def recommend(movie_name):
    movie_index = movie[movie['title'] == movie_name].index[0]
    distances = similarity[movie_index]
    movie_list = sorted(list(enumerate(distances)), reverse=True, key=lambda x: x[1])

    recommended_movies = []
    recommended_movies_poster = []
    for i in movie_list[1:6]:  # Top 5 similar movies
        movie_id = movie.iloc[i[0]]['id']
        recommended_movies.append(movie.iloc[i[0]].title)
        recommended_movies_poster.append(fetch_poster_path(movie_id))
    return recommended_movies, recommended_movies_poster

# Streamlit UI
st.title("🎬 Movie Recommender System")

selected_movie_name = st.selectbox(
    "Select a movie to get recommendations:",
    movie['title'].values
)

if st.button('Recommend'):
    names, posters = recommend(selected_movie_name)
    st.subheader("Recommended Movies:")

    col1, col2, col3, col4, col5 = st.columns(5)

    with col1:
        st.text(names[0])
        st.image(posters[0])
    with col2:
        st.text(names[1])
        st.image(posters[1])
    with col3:
        st.text(names[2])
        st.image(posters[2])
    with col4:
        st.text(names[3])
        st.image(posters[3])
    with col5:
        st.text(names[4])
        st.image(posters[4])
