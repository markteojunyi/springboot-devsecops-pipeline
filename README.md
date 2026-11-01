# 🔐 Spring Boot DevSecOps Pipeline

A CI/CD pipeline with security checks built in, wrapped around a small Spring Boot web app. Every push to `main` is automatically built, tested, scanned for vulnerable libraries, attacked by a security scanner, and the result is pushed to my phone via Telegram.

> **Context:** built during an NUS-ISS software engineering course (Dec 2025). The Spring Boot application was provided by the instructor ([@darryl1975](https://github.com/darryl1975)). **My contribution is the pipeline and tests** described below.

## Why this matters

Security is usually checked at the end, by a separate team, right before release. That's slow and expensive. **DevSecOps** moves those checks into the everyday pipeline ("shift left"), so problems surface minutes after the code is written instead of weeks later.

## What I built

```
push to main
     │
     ▼
┌─────────────────┐
│ build-and-test  │  Maven build + JUnit controller tests ──► Telegram alert
└────────┬────────┘
         │ only if tests pass
   ┌─────┴──────────────┐
   ▼                    ▼
┌──────────────┐  ┌───────────────────┐
│ Snyk scan    │  │ OWASP ZAP scan    │  attacks the live app on Railway
│ (libraries)  │  │ (running app)     │
└──────────────┘  └─────────┬─────────┘
                            ▼
                  Telegram alert with result
```

| Stage | What it checks | In plain English |
|---|---|---|
| **Unit tests** (JUnit, MockMvc) | `PersonController` and `DataController` behave correctly | Does the code do what it should? |
| **Snyk** (SCA) | Third-party libraries against known vulnerability databases; fails on *high* severity | Are we using any parts with known security holes? |
| **OWASP ZAP** (DAST) | Full scan of the deployed app for issues like XSS and missing security headers | If a hacker poked at the live site, what would they find? |
| **Telegram alerts** | Posts each stage's result | The team knows immediately, without checking GitHub |

*SCA = Software Composition Analysis. DAST = Dynamic Application Security Testing.*

## Tech stack

GitHub Actions · Maven · Java 17 / Spring Boot · JUnit + MockMvc · Snyk · OWASP ZAP · Docker (multi-stage build) · Railway · Telegram Bot API

## Run the app locally

```bash
./mvnw spring-boot:run      # app on http://localhost:8080
./mvnw test                 # run the unit tests
```

Pipeline secrets required: `SNYK_TOKEN`, `TELEGRAM_TO`, `TELEGRAM_TOKEN`.

## What I learned

- Ordering matters: scans only run after tests pass, so you don't waste scan time on broken code.
- The two scanners catch different things. Snyk finds risky *ingredients*; ZAP finds risky *behaviour*. You need both.
