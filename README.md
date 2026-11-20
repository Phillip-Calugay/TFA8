# Simple POS System — TFA8

Phillip E. Calugay · IT0049 · TC31

TFA8 continues the Laravel TFA5–TFA7 POS: customer and user management now use complete resource controllers, and Laravel Breeze supplies registration, login, logout and password management. All customer and user routes require authentication. Existing validated forms, staff roles and prepared avatars remain available.

Repository: https://github.com/Phillip-Calugay/TFA8

**Local application: http://127.0.0.1:8000**

Local hosting only, as requested. This address is accessible on the computer running PHP. No public deployment or tunnel is used.

## Run this working copy

```bat
cd /d C:\xampp\htdocs\TFA8
start-local.bat
```

Open http://127.0.0.1:8000/register and create your account, then use Customers or Users. Login uses your **email and password**. Keep the server terminal open. If the application is already running on port 8000, open it directly.

The database in this working copy has already been migrated. Keep the existing `.env` and database; do not run `migrate:fresh` or overwrite `.env`.

## Fresh checkout setup

Requirements: PHP 8.2+, Composer, PDO SQLite, Fileinfo, Mbstring, Exif and GD. Dependencies are locked in `composer.lock`. Breeze Blade views use the existing local `public/css/pos.css`; running the application requires no Node build, CDN or external assets.

```bat
git clone https://github.com/Phillip-Calugay/TFA8.git
cd TFA8
composer install
copy .env.example .env
php artisan key:generate
php -r "file_exists('database/database.sqlite') || touch('database/database.sqlite');"
php artisan migrate --seed
php artisan storage:link
start-local.bat
```

Enable `extension=gd` in the PHP configuration before `composer install`. For XAMPP without global GD, use `php -d extension=gd C:\ProgramData\ComposerSetup\bin\composer.phar install`. The batch launcher enables GD, sets upload limits above Laravel's 2 MB limit and uses a writable upload directory. `php artisan serve` requires GD enabled in the PHP configuration because its server child does not inherit command-line `-d` options.

`public/storage` must link to `storage/app/public`. If a copied checkout contains an ordinary empty directory there, move that directory aside before `php artisan storage:link`. Creating the link on Windows may require Developer Mode or an elevated terminal.

Register your own login at `/register`; there is no shared default password. The repeatable seed creates five customers and five staff records. Existing staff records have nullable email/password columns and cannot sign in until credentials are assigned. Staff management creates POS records; registration creates login credentials. New registrations always receive the Staff role. For this assignment, every authenticated user may manage both account types; roles are descriptive, with no additional role restrictions.

## Numbered assignment tasks

The [TFA8 screenshot walkthrough](docs/TFA8-SCREENSHOTS.md) embeds **19 real browser screenshots** covering every numbered task. The [manifest](docs/screenshots/tfa8/manifest.json) records each capture. Earlier TFA7 evidence remains in [its original gallery](docs/SCREENSHOTS.md).

| Task | Implementation | Output evidence |
| --- | --- | --- |
| 1. Continue the Laravel POS | Existing pages, Eloquent records, validation and avatars retained | [Existing POS](docs/screenshots/tfa8/task-1-existing-pos.png) |
| 2. Customer resource refactor | One `Route::resource('customers', CustomerController::class)` with index, create, store, show, edit, update and destroy | [Listing](docs/screenshots/tfa8/task-2-customer-resource-index.png), [create](docs/screenshots/tfa8/task-2-customer-create.png), [updated details](docs/screenshots/tfa8/task-2-customer-show-updated.png), [delete](docs/screenshots/tfa8/task-2-customer-deleted.png) |
| 3. User resource refactor | One `Route::resource('users', UserController::class)` with all seven actions and avatar cleanup on deletion | [Listing](docs/screenshots/tfa8/task-3-user-resource-index.png), [create](docs/screenshots/tfa8/task-3-user-create.png), [update and avatar](docs/screenshots/tfa8/task-3-user-updated-avatar.png), [details](docs/screenshots/tfa8/task-3-user-show.png), [delete](docs/screenshots/tfa8/task-3-user-deleted.png) |
| 4. Breeze authentication | Breeze 2.4.2 installed and Blade scaffolding generated; registration adapted to existing full_name/username fields | [Registration](docs/screenshots/tfa8/task-4-registration.png), [signed-in dashboard](docs/screenshots/tfa8/task-4-registered-dashboard.png) |
| 5. Auth middleware | Both resource registrations are inside an auth middleware group; every read/write action requires login | [Customers redirects](docs/screenshots/tfa8/task-5-customers-requires-login.png), [Users redirects](docs/screenshots/tfa8/task-5-users-requires-login.png) |
| 6. Verify access control | Logout blocks access, invalid credentials fail, successful login returns to the intended page, both lists become accessible | [Logged out](docs/screenshots/tfa8/task-6-logged-out-redirect.png), [invalid password](docs/screenshots/tfa8/task-6-invalid-login.png), [customers after login](docs/screenshots/tfa8/task-6-logged-in-customers.png), [users after login](docs/screenshots/tfa8/task-6-logged-in-users.png) |

