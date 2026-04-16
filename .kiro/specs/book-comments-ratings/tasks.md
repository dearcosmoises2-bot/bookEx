# Implementation Plan: Book Comments & Ratings

## Overview

Implement the `BookReview` model, form, views, URL routes, and template updates for the book comments and ratings feature in the BookEx Django application. Each task builds incrementally on the previous, ending with all components wired together.

## Tasks

- [x] 1. Create the `BookReview` model and database migration
  - Add `BookReview` class to `bookEx/bookMng/models.py` with `book` (FK → Book, CASCADE), `user` (FK → AUTH_USER_MODEL, CASCADE), `rating` (PositiveSmallIntegerField with MinValueValidator(1)/MaxValueValidator(5)), `comment` (TextField), `created_at` (auto_now_add), `updated_at` (auto_now)
  - Add `Meta` with `ordering = ["-created_at"]` and `UniqueConstraint(fields=["book", "user"], name="unique_review_per_user_per_book")`
  - Import `MinValueValidator`, `MaxValueValidator` from `django.core.validators` and `settings` from `django.conf`
  - Run `python manage.py makemigrations` and `python manage.py migrate` to generate and apply `0004_bookreview.py`
  - _Requirements: 1.1, 1.2, 4.1, 6.1, 6.2_

  - [ ]* 1.1 Write property test for cascade delete (Property 8)
    - **Property 8: Cascade delete removes all associated reviews**
    - Use `@given(num_reviews=integers(min_value=1, max_value=10))` to create N reviews for a book, delete the book, assert zero `BookReview` records remain
    - Repeat for user deletion: create N reviews across books, delete the user, assert zero `BookReview` records remain
    - **Validates: Requirements 6.1, 6.2**

  - [ ]* 1.2 Write unit tests for `BookReview` model
    - Test valid creation with rating 1–5 and non-empty comment
    - Test `full_clean()` raises `ValidationError` for rating 0 and rating 6
    - Test `full_clean()` raises `ValidationError` for empty comment
    - Test `UniqueConstraint` raises `IntegrityError` on duplicate `(book, user)` pair
    - _Requirements: 1.1, 4.1, 6.1, 6.2_

- [x] 2. Create the `BookReviewForm`
  - Add `BookReviewForm` to `bookEx/bookMng/forms.py` as a `ModelForm` for `BookReview` with fields `["rating", "comment"]`
  - Override `rating` field as `forms.IntegerField(min_value=1, max_value=5, widget=forms.NumberInput(attrs={"min": 1, "max": 5}))`
  - _Requirements: 1.1, 1.4, 1.5_

  - [ ]* 2.1 Write unit tests for `BookReviewForm`
    - Test form is valid for all integer ratings 1–5 with a non-empty comment
    - Test form is invalid for rating 0, rating 6, and non-integer values
    - Test form is invalid for empty comment and whitespace-only comment
    - _Requirements: 1.4, 1.5_

  - [ ]* 2.2 Write property test for invalid input rejection (Property 2)
    - **Property 2: Invalid input is rejected without side effects**
    - Use `@given(rating=one_of(integers(max_value=0), integers(min_value=6)), comment=text(min_size=1))` — assert form is invalid and `BookReview.objects.count()` is unchanged
    - Use `@given(comment=text().filter(lambda s: not s.strip()))` with a valid rating — assert form is invalid and count is unchanged
    - **Validates: Requirements 1.4, 1.5**

