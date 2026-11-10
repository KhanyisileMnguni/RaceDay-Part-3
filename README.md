# Part 1 - System Description

RaceDay is a full-stack, web-based event management system built for the South African road running, walking, and cycling community.
It replaces the paper-based registration, spreadsheets, and disconnected communication that many local road events
(park runs, charity walks, community cycling events) currently rely on.

The platform allows Event Organisers to create and manage events, define categories (e.g. distance-based race categories),and capture participant results.
Participants can browse upcoming events, enter events by enrolling in a category, track their personal performance history across past events,
and prepare for race day using live weather and route information.

This repository covers Part 1:
System Planning and Database — the ERD, API endpoint plan, and SQL database script that were completed before any application code was written.

Roles:
Organiser — creates and manages Events and their Categories, views who has enrolled in their events, and captures results once a race is completed.
An organiser can only manage the events they created themselves.
Participant — browses public events and categories, enrols in a category, views their own enrolment history,
and can view their own results and overall performance history across multiple events.

Database Design (ERD)

See docs/raceday_erd.docx, the data model consists of six entities:
Organiser, Participant, Event, Category, Enrolment, and Result.
Enrolment is the associative table resolving the many-to-many relationship between Participants and Categories;
Result stores one outcome per Enrolment.

API Endpoint Plan

See docs/api_endpoint_plan.md.docx for the full table of planned endpoints
(Method, Route, Description, Role Required, Request Body, Expected Response),
covering authentication, user profile, events, categories, enrolments, and results.

SQL Script

See docs/raceday_database.sql. Run in SQL Server Management Studio (SSMS).
The script creates all six tables with primary keys, foreign keys, and constraints,
and seeds the database with 2 Organisers, 2 Participants, 3 Events, categories for each event, and sample enrolments/results.

CI/CD

A GitHub Actions workflow (.github/workflows/validate-docs.yml) runs on every push and pull request to main,
validating that the /docs folder exists and contains the ERD, API endpoint plan, and SQL script.

<img width="1498" height="895" alt="Screenshot 2026-09-04 131118" src="https://github.com/user-attachments/assets/0aed4c0c-8383-484c-b506-a128097eef45" />


# Part 2 - RaceDay RESTful API

A RESTful API for managing race events, participants, event enrolments, and user authentication. The API uses role-based access control to help ensure that users can only access functionality permitted by their assigned roles.

Features:
User Authentication: User registration and login using JWT (JSON Web Token) authentication.
Role-Based Access Control: Restricts protected endpoints according to user roles.
Event Management: Allows authorised Organisers to manage race events.
Event Enrolments: Allows Participants to enrol in events.
Categories: Provides functionality for managing event categories.
Participant Management: Supports participant-related operations.
Profile Management: Provides profile-related functionality.
Automated Testing: Includes integration tests for authentication, event management, enrolments, and role-based access.

Technologies Used:
C#
ASP.NET Core Web API (.NET 10)
Entity Framework Core
Microsoft SQL Server
JWT Authentication
xUnit
ASP.NET Core MVC Testing

Project Structure:
API Project

| Folder / File | Description |
|---|---|
| `RaceDay.API/Controllers/` | Contains API controllers for authentication, categories, enrolments, events and participants. |
| `RaceDay.API/Data/` | Contains `AppDbContext.cs` for database configuration and access. |
| `RaceDay.API/Dtos/` | Contains Data Transfer Objects used for API requests and responses. |
| `RaceDay.API/Migrations/` | Contains Entity Framework database migrations. |
| `RaceDay.API/Models/` | Contains the application's data models. |
| `RaceDay.API/Services/` | Contains supporting services, including `JwtService.cs` and `PasswordService.cs`. |
| `RaceDay.API/Program.cs` | Configures services, middleware and API startup. |
| `RaceDay.API/appsettings.json` | Contains application configuration settings. |

### API Test Files

| File | Description |
|---|---|
| `AuthTests.cs` | Authentication tests. |
| `CategoryTests.cs` | Category-related tests. |
| `EnrolmentTests.cs` | Enrolment-related tests. |
| `EventTests.cs` | Event-related tests. |
| `ProfileTests.cs` | Profile-related tests. |
| `ResultTests.cs` | Results-related tests. |
| `RoleAccessTests.cs` | Role-based access tests. |
| `RulesTests.cs` | Application rules tests. |
| `TestSupport.cs` | Shared test support code. |

