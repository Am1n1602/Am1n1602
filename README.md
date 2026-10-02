<h1 align="center">Aman Gautam</h1>

<p align="center">
  Software developer and student at IIT (BHU) Varanasi.<br>
  Systems programming, data and machine learning, and games.
</p>

<p align="center">
  <a href="https://www.linkedin.com/in/am1n-gautam/"><img alt="LinkedIn" src="https://img.shields.io/badge/LinkedIn-0A66C2?style=flat-square"></a>
  <a href="mailto:amangautam1602@gmail.com"><img alt="Email" src="https://img.shields.io/badge/Email-EA4335?style=flat-square&logo=gmail&logoColor=white"></a>
  <a href="https://x.com/Am1n1602"><img alt="X" src="https://img.shields.io/badge/X-000000?style=flat-square&logo=x&logoColor=white"></a>
  <a href="https://peepow.itch.io/just-another-easy-game"><img alt="itch.io" src="https://img.shields.io/badge/itch.io-FA5C5C?style=flat-square&logo=itchdotio&logoColor=white"></a>
</p>

<p align="center">
  <img alt="Profile views" src="https://komarev.com/ghpvc/?username=Am1n1602&label=Profile%20views&color=0e75b6&style=flat-square">
  <img alt="GitHub followers" src="https://img.shields.io/github/followers/Am1n1602?style=flat-square&logo=github&label=Followers">
</p>

<p align="center">
  <a href="#about">About</a> ·
  <a href="#selected-projects">Projects</a> ·
  <a href="#games-and-graphics">Games</a> ·
  <a href="#tech-stack">Tech stack</a> ·
  <a href="#github-activity">Activity</a> ·
  <a href="#contact">Contact</a>
</p>

## About

I'm in the pre-final year of an Integrated Dual Degree at IIT (BHU) Varanasi. I like software that sits close to how things really work: network clients, data pipelines, game loops. Most of my code is in Rust, Python, TypeScript and C++, and I pick whichever fits the problem.

One habit I try to keep is checking my assumptions against reality before I build on top of them, and having a program say "I don't know" instead of guessing. In practice that has meant testing against live responses, storing missing values as null with a reason attached, and writing down what I found rather than what I assumed.

## Selected projects