![Authenticated user accounts](docs/screenshots/tfa8/task-6-logged-in-users.png)

## Authentication and database structure

- `routes/web.php`: public Home/About; protected dashboard, customer/user resources and profile.
- `routes/auth.php`, `app/Http/Controllers/Auth/`, `app/Http/Requests/Auth/LoginRequest.php`: Breeze authentication, validation, throttling, password reset and confirmation.
- `app/Models/User.php`: Laravel Authenticatable, notifications, hidden password/token and hashed password cast.
- `database/migrations/2026_11_20_000000_add_authentication_to_users.php`: restores nullable email, verification timestamp, password and remember token without replacing existing staff records. The authentication email index has a distinct name because TFA7 renamed the original email column to username.
- `resources/views/auth/`: Breeze Blade forms styled with the existing local stylesheet. Profile supports name/email updates, password updates and password-confirmed account deletion.
- `app/Http/Controllers/CustomerController.php`, `UserController.php`: conventional resource actions, model binding, validation and storage cleanup. Delete actions use CSRF-protected DELETE forms on details pages. Self-deletion through staff management is blocked; the profile page requires the current password.

All migrations and seeders are included as the database submission. Local `.env`, SQLite data, sessions, uploaded files, dependencies and browser tooling are excluded from Git. Password reset mail uses the local log mailer in `.env.example`; reset links appear in `storage/logs/laravel.log`, and no external mail server is required. Email verification routes are scaffolded but verification is not required for account management in this exercise.

Breeze's normal password hashing, login throttling, session regeneration, intended redirects, logout invalidation and CSRF protection are retained. Open registration follows the assignment's login-and-registration requirement.

## Verification and reproduction

```bat
php -d extension=gd vendor/phpunit/phpunit/phpunit
php artisan route:list --except-vendor
php artisan view:cache
php artisan view:clear
composer audit
```

Verified: **44 tests, 323 assertions**, successful Blade compilation and no Composer security advisories after compatible Laravel/Symfony patch updates. Tests use in-memory SQLite and fake storage. They cover all guest resource routes, authenticated CRUD, avatar processing/deletion, registration uniqueness and password confirmation, hashed passwords, login failures/throttling, intended redirects, logout, password reset/confirmation/update, profile changes, seed repeatability and migration rollback/reapply.

To reproduce the screenshots, install Python, Playwright and Microsoft Edge, run the local application, set the `TFA8_DEMO_PASSWORD` environment variable to a temporary password of at least eight characters, then run:

```bat
python -m pip install playwright
python scripts/capture_tfa8.py
python scripts/build_tfa8_docs.py
```

The browser workflow creates a demo login, creates/updates/deletes temporary customer and staff records, uploads and checks a 256×256 avatar, verifies both access states and checks a mobile profile view. Capture passwords are never written to screenshot artifacts or the repository. The manifest uses UTC timestamps.

## References

- [Laravel resource controllers](https://laravel.com/docs/12.x/controllers#resource-controllers)
- [Laravel Breeze source and scaffolding](https://github.com/laravel/breeze)
- [Breeze installation documentation](https://laravel.com/docs/11.x/starter-kits#laravel-breeze)
- [Laravel authentication](https://laravel.com/docs/12.x/authentication)
- [Laravel middleware](https://laravel.com/docs/12.x/middleware)
