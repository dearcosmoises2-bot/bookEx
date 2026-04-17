# Search Books Features

## 1. Text-Based Search
- **Case-insensitive matching**: Users can type any part of the book title in the search bar, and the system will find all books whose titles contain that text.
- **Combined Filtering**: The text search can be used together with the letter filter (e.g., searching "harry" under the letter "H").

## 2. Alphabetical Letter Filter (A-Z)
- **A-Z Navigation**: Clickable letters A through Z are dynamically generated and displayed at the top of the page.
- **First Letter Filtering**: Clicking a letter instantly filters the list to show only books whose titles start with that specific letter.
- **"All" Option**: An "All" link is provided to quickly clear the active letter filter and view all available books.
- **Active State Highlights**: The currently selected letter (or "All") is bolded and underlined so users securely know what filter is active.

## 3. Results Display
- **Book Cards**: Matching books are presented in a clean, vertical card list.
- **Information Shown**: Each card displays the book's title, price, publish date, and a thumbnail of the cover image.
- **Safe Image Handling**: If a book is missing its cover picture, the page gracefully displays a fallback "No Image Available" placeholder instead of a broken image link.
- **View Details Link**: Each book card includes a quick link routing straight to that book's individual detail page.

## 4. Edge Cases Addressed
- **No Results Message**: If a search query or selected letter returns zero matching books, a friendly "No books found" message safely informs the user.
- **Whitespace Handling**: Extra spaces around the user's text query or letter selection are stripped before running the database query.
