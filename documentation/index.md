# Documentation

[Project README](../README.md)

These documents describe the implementation in the repository. They do not establish that a particular deployment, provider account, browser, or release has passed acceptance. Source references identify where to verify behavior when the implementation changes.

For a short overview of the product, vision, and next steps, read the [one-page solution pitch](solution-pitch.md) or its [PDF export](../output/pdf/samvaad-solution-pitch.pdf).

## For students

Start with the [user guide](user-guide.md). It explains account access, choosing a round, the microphone check, controls, reports, history, and connection problems. The application's `/privacy` page identifies the configured providers and operator contact.

## For developers and operators

| Document                           | Owns                                                                                |
| ---------------------------------- | ----------------------------------------------------------------------------------- |
| [Development](development.md)      | Prerequisites, local startup, source navigation, and commands                       |
| [Configuration](configuration.md)  | Environment variables, defaults, and process-specific settings                      |
| [Architecture](architecture.md)    | Components, trust boundaries, audio/evidence flow, and processing lifecycle         |
| [API](api.md)                      | HTTP endpoints, validation, errors, and owned WebSocket messages                    |
| [Database](database.md)            | Tables, ordered SQL changes, permissions, constraints, and historical compatibility |
| [Operations](operations.md)        | Deployment procedure, safe diagnostics, recovery, and cost controls                 |
| [Validation](validation.md)        | Automated checks, manual acceptance, and known implementation limits                |
| [Contributing](../CONTRIBUTING.md) | Scope, review, testing, and documentation responsibilities                          |
| [Security](../SECURITY.md)         | Security boundaries, private reporting, and data-handling precautions               |

For onboarding, read Development, Configuration, and Architecture. Read API and Database before changing session behavior. Read Operations and Validation before enabling or deploying voice practice.

Keep environment details in Configuration, behavior contracts in API/Architecture/Database, and release evidence outside claims about implemented functionality. Update the owning document and link to it from other pages instead of copying the same rule into several places.
