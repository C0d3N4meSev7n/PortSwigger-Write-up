![Status](https://img.shields.io/badge/status-in--progress-yellow)
![Category](https://img.shields.io/badge/category-web--exploitation-blue)

# PortSwigger Write-Ups & Solution Notes

> I'm showing the way I solve challenges, not just giving the solution.

## Table of Contents
- [About](#about)
- [Write-ups](#write-ups)
- [Solution Notes](#solution-notes)

## About
I'm a self-taught web application security learner working toward a career as a web penetration tester. This repo collects my work on the PortSwigger Web Security Academy: in-depth write-ups of Expert labs that walk through my reasoning and end with root cause analysis, with solution notes for the other levels on the way.

## Write-ups
### Server-Side Vulnerabilities
| Topic | Challenge | Difficulty |
|-------|-----------|------------|
| Authentication | [2FA bypass using a brute-force attack](server_side_vulns/authentication/2FA_bypass_using_a_brute-force_attack/write-up.md) | Expert |
| Authentication | [Broken brute-force protection, multiple credentials per request](server_side_vulns/authentication/broken_brute-force_protection_multiple_credentials_per_request/write-up.md) | Expert |
| Business logic vulnerabilities | [Bypassing access controls using email address parsing discrepancies](server_side_vulns/business_logic/bypassing_access_controls_using_email_address_parsing_discrepancies/write-up.md) | Expert |
| File upload vulnerabilities | [Web shell upload via race condition](server_side_vulns/file_upload/web_shell_upload_via_race_condition/write-up.md) | Expert |
| Race conditions | [Partial construction race conditions](server_side_vulns/race_conditions/partial_construction_race_conditions/write-up.md) | Expert |
| Server-side request forgery (SSRF) | [SSRF with whitelist-based input filter](server_side_vulns/SSRF/SSRF_with_whitelist-based_input_filter/write-up.md) | Expert |
| XML external entity (XXE) injection | [Exploiting XXE to retrieve data by repurposing a local DTD](server_side_vulns/XXE/exploiting_XXE_to_retrieve_data_by_repurposing_a_local_DTD/write-up.md) | Expert |
| API testing | [Exploiting server-side parameter pollution in a REST URL](server_side_vulns/API/Exploiting_server-side_parameter_pollution_in_a_REST_URL/write-up.md) | Expert |
| Web cache deception | [Exploiting exact-match cache rules for web cache deception](server_side_vulns/web_cache_deception/exploiting_exact-match_cache_rules_for_web_cache_deception/write-up.md) | Expert |

### Client-Side Vulnerabilities
| Topic | Challenge | Difficulty |
|-------|-----------|------------|
| Cross-site scripting (XSS) | [Reflected XSS with event handlers and href attributes blocked](client_side_vulns/XSS/reflected_XSS_with_event_handlers_and_href_attributes_blocked/write-up.md) | Expert |
| Cross-site scripting (XSS) | [Reflected XSS in a JavaScript URL with some characters blocked](client_side_vulns/XSS/reflected_XSS_in_a_JavaScript_URL_with_some_characters_blocked/write-up.md) | Expert |

### Advanced Topics
| Topic | Challenge | Difficulty |
|-------|-----------|------------|
| Insecure deserialization | [Developing a custom gadget chain for Java deserialization](advanced_topics/insecure_deserialization/developing_a_custom_gadget_chain_for_Java_deserialization/write-up.md) | Expert |
| Insecure deserialization | [Developing a custom gadget chain for PHP deserialization](advanced_topics/insecure_deserialization/developing_a_custom_gadget_chain_for_PHP_deserialization/write-up.md) | Expert |
| Server-side template injection (SSTI) | [Server-side template injection in a sandboxed environment](advanced_topics/SSTI/server-side_template_injection_in_a_sandboxed_environment/write-up.md) | Expert |
| Server-side template injection (SSTI) | [Server-side template injection with a custom exploit](advanced_topics/SSTI/server-side_template_injection_with_a_custom_exploit/write-up.md) | Expert |
| JWT attacks | [JWT authentication bypass via algorithm confusion](advanced_topics/JWT/JWT_authentication_bypass_via_algorithm_confusion/write-up.md) | Expert |
| JWT attacks | [JWT authentication bypass via algorithm confusion with no exposed key](advanced_topics/JWT/JWT_authentication_bypass_via_algorithm_confusion_with_no_exposed_key/write-up.md) | Expert |

## Solution Notes
```http
HTTP/1.1 202 Accepted
Content-Type: text/plain

Solution notes for Practitioner and Apprentice labs are in progress. Please wait...
```
