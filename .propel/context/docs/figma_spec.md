# Figma Design Specification

## 1. UI Impact Assessment

The following Functional Requirements (FR) and Use Cases (UC) from the Product Specification necessitate specific UI/UX design considerations for the Login & Authentication feature:

*   **FR-001: Display Login Form**
    *   **UI Impact:** Directly dictates the need for a dedicated login page layout, including a container for inputs, buttons, and links. Requires clear visual hierarchy for all form elements.
*   **FR-002: Email Address Input**
    *   **UI Impact:** Design of a standard text input field. Requires visual styling for its default, focused, error, and disabled states. Needs a placeholder text.
*   **FR-003: Password Input**
    *   **UI Impact:** Design of a password input field with character masking. Requires a "show/hide password" toggle for usability (UXR-003) and associated iconography. Visual styling for states similar to email input.
*   **FR-004: Client-Side Input Validation (Empty Fields)**
    *   **UI Impact:** Design for inline error message display immediately below the respective input fields. Error messages must be visually distinct (e.g., red text, error icon). This also impacts the input field's border/background color when in an error state.
*   **FR-005: Credential Submission**
    *   **UI Impact:** Design of the primary "Login" button and its various interactive states: default, hover, pressed, disabled (during client-side validation failure or submission), and a loading state (e.g., spinner within the button).
*   **FR-011: Role-Based Dashboard Redirection**
    *   **UI Impact:** While the dashboards are out of scope, the successful login flow visually concludes with a clear transition (e.g., brief loading, then a full page redirect). This confirms the end state of the login interaction.
*   **FR-013: Invalid Credentials Error Message**
    *   **UI Impact:** Design for a prominent, global error message display area on the login page, typically at the top or center. This message must be visually distinct and use consistent error styling.
*   **FR-014: Account Locked Error Message**
    *   **UI Impact:** Similar to FR-013, requires a prominent, global error message display area with specific text and error styling.
*   **FR-015: Generic System Error Message**
    *   **UI Impact:** Similar to FR-013/FR-014, for displaying unexpected server errors.
*   **FR-017: Forgot Password Redirection**
    *   **UI Impact:** Design of a clickable "Forgot Password?" link. Requires standard link styling (e.g., underlined text) and hover/focus states.

---

## 2. UX Requirements (UXR-XXX)

Derived from the Product Specification's NFRs (especially Usability & Security) and best practices.

**UXR-001: Clear and Actionable Error Messages**
*   **Description:** All error messages (client-side and server-side) shall be easily distinguishable, use simple, non-technical language, and guide the user on how to resolve the issue where applicable.
*   **Acceptance Criteria:**
    *   Error messages (FR-004, FR-013, FR-014, FR-015) MUST use `color-error-text` (see Design System).
    *   Error messages MUST be presented near the relevant input field for inline errors or at a prominent global location for server errors.
    *   Message text MUST adhere to the exact wording specified in FRs or defined in this spec.
    *   Message text MUST be legible (minimum `font-size-body-sm` and `font-weight-regular`).
*   **Reference:** NFR-USAB-001

**UXR-002: Intuitive Form Design**
*   **Description:** The login form shall have clear, descriptive labels for all input fields, ensure a logical tab order, and prominently display the primary "Login" action.
*   **Acceptance Criteria:**
    *   Input fields MUST have persistent, visible labels (e.g., "Email Address", "Password") not just placeholders.
    *   The tab order MUST flow logically from "Email Address" to "Password" to "Login" button to "Forgot Password?" link.
    *   The "Login" button MUST be a primary button style, distinct from other elements.
*   **Reference:** NFR-USAB-002

**UXR-003: Password Visibility Toggle**
*   **Description:** The password input field shall include an option (e.g., an eye icon) to toggle password visibility to aid in accurate input.
*   **Acceptance Criteria:**
    *   An interactive icon (e.g., `icon-visibility` / `icon-visibility-off`) MUST be present within or adjacent to the password input field.
    *   Clicking the icon MUST toggle the input type between `password` and `text`.
    *   The icon MUST visually reflect its current state (e.g., open eye for visible, crossed-out eye for masked).
    *   This toggle MUST be keyboard-operable.
*   **Reference:** FR-003 (enhancement for usability)