- [x] 3. Implement `submit_review` view
  - Add `submit_review(request, book_id)` to `bookEx/bookMng/views.py` decorated with `@login_required` and `@require_POST`
  - Fetch book with `get_object_or_404(Book, pk=book_id)`
  - Bind `BookReviewForm(request.POST)` and validate; on failure re-render `book_detail.html` with form errors
  - On success use `BookReview.objects.update_or_create(book=book, user=request.user, defaults={"rating": ..., "comment": ...})` to enforce one-review-per-user rule
  - Redirect to `book_detail` for the same book on success
  - Register URL `path('book_detail/<int:book_id>/review/submit', views.submit_review, name='submit_review')` in `bookEx/bookMng/urls.py`
  - _Requirements: 1.1, 1.2, 1.3, 1.4, 1.5, 1.6, 4.3_

  - [ ]* 3.1 Write property test for review submission round-trip (Property 1)
    - **Property 1: Review submission round-trip**
    - Use `@given(rating=integers(min_value=1, max_value=5), comment=text(min_size=1).filter(str.strip))` — POST to `submit_review`, assert a `BookReview` exists with matching `user`, `book`, `rating`, `comment`
    - **Validates: Requirements 1.1, 1.2**

  - [ ]* 3.2 Write property test for one review per user per book (Property 3)
    - **Property 3: One review per user per book**
    - Use `@given(submissions=lists(tuples(integers(1, 5), text(min_size=1).filter(str.strip)), min_size=2, max_size=10))` — POST each submission for the same `(user, book)` pair, assert exactly one `BookReview` record exists and it reflects the last submitted values
    - **Validates: Requirements 4.1, 4.3**

  - [ ]* 3.3 Write unit tests for `submit_review` view
    - Test unauthenticated POST redirects to login (Requirement 1.3)
    - Test valid POST creates a new `BookReview` and redirects to `book_detail` (Requirements 1.1, 1.6)
    - Test second valid POST for same `(user, book)` updates the existing record without creating a duplicate (Requirements 4.1, 4.3)
    - Test invalid POST (empty comment) re-renders with form errors and does not save (Requirement 1.4)
    - _Requirements: 1.1, 1.3, 1.4, 1.6, 4.1, 4.3_

- [x] 4. Checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

- [x] 5. Implement `edit_review` and `delete_review` views
  - Add `edit_review(request, book_id)` decorated with `@login_required` and `@require_POST`
    - Fetch review with `get_object_or_404(BookReview, book=book, user=request.user)` (non-authors get 404)
    - Bind `BookReviewForm(request.POST, instance=review)`, validate, save, redirect to `book_detail`
  - Add `delete_review(request, book_id)` decorated with `@login_required` and `@require_POST`
    - Fetch review with `get_object_or_404(BookReview, book=book, user=request.user)`
    - Delete the review and redirect to `book_detail`
  - Register URL entries in `bookEx/bookMng/urls.py`:
    - `path('book_detail/<int:book_id>/review/edit', views.edit_review, name='edit_review')`
    - `path('book_detail/<int:book_id>/review/delete', views.delete_review, name='delete_review')`
  - _Requirements: 5.1, 5.2, 5.3, 5.4, 5.5_

  - [ ]* 5.1 Write property test for edit round-trip (Property 6)
    - **Property 6: Edit round-trip**
    - Use `@given(new_rating=integers(min_value=1, max_value=5), new_comment=text(min_size=1).filter(str.strip))` — create an initial review, POST edit with new values, assert the stored record matches new values and no new record was created
    - **Validates: Requirements 5.3**

  - [ ]* 5.2 Write property test for ownership enforcement (Property 7)
    - **Property 7: Ownership enforcement**
    - Use `@given(num_other_users=integers(min_value=1, max_value=5))` — for each non-author user, attempt POST to `edit_review` and `delete_review`, assert response is non-2xx and the review record is unchanged
    - **Validates: Requirements 5.5**

  - [ ]* 5.3 Write unit tests for `edit_review` and `delete_review` views
    - Test `edit_review` returns 404 when the requesting user does not own the review (Requirement 5.5)
    - Test `edit_review` with valid data updates the record and redirects to `book_detail` (Requirement 5.3)
    - Test `delete_review` removes the review and redirects to `book_detail` (Requirement 5.4)
    - _Requirements: 5.3, 5.4, 5.5_

