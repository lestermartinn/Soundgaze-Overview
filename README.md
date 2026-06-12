# Soundgaze

**An interactive 3D system for *exploring* a music collection instead of being recommended one**; every track is a point in space, and proximity means sonic similarity.

This repo is the home for the project. Soundgaze exists in two versions, built for different purposes:

- **Soundgaze (v1):** the original, built at a hackathon. A rendering-and-performance build focused on getting a large point cloud navigable in the browser at speed.
- **Soundgaze 2.0:** a from-scratch research extension and deployed system. It rebuilds the similarity space on a perceptually validated embedding, formally compares three dimensionality-reduction methods, and ships as a live web app. Written up as a short paper (included here).

---

## Links

| | |
|---|---|
| **Live app (Soundgaze 2.0)** | https://soundgaze2.vercel.app/ |
| **Paper (Soundgaze 2.0)** | [Soundgaze_2.0.pdf](./Soundgaze_2.0.pdf) |
| **Soundgaze 2.0 Source** | https://github.com/lestermartinn/sgazev2 |
| **Soundgaze (v1, hackathon) Source** | https://github.com/lestermartinn/Soundgaze |

> The deployed app runs a 200-song subset for hosting cost; the full pipeline and evaluation use the 8,000-track FMA Small dataset.

---

## Soundgaze (v1): the hackathon build

Built at Hacklytics 2026. The focus was making a large song point cloud feel fluid to navigate in a browser.

Highlights:
- Rendered a 100K+ song point cloud in 3D with **React Three Fiber**, projecting every point to screen-space NDC coordinates each frame for hover detection — sub-20px accuracy at 60fps, with no raycasting overhead.
- Built a **UMAP** pipeline compressing audio feature vectors into a navigable 3D space, with a "Journey" mode that traverses the cloud via a random walk over the similarity graph.

**Stack:** Next.js · TypeScript · React Three Fiber · FastAPI · Gemini · UMAP

---

## Soundgaze 2.0: the research + deployed version

Where v1 asked *"can we render this fast,"* 2.0 asks *"is the space we're rendering actually perceptually meaningful, and what's the best way to flatten it to 3D?"* It answers both with a measured pipeline and a deployed tool.

### What it does
Maps 8,000 tracks (FMA Small) into a navigable 3D point cloud where proximity reflects sonic similarity, lets you click any track to surface its nearest neighbors, and lets you switch the 3D layout between PCA, t-SNE, and UMAP on the fly.

### Technical highlights
- **Hybrid 578-D embedding.** Concatenates a 512-D **CLAP** semantic embedding with a 66-D hand-crafted acoustic vector (MFCCs, chroma, spectral contrast, rhythm), magnitude-weighted so cosine similarity on the hybrid is a clean convex combination of the two component similarities — no custom distance function needed.
- **Validated against human perception.** The hybrid space beats raw CLAP on every metric of the **Timbremetrics** and **Covers80** perceptual benchmarks before it's used for anything else.
- **A real DR comparison.** PCA, t-SNE, and UMAP are evaluated head-to-head on structural (Trustworthiness, KNN Recall) and perceptual axes. t-SNE preserves local neighborhood structure best (Trustworthiness 0.977, KNN Recall 0.224); PCA is weakest; all three lose substantial fidelity at 3D.
- **The key design decision.** Because even the best 3D projection recovers only ~22% of the true neighborhood, **nearest-neighbor search runs on the full 578-D space**, and the chosen reduction governs *only* the visual layout. Switching PCA/t-SNE/UMAP rearranges the same neighbors rather than changing who the neighbors are.

### Architecture
- **Backend:** Python · FastAPI · Uvicorn. Lazily computes and caches embeddings + the three precomputed 3D reductions (Parquet/NumPy on disk); a brute-force scikit-learn `NearestNeighbors` index over the hybrid embedding serves all queries in a few ms at 8K tracks.
- **Frontend:** Next.js 14 · TypeScript · React Three Fiber / Three.js. Prefetches all three coordinate sets in parallel so layout switches are an instant tween with no network round-trip.

---

## The paper

[`Soundgaze_2.0.pdf`](./Soundgaze_2.0.pdf) — *Soundgaze 2.0: Evaluating Dimensionality Reduction Methods for Interactive 3D Audio Similarity Exploration.* Independent project, Emory CS, Spring 2026. Distributed as a preprint; not submitted to or accepted at any venue.

---

## Credits

- **Soundgaze (v1):** Lester Martin, Parker Wischhover, Rithwik Sharma, Aazam Alam
- **Soundgaze 2.0:** Lester Martin, Parker Wischhover