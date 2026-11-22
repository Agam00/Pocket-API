# 📱 Pocket API

> A Postman-style API client that lives in your pocket.

Build, send, save, and inspect HTTP requests from your phone. Organize them into collections, switch between environments, and generate code snippets.

🚧 **Status:** early development, being built phase by phase as a learning project.

## Tech stack

- [Expo](https://expo.dev) (SDK 58) + React Native
- TypeScript (strict mode)
- Expo Router (file-based navigation)
- ESLint + Prettier

Coming in later phases: Zustand, React Hook Form + Zod, expo-sqlite + Drizzle, Reanimated, Jest, Maestro, EAS.

## Planned features

- Send requests with any HTTP method, with query params, headers, body, and auth
- View the response: status, time, size, headers, and pretty-printed JSON
- Request history and collections, saved on the device
- Environments with variables (`{{baseUrl}}`, `{{token}}`)
- cURL import/export and code snippet generation
- Dark and light themes, biometric lock, WebSocket and GraphQL modes

## Getting started

```bash
npm install
npx expo start
```

Then scan the QR code with [Expo Go](https://expo.dev/go) on your phone.

## Scripts

| Command             | What it does                  |
| ------------------- | ----------------------------- |
| `npm start`         | Start the Metro dev server    |
| `npm run lint`      | Lint with ESLint and Prettier |
| `npm run typecheck` | Type-check with TypeScript    |
| `npm run format`    | Format all files              |

## Project structure

```
src/
└── app/          # Screens (every file is a route)
docs/
└── notes.md      # My learning notes, phase by phase
```

## Roadmap

| Phase | Topic                   | Status |
| ----- | ----------------------- | ------ |
| 0     | Setup                   | ✅     |
| 1     | UI & navigation         | ⬜     |
| 2     | Request engine          | ⬜     |
| 3     | State & forms           | ⬜     |
| 4     | Persistence             | ⬜     |
| 5     | Environments, auth      | ⬜     |
| 6     | Native features, polish | ⬜     |
| 7     | Device integration      | ⬜     |
| 8     | Advanced protocols      | ⬜     |
| 9     | Testing                 | ⬜     |
| 10    | Ship it                 | ⬜     |
