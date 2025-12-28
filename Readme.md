# Widget Configuration Platform
**AI Implementation README**

---

## 1. Purpose of the Project

This project implements a **multi-tenant widget configuration platform** that allows lightweight frontend widgets (e.g. calculators, forms, visual components) to be embedded into third-party platforms with **fully declarative configuration**.

The system must support:
- configurable widget parameters via `data-*` attributes or JSON,
- multi-level configuration inheritance,
- tenant- and admin-level restrictions,
- widget versioning,
- deterministic resolution of final widget configuration.

This README is written as a **technical specification for an AI agent** that will implement the system.

---

## 2. Widget Concept

A **widget** is:
- a standalone frontend component (e.g. Vue/JS),
- embedded via `<script>` or `<iframe>`,
- configured only through parameters (no runtime user-specific backend calls).

### Example embed output (final system result)

```html
<div
  data-widget="bav-calculator"
  data-variant="standard"
  data-alter="30"
  data-nettobeitrag="100"
  data-bruttoeinkommenmonatlich="4300"
  data-open-area-chart="false"
  data-zuschuss-prozent="9">
</div>
```

The platform’s job is to **generate and validate these parameters**.

---

## 3. Core Architecture Principles

### 3.1 Declarative Configuration
Widgets do not contain tenant-specific logic.  
All behavior is driven by configuration.

### 3.2 Central Registry
All configuration parameters are declared centrally in a **Registry**.  
Profiles and configs may only override or restrict existing parameters.

### 3.3 Multi-Level Inheritance
Configurations inherit values from parent profiles/configs with controlled overrides.

---

## 4. Configuration Hierarchy

Configuration is resolved top-down:

```
Registry (global definitions)
   ↓
Base Tenant Profile (admin-defined)
   ↓
Tenant Profile (tenant-defined)
   ↓
Tenant Configurations (N, optional inheritance)
```

---

## 5. Configuration Levels Explained

### 5.1 Registry (Global Parameter Definitions)
- Global, system-level
- Defines **what parameters exist**
- Defines types, defaults, options
- No tenant-specific data

### 5.2 Base Tenant Profile
- Created automatically when a tenant is registered
- Depends on:
    - subscription
    - widget version access
- Admin-controlled
- Can:
    - set default values
    - lock parameters (read-only)
    - hide parameters

### 5.3 Tenant Profile
- Main editable profile of a tenant
- Inherits Base Tenant Profile
- Tenant may:
    - override allowed defaults
    - restrict option sets
- Cannot:
    - unlock locked parameters
    - add new parameters

### 5.4 Tenant Configurations
- Arbitrary number per tenant
- Used for:
    - different websites
    - partners
    - A/B testing
- Can inherit from:
    - Tenant Profile
    - another Tenant Configuration

---

## 6. Inheritance Rules

**Rule:**  
Child configuration overrides parent values **only where explicitly defined**.

### Example

Parent config:
```json
{
  "alter": 30,
  "openAreaChart": false
}
```

Child config:
```json
{
  "alter": 35
}
```

Resolved result:
```json
{
  "alter": 35,
  "openAreaChart": false
}
```

---

## 7. Registry Model (Parameter Definition)

### ConfigurationParameterDefinition

```yaml
id: string
widgetKey: string
widgetVersion: string | null

parameterType: INPUT | CONFIG

valueType:
  # INPUT
  - integer
  - decimal
  - currency
  - enum
  - boolean

  # CONFIG
  - boolean
  - color
  - ui-flag
  - ui-variant

defaultValue: any

options: optional
  - list of allowed values (enum only)

description: string
```

---

## 8. Profile / Configuration Model

### WidgetProfile

```yaml
profileId: string
tenantId: string
widgetKey: string
widgetVersion: string

parentProfileId: string | null

parameters:
  parameterKey:
    value: any
    visible: boolean
    editable: boolean
    allowedOptions: optional list
```

---

## 9. Editing Rules & Constraints

- Values can only be changed if editable
- Options can only be restricted
- Visibility applies only to input fields

---

## 10. Widget Versioning

- All parameters are bound to widgetKey and widgetVersion
- Profiles are version-bound

---

## 11. Configuration Resolution

The system must provide:

```
resolveConfiguration(configId) → FinalConfig
```

---

## 12. Invariants

- Registry is the single source of truth
- Inheritance is deterministic
- Invalid overrides are rejected

---

## 13. AI Agent Expectations

The AI agent should implement:
1. Domain models
2. Inheritance engine
3. Validation logic
4. Configuration resolution API
