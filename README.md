# Online Student Attendance & Academic Monitoring System

A web app for a college to record attendance and internal marks in one place.
It flags students who are falling behind early and shows the reason for each flag, so their mentor can step in before exam eligibility is checked.

## Process model

Agile Scrum with 2 sprints. Sprint 1 covers requirements, design, and planning; Sprint 2 covers implementation and testing.

## Tools

- **Python / Flask & Bootstrap** for the web application
- **SQLite** (development) and **MySQL** (production) for the database
- **Mermaid & draw.io** for system diagrams

## Folder structure

| Folder | Contents |
|---|---|
| `requirements/` | SRS, use cases, survey, sequence & state diagrams, ethics |
| `planning/` | Project plan, WBS, risks |
| `diagrams/` | Architecture, DFD, ER diagrams and data dictionary |
| `src/` | Flask app, schemas, synthetic seed data, and tests |
| `report/` | Test case table and reports |
| `presentation/` | Presentation materials |

## Setup

See [`src/README.md`](src/README.md) for local setup and running the 22 unit/functional tests.

Use made-up data only. Never commit real student data, passwords or `.env` files.

## License

MIT. See [LICENSE](LICENSE).