**UXR-004: Immediate Client-Side Validation Feedback**
*   **Description:** Client-side validation errors (FR-004) shall be displayed immediately and inline with the relevant input field upon attempted submission, without requiring a full page refresh.
*   **Acceptance Criteria:**
    *   Validation messages for empty fields MUST appear on blur or attempted form submission.
    *   The input field itself (e.g., border color) MUST change to an error state.
    *   The error message MUST disappear when the user starts typing valid input into the field.
*   **Reference:** FR-004

**UXR-005: Loading Indicator for Submission**
*   **Description:** Upon clicking the "Login" button (FR-005), a clear visual indicator shall be displayed to inform the user that the request is being processed.
*   **Acceptance Criteria:**
    *   The "Login" button MUST change to a loading state (e.g., display `icon-spinner` and disable clicks) immediately after being clicked and before a server response.
    *   The loading state MUST persist until a response (success or error) is received from the server.
*   **Reference:** NFR-PERF-001 (impacts perceived performance)

**UXR-006: Accessible Interactions**
*   **Description:** All interactive elements (input fields, buttons, links) shall be fully navigable and operable using a keyboard, and provide appropriate semantic information for screen readers.
*   **Acceptance Criteria:**
    *   All interactive elements MUST have a visible focus state when navigated via keyboard.
    *   Semantic HTML5 elements (e.g., `<form>`, `<input>`, `<button>`, `<a>`) MUST be used correctly.
    *   `aria-label` or `aria-describedby` attributes MUST be used for improved screen reader context, especially for error messages.
    *   Form elements MUST be correctly associated with their labels (e.g., using `for` and `id` attributes).
*   **Reference:** NFR-USAB-002, WCAG Compliance (Section 9)

**UXR-007: Email Format Validation (Client-side)**
*   **Description:** The email input field shall provide immediate client-side validation for basic email format, in addition to checking for emptiness.
*   **Acceptance Criteria:**
    *   If the email field content does not match a standard email regex pattern (e.g., `^[^\s@]+@[^\s@]+\.[^\s@]+$`) upon blur or submission, the message "Please enter a valid email address" MUST be displayed beneath the field.
    *   This validation MUST occur before server submission.
*   **Reference:** Best practice for user experience.

---

## 3. Persona Analysis and User Journeys

**Primary Persona: End User (e.g., Customer, Admin, Employee)**

*   **Archetype:** A user who needs to securely access the application to perform their role-specific tasks.
*   **Goals:**
    *   Quickly and easily log in to the application.
    *   Access their personalized dashboard/features.
    *   Receive clear feedback if login fails.
    *   Recover access if credentials are forgotten or account is locked.
*   **Motivations:** Efficiency, security, task completion, access to information/tools.
*   **Pain Points (Login & Auth specific):**
    *   Forgetting passwords.
    *   Unclear or technical error messages.
    *   Slow login times.
    *   Difficulty identifying input errors.
    *   Being locked out without clear instructions.
    *   Security concerns about personal data.

**Key User Journey: Login to the Application**

1.  **Entry Point:** User navigates to the application URL (e.g., from a bookmark, email link, or after session expiry).
    *   **Emotional State:** Intentional, sometimes frustrated if it's due to a timeout.
    *   **Design Consideration:** The login page must be immediately recognizable and inviting.
2.  **Input Credentials:** User sees the login form and enters their email address and password.
    *   **Emotional State:** Focused, expecting a smooth process.
    *   **Design Consideration:** Fields must be clearly labeled, input masking for password, optional password visibility toggle (UXR-003).
3.  **Attempt Login:** User clicks the "Login" button.
    *   **Emotional State:** Hopeful, expecting access.
    *   **Design Consideration:** Clear button styling (UXR-002), visual feedback on click and loading (UXR-005).

4.  **System Feedback & Resolution (Happy Path - UC-001):**
    *   System validates credentials.
    *   User sees a brief loading state.
    *   User is redirected to their role-specific dashboard.
    *   **Emotional State:** Satisfied, ready to work.
    *   **Design Consideration:** Fast redirection (NFR-PERF-001).

5.  **System Feedback & Resolution (Unsuccessful Path - Client-Side Validation - UC-003):**
    *   User clicks "Login" with empty fields.
    *   System immediately displays inline validation messages.
    *   **Emotional State:** Mild annoyance, but quickly corrected.
    *   **Design Consideration:** Instant, clear, inline error messages (UXR-004, UXR-001). Input fields highlight the error.

