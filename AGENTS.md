# Weather Dashboard (Next.js Clean Architecture) - Project Instructions

This project is a training exercise for implementing a Frontend application using **Clean Architecture** principles in Next.js.

## Project Overview

The goal is to build a Weather Dashboard that fetches data from the Open-Meteo API. The focus is not just on functionality, but on strict architectural separation and clean code practices.

### Core Tech Stack

- **Framework:** Next.js 14+ (App Router)
- **Language:** TypeScript
- **Styling:** Tailwind CSS + DaisyUI
- **Animation:** Framer Motion
- **State Management:** React Hooks (Custom Hooks as Presenters)
- **API:** Open-Meteo API (Native `fetch`)
- **Data Validation:** Zod (for DTO validation)

## Architecture Layers

The project follows a strict 5-layer Clean Architecture:

1.  **App Layer (`src/app`)**: Responsible for routing and layouts. **Rules:** No fetching, no business logic, no state management.
2.  **Domain Layer (`features/weather/domain`)**: Contains business entities and pure logic functions. **Rules:** Pure functions only, no side effects, no dependencies on external layers.
3.  **Data Layer (`features/weather/data`)**: Handles API communication and data mapping. **Rules:** Use `fetch`, validate DTOs with Zod, and map DTOs to Domain Entities.
4.  **Presenter Layer (`features/weather/presenter`)**: Manages UI state and orchestrates data flow. **Rules:** Use Custom Hooks to bridge the Data/Domain layers with the View.
5.  **View Layer (`features/weather/view`)**: Presentational UI components. **Rules:** No fetching, no complex logic, receive data via props.

## Building and Running

_Note: This project is currently in the specification phase._

### Recommended Setup

```bash
# Initialize Next.js project
npx create-next-app@latest .

# Install dependencies
npm install daisyui motion zod recharts
npm install -D vitest
```

### Key Commands

- `npm run dev`: Start development server.
- `npm run build`: Build for production.
- `npm test`: Run unit tests (Vitest).

## Development Conventions

### Strict Architectural Rules

- **No Fetching in View**: Components must never call `fetch` or Repositories directly. They must use Presenter hooks.
- **DTO Isolation**: View components must never know the shape of the API response (DTO). They only work with Domain Entities.
- **Business Logic belongs to Domain**: Rules like "temperature > 38 is danger" must reside in `weather.logic.ts`, not in the UI components.
- **Dumb Components**: UI components should be presentational and focus on DaisyUI/Motion implementations.

### Error Handling & Resilience (Bonus 7+)

- Use **Custom Error Classes** (`NetworkError`, `ValidationError`) for precise error handling.
- **Zod Validation** is mandatory for all API responses in the Data Layer.
- Implement **Next.js Error Boundaries** (`error.tsx`) for page-level failures.
- Provide user-friendly Thai error messages for common failure cases.

### Component Design

- Use **DaisyUI** components (cards, badges, alerts, stats).
- Implement **Framer Motion** for all state transitions (city changes, loading states, list animations).
