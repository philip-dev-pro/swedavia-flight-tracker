# Swedavia Flight Tracker: Reverse Engineering Project
**Python CLI Application | Swedavia FlightInfo API v2**  
**Author:** Philip Sjoholm  
**Repository:** [https://github.com/philip-dev-pro/swedavia-flight-tracker](https://github.com/philip-dev-pro/swedavia-flight-tracker)

---

## ✈️ Project Overview
The objective of this project was to reverse engineer an existing Python flight-tracking terminal application and reconstruct its source code from scratch. The application connects to Sweden's official airport operator (**Swedavia FlightInfo API v2**) to display, paginate, and analyze live flight arrivals and departures across Sweden's 10 major airports (including Stockholm Arlanda and Göteborg Landvetter).

---

## 📑 Phase 1: Research Phase (Pre-Study)

### Information Gathering
I analyzed video recordings and screenshots of a finished reference implementation to understand how the system worked:
- Identified that the application interfaces with the **Swedavia FlightInfo API v2** using REST endpoints (`/query/arrivals` and `/query/departures`).
- Investigated the authentication header required by Swedavia: `Ocp-Apim-Subscription-Key`.
- Studied how city-to-country relationships were handled offline using an internal mapping file (`city_country.json`) rather than making extra API calls.

### Tools & Technologies
- **Language:** Python (using standard libraries: `urllib.request`, `json`, `datetime`, `collections`).
- **Development Environment:** VS Code and the VS Code Integrated Terminal.
- **Version Control:** Git & GitHub.
- **Data Source:** Swedavia FlightInfo API v2 + Local `city_country.json` database.

### Research Problems & Solutions
- **Problem: Understanding API Authentication**  
  *Problem:* I needed to understand how the application authenticated against Swedavia without requiring a private login.  
  *Solution:* Identified that Swedavia provides a public educational test key for FlightInfo API v2 that can be passed in the `Ocp-Apim-Subscription-Key` request header. I also implemented an offline fallback generator (`_generate_mock_flights`) to guarantee the app never crashes even if the network drops.
- **Problem: Airport Identification Mismatch**  
  *Problem:* Users often type full city names (e.g., "Arlanda" or "Visby") instead of three-letter IATA codes (`ARN`, `VBY`).  
  *Solution:* Built an internal dictionary (`_AIRPORT_ALIASES`) that automatically translates common Swedish city names and lowercase inputs into valid IATA airport codes.

---

## 🛠 Phase 2: Implementation (Development)

### Project Structure
The application was built across three core files:
1. **`city_country.json`:** An offline reference database mapping 60+ global flight destinations to their respective countries.
2. **`destinationer.py`:** A helper script that calculates flight frequencies and filters routes by city or country.
3. **`airport.py`:** The primary application containing the interactive loop, input validation, API caller, and pagination display.

### Key Features Developed
- **Interactive Main Menu:** A central `while True` loop presenting 6 numbered options, exiting cleanly on user input `q`.
- **Flexible Input Validation:** Reusable loops (`_choose_airport()` and `_choose_date()`) that reject invalid inputs with warning messages and accept human-friendly date keywords (`idag`, `imorgon`, `igår`, `nu`).
- **Timezone Normalization:** A conversion function (`_format_utc_to_cet()`) translating raw UTC timestamps into Swedish local time (CET, UTC+1).
- **Interactive Paged Output:** A pagination engine displaying 50 flights at a time (`-- Visar 50/120 --`), prompting the user with `[Enter]` for the next page, `[a]` for all flights, and `[q]` to return to the menu.

### Implementation Problems & Solutions
- **Problem 1: Rapid Loop Re-rendering (Flashing Menu)**  
  *Problem:* After selecting an option, the script printed output and instantly re-rendered the main menu, causing text to scroll off the screen unread.  
  *Solution:* Inserted structured `input("\nTryck [Enter] för att gå tillbaka...")` pauses after each menu execution block.
- **Problem 2: Python Command in the VS Code Terminal**  
  *Problem:* Running `python3 airport.py` in the VS Code Integrated Terminal triggered a system error stating that Python was not found.  
  *Solution:* Identified that the Windows environment uses the `py` launcher command instead of `python3`, running the application seamlessly with `py airport.py`.
- **Problem 3: Missing Module Entrypoint (`AttributeError`)**  
  *Problem:* Selecting Option 4 threw `AttributeError: module 'destinationer' has no attribute 'main'`.  
  *Solution:* Refactored `destinationer.py` to introduce an explicit `def main():` function so it can run both as an imported helper and as an independent script.
- **Problem 4: Swedish vs. English Country Name Mismatch**  
  *Problem:* Entering Swedish country names (such as "Spanien") in the destination filter returned zero results because the database (`city_country.json`) uses English country names ("Spain").  
  *Solution:* Implemented a translation dictionary (`SWE_TO_ENG`) inside `destinationer.py` that automatically converts Swedish country names into English before querying the data.

---

## ✅ Phase 3: Finalization & Testing

### Testing & Optimization
- **Functional Testing:** Systematically verified all 6 menu options (Arrivals, Departures, Flight Search, Destination Analytics, API Health Check, and Auto-Demo).
- **Pagination Stress Test:** Tested high-volume airports (Stockholm Arlanda) to verify that lists exceeding 100+ flights page smoothly in chunks of 50 without index errors.
- **Local Verification:** Verified that the entire project executes locally inside the VS Code Integrated Terminal with zero external pip dependencies needed.

### Finalization Problems & Solutions
- **Problem: SSH Authentication Failure During Initial Git Push**  
  *Problem:* Pushing the project from the VS Code terminal via SSH failed with `Permission denied (publickey)`.  
  *Solution:* Updated the remote repository URL to HTTPS using `git remote set-url origin https://...`, allowing seamless authentication via the Windows browser.

---

## ⚠️ Phase 4: Summary of Problems & Solutions (Compilation)

| Identified Problem | Root Cause | Implemented Solution |
| :--- | :--- | :--- |
| **Flashing Terminal Output** | Main loop recycled instantly without pauses | Added `input()` confirmation prompts after output |
| **Command Not Found (`python3`)** | The VS Code terminal uses a different launcher command | Executed with `py airport.py` instead |
| **Missing Module Attribute** | `destinationer.py` lacked an entrypoint function | Implemented `def main():` in `destinationer.py` |
| **Language Filter Mismatch** | User typed Swedish names ("Spanien") while DB is in English | Added `SWE_TO_ENG` translation dictionary in `destinationer.py` |
| **Git Push Permission Denied** | Terminal lacked an active SSH agent key | Switched remote to HTTPS using Git Credential Manager |
| **Timezone Ambiguity** | API sends raw UTC format strings | Built `_format_utc_to_cet()` to calculate Swedish CET time |

---

## 📜 Conclusion
Through this project, I successfully reverse-engineered and reconstructed a fully functioning Swedavia flight tracker using visual clues and logic analysis. The final application is robust, easy to navigate, and faithfully reproduces the behavior and output of the original system.
