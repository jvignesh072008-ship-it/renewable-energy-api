# renewable-energy-api
A Spring Boot microservices application for managing a renewable energy grid — solar panels, wind turbines, battery storage, and load distribution — with an API gateway routing requests across four independent services, each backed by its own H2 database.

# Renewable Energy Management API

A Spring Boot microservices project that manages solar panels, wind turbines, batteries, and energy distribution for a renewable energy grid. Each domain is its own independently deployable service, backed by an in-memory H2 database, with a gateway routing requests to all of them.

## Repository Name

`renewable-energy-api`

(Suggested GitHub repo title: **Renewable Energy Management API — Capstone Project**)

## Tech Stack

- Java 26
- Spring Boot 4.1.0
- Spring Framework 7
- Spring Cloud Gateway (2025.1.2)
- Spring Data JPA / Hibernate
- H2 Database (in-memory)
- Lombok 1.18.46
- Maven 3.9.16

## Architecture

```
                     ┌────────────────────┐
                     │    api-gateway       │
                     │   (port 8080)         │
                     └─────────┬───────────┘
        ┌───────────┬──────────┼──────────┬──────────────┐
        ▼           ▼          ▼          ▼
solar-panel-  wind-turbine- battery-   energy-distribution-
service       service       service    service
(8081)        (8082)        (8083)     (8084)
```

- **solar-panel-service** — manages solar panel devices and their generation records
- **wind-turbine-service** — manages wind turbine devices and their generation records
- **battery-service** — manages battery storage devices, charge/discharge transactions, and low-charge alerts
- **energy-distribution-service** — balances load against live solar/wind output and battery reserves, and produces daily reports/fault summaries by calling the other three services
- **api-gateway** — single entry point routing requests to the four services above

## Project Structure

```
renewable-energy-api/
├── solar-panel-service/
├── wind-turbine-service/
├── battery-service/
├── energy-distribution-service/
├── api-gateway/
├── docs/
│   └── postman/
└── README.md
```

Each service module is a standalone Maven project (its own `pom.xml`, `mvnw`, `mvnw.cmd`) — there is no aggregator/parent POM, so each is built and run independently.

## Prerequisites

- Java 26 (JDK 26)
- Maven 3.9.16 (or use the bundled `mvnw` wrapper)

Verify with:
```
java -version
mvn -version
```

## Building and Running

Each service is built and started from its own folder. Example for solar-panel-service:

```
cd solar-panel-service
mvnw clean install -DskipTests
mvnw spring-boot:run
```

Repeat for `wind-turbine-service`, `battery-service`, `energy-distribution-service`, and finally `api-gateway` (start the gateway last, since it routes to the other four).

### Ports

| Service | Port |
|---|---|
| solar-panel-service | 8081 |
| wind-turbine-service | 8082 |
| battery-service | 8083 |
| energy-distribution-service | 8084 |
| api-gateway | 8080 |

## H2 Console

Each data-backed service exposes its own H2 console once running:

| Service | Console URL | JDBC URL |
|---|---|---|
| solar-panel-service | http://localhost:8081/h2-console | `jdbc:h2:mem:solar_db` |
| wind-turbine-service | http://localhost:8082/h2-console | `jdbc:h2:mem:wind_db` |
| battery-service | http://localhost:8083/h2-console | `jdbc:h2:mem:battery_db` |
| energy-distribution-service | http://localhost:8084/h2-console | `jdbc:h2:mem:distribution_db` |

Login: user `sa`, password blank. Data is in-memory and resets on service restart.

## API Reference

### solar-panel-service (`http://localhost:8081`)

| Method | Path | Description |
|---|---|---|
| POST | `/solar-panels` | Create a solar panel |
| GET | `/solar-panels` | List all solar panels |
| GET | `/solar-panels/{id}` | Get a panel by id |
| PUT | `/solar-panels/{id}` | Update a panel |
| DELETE | `/solar-panels/{id}` | Delete a panel |
| POST | `/solar-panels/{id}/generation` | Log a generation record |
| GET | `/solar-panels/{id}/generation` | List generation records for a panel |
| GET | `/solar-panels/faults` | List panels currently under maintenance |
| GET | `/solar-panels/reports/daily-total` | Daily generation total |

### wind-turbine-service (`http://localhost:8082`)

