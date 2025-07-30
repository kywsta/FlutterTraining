# Flutter Training Documentation

A simple web-based documentation viewer for Flutter training materials.

## Features

- **Sidebar Navigation**: Easy navigation through different Flutter topics
- **Markdown Rendering**: Displays markdown content with proper formatting
- **Responsive Design**: Works on desktop and mobile devices
- **Clean UI**: Modern, clean interface with good typography

## How to Use

1. **Open the Documentation**: Open `index.html` in your web browser
2. **Navigate**: Click on any topic in the sidebar to view its content
3. **Mobile**: On mobile devices, use the hamburger menu (☰) to toggle the sidebar

## Available Topics

The documentation includes the following sections:

### 1. Basic

- 1.1 Flutter Basics
- 1.2 Flutter Widgets
- 1.3 Flutter Slivers
- 1.4 Theme
- 1.5 Localization

### 3. Json Serialization

- JSON serialization in Flutter

### 4. Data Persistent

- Data persistence techniques

### 6. Navigation

- Navigation patterns in Flutter

### 7. Dependency Injection

- Dependency injection concepts

### 8. Architecture

- Clean Architecture and MVVM patterns

### 9. State Management

- State management with Bloc and Riverpod

### 11. Consuming Data

- Network calls and data consumption

### 12. Error Handling

- Error handling with Dartz

### 13. Push Notification

- Push notification implementation

### 15. Isolate

- Isolate usage in Flutter

## Technical Details

- **Pure HTML/CSS/JavaScript**: No external dependencies required
- **Markdown Parser**: Simple client-side markdown to HTML conversion
- **File Loading**: Uses fetch API to load markdown files
- **Responsive**: CSS Grid and Flexbox for responsive layout

## Browser Compatibility

Works in all modern browsers that support:

- ES6+ JavaScript
- Fetch API
- CSS Grid and Flexbox

## Local Development

To run this locally, you can use any simple HTTP server:

```bash
# Using Python 3
python -m http.server 8000

# Using Node.js (if you have http-server installed)
npx http-server

# Using PHP
php -S localhost:8000
```

Then open `http://localhost:8000` in your browser.

## File Structure

```
FlutterTraining/
├── index.html              # Main documentation viewer
├── README.md              # This file
├── course_detail.md       # Course outline
├── 1.1_flutter_basics.md # Flutter basics content
├── 1.2_flutter_widgets.md # Widgets content
├── 1.3_flutter_slivers.md # Slivers content
├── 1.4_theme.md          # Theme content
├── 1.5_localization.md   # Localization content
├── 3_json_serialization.md # JSON serialization content
├── 4_data_persistent.md  # Data persistence content
├── 6_navigation.md       # Navigation content
├── 7_dependency_injection.md # Dependency injection content
├── 8_architecture.md     # Architecture content
├── 9_state_management.md # State management content
├── 11_network_calls.md   # Network calls content
├── 12_error_handling.md  # Error handling content
├── 13_push_notification.md # Push notification content
└── 15_isolate.md         # Isolate content
```