| Project | What it is | Built with |
| --- | --- | --- |
| [Fin_QA](#fin_qa) | Question answering over NSE and BSE filings, with a deterministic financial engine under the LLM | Python, FastAPI, React, PostgreSQL |
| [jugaad-rs](#jugaad-rs) | NSE market data as a Rust library, a CLI, a gRPC server and a Python package | Rust, tokio, tonic |
| [Audio Notes Platform](#audio-notes-platform) | Upload a recording, get a transcript and a structured summary | Next.js, FastAPI, PostgreSQL, Cloud Run |
| [OpenTerminal](#openterminal-india-market-fork) | Indian-market data layer for an open-source terminal-style dashboard (a fork) | TypeScript, Next.js, Express |

### [Fin_QA](https://github.com/Am1n1602/Fin_QA)

[![Live demo](https://img.shields.io/badge/Live%20demo-open-2563eb?style=flat-square)](https://am1n1602.me/finqa-v2)
[![CI](https://img.shields.io/github/actions/workflow/status/Am1n1602/Fin_QA/ci.yml?style=flat-square&label=CI)](https://github.com/Am1n1602/Fin_QA/actions/workflows/ci.yml)
![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)

Ask it *"Compare RELIANCE and ONGC on leverage"* or *"Why did HCLTECH's profitability decline?"* and it answers with the calculation behind every number and the filing passage behind every claim. It covers all 50 NIFTY 50 companies.

- The language model never computes or invents a figure. A plain Python engine does the arithmetic over normalised XBRL data, and the model only writes up what the engine and the retrieval layer produced, citing the evidence for each sentence.
- A separate verification pass recomputes the calculations and checks every citation before an answer goes out. If it can't confirm something, the answer is downgraded or the system abstains. A metric that can't be computed cleanly comes back as null with the reason, never as zero or an approximation.
- On my 51-question regression set, run with no LLM calls, numerical accuracy and claim groundedness are both 100%, correct abstention is 98.0% and overall correctness is 86.1%. There is also an 850-question internal benchmark and 547 unit tests.
- One result surprised me: plain BM25 beat both dense and hybrid retrieval on this corpus (Recall@5 of 68.2% against 40.9% for dense and 50.0% for hybrid). I put it down to near-duplicate boilerplate across quarterly filings diluting embedding similarity. A cross-encoder reranker helped at pilot scale, but it stays off by default until I've re-validated it on the full corpus.
- The demo runs on Cloud Run and Firebase Hosting, both scaled to zero, so the first visit after a quiet spell waits about two minutes while the 261,479-chunk corpus loads.

### [jugaad-rs](https://github.com/Am1n1602/jugaad-rs)

[![CI](https://img.shields.io/github/actions/workflow/status/Am1n1602/jugaad-rs/ci.yml?style=flat-square&label=CI)](https://github.com/Am1n1602/jugaad-rs/actions/workflows/ci.yml)
[![Release](https://img.shields.io/github/v/release/Am1n1602/jugaad-rs?style=flat-square&label=release)](https://github.com/Am1n1602/jugaad-rs/releases/latest)
[![PyPI](https://img.shields.io/pypi/v/sauda?style=flat-square&label=PyPI%20%C2%B7%20sauda)](https://pypi.org/project/sauda/)
![License](https://img.shields.io/badge/license-MIT-blue?style=flat-square)

A Rust rewrite of the Python [jugaad-data](https://github.com/jugaad-py/jugaad-data) library for downloading historical and live data from the National Stock Exchange of India. One core library, used four ways: as a Rust crate, through a 47-command CLI that writes CSV, through a gRPC server with 10 RPCs that is published as a container image, and from Python through `sauda`, a pip package that bundles the server and starts it for you.

NSE has no official API documentation, so I checked every endpoint against live responses, including a retest while the market was open, and logged what I found in [`docs/nse-findings.md`](https://github.com/Am1n1602/jugaad-rs/blob/main/docs/nse-findings.md). The project's rule is to verify NSE behaviour live before writing code around it, not to infer it from the Python library.

- Transient failures are retried with exponential backoff and jitter. A 403 from NSE's bot protection comes back as its own `Blocked` error and is not retried, because the session itself is flagged.
- Errors are typed. `NotFound`, `NoData` (nothing published yet) and `Blocked` are different situations and are never collapsed into one.
- NSE changed the bhavcopy file format on 8 July 2024. The library picks the right format for the date you ask for, so callers don't have to know.
- `unsafe` is forbidden across the workspace. Releases ship prebuilt binaries for Linux, macOS and Windows, and `sauda` has wheels on PyPI for Windows, Linux and Apple Silicon.
- What it doesn't do is written down too. The pre-open session endpoint isn't implemented, and a few live figures never populate through NSE at all. The docs say so instead of working around it.

### [Audio Notes Platform](https://github.com/Am1n1602/Audio-notes-app)

[![Live demo](https://img.shields.io/badge/Live%20demo-open-2563eb?style=flat-square)](https://audio-notes-app-five.vercel.app)

Upload a recording, watch it move through transcription and summarisation, and reopen the result later. It started as an internship take-home, and the brief asked for visible progress and failure states. Transcripts come from Gnani's batch speech-to-text API and summaries from Groq.

- The audio goes from the browser straight to S3 on a signed URL, and the API only handles metadata, so no web request ever waits on a provider.
- Processing is a chain of short steps. Each one re-reads the job from PostgreSQL and schedules the next, so a step that crashes, runs twice or arrives late does no harm.
- No free hosting tier would run an always-on worker, so in production the queue swaps from Celery and Redis to Cloud Tasks calling a private Cloud Run service. The step logic and the tests are shared between the two.
- More than 300 tests, plus 23 injected failure scenarios that run end to end through a fault-injecting proxy. Each one ends in a plain error message and, where it makes sense, a retry.
- The demo is public with no sign-in, so uploads are limited per browser and recordings are deleted from storage after two hours. The transcript and summary stay.

### [OpenTerminal (India-market fork)](https://github.com/Am1n1602/OpenTerminal)

[![Fork of](https://img.shields.io/badge/fork%20of-ErTasselli%2FOpenTerminal-555555?style=flat-square)](https://github.com/ErTasselli/OpenTerminal)

OpenTerminal is Andrea Tasselli's dense, keyboard-driven market dashboard (Next.js and Express) that runs on free public data. The project and its US and European coverage are his and his contributors'. My changes tune it for Indian markets:

- A gRPC provider that pulls NSE data from `jugaad-rpc`, the server from jugaad-rs: live quotes, history, option chains and expiries for NIFTY, BANKNIFTY and FINNIFTY, index snapshots including India VIX, bulk and block deals, and corporate announcements. If the service is down, every call falls back to Yahoo Finance or TradingView.
- NSE as the default market across the screener, heatmap, macro, news and portfolio widgets, with US and European markets still a tab away.
- A TradingView-backed fallback for the SENSEX quote. SENSEX chart history still has no working source, and the README lists that as a known gap instead of hiding it.
- The option chain also led to a new RPC in jugaad-rs, `GetOptionExpiries`.

## Games and graphics

2D games and simulations in C++, mostly written without an engine.

| Project | What it is |
| --- | --- |
| [Just Another Easy Game](https://github.com/Am1n1602/Just-another-easy-game) | A 2D platformer in C++17 and raylib: movement and collision, traps, level design, game state and audio. Playable in the browser on [itch.io](https://peepow.itch.io/just-another-easy-game). The online leaderboard runs on Node.js / Express and MongoDB. |
| [ParticleCollision](https://github.com/Am1n1602/ParticleCollision) | Interactive 2D simulation of charged particles with attraction and repulsion, elastic collisions and a live centre-of-mass readout. |
| [FP-Movement](https://github.com/Am1n1602/FP-Movement) | First-person camera and movement controller in raylib. |
| [Just-Another-World-Godot](https://github.com/Am1n1602/Just-Another-World-Godot) | A 2D project in Godot with a web export, made to compare an engine-based workflow with writing the systems by hand. |

## Tech stack

| Area | Tools |
| --- | --- |
| **Languages** | ![Rust](https://img.shields.io/badge/Rust-000000?style=flat-square&logo=rust&logoColor=white) ![C++](https://img.shields.io/badge/C%2B%2B-00599C?style=flat-square&logo=cplusplus&logoColor=white) ![Python](https://img.shields.io/badge/Python-3776AB?style=flat-square&logo=python&logoColor=white) ![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white) ![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat-square&logo=javascript&logoColor=black) ![C](https://img.shields.io/badge/C-A8B9CC?style=flat-square&logo=c&logoColor=black) ![Kotlin](https://img.shields.io/badge/Kotlin-7F52FF?style=flat-square&logo=kotlin&logoColor=white) ![SQL](https://img.shields.io/badge/SQL-4479A1?style=flat-square) |
| **Backend** | ![Tokio](https://img.shields.io/badge/Tokio-000000?style=flat-square&logo=tokio&logoColor=white) ![tonic / gRPC](https://img.shields.io/badge/tonic%20%2F%20gRPC-244C5A?style=flat-square) ![FastAPI](https://img.shields.io/badge/FastAPI-009688?style=flat-square&logo=fastapi&logoColor=white) ![Node.js](https://img.shields.io/badge/Node.js-5FA04E?style=flat-square&logo=nodedotjs&logoColor=black) ![Express](https://img.shields.io/badge/Express-0A0A0A?style=flat-square&logo=express&logoColor=white) ![Celery](https://img.shields.io/badge/Celery-37814A?style=flat-square&logo=celery&logoColor=white) |
| **Data stores** | ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-4169E1?style=flat-square&logo=postgresql&logoColor=white) ![SQLite](https://img.shields.io/badge/SQLite-003B57?style=flat-square&logo=sqlite&logoColor=white) ![MongoDB](https://img.shields.io/badge/MongoDB-47A248?style=flat-square&logo=mongodb&logoColor=white) ![Redis](https://img.shields.io/badge/Redis-FF4438?style=flat-square&logo=redis&logoColor=white) |
| **Frontend** | ![React](https://img.shields.io/badge/React-61DAFB?style=flat-square&logo=react&logoColor=black) ![Next.js](https://img.shields.io/badge/Next.js-000000?style=flat-square&logo=nextdotjs&logoColor=white) ![Vite](https://img.shields.io/badge/Vite-9135FF?style=flat-square&logo=vite&logoColor=white) ![Tailwind CSS](https://img.shields.io/badge/Tailwind%20CSS-06B6D4?style=flat-square&logo=tailwindcss&logoColor=black) |
| **Data, ML and LLMs** | ![pandas](https://img.shields.io/badge/pandas-150458?style=flat-square&logo=pandas&logoColor=white) ![Polars](https://img.shields.io/badge/Polars-0075FF?style=flat-square&logo=polars&logoColor=white) ![NumPy](https://img.shields.io/badge/NumPy-013243?style=flat-square&logo=numpy&logoColor=white) ![scikit-learn](https://img.shields.io/badge/scikit--learn-F7931E?style=flat-square&logo=scikitlearn&logoColor=black) ![LightGBM](https://img.shields.io/badge/LightGBM-2E8B57?style=flat-square) ![PyTorch](https://img.shields.io/badge/PyTorch-EE4C2C?style=flat-square&logo=pytorch&logoColor=white) ![FAISS](https://img.shields.io/badge/FAISS-0467DF?style=flat-square) ![sentence-transformers](https://img.shields.io/badge/sentence--transformers-5B5B5B?style=flat-square) ![Anthropic API](https://img.shields.io/badge/Anthropic%20API-191919?style=flat-square&logo=anthropic&logoColor=white) ![Groq](https://img.shields.io/badge/Groq-F55036?style=flat-square) ![Ollama](https://img.shields.io/badge/Ollama-000000?style=flat-square&logo=ollama&logoColor=white) |
| **Cloud and tooling** | ![Docker](https://img.shields.io/badge/Docker-2496ED?style=flat-square&logo=docker&logoColor=white) ![Google Cloud](https://img.shields.io/badge/Google%20Cloud-4285F4?style=flat-square&logo=googlecloud&logoColor=white) ![Firebase](https://img.shields.io/badge/Firebase-DD2C00?style=flat-square&logo=firebase&logoColor=white) ![Vercel](https://img.shields.io/badge/Vercel-000000?style=flat-square&logo=vercel&logoColor=white) ![AWS S3](https://img.shields.io/badge/AWS%20S3-569A31?style=flat-square) ![GitHub Actions](https://img.shields.io/badge/GitHub%20Actions-2088FF?style=flat-square&logo=githubactions&logoColor=white) ![Git](https://img.shields.io/badge/Git-F03C2E?style=flat-square&logo=git&logoColor=white) ![CMake](https://img.shields.io/badge/CMake-064F8C?style=flat-square&logo=cmake&logoColor=white) ![Cargo](https://img.shields.io/badge/Cargo-B7410E?style=flat-square) ![Linux](https://img.shields.io/badge/Linux-FCC624?style=flat-square&logo=linux&logoColor=black) |
| **Graphics and games** | ![raylib](https://img.shields.io/badge/raylib-000000?style=flat-square&logo=raylib&logoColor=white) ![Godot](https://img.shields.io/badge/Godot-478CBF?style=flat-square&logo=godotengine&logoColor=white) ![OpenGL](https://img.shields.io/badge/OpenGL-5586A4?style=flat-square&logo=opengl&logoColor=white) ![SDL2](https://img.shields.io/badge/SDL2-4A4A4A?style=flat-square) |

## GitHub activity

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://github-readme-stats.vercel.app/api?username=Am1n1602&show_icons=true&hide_border=true&theme=github_dark">
    <img alt="GitHub stats" src="https://github-readme-stats.vercel.app/api?username=Am1n1602&show_icons=true&hide_border=true&theme=default" height="165">
  </picture>
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://github-readme-stats.vercel.app/api/top-langs/?username=Am1n1602&layout=compact&langs_count=8&hide_border=true&theme=github_dark">
    <img alt="Most used languages" src="https://github-readme-stats.vercel.app/api/top-langs/?username=Am1n1602&layout=compact&langs_count=8&hide_border=true&theme=default" height="165">
  </picture>
</p>

<p align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="https://streak-stats.demolab.com/?user=Am1n1602&hide_border=true&theme=dark">
    <img alt="Contribution streak" src="https://streak-stats.demolab.com/?user=Am1n1602&hide_border=true&theme=default" height="165">
  </picture>
</p>

## Currently

- Studying deep learning, with an eye on applying it to time series and documents.
- Next on Fin_QA: a full re-run of the 850-question benchmark against the final corpus, and re-validating the cross-encoder reranker at full scale.
- Further out: a 2D RPG.

## Contact

The easiest way to reach me is email: [amangautam1602@gmail.com](mailto:amangautam1602@gmail.com)

[LinkedIn](https://www.linkedin.com/in/am1n-gautam/) · [X](https://x.com/Am1n1602) · [itch.io](https://peepow.itch.io/just-another-easy-game)
