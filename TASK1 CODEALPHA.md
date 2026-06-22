```json
{
  "name": "flashcard-quiz-app",
  "private": true,
  "version": "1.0.0",
  "type": "module",
  "scripts": {
    "dev": "vite",
    "build": "tsc && vite build",
    "preview": "vite preview"
  },
  "dependencies": {
    "@tanstack/react-query": "^5.80.0",
    "framer-motion": "^12.18.1",
    "lucide-react": "^0.525.0",
    "react": "^19.1.0",
    "react-dom": "^19.1.0",
    "wouter": "^3.3.5"
  },
  "devDependencies": {
    "@types/react": "^19.1.8",
    "@types/react-dom": "^19.1.6",
    "@vitejs/plugin-react": "^4.5.2",
    "autoprefixer": "^10.4.21",
    "postcss": "^8.5.6",
    "tailwindcss": "^3.4.17",
    "typescript": "^5.8.3",
    "vite": "^7.0.0"
  }
}
```
```ts
import { defineConfig } from "vite";
import react from "@vitejs/plugin-react";
import path from "path";

export default defineConfig({
  plugins: [react()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
});
```
```json
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "module": "ESNext",
    "skipLibCheck": true,

    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },

    "moduleResolution": "bundler",
    "allowImportingTsExtensions": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "jsx": "react-jsx",

    "strict": true
  },
  "include": ["src"],
  "references": []
}
```
```tsx
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";
import "./index.css";

ReactDOM.createRoot(
  document.getElementById("root") as HTMLElement
).render(
  <React.StrictMode>
    <App />
  </React.StrictMode>
);
```
```tsx
import { QueryClient, QueryClientProvider } from "@tanstack/react-query";
import { Route, Switch } from "wouter";

import Home from "./pages/Home";
import Quiz from "./pages/Quiz";
import Manage from "./pages/Manage";
import CardForm from "./pages/CardForm";
import NotFound from "./pages/NotFound";

import { Header } from "./components/Header";
import { ThemeProvider } from "./components/ThemeProvider";

const queryClient = new QueryClient();

function Router() {
  return (
    <Switch>
      <Route path="/">
        <Home />
      </Route>

      <Route path="/quiz">
        <Quiz />
      </Route>

      <Route path="/manage">
        <Manage />
      </Route>

      <Route path="/manage/new">
        <CardForm />
      </Route>

      <Route path="/manage/:id">
        <CardForm />
      </Route>

      <Route>
        <NotFound />
      </Route>
    </Switch>
  );
}

export default function App() {
  return (
    <QueryClientProvider client={queryClient}>
      <ThemeProvider>
        <div className="min-h-screen flex flex-col bg-background text-foreground">
          <Header />

          <main className="flex-1">
            <Router />
          </main>
        </div>
      </ThemeProvider>
    </QueryClientProvider>
  );
}
```
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

:root {
  --background: #f8f6f2;
  --foreground: #1d2433;
  --primary: #2f3f78;
  --secondary: #c78620;
}

body {
  margin: 0;
  font-family: Inter, Arial, sans-serif;
  background: var(--background);
  color: var(--foreground);
}

.bg-background {
  background: var(--background);
}

.text-foreground {
  color: var(--foreground);
}
```
src/
├── App.tsx
├── main.tsx
├── index.css
├── pages/
├── components/
├── lib/
```tsx
import {
  createContext,
  useContext,
  useEffect,
  useState,
  ReactNode,
} from "react";

type Theme = "light" | "dark";

interface ThemeContextType {
  theme: Theme;
  toggleTheme: () => void;
}

const ThemeContext = createContext<ThemeContextType | null>(null);

export function ThemeProvider({
  children,
}: {
  children: ReactNode;
}) {
  const [theme, setTheme] = useState<Theme>(() => {
    const stored = localStorage.getItem("flashlearn_theme");

    if (stored === "light" || stored === "dark") {
      return stored;
    }

    return "light";
  });

  useEffect(() => {
    document.documentElement.classList.remove(
      "light",
      "dark"
    );

    document.documentElement.classList.add(theme);

    localStorage.setItem(
      "flashlearn_theme",
      theme
    );
  }, [theme]);

  const toggleTheme = () =>
    setTheme((prev) =>
      prev === "light" ? "dark" : "light"
    );

  return (
    <ThemeContext.Provider
      value={{
        theme,
        toggleTheme,
      }}
    >
      {children}
    </ThemeContext.Provider>
  );
}

export function useTheme() {
  const context = useContext(ThemeContext);

  if (!context) {
    throw new Error(
      "useTheme must be used inside ThemeProvider"
    );
  }

  return context;
}
```
```tsx
import { ButtonHTMLAttributes } from "react";

