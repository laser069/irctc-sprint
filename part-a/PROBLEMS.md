# IRCTC Tatkal Booking System – Technical Problem Report

## Problem 1: Tatkal Booking System Failures

### 1. What Is Broken?

The IRCTC Tatkal booking process suffers from several critical technical bottlenecks during peak booking hours:

* **Server Capacity Issues:** Millions of users attempting to log in simultaneously cause the servers to lag, crash, or refuse connections.
* **Endless Buffering:** The application interface often freezes during seat selection or passenger data entry.
* **Sudden Logouts:** Users are frequently logged out during sensitive stages such as payment processing.
* **Transaction Failures:** Users experience the “Failed Payment, No Ticket” issue where money is debited but booking confirmation fails due to session timeouts.

### 2. Who Is Affected?

* **Last-Minute Travelers:** Users requiring urgent travel often face “Regret” status within minutes.
* **General Public:** Users encounter failed transactions and “Not Found” errors even when seats were initially available.

### 3. Frequency

* **Highly Frequent:** Failures occur daily during the peak demand windows:

  * **10:00 AM** – AC Classes
  * **11:00 AM** – Sleeper Class

### 4. Where Exactly It Breaks

#### Critical Time Window (10:00 AM – 10:05 AM)

* Login requests frequently fail.
* Server error pages appear.
* Session timeouts increase sharply.

#### Technical Root Cause

* Massive concurrent traffic pushes the server infrastructure to maximum capacity.
* Database deadlocks and session drops occur under heavy load.

---

## Current User Flow (Minimum 7 Steps)

| Step  | Action                                              | Pro Tips                                                                       |
| ----- | --------------------------------------------------- | ------------------------------------------------------------------------------ |
| **1** | **User opens IRCTC at 9:50 AM**                     | User prepares for Tatkal booking before the booking window opens.              |
| **2** | **User logs into account**                          | User authenticates and waits for the Tatkal quota window.                      |
| **3** | **User searches trains with Tatkal quota selected** | User enters source, destination, and journey date.                             |
| **4** | **Booking window opens at 10:00 AM**                | User clicks “Book Now” immediately as Tatkal quota becomes available.          |
| **5** | **Passenger details page loads slowly**             | User attempts to enter or auto-fill passenger details.                         |
| **6** | **Session instability begins**                      | Application buffers, logs users out, or freezes during Captcha/payment stages. |
| **7** | **Tatkal quota disappears**                         | User receives WL/Regret status or failed transaction after repeated retries.   |

---

# IRCTC Technical Analysis: Search & Filter Instability

## Problem 2: Search Filter & State Persistence Issues

### 1. What Is Broken?

* **Filter Logic Desynchronization:** Applying multiple filters (e.g., “Sleeper” + “Available Only”) often fails to trigger a proper DOM update.
* **State Reset on Navigation:** Filters reset when the user navigates back to the results page.
* **Persistent Cache Lag:** The “Available Only” filter frequently shows trains with **WL** or **Regret** status due to stale cached data.
* **Filter Conflict:** Selecting one filter may automatically deselect previously chosen filters.

### 2. Who Is Affected?

* **Power Users:** Travelers filtering through large route lists such as Delhi–Mumbai or Bangalore–Chennai.
* **Mobile Users:** Users on the RailConnect app who lose filter state during accidental navigation.

### 3. Frequency

* **Consistent:** This issue occurs regularly and is not limited to peak booking hours.
* The instability is tied to architectural flaws in search-state management.

---

## Current User Flow (Minimum 7 Steps)

1. **User opens train search** – User searches between two major cities.
2. **User applies Sleeper filter** – User selects Sleeper (SL) class.
3. **User applies Available Only filter** – User expects waitlisted trains to disappear.
4. **User applies Morning Departure filter** – User narrows trains to morning timings.
5. **Results fail to refresh properly** – Train list often remains unchanged or partially updated.
6. **User opens train details and navigates back** – User expects filters to persist.
7. **Filters reset completely** – User loses selected filters and must start filtering again.

---

## Where Exactly It Breaks

* Failure primarily occurs during the transition between passenger details and payment processing.
* At peak load, concurrent requests overload backend services.
* Sessions expire before payment confirmation completes.
* Users receive no meaningful feedback such as queue position, retry status, or recovery instructions.

### Why It Breaks

### URL Parameter Mapping Failure

* IRCTC does not use dynamic URL query parameters such as:

```txt
?class=SL&dep=morning
```

* Filter state is stored only in temporary session variables.
* Refreshing or navigating destroys the filtered state.

### API Race Conditions

* Multiple rapid filter selections trigger asynchronous API calls.
* Responses return out of order, causing UI and backend data mismatches.

---

# IRCTC Technical Analysis: Seat Selection Logic

## Problem 3: Seat Selection & Berth Preference Resets

### 1. What Is Broken?

* **Visual–Data Desynchronization:** Seat selections on the graphical map fail to persist in backend passenger objects.
* **Mobile UI Override:** Manual berth selection often resets to **No Preference** after proceeding.
* **Redundant Selection Requirement:** Users are forced to reselect berth preferences during final review.
* **Validation Conflict:** “Book only if lower berth is allotted” conflicts with live seat availability updates.

### 2. Who Is Affected?

* **Senior Citizens & PWDs:** Users who specifically require lower berths.
* **Group Travelers:** Users attempting to reserve adjacent seats.

### 3. Frequency

* **Critical on Mobile Devices:**

  * Reset rates are significantly higher on the RailConnect mobile application.
  * Estimated reset rate during peak traffic: **60–70%**.

---

## Current User Flow (Minimum 7 Steps)

1. **User selects a train** – User proceeds to booking for a train with available seats.
2. **Seat map is opened** – User clicks Select Seats or View Seat Map.
3. **User selects a Lower Berth** – UI visually confirms berth selection.
4. **User clicks Proceed** – System transitions to passenger review page.
5. **Passenger details page loads** – User checks berth preference section.
6. **Observe Reset:** Verify whether it reverted to **No Preference**.
7. **Cross-Platform Test:** Repeat the process on the RailConnect app.

---

## Where Exactly It Breaks

### POST Request Payload Failure

* The issue occurs during transfer from the seat map modal to the passenger form.
* The **Seat ID** attribute is often dropped from the JSON payload.

### Session Latency on Mobile

* Slow seat-map refreshes trigger timeout handling.
* The application defaults to **No Preference** to avoid hanging states.