- [x] 6. Update `book_detail` view and template
  - In `book_detail` view (`bookEx/bookMng/views.py`):
    - Fetch all reviews: `reviews = BookReview.objects.filter(book=book).order_by("-created_at")`
    - Determine user's existing review: `user_review = BookReview.objects.filter(book=book, user=request.user).first()` if authenticated, else `None`
    - Instantiate form: `BookReviewForm(instance=user_review)` (pre-populated if user already has a review)
    - Pass `reviews`, `user_review`, and `form` to the template context
  - Update `bookEx/bookEx/templates/bookMng/book_detail.html`:
    - Display all reviews below the book info table, ordered newest-to-oldest, each showing: author username, star representation of rating (e.g., `"★" * review.rating`), comment text, submission date
    - Show "No reviews yet" message when `reviews` is empty (Requirement 2.6)
    - For authenticated users: show `BookReviewForm` below reviews; if `user_review` exists, show edit/delete controls for that review (Requirements 2.7, 5.1, 5.2)
    - For unauthenticated users: show a login prompt with a link to the login page instead of the form (Requirement 2.8)
  - _Requirements: 2.1, 2.2, 2.3, 2.4, 2.5, 2.6, 2.7, 2.8, 4.2, 5.1, 5.2_

  - [ ]* 6.1 Write property test for review display completeness (Property 4)
    - **Property 4: Review display completeness**
    - Use `@given(reviews=lists(tuples(integers(1, 5), text(min_size=1).filter(str.strip)), min_size=1, max_size=20))` — create N reviews for a book, GET `book_detail`, assert each review's username, star rating, comment, and date appear in the response content, and that reviews appear newest-to-oldest
    - **Validates: Requirements 2.1, 2.2, 2.3, 2.4, 2.5**

  - [ ]* 6.2 Write unit tests for `book_detail` view and template
    - Test context contains `reviews`, `user_review`, and `form` (Requirement 2.7)
    - Test page shows "No reviews yet" when no reviews exist (Requirement 2.6)
    - Test page shows login prompt for unauthenticated users instead of the form (Requirement 2.8)
    - Test page pre-populates the form when the authenticated user already has a review (Requirement 4.2)
    - Test edit/delete controls appear only for the review author (Requirements 5.1, 5.2)
    - _Requirements: 2.6, 2.7, 2.8, 4.2, 5.1, 5.2_

- [x] 7. Update `displaybooks` view and template
  - In `displaybooks` view (`bookEx/bookMng/views.py`):
    - Import `Avg` from `django.db.models`
    - Annotate queryset: `books = Book.objects.annotate(avg_rating=Avg("reviews__rating"))`
  - Update `bookEx/bookEx/templates/bookMng/displaybooks.html`:
    - Add an "Avg Rating" column to the table header
    - Render `{{ book.avg_rating|floatformat:1|default:"No ratings yet" }}` in each book row
  - _Requirements: 3.1, 3.2, 3.3_

  - [ ]* 7.1 Write property test for average rating calculation (Property 5)
    - **Property 5: Average rating calculation**
    - Use `@given(ratings=lists(integers(min_value=1, max_value=5), min_size=1, max_size=100))` — create one review per rating value for a book, assert `avg_rating` annotation equals `round(sum(ratings) / len(ratings), 1)`
    - **Validates: Requirements 3.1, 3.3**

  - [ ]* 7.2 Write unit tests for `displaybooks` view and template
    - Test `avg_rating` annotation is present on each book in the context queryset
    - Test page displays "No ratings yet" for a book with no reviews (Requirement 3.2)
    - Test page displays the correct rounded average for a book with multiple reviews (Requirement 3.1)
    - _Requirements: 3.1, 3.2, 3.3_

- [x] 8. Register `BookReview` in Django admin
  - In `bookEx/bookMng/admin.py`, import `BookReview` and call `admin.site.register(BookReview)`
  - _Requirements: 4.1, 6.1, 6.2_

- [x] 9. Final checkpoint — Ensure all tests pass
  - Ensure all tests pass, ask the user if questions arise.

## Notes

- Tasks marked with `*` are optional and can be skipped for a faster MVP
- Property-based tests use **Hypothesis** (`pip install hypothesis`) — each property maps to a numbered property in the design document
- Each task references specific requirements for traceability
- The `update_or_create` pattern in `submit_review` enforces the one-review-per-user-per-book rule at the application layer; the DB `UniqueConstraint` is the safety net
- Run `python manage.py makemigrations bookMng && python manage.py migrate` after completing Task 1
