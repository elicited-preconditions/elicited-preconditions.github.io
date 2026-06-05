# REAP project page

Static project page for the REAP CoRL 2026 submission. Anonymous-safe by
construction: no external font CDNs, no analytics, no author identifiers,
no GitHub organisation in the source paths.

## Deploying to anonymous GitHub Pages

1. Create a fresh GitHub account using a throw-away email address. Pick a
   handle that does not identify the lab or any author. Do not enable
   2FA-by-SMS with a personal phone number — use a TOTP authenticator
   or a temporary email-only setup.
2. Create a new public repository, e.g. `reap-corl-2026`.
3. Push the **contents of this `website/` directory** (not the directory
   itself) to the root of that repo:

   ```bash
   cd website
   git init
   git checkout -b main
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/<ANON-HANDLE>/<REPO>.git
   git push -u origin main
   ```

4. In the repo settings, enable **Pages** with the source set to `main`,
   folder `/ (root)`. GitHub will publish at
   `https://<ANON-HANDLE>.github.io/<REPO>/`.
5. The `.nojekyll` file in the repo root disables Jekyll preprocessing so
   files starting with `_` are served as-is.

## Video assets to populate

Drop your real renders into `static/videos/` using these exact names. The
existing files are 1-second "coming soon" placeholders that the page
gracefully falls back to until you replace them.

### Per-environment policy execution (5 videos, grid)

```
static/videos/open_tabletop.mp4
static/videos/confined_shelf.mp4
static/videos/multilevel_shelf.mp4
static/videos/access19.mp4
static/videos/blocks.mp4
```

Each ~15–30 s. The grid currently labels these as **2×** speed; if you
render at 1× or 4× change the `<span class="video-speed">` in `index.html`.

### Hero banner — `banner_montage.mp4`

A 15–30 s autoplaying montage of the five envs. **16:9 aspect, no audio.**
Fastest path: concatenate the five env clips (3–5 s each from each) with
ffmpeg:

```bash
ffmpeg -i open_tabletop.mp4 -i confined_shelf.mp4 -i multilevel_shelf.mp4 \
       -i access19.mp4 -i blocks.mp4 \
       -filter_complex "[0:v]trim=duration=3,setpts=PTS-STARTPTS[v0]; \
                        [1:v]trim=duration=3,setpts=PTS-STARTPTS[v1]; \
                        [2:v]trim=duration=3,setpts=PTS-STARTPTS[v2]; \
                        [3:v]trim=duration=3,setpts=PTS-STARTPTS[v3]; \
                        [4:v]trim=duration=3,setpts=PTS-STARTPTS[v4]; \
                        [v0][v1][v2][v3][v4]concat=n=5:v=1:a=0[outv]" \
       -map "[outv]" -c:v libx264 -crf 22 -preset slow \
       -movflags +faststart -an banner_montage.mp4
```

### Comparison video — `comparison_baselines.mp4`

REAP-GBFS vs BrFS on the same instance. Easiest: split-screen with
ffmpeg's `hstack`:

```bash
ffmpeg -i reap_rollout.mp4 -i brfs_rollout.mp4 \
       -filter_complex hstack=inputs=2 -c:v libx264 -crf 22 \
       -movflags +faststart -an comparison_baselines.mp4
```

Speed-label this at **8×** since BrFS will be much slower in real time.

### OOD demo — `ood_demo.mp4`

A single instance with **≥ 20 objects** (more than the training max of
14) being solved by REAP-GBFS. Demonstrates the OOD claim from the
results plot.

### Failure case — `failure_access19.mp4`

The Access-19 18-blocker held-out instance. Show REAP-GBFS expanding
search nodes until the 5-minute cap; cut to a "no plan found" overlay.
This is the rare-but-good case where showing a failure builds
credibility.

### Emergent behaviour — `behavior_parking.mp4`

A confined-shelf non-monotone problem where REAP performs a parking
move (places a cylinder at a temporary cell before moving it to its
final colour-sorted destination). Caption highlights that parking was
absent from every training rollout.

### Recommended encoding for all videos

```bash
ffmpeg -i source.mp4 -c:v libx264 -crf 24 -preset slow \
       -vf "scale=-2:720" -movflags +faststart \
       -an target.mp4
```

(720p, no audio, web-optimised headers, ~10–20 MB per minute.)

## Anonymisation checklist

Before pushing:

- [ ] No author names anywhere in HTML/CSS/JS
- [ ] No institutional links
- [ ] No GitHub org or repo path that could identify the lab
- [ ] No external CDN that logs IP (Google Fonts, GA, Statcounter, etc.)
- [ ] No favicon that is a re-used personal/lab icon
- [ ] Repo description on GitHub does not reveal authorship
- [ ] Account creation email and display name do not reveal authorship
- [ ] Video file metadata (mp4 `tags`) does not embed author / camera
      / username info — run `ffmpeg -i in.mp4 -map_metadata -1 -c copy out.mp4`
      to strip metadata before publishing