The two roles:
Organiser - creates, updates and deletes *their own* events; defines categories for them (name, distance, max
participants, entry fee); views all enrolments for their events; captures finish time, position and status for
enrolments in their events. Cannot enrol in events.

Participant - browses events and categories; enrols in a category (one enrolment per category, blocked when
the category is full); views and cancels their own enrolments; views their own results and full performance
history. Cannot create or change events, categories or results.

Anyone can register as either role (the role is chosen at registration). Browsing events/categories is public;
everything else requires login, and role/ownership is checked on every protected endpoint
(401 = not logged in, 403 = wrong role or not the owner).

Prerequisites:
Before running the project, ensure that you have installed:
Visual Studio 2022 or a compatible IDE that supports .NET 10.
The .NET 10 SDK.
Microsoft SQL Server.
SQL Server Management Studio (SSMS) or another suitable database management tool.
Git.

Installation:
Clone the repository:
git clone <YOUR_GITHUB_REPOSITORY_URL>
Navigate to the solution directory:
cd RaceDay.API
Open the solution file in Visual Studio.
Restore the project's NuGet packages.

Setup and run:
1. Install Visual Studio with the ASP.NET workload, .NET SDK matching the project, and SQL Server (LocalDB is fine).
2. Clone the repo and open `RaceDay_API.slnx` (or the solution in the Part 2 folder).
3. Check `ConnectionStrings:DefaultConnection` in `RaceDay.API/appsettings.json` points at your SQL Server.
4. Create the database from the models. In **Package Manager Console** (Default project = `RaceDay.API`):
   ```
   Add-Migration InitialCreate
   Update-Database
   ```
5. Run `RaceDay.API` (https profile). Swagger opens at `https://localhost:<port>/swagger`.
6. In Swagger: `POST /api/auth/register`, then `POST /api/auth/login`. The browser keeps the session cookie,
   so every later "Try it out" call in that tab is authenticated. Use `POST /api/auth/logout` to switch user.
   
Endpoints (from the Part 1 plan)
| # | Method | Route | Access |
|---|---|---|---|
| 1 | POST | /api/auth/register | Public |
| 2 | POST | /api/auth/login | Public |
| 3 | GET | /api/profile | Logged in |
| 4 | PUT | /api/profile | Logged in |
| 5 | POST | /api/events | Organiser |
| 6 | GET | /api/events | Public |
| 7 | GET | /api/events/{id} | Public |
| 8 | PUT | /api/events/{id} | Organiser (owner) |
| 9 | DELETE | /api/events/{id} | Organiser (owner) |
| 10 | POST | /api/events/{id}/categories | Organiser (owner) |
| 11 | GET | /api/events/{id}/categories | Public |
| 12 | PUT | /api/categories/{id} | Organiser (owner) |
| 13 | DELETE | /api/categories/{id} | Organiser (owner) |
| 14 | POST | /api/categories/{id}/enrolments | Participant |
| 15 | GET | /api/enrolments/customer | Participant |
| 16 | GET | /api/events/{id}/enrolments | Organiser (owner) |
| 17 | DELETE | /api/enrolments/{id} | Participant (owner) |
| 18 | POST | /api/enrolments/{id}/result | Organiser (owner) |
| 19 | GET | /api/enrolments/{id}/result | Logged in (own result / own event) |
| 20 | GET | /api/participants/me/history | Participant |

Extra: `POST /api/auth/logout` ends the session.

Database Configuration:
Configure the database connection string in appsettings.json or through your development environment's configuration settings.
Example connection string:
{
  "ConnectionStrings": {
    "DefaultConnection": "YOUR_SQL_SERVER_CONNECTION_STRING"
  }
}
Replace the placeholder with your actual SQL Server connection string. Keep private credentials and production connection details out of source control.

JWT Configuration:
Configure the JWT settings expected by the application in appsettings.json or your development environment's configuration.
Use a secure signing key of at least 32 characters, as required by the application. Do not commit real signing keys or production secrets to GitHub.

