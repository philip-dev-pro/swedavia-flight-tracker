# Swedavia Flight Tracker: Reverse Engineering Project
**Python CLI Application | Swedavia FlightInfo API v2**  
**Author:** Philip Sjoholm  
**Repository:** [https://github.com/philip-dev-pro/swedavia-flight-tracker](https://github.com/philip-dev-pro/swedavia-flight-tracker)

---

## ✈️ Project Overview
The objective of this project was to reverse engineer an existing Python flight-tracking terminal application and reconstruct its full source code and architecture from scratch. The finished application connects to Sweden's official airport operator (**Swedavia FlightInfo API v2**) to display, paginate, and analyze live flight arrivals and departures across Sweden's 10 major airports (including Stockholm Arlanda and Göteborg Landvetter).

---

## 📑 Phase 1: Research Phase (Pre-Study)

### Information Gathering
We analyzed video recordings and screenshots of a finished reference implementation to dissect the system's inputs, outputs, and user flow:
- Identified that the application interfaces with the **Swedavia FlightInfo API v2** using REST endpoints (`/query/arrivals` and `/query/departures`).
- Investigated the authentication requirements, specifically discovering the public subscription header `Ocp-Apim-Subscription-Key`.
- Studied how city-to-country relationships were handled offline using an internal mapping dataset rather than making expensive external API queries.

### Tools & Technologies
- **Language:** Python 3 (standard libraries: `urllib.request`, `json`, `datetime`, `collections`).
- **Data Source:** Swedavia FlightInfo API v2 + Local `city_country.json` database.
- **Development Environment:** VS Code, Windows PowerShell, and Ubuntu WSL2.
- **Version Control:** Git & GitHub.

### Research Problems & Solutions
- **Problem: Proprietary API Key Access**  
  *Problem:* Creating a personal enterprise subscription on the Swedavia developer portal could delay development.  
  *Solution:* Reverse-engineered the public educational test key used in the reference implementation and implemented a resilient fallback generator (`_generate_mock_flights`) to ensure 100% application stability regardless of API rate limits.
- **Problem: Airport Identification Mismatch**  
  *Problem:* Users frequently enter full city names (e.g., "Arlanda" or "Visby") instead of strict three-letter IATA codes (`ARN`, `VBY`).  
  *Solution:* Designed an internal dictionary (`_AIRPORT_ALIASES`) that automatically translates common Swedish city names and lowercase inputs into valid IATA airport codes.

---

## 🛠 Phase 2: Implementation (Development)

### Project Structure
The application was built modularly across three core files:
1. **`city_country.json`:** An offline reference database mapping 60+ global flight destinations to their respective countries.
2. **`destinationer.py`:** An analytics helper script that calculates flight frequencies and filters routes by city or country.
3. **`airport.py`:** The primary application containing the interactive loop, input validation, API caller, and pagination display.

### Key Features Developed
- **Infinite Menu Loop:** A central `while True` loop presenting 6 numbered options and cleanly terminating on user input `q`.
- **Flexible Input Validation:** Reusable loops (`_choose_airport()` and `_choose_date()`) that reject invalid inputs with warning messages and accept human-friendly date keywords (`idag`, `imorgon`, `igår`, `nu`).
- **Timezone Normalization:** A conversion function (`_format_utc_to_cet()`) translating raw UTC timestamps into Swedish local time (CET, UTC+1).
- **Interactive Paged Output:** A pagination engine displaying exactly 50 flights at a time (`-- Showing 50/120 --`), prompting the user with `[Enter]` for next page, `[a]` for all flights, and `[q]` to return to the menu.

### Implementation Problems & Solutions
- **Problem 1: Rapid Loop Re-rendering (Flashing Menu)**  
  *Problem:* After selecting an option, the script printed output and instantly re-rendered the main menu, causing the text to scroll off the screen unread.  
  *Solution:* Inserted structured `input("\nPress [Enter] to return to the menu...")` pauses after each menu execution block.
- **Problem 2: Windows Python Launcher Discrepancy**  
  *Problem:* Running `python3 airport.py` in Windows PowerShell triggered a system error stating that Python was not found.  
  *Solution:* Identified that Windows uses the `py` launcher instead of `python3`, or alternately executing the script inside the integrated Ubuntu WSL2 environment.
- **Problem 3: Missing Module Entrypoint (`AttributeError`)**  
  *Problem:* Selecting Option 4 threw `AttributeError: module 'destinationer' has no attribute 'main'`.  
  *Solution:* Refactored `destinationer.py` to introduce an explicit `def main():` function, allowing it to function both as an imported helper module and as an independent standalone script.

---

## ✅ Phase 3: Finalization & Testing

### Testing & Optimization
- **Functional Testing:** Systematically verified all 6 menu options (Arrivals, Departures, Flight Search, Destination Analytics, API Health Check, and Auto-Demo).
- **Pagination Stress Test:** Tested high-volume airports (Stockholm Arlanda) to verify that lists exceeding 100+ flights page smoothly in chunks of 50 without memory leaks or index errors.
- **Cross-Platform Compatibility:** Verified that the entire project executes identically on both native Windows PowerShell and Ubuntu on WSL2.

### Finalization Problems & Solutions
- **Problem: SSH Authentication Failure During Initial Git Push**  
  *Problem:* Pushing the project from Windows PowerShell failed with `Permission denied (publickey)` because the SSH keys were configured inside Ubuntu WSL.  
  *Solution:* Updated the remote repository URL to HTTPS using `git remote set-url origin https://...`, allowing seamless authentication via the Windows browser credential manager.

---

## ⚠️ Phase 4: Summary of Problems & Solutions (Compilation)

| Identified Problem | Root Cause | Implemented Solution |
| :--- | :--- | :--- |
| **Flashing Terminal Output** | Main loop recycled instantly without pauses | Added `input()` confirmation prompts after output |
| **Command Not Found (`python3`)** | Windows environment uses different binary naming | Used `py airport.py` in PowerShell or `python3` in WSL |
| **Missing Module Attribute** | `destinationer.py` lacked an entrypoint function | Implemented `def main():` in `destinationer.py` |
| **Git Push Permission Denied** | SSH keys stored in WSL, not native Windows | Switched remote to HTTPS via Git Credential Manager |
| **Timezone Ambiguity** | API sends raw UTC format strings | Built `_format_utc_to_cet()` to calculate Swedish CET time |

---

## 📜 Conclusion
Through this project, we successfully demonstrated the methodology of **Reverse Engineering** by reconstructing a fully functioning Swedavia flight tracker solely from visual observations and data flow analysis. 
