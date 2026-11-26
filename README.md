# Campus Eats Project

Good food. For everyone.

One food-ordering experience for the web and Android.

- Live website: https://khayaguzu.helioho.st/
- Demonstration video: https://youtu.be/AyzUCJlN56Q
- GitHub repository:
  https://github.com/HChristopherNaoyuki/campus-eats-project.git
- Release notes: RELEASE_NOTES.md
- Screenshots: see the `docs/images` folder

Version Information: 12 November 2026

This README describes the current, deployed Campus Eats experience
(Android v1.1.0). The updated source is on the
`codex/google-sso-web-android` branch. The repository's default
`main` source still includes some older prototype files until those
changes are merged. Do not use an old APK or an old database schema
as evidence of the updated release.

Payment safety.

Campus Eats currently demonstrates no-charge practice checkout. The
card and Instant EFT fields use fictitious sample values. No actual
funds are collected or transferred. Real PayFast processing is
disabled until configured and independently tested. Google sign-in
is also unavailable until OAuth credentials are configured.

## Table of Contents

1. [Purpose and Problem Statement](#1-purpose-and-problem-statement)
2. [Target Users and Functional Scope](#2-target-users-and-functional-scope)
3. [Screenshots](#3-screenshots)
4. [System Design and Architecture](#4-system-design-and-architecture)
5. [Design Considerations](#5-design-considerations)
6. [Technology Stack and Project Structure](#6-technology-stack-and-project-structure)
7. [How to Use Campus Eats](#7-how-to-use-campus-eats)
8. [Local Development and Android Build](#8-local-development-and-android-build)
9. [GitHub and Version-Control Workflow](#9-github-and-version-control-workflow)
10. [GitHub Actions and Automated Testing](#10-github-actions-and-automated-testing)
11. [Deployment, Configuration and Data Safety](#11-deployment-configuration-and-data-safety)
12. [Innovations and Improvements](#12-innovations-and-improvements)
13. [Testing Evidence, Limitations and Next Steps](#13-testing-evidence-limitations-and-next-steps)
14. [Documentation and Release Notes](#14-documentation-and-release-notes)
15. [Disclaimer](#15-disclaimer)

## 1. Purpose and Problem Statement

### What Is Campus Eats?

Campus Eats is a combined food-ordering website and Android
application. Customers can browse meals from different kitchens,
read product descriptions and prices, place collection orders, and
follow progress using a unique collection number. Authorized
administrators manage menu items, vendors, stock, and order
preparation.

### The Problem We Address

A customer who visits several food counters may need to search for
available items manually, check prices at each counter, and wait
while an order is prepared. Manual ordering can also make it hard
for the customer to know when to collect an item, and hard for a
kitchen to keep an organized record of orders. Campus Eats provides
one digital ordering interface that brings together the catalog,
account information, basket, order details, and collection status.

### Purpose of Each Platform

The responsive website is accessible from a desktop, tablet, or
mobile browser without installing software. It offers convenient
menu browsing and online collection ordering.

The Android application provides a phone application using a Kotlin
WebView that opens the same hosted interface. It gives consistent
ordering on a personal Android device.

The shared PHP API and MariaDB database enforce login permissions,
calculate prices, and store menus, customer profiles, and orders
centrally. They provide consistency between the two interfaces and
one authoritative data source.

The administrator dashboard manages products, kitchens, stock,
account information, and order progression. It offers better
operational visibility and control.

### Objectives and Expected Impact

- Make the menu easier to discover through search, categories,
  kitchen filters, prices, images, and stock labels.
- Allow customers to order ahead, choose a collection time, and
  obtain a trackable order reference.
- Provide the same account and order records through the website
  and the Android app.
- Protect administrator-only actions through server-side
  authorization.
- Demonstrate the complete ordering workflow without asking for or
  collecting real payment details.
- Reduce the complexity of maintaining two separate frontends by
  reusing the same hosted interface.

These are project objectives and intended benefits. They are not
measured claims about reduced waiting times or commercial sales.

## 2. Target Users and Functional Scope

There are two active account roles in the updated application. The
capabilities of each role are described below in plain terms.

Customers can create an account using name, email, and password.
Administrators are provisioned separately.

Customers and Administrators can both browse products and search or
filter by category or kitchen.

Customers and Administrators can both manage their own profile,
theme, and notification preference.

Customers and Administrators can both add food to a basket and
select a collection time.

Customers and Administrators can both complete no-charge practice
checkout and receive a collection number.

Customers and Administrators can both view their own orders and the
current progress of those orders.

Only Administrators can manage vendors, product descriptions,
prices, and stock.

Only Administrators can advance or cancel applicable orders.

Only Administrators can access administration and account-
management controls.

The updated public registration has no Student or Lecturer
selection, no institution-number requirement, and no new-order
student discount. Legacy database fields may remain solely for
historical compatibility. Historical order totals are preserved
during migration.

The sample catalog includes multiple kitchens, food categories,
descriptions, illustrative photographs, prices in South African
rand, and collection locations.

## 3. Screenshots

Screenshots are stored in the `docs/images` folder. No images are
embedded in this README file. See Section 15 for the full media
policy.

The screenshots are actual captures of the Campus Eats website and
Android application, supplied during project testing on
12 November 2026. They are documentation evidence, not generated UI
mockups. Screens show sample catalog items and no-charge
transactions.

### Website Interface

Figure 1 shows the website homepage, branding, category navigation,
and food catalog. File: `docs/images/website-home.jpg`.

Figure 2 shows general customer registration. A new customer enters
a full name, email, and password, without choosing an academic
account type. File: `docs/images/website-registration.jpg`.

Figure 3 shows vendor management, visible only to an authorized
administrator. File: `docs/images/website-admin-vendors.jpg`.

### Android Application

The application loads the same hosted catalog through a Kotlin
WebView, with responsive bottom navigation on a phone.

Figure 4 shows the Android home screen. File:
`docs/images/android-home.jpg`.

Figure 5 shows the Android basket. File:
`docs/images/android-basket.jpg`.

Figure 6 shows order confirmation. File:
`docs/images/android-order-confirmation.jpg`.

Screenshots are reduced in resolution for repository readability
and deliberately exclude passwords and real payment credentials.

## 4. System Design and Architecture

### Architecture

The system has four main parts.

1. A desktop or mobile browser connects to a hosted responsive
   PHP, HTML, CSS, and JavaScript interface.
2. An Android Kotlin WebView connects to the same hosted interface.
3. The interface calls a same-origin PHP JSON API.
4. The API validates sessions and Customer or Administrator
   permissions, then applies products, baskets, orders, and
   checkout rules against a hosted MySQL or MariaDB database.

Key design decision. The Android application is not a second
independent ordering implementation. It loads the shared HTTPS web
interface, which calls the same-origin PHP API. The database and PHP
application remain on the server. This reduces duplicated logic and
helps ensure that a price or role rule is enforced once.

### Data Flow for a Collection Order

1. The customer signs in to a Customer account on the browser or
   within the Android app.
2. The frontend requests catalog data from the hosted `api.php`
   endpoint.
3. Products are added to a basket, and quantities are displayed
   with server-backed prices.
4. The customer chooses a collection time and enters optional
   notes.
5. The practice checkout shows Card or Instant EFT sample options
   and requires account confirmation.
6. The PHP backend validates the account and order, creates the
   order record, and generates a collection reference.
7. The customer views the reference under My orders. The
   Administrator updates order progress as the kitchen prepares it.

### Database Model

The project uses an SQL schema with records for `users`,
`vendors`, `products`, `cart`, `orders`, `order_items`,
`order_payments`, `mobile_tickets`, `audit_log`, and `rate_limits`.
The relationship is based on account and product identifiers, and
order lines preserve a snapshot of product information.

Relationships, in plain terms. A user places many orders. A user
owns many cart entries. A vendor supplies many products. A product
can be selected in many cart entries. An order contains many order
items. An order item references a product. An order has at most one
checkout payment record. A user can create many mobile handoff
tickets. A user can be the administrator of many audit log entries.

Money is stored as integer cents, so R45.00 is represented by 4500.
The updated migration changes academic account roles to Customer
while keeping existing accounts, orders, and historical amounts. The
source schema and migration are linked in Section 11.

## 5. Design Considerations

Usability. The design uses clear labels and a straightforward
ordering journey: menu, basket, checkout, collection number, order
tracking.

Visual consistency. The design uses the same green, orange, and
white Campus Eats identity across website and Android rather than
different branding on each platform.

Responsive design. Desktop screens use a sidebar. Narrow screens
use a bottom navigation bar, adaptive cards, fluid dialogs, and
appropriately sized inputs.

Accessibility. Semantic navigation, labels, understandable buttons,
focus indications, and readable contrast are part of the UI design.
Further accessibility auditing is still valuable.

Maintainability. The HTML, CSS, and JavaScript frontend is shared
instead of implementing duplicate price calculations and ordering
workflows in Kotlin.

Security. The hosted system uses HTTPS, hashed passwords, PHP
sessions, role checks, and private server configuration outside the
public web root. A hidden menu item is not a substitute for
server-side authorization.

Data integrity. Product prices are stored in cents, quantity and
stock are validated on the server, order history is preserved, and
destructive migration over an existing database is avoided.

Payment integrity. No-charge practice checkout is clearly
identified, fictitious data is used, and validated payment-provider
callbacks are required before any future real-money claims.

Reliability. The system includes error and retry behavior, isolated
tests, and repeatable builds. Android connectivity requires a
reachable HTTPS host.

Privacy. Database passwords, OAuth client secrets, customer exports,
and real card data are not committed to a public repository.

Scalability. A centralized API and database simplify consistency,
but production scale would require performance, capacity, security,
and operational testing.

## 6. Technology Stack and Project Structure

### Technology Stack

The frontend uses HTML5, CSS3, and vanilla JavaScript.

The website and backend use PHP and a JSON REST-style API.

The database uses MySQL or MariaDB with PDO.

The Android client uses Kotlin, Android WebView, and AndroidX.

The build uses the Gradle wrapper, Java 17, and Android SDK 35.

Hosting uses HelioHost Plesk and HTTPS.

Version control uses Git and GitHub.

Continuous integration uses GitHub Actions for APK compilation,
PHP checkout, and SSO or integration checks.

### Project Structure

```text
campus-eats-project/
├── README.md
├── RELEASE_NOTES.md
├── docs/
│   └── images/
├── .github/workflows/
│   ├── build-android-apk.yml
│   ├── test-checkout-php.yml
│   └── test-sso.yml
└── CampusEats_Website_Android_App_Separated/
    ├── Website/
    │   ├── web/
    │   ├── server/
    │   ├── database/
    │   ├── tests/
    │   └── docs/
    └── Android_App/
        ├── app/src/main/
        ├── gradle.properties
        └── gradlew / gradlew.bat
```

### Folder Notes

The `docs/images` folder holds all visual media. No images belong in
this README.

The `web` folder holds the public PHP interface, API, and assets.

The `server` folder holds private PHP logic and the configuration
template.

The `database` folder holds the schema and migrations.

The `tests` folder holds automated tests.

The `docs` folder holds deployment, API, and SSO details.

The active implementation is in the general-customer source branch.
Version 1.1.0 is configured in its
`Android_App/app/build.gradle.kts`. The older default branch may not
yet contain all those source changes.

## 7. How to Use Campus Eats

### Customer

1. Open the website at https://khayaguzu.helioho.st/ or install the
   Android test APK.
2. Choose Sign in, or Create an account with a full name, email,
   and password.
3. Open Explore menu. Search, filter, and select food.
4. Open Basket, check quantities, and choose a collection time.
5. Continue to the clearly labeled no-charge checkout. Select a
   sample Card or Instant EFT option, and never enter real payment
   details.
6. Complete the account-confirmation step.
7. Record the generated `CE...` collection number.
8. Open My orders and refresh to see progress through Placed,
   Preparing, Ready, and Collected.

### Administrator

A separately provisioned Administrator sees the Administration
section. Its tabs support Orders, Products, Users, and Vendors. The
Administrator can update menu products and availability, maintain
collection locations, and progress valid orders through their
fulfillment stages. Normal Customers cannot access these functions.

## 8. Local Development and Android Build

### Requirements

- Android Studio with JDK 17 and a compatible Android SDK.
- PHP with PDO MySQL and a local MySQL or MariaDB instance for
  optional website development.
- WampServer on Windows if local PHP hosting is required.
- Internet access for testing against HelioHost.

### Clone the Source

```powershell
git clone \
  https://github.com/HChristopherNaoyuki/campus-eats-project.git
cd campus-eats-project
git switch codex/google-sso-web-android
```

### Run the Android App

1. Open only
   `CampusEats_Website_Android_App_Separated/Android_App` in Android
   Studio.
2. Check its `gradle.properties` values.

   ```properties
   CAMPUS_BASE_URL=https://khayaguzu.helioho.st/
   CAMPUS_SSO_URL=https://khayaguzu.helioho.st/
   ```

3. Synchronize Gradle, select an emulator or Android phone, and
   press Run.
4. Or build a debug APK.

   ```powershell
   cd CampusEats_Website_Android_App_Separated/Android_App
   .\gradlew.bat :app:assembleDebug
   ```

5. Find the APK at
   `app/build/outputs/apk/debug/app-debug.apk`.

Note. `10.0.2.2` is an Android emulator address for a developer's
local machine. It is not the URL for a physical phone. For shared or
phone builds, use the actual hosted HTTPS address. An APK created
before a configuration change must be rebuilt and reinstalled.

### Run the Website Locally

For a new local database only, use the schema and seed SQL provided
by the source branch. Copy `Website/server/config.example.php` to
the private `config.php` and set your local database credentials. Do
not place this file in Git or inside a publicly served directory.

For an existing database with orders and accounts, back up the
database and follow the migration guide instead of running a fresh
seed import.

See the Website setup and Android setup documents in the
`CampusEats_Website_Android_App_Separated/Website` and
`CampusEats_Website_Android_App_Separated/Android_App` folders.

## 9. GitHub and Version-Control Workflow

GitHub is the project's source-control, collaboration,
documentation, and continuous-integration platform. It is distinct
from the HelioHost server, which actually runs the PHP website.

How GitHub is used.

Repository. `HChristopherNaoyuki/campus-eats-project` stores the
website, Android app, SQL migrations, tests, and documentation.

Commits. Track changes to source files and provide a history of the
prototype and later improvements.

Branches. Separate the main project baseline from the more recent
general-customer and SSO update.

Pull requests. Allow changes to be reviewed and tested before they
are merged into `main`.

Workflow files. Keep build and test steps versioned under
`.github/workflows/`.

Artifacts. Distribute a generated debug APK for testing without
requiring the reviewer to compile it locally.

README and release notes. Provide onboarding, architecture,
development steps, and a change history.

Private configuration. Actual database passwords and OAuth secrets
must never be committed.

### Example Contribution Process

```bash
git checkout -b docs/update-project-report
git add README.md RELEASE_NOTES.md docs/images
git commit -m "docs: document Campus Eats design, CI and release"
git push -u origin docs/update-project-report
```

Create a pull request, review the diff, and check the relevant
workflows before merging. The command above illustrates a good
workflow. It is not a claim that this exact branch was created
during development.

## 10. GitHub Actions and Automated Testing

GitHub Actions runs workflows defined in YAML files. It can install
build dependencies, compile the Android project, run PHP and
JavaScript tests, and publish artifacts. This provides repeatable
evidence that a commit can build and pass defined checks. It does
not automatically prove that the live site, phone, or real payment
provider is working.

### Workflow Overview

The Build Campus Eats Android APK workflow is triggered by Android
changes on `main`, by pull requests in the updated branch, or by a
manual run. It checks out the repository, installs Temurin Java 17
and Android SDK 35, configures Gradle caching, runs
`:app:assembleDebug`, verifies the APK exists, and confirms the
HTTPS URL is compiled. Its output is the
`CampusEats-Android-debug-APK` artifact, kept for 30 days.

The Test Campus Eats Checkout PHP workflow is triggered by
checkout-related changes, a pull request, or a manual run. It
installs PHP 8.3, lints payment, API, and test PHP, and runs payment
helper tests. Its output is a pass or fail result in Actions.

The Test Campus Eats SSO workflow is triggered by an updated branch
pull request, a manual run, or a push to `main` after merge. It
launches an isolated MySQL 8 service, lints PHP, checks JavaScript
syntax, runs session and payment tests, runs migration checks,
customer-order integration, the shared API contract, and SSO ticket
and session tests. Its output is a pass or fail integration result.

### Successful Workflow Evidence, 12 November 2026

The following GitHub Actions runs completed successfully against
the updated feature-branch commit
`abaade98cbc3a5276629fd1a0885e1bbcf35298a`.

- Android APK build, run 43159988806.
- Checkout PHP tests, run 43159988817.
- SSO and customer integration tests, run 43159988788.

### How to Download the Generated APK

1. Open GitHub Actions at
   https://github.com/HChristopherNaoyuki/campus-eats-project/actions.
2. Choose a successful Build Campus Eats Android APK run for the
   updated source branch, not an older prototype run.
3. Open Artifacts and download `CampusEats-Android-debug-APK`.
4. Extract the ZIP and install `app-debug.apk` on an Android test
   phone.
5. Confirm the app points to https://khayaguzu.helioho.st/.

Artifacts expire after the configured retention period. A debug APK
is suitable for testing and coursework. It is not a signed Google
Play release.

### Pipeline Summary, in Plain Terms

1. A commit or pull request starts GitHub Actions.
2. Actions runs the Android APK build, the PHP checkout tests, and
   the customer and SSO integration tests.
3. The debug APK is stored as an artifact for phone testing.
4. Test results are reported back as CI feedback.

## 11. Deployment, Configuration and Data Safety

The hosted deployment separates public website code from private
server code.

```text
HelioHost home directory/
├── httpdocs/     Public index.php, api.php, SSO, and assets
└── server/       Private bootstrap, sessions, payments, config.php
```

For the upgraded Customer release, follow these steps.

1. Back up the existing MariaDB database and private configuration
   first.
2. Apply `database/general_customers_migration.sql` to the existing
   database after reviewing it.
3. Preserve existing customer IDs, passwords, baskets, orders, and
   historical prices.
4. Update the public web code and the private PHP server code
   together.
5. Keep `server/config.php` out of GitHub and out of `httpdocs`.
6. Test customer registration, customer permissions, account
   settings, menu, checkout, order history, and administrator
   controls.
7. Rebuild the Android APK when Kotlin or Gradle settings change.

### Useful Documents in the Updated Branch

The following documents are in the
`CampusEats_Website_Android_App_Separated/Website/docs` folder.

`GENERAL-CUSTOMERS.md` covers migration and the updated role model.

`HELIOHOST-DEPLOYMENT.md` covers backups, deployment sequence,
tests, and rollback.

`API.md` provides API documentation.

`SSO-DEMONSTRATION.md` provides the SSO configuration guide.

## 12. Innovations and Improvements

This release is an incremental improvement of an existing working
prototype. It is not a claim that every feature is new to computing.
The following changes differentiate the updated project from its
earlier Campus Eats version.

Audience. The earlier prototype used Student and Lecturer oriented
registration. The updated v1.1.0 uses general public Customer
registration, with a separate Administrator. This supports more
inclusive onboarding and simpler role rules.

Branding. The earlier prototype used an institution-related design.
The updated v1.1.0 uses a consistent independent Campus Eats logo,
colors, and food-led UI. This creates a stronger, coherent product
identity.

Android experience. The earlier prototype could suffer from local
server or emulator URL connectivity issues. The updated v1.1.0 uses
a shared hosted HTTPS interface with responsive styling and
physical-device testing. This gives the same experience across
browser and phone.

Identity and permissions. The earlier prototype used academic
account verification and role requests. The updated v1.1.0 uses
Customer-only public signup with server-enforced Administrator
permissions. This reduces privilege-escalation risk.

Shared information. The earlier prototype had existing website and
app integration. The updated v1.1.0 provides verified current
shared accounts, order records, and profile settings. This gives one
central source of truth.

Checkout. The earlier prototype had earlier checkout controls. The
updated v1.1.0 has an explicit sample Card and Instant EFT
no-charge workflow with a confirmation reference. This offers an
end-to-end demonstration without financial transactions.

Order tracking. The earlier prototype had a basic collection
workflow. The updated v1.1.0 has a clear order stage, collection
number, and order-history views. This gives better visibility of
progress.

Preferences. The earlier prototype had earlier account
configuration. The updated v1.1.0 has a mobile-friendly profile,
theme, and order-update preferences. This provides a personalized
UI and account management.

Quality assurance. The earlier prototype relied primarily on manual
testing. The updated v1.1.0 uses GitHub Actions APK compilation, PHP
checkout checks, and SSO and customer integration tests. This gives
repeatable build and regression evidence.

Deployment. The earlier prototype used local-development
assumptions. The updated v1.1.0 uses HelioHost private and public
file separation with migration and rollback guidance. This supports
more reliable deployment practice.

### Innovative Features in the Context of This Coursework

1. Shared web-and-app architecture. A Kotlin WebView reuses the
   hosted responsive interface and server-side rules.
2. Controlled account migration. Historical orders remain while
   prior academic roles are converted to general Customers.
3. Server-authoritative ordering. Stock, totals, permissions, and
   order states are validated by the PHP backend.
4. Practice payment experience. Checkout supports realistic Card
   and Instant EFT selection and collection confirmation without
   charging customers.
5. Cross-platform order visibility. The browser and phone display
   the same persisted order records once each client is
   authenticated.
6. Automated build and test pipeline. GitHub Actions provides
   reproducible verification and testable Android APK artifacts.

For the complete detailed change log and evidence, read
`RELEASE_NOTES.md`.

## 13. Testing Evidence, Limitations and Next Steps

### Testing Recorded for This Project

- The live website loaded its updated menu, logo, category filters,
  and food images.
- Android Studio produced a successful APK build, and the app opened
  against the hosted HTTPS address.
- Android's responsive CSS issue was corrected and verified
  visually in the emulator.
- Customer login and My orders were exercised through Android.
- A no-charge practice checkout produced a collection number, for
  example `CE000016` and `CE000019`. Corresponding order details
  were visible under My orders.
- Customer access and administrator-only navigation were checked
  during testing.
- Three GitHub Actions workflows passed for the updated source
  commit, as listed in Section 10.

### Known Limitations

No actual payment processing. No-charge checkout must not be
described as payment received. Real provider integration requires
additional validation and merchant setup.

Google Sign-In not configured. Source support exists, but production
OAuth keys, consent-screen configuration, and redirect validation
are still required.

Debug APK only. Publishing to Google Play requires a separately
signed Android App Bundle and further release testing.

Internet dependence. The Android WebView requires the hosted site
and API to be reachable.

Sample content. Product photographs and catalog entries are
illustrative.

Unmerged source branch. The v1.1.0 Customer implementation is on
`codex/google-sso-web-android` until reviewed and merged to `main`.

Accessibility and load testing. More systematic accessibility,
performance, device compatibility, and security testing are
appropriate before production use.

### Future Improvements

- Configure and test Google OAuth sign-in in a controlled
  environment.
- Integrate a genuine payment provider with verified server-side
  callbacks before enabling real charges.
- Complete release signing, Play Store assets, device testing, and
  store policy checks.
- Add wider integration, usability, accessibility, and performance
  test coverage.
- Improve customer notifications and production monitoring when
  justified by requirements.

## 14. Documentation and Release Notes

`RELEASE_NOTES.md` provides the prototype-to-v1.1.0 update history
and specifically identified innovative features.

`GENERAL-CUSTOMERS.md` covers migration and the updated role model.

`HELIOHOST-DEPLOYMENT.md` covers backups, deployment sequence,
tests, and rollback.

`Website/README.md` covers local website setup.

`Android_App/README.md` covers device configuration and APK
creation.

The first three documents are in
`CampusEats_Website_Android_App_Separated/Website/docs`. The last
two are the README files of their respective folders.

Project Team: 
- Makaya Guzu
- Naoyuki Christopher Higaki.

Project type:
- Academic software-development portfolio.

Last Documentation Update:
- 26 November 2026.

## 15. Disclaimer

```
UNDER NO CIRCUMSTANCES SHOULD IMAGES OR EMOJIS BE INCLUDED DIRECTLY IN
THIS FILE. ALL VISUAL MEDIA, INCLUDING SCREENSHOTS AND IMAGES OF THE
APPLICATION, MUST BE STORED IN A DEDICATED FOLDER WITHIN THE PROJECT
DIRECTORY. THIS FOLDER SHOULD BE CLEARLY STRUCTURED AND NAMED
ACCORDINGLY TO INDICATE THAT IT CONTAINS ALL VISUAL CONTENT RELATED TO
THE APPLICATION, FOR EXAMPLE A FOLDER NAMED IMAGES, SCREENSHOTS, OR
MEDIA. THE AUTHOR IS NOT LIABLE OR RESPONSIBLE FOR ANY MALFUNCTIONS,
DEFECTS, OR ISSUES THAT MAY OCCUR AS A RESULT OF COPYING, MODIFYING, OR
USING THIS SOFTWARE. IF ANY PROBLEMS OR ERRORS ARE ENCOUNTERED, PLEASE
DO NOT ATTEMPT TO FIX THEM SILENTLY OR OUTSIDE THE PROJECT. INSTEAD,
SUBMIT A PULL REQUEST OR OPEN AN ISSUE ON THE CORRESPONDING GITHUB
REPOSITORY SO THAT IT CAN BE ADDRESSED APPROPRIATELY BY THE MAINTAINERS
OR CONTRIBUTORS.
```

---

END OF DOCUMENT

---
