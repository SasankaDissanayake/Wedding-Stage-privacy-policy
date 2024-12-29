# GameHive
#### Video Demo:  <[URL HERE (Click me)](https://www.youtube.com/watch?v=2EdBd5nvfL4)>
#### Description: Game searching web app
#### This repository was submitted as the final project for CS50X 2024 online course offered by Harvard University. It’s a web application built with Flask framework

> [!NOTE]
> I'm Methuka Pasandul, im from Sri Lanka and 18yrs old. my final project is basically a game searching webapp. The specialty of this webapp is user can easily find and search for games by using filters such as categories, sorting and more. you can find over 400+ games in here, since the Api only returning around 400 games. so my sincere apology if u won't find your favorite game here, and I'm looking forward to publishing and implement some advanced user-friendly features in future.

## Introduction
This document details the development and functionality of a web application designed for discovering and managing free-to-play games. Leveraging the FreeToGame API, this platform provides users with a comprehensive catalog of games, complete with detailed information and categorization. The application prioritizes user engagement and personalization through several key features:

- **User Authentication and Management:** Users can create accounts, log in, and manage their profiles, including the option to delete their accounts.

- **Game Discovery and Exploration:** The core functionality revolves around searching and browsing games. Users can refine their searches using various categories provided by the FreeToGame API, allowing for targeted discovery based on genre, platform, and other criteria. Each game listing provides detailed information, enabling users to make informed decisions.

- **Personalized Game Library:** A key feature of the application is the ability for users to add their favorite game thier library. By "liking" games, users can save them to their personal collections for easy access and future reference.

- **Community Engagement (Blog-like Functionality):** The platform incorporates elements of a blog, encouraging user read about thier favorite games. This feature allows for potential future expansion into user reviews, discussions, and other forms of community engagement.

## Core Features


### Register
GameHive prioritizes user security by implementing a robust registration process:

HTTP Methods: The register route handles both GET and POST requests. GET requests display the registration form, while POST requests submit user-provided data for account creation.

Form Validation: Upon receiving a POST request:

- **Missing Username:** If the username field is blank, a user-friendly flash message is displayed using Flask's flash function, indicating that a username is required.

- **Missing Password:** Similarly, missing password or password confirmation triggers error messages.

- **Password Matching:** Submitted passwords must match exactly. A mismatch prompts an error message.

