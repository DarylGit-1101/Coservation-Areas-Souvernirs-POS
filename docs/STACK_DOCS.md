# Up-to-Date Stack Versions & Official Documentation

*Last verified: November 2026*

Always consult the official documentation below before configuring, upgrading, or writing framework-specific code.

---

## Backend & Runtime

| Technology | Latest Verified Version | Official Documentation | Key Notes |
| :--- | :--- | :--- | :--- |
| **PHP** | `8.4` (Docker `php:8.4-fpm-alpine`; active branches `8.3`–`8.6`) | https://www.php.net/docs.php | Laravel 13 requires **PHP 8.3+**. |
| **Laravel** | `13.44.x` (`^13.17`) | https://laravel.com/docs | Requires PHP 8.3+; bug fixes until Q3 2027, security fixes until Mar 2028. |
| **Laravel Sanctum** | `4.5.x` (`^4.0`) | https://laravel.com/docs/sanctum | Installed via `php artisan install:api`; powers POS API token auth & Reverb private channel auth. |
| **Laravel Reverb** | `1.14.x` (`^1.0`) | https://laravel.com/docs/broadcasting#laravels-reverb | Installed via `php artisan install:broadcasting --reverb --without-node`, runs via `pos_reverb` (`--profile reverb`). |
| **Filament** | `5.12.x` (`^5.0`) | https://filamentphp.com/docs/5.x | Native Livewire v4 (`4.4.x`) support; back-office admin panel at `/admin`. |
| **PostgreSQL** | `17` (Docker `postgres:17-alpine`) | https://www.postgresql.org/docs/current/ | Use `BIGINT` for all money columns (minor units / cents). |

---

## Frontend & Offline / Mobile

| Technology | Latest Verified Version | Official Documentation | Key Notes |
| :--- | :--- | :--- | :--- |
| **Node.js** | `26` (Active LTS) | https://nodejs.org/docs/latest/api/ | `vite-plugin-pwa` v2 requires Node 20.19+ / 22+. |
| **Vue** | `3.5.x` | https://vuejs.org/guide/introduction.html | Plain Vue 3 SPA (no Nuxt SSR). |
| **Vite** | `8.3.x` | https://vite.dev/guide/ | Core frontend build tool. |
| **Pinia** | `4.1.x` | https://pinia.vuejs.org/ | Pinia v4 requires `@vue/devtools-api` as an explicit peer dependency. |
| **vite-plugin-pwa** | `2.0.x` | https://vite-pwa-org.netlify.app/ | Uses Workbox v7 for offline service worker caching. |
| **Dexie.js** | `4.4.x` | https://dexie.org/docs/ | IndexedDB wrapper for offline catalog, shifts, and outbox queue; request `navigator.storage.persist()`. |
| **Capacitor** | `v8.x` | https://capacitorjs.com/docs | Optional native mobile wrapper for Bluetooth printing, native camera scanner, and mobile storage reliability. |

---

## Infrastructure

| Technology | Official Documentation |
| :--- | :--- |
| **Docker & Docker Compose** | https://docs.docker.com/compose/ |
