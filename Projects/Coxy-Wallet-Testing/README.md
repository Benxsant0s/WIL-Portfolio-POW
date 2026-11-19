# Coxygen Global University – Website Testing Report

A usability and functionality test of the **Coxygen Global University** website (coxygen.co), carried out as part of the WIL/IE module.

| | |
|---|---|
| **Website tested** | Coxygen Global University |
| **Testers** | Nsungu Mushaba, Kamogelo Moruri, Benedict Monsemvula |
| **Date** | 21/08/2026 |
| **Module** | WIL/IE |

---

## Table of Contents

1. [Introduction](#introduction)
2. [Testing Objectives](#testing-objectives)
3. [Areas Tested](#areas-tested)
4. [Summary of Findings](#summary-of-findings)
5. [Detailed Findings](#detailed-findings)
6. [Overall Recommendations](#overall-recommendations)
7. [Conclusion](#conclusion)

---

## Introduction

The purpose of this testing is to identify problems and areas for improvement on the website. The testing focuses on functionality, navigation, usability, responsiveness and overall user experience.

The main goal is to find issues that make the website difficult or confusing to use, and to recommend suitable solutions. The website was tested from a user's perspective, with particular attention to navigation, buttons, links, scrolling, loading, mobile usability and accessibility.

## Testing Objectives

- Identify problems users may experience while using the website
- Determine whether buttons, links and navigation features work correctly
- Identify usability problems that could confuse users
- Test whether important features are easily accessible
- Identify problems on different devices
- Evaluate the overall user experience
- Recommend improvements that make the website easier and more convenient to use

## Areas Tested

| Testing Area | What Was Tested |
|---|---|
| Navigation | Buttons, menus and page navigation |
| Functionality | Whether website features work correctly |
| Links | Social media, email and telephone links |
| Scrolling | Mouse and page scrolling functionality |
| Homepage | Homepage content and interactive elements |
| Mobile Responsiveness | How features behave on mobile devices |
| Usability | Whether features are easy for users to understand |
| Loading | Loading indicators and page behaviour |
| Coxy Wallet | Functionality, usability and mobile behaviour |

## Summary of Findings

**11 issues** were found: **6 High** and **5 Medium** severity.

| ID | Area Tested | Problem | Severity |
|---|---|---|---|
| [P01](#p01--basic-information-button) | Basic Information | Button redirects to homepage | 🟠 Medium |
| [P02](#p02--performance-reports-back-button) | Performance Reports | Back button does not work | 🔴 High |
| [P03](#p03--website-navigation-gets-stuck) | Navigation | Website gets stuck on a section | 🟠 Medium |
| [P04](#p04--mouse-scrolling-stops-working) | Scrolling | Mouse scrolling stops working | 🔴 High |
| [P05](#p05--coxy-wallet-blocks-scroll-button) | Coxy Wallet | Blocks scroll button | 🟠 Medium |
| [P06](#p06--youtube-videos-disappear) | Homepage | YouTube videos disappear | 🟠 Medium |
| [P07](#p07--previous-page-navigation) | Navigation | Previous Page function does not work | 🔴 High |
| [P08](#p08--telegram-social-media-link) | Social Links | Telegram link does not redirect | 🟠 Medium |
| [P09](#p09--email-and-phone-number-links) | Contact Information | Email and phone links do not work | 🔴 High |
| [P10](#p10--coxy-wallet--beginner-usability) | Coxy Wallet | Not beginner-friendly | 🔴 High* |
| [P11](#p11--coxy-wallet--mobile-usability) | Mobile | Coxy Wallet controls do not work correctly | 🔴 High |

\* *P10 is rated **Medium** in the detailed finding but **High** in the original summary table; the discrepancy is in the source document.*

## Detailed Findings

### P01 – Basic Information Button
- **Feature:** Basic Information
- **Problem:** The button redirects to the homepage (after a green loading bar) instead of the section with the basic information. Users must scroll to find it.
- **Expected:** The button takes the user directly to the relevant section or page.
- **Impact:** Confusing, especially for new or inexperienced users.
- **Severity:** Medium
- **Recommendation:** Link the button directly to the relevant section or page.

### P02 – Performance Reports Back Button
- **Feature:** Performance Reports
- **Problem:** The Back button does not return the user to the previous page; the user has to use the website tabs.
- **Expected:** Back returns the user to the previous page.
- **Impact:** User frustration from a missing expected navigation feature.
- **Severity:** High
- **Recommendation:** Fix the Back button.

### P03 – Website Navigation Gets Stuck
- **Feature:** Website Navigation
- **Problem:** The page sometimes becomes stuck on the Real-World Assets solutions section; the user must click **Show Menu** to continue.
- **Expected:** Normal navigation without getting stuck.
- **Impact:** Interrupts browsing.
- **Severity:** Medium
- **Recommendation:** Review the navigation to prevent the page sticking on a section.

### P04 – Mouse Scrolling Stops Working
- **Feature:** Scrolling
- **Problem:** After some time on the site, the mouse scroll wheel stops responding.
- **Expected:** Uninterrupted scrolling throughout the session.
- **Impact:** Users cannot reach content further up or down the page.
- **Severity:** High
- **Recommendation:** Investigate the cause and ensure scrolling works throughout a session.

### P05 – Coxy Wallet Blocks Scroll Button
- **Feature:** Coxy Wallet and page navigation
- **Problem:** The Coxy Wallet overlaps the scroll-down button, making it hard or impossible to use.
- **Expected:** Navigation controls remain visible and accessible.
- **Impact:** Users may be unable to use the scroll button.
- **Severity:** Medium
- **Recommendation:** Reposition the Coxy Wallet so it does not overlap navigation controls.

### P06 – YouTube Videos Disappear
- **Feature:** Homepage YouTube videos
- **Problem:** After exploring the site, the homepage videos sometimes disappear until the page is reloaded.
- **Expected:** Videos remain visible when the user returns to the homepage.
- **Impact:** Users may think the videos are unavailable or the site is broken.
- **Severity:** Medium
- **Recommendation:** Find out why videos disappear after navigation and keep them available.

### P07 – Previous Page Navigation
- **Feature:** Previous Page navigation
- **Problem:** The Previous Page function does not work correctly; users must use the website tabs instead.
- **Expected:** Users can return to the previous page easily.
- **Impact:** Less convenient navigation and possible confusion.
- **Severity:** High (per summary table)
- **Recommendation:** Review navigation so Previous Page works correctly.

### P08 – Telegram Social Media Link
- **Feature:** Social media links
- **Problem:** The Telegram link stays on the same page instead of redirecting to the official Telegram account.
- **Expected:** The link opens the official Telegram account.
- **Impact:** Users cannot easily reach the organisation's Telegram.
- **Severity:** Medium
- **Recommendation:** Check and update the link to the correct official account.

### P09 – Email and Phone Number Links
- **Feature:** Homepage contact information
- **Problem:** The email addresses and phone number on the homepage do not work as links.
- **Expected:** Email opens a mail client; phone number starts a call on supported devices.
- **Impact:** Users may struggle to contact the organisation.
- **Severity:** High
- **Recommendation:** Check and correct the email and telephone links.

### P10 – Coxy Wallet – Beginner Usability
- **Feature:** Coxy Wallet
- **Problem:** The wallet's purpose and functionality are not immediately clear to inexperienced users.
- **Expected:** First-time users understand what it does and how to use it.
- **Impact:** New users may be confused or unsure how to use it.
- **Severity:** Medium (detailed finding) / High (summary table)
- **Recommendation:** Add clearer labels, instructions, explanations or tooltips.

### P11 – Coxy Wallet – Mobile Usability
- **Feature:** Mobile responsiveness
- **Problem:** The wallet's minimise/maximise controls do not respond on mobile; the wallet stays in place.
- **Expected:** The wallet responds to touch and can be minimised and maximised easily.
- **Impact:** Difficult and frustrating to use on mobile.
- **Severity:** High
- **Recommendation:** Improve mobile responsiveness and make the controls work on mobile devices.

## Overall Recommendations

The website needs improvements in navigation, functionality, mobile responsiveness, security and user-friendliness. Suggested technologies:

| Technology | Suggested Use |
|---|---|
| HTML & CSS | Improve layout, design and mobile responsiveness |
| JavaScript | Fix buttons, links, scrolling, menus, loading and other interactive functions |
| Haskell | Backend functions where reliable, well-structured code is needed |
| Blockchain | Secure, trustworthy records such as transactions or important user information |
| SQL | Store and manage website and user information |
| Python, Java, PHP or Node.js | Backend functions and database connectivity |

The website should be **re-tested** after the improvements to confirm the problems have been fixed.

## Conclusion

The testing identified several issues that may affect the overall user experience. While the website provides useful features and information, some navigation, functionality and usability problems make it difficult for users to access information and use certain features.

Fixing these problems would improve usability, accessibility and the overall user experience. Further testing should follow once the improvements are implemented.

---

## Authors

- Nsungu Mushaba
- Kamogelo Moruri
- Benedict Monsemvula
