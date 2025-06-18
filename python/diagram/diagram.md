# Signup sequence diagram
sequenceDiagram
    participant Anonymous
    participant App
    participant BE as Backend
    participant Database

    Anonymous->>App: Access sign-up form
    App->>BE: Send sign-up data
    BE->>BE: Validate data

    alt Valid data
        BE->>Database: Valid data
        Database-->>BE: Successfully registered
        BE-->>App: Successfully registered
        App-->>Anonymous: Successfully registered
    else Invalid data
        BE-->>App: Return error
        App-->>Anonymous: Display error
    end

# Veify signup sequence diagram
sequenceDiagram
    participant Anonymous
    participant App
    participant BE as Backend
    participant Database

    Anonymous->>App: Access verify signup screen
    App->>BE: Send token
    BE->>BE: Validate token

    alt Valid data
        BE->>Database: Activate user
        Database-->>BE: Successfully
        BE-->>App: Successfully
        App-->>Anonymous: Successfully
    else Invalid data
        BE-->>App: Return invalid token
        App-->>Anonymous: Display error
    end

# Login sequence diagram
sequenceDiagram
    participant User
    participant App
    participant BE as Backend
    participant Database

    User->>App: Access login form
    App->>BE: Send login data
    BE->>BE: Verify data

    alt Valid data
        BE->>Database: Query token
        Database-->>BE: Token
        BE-->>App: Return token
        App-->>User: Display home screen
    else Invalid data
        BE-->>App: Return error
        App-->>User: Display error
    end


# Get course list/ course detail sequence diagram
sequenceDiagram
    participant Anonymous_User as Anonymous/ User
    participant App
    participant BE as Backend
    participant Database

    Anonymous_User->>App: Access course list UI
    App->>BE: Call API get course list
    BE->>Database: Query data
    Database-->>BE: Course list
    BE-->>App: Course list
    App-->>Anonymous_User: Display Course list

    alt Select course
        Anonymous_User->>App: Select course
        App->>BE: Call API get course detail
        BE->>Database: Query data
        Database-->>BE: Course detail
        BE-->>App: Course detail
        App-->>Anonymous_User: Successfully registered
    end

# Enroll course sequence diagram
sequenceDiagram
    participant Student
    participant App
    participant BE as Backend
    participant Database

    Student->>App: Select course to enroll
    App->>BE: Call API enroll course
    BE->>Database: Get course
    Database-->>BE: Course data
    BE->>Database: Filter enrolled with student and course
    Database-->>BE: Enrolled list

    alt Empty list
        BE->>Database: Make query to create enrollment
        Database-->>BE: Successfully enrolled
        BE->>Database: Make query to create notification
        Database-->>BE: Successfully created
        BE-->>App: Successfully enrolled
        App-->>Student: Display successfully enrolled
    else Not empty list
        BE-->>App: Return enrolled error
        App-->>Student: Display enrolled error
    end

# Unenroll course sequence diagram
sequenceDiagram
    participant Student
    participant App
    participant BE as Backend
    participant Database

    Student->>App: Select course to unenroll
    App->>BE: Call API unenroll course
    BE->>Database: Get course
    Database-->>BE: Course data
    BE->>Database: Filter enrolled with student and course
    Database-->>BE: Enrolled list

    alt Not empty list
        BE->>Database: Make query to remove enrollment
        Database-->>BE: Successfully remove
        BE->>Database: Make query to create notification
        Database-->>BE: Successfully created notification
        BE-->>App: Successfully unenrolled
        App-->>Student: Display successfully unenrolled
    else Empty list
        BE-->>App: Return not enrolled error
        App-->>Student: Display not enrolled error
    end

# Update course sequence diagram
sequenceDiagram
    participant Instructor
    participant App
    participant BE as Backend
    participant Database

    Instructor->>App: Access update course
    App->>BE: Call API get category list
    BE->>Database: Make query get category list
    Database-->>BE: Category list data
    BE-->>App: Category list data
    App-->>Instructor: Display update course form
    Instructor->>App: Enter course input
    App->>BE: Call API update course
    BE->>Database: Make query get enrollment with this course
    Database-->>BE: Enrollment list

    alt Update status and not empty enrollment list
        BE-->>App: Return error
        App-->>Instructor: Display error
    else Not update status
        BE->>Database: Make query update course
        Database-->>BE: Successfully updated
        BE-->>App: Successfully updated
        App-->>Instructor: Display successfully created
    end

# Create course sequence diagram
sequenceDiagram
    participant Instructor
    participant App
    participant BE as Backend
    participant Database

    Instructor->>App: Access create course
    App->>BE: Call API get category list
    BE->>Database: Make query get category list
    Database-->>BE: Category list data
    BE-->>App: Category list data
    App-->>Instructor: Display create course form
    Instructor->>App: Enter course input
    App->>BE: Call API crate course
    BE->>BE: Verify data

    alt Valid data
        BE->>Database: Make query create course
        Database-->>BE: Successfully created
        BE-->>App: Successfully created
        App-->>Instructor: Display successfully created
    else Invalid data
        BE-->>App: Return data
        App-->>Instructor: Display error
    end


# Notification list/ detail sequence diagram
sequenceDiagram
    participant User
    participant App
    participant BE as Backend
    participant Database

    User->>App: Access notifications page
    App->>BE: Call API get notification list
    BE->>Database: Make query get notifications data
    Database-->>BE: Notifications data
    BE-->>App: Notifications data
    App-->>User: Display notification list

    alt Select a notification
        User->>App: Select a notification
        App->>BE: Call API get notification
        BE->>Database: Make query to get notification
        Database-->>BE: Notification data
        BE-->>App: Notification data
        App-->>User: Display notification
    end

# Verify reset password sequence diagram
sequenceDiagram
    participant Anonymous
    participant App
    participant BE as Backend

    Anonymous->>App: Access verify reset password screen
    App->>BE: Send data
    BE->>BE: Validate data

    alt Valid data
        BE->>BE: Send token to email
        BE-->>App: Successfully
        App-->>Anonymous: Successfully
    else Invalid data
        BE-->>App: Return error
        App-->>Anonymous: Display error
    end

# Reset password sequence diagram
sequenceDiagram
    participant Anonymous
    participant App
    participant BE as Backend
    participant Database

    Anonymous->>App: Access reset password screen
    App->>BE: Send token
    BE->>BE: Validate token

    alt Valid data
        BE->>Database: Update user password
        Database-->>BE: Successfully
        BE-->>App: Successfully
        App-->>Anonymous: Successfully
    else Invalid data
        BE-->>App: Return invalid token
        App-->>Anonymous: Display error
    end