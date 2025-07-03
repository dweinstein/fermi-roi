# Fermi ROI Prioritization App

## Concept

> We want to create an app that will enable an executive/leadership team to ultimately create a stack ranked list of actions to take within the current quarter. To achieve this we want to use a Fermi ROI (learned from this article https://longform.asmartbear.com/roi-rubric/) approach, basically provide order of magnitude for revenue measured in ARR, effort measured in time for a number of Actions (provided as a list collected from each member of the group) grouped/nested by Strategic Items (configurable list of items including Customer value, pipeline generation, growing partnerships). We need the ability to get everyone's input on order of magnitude for revenue, order of magnitude for time to complete with the ability to produce a sorted list. Allow for additional measured dimensions to be added by the admin. 

> Make it so that there is a concept of different user types of the system: admin, contributor, and viewer. Enable the admin to construct a link with a pre-populated name/email to send to the user as either Contributor or Viewer. The link can be copied and shared with the expected recipient. A contributor can construct a link for viewers (with the intended viewer email/name). Viewers can only share their link with other viewers (without changing the recipient name/email that shows up in the page).

> The app must be safe/secure and use best practices in regard to any REST, URL parameters, software components/libraries leveraged, handling sensitive data, threat modeling, input sanitization, SQL injection, etc.
> The app should feel similar to excalidraw in terms of the lack of requirement to log in and persistence of data primarily in-browser dstorage.

## Overview

A browser-centric collaborative decision-making tool for teams to create stack-ranked lists of strategic actions using order-of-magnitude (fermi) estimation principles. The app follows local-first architecture principles and is delivered as a single HTML file.

## Core Architecture Requirements

### Local-First Software Principles
- **Data lives in browser**: Use localStorage for persistence, works offline
- **No accounts needed**: Instant start, URL-based sessions
- **Event-sourced data**: Immutable event log for conflict-free updates. Multiple contributors must be able to contribute at the same time.
- **URL-based access control**: Admin/Edit/View modes via URL hash parameters
- **Client-side mainly**: Minimal back end requirements, primarily static file deployment
- **Slick, simple UI**: fun, intuitive, easy to use interface.
- **Avoid modals**: Use only modals if needed and if it's more intuitive.

## Core Features

### Strategic Action Management
- Add strategic actions with title, description, and category assignment
- Visual order-of-magnitude estimation scales:
  - **Yearly Revenue Impact**: $0/yr -> $1k/yr -> $10k/yr -> $100K/yr -> $1M/yr
  - **Implementation Effort**: 0 days -> 1 day → 10 days → 100 days -> 1000 days
  - **Strateic Impact**: 0-> 1 → 10 → 100 -> 1000
- Automatic ROI calculation and ranking (Revenue Impact / Implementation Effort)
- Real-time sorting by ROI score
- Ability to track which user contributed the action, and for each action, who provided which estimate.

#### Core Value Proposition

- Zero Setup: No accounts, no installations - instant start
- Browser-First: Data lives in your browser, works offline
- Simple Sharing: Share via URLs with embedded permissions
- Real-time Collaboration: Live editing when multiple people work on the same session
- Privacy-Focused: Your data stays local unless explicitly shared

### Strategic Categories System
- **Default categories**: Customer Value (blue), Pipeline Generation (green), Growing Partnerships (orange), Strategic (purlpe)
- **Admin-only category management**: Add custom categories with names, descriptions, and color selection
- **Color-coded visualization**: Categories shown with colored dots throughout the interface

### Access Control System (CRITICAL REQUIREMENTS)
URL format: `#[userType]/:userName/[sessionId]/[accessKey]`

- Access keys should not be guessable. Use HMAC to generate a valid key that validates the URL.

**Admin** (`#admin/abc123/sess123/key789`):
- Full read/write access to everything
- Can add/edit actions and estimates
- Can manage strategic categories (add new ones)
- Can generate both Contributor and Viewer share links
- Generated links should request a Name for who the link will be shared with.

**Contributor** (`#edit/abc123/sess456/key456`):
- Can add/edit actions and estimates
- Cannot manage categories
- Can ONLY generate Viewer links (cannot create Contributor links)
- Links should request a Name for who the link will be shared with.
- Can see overall reporting and analytics as well export information.

**Viewer** (`#view/abc123/sess678/key123`):
- Read-only access to all data
- Viewers know they are read-only.
- Cannot edit anything
- Cannot generate any new share links
- Share menu shows "Viewers cannot generate new links"
- Can see overall reporting and analytics as well export information.

### Action and Estimation Management

Action Creation:
- Select strategic category
- Title, description, strategic category assignment
- Drag-and-drop reordering and categorization for admins.
- Bulk import from CSV/text paste

Estimation Interface:

- Visual order-of-magnitude scales (1K, 10K, 100K, 1M, etc.)
- Confidence indicators (1-5 scale)
- Collaborative estimation with multiple contributors
- Real-time estimate aggregation and display
- Ability to edit and save estimations.

### Version Control & History

- Complete History: Every change creates immutable commit in local history

### 4. Analytics Dashboard
- ROI rankings with numbered list
- Category analysis showing action count and average ROI per category
- Toggle between Actions view and Analytics view

### 5. Data Management
- Export session data as JSON file
- Event-sourced storage in localStorage
- Session persistence across browser restarts
- Real-time updates when multiple people use same session

## Technical Implementation Requirements

### Delivery Format
- **Single HTML file** inline CSS and JavaScript
- **Uses React via CDN**: React 18
- **Simple CSS**: For styling
- **Lucide icons** 

### Event-Sourced Data Model
```javascript
// Event types that modify state:
ACTION_ADDED: { action: {...} }
ESTIMATE_ADDED: { actionId, estimate: {...} }
CATEGORY_ADDED: { category: {...} }

// State reconstruction from events
// Current state built by applying all events in order
```

### Storage Schema
```javascript
// localStorage key: fermi_session_[sessionId]
{
  userName: '...',
  contributorId: '<uuid>',
  events: [...], // Complete event history
  lastModified: timestamp
}
```

### Core Data Structures
```javascript

Contributor: {
  id, name, description, email,
  createdAt
}

Action: {
  id, title, description, categoryId, estimates[],
  createdAt, createdBy
}

Estimate: {
  id, actionId, dimensionId, contributorId,
  estimateValue, confidenceLevel, createdAt
}

Category: {
  id, name, description, color, displayOrder, isActive
}
```

## User Interface Requirements

### Header
- App title with target icon
- Access mode indicator (Admin/Contributor/Viewer)
- Actions/Analytics toggle button
- Share drop down with access-controlled options
- Categories button (Admin only)
- Export button (json)

### Actions View
- "Add new strategic action" button (non-viewers)
- Action cards showing:
  - Ranking number and ROI score
  - Category color dot and name
  - Title and description
  - Current estimates summary
  - "Add/Edit Estimates" button (non-viewers)
- Estimation interface with 5-point visual scales
- Form validation and keyboard shortcuts

### Analytics View
- ROI Rankings: Top 10 actions with scores
- Category Analysis: Grid showing action count and average ROI per category

### Category Management Modal (Admin Only)
- List current categories with colors
- Add new category form with:
  - Name field (required)
  - Description field (optional)
  - Color picker with 10 predefined colors
- Form validation and keyboard shortcuts (Enter to save, Escape to cancel)

## Access Control Implementation Details

### Share Menu Behavior
```javascript
// Admin can generate both link types
if (accessMode === 'admin') {
  showOptions: ['Copy Contributor Link', 'Copy Viewer Link']
}

// Contributors can only generate viewer links
if (accessMode === 'edit') {
  showOptions: ['Copy Viewer Link']
}

// Viewers cannot generate any links
if (accessMode === 'view') {
  showMessage: 'Viewers cannot generate new links'
}
```

### URL Generation
```javascript
generateShareableUrl(mode) {
  baseUrl + '#' + mode + '/' + sessionId + '/key_' + randomKey
}
```

## Deployment & Usage

### File Structure
```
index.html (single file containing everything)
```

### Local Development Setup
```bash
# Save HTML file to directory
# Start Python server
python3 -m http.server 8000

# Optional: Share via ngrok
ngrok http 8000
```

### Browser Compatibility
- Modern browsers with localStorage support

## Success Criteria

### Functional Requirements
- ✅ Create and estimate strategic actions
- ✅ Automatic ROI ranking
- ✅ Category management (admin only)
- ✅ Secure access control (viewers can only view existing data)
- ✅ Local data persistence
- ✅ Ability to Edit estimates
- ✅ Export functionality
- ✅ Real-time collaboration foundation

### Technical Requirements
- ✅ Single HTML file deployment
- ✅ Works offline
- ✅ Minimal server dependencies
- ✅ Local-first architecture

### User Experience
- ✅ Zero setup - works immediately
- ✅ Intuitive interface
- ✅ Fast, responsive interactions
- ✅ Clear visual hierarchy

