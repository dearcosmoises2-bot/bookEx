# Requirements Document

## Introduction

This feature adds a comments and star rating system to the BookEx book exchange application. Authenticated users will be able to leave a text comment and a 1–5 star rating on any book listed in the application. Comments and ratings are displayed on the book detail page (`book_detail.html`) alongside the commenter's username. The `displaybooks.html` listing page will show each book's average rating as a summary. Users may edit or delete their own comments and ratings, but cannot modify those of other users.

## Glossary

- **Book**: A listing in the BookEx system represented by the `Book` model.
- **Comment**: A text review submitted by an authenticated user for a specific Book.
- **Rating**: An integer star value between 1 and 5 (inclusive) submitted by an authenticated user for a specific Book.
- **BookReview**: The combined record that stores one Comment and one Rating from one user for one Book.
- **Average_Rating**: The arithmetic mean of all Rating values for a given Book, rounded to one decimal place.
- **Review_Author**: The authenticated `User` who submitted a BookReview.
- **Comment_Form**: The Django form used to submit or edit a BookReview.
- **Book_Detail_Page**: The page rendered by `book_detail.html` that shows full information for a single Book.
- **Display_Books_Page**: The page rendered by `displaybooks.html` that lists all Books.
- **System**: The BookEx Django web application.

---

## Requirements

### Requirement 1: Submit a Book Review

**User Story:** As an authenticated user, I want to submit a comment and a star rating for a book, so that I can share my opinion with other users.

#### Acceptance Criteria

1. WHEN an authenticated user submits the Comment_Form with a non-empty comment text and a Rating value between 1 and 5, THE System SHALL save a new BookReview record linked to the Book and the Review_Author.
2. WHEN an authenticated user submits the Comment_Form, THE System SHALL associate the Review_Author's username with the saved BookReview.
3. IF an unauthenticated user attempts to submit the Comment_Form, THEN THE System SHALL redirect the user to the login page.
4. IF the submitted comment text is empty, THEN THE System SHALL return a validation error and SHALL NOT save the BookReview.
5. IF the submitted Rating value is outside the range 1–5, THEN THE System SHALL return a validation error and SHALL NOT save the BookReview.
6. WHEN a BookReview is saved successfully, THE System SHALL redirect the user back to the Book_Detail_Page for the same Book.

---

### Requirement 2: Display Comments and Ratings on the Book Detail Page

**User Story:** As a user, I want to see all comments and ratings for a book on its detail page, so that I can read other users' opinions before deciding to exchange.

#### Acceptance Criteria

1. WHEN the Book_Detail_Page is loaded, THE System SHALL display all BookReview records associated with the Book, ordered from newest to oldest by submission date.
2. THE System SHALL display the Review_Author's username alongside each BookReview.
3. THE System SHALL display the Rating value as a star representation (e.g., filled star characters) for each BookReview.
4. THE System SHALL display the comment text for each BookReview.
5. THE System SHALL display the submission date of each BookReview.
6. WHEN no BookReview records exist for a Book, THE System SHALL display a message indicating that no reviews have been submitted yet.
7. WHEN an authenticated user views the Book_Detail_Page, THE System SHALL display the Comment_Form below the existing reviews.
8. WHEN an unauthenticated user views the Book_Detail_Page, THE System SHALL display a prompt with a link to the login page instead of the Comment_Form.

---

### Requirement 3: Display Average Rating on the Book Listing Page

**User Story:** As a user browsing the book listing, I want to see the average star rating for each book, so that I can quickly identify highly-rated books.

#### Acceptance Criteria

1. WHEN the Display_Books_Page is loaded, THE System SHALL display the Average_Rating for each Book alongside the existing book information.
2. WHEN a Book has no BookReview records, THE System SHALL display "No ratings yet" in place of the Average_Rating.
3. THE System SHALL calculate the Average_Rating as the arithmetic mean of all Rating values for the Book, rounded to one decimal place.

---

### Requirement 4: One Review Per User Per Book

**User Story:** As a system administrator, I want each user to have at most one review per book, so that users cannot inflate or deflate ratings by submitting multiple reviews.

#### Acceptance Criteria

1. THE System SHALL enforce a constraint that each combination of Review_Author and Book corresponds to at most one BookReview record.
2. WHEN an authenticated user who has already submitted a BookReview for a Book visits the Book_Detail_Page, THE System SHALL display the user's existing BookReview pre-populated in an edit form instead of a blank submission form.
3. WHEN an authenticated user submits an updated BookReview, THE System SHALL update the existing BookReview record and SHALL NOT create a duplicate.

---

### Requirement 5: Edit and Delete Own Reviews

**User Story:** As an authenticated user, I want to edit or delete my own review, so that I can correct mistakes or remove outdated opinions.

#### Acceptance Criteria

1. WHEN an authenticated user views a BookReview they authored on the Book_Detail_Page, THE System SHALL display an edit option for that BookReview.
2. WHEN an authenticated user views a BookReview they authored on the Book_Detail_Page, THE System SHALL display a delete option for that BookReview.
3. WHEN an authenticated user submits an edited BookReview with valid data, THE System SHALL update the BookReview record and redirect the user to the Book_Detail_Page.
4. WHEN an authenticated user confirms deletion of their BookReview, THE System SHALL delete the BookReview record and redirect the user to the Book_Detail_Page.
5. IF an authenticated user attempts to edit or delete a BookReview they did not author, THEN THE System SHALL return a 403 Forbidden response.

---

### Requirement 6: Data Integrity

**User Story:** As a system administrator, I want reviews to be automatically removed when a book or user is deleted, so that orphaned data does not accumulate in the database.

#### Acceptance Criteria

1. WHEN a Book record is deleted, THE System SHALL delete all BookReview records associated with that Book.
2. WHEN a User account is deleted, THE System SHALL delete all BookReview records authored by that User.