Run the API:
In Visual Studio, set RaceDay.API as the startup project and run the application.
Alternatively, from the directory containing RaceDay.API.csproj, execute:
dotnet restore
dotnet run
The application will display the local address where the API is running.

RaceDay API integration tests:
These tests exercise the API through HTTP using `WebApplicationFactory<Program>`. They cover:
- Registration and login, including invalid credentials.
- Anonymous requests to protected endpoints.
- Organiser-only event creation and event ownership checks.
- Participant-only enrolment and checking that the enrolment is returned by the participant's enrolment endpoint.
- Wrong-role rejection and duplicate enrolment.
The test host replaces SQL Server with a uniquely named EF Core InMemory database, so tests do not require a local SQL Server instance.
Run locally
From the solution folder:
```bash
dotnet restore RaceDay.API.slnx
dotnet test RaceDay.API.slnx --configuration Release
```
The test project uses the .NET 10 target framework to match the API. These tests still need to be run on a machine with the .NET 10 SDK installed; they have not been executed in this environment.
Test Coverage
The current local test run completed successfully with 5 tests passed, 0 failed, and 0 skipped.
Authentication and Authorisation
Protected endpoints require a valid JWT access token where authentication is enforced.

Include the token in the HTTP request's Authorization header:
Authorization: Bearer YOUR_ACCESS_TOKEN
Access to role-restricted functionality depends on the authenticated user's assigned role and the API's authorisation rules.

Continuous Integration:
The project is intended to run automated tests locally and through **GitHub Actions CI/CD**.
The GitHub Actions workflow should restore dependencies, build the solution, and run the test suite. Successful local test results do not by themselves confirm that the GitHub Actions workflow is configured or passing.

Screenshot of CI/CD workflow
<img width="1510" height="515" alt="image" src="https://github.com/user-attachments/assets/54940fa0-698e-4485-8986-9f37dd5a095a" />

YouTube Video link:
https://youtu.be/05gTS_h3Tv8

# Part 3 - RaceDay Web Application (ASP.NET Core MVC, Azure Blob Storage, Docker)

The RaceDay MVC web application is the front end for the Part 2 API. Organisers and Participants use it to create, browse,
enter and track road running, walking and cycling events. The site has **no database code and no business logic of its own**:
every piece of data comes from the Part 2 API, and every image upload goes through the API into Azure Blob Storage.
```
Unlisted YouTube link: Could not upload video due to internet issues
```
## Features

| Area | What it does |
|---|---|
| Registration and login | Register as an Organiser or a Participant, log in through the API, log out. |
| Session management | After login the session stores the JWT, user ID, role and name. Logging out clears the session and revokes the token on the API. Protected pages redirect unauthenticated users to the login page. |
| Role-based navigation | Organisers and Participants see different menus. Visiting a page for the other role shows an "access denied" page. |
| Organiser | Dashboard (events, enrolments per event, upcoming dates), create/edit/delete events with a banner image, add/edit/remove categories, view enrolments, enter and correct results. |
| Participant | Home page with search and filters, event details with "Enter event", My Events (cancel an entry), My Results (finish time and position such as "47th of 312"), Profile with picture upload. |
| Azure Blob Storage | Event banners and profile pictures are uploaded from the MVC site to the API, stored in private Azure containers by the API, saved in the database as a URL, and shown on the pages. |
| Docker | The whole system (SQL Server, API and MVC site) starts with one command (see below). |
| UI | Bootstrap 5 with a RaceDay sporting theme. Enrolment and result statuses are colour coded (amber = registered, green = completed, red = did not finish or similar). |

## How the pieces fit together

Browser  ->  RaceDay.MVC (ASP.NET Core MVC, session)  ->  RaceDay.API (REST, JWT, EF Core)  ->  SQL Server
                                                                    |
                                                                    +->  Azure Blob Storage (event banners, profile pictures)

* `RaceDay.MVC/Controllers` - one controller per area (Account, Home, Events, Participant, Organiser, Media).
* `RaceDay.MVC/Services/ApiClient.cs` - the only class that calls the API. It adds the logged-in user's token and turns every failure into a friendly message.
* `RaceDay.MVC/Infrastructure` - session helpers and the `[SessionAuthorize("Organiser")]` filter that protects pages.
* `RaceDay.MVC/Views` - Razor views, with shared layout, alerts, event card and status badge.
* `RaceDay.MVC/Models` - view models and simple copies of the API's responses.