6.  **System Feedback & Resolution (Unsuccessful Path - Invalid Credentials - UC-002):**
    *   User clicks "Login" with incorrect credentials.
    *   System displays a global "Invalid credentials" error message.
    *   User may try again, or click "Forgot Password?".
    *   **Emotional State:** Frustration, confusion.
    *   **Design Consideration:** Clear, prominent global error message (FR-013, UXR-001). "Forgot Password?" link is easily discoverable (FR-017).

7.  **System Feedback & Resolution (Unsuccessful Path - Locked Account - UC-004):**
    *   User clicks "Login" with a locked account.
    *   System displays a global "Account is locked" error message.
    *   User may seek help or wait.
    *   **Emotional State:** Higher frustration, feeling helpless.
    *   **Design Consideration:** Specific, prominent global error message (FR-014, UXR-001) that might include guidance (e.g., "Please contact support or try again in 30 minutes").

---

## 4. Screen Inventory

Derived directly from the Use Case Analysis, focusing on screens requiring UI design within this specification.

1.  **Login Page:** (Primary focus of this spec)
    *   This is the initial entry point for all users requiring authentication.
    *   It encompasses all input, validation, loading, and error states related to the login process.
    *   **Derived from:** FR-001, FR-002, FR-003, FR-004, FR-005, FR-013, FR-014, FR-015, FR-017, UC-001, UC-002, UC-003, UC-004, UC-005.

2.  **Dashboard Pages (Customer, Admin, Employee):** (Target screens, design out of scope for *this* spec)
    *   These pages represent the successful destination after authentication (FR-011).
    *   They define the successful end-state of the login flow, but their internal design is assumed to be handled separately.

3.  **Forgot Password Page:** (Target screen, design out of scope for *this* spec)
    *   This is the destination when a user clicks the "Forgot Password?" link (FR-017, UC-005).
    *   Its design and functionality are assumed to be an existing or separately defined feature.

**Focus of this document:** The **Login Page** and its various states.

---

## 5. Screen Specifications: Login Page

### 5.1. Login Page - Default State

*   **Description:** The initial state of the login page before any user interaction or submission.
*   **Elements:**
    *   **Application Logo:** Prominently displayed at the top, centered (e.g., `brand-logo.svg`).
    *   **Header:** "Welcome Back" (H1, `typography-heading-h1`).
    *   **Subheader:** "Sign in to your account" (Body, `typography-body-lg`).
    *   **Email Address Input Field:**
        *   Label: "Email Address" (`typography-label`).
        *   Placeholder: "Enter your email" (`color-text-placeholder`).
        *   Initial state: Empty, default border (`border-input-default`).
    *   **Password Input Field:**
        *   Label: "Password" (`typography-label`).
        *   Placeholder: "Enter your password" (`color-text-placeholder`).
        *   Initial state: Empty, masked characters, default border (`border-input-default`).
        *   Icon: Password visibility toggle (`icon-visibility-off`).
    *   **Login Button:**
        *   Text: "Login" (`typography-button-primary`).
        *   Style: Primary button (`component-button-primary`).
        *   State: Active, clickable.
    *   **Forgot Password? Link:**
        *   Text: "Forgot Password?" (`typography-link`).
        *   Style: Underlined link (`component-link`).
        *   Alignment: Typically right-aligned or centered below the password field.

### 5.2. Login Page - Loading State

*   **Description:** The state immediately after the "Login" button is clicked and credentials are being submitted to the server.
*   **Elements:**
    *   **Login Button:**
        *   Text: "Logging in..." (or visually replaced by spinner).
        *   State: Disabled (`component-button-disabled`).
        *   Visual: Integrated spinner icon (`icon-spinner`) replaces button text or appears within the button, indicating ongoing process.
    *   **Other Form Elements:** Email and Password input fields, and "Forgot Password?" link are disabled (`component-input-disabled`, `component-link-disabled`) to prevent further interaction during submission.
    *   **Global Message Area:** Empty.

### 5.3. Login Page - Empty State

*   **Description:** This state is not applicable in the traditional sense, as it refers to a lack of data. For a login page, "empty" fields are handled by client-side validation, which falls under the "Validation State."

### 5.4. Login Page - Error States (Server-Side)

