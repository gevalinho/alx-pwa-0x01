# alx-project-0x14
API Explorer: Mastering RESTful Connections

## API Overview

The TMDb (The Movie Database) API provides access to a vast, community-supported database of movies, TV shows, and people (actors, directors, etc.). You can retrieve metadata (titles, overviews, genres, release dates), images, credits (cast & crew), reviews, ratings, and search/discover content using various filters. 
The Movie Database (TMDB)


It’s useful for building movie/TV web apps, recommendation systems, streaming guides, or any app needing entertainment metadata.

## Version

The “3” in TMDb’s base path indicates version v3 of the API. 
The Movie Database (TMDB)


TMDb also has a newer v4 API (for some endpoints) documented separately. 
The Movie Database (TMDB)

## Available Endpoints

Below are some of the primary (and commonly-used) endpoints provided by TMDb. (This is not exhaustive — the official docs list many more.) 
The Movie Database (TMDB)


Endpoint	HTTP Method	Purpose / Description
/movie/{movie_id}	GET	Get detailed information about a movie by its ID (title, overview, runtime, etc.) 
The Movie Database (TMDB)


/movie/{movie_id}/credits	GET	Get cast and crew details (actors, director, etc.) for a movie 
The Movie Database (TMDB)


/search/movie	GET	Search for movies by title, with query parameters like page, year, etc. 
The Movie Database (TMDB)


/discover/movie	GET	Discover movies by filters (genre, vote average, release date, etc.) 
launchschool.com


/movie/{movie_id}/rating	POST / DELETE	Submit or delete a user rating for a movie (requires authentication) 
launchschool.com


/genre/movie/list	GET	List all movie genres (with IDs) 
The Movie Database (TMDB)


/tv/{tv_id}	GET	Get detailed info about a TV show by id 
The Movie Database (TMDB)


/search/tv	GET	Search for TV shows by name 
The Movie Database (TMDB)


/person/{person_id}	GET	Get detailed info about a person (actor, crew) 
The Movie Database (TMDB)


/search/multi	GET	Perform a “multi” search across movies, TV, and people in one query 
The Movie Database (TMDB)


You should check the official docs for more endpoints (e.g. images, recommendations, similar, changes, lists, external IDs) 
The Movie Database (TMDB)


Request and Response Format
Base URL & Common Prefix

Most v3 API requests use a base like:

https://api.themoviedb.org/3/


You then append the endpoint path (e.g. movie/550) and include query parameters. 
The Movie Database (TMDB)


All responses are JSON-formatted.

Example: Request /movie/{movie_id}

Request

GET https://api.themoviedb.org/3/movie/550  
Authorization: Bearer YOUR_ACCESS_TOKEN  
Accept: application/json  


Sample Response (abridged)

{
  "adult": false,
  "backdrop_path": "/some_path.jpg",
  "belongs_to_collection": null,
  "budget": 63000000,
  "genres": [
    { "id": 18, "name": "Drama" }
  ],
  "homepage": "http://someurl.com",
  "id": 550,
  "imdb_id": "tt0137523",
  "original_language": "en",
  "original_title": "Fight Club",
  "overview": "A depressed man …",
  "popularity": 45.3,
  "poster_path": "/poster.jpg",
  "production_companies": [ /* … */ ],
  "release_date": "1999-10-15",
  "revenue": 100853753,
  "runtime": 139,
  "spoken_languages": [ /* … */ ],
  "status": "Released",
  "tagline": "Mischief. Mayhem. Soap.",
  "title": "Fight Club",
  "vote_average": 8.4,
  "vote_count": 34321
}


Note that some fields are optional or may appear as null depending on the movie. Also, some fields (like poster_path or backdrop_path) provide only a partial path; you often need to prepend a base image URL (available in configuration endpoints) to build full image URLs. 
launchschool.com


Example: Request /movie/{movie_id}/credits

Request

GET https://api.themoviedb.org/3/movie/550/credits  
Authorization: Bearer YOUR_ACCESS_TOKEN  
Accept: application/json  


Sample Response (abridged)

{
  "id": 550,
  "cast": [
    {
      "cast_id": 4,
      "character": "The Narrator",
      "credit_id": "52fe4250c3a36847f80149f3",
      "gender": 2,
      "id": 819,
      "name": "Edward Norton",
      "order": 0,
      "profile_path": "/path.jpg"
    },
    /* more cast entries */
  ],
  "crew": [
    {
      "credit_id": "52fe4250c3a36847f80149bf",
      "department": "Directing",
      "gender": 2,
      "id": 7467,
      "job": "Director",
      "name": "David Fincher",
      "profile_path": "/path2.jpg"
    },
    /* more crew entries */
  ]
}


From crew, you can filter where job == "Director" to find the director(s). 
launchschool.com


Notes on Error Responses

When something goes wrong (e.g. bad API key, resource not found, invalid request), TMDb returns an HTTP status code plus a JSON object describing the error. For example:

{
  "status_code": 34,
  "status_message": "The resource you requested could not be found.",
  "success": false
}


You should check the HTTP status first (e.g. 401, 404, 429, etc.), then inspect status_code and status_message in the body to get more specific error info. 
launchschool.com


Authentication

You need a valid API Read Access Token (or API key) to make requests. 
The Movie Database (TMDB)


Typically, you include it in the Authorization header as a Bearer token, e.g.

Authorization: Bearer YOUR_ACCESS_TOKEN


Some endpoints might also allow API key via query parameter (e.g. ?api_key=XYZ) depending on API version or mode. (Check the docs for the variant you use.) 


You may also need to set Accept: application/json in headers (though JSON is usually default).

Error Handling

Here are some common error cases and how to handle them:

HTTP Status	Meaning	Sample Body / status_code	Suggested Handling
401 Unauthorized	Invalid or missing token	“status_code”:7, “status_message”:“Invalid API key…”	Refresh or correct the token, don’t retry repeatedly without delay 


404 Not Found	The resource (e.g. movie id) doesn’t exist	“status_code”:34, “status_message”:“The resource you requested could not be found.”	Inform user “not found”, don’t try again with same ID 


400 Bad Request	Invalid parameters (e.g. invalid query param)	status message will say what’s wrong	Validate parameters on client side before sending
429 Too Many Requests	Rate limit exceeded	Might return a header like Retry-After or similar	Throttle requests, implement exponential backoff, pause before retrying
5xx Server Error	Server-side error	status message may be generic	Retry after delay, possibly fallback logic

In your code, wrap API calls in try/catch (or catch on promise), check HTTP status, and surface meaningful messages (e.g. “Movie not found”, “Rate limit exceeded — wait and retry”, etc.).

Usage Limits and Best Practices

TMDb enforces rate limiting (for example, requests per second or per time window). (Exact limits depend on your API plan) 
The Movie Database (TMDB)


To avoid hitting limits, batch requests, cache responses (especially for data unlikely to change often, like movie metadata or genres), and reuse data across calls.

Use pagination when retrieving large lists (e.g. /search/movie returns multiple pages) rather than trying to fetch everything at once.

Don’t request unnecessary fields — many endpoints support filtering or appending only needed parts (e.g. append_to_response=videos,credits). 


Respect “retry-after” headers or backoff hints.

Handle errors gracefully and avoid infinite retry loops.

Log your requests (endpoints, params, response times) for debugging and performance monitoring.

Read and abide by the API terms of service, especially for commercial usage (some usage tiers may require negotiation with TMDb). 