## Run everything with Docker (one command)

Requirements: Docker Desktop.

1. Create your private settings file from the template and fill in the values (it is ignored by git and must never be committed):
   Copy-Item .env.example .env
   (on macOS/Linux: `cp .env.example .env`). Set `RACEDAY_DB_PASSWORD`, `RACEDAY_JWT_KEY` and, for image uploads,
   `RACEDAY_AZURE_CONNECTION_STRING` (from the Azure portal: your storage account > Security + networking > Access keys).
2. From the repository root, start SQL Server, the API and the MVC site:
   docker compose up --build
3. Open the site at **http://localhost:8080** and the API's Swagger page at **http://localhost:5285/swagger**.

The API creates the database tables automatically the first time it starts. Stop everything with `Ctrl+C`, or `docker compose down`.
The containerised site behaves the same as the one started from Visual Studio; only the API address differs, and it is passed in with the
`ApiSettings__BaseUrl` environment variable.

### Run only the MVC site in Docker (API running on your computer)

dotnet run --project RaceDay.API --urls "http://0.0.0.0:5285"
docker build -t raceday-mvc ./RaceDay.MVC
docker run --rm -p 8080:8080 -e ApiSettings__BaseUrl=http://host.docker.internal:5285/ raceday-mvc


## Run it locally (without Docker)

1. Start the API with Visual Studio (http or https profile) or `dotnet run --project RaceDay.API`.
2. Store the Azure Storage connection string for the API (it is never committed):
   dotnet user-secrets set "AzureBlobStorage:ConnectionString" "<your connection string>" --project RaceDay.API
3. `RaceDay.MVC/appsettings.json` already points at the API (`"ApiSettings": { "BaseUrl": "http://localhost:5285/" }`). Run `RaceDay.MVC`.

## First steps once it is running

The database starts empty, so create your own test users:

1. Open **http://localhost:8080** and click **Join RaceDay**.
2. Register an **Organiser** (choose "Run events (Organiser)"). Log in. You land on the Organiser dashboard.
3. Create an event with a banner image, then add one or more categories.
4. Log out, register a **Participant** (choose "Enter events (Participant)"), log in, open the event and click **Enter event**.
5. Log back in as the Organiser, open **Enrolments**, then **Results**, and capture the Participant's finish time and position.
6. Log in as the Participant again and open **My Results** to see the result, for example "1st of 1 finishers".

Image uploads need a valid `RACEDAY_AZURE_CONNECTION_STRING` in `.env`. Without it the site still works but uploading a banner or profile picture shows an error.

## Viewing the database in SSMS

The SQL Server container is published on this computer's port 1433, so SQL Server Management Studio can connect to it:

| Setting | Value |
|---|---|
| Server name | `localhost,1433` |
| Authentication | SQL Server Authentication |
| Login | `sa` |
| Password | the value of `RACEDAY_DB_PASSWORD` in your `.env` |
| Options | tick **Trust server certificate** |
| Database | `RaceDayDb` |

Useful sql queries:

SELECT OrganiserID, Name, Email, Password FROM Organiser;      -- Password is a salted hash, never plain text
SELECT ParticipantID, Name, Email, Password, ProfilePictureUrl FROM Participant;
SELECT EventID, EventName, EventDate, BannerUrl FROM Event;    -- BannerUrl is the Azure Blob Storage URL
SELECT * FROM Category;
SELECT * FROM Enrolment;
SELECT * FROM Result;

### Database running in Docker (evidence)

