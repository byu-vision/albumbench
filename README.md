# AlbumBench Dataset

[Project page](https://byu-vision.github.io/albumbench/)

This repo contains the AlbumBench annotated dataset built on top of CUFED.
It includes album inventories, task annotations, source labels, and train/test
split membership for vision-language album understanding tasks.

## CUFED Images

The CUFED images can be downloaded from [Hugging Face](https://huggingface.co/datasets/Shawn-Huang/CUFED-AlbumBench).

## Files

### `albums.jsonl`

Album inventory. Each line is one album. This file defines the image IDs used by
the rest of the dataset by mapping each album-local `image_id` to an image
`path`.

Main fields:

- `album_id`: Stable CUFED album identifier. This is the join key used by the
  other files.
- `event_type`: CUFED event category for the album, such as `Wedding`,
  `Birthday`, or `ThemePark`.
- `num_images`: Number of images in the album.
- `images`: Ordered image records for the album. Each image has:
  - `image_id`: Album-local image ID such as `001`, `026`, or `098`. These IDs
    are assigned from the original CUFED image identifiers after sorting them in
    ascending order.
  - `path`: Image path relative to the CUFED image root.
  - `metadata.source_filename`: Original image filename.
- `metadata`: Conversion metadata, including `source`, `split`, and the
  `image_root` recorded when the annotations were generated.

Current snapshot: 641 albums and 26,973 image records.

### `tasks.jsonl`

Flat runtime task view consumed by evaluation. Each line is one benchmark task.

Main fields:

- `task_id`: Unique task identifier in the form
  `cufed:<album_id>:<task_type>:<source_task_type>`.
- `album_id`: Links the task to an album in `albums.jsonl`.
- `task_type`: AlbumBench task type.
- `prompt`: The natural-language query shown to the model.
- `image_ids`: Album-local image IDs available for the task.
- `target`: Ground-truth answer for scoring. Its shape depends on `task_type`.
- `metadata.source_task_type`: Original CUFED query name, such as
  `selection_query_1`, `rating_query_2`, or `grouping_query_1`.

Task types in this snapshot:

- `intent_selection`: Select all images matching the prompt.
  `target.selected_images` contains the correct image IDs.
- `intent_rating`: Rate each image for relevance to the prompt on a 0-3 scale.
  `target.images` and `target.ratings` are aligned lists.
- `group_labeling`: Assign images to named groups. `target.groups` contains
  group objects with `category` and `images`, and `target.total_groups` records
  the number of groups. When available, `metadata.categories` lists the source
  categories.

Current snapshot: 5,124 tasks: 1,923 `intent_selection`, 1,923
`intent_rating`, and 1,278 `group_labeling`. Most albums have eight tasks
across three selection queries, three rating queries, and two grouping queries;
four albums do not have `grouping_query_2`.

### `task_labels.jsonl`

Normalized source labels preserved from CUFED. Each line is one album and keeps
the source annotations grouped by their original CUFED query names.

Main fields:

- `album_id`: Links the labels to `albums.jsonl` and `tasks.jsonl`.
- `source`: Source dataset name, currently `CUFED`.
- `intent_selection`: Original selection-query labels keyed by
  `selection_query_1` through `selection_query_3`.
- `intent_rating`: Original rating-query labels keyed by `rating_query_1`
  through `rating_query_3`.
- `grouping`: Original grouping-query labels keyed by `grouping_query_1` and,
  when present, `grouping_query_2`.

Use this file when you need to inspect or audit the normalized CUFED labels.
Use `tasks.jsonl` for the flat task records passed to evaluation.

Current snapshot: 641 album-level label records.

### `splits.json`

Named dataset splits. This is a regular JSON file, not JSONL.

Main fields:

- `train`: Album IDs assigned to the training split.
- `test`: Album IDs assigned to the test split.

These IDs match `album_id` in `albums.jsonl`, `tasks.jsonl`, and
`task_labels.jsonl`. The same split membership is also mirrored in
`albums.jsonl` under `metadata.split`.

The test split includes two hold-out event types that do not appear in the
training split:

- `CasualFamilyGather`: casual family gatherings.
- `BeachTrip`: beach trips.

Current snapshot: 508 train albums and 133 test albums.

## How The Files Relate

1. Start with `splits.json` to choose train or test album IDs.
2. Use `albums.jsonl` to find each album's images and relative image paths.
3. Use `tasks.jsonl` to run evaluation tasks for those albums.
4. Use `task_labels.jsonl` when you need the album-level CUFED labels
   before they were expanded into flat runtime tasks.

All image references in `tasks.jsonl` and `task_labels.jsonl` are
album-local IDs. Resolve them through the matching album's `images` list in
`albums.jsonl`.
