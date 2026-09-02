![Status](https://img.shields.io/badge/status-in--progress-yellow)
![Category](https://img.shields.io/badge/category-web--exploitation-blue)

# PortSwigger Expert Write-Ups

> I'm showing the way I solve challenges, not just giving the solution.

## Table of Contents
- [About](#about)
- [Write-ups](#write-ups)
- [Structure](#structure)

## About
I'm an aspiring web application penetration tester. 
This repo documents my problem-solving process through PortSwigger labs — 
focusing on *why* I made each decision, not just the final payload.

## Write-ups
| Challenge | Category | Difficulty | Write-up |
|-----------|----------|------------|----------|
| 2FA bypass using a brute-force attack | Web | Expert | [Read](server_side_vulns/authentication/2FA_bypass_using_a_brute-force_attack/write-up.md) |
| Broken brute-force protection, multiple credentials per request | Web | Expert | [Read](server_side_vulns/authentication/broken_brute-force_protection_multiple_credentials_per_request/write-up.md) |
| Bypassing access controls using email address parsing discrepancies | Web | Expert | [Read](server_side_vulns/business_logic/bypassing_access_controls_using_email_address_parsing_discrepancies/write-up.md) |
| Web shell upload via race condition | Web | Expert | [Read](server_side_vulns/file_upload/web_shell_upload_via_race_condition/write-up.md) |
| Partial construction race conditions | Web | Expert | [Read](server_side_vulns/race_conditions/partial_construction_race_conditions/write-up.md) |

## Structure
```
server_side_vulns/
└── authentication/
    └── 2FA_bypass_using_a_brute-force_attack/
        ├── write-up.md
        └── images/
    └── broken_brute-force_protection_multiple_credentials_per_request/
        ├── write-up.md
        └── images/
└── business_logic/
    └── bypassing_access_controls_using_email_address_parsing_discrepancies/
        ├── write-up.md
        └── images/
└── file_upload/
    └── web_shell_upload_via_race_condition/
        ├── write-up.md
        └── images/
└── race_conditions/
    └── partial_construction_race_conditions/
        ├── write-up.md
        └── images/
```
