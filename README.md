# Aman Gautam

Pre-final year student (Integrated Dual Degree) at IIT (BHU) Varanasi. I build data infrastructure and analysis tools for Indian equity markets, mostly in Rust and Python, and write C++ game and simulation code on the side.

Much of my recent work starts from the same problem: NSE and BSE publish a great deal of useful data, but through undocumented endpoints, inconsistent XBRL tagging, and formats that change without notice. I try to get that layer right first then verified against live behaviour, explicit about what it doesn't know before building analysis on top of it.

---

## Selected work

### [jugaad-rs](https://github.com/Am1n1602/jugaad-rs) — NSE market data library and CLI in Rust

`Rust` `tokio` `tonic / gRPC` `Docker` `polars`

An idiomatic Rust rewrite of the Python [`jugaad-data`](https://github.com/jugaad-py/jugaad-data) library, structured as a Cargo workspace:

- **`jugaad-core`** — async client library covering historical and live data: whole-market and F&O bhavcopy, stock / index / derivatives history, live quotes with order-book depth, option chains, market movers, corporate announcements, and financial-results filings with XBRL download.
- **`jugaad`** — a 47-command CLI that writes clean CSV for every endpoint.
- **`jugaad-rpc`** — a gRPC server exposing the core library to non-Rust clients, published as a container image on GHCR, with working Python and Node.js example clients.

Design points:

- NSE has no public API documentation, so every endpoint was verified against live responses, including a retest during market hours. Undocumented behaviour is recorded in [`docs/nse-findings.md`](https://github.com/Am1n1602/jugaad-rs/blob/main/docs/nse-findings.md) rather than inferred from the Python source.
- Bhavcopy calls transparently handle NSE's July 2024 format change, so callers never need to know which side of it a date falls on.
- Network layer with connect and request timeouts and exponential backoff with jitter on transient failures; bot-protection 403s surface as a distinct error and are deliberately not retried.
- Typed error model that distinguishes "not found", "no data published yet" and "blocked" instead of collapsing them into one failure.
- Optional `polars` DataFrame conversion behind a feature flag; prebuilt binaries for Linux, macOS (Intel and Apple Silicon) and Windows on every release.

### [Fin_QA](https://github.com/Am1n1602/Fin_QA) — question answering over Indian company filings

`Python` `SQLite` `FAISS` `sentence-transformers` `Ollama / Groq` `FastAPI` `React`

A local-first system that turns raw NSE/BSE regulatory filings into answers to questions such as *"Compare AXISBANK and HDFCBANK on leverage"* or *"Why did HCLTECH's profitability decline?"*, across the live NIFTY 50 universe.

- **Numbers never come from the LLM.** A deterministic engine computes every ratio, trend, peer percentile, composite ranking score, Piotroski F-Score and partial Altman Z''. The language model only writes up figures that have already been computed, and each numeric claim it makes is checked against source values and units before it is shown.
- **Routing by question type.** Factual, trend and ranking questions resolve to structured database queries with no LLM involved. Narrative questions go through hybrid retrieval over filing PDFs (dense embeddings + BM25 + phrase overlap, cross-encoder reranking).
- **No silent approximation.** XBRL facts that can't be mapped cleanly to the canonical schema are stored as null with a recorded reason — never zero-filled or proxied.
- Computed figures for TCS were cross-checked against independent sources (GuruFocus, Value Research, Tickertape).
- Ships as a pip-installable CLI with OS-level scheduled data refresh. A FastAPI backend and React dashboard are in progress.

### Games and simulation

| Project | Description |
| --- | --- |
| [Just Another Easy Game](https://github.com/Am1n1602/Just-another-easy-game) | 2D platformer in C++17 and raylib, built without an engine: movement and collision, traps, level design, game-state management and audio. Playable in the browser on [itch.io](https://peepow.itch.io/just-another-easy-game), with an online leaderboard served by Node.js / Express and MongoDB. |
| [ParticleCollision](https://github.com/Am1n1602/ParticleCollision) | Interactive 2D simulation of charged particles with attraction and repulsion, elastic collisions, and a live centre-of-mass readout. |
| [FP-Movement](https://github.com/Am1n1602/FP-Movement) | First-person camera and movement controller in raylib. |
| [Just-Another-World-Godot](https://github.com/Am1n1602/Just-Another-World-Godot) | A 2D project in Godot with a web export, used to compare an engine-based workflow against writing the systems by hand. |

---

## Technical skills

| | |
| --- | --- |
| **Languages** | Rust, C++, Python, C, Kotlin, JavaScript, SQL |
| **Systems and backend** | tokio, tonic / gRPC, FastAPI, Node.js / Express, Docker, SQLite, MongoDB |
| **Data and ML** | pandas, polars, NumPy, scikit-learn, LightGBM, PyTorch |
| **Retrieval and LLMs** | FAISS, sentence-transformers, BM25, cross-encoder reranking, Ollama, Groq, Anthropic API |
| **Graphics and games** | raylib, Godot, OpenGL, SDL2 |
| **Tooling** | Git, CMake, Cargo, GitHub Actions, Linux |

## Interests

- **Financial data engineering** — filings, XBRL, exchange data, and reliable pipelines over sources that were never designed to be consumed programmatically.
- **Machine learning** — currently studying deep learning, with an eye to applying it to financial time series and documents.
- **Systems programming** — performance-oriented Rust and C++, and understanding what the code actually does at runtime.
- **Game engines and graphics** — physics, rendering, and how engine architecture is put together.

---

## Contact

[amangautam1602@gmail.com](mailto:amangautam1602@gmail.com) · [LinkedIn](https://www.linkedin.com/in/am1n-gautam/) · [X](https://x.com/Am1n1602) · [itch.io](https://peepow.itch.io/just-another-easy-game)