*   **Description:** States reflecting server-side authentication failures. These typically display a global error message.
*   **Common Elements:**
    *   **Global Error Message Area:** Located prominently (e.g., at the top of the form, below the subheader).
        *   Style: `component-alert-error`, `color-error-text`, `background-error-subtle`.
        *   Icon: `icon-error`.
    *   **Input Fields:** Return to default state, allowing user to correct input.
    *   **Login Button:** Returns to default active state.

#### 5.4.1. Error State: Invalid Credentials (UC-002, FR-013)

*   **Global Error Message:** "Invalid credentials"

#### 5.4.2. Error State: Account Locked (UC-004, FR-014)

*   **Global Error Message:** "Account is locked. Please try again in 30 minutes or contact support." (Adding actionability for user).

#### 5.4.3. Error State: Generic System Error (FR-015)

*   **Global Error Message:** "An unexpected error occurred. Please try again later."

### 5.5. Login Page - Validation State (Client-Side)

*   **Description:** States reflecting immediate client-side validation failures before server submission.
*   **Common Elements:**
    *   **Global Message Area:** Empty (no server errors yet).
    *   **Login Button:** Disabled (`component-button-disabled`) if validation fails.

#### 5.5.1. Validation State: Email Address Required (UC-003, FR-004)

*   **Email Address Input Field:**
    *   State: Error (`component-input-error`).
    *   Error Message: "Email address is required" (`color-error-text`, `typography-body-sm`) displayed immediately below the field.
*   **Password Input Field:** Default state (unless also empty).

#### 5.5.2. Validation State: Password Required (UC-003, FR-004)

*   **Password Input Field:**
    *   State: Error (`component-input-error`).
    *   Error Message: "Password is required" (`color-error-text`, `typography-body-sm`) displayed immediately below the field.
*   **Email Address Input Field:** Default state (unless also empty).

#### 5.5.3. Validation State: Invalid Email Format (UXR-007)

*   **Email Address Input Field:**
    *   State: Error (`component-input-error`).
    *   Error Message: "Please enter a valid email address" (`color-error-text`, `typography-body-sm`) displayed immediately below the field.

---

## 6. User Flows and Navigation

### 6.1. Main Flow: Successful Login (UC-001)

```mermaid
graph TD
    A[User lands on Login Page] --> B{Form Displayed: Email, Password, Login Btn, Forgot Password link};
    B --> C[User enters Email];
    C --> D[User enters Password];
    D --> E[User clicks Login Button];
    E --> F{Client-side validation: OK?};
    F -- Yes --> G[Login Button -> Loading State (UXR-005)];
    G --> H[Credentials submitted to server (FR-005)];
    H --> I{Server validates credentials (FR-006, FR-008, FR-009, FR-010): OK?};
    I -- Yes --> J[Redirect to Role-based Dashboard (FR-011)];
    J --> K[User on Dashboard];
```

### 6.2. Alternate Flow: Invalid Credentials (UC-002)

```mermaid
graph TD
    A[User lands on Login Page] --> B{Form Displayed};
    B --> C[User enters Email];
    C --> D[User enters Incorrect Password];
    D --> E[User clicks Login Button];
    E --> F{Client-side validation: OK?};
    F -- Yes --> G[Login Button -> Loading State (UXR-005)];
    G --> H[Credentials submitted to server];
    H --> I{Server validates credentials: Fails (FR-006)};
    I --> J[Increment failed attempt counter (FR-007)];
    J --> K[Login Button -> Default State];
    K --> L[Display Global Error: "Invalid credentials" (FR-013)];
    L --> M[User on Login Page (tries again or "Forgot Password")];
```

### 6.3. Alternate Flow: Empty Fields (UC-003)

```mermaid
graph TD
    A[User lands on Login Page] --> B{Form Displayed};
    B --> C[User leaves Email/Password field(s) empty];
    C --> D[User clicks Login Button];
    D --> E{Client-side validation: Fails (FR-004, UXR-004)};
    E --> F[Input field(s) highlight error];
    F --> G[Display inline error message(s): "Email address is required", "Password is required"];
    G --> H[Login Button -> Default/Disabled state];
    H --> I[User on Login Page (corrects input)];
```

### 6.4. Alternate Flow: Account Locked (UC-004)

