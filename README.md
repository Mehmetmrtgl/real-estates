 Real Estates (2024 Combined Backend & Frontend)

This project bundles a Spring Boot backend and a React frontend that were originally maintained as separate repositories in 2024. It provides a simple real-estate listing workflow where dealers can manage property listings and customers can browse and view them.

## Repository layout
- `backend/demo`: Spring Boot 3.2 application (Java 21, Maven) exposing REST APIs for authentication, client management, property CRUD, and image handling.
- `frontend`: React 18 single-page application that consumes the backend APIs and provides dealer/customer flows.

## Prerequisites
- Java 21+
- Maven 3.9+ (or use the included `mvnw`/`mvnw.cmd` wrappers)
- Node.js 18+ and npm
- PostgreSQL 14+

## Database
- Default JDBC URL: `jdbc:postgresql://localhost:5432/emlak`
- The application creates and drops tables on startup/shutdown (`spring.jpa.hibernate.ddl-auto=create-drop`).
- Configure credentials in `backend/demo/src/main/resources/application.properties` via `spring.datasource.username` and `spring.datasource.password`.
- Core tables (auto-created by JPA):
  - `client`: dealers/customers with unique email, password, mobile number, and `userType` enum.
  - `properties`: property owner name, location, description, rooms, type, status, price, linked to a `client`, optional `feature`, and associated `images`.
  - `features`: bedrooms, bathrooms, lounges, storeys for a property.
  - `images`: binary image data, MIME type, filename, linked to a property.

## Backend (Spring Boot)
### Configuration
- File: `backend/demo/src/main/resources/application.properties`
  - Update PostgreSQL credentials and port if needed.
  - Default server port: 8080.
  - Example override using environment variables:
    ```bash
    export SPRING_DATASOURCE_URL="jdbc:postgresql://localhost:5432/emlak"
    export SPRING_DATASOURCE_USERNAME="postgres"
    export SPRING_DATASOURCE_PASSWORD="secret"
    ```

### Run
From `backend/demo`:
```bash
./mvnw spring-boot:run
```
This starts the API server on `http://localhost:8080`.

### Key endpoints
- **Auth**
  - `POST /login` – email/password login; returns user id and role on success.

- **Dealers** (`/dealers`)
  - `POST /add` – register dealer.
  - `GET /properties/{clientId}` – list dealer’s properties.
  - `GET /property/{propertyId}` – fetch property details (includes features/images metadata).
  - `POST /property/add` – create property with embedded features.
  - `PUT /property/{propertyId}` – update property details/features.
  - `DELETE /property/{propertyId}` – delete property.
  - `POST /image/add/{propertyId}` – upload images (multipart files array named `images`).
  - `DELETE /property/image/{imageId}` – delete a specific image.
  - `GET /property/images/{propertyId}` – fetch property images (returns first image bytes as `image/jpg`).

- **Customers** (`/customers`)
  - `POST /add` – register customer.
  - `GET /properties` – list all properties.
  - `GET /property/{propertyId}` – fetch property details.

### Domain model
- `Client`: first/last name, email (unique), password, mobile number, `UserType` (`DEALER` or `CUSTOMER`).
- `Property`: owner name, location, description, rooms, type, status, price; linked to `Client`, optional `Feature`, and `Image` collection.
- `Feature`: bathrooms, bedrooms, lounges, storeys.
- `Image`: binary data, MIME type, filename tied to a property.

## Frontend (React)
### Configuration
- API base URL defaults to `http://localhost:8080` in `src/components/ApiService/ApiService.js`. Change `BASE_URL` if the backend runs elsewhere.

### Run
From `frontend`:
```bash
npm install
npm start
```
The app runs at `http://localhost:3000` and proxies API requests directly to the backend URL.

### Application flows
- **Routing** (`src/App.js`)
  - `/` – Home landing page.
  - `/register` – user registration.
  - `/login` – login form.
  - `/customer/get-all-properties` – customer property list.
  - `/dealer*` – dealer dashboard for CRUD operations.
  - `/dealer/property-update/:propertyId` – dealer property edit form.

- **API integration** (`src/components/ApiService/ApiService.js`)
  - Fetch dealer properties: `getAllPropertiesByDealerId(clientId)` → `GET /properties/{dealerId}`.
  - Fetch single property: `getOnePropertyById(propertyId)` → `GET /property/{propertyId}`.
  - Create/update/delete properties: `POST/PUT/DELETE /property/...`.
  - Upload images: `POST /image/add/{propertyId}` with multipart payload.
  - Delete image: `DELETE /property/image/{imageId}`.

## Development tips
- The backend enables SQL logging (`spring.jpa.show-sql=true`) and formatted output to help debugging.
- CORS is configured via `WebConfig` to allow `http://localhost:3000`, matching the frontend dev server.
- Because DDL is `create-drop`, any restart clears data; adjust to `update` or `validate` for persistence.

## Combining the repositories
This README documents the combined 2024 backend and frontend codebases so you can run the full stack locally, point the React app at the Spring Boot API, and back the service with PostgreSQL.
