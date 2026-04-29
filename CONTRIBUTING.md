# Contributing to SYNAPSE

SYNAPSE is an active Android project. Contributions, bug reports, and feature ideas are welcome.

---

## Requirements

- Android Studio Hedgehog (2023.1.1) or newer
- Android device running API 26+ (physical device required for BLE features)
- A [Google Gemini API key](https://ai.google.dev/) (free tier is sufficient for development)
- A [Supabase](https://supabase.com/) project (free tier)
- Mi Band 5 paired via system Bluetooth (optional — only required for wearable features)

---

## Getting Started

```bash
git clone https://github.com/MarcosCF1/synapse.git
# Open in Android Studio
cp local.properties.example local.properties
```

Edit `local.properties` and set:
```
GEMINI_API_KEY=your_key_here
SUPABASE_URL=https://your-project.supabase.co
SUPABASE_ANON_KEY=your_anon_key_here
```

Connect your Android device → Run the `app` configuration.

> **Note:** BLE features require a physical device. The emulator cannot scan for Bluetooth peripherals.

---

## Branching & Commits

- Branch from `main`: `git checkout -b feat/your-feature-name`
- Use [Conventional Commits](https://www.conventionalcommits.org/):
  - `feat:` new feature
  - `fix:` bug fix
  - `docs:` documentation
  - `chore:` maintenance
  - `refactor:` non-breaking code changes

---

## Code Conventions

- **Kotlin** — follow [Android Kotlin style guide](https://developer.android.com/kotlin/style-guide)
- **Coroutines + Flow** — all async work uses coroutines; no callbacks or `AsyncTask`
- **No hardcoded secrets** — all API keys must come from `local.properties` (gitignored)
- **No real user data in tests** — use synthetic mock data only

---

## Reporting Issues

Open a GitHub Issue with:
- Android version and device model
- Steps to reproduce
- Logcat output (sanitized — remove any personal data)
- Expected vs. actual behaviour

---

## License

By contributing, you agree that your contributions will be licensed under the [MIT License](LICENSE).