- **Password Strength:** The password is validated using regular expressions to ensure it contains:
  - Special characters ([!@#$%^&*()])
  - Digits ([0-9])

- **Username Restriction:** Usernames are restricted to letters only (no special characters) using regular expressions.
Excessive Username Length: A maximum character limit for usernames is enforced to prevent potential database issues.
Secure Password Hashing: If all validation checks pass, the user's password is securely hashed using Flask-WTF's generate_password_hash function. This one-way hashing process prevents storing passwords in plain text, enhancing security.


### LogIn
This part describes the secure user login functionality implemented in the GameHive:

HTTP Methods and Session Management: The /login route handles both GET and POST requests. A GET request renders the login form. Upon a POST request (form submission), the existing user session is cleared using session.clear(). This ensures that any previous user's session data is removed before a new login attempt.

- **Input Validation:** The login process starts with input validation:

- **Missing Username:** If the username field is empty, a flash message "Username is required." is displayed, and the login form is re-rendered.
- **Missing Password:** Similarly, if the password field is empty, a flash message "Password is required." is shown, and the login form is re-rendered. This prevents empty submissions and provides immediate feedback to the user.
- **Database Query and Authentication:**

  - **Database Lookup:** A database query is executed using a parameterized query (SELECT * FROM users WHERE username = ?) to retrieve the user's information based on the provided username. This parameterized query is crucial for preventing SQL injection vulnerabilities.
  - **Credential Verification:** The code checks two conditions:
    - len(rows) != 1: Ensures that exactly one user with the given username exists in the database. If no user or multiple users are found (which should not happen with a properly designed database), the login fails.
    - not check_password_hash(rows[0]["hash"], request.form.get("password")): Uses Flask-WTF's check_password_hash function to securely compare the provided password with the stored hash from the database. This is the correct and secure way to verify passwords. Never compare passwords in plain text.
- **Session Initialization and Redirection:** If both the username exists and the password matches its hash:

- **Session Setup:** The user's id from the database is stored in the session using session["user_id"] = rows[0]["id"]. This establishes a logged-in session for the user.
Success Message and Redirection: A success flash message "Log in successful!" is displayed, and the user is redirected to the home page (/).

- **GET Request Handling:** If the user accesses the /login route via a GET request (e.g., clicking a link), the login form is rendered.


### Game Search and Detail View
This part showcases the core functionality of the application: searching for and displaying free-to-play games. Here's a breakdown:

Route Configuration:
The index route is defined with two URL patterns:
- /: Handles the root path without a specific game ID, displaying a game listing page.
- /\<int:game_id>: Handles URLs with an integer game ID, allowing users to view details of a specific game.
The login_required decorator ensures that only logged-in users can access this page.
User Context and Search Parameters:

The user's ID is retrieved from the session using session["user_id"].
Building the API Request:

The code retrieves search filters from the query string using request.args.get():
- query: Game name (optional)
- platform: Platform (optional)
- category: Category (optional)
- sort: Sort criteria (optional)
- page: Page number (defaults to 1)
  
An API request URL is constructed based on the FreeToGame API URL and the extracted parameters (API_URL) using a dictionary (params).
The request URL is dynamically built based on the presence of filters:
- If platform is provided, it's added to the params dictionary.
- Similar logic applies for category and sort.

Fetching and Filtering Games:
- The application makes a GET request to the FreeToGame API using the constructed URL with requests.get().
- A successful response (status code 200) indicates valid data.
- The response JSON is parsed and stored in the games variable.
- If a search query (query) exists, the games are filtered to include only those with matching titles (case-insensitive).
- If the response is unsuccessful, the games variable remains an empty list.

Pagination:
- A fixed number of games are displayed per page (games_per_page).
- The total number of pages is calculated using integer division and rounding up to accommodate all games.
- Pagination logic is implemented:
  - start index defines the first game to show for the current page.
  - end index defines the last game (exclusive) to show for the current page.
  - paginated_games list contains the subset of games for the current page.

Handling Game Details:
If a specific game ID (game_id) is present in the URL:
- Another GET request is made to the FreeToGame API, but with the specific game ID appended to the URL.
- A successful response populates the game_details variable with the detailed information for that game.
- The detailed.html template is rendered, displaying the specific game details.
If the game details request fails, an error message is returned.

Rendering Results:
For the root path (/), the index.html template is rendered:
- games: List of paginated games for the current page.
- current_page: Current page number.
- total_pages: Total number of pages.
- Search filters (query, platform, category, sort) are passed to retain the filtering state across page navigation.


### Personalized Game Library
This part showcases the user's library feature and liking feature

User's Liked Games:
- The /library route displays a user's library of liked games.
- login_required ensures only authenticated users can access their library.
- The user's ID is retrieved from the session (session["user_id"]).

Fetching Liked Games:
- A database query retrieves game IDs from the likes table where the userid matches the logged-in user:
- SELECT gameid FROM likes WHERE userid = ?
- If liked games exist:
  - A list of game IDs is extracted (game_ids).
  - For each game ID, an API request is made to the FreeToGame API to fetch detailed game information.
  - Successful responses populate the liked_games list with individual game data.
- Error handling provides user feedback if game details cannot be retrieved for a specific ID.

Game Recommendations:
This section leverages the user's liked games to suggest new games.

It retrieves tags associated with the user's liked games:
- SELECT tag FROM liked_game_tags WHERE like_id IN (SELECT id FROM likes WHERE userid = ?)
- Tag frequency is calculated:
  - A dictionary tag_counts stores the number of occurrences for each unique tag.
  - Top 3 most frequent tags are identified using sorting with sorted and tag_counts.get.
- For each top tag:
  - An API request retrieves games based on the tag category using the FreeToGame API:
    - https://www.freetogame.com/api/games?category={tag}

Recommendation filtering:
- Excludes games already present in the user's library (using previously retrieved liked_game_ids).
- Shuffling ensures variety and the top 5 recommendations are selected.

- Rendering Library:
- The library.html template is rendered, displaying:
  - liked_games: List of the user's liked games with detailed information.
  - recommended_games: Top 5 recommended games based on user preferences.

Liking a Game:
- The /like route (POST request) handles the action of liking a game.
- The user's ID and game ID are retrieved from the form data.
- Validation ensures a game ID is provided.

The code checks for duplicate likes before insertion:
- It queries the likes table to see if the user has already liked the specific game.
- If a duplicate is found, an error message is displayed, and a redirect occurs.
- If the game is not already liked:
  - A new entry is inserted into the likes table with the user ID and game ID.
  - The ID of the newly inserted like record is retrieved for further operations.
  - Game details (including tags) are fetched from the FreeToGame API:
- Tags are extracted from the game data (adapt as needed based on the API response format).
- Each tag is inserted into the liked_game_tags table, associating it with the like record's ID.
- Error handling provides user feedback in case of issues during game detail retrieval or tag insertion.


## Beyond the features detailed above, the application also includes several other important functionalities that contribute to a complete and user-friendly experience:

### Change Password: 
  - Users can update their passwords through a secure process, ensuring they maintain control over their account security. This feature typically involves verifying the user's current password before allowing them to set a new one.

### Delete Account: 
  - Users have the option to permanently delete their accounts and associated data. This feature respects user privacy and provides control over their information.

### Unlike a Game: 
  - Users can remove games from their library by "unliking" them. This allows users to manage their collections and refine their preferences.

### Logout: 
  - A secure logout function allows users to end their sessions and protect their accounts, especially when using public or shared computers. This clears the user's session data on the server.

These additional features enhance the overall user experience by providing necessary account management options and further personalization.

#### Thank you for taking the time to read this documentation. We hope it has provided a clear understanding of the application's functionality and design.





