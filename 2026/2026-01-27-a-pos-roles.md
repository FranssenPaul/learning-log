# POS Role Model – Clear Synthesis

This document defines the **role system** for the POS application. It is designed to be **simple, secure, offline‑friendly**, and aligned with real restaurant operations.

---

## Core Principles

- **One role per user** (no role stacking)
- **Hierarchical roles with inheritance**
- **Permissions enforced on the backend** (never trust UI only)
- **Clear separation of responsibilities**
- **Never allow loss of ultimate control (owner lockout)**

---

## Role Hierarchy

```text
owner
 └── manager
      ├── cashier
      ├── waiter
      ├── kitchen
      └── bar
```

- Higher roles **inherit permissions** of lower roles
- Lower roles **cannot escalate** privileges

---

## Roles Definition

### Owner (Boss)
**Purpose:** Ultimate authority over the system

**Can do:**
- Add / remove **owners**
- Add / remove **managers**
- Access all system features
- Emergency recovery (PIN reset, access restore)

**Cannot:**
- Be removed if they are the **last owner**

**Notes:**
- There can be **multiple owners**
- The system must **never allow zero owners**
- Owner creation happens at **first boot / installation** or via secure local access

**Example:**
> Two business partners both have the `owner` role. Either can revoke a manager who leaves the company.

---

### Manager
**Purpose:** Day‑to‑day administration and operations

**Can do:**
- Add / remove **workers** (cashier, waiter, kitchen, bar)
- Assign worker roles
- Manage products, prices, taxes
- View reports
- Perform **all worker actions**
  - take orders
  - handle payments
  - mark items ready

**Cannot:**
- Add or remove **owners**
- Promote themselves to owner

**Example:**
> A manager works the cashier during rush hour, then later adds a new waiter for the evening shift.

---

## Worker Roles

Workers have **operational access only**. They never manage users or system settings.

### Waiter
- Open orders
- Add / modify order lines
- Send items to kitchen / bar
- View ready items

**Example:**
> A waiter opens a table, adds drinks and food, sends them, and picks up ready dishes.

---

### Cashier
- View orders marked `to_pay`
- Process payments
- Close tickets

**Example:**
> A cashier closes an order, prints the receipt, and marks it as paid.

---

### Kitchen
- View food items in `sent` state
- Mark items as `ready`
- Dismiss canceled items

**Example:**
> Kitchen staff sees a sandwich order, prepares it, and marks it ready.

---

### Bar
- View drink items in `sent` state
- Mark drinks as `ready`
- Dismiss canceled items

**Why `bar` and not `barman`:**
- Gender‑neutral
- Industry standard
- Scales to multiple bartenders

**Example:**
> Bar staff prepares drinks and marks them ready for pickup.

---

## Permission Inheritance (Key Rule)

> **A manager can do everything a worker can do, but not the opposite.**

Examples:
- Manager accessing cashier screen → ✅ allowed
- Cashier accessing manager settings → ❌ denied
- Waiter managing users → ❌ denied

---

## Backend Enforcement (Conceptual)

```python
ROLE_RANK = {
    "bar": 1,
    "kitchen": 1,
    "waiter": 1,
    "cashier": 1,
    "manager": 2,
    "owner": 3,
}


def has_permission(user_role: str, required_role: str) -> bool:
    return ROLE_RANK[user_role] >= ROLE_RANK[required_role]
```

- All sensitive actions must check permissions server‑side
- UI visibility follows the same rule, but is **not authoritative**

---

## Design Benefits

- Simple database model (`role` enum)
- Clear audit logs
- No ambiguous “multi‑role” users
- Matches real restaurant hierarchy
- Works well in LAN / offline environments

---

## Final Summary

- **Owner** manages owners and managers
- **Manager** manages workers and daily operations
- **Workers** perform operational tasks only
- **One role per user**
- **Managers inherit all worker permissions**
- **Never allow zero owners**

This role model is stable, scalable, and production‑ready for your POS system.

