# Postman API Testing Showcase

[![API Tests](https://github.com/sudeshna-bhattacharya/postman-api-testing/actions/workflows/api-tests.yml/badge.svg)](https://github.com/sudeshna-bhattacharya/postman-api-testing/actions/workflows/api-tests.yml)
[![Report](https://img.shields.io/badge/Newman-Live%20HTML%20Report-FF6C37?logo=postman&logoColor=white)](https://sudeshna-bhattacharya.github.io/postman-api-testing/)
[![Run in Postman](https://img.shields.io/badge/Postman-Public%20Workspace-FF6C37?logo=postman&logoColor=white)](https://www.postman.com/restless-eclipse-769270/api-testing-showcase/)
![Postman](https://img.shields.io/badge/Postman-FF6C37?logo=postman&logoColor=white)
![Newman](https://img.shields.io/badge/Newman-CLI-FF6C37?logo=postman&logoColor=white)

An **advanced Postman collection** run with **Newman** in CI. The suite goes beyond status-code checks: **request chaining**, **JWT and cookie auth**, **JSON-schema contracts**, **workflow loops**, **business-logic assertions** and **data-driven runs**.

> APIs under test (public demo APIs): [Restful-Booker](https://restful-booker.herokuapp.com/apidoc/index.html) and [DummyJSON](https://dummyjson.com/docs).

**33 requests · 7 folders · 140+ assertions per run · 5 data-driven iterations**

---

## ✨ Techniques demonstrated

| Technique | Where |
|---|---|
| **Request chaining**: ids, tokens and expected state passed between requests | `03`, `04`, `06` |
| **Script hierarchy**: collection, folder and request-level pre-request and test scripts | whole collection |
| **Global checks**: response-time budget and "no errors" checks run after *every* request; requests prefixed `NEG` are exempt | collection `Tests` |
| **Two auth styles**: cookie token (Restful-Booker) and JWT Bearer with **refresh-token rotation** (DummyJSON) | `02` |
| **JWT claim validation**: token decoded with `crypto-js`, subject and expiry checked | `02 / login` |
| **Self-healing auth**: `pm.sendRequest` fetches a token only if missing, so any folder can run on its own | folders `03`, `06`, `07` |
| **JSON-schema contract tests**: schemas stored as collection variables, checked with `pm.response.to.have.jsonSchema` | `02`, `03`, `06` |
| **Stateful E2E**: expected state updated after PUT and PATCH, so the API is compared with what it *should* hold | `03` |
| **Workflow control**: `pm.execution.setNextRequest` loops through every page, then checks the total count and that no item repeats | `05` |
| **Business-logic validation**: cart totals recalculated independently (price × qty, sum of lines, discounts) | `06` |
| **Negative and authorization tests**: missing or invalid token, 404s, malformed JSON, missing fields | `04` |
| **Known-defect tracking**: `pm.test.skip` records API defects in the report without failing CI | `02`, `04` |
| **Data-driven testing**: CSV with edge cases (apostrophes, accents, zero and very large prices); each row is created, verified and cleaned up | `07` |
| **Dynamic test data**: `{{$randomFirstName}}`, `{{$guid}}`, lodash, date maths | `03` |
| **Request tracing**: `X-Request-Id` correlation header added to every request | collection pre-request |
| **CI/CD**: Newman in GitHub Actions on push, PR and a weekday schedule, with the HTML report published to GitHub Pages | `.github/workflows` |

---

## 🗂️ Collection structure

```
API Testing Showcase
├── 01 Smoke & Health                     fail fast if an API is down
├── 02 Authentication                     cookie token · JWT login → /me → refresh → reuse · negatives
├── 03 Booking Lifecycle (E2E chain)      Create → Get → Search → PUT → PATCH → Delete → Verify 404
├── 04 Negative & Authorization           403 / 404 / 400 / known defects
├── 05 Pagination & Workflow Control      setNextRequest loop · sorting · search
├── 06 Business Logic (cross-request)     prices → cart → independent total calculation
└── 07 Data-Driven Bookings               one iteration per CSV row, with clean-up
```

### Example: stateful chain (03)

```js
// PATCH pre-request: expected = previous state + patch
const patch = { depositpaid: false, additionalneeds: 'Airport transfer' };
pm.collectionVariables.set('patchBody', JSON.stringify(patch));
const merged = Object.assign(JSON.parse(pm.collectionVariables.get('expectedBooking')), patch);
pm.collectionVariables.set('expectedBooking', JSON.stringify(merged));

// PATCH test: patched fields changed, all other fields untouched
pm.expect(pm.response.json()).to.eql(JSON.parse(pm.collectionVariables.get('expectedBooking')));
```

### Example: pagination loop (05)

```js
if (nextSkip < body.total) {
    pm.collectionVariables.set('skip', nextSkip);
    pm.execution.setNextRequest(pm.info.requestName); // run me again
} else {
    pm.test(`collected all ${body.total} products`, () => pm.expect(ids.length).to.eql(body.total));
    pm.test('no duplicates across pages', () => pm.expect(new Set(ids).size).to.eql(ids.length));
}
```

---

## 🚀 Run it

### In Postman
[![Run in Postman](https://img.shields.io/badge/Postman-Public%20Workspace-FF6C37?logo=postman&logoColor=white)](https://www.postman.com/restless-eclipse-769270/api-testing-showcase/)
1. **Import:** `collections/api-testing-showcase.postman_collection.json` and `environments/demo.postman_environment.json`
2. Select the **Public Demo APIs** environment.
3. Right-click the collection → **Run collection**. Run folders in order.
4. For data-driven runs, open folder **07** in the Runner and select `data/bookings.csv`.

### With Newman (CLI)

```bash
npm ci
npm test               # full suite → reports/report.html + reports/junit.xml
npm run test:data      # data-driven folder with data/bookings.csv
npm run test:smoke     # health checks only
npm run test:auth      # auth folder only
npm run test:e2e       # booking lifecycle only
npm run test:negative  # negative & authorization only
```

Open `reports/report.html` for the detailed **htmlextra** report.

---

## 🔁 CI/CD

`.github/workflows/api-tests.yml` runs on every push and PR, and on a weekday schedule:

1. Full regression run → HTML + JUnit reports
2. Data-driven run
3. Reports uploaded as build artifacts
4. On `main`: HTML report published to **GitHub Pages**

**One-time setup:** in the repo, go to **Settings → Pages → Source: GitHub Actions**.

---

## 📁 Repository layout

```
.
├── collections/api-testing-showcase.postman_collection.json
├── environments/demo.postman_environment.json
├── data/bookings.csv
├── .github/workflows/api-tests.yml
└── package.json                 # Newman + htmlextra scripts
```

> Credentials in the environment file are the **public demo credentials** published by each API. In a real project, store secrets as Postman *secret* variables locally and as **GitHub Actions secrets** in CI (`--env-var "password=${{ secrets.API_PASSWORD }}"`).

---

## 👩‍💻 Author

**Sudeshna**, QA Technical Lead · [LinkedIn](https://www.linkedin.com/in/sudeshna-bhattacharya/) · [GitHub](https://github.com/sudeshna-bhattacharya)
