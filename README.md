

# STX-LegacyVault

## 📘 Overview

The **Intergenerational Wealth Trust** is a **Clarity smart contract** designed to manage and distribute digital assets (STX) across generations based on pre-defined **milestones, vesting schedules, and guardian approvals**.

This contract emphasizes **security, transparency, and responsible asset management** through a structured set of validation rules, error handling, and guardian oversight.

---

## ⚙️ Core Features

### 👤 Heir Management

* **Add heirs** with validated birth heights and allocations.
* **Assign guardians** to heirs for additional oversight.
* **Track education bonuses** and vesting timelines.
* Prevent self-guardianship and double-initialization.

### 🎯 Milestone Tracking

* Define milestones tied to **age requirements**, **deadlines**, and **bonuses**.
* Each milestone includes:

  * A human-readable description.
  * A reward amount (in STX).
  * An optional deadline.
  * A multiplier for bonus rewards.
  * A guardian approval requirement.

### 🛡️ Guardian Approvals

* Guardians can **approve or deny milestone claims** for minors or dependents.
* The system tracks guardian actions and timestamps approvals for transparency.

### 💰 Secure Transfers

* Uses a `safe-transfer` function that ensures STX transfers succeed, reverting if they fail.
* Prevents double-claims or unauthorized distributions.

### 🕰️ Vesting & Validation

* Ensures heirs meet **minimum vesting periods** before withdrawals.
* Validates **age requirements**, **status**, and **allocation size**.
* Detects invalid or expired deadlines before allowing milestone claims.

---

## 📑 Data Structures

### 🔹 Global Variables

| Variable                 | Type        | Description                                            |
| ------------------------ | ----------- | ------------------------------------------------------ |
| `contract-owner`         | `principal` | The deploying address with full authority.             |
| `active`                 | `bool`      | Indicates if the trust is currently operational.       |
| `emergency-contact`      | `principal` | Authorized contact for emergencies.                    |
| `minimum-vesting-period` | `uint`      | Minimum required blocks for vesting (default ~1 year). |

---

### 🔹 Heirs Map

Stores all registered heirs and their trust data.

| Field              | Type                   | Description                                                |
| ------------------ | ---------------------- | ---------------------------------------------------------- |
| `birth-height`     | `uint`                 | Block height of birth (approximation for age calculation). |
| `total-allocation` | `uint`                 | Total STX allocated to the heir.                           |
| `claimed-amount`   | `uint`                 | Amount of STX already claimed.                             |
| `status`           | `(string-ascii 9)`     | Status of the heir (`active`, `paused`, `completed`).      |
| `guardian`         | `(optional principal)` | Guardian address for minors.                               |
| `vesting-start`    | `uint`                 | Block height when vesting began.                           |
| `education-bonus`  | `uint`                 | STX bonus for educational achievements.                    |
| `last-activity`    | `uint`                 | Last recorded interaction block height.                    |

---

### 🔹 Milestones Map

Defines achievement goals for heirs to unlock rewards.

| Field               | Type                 | Description                                                |
| ------------------- | -------------------- | ---------------------------------------------------------- |
| `description`       | `(string-ascii 100)` | Description of milestone goal.                             |
| `reward-amount`     | `uint`               | STX reward for achieving the milestone.                    |
| `age-requirement`   | `uint`               | Minimum age in years to claim milestone.                   |
| `completed`         | `bool`               | Indicates if the milestone is already claimed.             |
| `deadline`          | `(optional uint)`    | Optional block height by which milestone must be achieved. |
| `bonus-multiplier`  | `uint`               | Multiplier applied to reward (default 100 = 1x).           |
| `requires-guardian` | `bool`               | Whether guardian approval is needed.                       |

---

### 🔹 Guardian Approvals Map

| Field       | Type   | Description                              |
| ----------- | ------ | ---------------------------------------- |
| `approved`  | `bool` | Whether guardian approved the milestone. |
| `timestamp` | `uint` | Block height of the approval.            |

---

## 🧩 Public Functions

### 👶 `add-heir`

Registers a new heir in the trust.

```clarity
(add-heir heir principal birth-height uint allocation uint guardian (optional principal))
```

**Validations:**

* Only the contract owner can add heirs.
* Birth height must be ≤ current block height.
* Allocation ≥ 1 STX and ≤ sender’s balance.
* Guardian cannot be the heir themselves.

---

### 🧔 `update-guardian`

Updates or sets a guardian for an heir.

```clarity
(update-guardian heir principal new-guardian principal)
```

**Rules:**

* Only the current guardian or contract owner can update.
* Heir cannot be their own guardian.

---

### 🎓 `add-education-bonus`

Adds an education bonus to an heir’s account.

```clarity
(add-education-bonus heir principal bonus-amount uint)
```

**Only the contract owner** can execute this.
Amount must be greater than zero.

---

### 🏁 `add-milestone`

Defines a new milestone goal.

```clarity
(add-milestone milestone-id uint description (string-ascii 100)
 reward-amount uint age-requirement uint deadline (optional uint)
 bonus-multiplier uint requires-guardian bool)
```

**Validations:**

* Must be created by contract owner.
* Age requirement within bounds (16–100 years).
* Bonus multiplier between 1x–5x (100–500).
* Reward amount > 0.
* Optional deadline must be in the future.

---

## 🔍 Read-Only Functions

| Function                                    | Description                                |
| ------------------------------------------- | ------------------------------------------ |
| `get-heir-info(heir)`                       | Returns heir details if registered.        |
| `get-milestone(milestone-id)`               | Returns milestone data.                    |
| `calculate-age(birth-height)`               | Returns current age estimate in years.     |
| `get-guardian-approval(heir, milestone-id)` | Returns guardian approval status.          |
| `get-vesting-status(heir)`                  | Checks if heir met minimum vesting period. |

---

## ⚠️ Error Codes

| Code   | Meaning                           |
| ------ | --------------------------------- |
| `u100` | Not authorized                    |
| `u101` | Already initialized               |
| `u102` | Contract not active               |
| `u103` | Invalid age requirement           |
| `u104` | Milestone not found               |
| `u105` | Milestone already completed       |
| `u106` | Insufficient balance              |
| `u107` | Invalid milestone                 |
| `u108` | Invalid time/deadline             |
| `u109` | Guardian already set              |
| `u110` | No guardian assigned              |
| `u111` | Invalid amount                    |
| `u112` | Invalid birth height              |
| `u113` | Zero allocation not allowed       |
| `u114` | Invalid bonus multiplier          |
| `u115` | Invalid deadline                  |
| `u116` | Invalid status                    |
| `u117` | Heir cannot be their own guardian |
| `u118` | STX transfer failed               |

---

## 🔐 Security Highlights

* **Owner-only control** for sensitive operations.
* **Full validation** of ages, bonuses, allocations, and deadlines.
* **Guardian protections** for underage heirs.
* **Fail-safe transfers** prevent loss of STX.
* **Extensive error handling** for predictable behavior.

---

## 🚀 Future Extensions

* Add `claim-milestone` logic for reward distribution.
* Implement `pause` / `resume` contract functionality.
* Support `NFTs` or `SIP-010 tokens` as assets.
* Add `emergency-recover` mechanism for heirs or guardians.

---

## 🧠 License

MIT License – open for educational, philanthropic, or financial management use cases.

---