interface Props
  extends ButtonHTMLAttributes<HTMLButtonElement> {
  variant?: "primary" | "secondary";
}

export default function Button({
  children,
  variant = "primary",
  className = "",
  ...props
}: Props) {
  const styles =
    variant === "primary"
      ? "bg-blue-600 text-white"
      : "bg-gray-200 text-black";

  return (
    <button
      {...props}
      className={`px-4 py-2 rounded-lg font-medium transition hover:opacity-90 ${styles} ${className}`}
    >
      {children}
    </button>
  );
}
```
```tsx
import { useLocation } from "wouter";
import { useTheme } from "./ThemeProvider";

export function Header() {
  const [, setLocation] = useLocation();
  const { theme, toggleTheme } = useTheme();

  return (
    <header className="border-b bg-white shadow-sm">
      <div className="max-w-5xl mx-auto h-16 px-4 flex items-center justify-between">
        <h1
          onClick={() => setLocation("/")}
          className="font-bold text-xl cursor-pointer"
        >
          Flashcard Quiz App
        </h1>

        <div className="flex gap-2">
          <button
            onClick={() => setLocation("/")}
            className="px-3 py-2 rounded"
          >
            Home
          </button>

          <button
            onClick={toggleTheme}
            className="px-3 py-2 rounded border"
          >
            {theme === "light"
              ? "🌙 Dark"
              : "☀️ Light"}
          </button>
        </div>
      </div>
    </header>
  );
}
```
```tsx
import { useEffect, useState } from "react";

export interface Flashcard {
  id: string;
  front: string;
  back: string;
  createdAt: number;
}

const STORAGE_KEY = "flashlearn_cards";

const SEED_CARDS: Flashcard[] = [
  {
    id: "1",
    front: "What is React?",
    back: "A JavaScript UI library",
    createdAt: Date.now(),
  },
  {
    id: "2",
    front: "What is TypeScript?",
    back: "A typed superset of JavaScript",
    createdAt: Date.now(),
  },
];

export function getCards(): Flashcard[] {
  const stored =
    localStorage.getItem(STORAGE_KEY);

  if (!stored) {
    localStorage.setItem(
      STORAGE_KEY,
      JSON.stringify(SEED_CARDS)
    );

    return SEED_CARDS;
  }

  try {
    return JSON.parse(stored);
  } catch {
    return [];
  }
}

function saveCards(cards: Flashcard[]) {
  localStorage.setItem(
    STORAGE_KEY,
    JSON.stringify(cards)
  );

  window.dispatchEvent(
    new Event("flashcards-updated")
  );
}

export function addCard(
  front: string,
  back: string
) {
  const cards = getCards();

  const card: Flashcard = {
    id: Date.now().toString(),
    front,
    back,
    createdAt: Date.now(),
  };

  saveCards([card, ...cards]);
}

export function updateCard(
  id: string,
  front: string,
  back: string
) {
  const cards = getCards();

  const updated = cards.map((c) =>
    c.id === id
      ? {
          ...c,
          front,
          back,
        }
      : c
  );

  saveCards(updated);
}

export function deleteCard(id: string) {
  const cards = getCards();

  saveCards(
    cards.filter((c) => c.id !== id)
  );
}

export function useFlashcards() {
  const [cards, setCards] =
    useState<Flashcard[]>([]);

  useEffect(() => {
    const load = () => {
      setCards(getCards());
    };

    load();

    window.addEventListener(
      "flashcards-updated",
      load
    );

    return () =>
      window.removeEventListener(
        "flashcards-updated",
        load
      );
  }, []);

  return {
    cards,
  };
}
```
```tsx
import { Flashcard } from "../lib/flashcards";

interface Props {
  card: Flashcard;
  isFlipped: boolean;
  onFlip: () => void;
}

export function FlashCard({
  card,
  isFlipped,
  onFlip,
}: Props) {
  return (
    <div
      onClick={onFlip}
      className="cursor-pointer bg-white rounded-xl shadow-md p-8 min-h-[250px] flex items-center justify-center"
    >
      <h2 className="text-2xl text-center">
        {isFlipped
          ? card.back
          : card.front}
      </h2>
    </div>
  );
}
```
src/
├── App.tsx
├── main.tsx
├── index.css
│
├── lib/
│   └── flashcards.ts
│
├── components/
│   ├── Header.tsx
│   ├── ThemeProvider.tsx
│   ├── FlashCard.tsx
│   └── Button.tsx
│
├── pages/