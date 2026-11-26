# robolake

**Open-source pipelines to ingest, time-align, validate and serve multimodal robot data for training and fleet operations.**

> 🚧 Under active development

## The problem

Robots log camera feeds, joint telemetry and action commands at different rates, on different clocks, in different formats (MCAP / ROS 2 bags, HDF5). Before any of it can train a policy, someone has to align it, catch bad episodes and convert it. `robolake` does that step.

## What it will do

- **Ingest** MCAP / ROS 2 bags, HDF5 and LeRobot datasets
- **Time-align** multi-rate streams onto a common timeline
- **Validate** episodes with robot-specific quality checks (frame drops, timestamp gaps, joint limits, action lag)
- **Store** telemetry in Parquet / Apache Iceberg, queryable with DuckDB
- **Export** training-ready LeRobot v3 datasets to the Hugging Face Hub
- **Serve** data through a CLI, a REST API and a live demo

## Roadmap

- [x] Project setup
- [ ] Explore real robot data
- [ ] Canonical data model
- [ ] Ingestion connectors
- [ ] Time-alignment engine
- [ ] Validation and quality reports
- [ ] Storage and LeRobot export
- [ ] CLI, API and Hugging Face Space demo
- [ ] Streaming fleet ingestion
- [ ] PyPI release and docs

## License

Apache 2.0