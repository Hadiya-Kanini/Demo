# Design System

## 1. Design Tokens

Design tokens are the visual atoms of our design system. They represent stylistic choices, such as color, typography, and spacing, in a platform-agnostic way.

### 1.1. Colors

| Token Name          | Value          | Description                                    | Usage                                         |
| :------------------ | :------------- | :--------------------------------------------- | :-------------------------------------------- |
| `color-primary`     | `#007BFF`      | Main brand color, for primary actions.         | Primary buttons, active states                |
| `color-primary-dark`| `#0056B3`      | Darker primary for hover/pressed states.       | Primary button hover, active links            |
| `color-secondary`   | `#6C757D`      | Secondary brand color, for less prominent actions. | Secondary buttons, secondary text             |
| `color-text-primary`| `#212529`      | Primary text color for readability.            | Headings, body text                           |
| `color-text-secondary`| `#6C757D`      | Secondary text color for less emphasis.        | Subheaders, helper text                       |
| `color-text-placeholder`| `#ADB5BD`      | Placeholder text color in input fields.        | Input field placeholders                      |
| `color-text-link`   | `#007BFF`      | Link text color.                               | Hyperlinks, "Forgot Password?"                |
| `color-background-page`| `#F8F9FA`      | Main background color for pages.               | Page background                               |
| `color-background-card`| `#FFFFFF`      | Background for cards and elevated elements.    | Login form container                          |
| `color-border-default`| `#CED4DA`      | Default border color for inputs.               | Input field borders                           |
| `color-border-focused`| `#80BDFF`      | Border color for focused inputs.               | Input field focus state                       |
| `color-error`       | `#DC3545`      | Base error color.                              | Error icons, critical alerts                  |
| `color-error-text`  | `#B02A37`      | Darker error text for better contrast.         | Error messages                                |
| `color-error-subtle`| `#F8D7DA`      | Light background for error alerts.             | Background of error message banners           |
| `color-success`     | `#28A745`      | Base success color.                            | Success messages (out of scope for login)     |
| `color-disabled`    | `#E9ECEF`      | Background for disabled elements.              | Disabled buttons, inputs                      |
| `color-disabled-text`| `#ADB5BD`      | Text color for disabled elements.              | Disabled button text, disabled input text     |

### 1.2. Typography