```mermaid
graph TD
    A[User lands on Login Page] --> B{Form Displayed};
    B --> C[User enters Email/Password];
    C --> D[User clicks Login Button];
    D --> E{Client-side validation: OK?};
    E -- Yes --> F[Login Button -> Loading State (UXR-005)];
    F --> G[Credentials submitted to server];
    G --> H{Server checks account status: Locked (FR-008)};
    H --> I[Login Button -> Default State];
    I --> J[Display Global Error: "Account is locked" (FR-014)];
    J --> K[User on Login Page (waits or contacts support)];
```

### 6.5. Alternate Flow: Initiate Forgot Password (UC-005)

```mermaid
graph TD
    A[User lands on Login Page] --> B{Form Displayed};
    B --> C[User clicks "Forgot Password?" link (FR-017)];
    C --> D[Redirect to /forgot-password URL];
    D --> E[User on Forgot Password Page];
```

---

## 7. Component Mapping

This section maps the UI elements on the Login Page to generic components within the Design System (to be defined in `designsystem.md`).

*   **Application Logo:** `component-logo`
*   **Header (Welcome Back):** `typography-heading-h1`
*   **Subheader (Sign in...):** `typography-body-lg`
*   **Email Address Input Field:** `component-input-text`
    *   Labels: `typography-label`
    *   Placeholders: `color-text-placeholder`
    *   Error state: `component-input-error`, `color-error-text`
*   **Password Input Field:** `component-input-password`
    *   Labels: `typography-label`
    *   Placeholders: `color-text-placeholder`
    *   Password Visibility Toggle: `component-icon-button` with `icon-visibility` / `icon-visibility-off`
    *   Error state: `component-input-error`, `color-error-text`
*   **Login Button:** `component-button-primary`
    *   Loading state: `component-button-primary-loading` (includes `icon-spinner`)
    *   Disabled state: `component-button-disabled`
*   **Forgot Password? Link:** `component-link`
*   **Global Error Message Display:** `component-alert-error`
    *   Includes `icon-error`
    *   Text: `typography-body-sm`, `color-error-text`
*   **Inline Validation Messages:** `typography-body-sm`, `color-error-text`

---

## 8. Interaction Patterns and Micro-animations

*   **Input Field Focus:**
    *   **Interaction:** User clicks or tabs into an input field.
    *   **Animation:** Input field border changes from `border-input-default` to `border-input-focused` (e.g., thicker, accent color) with a `animation-duration-fast` transition. Placeholder text may recede or a floating label appears (if applicable to the chosen input style).
*   **Button Hover/Pressed:**
    *   **Interaction:** User hovers over the "Login" button, then clicks.
    *   **Animation:**
        *   **Hover:** Button background color subtly darkens or lightens (`background-primary-hover`) with `animation-duration-fast`.
        *   **Pressed:** Button slightly depresses or changes background more prominently (`background-primary-pressed`).
*   **Password Visibility Toggle:**
    *   **Interaction:** User clicks the eye icon in the password field.
    *   **Animation:** The icon instantly switches between `icon-visibility` and `icon-visibility-off`. No complex animation beyond state change.
*   **Login Button Loading State:**
    *   **Interaction:** User clicks "Login".
    *   **Animation:** Button text fades out (`animation-duration-fast`), and `icon-spinner` fades in and rotates continuously (`animation-spinner-rotate`). Button becomes unclickable.
*   **Error Message Appearance:**
    *   **Interaction:** Validation fails (client-side) or server returns error.
    *   **Animation:** Error messages (inline or global) should appear without jarring movement. A subtle slide-down or fade-in (`animation-duration-medium`) can be used, but prioritize immediate visibility.

---

## 9. Accessibility Requirements (WCAG Compliance)

Adherence to WCAG 2.1 AA standards for the Login Page.

*   **Perceivable:**
    *   **Color Contrast (WCAG 1.4.3):** All text (labels, placeholders, button text, error messages) and interactive elements MUST meet a minimum contrast ratio of 4.5:1 against their background. This includes `color-primary-text` on `background-page`, `color-error-text` on `background-error-subtle`, `color-button-primary-text` on `background-primary`, etc.
    *   **Non-Text Contrast (WCAG 1.4.11):** Visual presentation of UI components (e.g., input borders, password toggle icon) and graphical objects MUST have a contrast ratio of at least 3:1 against adjacent colors.
    *   **Resizable Text (WCAG 1.4.4):** Users MUST be able to resize text up to 200% without loss of content or functionality, without requiring assistive technology.
    *   **Reflow (WCAG 1.4.10):** Content MUST reflow vertically on small screens without requiring horizontal scrolling at 320 CSS pixels width.
