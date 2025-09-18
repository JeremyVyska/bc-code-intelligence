---
title: "VS Code AL Extension Keyboard Shortcuts"
domain: "sam-coder"
difficulty: "beginner"
bc_versions: "14+"
tags: ["vs-code", "productivity", "shortcuts", "al-extension", "navigation"]
prerequisites: ["vs-code-setup", "al-extension-basics"]
---

# VS Code AL Extension Keyboard Shortcuts

## Overview

Master essential keyboard shortcuts for AL development in VS Code to dramatically increase coding speed and navigation efficiency. These shortcuts are specifically optimized for Business Central AL development workflows.

**Key Principle**: Muscle memory for these core shortcuts reduces context switching and maintains coding flow state.

## Essential AL Development Shortcuts

### Code Generation & IntelliSense
- **Ctrl+Space**: Trigger IntelliSense suggestions for AL objects and properties
- **Ctrl+Shift+Space**: Parameter hints for procedure calls and method signatures
- **Tab**: Accept IntelliSense suggestion and move to next parameter
- **Ctrl+.**: Quick actions and refactoring suggestions (AL-specific fixes)

### Navigation & Symbol Management
- **F12**: Go to definition of AL object, procedure, or variable
- **Alt+F12**: Peek definition in popup window without navigation
- **Ctrl+F12**: Go to implementation (useful for interface implementations)
- **Shift+F12**: Find all references to current symbol across workspace
- **Ctrl+T**: Quick symbol search across entire AL workspace
- **Ctrl+Shift+O**: Navigate to symbol within current file
- **Ctrl+G**: Go to specific line number

### AL Object Operations
- **Ctrl+Shift+P → "AL: Go!"**: Quick navigation to any AL object by name
- **Ctrl+Shift+P → "AL: Download symbols"**: Refresh symbol references
- **F7**: Compile current AL file and show diagnostics
- **Ctrl+Shift+F7**: Compile entire AL project
- **F5**: Start debugging session with AL debugger

## Code Editing Efficiency

### Selection & Manipulation
- **Ctrl+D**: Select next occurrence of current word (multi-cursor editing)
- **Ctrl+Shift+L**: Select all occurrences of current word
- **Alt+Click**: Add cursor at click position (multi-cursor)
- **Ctrl+Shift+K**: Delete entire line
- **Alt+Up/Down**: Move line or selection up/down
- **Shift+Alt+Up/Down**: Copy line or selection up/down

### Code Structure Navigation
- **Ctrl+Shift+\\**: Jump to matching bracket/begin-end block
- **Ctrl+]**: Indent selected lines (AL code formatting)
- **Ctrl+[**: Outdent selected lines
- **Ctrl+K, Ctrl+C**: Comment selected lines
- **Ctrl+K, Ctrl+U**: Uncomment selected lines

### AL-Specific Formatting
- **Shift+Alt+F**: Format entire AL document according to AL formatting rules
- **Ctrl+K, Ctrl+F**: Format selected AL code block
- **Ctrl+Shift+I**: Format selection with AL-specific indentation

## Debugging & Problem Solving

### Debugging Navigation
- **F9**: Toggle breakpoint on current line
- **F5**: Start/continue debugging
- **F10**: Step over (next line in same procedure)
- **F11**: Step into (enter called procedure)
- **Shift+F11**: Step out (return to calling procedure)
- **Ctrl+Shift+F5**: Restart debugging session

### Problem Diagnosis
- **Ctrl+Shift+M**: Open Problems panel to view AL compilation errors
- **F8**: Go to next error/warning in current file
- **Shift+F8**: Go to previous error/warning in current file
- **Ctrl+Shift+U**: Open Output panel for AL compilation details

## Customization & Productivity

### Workspace Management
- **Ctrl+Shift+E**: Focus file explorer (AL object tree)
- **Ctrl+Shift+F**: Global search across AL workspace
- **Ctrl+Shift+H**: Global find and replace across AL files
- **Ctrl+,**: Open settings for AL extension configuration

### Custom Keybinding Patterns
Most AL developers customize these common patterns:
- **AL object creation shortcuts**: Quick access to create new tables, pages, codeunits
- **Snippet triggers**: Custom shortcuts for frequently used AL code patterns
- **Build and deploy combinations**: Single-key publishing to sandbox environments

## Best Practices

### Learning Strategy
Start with 5-7 core shortcuts and add 2-3 new ones weekly until they become automatic. Focus on navigation shortcuts first, then code generation, finally debugging workflows.

### Muscle Memory Development
Practice shortcuts during routine tasks rather than trying to memorize them separately. The most valuable shortcuts save 3+ seconds per use and occur multiple times per hour.

### Team Consistency
Establish team standards for essential shortcuts to enable pair programming and knowledge sharing. Document custom keybindings in project documentation.

*Complete shortcut reference: samples/vs-code-shortcuts.md*
*Keybinding customization: samples/keybindings-json.md*
*AL extension configuration: sam-coder-index.json#vs-code-setup*