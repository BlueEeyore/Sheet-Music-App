# HighScore

An offline-first Android sheet-music library, reader and annotation app, designed for phones and tablets with a future iPad port in mind.

This repository currently contains the specification, design and implementation handoff. The app has not been implemented or validated on devices yet.

## Implementation handoff

Start with the [detailed implementation plan](docs/IMPLEMENTATION_PLAN.md). It defines 17 ordered milestones, module boundaries, failure handling, verification requirements and completion criteria.

- [Behavior and architecture](docs/DESIGN.md)
- [Confirmed user decisions](docs/DESIGN_DECISIONS.md)
- [Acceptance tests and release evidence](docs/ACCEPTANCE_TESTS.md)
- [Reliability requirements](docs/RELIABILITY_REQUIREMENTS.md)
- [Original specification](SPEC.md)
- [Existing visual reference](docs/mockups/highscore-review-v2.html)

Later written decisions override the original specification and historical mockup interactions. In particular, stylus contact opens both toolbars, and protected setlist access lasts until leaving that setlist. Backgrounding or screen-off does not revoke a live access session.

The selected interface uses Liberation Sans, green `#2F7434`, navy `#18243A`, and a pale-green `#D9F2D7` library plus glyph. No further mockups are required.

Reliability is the highest priority. All retained features belong in the first release; milestones are an implementation sequence, not a reduced release scope. The user manages git commits.
