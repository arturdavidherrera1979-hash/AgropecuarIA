# Data model (draft)

Entities
- Plot (field/parcel)
- Crop cycle (season)
- Dataset (capture event)
- Raw asset (image/sensor file)
- Processed asset (maps/indices)
- Insight (alert/recommendation)
- Action (spray/irrigation/scouting)
- Outcome (notes/metrics)

MVP minimum fields
Dataset:
- dataset_id, plot_id, captured_at, capture_type, operator, notes
- raw_assets[], processed_assets[]
