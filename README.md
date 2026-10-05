# calculator_frontend
Frontend of the Web calculator built with **uni-app (Vue 3)** and compiled to a static **H5** site. It communicates with the Spring Boot backend through HTTP/JSON APIs.

> Core calculation is 
>
> **never**
>
>  done on the frontend; it only sends the raw expression and displays the result returned by the backend.



***

## 1. Tech Stack



| Category    | Choice                            |
| ----------- | --------------------------------- |
| Framework   | uni-app (Vue 3, `<script setup>`) |
| Language    | JavaScript                        |
| HTTP client | `uni.request`                     |
| Build       | HBuilderX (Vite)                  |
| Target      | Static H5 (web)                   |
| Deployment  | GitHub Pages                      |



***

## 2. Features



* Basic arithmetic: `+ - * /`

* Compound expressions with operator precedence, parentheses, decimals, unary signs

* Scientific functions: `sin / cos / tan / sqrt / log`, constants `pi / e`

* Live calculation history (loaded from the backend database)

* Delete a single history record

* Favorite / unfavorite a history record

* Number-base conversion (base 2–36)

* Unit conversion (length, weight, area, volume, temperature)

* Keyboard shortcuts (Enter = calculate, Backspace = delete, Esc = clear, S/C/T/L/R/P/E)

* 4 switchable themes (light / dark / green / blue)

* Responsive layout for desktop and mobile



***

## 3. Project Structure



```
caculate/
├── pages/
│   └── index/
│       └── index.vue          # Calculator UI + logic
├── static/                    # Static assets (logo.png)
├── unpackage/dist/build/web/   # H5 build output
├── docs/                       # GitHub Pages deployment directory
├── manifest.json              # uni-app config (h5 publicPath "./", hash router)
├── pages.json
├── codestyle.md               # Code standard (Airbnb JS)
└── README.md
```



***

## 4. Prerequisites



* HBuilderX 3.6+ (5.24 used for this project)

* Node.js 16+ (for CLI builds)

* A running backend (see `calculator_backend`)



***

## 5. Run Locally



1. Clone the repository



```
git clone https://github.com/Lucas-JIN-cloud/calculator_frontend.git
```



1. Import the project folder into **HBuilderX**.

2. (Optional) Configure a dev proxy in `manifest.json` → `h5` → `devServer`:



```
"proxy": {
  "/api": { "target": "http://localhost:8080", "changeOrigin": true }
}
```



1. Select **Run → Run to Browser → Chrome** to start the dev server.



***

## 6. Connecting to the Backend

All API calls use a single constant at the top of `pages/index/index.vue`:



```
const API_BASE = 'https://calcula-backend-nshsffpxka.cn-hangzhou.fcapp.run/api'
```

To point to a local backend, change it to:



```
const API_BASE = 'http://localhost:8080/api'
```

The frontend calls the following endpoints (all return JSON):



| Method | Path                         | Purpose                                |
| ------ | ---------------------------- | -------------------------------------- |
| POST   | `/api/calculate`             | Send an expression, receive the result |
| GET    | `/api/history`               | Load calculation history               |
| DELETE | `/api/history/{id}`          | Delete one record                      |
| POST   | `/api/history/{id}/favorite` | Toggle favorite                        |
| POST   | `/api/convert/base`          | Number-base conversion                 |

The backend must allow CORS; the site is expected to be served from a different origin than the API.



***

## 7. Deploy to GitHub Pages



1. Build the H5 version in HBuilderX (**Publish → Web**).

2. Copy the build output (`unpackage/dist/build/web/`) into the repo's `docs/` folder.

3. In GitHub repo → **Settings → Pages → Build and deployment**, set the source to the `main` branch and the `/docs` folder.

4. The site becomes available at:



```
https://lucas-jin-cloud.github.io/calculator_frontend/
```



***

## 8. Notes



* History persistence is handled entirely by the backend database.

* Theme switching, unit conversion, and base conversion are purely client-side where noted (base conversion calls the backend; unit conversion is client-side).
- Single record deletion
- Error prompts for invalid input and network issues
- Responsive layout for desktop and mobile
