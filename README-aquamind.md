# PX4 Flight Review, local copy (AquaMind)

Third-party clone of https://github.com/PX4/flight_review.git, BSD-3-Clause (LICENSE.md).
Gitignored by AquaMind; not a build input. Interactive ULog review.

- Clone: HEAD aa4a1fce090ee6cd9040a22a22256d8c78536698 (commit dated 2026-10-03), cloned 2026-10-14, submodule nlohmann_json initialised.
- Venv: `/home/alan/.cache/aquamind/flight_review-venv`, built from `app/requirements.txt` (bokeh==3.8.2 pinned; pyulog 1.2.4, tornado 6.5.10 pulled in). pip printed dependency warnings about the ROS system packages (ament-black/uvloop, setuptools, typeguard); they are not part of this venv and are harmless.
- Why outside the tree: `tools/ci/test_repo_invariants.py` globs `tools/` for `test_*.py` and imports `px4_msgs`, so a venv under `tools/` (here, `tools/flight_review/.venv`) is a red build; the venv lives in `~/.cache/aquamind/` instead.

## Run
    cd /home/alan/AquaMind/tools/flight_review/app
    source /home/alan/.cache/aquamind/flight_review-venv/bin/activate
    python setup_db.py                      # once: creates ../data/logs.sqlite
    python serve.py --port 5006 --address 127.0.0.1   # then open http://127.0.0.1:5006/upload
Without `--address` it binds 0.0.0.0. Options: `-s/--show`, `-f/--file file.ulg` (show one log directly,
no upload, implies --show; `--3d`, `--pid-analysis` select other pages), `--port`, `--address`,
`--num-procs`, `--allow-websocket-origin`, `--use-xheaders`.

Direct CLI path: `python serve.py -f /path/to/log.ulg` (opens /plot_app?log=<abs path>; picks the next free port if 5006 is busy).

Uploads: stored under `tools/flight_review/data/log_files/` (+ `data/cache/`, `data/logs.sqlite`).
Scripted upload (verified 2026-10-14): multipart POST to /upload, the file field is named `filearg`
(not `file`), plus `description`, `email` (may be empty), `source=webui`, `type=personal`, `redirect=true`;
reply is 302 to `/plot_app?log=<uuid>`. A wrong field name gives a bare 500 with no traceback.
Smoke test: log_56_UnknownDate.ulg uploaded and /plot_app returned 200 (51 KB page, "Altitude Estimate" present).
Airframe/parameter metadata is fetched from px4-travis S3 URLs in config_default.ini (needs network).

## "Barometer Altitude"
The altitude plot's "Barometer Altitude" series is `vehicle_air_data.baro_alt_meter`:
app/plot_app/configured_plots.py:37 selects topic `vehicle_air_data` (falls back to `sensor_combined`
for old logs, :40), and :193-194 do `data_plot.change_dataset(baro_alt_meter_topic)` then
`add_graph(['baro_alt_meter'], ..., ['Barometer Altitude'])`. It is plotted raw, with no datum or
air-pressure-to-depth handling, so an underwater log's baro field is only meaningful relative to what
the FCU computed; the spike is in that uORB field, not a Flight Review artefact.

## Fork and branches
- Fork: https://github.com/Aquaairx-software/flight_review (fork of PX4/flight_review, created 2026-10-14). Default branch: `dev`.
- Remotes: `origin` = git@github.com:Aquaairx-software/flight_review.git (ssh); `upstream` = https://github.com/PX4/flight_review.git (https).
- Working branch: `dev`, branched 2026-10-14 from upstream base aa4a1fce (commit dated 2026-10-03). Upstream default branch is `main`.
- Rule: work on and push to `dev`. Pull upstream with `git fetch upstream && git merge upstream/main` on `dev`.
- This directory stays gitignored in AquaMind exactly like PX4-Autopilot/: the fork, not the AquaMind repo, owns its history.
