# Zonke.me: Laravel Hosting Setup and Troubleshooting

Deployment and troubleshooting record for **zonke.me**, Tobb Technologies' freelance marketplace, hosted on DirectAdmin. It documents how a blank white screen was traced to four separate faults and fixed, what is still open, and what must be cleaned up before the site is safe to leave public.


## At a glance

| | |
|---|---|
| **Site** | zonke.me |
| **Framework** | Laravel 11.47.0 on PHP 8.3.37 |
| **Control panel** | DirectAdmin (File Manager, MySQL Management, Error Log) |
| **Problems fixed** | 6 (white screen, wrong folder layout, debug lines, folder-name case, view crash, error visibility) |
| **Open issue** | Page renders without CSS, JavaScript or images |
| **Still to do** | Security cleanup (see [checklist](#security-cleanup-checklist)) |

## The problem

After upload, zonke.me showed a blank white page. DirectAdmin's error log was empty, so nothing pointed to the cause. Fixing it meant tracing the failure inside the application rather than the server.

## My role

- Deployed the Laravel project to the DirectAdmin account
- Diagnosed the white screen using a `phpinfo()` test and Laravel's own `laravel.log`
- Fixed the folder layout, folder-name case and a crashing view
- Documented each fault with its symptom, cause and fix
- Identified and listed the security cleanup still required

## Tech used

Laravel 11, PHP 8.3, DirectAdmin, MySQL (phpMyAdmin), Blade templates, Git/GitHub

## Required server layout

The developer's `index.php` loads the framework from one folder **above** the web root, so the server must look like this:

```
domains/
  zonke.me/
    zonke_app/     <- Laravel project: app, bootstrap, config, vendor,
                      storage, .env, artisan
    public_html/   <- index.php, .htaccess, assets/ (css, js, images)
```

Only `index.php`, `.htaccess` and `assets/` should be publicly reachable.

## Troubleshooting log

| # | Symptom | Cause | Fix |
|---|---------|-------|-----|
| 1 | White screen, empty DirectAdmin error log | Fault was inside the app, not the server (a `phpinfo()` test file loaded fine) | Used `laravel.log` instead; deleted the test file afterwards |
| 2 | Whole Laravel project found inside `public_html` | `index.php` expects the project in a separate `zonke_app` folder | Moved the project into `zonke_app` beside `public_html` |
| 3 | `ini_set` debug lines printed as text | Lines were placed above the `<?php` opening tag | Moved them directly below `<?php`, so errors displayed |
| 4 | `vendor/autoload.php` not found | Folder was named `Zonke_app`; Linux is case-sensitive | Renamed to exactly `zonke_app` |
| 5 | Generic "Internal server error" page | Real error hidden by the app's error page | Read `zonke_app/storage/logs/laravel.log`; older entries came from earlier deployments and were ignored |
| 6 | `Undefined array key "extension"` in `seo.blade.php` | View read the image extension with `pathinfo()`, but the SEO image setting was empty and PHP 8 stops on the missing key | Backed up the file as `seo.blade.php.bak`, then made two lines tolerate a missing image (see below) |

### Fix for issue 6

```blade
<meta property="og:image:type" content="image/{{ pathinfo($seoImage ?? getImage(getFilePath('seo') . '/' . $seo->image), PATHINFO_EXTENSION) ?: 'png' }}">
@php $socialImageSize = explode('x', getFileSize('seo') ?: '1200x630') @endphp
```

After this the site loaded. A proper SEO share image should still be uploaded in the admin panel under **SEO Manager**.

## Open issue: page loads without styling

The homepage renders its text, menu and search box but with no CSS or images, and the logo shows as a broken image. The CSS lives as separate files under `public_html/assets/` and the PHP only links to them, so the browser is failing to fetch those files.

**Diagnosis steps**
1. Press **F12**, open the **Console** tab and refresh. Note any red messages such as *Mixed Content* or *Failed to load resource*.
2. Press **Ctrl+U**, find a `<link rel="stylesheet" href="...">` line and open that address in a new tab.

**Likely causes and fixes**

| What happens | Likely cause | Fix |
|--------------|--------------|-----|
| "Mixed Content" or blocked; link starts with `http://` | Site is on https but links are http | Set `APP_URL=https://zonke.me` in `zonke_app/.env`, install an SSL certificate, force https |
| Link points to `localhost` or another domain | Wrong `APP_URL` or old cached config | Fix `APP_URL`, delete `zonke_app/bootstrap/cache/config.php` |
| Link opens a 404 page | File not at that path | Check `public_html/assets/` has the `css` and `js` folders at that exact path and capitalisation |
| CSS file opens fine | Browser cache | Hard refresh with **Ctrl+F5** |

**Status:** still unstyled after the first round of fixes. Next step: capture the console messages and the full address of one CSS link to pin down the cause.

## Security cleanup checklist

The site is **not safe to leave as it is** until these are done. The first three expose data or error details to visitors.

- [ ] Delete the database `.sql` export from `public_html` (anyone can download it). Import it in phpMyAdmin first if not already done
- [ ] Set `APP_DEBUG=false` in `zonke_app/.env`
- [ ] Remove the three `ini_set` / `error_reporting` debug lines from `index.php`
- [ ] Delete `test.php` and `hosting-test.txt` from `public_html`
- [ ] Move any remaining Laravel files (`app`, `config`, `vendor`, `.env`, `artisan`, `composer.json`) out of `public_html` into `zonke_app`
- [ ] Change folder permissions from 777 to 755 and files to 644 (`storage` and `bootstrap/cache` may need 775)
- [ ] Confirm `public_html/.htaccess` exists with the standard Laravel rewrite rules, otherwise inner pages return 404
- [ ] Set `APP_URL=https://zonke.me` and confirm an SSL certificate is installed
- [ ] Upload a proper SEO share image in the admin panel (SEO Manager)
- [ ] Delete `seo.blade.php.bak` once the site is confirmed stable

## Quick reference

**Rules that caused most of the trouble**
- Folder and file names are case-sensitive on this server (`zonke_app`, not `Zonke_app`)
- PHP code must come after `<?php`; anything before it is printed as text
- Check `laravel.log` first; the DirectAdmin error log was empty even though the app was failing

**Where things live**

| What | Where |
|------|-------|
| Laravel error log | `zonke_app/storage/logs/laravel.log` |
| Site settings (database, URL, debug) | `zonke_app/.env` |
| Entry file | `public_html/index.php` |
| SEO view that was patched | `zonke_app/resources/views/partials/seo.blade.php` (backup: `seo.blade.php.bak`) |
| Cached config to delete after changing `.env` | `zonke_app/bootstrap/cache/config.php` |
| CSS, JS and images | `public_html/assets/` |
| DirectAdmin error log | Error Log page for zonke.me |



## Notes

- Never commit `.env` or database exports to this repository
- Server account names and absolute paths have been left out on purpose
- Part of my WIL portfolio: see the [Daily-Log](../../Daily-Log/) for the related days
