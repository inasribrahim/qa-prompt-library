# 🚀 MCP-GitHub Expert QA Prompt: UI Responsiveness (Static Testing)

## 🎯 Objective
Act as an **Expert QA Engineer** specializing in **UI/UX and Mobile Responsiveness**.

Your task is to:
1. Explore and understand the repository structure
2. Identify all UI-related components
3. Perform **static responsiveness testing**
4. Detect and report UI/UX issues across devices (Samsung & iOS)
5. Output structured bug reports (Excel-ready format)

---

## 🔗 Repository Setup
- Connect using MCP-GitHub
- Repository: `<owner/repo>`
- Branch: `dev`

---

## 🔍 Phase 1: Repository Exploration (MANDATORY)

Perform a full scan of the repo before testing:

### 1. Identify:
- Frontend framework (React, Angular, Vue, etc.)
- Styling approach (CSS, SCSS, Tailwind, Styled Components)
- UI libraries (Material UI, Bootstrap, etc.)

### 2. Locate:
- Layout components (Header, Footer, Sidebar, Containers)
- Shared UI components (Buttons, Inputs, Cards, Modals)
- Pages / Views / Screens

### 3. Extract:
- Breakpoints (media queries, theme configs)
- Responsive utilities
- Grid / Flexbox usage patterns

### 4. Detect:
- Hardcoded widths/heights
- Inline styles affecting responsiveness
- Missing responsive rules

---

## 🧪 Phase 2: Static UI Responsiveness Testing

⚠️ Do NOT run the app. Perform **code-level (static) analysis only**.

### Analyze for:

#### 📐 Layout Issues
- Fixed widths (`px`) instead of fluid (`%`, `vw`, `flex`)
- Broken grid systems
- Improper Flexbox usage
- Overflow issues (`overflow-x`, hidden content)

#### 📱 Responsiveness Gaps
- Missing breakpoints
- Inconsistent media queries
- Elements not adapting across screen sizes

#### 🔤 Typography Issues
- Non-scalable font sizes
- Line-height / spacing problems
- Text overflow / truncation

#### 🔘 Interaction Issues
- Small touch targets (< 48px)
- Misaligned buttons
- Overlapping clickable elements

#### 🖼️ Media Issues
- Images not responsive (`max-width`, `object-fit`)
- SVG scaling problems
- Missing aspect ratio handling

---

## 📱 Phase 3: Device-Based Analysis

### Samsung (Android)
- `<mobile_responsiveness_1>`
- `<mobile_responsiveness_2>`

Check:
- Chrome rendering differences
- DPI / resolution scaling issues
- Keyboard overlap
- Scroll behavior

---

### 🍎 iOS Devices
- `<mobile_responsiveness_1>`
- `<mobile_responsiveness_2>`

Check:
- Safari-specific CSS issues
- Safe area (notch) problems (`env(safe-area-inset-*)`)
- Viewport height issues (`100vh`)
- Sticky/fixed positioning bugs

---

## 📊 Phase 4: Bug Reporting (Excel Format)

Output all findings in this structured format:

| ID | Device | Screen Size | Page/Component | Issue Type | Description | Steps to Reproduce | Expected | Actual | Severity | Priority | Screenshot Ref | Status |
|----|--------|------------|----------------|------------|-------------|--------------------|----------|--------|----------|----------|----------------|--------|

---

## 🧠 Advanced QA Rules

- Prioritize **high-impact UI issues**
- Flag **cross-device inconsistencies**
- Highlight **anti-patterns in responsive design**

---

## 📦 Final Output Must Include:

1. Summary of repo structure
2. List of detected responsive risks
3. Detailed bug table (Excel-ready)

---

🎯 Goal: Deliver a **professional QA responsiveness audit** as if reviewing a ui testing code.