| Method | Path | Description |
|---|---|---|
| POST | `/wind-turbines` | Create a wind turbine |
| GET | `/wind-turbines` | List all turbines |
| GET | `/wind-turbines/{id}` | Get a turbine by id |
| PUT | `/wind-turbines/{id}` | Update a turbine |
| DELETE | `/wind-turbines/{id}` | Delete a turbine |
| POST | `/wind-turbines/{id}/generation` | Log a generation record |
| GET | `/wind-turbines/{id}/generation` | List generation records for a turbine |
| GET | `/wind-turbines/faults` | List turbines currently under maintenance |
| GET | `/wind-turbines/reports/daily-total` | Daily generation total |

### battery-service (`http://localhost:8083`)

| Method | Path | Description |
|---|---|---|
| POST | `/batteries` | Create a battery |
| GET | `/batteries` | List all batteries |
| GET | `/batteries/{id}` | Get a battery by id |
| PUT | `/batteries/{id}` | Update a battery |
| DELETE | `/batteries/{id}` | Delete a battery |
| GET | `/batteries/available` | List batteries available (not under maintenance) |
| POST | `/battery/charge` | Charge a battery |
| POST | `/battery/discharge` | Discharge a battery |
| GET | `/battery/status` | Status summary of all batteries |
| GET | `/battery/status/{id}` | Status of a single battery |
| GET | `/battery/alerts` | List low-charge alerts |

### energy-distribution-service (`http://localhost:8084`)

| Method | Path | Description |
|---|---|---|
| POST | `/distribution/balance` | Balance a requested load against live solar/wind output and battery reserves |
| GET | `/reports/daily` | Combined daily report across all services |
| GET | `/faults` | Combined fault/alert list across all services |

### api-gateway (`http://localhost:8080`)

Routes requests through to the corresponding backend service (start the four services above first):

| Path prefix | Routed to |
|---|---|
| `/solar-panels/**` | solar-panel-service |
| `/wind-turbines/**` | wind-turbine-service |
| `/batteries/**`, `/battery/**` | battery-service |
| `/distribution/**`, `/reports/**`, `/faults` | energy-distribution-service |

## Testing with Postman

1. Import or manually create requests for the endpoints listed above.
2. Set `Content-Type: application/json` and use raw JSON bodies for POST/PUT requests.
3. Create parent records first (panels, turbines, batteries) before posting dependent records (generation, charge/discharge), since those require a valid parent `id`.
4. Verify results either through the corresponding `GET` endpoint or the H2 console.

### Postman Screenshots

<img width="1920" height="1080" alt="Screenshot 2026-07-21 092034" src="https://github.com/user-attachments/assets/8ee3a3bd-cef7-4188-95cf-813e6eb8cff8" />
<img width="1920" height="1080" alt="Screenshot 2026-07-21 092122" src="https://github.com/user-attachments/assets/59d20296-429b-45b2-b0bb-46054860b166" />
<img width="1920" height="1080" alt="Screenshot 2026-07-21 092151" src="https://github.com/user-attachments/assets/de19e065-0d67-4093-971f-d99bde88d1f0" />
<img width="1920" height="1080" alt="Screenshot 2026-07-21 092234" src="https://github.com/user-attachments/assets/1c2b2807-8644-47ab-98eb-999dffcc076f" />
<img width="1920" height="1080" alt="Screenshot 2026-07-21 092356" src="https://github.com/user-attachments/assets/b207fbd4-f7f0-44ec-a44e-266b37004de1" />
<img width="1920" height="1080" alt="Screenshot 2026-07-21 092441" src="https://github.com/user-attachments/assets/38efe194-5a14-4578-8403-2478425e5618" />
<img width="1920" height="1080" alt="Screenshot 2026-07-21 092503" src="https://github.com/user-attachments/assets/2ebf1750-b598-4cde-b5be-444335d12f0c" />
<img width="1920" height="1080" alt="Screenshot 2026-07-21 092607" src="https://github.com/user-attachments/assets/9a746baa-a102-4d6e-abfa-90e97aa7be40" />
<img width="1920" height="1080" alt="Screenshot 2026-07-21 092633" src="https://github.com/user-attachments/assets/d522a01b-984c-4865-9c35-f43a47f0fe2e" />

Screenshot files live under "postman" in the repository.

## Author
```
Name: Vignesh J
Register Number: 212225230297
Department: AI&DS
```
