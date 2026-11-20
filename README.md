# Eros Automatic Washing Machine

Laravel 13, Vue 3, Inertia 3 and the official Vue starter kit. Public visitors open `/`; the owner signs in at `/login` and works at `/admin`. Registration is disabled. The original account security, passkeys and two-factor settings remain available.

## First Run

Set `APP_URL`, `ADMIN_EMAIL` and `VITE_APP_URL` in `.env`. Optionally set a strong `ADMIN_PASSWORD` before the first seed; it is only used when creating the owner account, never to overwrite an existing password. Do not commit credentials.

```sh
php artisan migrate --seed
php artisan eros:admin --email=you@example.com
npm run dev
```

The admin command asks for the password twice, without displaying it. It can also reset an existing owner's password. If `ADMIN_PASSWORD` is empty during seeding, the seeded account gets an unknown random password until you run this command or use password reset with working mail.

Herd serves the application at `http://eros-automatic-washing-machine.test`. Without Herd, run `php artisan serve --port=8000` and open `http://127.0.0.1:8000`. Vite is the asset server, not the application URL.

The seeder is repeatable and preserves existing passwords, service edits and bookings. In `APP_ENV=local`, the first seed adds 15 clearly marked `DEMO-` bookings. Production seeding never creates fake customers. Remove demo records before taking real bookings; production uses a separate database.

## Mail and Queue

Set `ADMIN_EMAIL` to the inbox that receives booking notifications. This is separate from the public contact email in Business settings. Configure your SMTP provider using the `MAIL_*` variables, set `MAIL_FROM_ADDRESS`, and keep `QUEUE_CONNECTION=database`.

```sh
php artisan queue:work --tries=3 --backoff=30
```

Use a supervised queue worker in production. The default `MAIL_MAILER=log` writes mail to the application log instead of delivering it. Monitor failed jobs using `php artisan queue:failed`; retry after resolving the cause. Booking references remain valid even if mail dispatch fails. The application log records dispatch failures.

## Replace Before Launch

- Add the supplied logo as `public/eros-logo.png`. Its configured path is in `resources/js/config/business.ts`; until present, a washer icon and wordmark are displayed.
- Edit phone, public email, address, service area, hours, Messenger/Viber and map links in `resources/js/config/business.ts`, or save overrides at `/admin/settings`. Database settings take precedence.
- Set actual starting prices and review descriptions at `/admin/services`. Empty database prices intentionally display a peso placeholder.
- Replace `public/washer-service.jpg` with your own service photo. Current demonstration photo: PlanetCare on Unsplash, https://unsplash.com/photos/1626806787461-102c1bfaaea1 (source image: https://images.unsplash.com/photo-1626806787461-102c1bfaaea1). Verify licensing and attribution before launch.
- Confirm cleaning duration, supported brands, payment methods, travel fees and warranty wording in `resources/js/pages/Public/Landing.vue`.
- Set a unique owner password, enable two-factor authentication, configure SMTP, and set `APP_DEBUG=false` and `APP_ENV=production` for deployment.
- Set the real canonical `APP_URL` and `VITE_APP_URL`, without trailing slashes, then rebuild. Add `Sitemap: https://your-domain.example/sitemap.xml` to `public/robots.txt`.

## Booking and Privacy

Mobile numbers accept `09XXXXXXXXX` or `+639XXXXXXXXX`, with spaces and dashes normalized. Dates use Asia/Manila and are checked against configured service days. Booking requests are rate limited and have a honeypot. Available slots are preferences, not reserved capacity; the owner confirms every appointment manually.

Photos accept JPEG, PNG or WebP up to 5 MB. They are stored on Laravel's private local disk and served only through an admin-authorized endpoint. No public storage symlink is required. Back up the database and `storage/app/private`; configure a customer-data retention policy before launch.

Confirmation links expire after seven days and expose only a reference, service and requested schedule, not personal details. Revenue uses completed bookings with a completion history entry in the current Philippine calendar month. All prices are PHP.

## Checks and Production Build

```sh
php artisan test
composer types:check
vendor/bin/pint --test
npm run types:check
npm run check
npm run build
```

Source pages follow the starter kit's existing lowercase root directories: `resources/js/pages/Public`, `resources/js/pages/Admin`, `resources/js/layouts`. Shared UI primitives are preserved under `resources/js/components/ui`.

## Gallery

Public gallery: `/gallery`. Owner management: `/admin/gallery`. Run:

```sh
php artisan migrate
php artisan db:seed --class=GallerySeeder
php artisan storage:link
php artisan test --compact tests/Feature/GalleryTest.php
npm run build
```

The normal `php artisan migrate --seed` also includes the gallery seeder. It creates nine clearly labeled examples with generated placeholder images under `storage/app/public/gallery-samples`. Six photo examples are published and featured so the layout is visible. Three sample testimonials remain unpublished: fictional reviews must not contribute to public trust scores or search-engine review schema. The seeder skips existing sample titles, including those in Trash, rather than replacing owner edits.

Image processing uses native PHP GD, not an additional dependency. PHP must support GD with JPEG, PNG and preferably WebP. Uploaded files accept JPG, JPEG, PNG and WebP up to 5 MB and 12 megapixels (maximum 8000 px per dimension). Processing corrects JPEG orientation, strips metadata, resizes to a maximum width of 1600 px, and makes a maximum-width 600 px thumbnail. Smaller images are not upscaled. WebP is used when supported; JPEG is the fallback. Files receive generated UUID names under `storage/app/public/gallery/{year}/{month}`. Thumbnails use the matching `-thumb` filename, so no additional database columns are needed.

Gallery images are intentionally public media through `/storage`; do not upload private booking photos or identifying documents. Unpublished and soft-deleted gallery records are omitted from public queries and statistics. Soft deletion retains files for restoration. Replacing an image removes its previous full image and thumbnail; permanent deletion from Trash removes all associated files. Back up `storage/app/public` along with the database.

The admin index supports search, type/service/publication/featured filters, grid/table views, bulk publishing and soft deletion, and a Trash filter for restore or permanent deletion. Reorder mode loads the complete non-deleted collection; drag entries or use the move buttons, then save. A changed collection is rejected rather than overwriting a concurrent reorder with an incomplete list.

Public gallery filters use Inertia partial visits and query strings, with 12 entries per page. Comparison controls work with pointer/touch and keyboard input. The lightbox supports focus trapping, Escape, left/right arrow navigation and mobile swipes. Gallery counts and average ratings come from published records; the 120-second statistics cache is cleared on gallery/settings changes. Review and AggregateRating schema is emitted only when at least three published, rated reviews exist.

Before publishing:

- Replace or remove every `Sample:` entry and image with real work. The sample before/after pair deliberately uses the same labeled placeholder, not a fabricated cleaning result.
- Set your verified total machines serviced at `/admin/settings`; 0 hides the figure. This is separate from the number of gallery entries.
- Confirm each customer agreed to publication of their photos, review, name and city/barangay. Use `Anonymous` when requested. Never publish full addresses, phone numbers or private booking information.