**Font Family:** `Inter`, `sans-serif` (or project's defined brand font)

| Token Name             | Font Size   | Line Height | Font Weight | Letter Spacing | Usage                                         |
| :--------------------- | :---------- | :---------- | :---------- | :------------- | :-------------------------------------------- |
| `typography-heading-h1`| 40px (2.5rem)| 1.2         | 700 (Bold)  | -0.02em        | Main page titles, "Welcome Back"              |
| `typography-heading-h2`| 32px (2rem) | 1.25        | 700 (Bold)  | -0.02em        | Section titles (not used on Login Page)       |
| `typography-body-lg`   | 18px (1.125rem)| 1.5         | 400 (Regular)| 0              | Subheaders, prominent body text               |
| `typography-body-md`   | 16px (1rem) | 1.5         | 400 (Regular)| 0              | Default body text, input field text           |
| `typography-body-sm`   | 14px (0.875rem)| 1.4         | 400 (Regular)| 0              | Labels, helper text, error messages           |
| `typography-label`     | 14px (0.875rem)| 1.4         | 600 (Semi-Bold)| 0.02em         | Form input labels                             |
| `typography-button-primary`| 16px (1rem) | 1.5         | 600 (Semi-Bold)| 0.02em         | Primary button text                           |
| `typography-link`      | 14px (0.875rem)| 1.4         | 500 (Medium) | 0              | Link text, "Forgot Password?"                 |

### 1.3. Spacing

A 4-pixel base unit for consistent spacing.

| Token Name       | Value | Description                  |
| :--------------- | :---- | :--------------------------- |
| `spacing-xs`     | 4px   | Extra small spacing          |
| `spacing-sm`     | 8px   | Small spacing                |
| `spacing-md`     | 16px  | Medium spacing (default)     |
| `spacing-lg`     | 24px  | Large spacing                |
| `spacing-xl`     | 32px  | Extra large spacing          |
| `spacing-xxl`    | 48px  | Double XL spacing            |
| `spacing-3xl`    | 64px  | Triple XL spacing            |

### 1.4. Shadows

| Token Name          | Value                                 | Description                        |
| :------------------ | :------------------------------------ | :--------------------------------- |
| `shadow-sm`         | `0 1px 2px rgba(0, 0, 0, 0.05)`       | Subtle shadow, for inputs.         |
| `shadow-md`         | `0 4px 6px rgba(0, 0, 0, 0.1)`        | Medium shadow, for cards.          |
| `shadow-lg`         | `0 10px 15px rgba(0, 0, 0, 0.1), 0 4px 6px rgba(0, 0, 0, 0.05)` | Larger shadow, for prominent elements. |

### 1.5. Borders & Radii

| Token Name          | Value          | Description                    |
| :------------------ | :------------- | :----------------------------- |
| `border-radius-sm`  | 4px            | Small radius, for inputs       |
| `border-radius-md`  | 8px            | Medium radius, for buttons     |
| `border-input-default`| `1px solid color-border-default` | Default input border style     |
| `border-input-focused`| `2px solid color-border-focused` | Focused input border style     |
| `border-input-error`| `1px solid color-error`          | Error input border style       |

### 1.6. Animations

| Token Name          | Value                  | Description                          |
| :------------------ | :--------------------- | :----------------------------------- |
| `animation-duration-fast`| `0.1s`                 | Quick transitions (hover, focus)     |
| `animation-duration-medium`| `0.2s`                 | Standard transitions (fade, slide)   |
| `animation-spinner-rotate`| `1s linear infinite`   | Rotation for loading spinners        |

---

## 2. Component Specifications

### 2.1. Input Field (`component-input-text`, `component-input-password`)

*   **Structure:** Label (`typography-label`), Input element, optional helper/error text (`typography-body-sm`).
*   **States:**
    *   **Default:** `background-card`, `border-input-default`, `color-text-primary`, `border-radius-sm`, `padding-md` horizontally, `padding-sm` vertically. Placeholder: `color-text-placeholder`.
    *   **Focused:** `border-input-focused`, `shadow-sm`.
    *   **Error:** `border-input-error`, error text `color-error-text` below the input.
    *   **Disabled:** `background-disabled`, `color-disabled-text`, `border-input-default`. Uninteractable.
*   **Password Input (`component-input-password` specific):** Includes an integrated icon button for toggling visibility.
    *   Icon: `icon-visibility-off` (masked), `icon-visibility` (visible).
    *   Size: `20px`, `color-text-secondary`.
    *   Interaction: On click, toggles `type` attribute and icon.

### 2.2. Button (`component-button-primary`)

*   **Structure:** Text (`typography-button-primary`), optional icon.
*   **States:**
    *   **Default:** `background-primary`, `color-button-primary-text` (white), `border-radius-md`, `padding-lg` horizontally, `padding-md` vertically. `shadow-sm`.
    *   **Hover:** `background-primary-dark`, `shadow-md`. `animation-duration-fast`.
    *   **Pressed:** `background-primary-dark`, slightly reduced `box-shadow` to simulate depth.
    *   **Disabled:** `background-disabled`, `color-disabled-text`, no interactive effects.
    *   **Loading:** `background-primary-dark`, `color-button-primary-text`, `icon-spinner` (spinning `animation-spinner-rotate`) replaces button text or is integrated. Disabled and unclickable.

### 2.3. Link (`component-link`)

*   **Structure:** Text (`typography-link`).
*   **States:**
    *   **Default:** `color-text-link`, `text-decoration: underline`.
    *   **Hover:** `color-primary-dark`, slightly bolder `text-decoration`. `animation-duration-fast`.
    *   **Focused:** Same as hover, with a distinct outline (e.g., `outline: 2px solid color-border-focused`).
    *   **Pressed:** `color-primary-darker`.

### 2.4. Global Error Message (`component-alert-error`)

*   **Structure:** Container with `icon-error` and error text (`typography-body-sm`).
*   **Appearance:** `background-error-subtle`, `border: 1px solid color-error`, `border-radius-md`, `padding-md`. Text `color-error-text`.
*   **Placement:** Prominently at the top of the form container.
*   **Dismissal:** Can be dismissed manually (if applicable) or on subsequent user interaction (e.g., typing in a field, attempting re-submission).

### 2.5. Status Indicator (`icon-spinner`)

*   **Usage:** Integrated into buttons for loading states.
*   **Appearance:** Typically a circular animation. `color-button-primary-text` when in a primary button.
*   **Animation:** `animation-spinner-rotate`.

---

## 3. Brand Guidelines

*   **Logo Usage:** The application logo (`brand-logo.svg`) MUST be prominently displayed on the login page, centered at the top. It should maintain its aspect ratio and have sufficient clear space around it.
*   **Brand Colors:** `color-primary` and `color-primary-dark` are core brand colors, used for primary actions and key brand elements.
*   **Brand Voice:** Secure, reliable, professional, and user-centric. Error messages should be helpful and not technical. Copy should be clear and concise.

---

## 4. Accessibility Standards (WCAG Compliance Details)

Our Design System components are built to meet WCAG 2.1 AA standards.

*   **Color Contrast:**
    *   All defined `color-text-*` tokens are tested against `color-background-*` and `color-error-subtle` backgrounds to ensure 4.5:1 contrast ratio.
    *   Interactive elements and graphical components (e.g., input borders, icons) use `color-border-default`, `color-border-focused`, `color-error`, `color-primary`, which are validated for 3:1 contrast against their respective backgrounds.
*   **Semantic HTML:** All components use appropriate semantic HTML5 tags (e.g., `<button>`, `<input>`, `<label>`, `<a>`) to convey meaning to assistive technologies.
*   **Keyboard Operability:**
    *   Focus states (`border-input-focused`, button outlines) are implemented for all interactive components.
    *   `role="button"` for custom button-like elements (e.g., password toggle icon if not a native button).
    *   `tabindex="0"` for custom interactive elements requiring focus.
*   **Screen Reader Support:**
    *   `aria-label` attributes are used for icon-only buttons (e.g., "Toggle password visibility").
    *   Input `label` elements are explicitly associated with their inputs using `for` and `id`.
    *   Error messages on input fields utilize `aria-describedby` to link the input to its validation message, and the input itself is marked with `aria-invalid="true"` when in error state.
    *   Global error alerts may use `role="alert"` or `aria-live="assertive"` to ensure immediate announcement to screen reader users.

---

## 5. Usage Guidelines and Examples

*   **Input Fields:**
    *   Always use a visible label; do not rely solely on placeholders.
    *   Provide clear, concise error messages directly beneath the affected input field for client-side validation.
    *   **Example (Email Input):**
        ```html
        <label for="email-input" class="typography-label">Email Address</label>
        <input type="email" id="email-input" placeholder="Enter your email" class="component-input-text" />
        <p class="typography-body-sm color-error-text">Email address is required</p>
        ```
*   **Buttons:**
    *   Use `component-button-primary` for the main action on a page (e.g., "Login").
    *   Ensure buttons have clear, descriptive text.
    *   Never use disabled buttons without clear indication of why they are disabled (e.g., accompanying error message or helper text explaining what needs to be filled).
*   **Error Messages:**
    *   **Inline:** Use for field-specific validation (e.g., empty fields, invalid format). Display immediately on client-side failure.
    *   **Global:** Use for server-side errors (e.g., invalid credentials, account locked, generic system errors). Display prominently at the top of the form.
    *   Wording: Follow UXR-001 - clear, actionable, non-technical. Do not expose backend error codes.
*   **Loading Indicators:**
    *   Whenever a user action triggers a server request, the interactive element (e.g., button) should show a loading state (UXR-005). This provides feedback and prevents double submissions.

These guidelines ensure consistency, usability, and accessibility across the application, especially for critical flows like Login & Authentication.

---