*   **Operable:**
    *   **Keyboard Navigation (WCAG 2.1.1):** All interactive elements (inputs, buttons, links) MUST be operable via keyboard alone. Tab order MUST be logical (UXR-002).
    *   **Focus Indication (WCAG 2.4.7):** All interactive elements MUST have a clear and visible focus indicator when navigated via keyboard. This visual change MUST meet non-text contrast requirements (WCAG 1.4.11).
    *   **No Keyboard Trap (WCAG 2.1.2):** Users MUST be able to move focus away from any interactive element using standard keyboard navigation.
*   **Understandable:**
    *   **Labels and Instructions (WCAG 3.3.2):** Form elements MUST have clear and descriptive labels (e.g., `label` tags linked with `for` and `id` attributes for inputs). Instructions for input (e.g., error messages) MUST be provided when content is required (FR-004) or specific formatting is needed (UXR-007).
    *   **Error Identification (WCAG 3.3.1):** Input errors MUST be clearly identified to the user.
    *   **Error Suggestion (WCAG 3.3.3):** For input errors that are automatically detected, suggestions for correction MUST be provided (e.g., "Please enter a valid email address").
    *   **Consistent Navigation (WCAG 3.2.3):** The "Forgot Password?" link position and styling should be consistent if it appears on other pages.
*   **Robust:**
    *   **Parsing (WCAG 4.1.1):** HTML MUST be well-formed, valid, and free of major syntax errors.
    *   **Name, Role, Value (WCAG 4.1.2):** All UI components MUST have a programmatic `name`, `role`, and `value` so assistive technologies can interpret them. This includes:
        *   `type="email"` for email input.
        *   `type="password"` for password input.
        *   `aria-label` or `aria-labelledby` for the password visibility toggle.
        *   `aria-live="assertive"` for dynamic error messages to announce changes to screen readers.
        *   `aria-invalid="true"` attribute on input fields in an error state.

---

## 10. Responsive Design Breakpoints

The Login Page design must be responsive and adapt gracefully to various screen sizes.

*   **Extra Small (Mobile Portrait - < 576px):**
    *   **Layout:** Single-column layout. Form elements stack vertically.
    *   **Typography:** `font-size-body-sm` for labels/errors, `font-size-body-md` for input text, `font-size-heading-h2` for main header.
    *   **Spacing:** Reduced `spacing-md` between elements, `spacing-lg` for overall padding.
    *   **Form:** Input fields take 100% width. Button takes 100% width.
*   **Small (Mobile Landscape / Tablet Portrait - 576px - 768px):**
    *   **Layout:** Still primarily single-column, but with wider maximum content width for better readability.
    *   **Typography:** May slightly increase font sizes where appropriate, but generally consistent with mobile.
    *   **Spacing:** `spacing-lg` between elements, `spacing-xl` for padding.
    *   **Form:** Max width for the form container (e.g., 400px - 500px), centered.
*   **Medium (Tablet Landscape / Small Desktop - 768px - 992px):**
    *   **Layout:** Centered form with moderate max-width (e.g., 450px - 550px).
    *   **Typography:** `font-size-body-md` for labels/errors, `font-size-body-lg` for input text, `font-size-heading-h1` for main header.
    *   **Spacing:** `spacing-xl` between elements, `spacing-xxl` for padding.
*   **Large (Desktop - > 992px):**
    *   **Layout:** Centered form, potentially with more surrounding white space. Max-width for the form container (e.g., 500px - 600px).
    *   **Typography:** Standard desktop sizes as defined in Design System.
    *   **Spacing:** Ample use of `spacing-xxl` and `spacing-3xl` for a spacious feel.

The login form itself should generally remain a single column for simplicity and consistency, regardless of breakpoint, with its containing element adapting width.

---

### Conclusion

This Figma Design Specification provides a comprehensive blueprint for designing the Login Page, covering its various states, interactions, accessibility considerations, and responsive behavior. By adhering to these guidelines, the design team will create an intuitive, secure, and user-friendly login experience that aligns with the product's objectives and technical requirements. All components and styling references are to be sourced from the accompanying Design System specification.

---