The screenshots below were taken in SSMS connected to the SQL Server container (`localhost,1433`, login `sa`).
`RaceDayDb` is online and contains the six RaceDay tables plus EF Core's migrations history table, which is created
automatically by the API's migrations when the containers start.
<img width="626" height="967" alt="Screenshot 2026-11-07 150654" src="https://github.com/user-attachments/assets/c95eb68a-ef91-40dd-8e8c-614be122b020" />
<img width="1592" height="1003" alt="Screenshot 2026-11-07 150444" src="https://github.com/user-attachments/assets/7207de8b-2c3f-4565-9dda-f10e4a291df5" />
<img width="758" height="956" alt="Screenshot 2026-11-07 150255" src="https://github.com/user-attachments/assets/d3712b82-e71e-4b46-bc2d-7ac172f069ac" />
<img width="1605" height="965" alt="Screenshot 2026-11-07 150248" src="https://github.com/user-attachments/assets/be0540f9-f18d-4347-994c-2a7445eae0f7" />

Row counts while testing the application (the numbers change as you use the site):

| Table | Rows |
|---|---|
| Organiser | 5 |
| Participant | 7 |
| Event | 1 |
| Category | 1 |
| Enrolment | 2 |
| Result | 1 |

<img width="1062" height="980" alt="Screenshot 2026-11-07 151054" src="https://github.com/user-attachments/assets/06edb78a-094f-4ee4-8383-2bca3bdaa6d9" />
<img width="637" height="971" alt="Screenshot 2026-11-07 150831" src="https://github.com/user-attachments/assets/409e54ef-f991-4a06-ad38-98cd6a3c3bb5" />

The data was created through the MVC site, which calls the API, which writes to this database.

### Optional: loading an existing database backup

A fresh start has an empty database. To load a previous `.bak` file into the container instead
(the backup is not stored in this repository):

docker exec raceday-db mkdir -p /var/opt/mssql/backup
docker cp RaceDay.bak raceday-db:/var/opt/mssql/backup/RaceDay.bak

Then run this in SSMS (connected to `localhost,1433`):

USE master;
GO
RESTORE DATABASE RaceDayDb
FROM DISK = '/var/opt/mssql/backup/RaceDay.bak'
WITH MOVE 'RaceDayDb'     TO '/var/opt/mssql/data/RaceDayDb.mdf',
     MOVE 'RaceDayDb_log' TO '/var/opt/mssql/data/RaceDayDb_log.ldf',
     REPLACE, RECOVERY, STATS = 10;
GO

This works because the backup already contains the EF Core migrations history, so the API recognises the schema.

## Troubleshooting

* **Changed some code?** Rebuild and recreate the containers: `docker compose up -d --build`. Building the image alone does not replace a container that is already running.
* **Want a clean database?** `docker compose down -v` removes the containers and the SQL Server data volume.
* **Site loads but says it cannot reach the RaceDay service?** Check `docker compose ps`. The API container must be running (it waits for SQL Server and retries for about 90 seconds on first start).
* **Port already in use?** Stop whatever uses 8080, 5285 or 1433, or change the left-hand port numbers in `docker-compose.yml`.

## Part 2 feedback and how Part 3 addresses it

| Part 2 feedback | What was done / where to see it |
|---|---|
| Enforce the correct role on every endpoint | Every API endpoint carries `[Authorize(Roles = ...)]` or `[AllowAnonymous]`, and owner-only actions return 403 for other users. The MVC site adds `[SessionAuthorize("Organiser" / "Participant")]` on top, so each role only reaches its own pages. |
| Confirm password hashing | Passwords are hashed with ASP.NET's salted PBKDF2 hasher (`PasswordService`). The `Password` columns in SSMS contain hashes only. |
| Show registration and login for both roles, logout, and the database | Registration and login for both roles are in the MVC site; logout clears the session and revokes the JWT on the API; the SSMS section above shows how to inspect the data. |
| Keep CI running tests | `.github/workflows/ci.yml` builds the API, tests and MVC site and runs the tests on every push. |

## GitHub Actions

`.github/workflows/ci.yml` runs on every push and pull request. It validates the repository structure (and checks that no secrets file is committed),
builds the API, tests and MVC site, runs all tests, validates `docker-compose.yml` and builds both Docker images.

## Security notes

* Passwords are stored as salted hashes. The browser only holds a session cookie; the JWT stays on the server in the session.
* Secrets (database password, JWT key, Azure connection string) live in the git-ignored `.env` file, User Secrets or environment variables, never in the repository.
* Uploaded images are checked for type, size and real image content, and stored in private containers. Only the API can read them.
