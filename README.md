# calculator_frontend
## Introduction

Frontend of the front-end and back-end separated calculator system, built with uni-app and Vue 3.
Handles user input, interface rendering and history display. **No local calculation — all results come exclusively from the backend API**.

## Tech Stack

- uni-app
- Vue 3 Composition API
- CSS3 (Responsive layout)
- uni.request (HTTP client)

## Prerequisites

- HBuilderX 3.6+
- Modern browser (Chrome / Edge / Firefox)
- Backend service must be running first

## Quick Start

1. Clone the repository

```
git clone https://github.com/Lucas-JIN-cloud/calculator_frontend.git
```
2. Import the project into HBuilderX
3. Configure dev proxy
Add this to `manifest.json` → `h5` → `devServer`:

```
"proxy": {
  "/api": {
    "target": "http://localhost:8080",
    "changeOrigin": true
  }
}
```
4. Run
Top menu → **Run** → **Run to browser** → Select Chrome

## Features

- Basic arithmetic, parentheses and decimals
- Auto-saved calculation history
- Single record deletion
- Error prompts for invalid input and network issues
- Responsive layout for desktop and mobile
